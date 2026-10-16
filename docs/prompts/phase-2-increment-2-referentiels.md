# Prompt Claude CLI — Phase 2, incrément 2 : référentiels métier

Tu travailles principalement dans :

- API : `C:\Users\heric\dev\afrimyth-api`
- pilotage : `C:\Users\heric\dev\django-portfolio`

Lis intégralement ce prompt avant d’agir. Inspecte d’abord l’état réel des deux
dépôts et les instructions locales applicables. Le précédent incrément est
publié ; ne réécris pas son historique.

## Objectif de l’incrément

Ajouter les quatre référentiels validés de la phase 2 et les relier à
`Entity` :

- `CulturalArea` ;
- `Country` ;
- `Ethnicity` ;
- `Domain`.

Cet incrément doit produire un modèle relationnel testable, administrable et
visible dans l’API de détail. Il ne doit pas encore importer le corpus V2.

Les décisions métier sont déjà prises dans
`django-portfolio/docs/adr/0002-decisions-structurantes-phase-0.md`. Ne les
rouvre pas et n’ajoute pas d’architecture non demandée.

## 1. Modèles de référence

Implémente les modèles explicitement dans l’app `mythology`. N’introduis pas
de package ou d’app supplémentaire et n’ajoute pas de modèle abstrait pour
factoriser seulement deux champs.

### `CulturalArea`

- `name` : `CharField(max_length=255, unique=True)` ;
- `slug` : `SlugField(max_length=255, unique=True)` ;
- `kind` : `CharField` fondé sur une classe `TextChoices` avec les valeurs
  techniques anglaises suivantes :
  - `african_region` — libellé public `African region` ;
  - `transregional` — libellé public `Transregional` ;
  - `diasporic` — libellé public `Diasporic` ;
- `parent` : clé étrangère autoréférentielle facultative, avec
  `related_name="children"` et `on_delete=PROTECT`.

Une aire culturelle est un référentiel, jamais un type d’entité. Prévois une
validation simple interdisant qu’une aire soit son propre parent direct. Ne
construis pas maintenant un moteur générique d’arbre ni une détection complexe
de tous les cycles.

### `Country`

- `name` : `CharField(max_length=255, unique=True)` ;
- `slug` : `SlugField(max_length=255, unique=True)` ;
- `iso_code` : code ISO alpha-2 facultatif, limité à deux caractères et unique
  lorsqu’il est renseigné.

Choisis une représentation Django propre permettant plusieurs pays sans code
ISO sans conflit d’unicité. Valide et normalise un code fourni en majuscules,
sans créer de dépendance externe. `Country` ne doit contenir que de véritables
pays : aucune valeur `Pan-African`, `Congo Basin` ou `Diaspora`.

### `Ethnicity`

- `name` : `CharField(max_length=255, unique=True)` ;
- `slug` : `SlugField(max_length=255, unique=True)` ;
- `primary_cultural_area` : `ForeignKey` facultative vers `CulturalArea`,
  `on_delete=PROTECT`, avec un `related_name` explicite et lisible.

### `Domain`

- `name` : `CharField(max_length=255, unique=True)` ;
- `slug` : `SlugField(max_length=255, unique=True)`.

Les `__str__`, `Meta.ordering`, noms de contraintes et `related_name` doivent
être explicites. Les identifiants et les chaînes destinées au produit public
sont en anglais. Les commentaires et docstrings internes peuvent rester en
français, conformément à l’ADR 0003.

## 2. Relations ajoutées à `Entity`

Ajoute exactement les champs suivants :

- `primary_country` : `ForeignKey` facultative vers `Country`,
  `on_delete=PROTECT` ;
- `secondary_countries` : `ManyToManyField` facultatif vers `Country` ;
- `primary_ethnicity` : `ForeignKey` facultative vers `Ethnicity`,
  `on_delete=PROTECT` ;
- `secondary_ethnicities` : `ManyToManyField` facultatif vers `Ethnicity` ;
- `primary_cultural_area` : `ForeignKey` facultative vers `CulturalArea`,
  `on_delete=PROTECT` ;
- `secondary_cultural_areas` : `ManyToManyField` facultatif vers
  `CulturalArea` ;
- `domains` : `ManyToManyField` facultatif vers `Domain`.

Utilise des `related_name` non ambigus pour distinguer les rattachements
principaux des rattachements secondaires.

Les clés étrangères Django sont déjà indexées : n’ajoute pas d’index manuel
redondant sur les trois champs `primary_*`. N’ajoute pas de table de liaison
explicite si elle ne porte aucune donnée supplémentaire.

La règle « une même valeur ne doit pas être simultanément principale et
secondaire » est validée, mais une validation dans `Model.clean()` ne protège
pas correctement une nouvelle sélection M2M dans l’administration. Ne prétends
donc pas garantir cette règle avec une validation incomplète et n’ajoute pas
de signaux globaux pour cet incrément. Documente clairement que son application
complète appartiendra aux services d’import et d’écriture, avant le CRUD.

Ne crée pas encore `variant_of`, `EntityRelation`, `Source`, le workflow de
modération ou les champs de suppression logique : ce sont les incréments
suivants.

## 3. Migration et base existante

Crée une seule migration Django lisible pour cet incrément. Les nouveaux
champs de `Entity` doivent être facultatifs afin que l’entité `shango` déjà
présente reste valide sans migration de données.

Applique la migration à la base de développement et vérifie :

