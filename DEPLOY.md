# 🚀 Guide de Déploiement du Profil d'Organisation GitHub

Ce répertoire contient la configuration du profil pour l'organisation GitHub **velyon-core**.

Pour rendre ce profil visible sur la page d'accueil de votre organisation (https://github.com/velyon-core), suivez ces étapes simples.

---

## Étape 1 : Créer le dépôt sur GitHub

1. Allez sur votre compte GitHub.
2. Créez un nouveau dépôt au sein de votre organisation en cliquant sur le bouton **New repository** (ou rendez-vous directement sur https://github.com/organizations/velyon-core/repositories/new).
3. Nommez le dépôt exactement **`.github`**.
4. **Important** :
   * Le dépôt doit être **Public** (les profils d'organisation ne fonctionnent pas s'ils sont privés).
   * Ne l'initialisez pas avec un fichier README, `.gitignore` ou une Licence (laissez ces options décochées).

---

## Étape 2 : Initialiser et Pousser le Code depuis votre machine

Exécutez les commandes suivantes dans votre terminal pour lier ce dossier à votre dépôt distant et y pousser les fichiers :

```bash
# Placez-vous dans le dossier racine du profil
cd /Users/oumarou/Documents/velyon-github-profile

# Initialiser le dépôt git local
git init

# Ajouter tous les fichiers (le dossier .github/ et ce guide DEPLOY.md)
git add .

# Créer le premier commit
git commit -m "Initialisation du profil d'organisation velyon-core"

# Définir la branche principale sur main
git branch -M main

# Ajouter l'adresse de votre dépôt distant GitHub
git remote add origin https://github.com/velyon-core/.github.git

# Pousser le code vers le dépôt distant
git push -u origin main
```

Une fois cette commande exécutée avec succès, visitez https://github.com/velyon-core et vous verrez le profil magnifiquement affiché avec les diagrammes d'architecture, la stack technique et les détails des services !
