# Prompt Claude CLI — Finalisation de l’incrément 1 de la phase 2

Tu travailles sur les deux dépôts suivants :

- API : `C:\Users\heric\dev\afrimyth-api`
- Pilotage : `C:\Users\heric\dev\django-portfolio`

Lis intégralement ce fichier avant d’agir. Commence par vérifier l’état Git des
deux dépôts et les instructions de `afrimyth-api/CLAUDE.md` et
`afrimyth-api/AGENTS.md`.

## État déjà obtenu — ne pas recommencer

Les opérations suivantes sont déjà terminées et vérifiées :

- les entités de démonstration `soundiata-test-orm` et
  `mokele-mbembe-test-orm` ont été supprimées ;
- leurs profils ont été supprimés en cascade ;
- l’entité `shango` a été préservée ;
- `django-portfolio/.gitignore` existe et contient exactement `_audit/` et
  `.claude/` comme répertoires ignorés ;
- aucun fichier n’a été indexé, committé ou poussé.

Ne répète pas ces opérations. Vérifie seulement que cet état est toujours
vrai.

## 1. Corriger le rapport devenu obsolète

Dans `afrimyth-api/docs/reports/phase-2-increment-1.md`, corrige la section 11
qui affirme encore que `_audit/` n’est protégé par aucun `.gitignore`.

Consigne l’état réel :

- `django-portfolio/.gitignore` protège maintenant `_audit/` et `.claude/` ;
- cette protection appartient au dépôt et ne dépend plus de la configuration
  Git globale de cette machine ;
- ces deux répertoires n’ont été ni supprimés ni indexés ;
- ils n’apparaissent plus dans `git status` parce qu’ils sont ignorés.

Mets à jour la copie temporaire
`rapport-pour-codex-phase-2-increment-1.md` afin qu’elle soit strictement
identique au rapport du dépôt. Vérifie cette identité avec SHA-256 avant les
commits.

## 2. Consigner la politique linguistique exacte

La langue de travail interne reste le français.

Crée en français :

`django-portfolio/docs/adr/0003-langue-publique-api.md`

L’ADR doit établir la séparation suivante.

### Travail interne en français

Restent en français :

- les échanges pédagogiques avec l’utilisateur ;
- les prompts destinés aux assistants de code ;
- la roadmap et les documents de pilotage ;
- les ADR et les rapports internes ;
- les commentaires et docstrings internes.

### Produit public et contenu canonique en anglais

Doivent être en anglais :

- les identifiants du code ;
- les noms des champs exposés par l’API ;
- les valeurs techniques de l’API ;
- les descriptions publiques des champs et des fonctionnalités ;
- les messages d’erreur destinés aux clients de l’API ;
- le futur schéma OpenAPI et ses interfaces Swagger ou ReDoc ;
- le README public du dépôt API ;
- le corpus canonique, notamment les noms, descriptions et contenus
  éditoriaux importés ;
- les éléments d’interface destinés au public.

L’anglais est la langue canonique de l’API et du corpus. Une interface
bilingue pourra être réalisée ultérieurement, mais aucune architecture i18n
ne doit être ajoutée maintenant.

Le corpus V2 étant déjà principalement en anglais, l’import pilote puis
l’import complet devront conserver ses contenus canoniques anglais.

Ne traduis pas maintenant l’ensemble des documents, commentaires, docstrings
ou libellés Django existants. Cette opération ne doit pas retarder la phase 2.
Les libellés encore visibles publiquement en français — notamment certains
`verbose_name`, libellés de choix, `help_text` et futurs messages d’erreur —
devront cependant être normalisés en anglais avant de stabiliser le contrat
public et la documentation OpenAPI.

Mets à jour en français
`django-portfolio/docs/adr/0001-decisions-metier-reportees.md` afin d’indiquer
que la question de la langue canonique est résolue le 16 octobre 2026 par
l’ADR 0003.

Ajoute à `django-portfolio/docs/roadmap-projet-1.md` une tâche non cochée
précisant qu’avant la stabilisation de l’API publique, les libellés, aides et
messages exposés aux clients devront être normalisés en anglais.

Ne traduis ni la roadmap existante ni les anciennes ADR.

## 3. Mettre à jour les instructions de contribution

Dans `afrimyth-api/CLAUDE.md`, conserve les règles suivantes :

- code et identifiants en anglais ;
- commentaires, docstrings, prompts, rapports et documentation interne en
  français.

Ajoute explicitement :