- que `shango` existe toujours ;
- qu’aucune donnée existante n’a été modifiée ou supprimée ;
- que `makemigrations --check --dry-run` ne détecte ensuite aucun changement.

Ne crée aucune donnée de démonstration persistante dans la base de
développement. Les données nécessaires aux tests doivent rester dans la base
de test de pytest.

## 4. Administration Django

Enregistre les quatre référentiels dans l’administration. Fournis une
administration sobre et utile :

- recherche par nom et slug ;
- `prepopulated_fields` pour le slug à partir du nom ;
- filtres réellement utiles, par exemple `kind` pour `CulturalArea` ;
- affichage des relations de référence pertinentes.

Mets également à jour l’administration de `Entity` pour permettre la saisie
des nouvelles relations sans transformer l’interface en formulaire complexe.

## 5. API de lecture

L’API reste publique et strictement en lecture seule.

La liste des entités doit rester compacte et conserver exactement son contrat
actuel : n’y ajoute aucun référentiel.

Le détail d’une entité doit désormais exposer les clés anglaises suivantes :

- `primary_country` : objet synthétique ou `null` ;
- `secondary_countries` : liste ;
- `primary_ethnicity` : objet synthétique ou `null` ;
- `secondary_ethnicities` : liste ;
- `primary_cultural_area` : objet synthétique ou `null` ;
- `secondary_cultural_areas` : liste ;
- `domains` : liste.

Crée de petits serializers de référence en lecture seule. Les objets imbriqués
doivent fournir une identité utile et stable sans sur-imbriquer le graphe :

- socle commun : `id`, `slug`, `name` ;
- `Country` ajoute `iso_code` ;
- `CulturalArea` ajoute `kind` ;
- `Ethnicity` et `Domain` restent sur le socle commun dans la réponse d’une
  entité.

N’imbrique pas `parent`, `children` ou `primary_cultural_area` d’une ethnie
dans ces représentations : cela créerait des cascades de sérialisation et des
requêtes inutiles. Aucun endpoint autonome de référentiel n’est demandé dans
cet incrément ; il pourra être ajouté lors de la stabilisation de l’API
publique si un besoin client est confirmé.

Dans le `retrieve`, charge les trois relations principales avec
`select_related` et les quatre collections avec `prefetch_related`, en plus
des trois profils déjà joints. Ne modifie pas le queryset de la liste puisque
la liste n’expose pas ces relations.

Ajoute un test de nombre de requêtes démontrant que le détail d’une entité
complètement reliée ne provoque pas de N+1. Le test doit vérifier un nombre
déterministe de requêtes et expliquer ce qu’il protège ; il ne doit pas faire
de promesse de performance générale.

## 6. Tests attendus

Complète les factories et ajoute des tests couvrant au minimum :

- création et représentation des quatre référentiels ;
- unicité des noms et des slugs ;
- valeurs autorisées de `CulturalArea.kind` via `full_clean()` ;
- refus d’un parent égal à soi-même ;
- normalisation et validation de `Country.iso_code` ;
- protection à la suppression d’un référentiel utilisé par une relation
  principale ;
- coexistence des relations principales et secondaires ;
- plusieurs ethnies, pays, aires et domaines sur une même entité ;
- absence de ces champs dans la réponse de liste ;
- présence des objets et listes attendus dans la réponse de détail ;
- valeurs `null` et listes vides lorsqu’aucun rattachement n’existe ;
- absence de N+1 sur le détail grâce à `select_related` et
  `prefetch_related` ;
- non-régression des profils de divinité, héros et créature.

Ne teste pas encore l’import du corpus, les variantes, les relations entre
entités, les sources ou le CRUD.

## 7. Documentation et rapport

Ne coche pas encore la tâche de la roadmap : sa validation appartient à la
revue qui suivra. Ne crée pas de nouvelle ADR, car les choix métier sont déjà
consignés dans l’ADR 0002.

Crée un rapport interne en français :

`C:\Users\heric\dev\afrimyth-api\docs\reports\phase-2-increment-2-referentiels.md`

Copie-le à l’identique dans le scratchpad temporaire de la session sous le nom :

`rapport-pour-codex-phase-2-increment-2-referentiels.md`

Le rapport doit indiquer :

- l’état Git initial et final des deux dépôts ;
- les fichiers créés ou modifiés ;
- le schéma effectivement retenu ;
- la forme exacte du JSON de détail ;
- le nombre de requêtes attendu et mesuré ;
- les tests ajoutés et leur portée ;
- les commandes de contrôle et leurs sorties ;
- toute divergence avec ce prompt, avec justification ;
- les limites volontairement reportées.

## 8. Contrôles obligatoires

Dans `afrimyth-api`, avec l’environnement virtuel et le `.env` existants,
exécute :

- `python manage.py check` ;
- `python manage.py makemigrations --check --dry-run` ;
- `python manage.py showmigrations mythology` ;
- `pytest` ;
- `ruff check .` ;
- `ruff format --check .` ;
- `mypy .`.

Vérifie aussi qu’aucun secret, `.env`, cache, `_audit/` ou `.claude/` n’est
indexé.

## 9. Limites Git

Ne fais aucun commit et aucun push. N’utilise ni `git add -A` ni `git add .`.
Ne modifie pas l’historique existant. Laisse toutes les modifications visibles
pour la revue pédagogique de Codex et de l’auteur du projet.

Dans ta réponse finale, donne le chemin absolu du rapport temporaire et un
résumé factuel. N’affirme pas qu’une contrainte est garantie si elle ne l’est
que par `full_clean()` ou par l’administration.
