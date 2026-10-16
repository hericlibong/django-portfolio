# Prompt proposé à Claude CLI — Phase 2, incrément 1 : `HeroProfile` et `CreatureProfile`

Tu travailles sur les deux dépôts suivants :

- API : C:\Users\heric\dev\afrimyth-api
- Pilotage : C:\Users\heric\dev\django-portfolio

Commence par lire intégralement :

- C:\Users\heric\dev\afrimyth-api\CLAUDE.md
- C:\Users\heric\dev\afrimyth-api\AGENTS.md
- C:\Users\heric\dev\django-portfolio\docs\roadmap-projet-1.md
- C:\Users\heric\dev\django-portfolio\docs\modele-donnees.md
- C:\Users\heric\dev\django-portfolio\docs\adr\0002-decisions-structurantes-phase-0.md

Objectif : réaliser le premier incrément de la phase 2, consacré à
`HeroProfile` et `CreatureProfile`.

Avant toute modification, vérifie l'état Git des deux dépôts. Ne supprime,
n'indexe et ne modifie aucun fichier hors périmètre. Ne touche pas à `_audit/`
ni à `.claude/`.

## 1. Mettre à jour les instructions du dépôt API

Le fichier `afrimyth-api/CLAUDE.md` décrit encore le début de la phase 1 et
affirme que les modèles métier ne doivent pas exister. Actualise uniquement
les passages devenus obsolètes :

- la phase 1 est terminée ;
- `Entity` et `DivinityProfile` existent ;
- la phase 2 complète progressivement le modèle métier ;
- les données de rendu, les prompts, les styles visuels et la génération
  d'images restent hors de cet incrément.

Conserve les règles de simplicité, de tests, de migrations, de rapport et
d'absence de secrets.

## 2. Ajouter les deux profils métier

Dans l'application `mythology`, ajouter :

### `HeroProfile`

Relation `OneToOneField` vers `Entity`, avec
`related_name="hero_profile"` et suppression en cascade.

Champs :

- `titles`
- `achievements`
- `quests`
- `weapons_or_artifacts`

Ces quatre champs sont des listes de chaînes, en réutilisant la fonction
existante `string_array()`.

Ajouter également :

- `legacy` : `TextField`, vide autorisé ;
- `historical_context` : `TextField`, vide autorisé.

### `CreatureProfile`

Relation `OneToOneField` vers `Entity`, avec
`related_name="creature_profile"` et suppression en cascade.

Champs listes, avec `string_array()` :

- `habitat`
- `powers`
- `strengths`
- `weaknesses`
- `encounter_omens`
- `encounter_rules`
- `known_sightings`

Autres champs :

- `diet` : `CharField(max_length=255)`, vide autorisé ;
- `size` : `CharField(max_length=255)`, vide autorisé ;
- `appearance_notes` : `TextField`, vide autorisé ;
- `threat_level` : choix textuel nullable et facultatif parmi
  `low`, `medium`, `high`, `mythic`.

Définis `ThreatLevel` avec `models.TextChoices`, comme `EntityType` et
`Gender`.

Chaque profil doit :

- avoir un `verbose_name` et un `verbose_name_plural` en français ;
- avoir un `__str__` lisible ;
- refuser dans `clean()` une entité dont `entity_type` ne correspond pas ;
- rester explicite et simple, sans modèle abstrait ou mixin prématuré.

Ne construis pas encore le service garantissant qu'une entité possède
exactement un profil. Il sera traité dans un incrément ultérieur de la
phase 2.

## 3. Administration et lecture API

- Enregistrer les deux profils dans l'administration Django.
- Fournir une liste minimale, une recherche sur `entity__name` et
  `autocomplete_fields = ["entity"]`, comme pour `DivinityProfile`.
- Ajouter leurs serializers de lecture.
- Dans le détail d'une entité, exposer :
  - `divinity_profile`
  - `hero_profile`
  - `creature_profile`
- Le profil correspondant doit être imbriqué ; les deux autres doivent être
  absents ou `null` de manière cohérente et testée.
- Adapter la vue de détail avec `select_related` pour charger les trois
  relations `OneToOne` sans requêtes supplémentaires.
- Ne transforme pas l'API en CRUD : elle reste en lecture seule et `AllowAny`
  pour le moment.

## 4. Migrations et tests

Générer la migration Django correspondante.

Ajouter des tests couvrant au minimum :

- création valide des deux profils ;
- refus d'un profil rattaché au mauvais type d'entité via `full_clean()` ;
- unicité `OneToOne` ;
- affichage du profil du héros dans l'endpoint de détail ;
- affichage du profil de la créature ;
- conservation du comportement du profil de divinité ;
- absence de régression sur l'endpoint de liste ;
- utilisation effective de `select_related` sur le détail, sans test de
  performance excessivement fragile.

Ne traite pas encore `prefetch_related` : les relations multiples concernées
seront ajoutées dans l'incrément des référentiels et relations.

## 5. Documentation de la décision sur les prompts

Dans le dépôt `django-portfolio`, complète l'ADR 0002 et la roadmap pour
rendre explicite la décision suivante :

- les faits mythologiques restent dans `Entity` et ses profils ;
- aucun champ `prompt` isolé n'est ajouté à `Entity` ;
- `prompt_canon`, les variantes, la matrice de styles et les métadonnées
  d'images seront intégrés ultérieurement dans le module de rendu déjà prévu ;
- le corpus V2 reste la source conservée de ces données tant qu'elles ne sont
  pas importées ;
- l'API ne deviendra leur source de vérité qu'après cet import ;
- la phase 2 doit néanmoins préparer les entrées nécessaires au futur moteur,
  notamment l'ethnie, les signes physiques, les symboles et les objets de
  pouvoir.

Ne déplace pas la future phase de rendu dans la phase 2.

Lorsque l'implémentation et les tests sont terminés, coche seulement la tâche
« Ajouter HeroProfile et CreatureProfile » de la phase 2. Ne coche aucune
autre tâche.

## 6. Vérifications

Exécute dans `afrimyth-api` :

- `python manage.py check`
- `python manage.py makemigrations --check --dry-run`
- `pytest`
- `ruff check .`
- `ruff format --check .`
- `mypy .`

Si Ruff demande uniquement un formatage, applique `ruff format`, puis relance
les contrôles.

## 7. Rapport et Git

Crée :

`C:\Users\heric\dev\afrimyth-api\docs\reports\phase-2-increment-1.md`

Le rapport doit contenir :

- objectif et périmètre ;
- modèles et champs ajoutés ;
- décisions prises ;
- fichiers modifiés ;
- migration créée ;
- tests ajoutés ;
- résultats exacts des six commandes ;
- éventuels blocages et leur résolution ;
- état Git final ;
- éléments volontairement reportés.

Fournis également une copie conforme dans le scratchpad temporaire de la
session Claude, nommée :

`rapport-pour-codex-phase-2-increment-1.md`

et indique son chemin absolu dans ta réponse finale.

Ne modifie pas le rapport non suivi
`docs/reports/phase-1-push.md`, sauf si une erreur factuelle y est réellement
constatée.

Ne commite rien et ne pousse rien. Nous effectuerons d'abord la revue
pédagogique du code et du rapport.
