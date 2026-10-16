# Prompt Claude CLI — Clôture de la phase 2, incrément 2

Travaille dans les deux dépôts :

- `C:\Users\heric\dev\afrimyth-api`
- `C:\Users\heric\dev\django-portfolio`

Lis leurs instructions locales et vérifie l'état Git avant d'agir.
L'implémentation des référentiels et la revue pédagogique sont validées.
Codex a appliqué les petites corrections et actualisé la documentation.

## Contrôles ciblés

Lis le rapport `afrimyth-api/docs/reports/phase-2-increment-2-referentiels.md`,
en particulier les sections 13 et 14. Les 73 tests, Ruff, le formatage, mypy
et les contrôles Django ont été validés après correction. Ne relance pas
l'implémentation ni toute la suite en l'absence de changement de code ou
d'anomalie nouvelle. La clôture a ensuite modifié seulement la documentation.

Vérifie que :

- la branche est `main` dans les deux dépôts, sans changement inattendu ;
- les remotes correspondent à `hericlibong/afrimyth-api` et
  `hericlibong/django-portfolio` ;
- les corrections de `Country` et le test de sérialisation sans requête
  figurent dans le diff ;
- la roadmap coche seulement la tâche des référentiels pour cet incrément ;
- la précision géographique figure dans l'ADR 0002 et le rapport ;
- aucun secret réel, `.env`, cache, `_audit/` ou `.claude/` n'est indexé.

Ne crée pas les régions en base : l'import pilote les préparera. Ne commence
pas les variantes, les relations entre entités ou un autre incrément.

## Deux commits, avec indexation explicite

Inspecte le contenu de l'index et les diffs avant chaque commit. Utilise
uniquement les chemins ci-dessous, jamais `git add .` ni `git add -A`.
Préserve tout changement inattendu et signale-le au lieu de l'inclure.

Dans `afrimyth-api`, crée le commit :

`feat: add cultural and geographic reference models`

Fichiers attendus :

- `mythology/models.py`
- `mythology/admin.py`
- `mythology/serializers.py`
- `mythology/views.py`
- `mythology/tests/factories.py`
- `mythology/tests/test_api.py`
- `mythology/tests/test_reference.py`
- `mythology/migrations/0003_domain_country_entity_primary_country_and_more.py`
- `docs/reports/phase-2-increment-2-referentiels.md`

Dans `django-portfolio`, crée le commit :

`docs: close phase 2 reference models increment`

Fichiers attendus :

- `AGENTS.md`
- `docs/adr/0002-decisions-structurantes-phase-0.md`
- `docs/roadmap-projet-1.md`
- `docs/prompts/phase-2-increment-2-referentiels.md`
- `docs/prompts/phase-2-increment-2-cloture.md`

Les deux commits doivent porter l'identité Git déjà validée :
`héric Libong <hericlibong@gmail.com>`, auteur et committer. Vérifie-la avant
et après ; n'ajoute ni trailer `Co-authored-by`, ni attribution d'assistant.
Ne modifie pas la configuration Git globale.

Aucun push, amend, rebase ou réécriture d'historique.

## Rapport final temporaire

Crée uniquement dans ton scratchpad un rapport en français nommé :

`rapport-pour-codex-phase-2-increment-2-cloture.md`

Indique les deux SHA, leurs messages, les fichiers réellement inclus,
l'identité auteur/committer, les contrôles effectués et l'état Git final des
deux dépôts. Confirme l'absence de push. Signale toute divergence.

Ne crée pas de nouveau rapport dans les dépôts après les commits. Si tu
actualises le rapport d'incrément avant commit, synchronise sa copie temporaire.
Donne le chemin absolu du rapport de clôture dans ta réponse.