- README et documentation publique en anglais ;
- contrat API, noms de champs, valeurs, descriptions et messages publics en
  anglais ;
- contenu canonique du corpus en anglais ;
- aucune internationalisation ajoutée à ce stade.

Ne traduis pas les fichiers sources existants dans cette intervention.

## 4. Traduire uniquement le README public de l’API

Traduis `afrimyth-api/README.md` en anglais.

Préserve fidèlement son périmètre et toutes ses informations techniques :

- prérequis ;
- installation ;
- variables d’environnement ;
- migrations ;
- lancement du serveur ;
- endpoint de santé ;
- contrôles et avertissements de sécurité.

Ne transforme pas le README en documentation complète de l’API. Le README
bilingue de `django-portfolio` reste inchangé.

## 5. Enregistrer IPython comme dépendance de développement

IPython 9.17.1 est déjà installé dans l’environnement virtuel local.

Ajoute uniquement à `afrimyth-api/requirements-dev.txt` :

    ipython~=9.17

Ne l’ajoute pas à `requirements.txt`.

Vérifie son intégration avec :

    python manage.py shell --interface ipython -c "import IPython; print(IPython.__version__)"

## 6. Relancer les contrôles

Dans `afrimyth-api`, exécute :

1. `python manage.py check`
2. `python manage.py makemigrations --check --dry-run`
3. `pytest`
4. `ruff check .`
5. `ruff format --check .`
6. `mypy .`

Vérifie également :

- que la migration de phase 2 est appliquée ;
- que les deux entités de démonstration sont toujours absentes ;
- que `shango` existe toujours ;
- qu’aucun secret, `.env`, `_audit/` ou `.claude/` n’est indexé ;
- que le rapport persistant et sa copie temporaire ont le même SHA-256.

## 7. Préparer trois commits explicites

Avant chaque commit, vérifie que l’auteur et le committer sont :

`héric Libong <hericlibong@gmail.com>`

N’ajoute aucun trailer `Co-authored-by` et ne mentionne aucun assistant comme
auteur.

N’utilise ni `git add -A` ni `git add .`. Indexe uniquement les chemins
énumérés ci-dessous après avoir vérifié leur contenu.

### Commit 1 — `afrimyth-api`

Fichier unique :

- `docs/reports/phase-1-push.md`

Message : `docs: record phase 1 publication`

### Commit 2 — `afrimyth-api`

Fichiers attendus :

- `CLAUDE.md`
- `README.md`
- `requirements-dev.txt`
- `mythology/admin.py`
- `mythology/models.py`
- `mythology/serializers.py`
- `mythology/views.py`
- `mythology/tests/factories.py`
- `mythology/tests/test_api.py`
- `mythology/tests/test_profiles.py`
- `mythology/migrations/0002_creatureprofile_heroprofile.py`
- `docs/reports/phase-2-increment-1.md`

Message : `feat: add hero and creature profiles`

### Commit 3 — `django-portfolio`

Fichiers attendus :

- `.gitignore`
- `docs/adr/0001-decisions-metier-reportees.md`
- `docs/adr/0002-decisions-structurantes-phase-0.md`
- `docs/adr/0003-langue-publique-api.md`
- `docs/roadmap-projet-1.md`
- `docs/prompts/phase-2-increment-1-profiles.md`
- `docs/prompts/phase-2-increment-1-finalisation.md`

Message : `docs: record phase 2 profiles and API language policy`

Si un fichier attendu ne contient aucun changement pertinent, ne l’indexe pas
et explique-le dans le rapport final.

Ne fais ni amend, ni rebase, ni tag, ni push.

## 8. Produire un rapport final temporaire

Crée dans le scratchpad Claude :

`rapport-pour-codex-phase-2-increment-1-commits.md`

Le rapport reste en français et contient :

- la confirmation du nettoyage déjà réalisé et de la préservation de
  `shango` ;
- la politique linguistique exacte ;
- la vérification du `.gitignore` ;
- la version d’IPython ;
- les résultats des six commandes ;
- les SHA, messages et listes exactes de fichiers des trois commits ;
- la vérification de l’auteur et du committer ;
- l’état Git final des deux dépôts ;
- la confirmation qu’aucun push n’a été effectué ;
- toute divergence avec ce prompt et sa justification.

Indique le chemin absolu de ce rapport dans ta réponse finale.

Ne crée aucun nouveau rapport suivi après les commits, car cela laisserait
immédiatement un fichier non committé.
