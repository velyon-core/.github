<p align="center">
  <img src="profile/assets/banner.png" alt="Velyon Banner" width="100%" style="border-radius: 10px;">
</p>

<h1 align="center">🚀 Velyon Core</h1>

<p align="center">
  <strong>Engineering high-performance, resilient, and secure financial technology.</strong>
</p>

<p align="center">
  Bienvenue sur le hub d'ingénierie de <strong>Velyon Core</strong>, l'équipe technique à l'origine d'une passerelle de paiement mobile money hautement performante et résiliente.
</p>

<p align="center">
  Cette page présente l'architecture globale, la stack technique, les services critiques et les choix d'ingénierie qui garantissent la scalabilité, la sécurité et la haute disponibilité de notre plateforme.
</p>

---

## 🛠️ Stack Technique

Notre infrastructure et nos applications sont construites avec des technologies modernes, choisies pour leur performance brute, leur typage fort et leur intégration dans l'écosystème Cloud Serverless.

| Couche | Technologies | Détails & Versions |
| :--- | :--- | :--- |
| **Backend API** | ![Go](https://img.shields.io/badge/Go-1.25-00ADD8?style=flat-square&logo=go&logoColor=white) ![GORM](https://img.shields.io/badge/GORM-v2-blue?style=flat-square) | Code natif (`net/http`), GORM v2, Logs structurés (`slog`), Zéro framework HTTP externe. |
| **Traitement Asynchrone** | ![Redis](https://img.shields.io/badge/Redis-Asynq-DC382D?style=flat-square&logo=redis&logoColor=white) | Files d'attente distribuées basées sur Redis via la bibliothèque `Asynq`. |
| **Frontend** | ![Vue.js](https://img.shields.io/badge/Vue.js-3-4FC08D?style=flat-square&logo=vue.js&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white) | Single Page Application réactive, graphiques temps réel avec `Chart.js` & `Lucide Icons`. |
| **Bases de Données** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-AWS_Aurora-4169E1?style=flat-square&logo=postgresql&logoColor=white) | Rapprochement de données, indexation transactionnelle et réplication Multi-AZ. |
| **Recherche & Analytics** | ![OpenSearch](https://img.shields.io/badge/AWS_OpenSearch-Search-005EB8?style=flat-square&logo=opensearch&logoColor=white) | Indexation asynchrone des transactions pour la recherche rapide et les agrégations du dashboard. |
| **DevOps & Infra** | ![Terraform](https://img.shields.io/badge/Terraform-IaC-844FBA?style=flat-square&logo=terraform&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-ECS_Fargate-FF9900?style=flat-square&logo=amazon-aws&logoColor=white) | Infrastructure as Code (IaC) complète. Déploiement conteneurisé serverless (ECS Fargate). |

---

## 🏗️ Schémas d'Architecture

### 1. Architecture Infrastructure (AWS Cloud)
Notre infrastructure est entièrement hébergée sur Amazon Web Services (AWS) et orchestrée par Terraform. Elle repose sur des services managés pour éliminer la maintenance de serveurs et garantir une scalabilité horizontale automatique.

```mermaid
graph TB
    %% Definitions
    Client([Public Clients / Merchants])
    
    subgraph Cloudflare_Edge ["Cloudflare Edge Security & CDN"]
        CF_DNS["Cloudflare DNS"]
        CF_WAF["Cloudflare WAF / DDoS Shield"]
        CF_CDN["Cloudflare CDN (Edge Caching)"]
    end

    subgraph AWS_Cloud ["AWS Cloud (eu-west-3)"]
        subgraph VPC ["AWS VPC (Virtual Private Cloud)"]
            subgraph Public_Subnets ["Public Subnets (Multi-AZ)"]
                ALB["AWS Application Load Balancer (ALB)"]
            end
            
            subgraph Private_App_Subnets ["Private Application Subnets"]
                subgraph ECS_Cluster ["AWS ECS Fargate Cluster"]
                    Go_API["Go Web Service (API Container)"]
                    Asynq_Worker["Asynq Background Workers"]
                end
            end
            
            subgraph Private_Data_Subnets ["Private Data Subnets (Isolated)"]
                subgraph Aurora_Cluster ["AWS Aurora PostgreSQL (Multi-AZ)"]
                    DB_Primary[("Aurora Primary (Write)")]
                    DB_Replica[("Aurora Replica (Read Only)")]
                end
                
                Redis[("AWS ElastiCache Redis (Cache & Queue)")]
                OpenSearch[("AWS OpenSearch (Analytics & Search)")]
            end
        end
        
        subgraph Storage_Tier ["Storage Layer"]
            S3[("AWS S3 Bucket (Exports & Proofs)")]
        end
    end

    %% Network flows
    Client -->|HTTPS / DNS Request| CF_DNS
    CF_DNS --> CF_WAF
    CF_WAF --> CF_CDN
    CF_CDN -->|Proxy HTTPS Traffic| ALB
    
    ALB -->|Forward Request| Go_API
    
    %% Application dependencies
    Go_API -->|Read/Write Session & Cache| Redis
    Go_API -->|Write Transactions| DB_Primary
    Go_API -.->|Read Analytics| DB_Replica
    
    Asynq_Worker -->|Process Queue Tasks| Redis
    Asynq_Worker -->|Read/Write Data| DB_Primary
    Asynq_Worker -->|Index Data| OpenSearch
    Go_API -.->|Query Search & Analytics| OpenSearch
    Go_API -->|Store Exports & Files| S3
    
    %% Styling
    classDef edgeStyle fill:#FFF2E6,stroke:#FF8000,stroke-width:2px;
    classDef awsStyle fill:#F2F5FA,stroke:#FF9900,stroke-width:2px;
    classDef vpcStyle fill:#FFF,stroke:#4A90E2,stroke-width:2px,stroke-dasharray: 5 5;
    
    class CF_DNS,CF_WAF,CF_CDN edgeStyle;
    class AWS_Cloud awsStyle;
    class VPC vpcStyle;
```

---

### 2. Architecture Logicielle (Clean Architecture Backend)
Le code backend en Go suit strictement les principes de la **Clean Architecture**, assurant une séparation claire des responsabilités et une testabilité maximale. Les dépendances pointent uniquement vers l'intérieur (vers le Domain), et toutes les implémentations externes sont injectées au démarrage.

```mermaid
graph TD
    subgraph Presentation_Layer ["Presentation Layer (HTTP API)"]
        Handlers["HTTP Handlers / Controllers"]
        Middlewares["Middlewares (Auth, Rate Limit)"]
    end
    
    subgraph Application_Layer ["Application Layer (Use Cases)"]
        UseCases["Use Cases (Business Logic)"]
        DTOs["Data Transfer Objects (DTOs)"]
    end
    
    subgraph Domain_Layer ["Domain Layer (Core Entities)"]
        Entities["Domain Entities"]
        Interfaces["Repository & Provider Interfaces"]
    end
    
    subgraph Infrastructure_Layer ["Infrastructure Layer (External Services)"]
        GORM["PostgreSQL Repository (GORM v2)"]
        AsynqQueue["Redis Queue & Caching (Asynq)"]
        PaymentClients["Third-Party Payment Providers"]
    end
    
    %% Dependency flows (unidirectional pointing inwards)
    Handlers --> UseCases
    Middlewares --> Handlers
    UseCases --> Entities
    UseCases --> Interfaces
    
    %% Implementations (pointing inwards via interface dependency injection)
    GORM -.->|Implements| Interfaces
    AsynqQueue -.->|Implements| Interfaces
    PaymentClients -.->|Implements| Interfaces
```

---

### 3. Flux Transactionnel Optimisé (Hot Path)
Pour maintenir un temps de réponse moyen **inférieur à 300 ms** sur l'initiation des paiements (Hot Path), l'API valide d'abord les données via Redis avant d'effectuer une transaction unique dans la base de données PostgreSQL Aurora. Les opérations lourdes (appels API opérateurs, webhooks) sont immédiatement déportées dans la file d'attente asynchrone.

```mermaid
sequenceDiagram
    autonumber
    actor Client as API Client / Merchant
    participant API as Go API Service (Fargate)
    participant Redis as ElastiCache Redis
    participant DB as Aurora PostgreSQL
    participant Queue as Redis Queue (Asynq)
    participant Operator as Mobile Money Operator

    Client->>API: POST /v1/pay-in (API Key & Payload)
    
    rect rgb(240, 240, 240)
        note over API, Redis: 1. Hot-Path Validation & Caching
        API->>Redis: Get API Key / Client Profile (TTL 5m)
        Redis-->>API: Valid Profile
        API->>Redis: Get Payment Method Status (TTL 10m)
        Redis-->>API: Method Available
        API->>Redis: Check Wallet Balance & Limits (TTL 30s)
        Redis-->>API: Balance Valid
    end
    
    rect rgb(240, 240, 240)
        note over API, DB: 2. Atomic Database Transaction
        API->>DB: Create Transaction (PENDING) & Update Wallet
        DB-->>API: Success (UUID v7 & NanoID Ref)
    end
    
    API->>Redis: Invalidate Wallet Balance Cache
    
    rect rgb(240, 240, 240)
        note over API, Queue: 3. Async Task Delegation
        API->>Queue: Enqueue Provider Pay-in Job (Critical Priority)
        Queue-->>API: Job Enqueued
    end
    
    API-->>Client: 201 Created (Ref, Status: PENDING)
    
    note over Queue, Operator: Async Background Execution (Asynq Worker)
    Queue->>Operator: Send debit/credit request (Orange, MTN, Wave...)
    Operator-->>Queue: Callback / Status Update
```

---

## ⚙️ Services Critiques

### 1. Système de Queue et Traitement Asynchrone
Toutes les tâches complexes ou sujettes aux pannes externes sont encapsulées dans des jobs persistés sous Redis. 

*   **At-least-once delivery** : Chaque tâche est garantie d'être exécutée au moins une fois.
*   **Idempotence** : Tous les workers sont conçus pour être relancés en cas d'erreur sans causer d'effets de bord financiers.
*   **Observabilité** : Suivi rigoureux du statut des tâches, de leur durée d'exécution et de l'historique des erreurs.

#### Priorités des files d'attente
| Type de Job | Déclencheur | Priorité | Garantie |
| :--- | :--- | :--- | :--- |
| **Appel Fournisseur** | Création de transaction pay-in/payout | **Critique** | Exécution immédiate |
| **Livraison Webhook** | Changement de statut de transaction | **Critique** | Retry avec backoff exponentiel |
| **Réessai Webhook** | Échec de livraison précédent | **Haute** | Intervalle progressif |
| **Indexation OpenSearch** | Écriture de données en base | **Normale** | Traitement sous < 2s |
| **Vérification Statut Provider** | Post-transaction différée (polling) | **Normale** | Exécution en tâche de fond |
| **Réconciliation & Rapports** | Tâches périodiques ou demande manuelle | **Basse** | Exécution asynchrone |

---

### 2. Suivi des Soldes (Wallets) & Réconciliation
La précision financière est assurée par un modèle transactionnel strict :

*   **Atomicité** : Le solde du portefeuille (Wallet) et le relevé détaillé de l'opération sont écrits dans la **même transaction de base de données**.
*   **Dirty Tracking** : Seuls les champs modifiés sont écrits afin d'éviter tout écrasement concurrent ou perte de données.
*   **Piste d'Audit** : Chaque ligne de relevé enregistre l'état exact du solde *avant* et *après* l'opération, créant un historique inaltérable.
*   **Réconciliation Automatique** : Un moteur planifié compare régulièrement le solde calculé (somme des relevés système) avec le solde communiqué par les opérateurs partenaires. Tout écart déclenche une alerte critique immédiate.

---

### 3. Cycle de Vie et Sécurité des Webhooks
Les webhooks notifient en temps réel les serveurs marchands des changements de statut des transactions.

```mermaid
graph LR
    Tx[Transaction complétée] --> Job[Planification du Job de Webhook]
    Job --> Attempt[Tentative de livraison HTTP POST]
    Attempt -->|HTTP 200/201| Success[Webhook marqué comme SUCCESS]
    Attempt -->|HTTP Autre / Timeout| Retry{Nb essais < 10?}
    Retry -->|Oui| Schedule[Attente avec Backoff Exponentiel]
    Schedule --> Attempt
    Retry -->|Non| Fail[Webhook marqué comme FAILED - Notification Admin]
```

#### Signature & Intégrité
Pour chaque webhook envoyé, nous incluons des en-têtes de sécurité stricts pour permettre au marchand de valider l'authenticité de la requête :
*   `X-Velyon-Signature` : Signature **HMAC-SHA256** du corps du payload utilisant la clé secrète du webhook du marchand (`webhook_secret`).
*   `X-Velyon-Timestamp` : Horodatage Unix de l'envoi pour prévenir les attaques par rejeu (Replay Attacks).
*   `X-Velyon-Event` : Type d'événement déclencheur (ex: `transaction.completed`).

#### Politique de relance (Exponential Backoff)
Si le serveur du marchand ne répond pas avec un statut HTTP 2xx, nous réessayons jusqu'à 10 fois sur un intervalle cumulé de près de **49 heures** :
1. Immédiat (0s)
2. +10s (cumul : 10s)
3. +1min (cumul : 1m 10s)
4. +5min (cumul : 6m 10s)
5. +15min (cumul : 21m 10s)
6. +1h (cumul : 1h 21m)
7. +4h (cumul : 5h 21m)
8. +8h (cumul : 13h 21m)
9. +12h (cumul : 25h 21m)
10. +24h (cumul : 49h 21m)

---

## 🔒 Sécurité et Conformité

Notre plateforme met en œuvre les standards de sécurité les plus stricts de l'industrie fintech :

*   **Chiffrement des Données** :
    *   *Au repos* : Chiffrement **AES-256-GCM** de tous les secrets sensibles (clés API opérateurs, identifiants marchands) stockés en base de données.
    *   *En transit* : Communication exclusivement sous **TLS 1.2+** avec des suites de chiffrement sécurisées.
*   **Authentification et Autorisations** :
    *   *API Publique* : Authentification par paire de clés API robustes (`API Key` + `API Secret`).
    *   *Backoffice* : Jetons **JWT** signés et sécurisés avec double authentification obligatoire (**OTP - One-Time Password**) pour tous les administrateurs et collaborateurs.
*   **Bonnes Pratiques de Développement** :
    *   Validation stricte de toutes les entrées utilisateur pour se prémunir des injections SQL et attaques XSS.
    *   Comparaison de signatures cryptographiques en **temps constant** pour prévenir les attaques par timing (Timing Attacks).
    *   Aucun secret n'est écrit en clair dans le code source ou dans les dépôts GitHub (utilisation d'AWS Secrets Manager et variables d'environnement chiffrées).
    *   *Soft Delete* systématique pour préserver l'historique et éviter les pertes accidentelles de données.

---

## 📈 Objectifs de Service (SLA & SLI)

| Métrique | Cible / Objectif |
| :--- | :--- |
| **Disponibilité globale de l'API (Uptime)** | **99.9%** |
| **Temps de réponse moyen (Hot Path)** | **< 300 ms** |
| **RPO (Recovery Point Objective)** | **< 5 minutes** (perte maximale tolérée en cas de sinistre) |
| **RTO (Recovery Time Objective)** | **< 15 minutes** (temps de rétablissement complet du service) |

---

*Pour toute question concernant l'utilisation des APIs ou les contributions de code, veuillez vous référer aux documentations internes ou contacter l'équipe d'ingénierie à dev@velyon.com.*
