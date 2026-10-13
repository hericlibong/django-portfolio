# Prompt proposé à Claude CLI — Phase 1, incrément 1

Tu travailles dans `C:\Users\heric\dev\afrimyth-api`.

Un premier squelette non committé existe déjà à la suite d'une exécution
interrompue. Commence par examiner son état et conserve ce qui est conforme ;
corrige ou complète le reste. Ne repars pas aveuglément de zéro.

Avant d'écrire, lis ces documents de conception :

- `C:\Users\heric\dev\django-portfolio\docs\roadmap-projet-1.md`
- `C:\Users\heric\dev\django-portfolio\docs\adr\0002-decisions-structurantes-phase-0.md`
- `C:\Users\heric\dev\django-portfolio\docs\modele-donnees.md`

Implémente uniquement le premier socle technique de la phase 1 :

- utilise Python 3.12 et Django 5.2 LTS (`Django~=5.2.18`) ;
- initialise ou finalise le projet Django avec Django REST Framework et une
  configuration PostgreSQL par variables d'environnement ;
- crée les apps `accounts` et `mythology` ;
- déclare dès la première migration un modèle utilisateur personnalisé minimal,
  fondé sur `AbstractUser`, avec `username` comme identifiant et une clé primaire
  `BigAutoField` ;
- ajoute un endpoint public de santé avec un test ;
- configure pytest/pytest-django, Ruff et mypy dès maintenant ;
- ajoute `.gitignore`, `.env.example`, `CLAUDE.md` et `AGENTS.md` ;
- garde une structure simple et conventionnelle, sans couche d'abstraction anticipée.

Contraintes :

- n'implémente pas encore `Entity` ni les profils métier ;
- n'ajoute ni Docker, ni CI, ni JWT, ni génération d'images ;
- ne copie rien depuis `_audit` ;
- ne committe et ne pousse rien ;
- la migration initiale de `accounts.User` doit être générée par Django, pas
  maintenue comme une migration écrite à la main ;
- utilise l'environnement `.venv` existant s'il est sain, sinon recrée-le avec
  `py -3.12` ;
- exécute et fais réussir :
  - `python manage.py check` ;
  - `python manage.py makemigrations --check --dry-run` ;
  - `pytest` ;
  - `ruff check .` ;
  - `ruff format --check .` ;
  - `mypy .`.

À la fin, ne committe rien. Résume les choix réalisés, les fichiers créés ou
modifiés, le résultat exact de chaque vérification et tout blocage réel. Nous
ferons ensuite la revue avant d'autoriser un commit.
