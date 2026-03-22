# 🎓 Formation Git — De Zéro à Expert

> **Objectif :** Maîtriser Git, du premier commit à la gestion de projets complexes en équipe.
> **Format :** Chaque concept est expliqué, illustré par des commandes concrètes, avec des schémas ASCII.

---

# 📋 Sommaire

- [PARTIE 1 — Les Bases (Débutant)](#partie-1--les-bases)
  - [1.1 C'est quoi Git ?](#11-cest-quoi-git-)
  - [1.2 Installation & Configuration](#12-installation--configuration)
  - [1.3 Les 3 Zones de Git](#13-les-3-zones-de-git)
  - [1.4 Premier Repo & Premier Commit](#14-premier-repo--premier-commit)
  - [1.5 Visualiser l'Historique](#15-visualiser-lhistorique)
  - [1.6 Annuler et Corriger](#16-annuler-et-corriger)
  - [1.7 .gitignore](#17-gitignore)
- [PARTIE 2 — Les Branches (Intermédiaire)](#partie-2--les-branches)
  - [2.1 Concept de Branche](#21-concept-de-branche)
  - [2.2 Créer, Basculer, Supprimer](#22-créer-basculer-supprimer)
  - [2.3 Merge (Fusion)](#23-merge-fusion)
  - [2.4 Résoudre les Conflits](#24-résoudre-les-conflits)
  - [2.5 Rebase](#25-rebase)
  - [2.6 Merge vs Rebase](#26-merge-vs-rebase)
- [PARTIE 3 — Travail en Équipe (Intermédiaire+)](#partie-3--travail-en-équipe)
  - [3.1 Remotes (Dépôts Distants)](#31-remotes-dépôts-distants)
  - [3.2 Clone, Push, Pull, Fetch](#32-clone-push-pull-fetch)
  - [3.3 Pull Requests / Merge Requests](#33-pull-requests--merge-requests)
  - [3.4 Stratégies de Branching](#34-stratégies-de-branching)
  - [3.5 Tags & Releases](#35-tags--releases)
- [PARTIE 4 — Techniques Avancées (Avancé)](#partie-4--techniques-avancées)
  - [4.1 Stash](#41-stash)
  - [4.2 Cherry-Pick](#42-cherry-pick)
  - [4.3 Rebase Interactif](#43-rebase-interactif)
  - [4.4 Bisect (Debug par dichotomie)](#44-bisect)
  - [4.5 Reflog (Filet de sécurité)](#45-reflog)
  - [4.6 Submodules & Subtrees](#46-submodules--subtrees)
  - [4.7 Worktrees](#47-worktrees)
- [PARTIE 5 — Niveau Expert](#partie-5--niveau-expert)
  - [5.1 Les Internals de Git](#51-les-internals-de-git)
  - [5.2 Hooks](#52-hooks)
  - [5.3 Git en CI/CD](#53-git-en-cicd)
  - [5.4 Monorepo vs Multirepo](#54-monorepo-vs-multirepo)
  - [5.5 Signer ses Commits (GPG)](#55-signer-ses-commits-gpg)
  - [5.6 Situations de Crise](#56-situations-de-crise)
- [PARTIE 6 — Bonnes Pratiques & Conventions](#partie-6--bonnes-pratiques--conventions)
  - [6.1 Messages de Commit](#61-messages-de-commit)
  - [6.2 Conventional Commits](#62-conventional-commits)
  - [6.3 Workflow Quotidien Idéal](#63-workflow-quotidien-idéal)
  - [6.4 Cheat Sheet](#64-cheat-sheet)

---

# PARTIE 1 — Les Bases

## 1.1 C'est quoi Git ?

**Git** est un système de **contrôle de version distribué** créé par Linus Torvalds en 2005 (le créateur de Linux). Il permet de :

- **Suivre l'historique** de chaque modification de ton code (qui a changé quoi, quand, pourquoi).
- **Collaborer** à plusieurs sur le même projet sans se marcher dessus.
- **Expérimenter** en toute sécurité via des branches (si ça casse, on revient en arrière).
- **Sauvegarder** son travail de manière fiable et distribuée.

### Distribué vs Centralisé

```
CENTRALISÉ (SVN, ancien) :                DISTRIBUÉ (Git) :
  Un serveur central                        Chaque dev a une copie COMPLÈTE
  Tu dois être connecté                     Tu travailles en local (offline OK)
  Le serveur tombe = tout s'arrête          Le serveur tombe = tu continues

     ┌─────────┐                          ┌────┐   ┌────┐   ┌────┐
     │ Serveur │                          │Dev1│   │Dev2│   │Dev3│
     └────┬────┘                          │FULL│   │FULL│   │FULL│
     ┌────┼────┐                          │REPO│   │REPO│   │REPO│
     ▼    ▼    ▼                          └──┬─┘   └──┬─┘   └──┬─┘
   Dev1  Dev2  Dev3                           └────┬───┘       │
   (pas de copie                                   ▼           │
    complète)                              ┌──────────┐        │
                                           │  Remote  │◄───────┘
                                           │ (GitHub) │
                                           └──────────┘
```

---

## 1.2 Installation & Configuration

### Installation

```bash
# macOS
brew install git

# Ubuntu/Debian
sudo apt install git

# Windows
# Télécharger sur https://git-scm.com/download/win
# ou via winget :
winget install Git.Git
```

### Configuration initiale (à faire une seule fois)

```bash
# Ton identité — apparaît dans chaque commit
git config --global user.name "Ton Nom"
git config --global user.email "ton.email@exemple.com"

# Éditeur par défaut (pour les messages de commit, rebase interactif...)
git config --global core.editor "code --wait"   # VS Code
# git config --global core.editor "vim"          # Vim
# git config --global core.editor "nano"         # Nano

# Branche par défaut (convention moderne)
git config --global init.defaultBranch main

# Couleurs dans le terminal
git config --global color.ui auto

# Alias utiles (optionnel mais recommandé)
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all --decorate"

# Vérifier ta config
git config --list
```

---

## 1.3 Les 3 Zones de Git

C'est LE concept fondamental à comprendre. Git gère ton code dans **3 zones** :

```
┌───────────────┐    git add     ┌───────────────┐   git commit   ┌───────────────┐
│  Working Dir  │ ──────────────→│  Staging Area │ ─────────────→│  Repository   │
│  (ton code)   │                │  (index)      │                │  (historique) │
│               │                │               │                │               │
│ Fichiers      │                │ Fichiers      │                │ Commits       │
│ modifiés      │                │ prêts à être  │                │ enregistrés   │
│               │                │ committés     │                │ définitivement│
└───────────────┘                └───────────────┘                └───────────────┘
                    ← git restore                   ← git reset
```

| Zone | Rôle | Analogie |
|------|------|---------|
| **Working Directory** | Tes fichiers tels que tu les vois dans ton éditeur | Ton bureau de travail |
| **Staging Area (Index)** | Zone de préparation — ce qui sera dans le prochain commit | Le carton "prêt à envoyer" |
| **Repository (.git)** | L'historique complet de tous les commits | L'archive permanente |

**Pourquoi une Staging Area ?** Elle te permet de **choisir précisément** ce qui va dans chaque commit. Tu as modifié 5 fichiers, mais seuls 3 sont liés à ta feature ? Tu stages les 3, tu commites. Les 2 autres restent dans le Working Dir pour un autre commit.

---

## 1.4 Premier Repo & Premier Commit

### Créer un repo

```bash
# Méthode 1 : Initialiser un nouveau repo
mkdir mon-projet
cd mon-projet
git init
# → Crée un dossier caché .git (le repository)

# Méthode 2 : Cloner un repo existant
git clone https://github.com/user/repo.git
cd repo
```

### Le cycle fondamental : edit → stage → commit

```bash
# 1. Créer/modifier des fichiers
echo "# Mon Projet" > README.md
echo "print('hello')" > main.py

# 2. Voir l'état de ton repo
git status
# → README.md et main.py sont "untracked" (Git ne les connaît pas encore)

# 3. Stager (préparer) les fichiers pour le commit
git add README.md        # ajoute un fichier précis
git add main.py
# ou
git add .                # ajoute TOUT (attention !)

# 4. Vérifier ce qui est stagé
git status
# → "Changes to be committed: README.md, main.py"

# 5. Commiter (enregistrer dans l'historique)
git commit -m "feat: initialisation du projet"
# → Crée un snapshot permanent de tes fichiers stagés
```

### Comprendre git add en détail

```bash
git add fichier.txt       # ajoute un fichier précis
git add dossier/          # ajoute tout un dossier
git add .                 # ajoute TOUT le répertoire courant
git add -A                # ajoute tout (y compris les suppressions)
git add -p                # mode interactif — choisir ligne par ligne (PUISSANT)
```

`git add -p` est un outil pro : il te montre chaque modification et te demande si tu veux la stager. Parfait pour séparer les changements en commits logiques.

---

## 1.5 Visualiser l'Historique

```bash
# Historique complet
git log

# Historique compact (1 ligne par commit)
git log --oneline

# Historique avec graphe des branches
git log --oneline --graph --all --decorate

# Les 5 derniers commits
git log -5

# Historique d'un fichier précis
git log -- chemin/vers/fichier.py

# Qui a modifié chaque ligne ? (blame)
git blame fichier.py

# Voir le contenu d'un commit
git show abc1234

# Voir les différences
git diff                    # working dir vs staging
git diff --staged           # staging vs dernier commit
git diff main..feature      # entre deux branches
git diff abc1234..def5678   # entre deux commits
```

---

## 1.6 Annuler et Corriger

C'est la partie la plus importante pour un débutant. Git te donne un filet de sécurité.

### Annuler des modifications non stagées

```bash
# Restaurer un fichier à la version du dernier commit
git restore fichier.txt

# Restaurer TOUT le working directory (⚠️ perte des modifications)
git restore .
```

### Retirer du staging (unstage)

```bash
# Retirer un fichier du staging (le garder modifié dans le working dir)
git restore --staged fichier.txt
```

### Corriger le dernier commit

```bash
# Modifier le message du dernier commit
git commit --amend -m "nouveau message"

# Ajouter des fichiers oubliés au dernier commit
git add fichier_oublie.txt
git commit --amend --no-edit    # garde le même message
```

### Revenir en arrière dans l'historique

```bash
# SAFE — Crée un NOUVEAU commit qui annule un ancien (préserve l'historique)
git revert abc1234

# DESTRUCTIF — Revenir à un commit précis (modifie l'historique)
git reset --soft abc1234    # garde les fichiers stagés
git reset --mixed abc1234   # garde les fichiers dans le working dir (défaut)
git reset --hard abc1234    # ⚠️ SUPPRIME tout — retour total à ce commit
```

```
reset --soft  : Repo ← revient    | Staging ✅ gardé  | Working Dir ✅ gardé
reset --mixed : Repo ← revient    | Staging ← vidé    | Working Dir ✅ gardé
reset --hard  : Repo ← revient    | Staging ← vidé    | Working Dir ← vidé ⚠️
```

---

## 1.7 .gitignore

Fichier qui dit à Git quels fichiers/dossiers **ignorer** (ne jamais tracker).

```gitignore
# Fichier .gitignore à la racine du repo

# Dépendances
node_modules/
vendor/
__pycache__/
*.pyc
venv/

# Build
dist/
build/
bin/
obj/
*.exe
*.dll
*.class

# IDE
.idea/
.vscode/
*.swp
*.swo
.DS_Store
Thumbs.db

# Environnement
.env
.env.local
*.env

# Logs
*.log
logs/

# Secrets (JAMAIS dans Git !)
*.pem
*.key
credentials.json
```

```bash
# Si un fichier est déjà tracké et que tu l'ajoutes au .gitignore :
git rm --cached fichier.env    # retirer du tracking sans supprimer le fichier
git commit -m "chore: retirer fichier.env du tracking"
```

---

# PARTIE 2 — Les Branches

## 2.1 Concept de Branche

Une branche est simplement un **pointeur** vers un commit. C'est ultraléger (pas de copie de fichiers). Créer une branche = créer un pointeur.

```
Avant :
  main → [A] ← [B] ← [C]

Après git branch feature :
  main    → [C]
  feature → [C]  ← les deux pointent au même endroit

Après un commit sur feature :
  main    → [C]
  feature → [C] ← [D]  ← feature a avancé, main n'a pas bougé
```

**Pourquoi les branches ?** Elles permettent de travailler sur une fonctionnalité, un bugfix, une expérimentation **sans toucher** au code principal. Si ça marche → merge. Si ça échoue → supprime la branche.

---

## 2.2 Créer, Basculer, Supprimer

```bash
# Voir les branches
git branch              # branches locales
git branch -a           # toutes (locales + remote)
git branch -v           # avec le dernier commit de chaque branche

# Créer une branche
git branch feature-login

# Basculer sur une branche
git switch feature-login         # moderne (Git 2.23+)
# git checkout feature-login     # ancienne syntaxe (fonctionne toujours)

# Créer ET basculer en une commande
git switch -c feature-login
# git checkout -b feature-login  # ancienne syntaxe

# Supprimer une branche (mergée)
git branch -d feature-login

# Supprimer une branche (non mergée — forcer)
git branch -D feature-login

# Renommer la branche courante
git branch -m nouveau-nom
```

---

## 2.3 Merge (Fusion)

Intégrer les modifications d'une branche dans une autre.

### Fast-Forward Merge (linéaire)

Quand la branche cible n'a pas avancé — Git déplace simplement le pointeur.

```
AVANT :
  main    → [A] ← [B]
  feature → [A] ← [B] ← [C] ← [D]

git switch main
git merge feature

APRÈS (fast-forward) :
  main    → [A] ← [B] ← [C] ← [D]  ← main a juste avancé
  feature → [A] ← [B] ← [C] ← [D]
```

### Three-Way Merge (avec commit de merge)

Quand les deux branches ont divergé — Git crée un commit de fusion.

```
AVANT :
  main    → [A] ← [B] ← [E]
  feature → [A] ← [B] ← [C] ← [D]

git switch main
git merge feature

APRÈS :
  main → [A] ← [B] ← [E] ← [M]  ← commit de merge
                  ↖ [C] ← [D] ↗
```

```bash
# Merge standard
git switch main
git merge feature-login

# Merge sans fast-forward (force un commit de merge même si possible)
git merge --no-ff feature-login

# Annuler un merge en cours (si conflit)
git merge --abort
```

---

## 2.4 Résoudre les Conflits

Un conflit arrive quand **deux branches ont modifié la même ligne** du même fichier. Git ne peut pas décider seul — il te demande de trancher.

```
CONFLIT dans fichier.py :

<<<<<<< HEAD (ta branche actuelle)
def greet():
    return "Bonjour"
=======
def greet():
    return "Hello"
>>>>>>> feature-login (la branche qu'on merge)
```

### Processus de résolution

```bash
# 1. Git te dit qu'il y a un conflit
git merge feature-login
# CONFLICT (content): Merge conflict in fichier.py

# 2. Ouvre le fichier, choisis la bonne version
# Supprime les marqueurs <<<<<<< ======= >>>>>>>
# Garde le code correct

# 3. Marque comme résolu
git add fichier.py

# 4. Termine le merge
git commit    # Git pré-remplit le message de merge

# Alternative : annuler tout et recommencer
git merge --abort
```

### Outils visuels pour les conflits

```bash
# Lancer un outil de merge visuel
git mergetool

# Configurer VS Code comme outil de merge
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

---

## 2.5 Rebase

Le rebase **rejoue tes commits** au-dessus d'une autre branche, créant un historique linéaire.

```
AVANT :
  main    → [A] ← [B] ← [E]
  feature → [A] ← [B] ← [C] ← [D]

git switch feature
git rebase main

APRÈS :
  main    → [A] ← [B] ← [E]
  feature → [A] ← [B] ← [E] ← [C'] ← [D']
  (C' et D' sont des COPIES de C et D, re-basées sur E)
```

```bash
# Rebase de ta branche sur main
git switch feature
git rebase main

# En cas de conflit pendant le rebase
git rebase --continue    # après avoir résolu le conflit
git rebase --abort       # annuler tout le rebase
git rebase --skip        # ignorer le commit problématique
```

---

## 2.6 Merge vs Rebase

| | Merge | Rebase |
|---|---|---|
| Historique | Préserve l'historique réel (branches visibles) | Crée un historique linéaire (plus propre) |
| Commits | Crée un commit de merge | Pas de commit supplémentaire |
| Sûreté | ✅ Non destructif | ⚠️ Réécrit l'historique |
| Conflits | À résoudre une fois | À résoudre commit par commit |
| Quand ? | Branches partagées, PRs | Branches locales, nettoyage |

### La règle d'or du rebase

```
⚠️ NE JAMAIS REBASE une branche qui a été PUSHÉE et que d'autres utilisent.
   Le rebase réécrit l'historique → chaos pour les autres.

✅ Rebase ta branche LOCALE sur main avant de push.
❌ Ne rebase JAMAIS main sur ta branche si d'autres travaillent sur main.
```

### Workflow recommandé

```bash
# Tu travailles sur ta branche feature
git switch feature
# ... des commits ...

# Avant de merger/PR, mets-toi à jour sur main
git fetch origin
git rebase origin/main    # rebase ta branche sur le dernier main

# Puis push (force push car historique réécrit)
git push --force-with-lease    # plus sûr que --force
```

---

# PARTIE 3 — Travail en Équipe

## 3.1 Remotes (Dépôts Distants)

Un remote est un repo Git hébergé ailleurs (GitHub, GitLab, Bitbucket, serveur privé).

```bash
# Voir les remotes configurés
git remote -v

# Ajouter un remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git  # fork

# Renommer un remote
git remote rename origin github

# Supprimer un remote
git remote remove upstream
```

---

## 3.2 Clone, Push, Pull, Fetch

```bash
# CLONE — copier un repo distant en local
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git          # SSH (recommandé)
git clone --depth 1 https://...                  # clone superficiel (rapide)

# FETCH — télécharger les changements SANS les appliquer
git fetch origin                # récupère tout
git fetch origin main           # récupère une branche spécifique
# → Met à jour origin/main mais NE TOUCHE PAS ta branche locale

# PULL — fetch + merge (récupère ET applique)
git pull origin main
# Équivalent à :
#   git fetch origin
#   git merge origin/main

# PULL avec rebase (historique plus propre)
git pull --rebase origin main
# Équivalent à :
#   git fetch origin
#   git rebase origin/main

# PUSH — envoyer tes commits vers le remote
git push origin main
git push origin feature-login

# Première push d'une nouvelle branche (établit le tracking)
git push -u origin feature-login
# Après ça, tu peux juste faire : git push

# Force push (après un rebase — ⚠️ avec prudence)
git push --force-with-lease    # refuse si quelqu'un a pushé entre temps
```

### Schéma des flux

```
     Local                          Remote (GitHub)
┌──────────────┐                ┌──────────────┐
│ main (local) │ ──── push ───→│ origin/main  │
│              │ ←── fetch ────│              │
│              │ ←── pull ─────│              │
└──────────────┘                └──────────────┘

pull  = fetch + merge (ou rebase)
push  = envoie tes commits
fetch = met à jour ta vue du remote sans toucher à tes branches
```

---

## 3.3 Pull Requests / Merge Requests

Les PR (GitHub) / MR (GitLab) sont le mécanisme central de collaboration. Ce n'est pas une fonctionnalité Git native, mais des plateformes (GitHub, GitLab, Bitbucket).

### Workflow d'une Pull Request

```
1. Crée une branche feature
   git switch -c feature/login

2. Travaille, commite
   git add . && git commit -m "feat: login form"

3. Push vers le remote
   git push -u origin feature/login

4. Ouvre une PR sur GitHub/GitLab
   → Compare feature/login → main
   → Ajoute description, reviewers, labels

5. Code review
   → Les collègues relisent, commentent, suggèrent des changements
   → Tu appliques les retours, push de nouveaux commits

6. Merge (par le reviewer ou toi)
   → Squash and merge (recommandé) : combine tous tes commits en un seul
   → ou Merge commit : garde l'historique détaillé
   → ou Rebase and merge : linéaire, pas de commit de merge

7. Supprime la branche
   → La branche feature n'a plus de raison d'exister
```

---

## 3.4 Stratégies de Branching

### Git Flow (complexe, projets avec releases)

```
main ──────────────────────────────────────────── (production)
  ↓                              ↑
develop ──────────────────────────── (intégration)
  ↓          ↑      ↓          ↑
feature/A ───┘    feature/B ───┘
                                 ↓
                           release/1.0 ──→ main (tag v1.0)
                                 ↓
                           hotfix/bug ──→ main + develop
```

| Branche | Rôle |
|---|---|
| `main` | Production, toujours stable |
| `develop` | Intégration des features |
| `feature/*` | Une feature par branche |
| `release/*` | Préparation de release (bug fixes) |
| `hotfix/*` | Correction urgente en production |

### GitHub Flow (simple, déploiement continu)

```
main ──────────────────────────────────── (toujours déployable)
  ↓                    ↑
feature/login ─────────┘ (PR → merge → déploiement auto)
  ↓                    ↑
fix/header ────────────┘
```

Règles : tout part de `main`, tout revient via PR, `main` est toujours déployable.

### Trunk-Based Development (équipes expérimentées, CI/CD fort)

```
main ──────────────────────────────────── (tronc commun)
  ↓↑    ↓↑    ↓↑
  petite branche (1-2 jours max, ou commit direct sur main)
```

Tout le monde travaille sur `main` (ou des branches très courtes). Nécessite une excellente CI et des feature flags.

### Quelle stratégie choisir ?

```
Seul / petit projet             → GitHub Flow
Équipe avec releases planifiées → Git Flow
Équipe expérimentée + CI/CD     → Trunk-Based
```

---

## 3.5 Tags & Releases

Les tags marquent un **point précis** dans l'historique (version, release).

```bash
# Créer un tag léger (juste un pointeur)
git tag v1.0.0

# Créer un tag annoté (recommandé — contient message, date, auteur)
git tag -a v1.0.0 -m "Version 1.0.0 — première release stable"

# Tagger un commit spécifique
git tag -a v0.9.0 abc1234 -m "Release 0.9.0"

# Lister les tags
git tag
git tag -l "v1.*"    # filtrer

# Voir les détails d'un tag
git show v1.0.0

# Pusher les tags
git push origin v1.0.0       # un tag spécifique
git push origin --tags        # tous les tags

# Supprimer un tag
git tag -d v1.0.0             # local
git push origin --delete v1.0.0  # remote
```

### Semantic Versioning (SemVer)

```
v MAJOR . MINOR . PATCH
  1     . 2     . 3

MAJOR : changements incompatibles (breaking changes)
MINOR : nouvelles fonctionnalités (rétro-compatible)
PATCH : corrections de bugs
```

---

# PARTIE 4 — Techniques Avancées

## 4.1 Stash

Met de côté tes modifications **temporairement** sans commiter. Comme un brouillon.

```bash
# Stash les modifications en cours
git stash
git stash -m "WIP: formulaire de login"   # avec un message

# Stash incluant les fichiers non trackés
git stash -u

# Lister les stashes
git stash list
# stash@{0}: On feature: WIP: formulaire de login
# stash@{1}: On main: debug temporaire

# Récupérer le dernier stash
git stash pop        # applique ET supprime du stash
git stash apply      # applique SANS supprimer (garde dans la pile)

# Récupérer un stash spécifique
git stash pop stash@{1}

# Supprimer un stash
git stash drop stash@{0}
git stash clear      # supprime TOUT
```

**Cas d'usage typique :** Tu travailles sur feature-A, et un collègue te demande de review feature-B. Tu `stash` ton travail, tu switch, tu review, puis tu reviens et tu `pop`.

---

## 4.2 Cherry-Pick

Prendre **un commit spécifique** d'une branche et l'appliquer sur une autre.

```bash
# Appliquer un commit sur ta branche courante
git cherry-pick abc1234

# Cherry-pick sans commiter (juste appliquer les changements)
git cherry-pick --no-commit abc1234

# Cherry-pick de plusieurs commits
git cherry-pick abc1234 def5678 ghi9012
```

**Cas d'usage :** Un bugfix a été fait sur `develop` et tu en as besoin sur `main` immédiatement, sans merger toute la branche.

---

## 4.3 Rebase Interactif

Le **couteau suisse** de Git. Permet de réécrire l'historique : réordonner, squasher, modifier, supprimer des commits.

```bash
# Rebase interactif sur les 4 derniers commits
git rebase -i HEAD~4
```

Git ouvre un éditeur avec la liste des commits :

```
pick abc1234 feat: ajouter le formulaire
pick def5678 fix: typo dans le formulaire
pick ghi9012 feat: validation du formulaire
pick jkl3456 fix: autre typo

# Commandes :
# pick   = garder le commit tel quel
# reword = garder le commit mais modifier le message
# squash = fusionner avec le commit précédent (garder les messages)
# fixup  = fusionner avec le commit précédent (jeter le message)
# drop   = supprimer le commit
# edit   = s'arrêter pour modifier le commit
```

### Exemple : nettoyer avant une PR

```
# AVANT (historique brouillon) :
abc1234 feat: login
def5678 fix: typo
ghi9012 WIP
jkl3456 fix: oups
mno7890 feat: validation

# Tu modifies en :
pick abc1234 feat: login
fixup def5678 fix: typo        ← fusionné dans le commit login
fixup ghi9012 WIP              ← fusionné aussi
fixup jkl3456 fix: oups        ← fusionné aussi
pick mno7890 feat: validation

# APRÈS (historique propre) :
abc1234 feat: login (contient tout le travail)
mno7890 feat: validation
```

---

## 4.4 Bisect

Trouver **quel commit a introduit un bug** par recherche dichotomique. Git teste automatiquement les commits du milieu.

```bash
# 1. Lancer le bisect
git bisect start

# 2. Indiquer un commit avec le bug (récent)
git bisect bad          # le commit actuel est mauvais

# 3. Indiquer un commit sans le bug (ancien)
git bisect good v1.0.0  # cette version marchait

# 4. Git checkout un commit au milieu — tu testes
# "Est-ce que le bug est présent ?"
git bisect good    # si pas de bug → Git cherche plus récent
git bisect bad     # si bug → Git cherche plus ancien

# 5. Après quelques itérations, Git te dit le commit coupable !
# abc1234 is the first bad commit

# 6. Terminer
git bisect reset

# BONUS : bisect automatique avec un script de test
git bisect run ./test.sh
# Git teste automatiquement chaque commit avec ton script
```

**Puissance :** Sur 1000 commits, bisect trouve le coupable en ~10 étapes (log2(1000)).

---

## 4.5 Reflog

Le **filet de sécurité ultime**. Le reflog enregistre TOUT ce que tu fais (même les reset --hard, les rebases ratés). Tant que c'est dans le reflog (~90 jours), tu peux récupérer.

```bash
# Voir le reflog
git reflog
# abc1234 HEAD@{0}: commit: feat: nouvelle feature
# def5678 HEAD@{1}: reset: moving to HEAD~1
# ghi9012 HEAD@{2}: commit: le commit que j'ai perdu
# jkl3456 HEAD@{3}: rebase: ...

# Récupérer un commit "perdu"
git reset --hard HEAD@{2}    # revenir à l'état de HEAD@{2}

# Ou créer une branche de récupération
git branch recovery HEAD@{2}
```

**Le reflog t'a sauvé ?** Tu n'es pas le premier. C'est le meilleur ami du développeur qui fait des `reset --hard` trop vite.

---

## 4.6 Submodules & Subtrees

Inclure un repo Git **à l'intérieur** d'un autre repo.

### Submodules (pointeur vers un commit d'un autre repo)

```bash
# Ajouter un submodule
git submodule add https://github.com/lib/utils.git libs/utils

# Cloner un repo avec submodules
git clone --recurse-submodules https://...

# Mettre à jour les submodules
git submodule update --remote

# Initialiser les submodules après un clone normal
git submodule init
git submodule update
```

### Subtrees (copie du code dans le repo)

```bash
# Ajouter un subtree
git subtree add --prefix=libs/utils https://github.com/lib/utils.git main --squash

# Mettre à jour
git subtree pull --prefix=libs/utils https://github.com/lib/utils.git main --squash
```

| | Submodule | Subtree |
|---|---|---|
| Stockage | Pointeur (référence) | Copie du code |
| Clone | Nécessite `--recurse-submodules` | Automatique |
| Complexité | Plus complexe | Plus simple |
| Offline | Nécessite accès au repo externe | Le code est là |

---

## 4.7 Worktrees

Avoir **plusieurs working directories** pour le même repo. Tu peux travailler sur 2 branches simultanément sans stash ni switch.

```bash
# Créer un worktree pour une branche
git worktree add ../mon-projet-hotfix hotfix/urgent
# → Crée un dossier séparé avec la branche hotfix checked out

# Lister les worktrees
git worktree list

# Supprimer un worktree
git worktree remove ../mon-projet-hotfix
```

**Cas d'usage :** Tu travailles sur une feature complexe et un hotfix urgent arrive. Au lieu de stash + switch, tu ouvres un worktree séparé pour le hotfix.

---

# PARTIE 5 — Niveau Expert

## 5.1 Les Internals de Git

Comprendre les entrailles de Git te donne un pouvoir de debug unique.

### Les 4 types d'objets Git

Git est fondamentalement un **système de fichiers adressé par contenu**. Tout est un objet identifié par son hash SHA-1.

```
1. BLOB   — contenu d'un fichier (pas le nom, juste le contenu)
2. TREE   — un répertoire (liste de blobs et d'autres trees)
3. COMMIT — un snapshot (pointe vers un tree + metadata)
4. TAG    — un pointeur annoté vers un commit

Commit → Tree → Blobs
  ↓
Parent commit(s)
```

```bash
# Explorer les objets
git cat-file -t abc1234    # type de l'objet (commit, tree, blob)
git cat-file -p abc1234    # contenu de l'objet

# Voir le tree d'un commit
git cat-file -p HEAD          # montre le commit
git cat-file -p HEAD^{tree}   # montre l'arbre de fichiers
```

### Les références (refs)

Les branches, tags, HEAD sont des **fichiers texte** qui contiennent un hash SHA-1.

```bash
cat .git/HEAD                 # ref: refs/heads/main
cat .git/refs/heads/main      # abc1234... (hash du dernier commit)
cat .git/refs/tags/v1.0.0     # def5678... (hash du commit tagué)
```

---

## 5.2 Hooks

Les hooks sont des **scripts** exécutés automatiquement à certains événements Git. Ils vivent dans `.git/hooks/`.

### Hooks côté client (locaux)

| Hook | Quand ? | Usage |
|---|---|---|
| `pre-commit` | Avant chaque commit | Lint, formatage, tests rapides |
| `commit-msg` | Après écriture du message | Valider le format du message |
| `pre-push` | Avant chaque push | Tests complets, vérifications |
| `post-merge` | Après un merge | npm install, migrations |

### Exemple : pre-commit hook

```bash
#!/bin/sh
# .git/hooks/pre-commit (doit être exécutable : chmod +x)

echo "🔍 Vérification du code avant commit..."

# Linter
npx eslint --fix .
if [ $? -ne 0 ]; then
    echo "❌ ESLint a trouvé des erreurs. Commit annulé."
    exit 1
fi

# Tests rapides
npm test -- --bail
if [ $? -ne 0 ]; then
    echo "❌ Tests échoués. Commit annulé."
    exit 1
fi

echo "✅ Tout est OK. Commit autorisé."
```

### Outils pour les hooks

Les hooks dans `.git/hooks/` ne sont PAS versionnés (ils sont dans `.git`). Pour partager les hooks en équipe, utilise un outil :

- **Husky** (Node.js) : `npx husky install`, hooks dans `.husky/`
- **pre-commit** (Python) : `.pre-commit-config.yaml`
- **Lefthook** (Go) : `lefthook.yml`

---

## 5.3 Git en CI/CD

Git est au cœur des pipelines CI/CD (Continuous Integration / Continuous Deployment).

```
Push sur main → CI/CD Pipeline :
  1. git clone → récupérer le code
  2. Build → compiler, bundler
  3. Test → tests unitaires, intégration, E2E
  4. Analyse → lint, sécurité (SAST), couverture
  5. Deploy → staging, puis production

Push sur feature/* → Pipeline léger :
  1. Build + Tests
  2. Preview environment (optionnel)
```

### Stratégies de déploiement

```
BLUE-GREEN :
  Deux environnements identiques. On déploie sur le "vert" (inactif),
  on teste, puis on switch le traffic.
  ✅ Rollback instantané (switch vers l'ancien)

CANARY :
  Déployer progressivement : 5% du trafic, puis 25%, puis 50%, puis 100%.
  Si erreur → rollback immédiat.
  ✅ Risque limité

ROLLING UPDATE :
  Remplacer les instances une par une.
  ✅ Pas de downtime, ❌ deux versions coexistent temporairement
```

---

## 5.4 Monorepo vs Multirepo

| | Monorepo | Multirepo |
|---|---|---|
| Structure | Un seul repo pour tout le projet | Un repo par service/package |
| Exemples | Google, Meta, Uber | Netflix (historiquement), beaucoup de PMEs |
| Avantages | Refactoring atomique, partage de code facile, CI unifiée | Autonomie des équipes, repos légers, permissions granulaires |
| Inconvénients | Repo énorme, CI complexe, tooling spécial | Duplication, mises à jour transversales difficiles |
| Outils | Nx, Turborepo, Bazel, Lerna | GitHub Organizations, npm packages |

```
Monorepo :                    Multirepo :
my-project/                   my-org/
├── packages/                   ├── frontend/     (repo séparé)
│   ├── frontend/               ├── backend-api/  (repo séparé)
│   ├── backend/                ├── shared-lib/   (repo séparé)
│   └── shared-lib/             └── infra/        (repo séparé)
├── apps/
│   ├── web/
│   └── mobile/
└── package.json
```

---

## 5.5 Signer ses Commits (GPG)

Prouver cryptographiquement que c'est bien toi qui as fait le commit (badge "Verified" sur GitHub).

```bash
# 1. Générer une clé GPG
gpg --full-generate-key
# Choisir RSA, 4096 bits

# 2. Lister tes clés
gpg --list-secret-keys --keyid-format=long
# sec   rsa4096/ABC123DEF456 2024-01-01

# 3. Configurer Git
git config --global user.signingkey ABC123DEF456
git config --global commit.gpgsign true   # signer par défaut

# 4. Exporter la clé publique (pour GitHub/GitLab)
gpg --armor --export ABC123DEF456
# → Copier dans GitHub > Settings > SSH and GPG keys

# 5. Commiter (signé automatiquement)
git commit -m "feat: feature signée"
git log --show-signature    # vérifier la signature
```

---

## 5.6 Situations de Crise

### "J'ai push un secret/mot de passe !"

```bash
# 1. URGENCE : Révoquer le secret immédiatement (API key, token, mot de passe)
# Le code est peut-être déjà dans le cache de GitHub

# 2. Supprimer de l'historique avec git filter-repo (recommandé)
pip install git-filter-repo
git filter-repo --path fichier-secret.env --invert-paths

# 3. Force push
git push --force --all
git push --force --tags

# 4. Demander à GitHub de purger le cache
# → Contact GitHub support ou utilise les outils de scanning
```

### "J'ai fait un reset --hard et perdu mes commits !"

```bash
# Le reflog est ton ami
git reflog
# Trouve le commit perdu
git reset --hard HEAD@{N}    # revenir à cet état
```

### "Mon merge est un désastre !"

```bash
# Si le merge n'est pas encore commité
git merge --abort

# Si le merge est commité mais pas pushé
git reset --hard HEAD~1

# Si le merge est pushé (ne PAS réécrire l'historique partagé)
git revert -m 1 <merge_commit_hash>
```

---

# PARTIE 6 — Bonnes Pratiques & Conventions

## 6.1 Messages de Commit

Un bon message de commit est **essentiel**. C'est la documentation de ton projet.

### La règle du "Si j'applique ce commit, il va..."

```
✅ "Ajouter la validation du formulaire de login"
✅ "Corriger le crash au démarrage sur iOS 17"
✅ "Refactorer le module de paiement pour supporter Stripe"

❌ "fix"
❌ "update"
❌ "changements"
❌ "WIP"
❌ "asdfgh"
```

### Structure recommandée

```
<type>: <description courte> (50 car max)

<corps optionnel — explique le POURQUOI, pas le quoi>
(wrap à 72 caractères par ligne)

<footer optionnel — références, breaking changes>
```

---

## 6.2 Conventional Commits

Convention standardisée, utilisée par de nombreux projets et outils (génération automatique de changelog, versioning).

### Types

| Type | Usage |
|---|---|
| `feat` | Nouvelle fonctionnalité |
| `fix` | Correction de bug |
| `docs` | Documentation uniquement |
| `style` | Formatage (pas de changement de logique) |
| `refactor` | Refactoring (ni feature ni fix) |
| `perf` | Amélioration de performance |
| `test` | Ajout/modification de tests |
| `build` | Build system, dépendances |
| `ci` | Configuration CI/CD |
| `chore` | Tâches diverses (maintenance) |
| `revert` | Annulation d'un commit précédent |

### Exemples

```
feat: ajouter l'authentification OAuth2
feat(auth): supporter le login via Google

fix: corriger le calcul de TVA sur les remises
fix(cart): empêcher les quantités négatives

docs: ajouter le guide de contribution

refactor!: renommer UserService en AccountService
BREAKING CHANGE: l'API /users est remplacée par /accounts

chore: mettre à jour les dépendances npm
```

---

## 6.3 Workflow Quotidien Idéal

```bash
# Début de journée — se mettre à jour
git switch main
git pull origin main

# Commencer une feature
git switch -c feature/ma-feature

# Travailler... commits atomiques et fréquents
git add -p                    # stage sélectif
git commit -m "feat: étape 1"
# ... plus de travail ...
git commit -m "feat: étape 2"

# Se mettre à jour régulièrement
git fetch origin
git rebase origin/main        # rester à jour sans commit de merge

# Avant de push — nettoyer l'historique
git rebase -i HEAD~N          # squash les commits WIP

# Push et créer la PR
git push -u origin feature/ma-feature
# → Ouvrir la PR sur GitHub/GitLab

# Après merge de la PR — nettoyage
git switch main
git pull origin main
git branch -d feature/ma-feature
```

---

## 6.4 Cheat Sheet

### Commandes essentielles

```bash
# CONFIGURATION
git config --global user.name "Nom"
git config --global user.email "email"

# CRÉATION
git init                    # nouveau repo
git clone <url>             # copier un repo

# CYCLE DE BASE
git status                  # état du repo
git add <fichier>           # stager
git add -p                  # stager interactif
git commit -m "message"     # commiter
git push                    # envoyer au remote
git pull                    # récupérer du remote

# BRANCHES
git branch                  # lister
git switch -c <nom>         # créer + basculer
git switch <nom>            # basculer
git merge <branche>         # fusionner
git branch -d <nom>         # supprimer

# HISTORIQUE
git log --oneline --graph   # historique compact
git diff                    # voir les modifications
git blame <fichier>         # qui a modifié quoi

# ANNULER
git restore <fichier>       # annuler les modifications
git restore --staged <f>    # unstage
git commit --amend          # modifier le dernier commit
git revert <hash>           # annuler un commit (safe)
git reset --hard <hash>     # revenir en arrière (destructif)

# AVANCÉ
git stash                   # mettre de côté
git stash pop               # récupérer
git cherry-pick <hash>      # copier un commit
git rebase -i HEAD~N        # réécrire l'historique
git bisect start            # trouver un bug
git reflog                  # filet de sécurité
```

### Raccourcis (si tu as configuré les alias)

```bash
git st    # status
git co    # checkout
git br    # branch
git ci    # commit
git lg    # log graphique
```

---

# 🏁 Conclusion

```
DÉBUTANT :
  ✅ init, add, commit, push, pull
  ✅ .gitignore, messages de commit propres

INTERMÉDIAIRE :
  ✅ Branches, merge, résolution de conflits
  ✅ PRs, code review, stratégie de branching

AVANCÉ :
  ✅ Rebase, stash, cherry-pick, rebase interactif
  ✅ Bisect, reflog, tags, conventional commits

EXPERT :
  ✅ Internals, hooks, CI/CD, GPG signing
  ✅ Monorepo, situations de crise, workflows optimisés

La maîtrise de Git vient avec la pratique quotidienne.
Fais des erreurs, utilise le reflog pour récupérer,
et bientôt Git deviendra une extension naturelle de ta pensée.
```
