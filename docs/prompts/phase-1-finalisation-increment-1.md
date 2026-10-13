# Prompt proposé à Claude CLI — Finaliser l'incrément 1 de la phase 1

Tu travailles sur deux dépôts Git distincts :

- code API : `C:\Users\heric\dev\afrimyth-api` ;
- pilotage et documentation : `C:\Users\heric\dev\django-portfolio`.

Commence par lire :

- `C:\Users\heric\dev\django-portfolio\docs\roadmap-projet-1.md` ;
- `C:\Users\heric\dev\django-portfolio\docs\adr\0002-decisions-structurantes-phase-0.md` ;
- les fichiers `CLAUDE.md` et `AGENTS.md` de `afrimyth-api`.

L'objectif est uniquement de documenter, vérifier et committer le premier
incrément déjà réalisé. N'ajoute aucun modèle métier et ne commence pas
l'incrément suivant.

## 1. Rapport dans `afrimyth-api`

Crée `docs/reports/phase-1-increment-1.md` avec :

- l'objectif et le périmètre de l'incrément ;
- les choix techniques principaux ;
- les fichiers créés ou modifiés ;
- les commandes de vérification et leurs résultats ;
- le blocage PostgreSQL initial et sa résolution manuelle, sans mot de passe ;
- la confirmation que PostgreSQL 16 est accessible avec la base et le rôle
  `afrimyth`, que les migrations ont été appliquées sur PostgreSQL, que
  `showmigrations` les marque toutes `[X]` et que `manage.py check` réussit ;
- l'état Git avant commit et les tâches restant ouvertes.

Ne mets aucun secret, aucune valeur de `.env` et aucun mot de passe dans ce
rapport.

Ajoute aussi dans `CLAUDE.md` et `AGENTS.md` une convention courte : chaque
intervention doit créer ou actualiser un rapport sous `docs/reports/`, avec
objectif, fichiers touchés, décisions, commandes et résultats, blocages et état
Git final, sans secret.

## 2. Vérifications dans `afrimyth-api`

Vérifie que `.env`, `.venv` et les caches sont ignorés, puis exécute :

- `python manage.py check` ;
- `python manage.py makemigrations --check --dry-run` ;
- `pytest` ;
- `ruff check .` ;
- `ruff format --check .` ;
- `mypy .`.

Corrige uniquement un écart appartenant au périmètre déjà réalisé. Ne crée ni
`Entity`, ni profils, ni nouvelle fonctionnalité.

## 3. Mise à jour de la roadmap dans `django-portfolio`

Dans `docs/roadmap-projet-1.md`, coche seulement les tâches suivantes de la
phase 1 :

- créer le dépôt autonome ;
- initialiser Django, DRF et la configuration PostgreSQL ;
- configurer pytest et pytest-django ;
- configurer Ruff et mypy ;
- créer `CLAUDE.md` et `AGENTS.md` ;
- créer les apps `accounts` et `mythology` ;
- déclarer le modèle utilisateur personnalisé dès la première migration ;
- exposer l'endpoint de santé ;
- ajouter les premiers tests de modèle et d'API.

Laisse ouvertes les tâches concernant `Entity`, `DivinityProfile`, leur
administration et la lecture liste/détail. La phase 1 reste donc **en cours**.

Ajoute sous l'estimation de la phase 1 une note concise intitulée
`Avancement — incrément 1 validé le 13 octobre 2026`, mentionnant que le socle
Python 3.12 / Django 5.2 / DRF, PostgreSQL 16, le modèle utilisateur, l'endpoint
de santé, les tests et les outils qualité sont opérationnels.

Ne modifie ni `_audit/` ni `.claude/`. Ajoute au suivi Git le prompt de
l'incrément déjà présent sous `docs/prompts/`.

## 4. Deux commits locaux, sans push

Avant chaque commit, affiche le diff préparé et vérifie qu'aucun secret ni
fichier `.env` n'est indexé.

Utilise exclusivement cette identité Git pour l'auteur et le committer :

- nom : `héric Libong` ;
- e-mail : `hericlibong@gmail.com`.

N'ajoute aucun trailer `Co-authored-by`, aucune mention de Claude et aucune
signature d'un autre auteur.

Crée ensuite deux commits séparés :

1. dans `afrimyth-api` : `chore: initialize Django REST API` ;
2. dans `django-portfolio` : `docs: record phase 1 first increment`.

Ne pousse aucun des deux dépôts.

Après les commits, affiche pour chacun :

- `git status --short --branch` ;
- `git show -s --format=fuller HEAD`.

Termine par un résumé concis des fichiers, vérifications, deux identifiants de
commit et éventuels blocages.
