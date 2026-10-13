# Modèle de données — mythologies africaines

Audit de consolidation des trois dépôts existants, en vue de la future API Django REST.

**Statut : audit consolidé.** Les décisions structurantes de la phase 0 sont actées dans
[`ADR 0002`](adr/0002-decisions-structurantes-phase-0.md). En cas d'écart avec une proposition
historique de ce document, l'ADR fait foi.

## 0. Périmètre et méthode

Trois dépôts ont été clonés en `--depth 1` dans `_audit/` et lus intégralement
(modèles, migrations, schémas Pydantic, types TypeScript, fichiers de données, contrat V2).

| Dépôt | Dernier commit lu | Pile | Rôle dans la chronologie |
|---|---|---|---|
| `AfricanPantheonAPI` | 2024-08-02 | Django 5.0.6 + DRF 3.15 + drf-yasg | Première modélisation (V0) |
| `AfrMyth_API` | 2025-02-26 | Django + DRF + PostgreSQL (Pipfile) | Seconde modélisation (V1) |
| `african_mythology_V2` | 2026-04-02 | React + Vite + FastAPI + Pydantic | Modélisation courante (V2) |

Deux observations préalables qui conditionnent la lecture du reste.

**La chronologie ne suit pas le volume de commits.** `AfricanPantheonAPI` a 32 commits mais
porte le modèle le plus naïf : trois modèles plats et indépendants. `AfrMyth_API` n'en a que 9
et porte une structure nettement plus mûre : un tronc commun `Entity` plus trois tables de
détails en relation un-à-un. V2 reprend ce principe tronc + extensions, en JSON. La
qualification « embryons antérieurs » est donc exacte sur le plan chronologique, mais
`AfrMyth_API` n'est pas un brouillon sur le plan conceptuel : c'est lui qui a posé le pattern
que V2 applique aujourd'hui.

**Aucun des trois dépôts ne porte de couche plateforme réelle.** Les deux projets Django ont un
modèle utilisateur (deux stratégies différentes) et un champ `created_by`, mais aucune
permission, aucun workflow de contribution, aucune modération. V2 n'a aucun utilisateur du
tout : la donnée est un fichier JSON que l'API réécrit sans trace d'auteur. Tout ce qui relève
de la plateforme est donc à concevoir, pas à reprendre.

---

## 1. Inventaire par dépôt

### 1.1 `AfricanPantheonAPI` (V0) — trois modèles plats

App `entities`, fichier `src/entities/models.py`. Les trois entités métier n'ont aucun champ
en commun au niveau du schéma : chaque modèle redéclare `name`, `country`, `origin`,
`category`, les dates et `created_by`.

**`Category`**

| Champ | Type | Notes |
|---|---|---|
| `name` | `CharField(100)` | Non unique au niveau base ; unicité tentée via un mixin applicatif |
| `description` | `TextField(max_length=100)` | `max_length` sur un `TextField` : contrainte de formulaire uniquement, non appliquée en base |
| `date_created` / `date_updated` | `DateTimeField` | `auto_now_add` / `auto_now` |

**`ImageWithCaption`**

| Champ | Type | Notes |
|---|---|---|
| `image` | `ImageField` | `upload_to` de signature incorrecte (trois paramètres au lieu de deux) |
| `image_caption` | `CharField(255)` | nullable |

Référencé en `ManyToManyField` par `Divinity` seule. `Hero` et `MythicalCreature` ont conservé
un `ImageField` direct plus un `image_caption`. L'unification des images est donc inachevée.

**`Divinity`** — 25 champs, le modèle le plus riche des trois dépôts.

| Groupe | Champs | Types |
|---|---|---|
| Identification | `name` | `CharField(100)` |
| | `category` | `FK(Category, CASCADE)` |
| Apparence | `gender` | `CharField(1)`, choix `M` / `F` / `A` |
| | `distincts_physicals_signs` | `ArrayField(CharField(100))` |
| Identité | `cultural_role` | `CharField(100)`, requis |
| | `pantheon`, `alignment` | `CharField(100)`, nullable |
| Attributs | `domain`, `main_symbol` | `ArrayField(CharField(100))` |
| | `symbolic_animals`, `power_objects` | `ArrayField(CharField(100))` |
| Récit | `story_description` | `TextField` |
| | `characteristics` | `ArrayField(CharField(100))` |
| | `manifestations` | `CharField(255)` |
| Origine | `country`, `origin` | `CharField(50)` / `CharField(100)` |
| | `ethnicity` | `ArrayField(CharField(100))` |
| Généalogie | `parents`, `descendants`, `conjoint` | `ArrayField(CharField(100))` |
| Images | `images` | `M2M(ImageWithCaption)` |
| Traçabilité | `created_by` | `FK(AUTH_USER_MODEL, CASCADE)`, nullable |
| | `date_created`, `date_updated` | `DateTimeField` |

**`Hero`** — 16 champs. Même intention, typage beaucoup plus pauvre : `parents`, `descendants`,
`titles`, `enemies`, `allies`, `achievements` sont des `CharField` ou `TextField` libres, là où
`Divinity` utilise des `ArrayField`. Pas de `pantheon`, pas de `ethnicity`, pas d'`alignment`.

**`MythicalCreature`** — 17 champs : `description`, `habitat` (`CharField(255)`), `powers`,
`diet`, `size`, `appareance` (faute de frappe présente en base depuis la migration 0008),
`weaknesses`, `strengths`. Pas de généalogie, pas de `pantheon`, pas de `ethnicity`.

**Historique des migrations** — intéressant pour ce qu'il révèle d'abandonné. Les champs
supprimés de `Divinity` au fil des 13 migrations : `status`, `prompt`, `associated_myths`,
`festivals`, `region`, `image`, `image_caption`. Trois d'entre eux sont des pertes de sens :

- `associated_myths` (`TextField`) — les récits mythiques rattachés à l'entité, absorbés dans
  `story_description`. Aucune des trois versions n'a de notion de *mythe* comme objet distinct.
- `festivals` (`TextField`) — réapparaît dans le contrat V2 sous `divinity.cult.festivals`,
  mais n'a jamais été réimplémenté.
- `prompt` — ancêtre direct de tout le bloc `rendering` de V2.

**App `user_profiles`** — `UserProfile(AbstractBaseUser, PermissionsMixin)` avec `email` comme
`USERNAME_FIELD`, `name`, `is_active`, `is_staff`, et un `UserProfileManager` maison.

**Couche API** — présente et la plus avancée des trois : viewsets DRF dédoublés en
`Admin*ViewSet` (`ModelViewSet`) et `*ViewSet` (`ReadOnlyModelViewSet`), sérialiseurs
liste/détail distincts, `DjangoFilterBackend` avec `filterset_class`. Trois défauts à ne pas
reproduire : aucune classe de permission n'est déclarée (le découpage admin / lecture seule ne
repose sur rien), `UserCreatedMixin` est placé en dernier dans l'ordre d'héritage donc son
`perform_create` n'est jamais appelé, et `MythicalCreatureViewSet` écrit `filterSet` au lieu de
`filterset_class` — le filtre est donc inactif. `django-filter` n'apparaît d'ailleurs pas dans
`requirements.txt`.

**Incohérence de pile à signaler** — les `ArrayField` de `django.contrib.postgres` exigent
PostgreSQL, mais `requirements.txt` ne contient aucun pilote Postgres et la base livrée est un
SQLite. Ce modèle n'a jamais tourné en l'état : la base `db.sqlite3` s'arrête à la migration
`0011`, soit avant l'introduction des `ArrayField` de la migration `0012`.

### 1.2 `AfrMyth_API` (V1) — tronc commun + extensions par type

App `entities`, fichier `src/entities/models.py`. Le pattern qui compte.

**`Entity`** — le tronc commun.

| Champ | Type | Notes |
|---|---|---|
| `name` | `CharField(255)` | |
| `entity_type` | `CharField(20)` | Choix `Divinity` / `Hero` / `Mythical Creature` |
| `country_of_origin` | `CharField(100)` | nullable |
| `ethnicity` | `CharField(100)` | nullable — **singulier**, là où V0 avait un `ArrayField` |
| `gender` | `CharField(15)` | Choix `Male` / `Female` / `Androgynous` / `Spirit` / `Element` |
| `images` | `M2M(ImageWithCaption)` | unifié pour les trois types, contrairement à V0 |
| `created_by` | `FK(AUTH_USER_MODEL, SET_NULL)` | nullable — `SET_NULL`, correction du `CASCADE` de V0 |
| `date_created`, `date_modified` | `DateTimeField` | |
| `Meta.unique_together` | `(name, entity_type)` | Première contrainte d'unicité réelle en base |

**`DivinityDetails`** — `OneToOneField(Entity, CASCADE, limit_choices_to={'entity_type': 'Divinity'})`,
plus `cultural_role` (requis), `pantheon`, `alignment`, `manifestations`, et six `ArrayField` :
`domains`, `main_symbols`, `characteristics`, `symbolic_animals`, `power_objects`, `consorts`.
Tous en `default=list`, ce qui est plus propre que le `null=True` de V0.

**`HeroDetails`** — `titles` (`CharField(255)`), `achievements` (`TextField`),
`enemies` et `allies` (`ArrayField`).

**`MythicalCreatureDetails`** — `habitat` (`CharField(255)`, requis), `diet`, `size`,
`weaknesses`, `strengths`.

**`ImageWithCaption`** — `image` (`ImageField`, requis), `caption` (`CharField(255)`).
Noter le renommage : `image_caption` en V0 devient `caption`.

**App `accounts`** — `User(AbstractUser)` plus un champ `bio` (`TextField`). Stratégie
**opposée** à celle de V0 : `AbstractUser` conserve `username` comme identifiant, là où V0
basculait sur l'email via `AbstractBaseUser`.

**Régressions nettes par rapport à V0** — à ne pas perdre de vue, ce sont des pertes, pas des
simplifications :

- **Aucun champ de récit.** Ni `story_description`, ni `description`, ni `legend`, nulle part.
  Les fichiers de données de ce dépôt n'en contiennent aucun non plus. Le récit, qui est la
  substance du domaine, a simplement disparu du schéma.
- **Aucune généalogie structurée.** `parents` et `descendants` ne sont nulle part.
  `DivinityDetails.consorts` subsiste seul. `HeroDetails` n'a aucune relation familiale.
- **Pas de modèle `Category`.** Remplacé par l'énumération `entity_type`, qui ne joue pas le
  même rôle : en V2 `category` est un libellé éditorial fin (« Orisha of Lightning »), pas un
  type.
- **Pas de `physical_signs`.** Le `distincts_physicals_signs` de V0 est perdu.

**Couche API** — inexistante. `entities/views.py` et `accounts/views.py` sont les stubs générés
par `startapp`. Seul `admin.py` est écrit. Le dépôt est une modélisation, pas une API.

### 1.3 `african_mythology_V2` — schéma JSON imbriqué, pas de base de données

Pas de couche de persistance : un fichier `src/data/mythology_data.json` validé par Pydantic
(`engine/domain.py`), typé en miroir côté React (`src/types.ts`), et spécifié dans
`docs/CONTRACT_V2.md`.

**`MythologicalEntity`** (Pydantic, `model_config = ConfigDict(extra='allow')` — tout champ
non déclaré est accepté silencieusement).

| Bloc | Champs | Types |
|---|---|---|
| racine | `entity_type` | `str` côté Pydantic ; `'Divinity' \| 'Hero' \| 'Creature'` côté TS |
| | `name`, `category` | `str` |
| `origin` | `country`, `ethnicity`, `pantheon` | `str` requis |
| | `cultural_region` | `str`, défaut `""` |
| `identity` | `gender`, `cultural_role`, `alignment` | `str` requis |
| `attributes` | `domains`, `symbols`, `power_objects`, `symbolic_animals` | `List[str]` requis |
| `appearance` | `physical_signs` | `List[str]` |
| | `manifestations`, `image_generation_prompt` | `str` |
| | `imageUrl` | `str`, défaut `""` — seul champ en camelCase du schéma |
| `story` | `description` | `str` |
| | `characteristics` | `List[str]` |
| `relations` | `parents`, `conjoint`, `descendants` | `List[str]` — **noms en texte libre, pas de référence** |
| `type_specific` | `Optional[Dict[str, Any]]` | Non typé côté Pydantic ; typé côté TS |
| `rendering` | `Optional[Dict[str, Any]]` | Non typé côté Pydantic ; typé côté TS |

**Extensions par type** (`type_specific`, clé = `entity_type` en minuscules) — typage réel
uniquement dans `src/types.ts` :

- `divinity` : `cult { offerings[], taboos[] }`, `domains[]`
- `hero` : `titles[]`, `achievements[]`, `weapons_or_artifacts[]`, `legacy`
- `creature` : `habitat[]`, `powers[]`, `strengths[]`, `weaknesses[]`, `diet`, `size`

**Bloc `rendering`** : `prompt_canon` (`str`), `prompt_variants` (liste de
`{style_id, label, prompt}`), `images` (dictionnaire `style_id` → URL).
Les `style_id` officiels, listés dans le contrat : `photoreal`, `regional_or_ethnic`, `manga`,
`comic_marvel`, `modern_african_painting`.

**`styles_matrix.json`** — objet de données à part entière, non décrit par les schémas.
Indexé par `origin.ethnicity`, avec un mécanisme d'héritage (`inherits: "DEFAULT"`). Chaque
entrée porte `visual_signature`, `visual_philosophy`, `material_science`, `surface_and_finish`,
`atmosphere`, `constraints { forbidden[], mandatory[] }`. C'est une table de référence
culturelle, pas de la donnée d'entité.

**Champs spécifiés au contrat mais absents du schéma Pydantic *et* des données** — à traiter
comme des intentions, pas comme de l'existant : `verification { status, confidence, notes }`,
`sources[{label, url, type, accessed_at}]`, `rendering.negative_prompt`,
`divinity.cult.rituals / places / festivals`, `divinity.genealogy`, `divinity.symbols`,
`hero.quests / allies / enemies / historical_context / genealogy`,
`creature.appearance_notes / encounter / threat_level`.

Le champ `sources` est typé dans `src/types.ts` mais pas dans `engine/domain.py`.
`verification` n'est typé nulle part.

**Couche plateforme** — aucune. `engine/api.py` expose `/health`, `/preview/{entity_name}` et
`POST /generate`. Aucune authentification, CORS ouvert à `*` par défaut. `/generate` appelle
Vertex AI Imagen, écrit l'image dans `public/generated_images/`, puis **réécrit intégralement
`mythology_data.json`** via `save_mythology_data`. Les entités sont résolues par nom, en
comparaison insensible à la casse. Pas d'identifiant, pas de verrou, pas d'historique, pas
d'auteur.

---

## 2. Comparaison des trois versions

### 2.1 Structure générale

| | V0 `AfricanPantheonAPI` | V1 `AfrMyth_API` | V2 `african_mythology_V2` |
|---|---|---|---|
| Stratégie | 3 modèles plats indépendants | `Entity` + 3 tables `*Details` (1-1) | 1 objet + blocs imbriqués + `type_specific` |
| Persistance | SQLite (livré) / Postgres (requis par le code) | PostgreSQL | Fichier JSON |
| Identifiant | `id` auto | `id` auto + `unique_together(name, entity_type)` | **aucun** — le nom fait office de clé |
| Typologie | `FK Category` + 3 tables | `entity_type` (énumération) | `entity_type` + `category` (texte libre) |
| Images | `M2M` pour `Divinity`, `ImageField` pour les deux autres | `M2M` uniformisé | `imageUrl` + `rendering.images{style}` |
| Validation | Mixins et `clean()` applicatifs | Contraintes en base | Pydantic, `extra='allow'` |
| Couche API | DRF, sans permissions | aucune | FastAPI, sans authentification |

### 2.2 Évolution champ par champ

Lecture : `—` absent, `?` présent mais sous une autre forme.

| Concept | V0 | V1 | V2 | Trajectoire |
|---|---|---|---|---|
| Nom | `name` ×3 modèles | `Entity.name` | `name` | Stable |
| Type | `FK Category` | `entity_type` énum | `entity_type` + `category` | **Scission** : le type et le libellé éditorial se séparent en V2 |
| Pays | `country` (`CharField`) | `country_of_origin` | `origin.country` | Renommé deux fois |
| Région / culture | `origin` (`CharField`) | — | `origin.cultural_region` | Perdu en V1, réintroduit en V2 avec un autre sens |
| Ethnie | `ethnicity` (`ArrayField`) | `ethnicity` (`CharField`) | `origin.ethnicity` (`str`) | **Multiple → singulier**, délibérément : devient la clé de la `styles_matrix` |
| Panthéon | `Divinity` seule | `DivinityDetails` seule | **toutes entités** | Promu au tronc commun en V2 |
| Genre | 3 choix (`M`/`F`/`A`) | 5 choix (+ `Spirit`, `Element`) | texte libre | **Contrainte relâchée à chaque étape** |
| Rôle culturel | `Divinity` seule | `DivinityDetails` seule | `identity.cultural_role`, toutes entités | Promu au tronc commun |
| Alignement | `Divinity` seule | `DivinityDetails` seule | `identity.alignment`, toutes entités | Promu au tronc commun |
| Domaines | `domain` (`ArrayField`) | `domains` | `attributes.domains` **et** `type_specific.divinity.domains` | **Dupliqué en V2** |
| Symboles | `main_symbol` | `main_symbols` | `attributes.symbols` | Renommé deux fois |
| Animaux, objets de pouvoir | `Divinity` | `DivinityDetails` | `attributes.*`, toutes entités | Promu au tronc commun |
| Signes physiques | `distincts_physicals_signs` | — | `appearance.physical_signs` | Perdu en V1, rétabli en V2 |
| Manifestations | `CharField(255)` | `CharField(255)` | `appearance.manifestations` (`str`) | Stable |
| Récit | `Divinity.story_description`, `Hero.story`, `MythicalCreature.description` | **—** | `story.description`, toutes entités | **Perdu en V1, unifié en V2** |
| Caractéristiques | `ArrayField` | `ArrayField` | `story.characteristics` | Stable, rattaché au récit en V2 |
| Parents, descendants | `Divinity` : `ArrayField` ; `Hero` : `CharField` | **—** | `relations.*` (`List[str]`), toutes entités | Perdu en V1, unifié en V2 |
| Conjoint | `Divinity.conjoint` | `DivinityDetails.consorts` | `relations.conjoint` | Renommé deux fois |
| Titres, exploits | `Hero` : `CharField` / `TextField` | `HeroDetails` : idem | `type_specific.hero` : **listes** | Scalaire → liste |
| Alliés, ennemis | `Hero` : `TextField` | `HeroDetails` : `ArrayField` | **contrat seulement, absent des données** | Régression en V2 |
| Habitat | `CharField(255)` | `CharField(255)` requis | `type_specific.creature.habitat` : **liste** | Scalaire → liste |
| Pouvoirs | `MythicalCreature.powers` | **—** | `type_specific.creature.powers` | Perdu en V1, rétabli en V2 |
| Forces, faiblesses, régime, taille | `MythicalCreature` | `MythicalCreatureDetails` | `type_specific.creature` | Stable |
| Prompt d'image | `Divinity.prompt`, supprimé en migration 0007 | — | `appearance.image_generation_prompt` + bloc `rendering` complet | **Trajectoire la plus marquée** : d'un champ supprimé à un sous-système |
| Mythes rattachés | `associated_myths`, supprimé en 0012 | — | — | **Abandonné** |
| Festivals, culte | `festivals`, supprimé en 0012 | — | `type_specific.divinity.cult` (partiel) | Abandonné puis partiellement rétabli |
| Sources, vérification | — | — | contrat seulement | **Jamais implémenté** |
| Auteur | `created_by` (`CASCADE`) | `created_by` (`SET_NULL`) | **—** | Régression en V2 |
| Horodatage | `date_created` / `date_updated` | `date_created` / `date_modified` | **—** | Régression en V2 |

### 2.3 Ce que la trajectoire indique

Trois mouvements se dégagent, indépendamment des accidents de parcours.

**Convergence vers un tronc commun.** `pantheon`, `cultural_role`, `alignment`, `domains`,
`symbols`, `power_objects`, `symbolic_animals`, le récit et les relations étaient réservés aux
divinités en V0 et V1 ; V2 les donne à toutes les entités. Le modèle consolidé doit partir du
tronc, pas des types.

**Abandon progressif des contraintes.** De choix énumérés en V0 vers un texte libre en V2 pour
le genre ; de contraintes en base en V1 vers `extra='allow'` en V2. C'est la conséquence du
passage à un fichier JSON, pas une décision de modélisation. Le retour à une base relationnelle
est l'occasion de rétablir ce qui doit l'être — et seulement cela.

**Montée en puissance de la génération d'images.** Un champ `prompt` supprimé en V0 devient en
V2 un sous-système complet : prompt canonique, variantes par style, images par style, matrice
de styles indexée par ethnie. C'est la fonctionnalité qui a le plus évolué et la seule qui soit
réellement aboutie en V2.

---

## 3. Données réelles et volumes

### 3.1 V2 — la seule source exploitable

`src/data/mythology_data.json`, 132 Ko, **44 entités**.

| `entity_type` | Nombre |
|---|---|
| `Divinity` | 25 |
| `Hero` | 8 |
| `Creature` | 11 |

Taux de remplissage réel, à distinguer de la présence structurelle des blocs — le contrat
impose que `type_specific` et `rendering` existent toujours, même vides :

| Champ | Rempli |
|---|---|
| `story.description`, `appearance.physical_signs`, `attributes.domains` | 44 / 44 |
| `type_specific` et `rendering` présents (structurellement) | 44 / 44 |
| `rendering.prompt_canon` | 44 / 44 |
| `rendering.prompt_variants` — 4 variantes déclarées par entité, soit 176 | **40 / 176 portent un prompt**, répartis sur **10 / 44 entités** |
| `appearance.imageUrl` non vide | 15 / 44 |
| `rendering.images` non vide | **6 / 44** |
| `relations.parents` non vide | 12 / 44 |
| `relations.conjoint` non vide | 9 / 44 |
| `relations.descendants` non vide | 5 / 44 |
| `type_specific.divinity.cult.offerings` et `cult.taboos` non vides | **4 / 25** |
| `type_specific.divinity.domains` non vide | **4 / 25** |
| `type_specific.hero` — `titles`, `achievements`, `weapons_or_artifacts`, `legacy` | **3 / 8** (les 4 clés sont présentes pour les 8, remplies pour 3) |
| `type_specific.creature` — `habitat`, `powers`, `size` | **3 / 11** |
| `type_specific.creature` — `strengths`, `weaknesses`, `diet` | **3 / 11** |
| `sources` | **0 / 44** |
| `verification` | **0 / 44** |

Autrement dit : le tronc commun (`origin`, `identity`, `attributes`, `appearance`, `story`) est
intégralement renseigné pour les 44 entités, mais **les extensions `type_specific` ne sont
réellement remplies que pour une dizaine d'entités** — 4 divinités, 3 héros, 3 créatures. La
règle « silent add » du contrat V2 fait que les blocs existent partout, ce qui rend le taux de
remplissage apparent trompeur. Le bloc `rendering` suit le même profil : 44 prompts canoniques,
mais seulement 10 entités avec des variantes de style effectivement rédigées.

Vocabulaires observés :

- **`origin.country`** — 24 valeurs distinctes. Nigeria 11, Afrique du Sud 7, Mali 3, Angola 2,
  Kenya 2, le reste unitaire. **Onze valeurs sont composées** : `"Benin / Nigeria"`,
  `"DRC / Angola"`, `"Nigeria / Diaspora"`, `"Pan-African"`, `"Mozambique / Japan"`. Ce dernier
  signale une entité historique (un samouraï d'origine africaine) qui n'est ni une divinité ni
  une créature.
- **`origin.ethnicity`** — 25 valeurs distinctes. Yoruba 12, Zulu 4, Fon 4, Mbundu 2, Mongo 2,
  le reste unitaire. Valeurs non ethniques présentes : `"Multi-ethnic"`, `"Southern Nations"`.
- **`origin.pantheon`** — 24 valeurs distinctes, vocabulaire très hétérogène : Orisha 11,
  `"Southern African"` 6, Vodun 3, **`"Historical Figures"` 3**, `"Legendary"` 2,
  `"East African Cryptids"`, `"Zanzibar Folklore"`, `"Vodun / Orisha"`, `"Vodun / Diaspora"`.
  Le champ mélange traditions religieuses, aires géographiques, genres narratifs et catégories
  historiques.
- **`origin.cultural_region`** — 7 valeurs, les seules qui forment un vocabulaire propre :
  West Africa 20, Southern Africa 9, Central Africa 6, East Africa 5, Cross-Regional 2,
  North Africa 1, Pan-African 1.
- **`category`** — **44 valeurs pour 44 entités** : aucune n'est réutilisée. Ce n'est pas une
  taxonomie mais un sous-titre éditorial (« Orisha of Lightning », « The Rain Queen »,
  « Samurai Warrior »).
- **`identity.gender`** — 5 valeurs : Male 17, Female 12, Neutral 11, Fluid 3, Androgynous 1.
- **`identity.alignment`** — 10 valeurs, majoritairement issues du vocabulaire du jeu de rôle
  (Lawful Good 11, True Neutral 7, Chaotic Evil 4), avec trois intrus hors de ce système :
  `"Neutral"`, `"Unpredictable"`, `"Territorial"`.

`src/data/styles_matrix.json`, 6 Ko : **4 entrées** — `DEFAULT`, `Yoruba`, `Akan`, `Kongo`.
Trois ethnies couvertes sur les 25 présentes dans les données. Le style
`regional_or_ethnic` n'est donc éligible, selon la règle du contrat, que pour **14 entités**
(12 Yoruba + 1 Akan + 1 Kongo).

Les images référencées (`/generated_images/*.png`) **ne sont pas dans le dépôt** :
`public/` ne contient qu'un `.keep`. Elles sont produites à l'exécution et perdues au
redéploiement.

### 3.2 V1 `AfrMyth_API` — données d'amorçage symboliques

Deux fichiers dans `src/datas/`, encodage UTF-8 correct, contenu en français :

- `entities_data.json` (1,5 Ko) — **3 entités** : Olorun (Divinity), Soundiata Keïta (Hero),
  Mokele-mbembe (Mythical Creature). Un exemplaire par type : un jeu de test, pas un corpus.
- `obatala_data.json` (644 o) — **1 entité** : Obatala (Divinity).

**Total : 4 entités**, dont aucune n'apporte de contenu absent de V2. C'est le seul endroit où
le contenu est en français — V2 est intégralement en anglais.

### 3.3 V0 `AfricanPantheonAPI` — base de développement, pas de jeu de données

- `src/db.json` (8 octets) : contient `[` et rien d'autre. **Fichier tronqué, inexploitable.**
- `src/db.sqlite3` (221 Ko) : base de développement committée par erreur, arrêtée à la
  migration `0011`.

| Table | Lignes | Contenu |
|---|---|---|
| `entities_category` | 3 | `divinity`, `heroes`, `creatures` |
| `entities_divinity` | 4 | Oshun, Olokun, Obatala, **`tesé`** (saisie de test) |
| `entities_hero` | 1 | Ditaolane |
| `entities_mythicalcreature` | 3 | Aziza, **Kammapa**, **Kamapa** (doublon orthographique) |
| `user_profiles_userprofile` | 2 | — |
| `auth_user` | 1 | — |

**Total : 8 entités, dont 2 sont du déchet** (`tesé`, et le doublon Kammapa/Kamapa). Les
`Category` y servent de types, confirmant que ce modèle n'avait pas de taxonomie éditoriale.

`src/media/` : 3 images authentiques (`oshun.webp`, `olokun.webp`, `aziza.webp`) et
**5 copies du même `plot.png`** sous `hero_images/` — artefacts de test de téléversement.

### 3.4 Conclusion sur les données

**Il n'existe qu'un seul corpus : les 44 entités de V2.** Les 4 entités de V1 et les 8 de V0
sont des jeux de test. Sur ces 44 entités, 6 disposent d'une image effectivement référencée par
style, une dizaine ont des données spécifiques à leur type, et aucune n'est sourcée.

Le contrat V2 mentionne un objectif de « 400+ entités ». L'écart entre 44 et 400 est le point
qui devrait gouverner les choix de modélisation : ce qui coûte peu à 44 entités (texte libre,
vocabulaire non contrôlé, relations par nom) devient ingérable à 400.

---

## 4. Proposition de modèle consolidé

Proposition, pas décision. Elle prend parti sur trois points, signalés comme tels.

### 4.1 Principes retenus

1. **Tronc commun + extensions par type**, c'est-à-dire le pattern de V1 confirmé par V2.
   Les trois modèles plats de V0 sont écartés : ils dupliquent sept champs et empêchent toute
   recherche transverse.
2. **Les relations généalogiques deviennent de vraies relations**, pas des listes de noms.
   V2 les stocke en `List[str]` parce qu'un fichier JSON ne sait pas faire autrement. En base,
   c'est la source de divergence la plus coûteuse à 400 entités.
3. **Les vocabulaires répétés utiles à l'API deviennent des tables de référence** : ethnie,
   région culturelle et domaine. Le champ `pantheon` n'est pas retenu dans le modèle cible :
   il mélange trop de notions différentes et ne répond pas à un besoin prioritaire de la V1.
4. **La `category` de V2 reste du texte libre** : 44 valeurs uniques pour 44 entités, ce n'est
   pas une taxonomie. Elle est conservée comme sous-titre éditorial.
5. **La plateforme est séparée du métier**, dans des apps distinctes (section 4.5).

### 4.2 Référentiels (app `reference`)

Vocabulaires contrôlés, alimentés par les données existantes, administrés et non saisis
librement.

| Modèle | Champs | Contraintes |
|---|---|---|
| `CulturalArea` | `name`, `slug`, `kind`, `parent` (`FK self`, nullable) | `kind` : `african_region`, `transregional`, `diasporic`. Remplace le référentiel plat initialement proposé |
| `Country` | `name`, `slug`, `iso_code` (nullable) | Les valeurs composées sont décomposées en pays principal et secondaires ; aucune pseudo-valeur comme `Pan-African` |
| `Ethnicity` | `name`, `slug`, `primary_cultural_area` (`FK`, nullable) | `unique` sur `name` et `slug`. 25 valeurs connues |
| `Domain` | `name`, `slug` | Vocabulaire de `attributes.domains` |
| `VisualStyle` | `style_id`, `label`, `is_canon` | 5 `style_id` fixés par le contrat V2 |
| `EthnicStyleProfile` | `ethnicity` (`O2O`), `inherits_from` (`FK self`, nullable), `visual_signature`, `visual_philosophy` (`JSON`), `material_science` (`JSON`), `surface_and_finish` (`JSON`), `atmosphere` (`JSON`), `constraints` (`JSON`) | Reprise directe de `styles_matrix.json`. Les sous-blocs restent en `JSONField` : ce sont des fragments de prompt, pas des données interrogeables |

### 4.3 Métier — mythologie (app `mythology`)

**`Entity`** — le tronc commun.

| Champ | Type | Contraintes et justification |
|---|---|---|
| `id` | `UUIDField`, clé primaire | V2 n'a aucun identifiant ; une clé stable est le préalable à toute relation |
| `slug` | `SlugField`, `unique` | Identifiant lisible en URL, dérivé du nom |
| `name` | `CharField(255)` | |
| `entity_type` | `CharField`, choix | `divinity`, `hero`, `creature`. La question des figures historiques est reportée dans l'ADR 0001 |
| `category` | `CharField(255)`, vide autorisé | Sous-titre éditorial, texte libre assumé |
| `primary_country` | `FK(Country)`, nullable | Ancrage géographique principal |
| `secondary_countries` | `M2M(Country)`, vide autorisé | Origines géographiques supplémentaires |
| `primary_ethnicity` | `FK(Ethnicity)`, nullable | Ancrage culturel principal, notamment pour le style |
| `secondary_ethnicities` | `M2M(Ethnicity)`, vide autorisé | Appartenances culturelles supplémentaires |
| `primary_cultural_area` | `FK(CulturalArea)`, nullable | Aire africaine, transrégionale ou diasporique principale |
| `secondary_cultural_areas` | `M2M(CulturalArea)`, vide autorisé | Aires culturelles supplémentaires |
| `variant_of` | `FK(self)`, nullable | Figure de référence dont cette entité est une variante culturelle ; le type peut différer |
| `gender` | `CharField`, choix | `male`, `female`, `neutral`, `androgynous`, `fluid` — les 5 valeurs observées |
| `cultural_role` | `CharField(255)`, vide autorisé | |
| `alignment` | `CharField`, vide autorisé | Décision éditoriale reportée dans l'ADR 0001 |
| `story_description` | `TextField`, vide autorisé | Le récit, rétabli au tronc commun |
| `characteristics` | `ArrayField(CharField(100))`, défaut `list` | |
| `physical_signs` | `ArrayField(CharField(100))`, défaut `list` | |
| `manifestations` | `TextField`, vide autorisé | `TextField` et non `CharField(255)` : la limite des trois versions est arbitraire |
| `verification_status` | `CharField`, choix, défaut `draft` | `draft`, `reviewed`, `curated`. Spécifié au contrat V2, jamais implémenté |
| `created_at`, `updated_at` | `DateTimeField` | Rétablis : V2 les a perdus |

Contraintes : `unique` sur `slug` ; `UniqueConstraint(name, entity_type)`, reprise de V1 ;
index sur `entity_type`, `primary_ethnicity`, `primary_country` et `primary_cultural_area`.

**Attributs symboliques** — `attributes.domains`, `symbols`, `power_objects`,
`symbolic_animals`. V2 les met au tronc commun, les trois versions antérieures les réservaient
aux divinités, et les données ne les renseignent de fait que pour les divinités.
Proposition : les porter sur `Entity` plutôt que sur `DivinityProfile`, puisque le contrat V2
l'a décidé et que les créatures ont bien des animaux symboliques. `domains` via une table de
liaison vers `Domain` ; `symbols`, `power_objects`, `symbolic_animals` en `ArrayField`, leur
vocabulaire n'étant pas assez stable pour justifier une table.

**Extensions par type** — `OneToOneField(Entity, CASCADE)`, suivant V1.

`DivinityProfile` — `cult_offerings`, `cult_taboos`, `cult_rituals`, `cult_places`,
`cult_festivals`, tous en `ArrayField(CharField(100))`, défaut `list`. Les cinq sous-champs sont
spécifiés au contrat V2 ; `festivals` restaure le champ supprimé en V0.

`HeroProfile` — `titles`, `achievements`, `quests`, `weapons_or_artifacts` en `ArrayField` ;
`legacy` et `historical_context` en `TextField`. Les alliés et ennemis ne sont pas ici : voir
`EntityRelation`.

`CreatureProfile` — `habitat`, `powers`, `strengths`, `weaknesses` en `ArrayField` ;
`diet`, `size` en `CharField(255)` ; `appearance_notes` en `TextField` ;
`threat_level` en choix (`low`, `medium`, `high`, `mythic`) nullable ;
`encounter_omens`, `encounter_rules`, `known_sightings` en `ArrayField`.

**`EntityRelation`** — le changement structurel principal par rapport à V2.

| Champ | Type |
|---|---|
| `from_entity` | `FK(Entity, CASCADE, related_name='relations_from')` |
| `to_entity` | `FK(Entity, CASCADE, related_name='relations_to')`, nullable |
| `to_name` | `CharField(255)`, vide autorisé |
| `kind` | `CharField`, choix : `parent`, `child`, `consort`, `ally`, `enemy` |
| `notes` | `TextField`, vide autorisé |

`to_entity` **ou** `to_name` est renseigné, jamais les deux :
`CheckConstraint` sur l'exclusivité, plus `UniqueConstraint(from_entity, to_entity, kind)`.
Le `to_name` est indispensable : sur les 44 entités de V2, les noms cités dans `relations`
désignent souvent des entités absentes du corpus. Les ignorer perdrait de l'information ;
exiger une entité cible bloquerait la saisie. Un lien non résolu reste du texte et devient une
référence le jour où la cible est créée.

Ce modèle unifie quatre choses aujourd'hui éparpillées : `relations.{parents, conjoint,
descendants}` de V2, `consorts` de `DivinityDetails` en V1, et `allies` / `enemies` de
`HeroDetails` — ces deux derniers étant spécifiés au contrat V2 mais absents de ses données.

**`Source`** — jamais implémenté dans aucune des trois versions, spécifié au contrat V2.

`entity` (`FK`), `label`, `url` (nullable), `kind` (choix : `book`, `article`, `encyclopedia`,
`museum`, `oral`, `other`), `accessed_at` (`DateField`, nullable), `notes`.

### 4.4 Métier — rendu visuel (app `rendering`)

| Modèle | Champs | Notes |
|---|---|---|
| `RenderingProfile` | `entity` (`O2O`), `prompt_canon` (`TextField`), `negative_prompt` (`TextField`) | Un profil par entité |
| `PromptVariant` | `profile` (`FK`), `style` (`FK VisualStyle`), `prompt` (`TextField`), `notes` | `UniqueConstraint(profile, style)` |
| `EntityImage` | `entity` (`FK`), `style` (`FK VisualStyle`, nullable), `image` (`ImageField`), `caption`, `is_primary` (booléen), `generated_from_prompt` (`TextField`), `generated_at`, `created_by` (`FK` utilisateur, nullable) | `UniqueConstraint(entity, is_primary)` conditionnelle sur `is_primary=True` |

`EntityImage` consolide trois approches divergentes : le `ImageWithCaption` en `M2M` de V0 et
V1, et le dictionnaire `rendering.images{style_id: url}` de V2. Le `M2M` était un mauvais
choix — une image appartient à une entité et n'est pas partagée. La `FK` et le `is_primary`
remplacent le `appearance.imageUrl` de V2, qui n'était que la case `photoreal` dupliquée.

`generated_from_prompt` conserve le prompt effectivement utilisé, que V2 perd à chaque
génération. C'est ce qui rend une image reproductible.

### 4.5 Plateforme minimale : utilisateurs, propriété et modération

Cette couche est nécessaire au projet portfolio : elle démontre les rôles, les permissions au
niveau objet et un workflow réel. Elle doit toutefois rester petite. Pour la première version,
**l'entité elle-même est l'objet contribué** : pas de copie parallèle de l'entité dans un
`JSONField`, pas de moteur générique de workflow et pas de journal d'audit maison.

`accounts.User` — sous-classe minimale de `AbstractUser`, avec `email` unique. Les trois rôles
`contributor`, `reviewer` et `administrator` sont des groupes Django créés par migration de
données. Il n'y a pas de champ `role` en doublon : l'appartenance aux groupes est l'unique source
d'autorisation et fournit le vocabulaire explicite utilisé par l'API et les tests.

`Entity` reçoit les champs de plateforme suivants :

| Champ | Type | Rôle |
|---|---|---|
| `created_by` | `FK(User, SET_NULL)`, nullable | Propriétaire de l'objet ; nullable pour les données importées |
| `status` | choix `draft`, `submitted`, `published`, `rejected` | État du cycle de contribution |
| `reviewed_by` | `FK(User, SET_NULL)`, nullable | Dernier relecteur ayant statué |
| `submitted_at` | `DateTimeField`, nullable | Date de soumission |
| `reviewed_at` | `DateTimeField`, nullable | Date de publication ou de rejet |
| `review_notes` | `TextField`, vide autorisé | Motif ou consigne du relecteur |

**Transitions autorisées**

| Depuis | Vers | Acteur autorisé | Intention |
|---|---|---|---|
| `draft` | `submitted` | propriétaire, relecteur, administrateur | Demander une revue |
| `submitted` | `draft` | propriétaire, relecteur, administrateur | Retirer une soumission pour la corriger |
| `submitted` | `published` | relecteur, administrateur | Publier |
| `submitted` | `rejected` | relecteur, administrateur | Refuser avec commentaire |
| `rejected` | `draft` | propriétaire, relecteur, administrateur | Corriger puis resoumettre |
| `published` | `draft` | administrateur uniquement | Retirer temporairement pour correction |

Le retour `submitted` → `draft` est un retrait, pas un rejet. Les transitions passent par une
action de service explicite et sont testées ; un client ne peut pas modifier librement
`status` avec un `PATCH` ordinaire.

**Matrice de permissions V1**

| Action | Anonyme | Contributeur | Relecteur | Administrateur |
|---|---|---|---|---|
| Lire une entité `published` | Oui | Oui | Oui | Oui |
| Lire son propre `draft` ou `rejected` | Non | Oui | Oui | Oui |
| Lire le `draft` ou `rejected` d'un autre | Non | Non | Oui | Oui |
| Lire son propre `submitted` | Non | Oui | Oui | Oui |
| Lire le `submitted` d'un autre | Non | Non | Oui | Oui |
| Créer une entité | Non | Oui, propriétaire automatique | Oui | Oui |
| Modifier un `draft` ou `rejected` | Non | Propriétaire seulement | Oui | Oui |
| Modifier une entité `submitted` | Non | Non | Oui | Oui |
| Modifier une entité `published` | Non | Non | Non | Oui |
| Supprimer un `draft` ou `rejected` | Non | Propriétaire seulement | Oui | Oui |
| Supprimer une entité `submitted` | Non | Non | Non | Oui |
| Supprimer une entité `published` | Non | Non | Non | Oui |
| Soumettre (`draft` → `submitted`) | Non | Propriétaire seulement | Oui | Oui |
| Retirer (`submitted` → `draft`) | Non | Propriétaire seulement | Oui | Oui |
| Publier ou rejeter | Non | Non | Oui | Oui |
| Rouvrir une entité publiée | Non | Non | Non | Oui |

Une évolution d'une entité déjà publiée par un contributeur nécessitera plus tard une vraie
proposition de modification séparée. Elle est volontairement hors V1 : l'ajouter maintenant
dupliquerait les données et complexifierait fortement les serializers et les permissions.

### 4.6 Coût d'import du corpus V2

Points de friction identifiés pour les 44 entités :

| Point | Difficulté |
|---|---|
| Absence d'identifiant | Génération de `slug` à partir de `name`, à vérifier pour les noms composés |
| `relations` par nom | Résolution en deux passes : créer les entités, puis rapprocher les noms. Les non-résolus partent en `to_name` |
| `country` composés (11 cas) | Décomposition en `primary_country` et `secondary_countries` selon l'ADR 0002 |
| `alignment` (3 valeurs hors système) | Import temporaire possible ; décision éditoriale reportée dans l'ADR 0001 |
| `type_specific` structurellement présent mais vide | Ne pas créer de `*Profile` vide : filtrer sur contenu réel |
| Images | Les fichiers `/generated_images/*.png` sont absents du dépôt. Les URL seront importées sans fichier, ou écartées |
| Doublon `attributes.domains` / `type_specific.divinity.domains` | Union sans perte dans `Entity.domains`, erreurs certaines corrigées et ambiguïtés signalées |

---

## 5. Ce qui est écarté de la proposition

Pour mémoire, afin que ces choix ne soient pas refaits implicitement.

- **Les trois modèles plats de V0** — dupliquent sept champs, interdisent la recherche
  transverse, et le typage y est incohérent d'un modèle à l'autre (`parents` en `ArrayField`
  pour `Divinity`, en `CharField` pour `Hero`).
- **`Category` comme modèle (V0)** — y servait de type, rôle désormais tenu par `entity_type`.
  La `category` de V2 est autre chose et reste en texte libre.
- **`type_specific` en `JSONField`** — reproduirait le `extra='allow'` de Pydantic : aucune
  contrainte, aucune requête possible, aucune validation. Les tables `*Profile` sont le prix à
  payer pour que `threat_level` ou `diet` soient interrogeables.
- **`relations` en `ArrayField` de noms** — tenable à 44 entités, pas à 400. Rend impossible
  tout graphe généalogique fiable, qui est pourtant déjà une fonctionnalité du front V2
  (`LineageGraph.tsx`, `LineageTree.tsx`).
- **`ManyToManyField` vers les images (V0, V1)** — une image n'est jamais partagée entre
  entités. La `FK` est correcte.
- **`appearance.imageUrl` comme champ (V2)** — duplique `rendering.images['photoreal']`.
  Remplacé par `EntityImage.is_primary`.
- **La réécriture intégrale du fichier de données (V2)** — remplacée par des écritures
  ciblées, les horodatages et l'identification du propriétaire et du relecteur. Un journal
  d'audit personnalisé reste hors de la V1.

---

## 6. Arbitrages de modélisation — décisions actées

Les choix ci-dessous sont désormais tranchés dans
[`ADR 0002`](adr/0002-decisions-structurantes-phase-0.md). Les questions non bloquantes restent
consignées dans [`ADR 0001`](adr/0001-decisions-metier-reportees.md).

### Décision actée — pas de modèle `Pantheon` dans la V1

Cette décision ne signifie pas que la notion n'a jamais existé. Dans V0 et V1, `pantheon`
était un champ texte porté par les divinités. Dans V2, `origin.pantheon` est un champ texte du
tronc commun. L'audit avait proposé de transformer ces valeurs en un véritable modèle Django
`Pantheon`, relié à `Entity` par une clé étrangère.

Cette proposition n'est pas retenue pour la première version. Les 24 valeurs observées dans
les 44 entités mélangent traditions religieuses, folklore, zones géographiques, figures
historiques et catégories narratives. Créer maintenant une table contrôlée donnerait une
apparence de cohérence à un vocabulaire qui n'en a pas encore et n'apporterait aucun bénéfice
prioritaire aux premiers endpoints.

Le champ source n'est toutefois pas effacé de l'histoire du projet : il reste décrit dans
l'audit et présent dans le corpus V2 original. L'importeur devra signaler dans son rapport que
`origin.pantheon` n'est pas importé, afin d'éviter une perte silencieuse. Si un futur besoin
produit exige de filtrer ou de regrouper les entités par tradition, la notion sera réexaminée
à partir d'un vocabulaire éditorial clarifié et fera l'objet d'une nouvelle ADR.

### A. Ethnies multiples — principale + secondaires

V0 avait `ethnicity` en `ArrayField`. V1 et V2 l'ont ramené au singulier, et le contrat V2
justifie ce choix : c'est la clé de la matrice de styles, elle doit être « aussi précise et
singulière que possible ».

Mais les données contiennent `"Multi-ethnic"` et `"Southern Nations"`, qui sont des aveux de
pluralité déguisés en valeur unique. Et 4 entités Fon relèvent d'aires de diffusion qui
dépassent une ethnie (`"Vodun / Diaspora"`).

Options : (a) garder `FK` unique, et traiter `"Multi-ethnic"` comme une valeur de référentiel
à part entière ; (b) `FK` principale `ethnicity` + `M2M` `secondary_ethnicities`, la `FK`
restant la clé de style ; (c) `M2M` seul avec un indicateur « ethnie de style ».

**Décision : option (b).** Une ethnie principale facultative sert d'ancrage, complétée par des
ethnies secondaires facultatives.

### B. Origines composées — pays principal + secondaires

Onze valeurs de `country` sont composées : `"Benin / Nigeria"`, `"DRC / Angola"`,
`"Nigeria / Diaspora"`, `"Zimbabwe / Zambia"`, `"Kenya / Tanzania"`,
`"South Africa / Namibia"`, `"Mozambique / Japan"`, `"Benin / Haiti"`, `"Benin / Togo"`,
plus `"Pan-African"` et `"Congo Basin"` qui ne sont pas des pays.

L'indicateur initialement proposé `is_composite` sur `Country` était un contournement : cela
stocke `"Benin / Nigeria"` comme une entité de référentiel, et le filtre « entités du Bénin »
ne la trouvera pas.

Options : (a) `M2M` `countries` sur `Entity`, qui est la modélisation correcte mais perd la
notion de pays principal ; (b) `FK primary_country` + `M2M secondary_countries` ;
(c) garder le texte libre et accepter de ne pas filtrer par pays.

Par ailleurs `"Pan-African"` et `"Congo Basin"` ne sont pas des pays. Ils sont classés dans
`CulturalArea`, avec un type distinct pour les aires africaines, transrégionales et
diasporiques.

**Décision : option (b).** `primary_country` porte l'ancrage principal et
`secondary_countries` les autres pays. Les variantes culturelles sont des entités autonomes
reliées par `variant_of`.

### C. Domaines dupliqués — emplacement unique sur `Entity`

Les données portent `attributes.domains` **et** `type_specific.divinity.domains`. Les 25
divinités ont les deux clés ; `attributes.domains` est rempli pour les 25,
`type_specific.divinity.domains` pour 4 seulement. Le contrat note que `divinity.*` est
« the richer home » tout en autorisant le maintien de `attributes.*`. Il laisse donc le doublon
ouvert, et les données non plus : pour ces 4 divinités, **les deux listes divergent**.

| Entité | `attributes.domains` | `type_specific.divinity.domains` |
|---|---|---|
| Mami Wata | Water, Wealth, Beauty, Seduction | Water, Wealth, **Healing**, Beauty |
| Oya | Wind, Storms, Cemeteries, Transformation | Wind, Storms, **`Ceteris`**, **Change** |
| Ogun | Iron, War, Technology, Strength | Iron, War, Technology, **Oaths** |
| Eshu | Crossroads, Choices, Communication | Crossroads, Communication, **Chance, Trickery** |

Aucun des deux emplacements n'est un sur-ensemble de l'autre : fusionner en gardant un seul
emplacement perd `Healing`, `Oaths`, `Chance`, `Trickery` d'un côté, ou `Seduction`, `Strength`,
`Choices` de l'autre. Et `"Ceteris"` est une corruption de `"Cemeteries"` : la version
`type_specific` est en partie dégradée.

C'est l'unique endroit du corpus où une fusion automatique détruirait de la donnée. Vu le
volume — 4 entités, une vingtaine de termes — je recommande une réconciliation manuelle avant
import, puis un emplacement unique (option a). Je n'invente pas la liste fusionnée : les termes
à retenir relèvent de ton arbitrage.

La même ambiguïté vaut pour `symbols`, `power_objects` et `symbolic_animals`, que le contrat
déclare à la fois dans `attributes` et dans le bloc `divinity`.

Options : (a) un seul emplacement sur `Entity`, et l'import privilégie `attributes.*` qui est
le seul rempli ; (b) un seul emplacement sur `DivinityProfile`, ce qui prive les créatures de
leurs animaux symboliques ; (c) conserver les deux avec des sémantiques distinctes, ce qui
suppose de définir laquelle.

**Décision : option (a).** `Entity.domains` est l'emplacement unique pour les trois types.
L'import part de `attributes.domains`, ajoute les valeurs spécifiques sans perte, corrige les
erreurs certaines et signale les ambiguïtés.

### D. Corpus V2 — import initial puis source de vérité PostgreSQL

Question de périmètre, pas de modélisation. Les 44 entités sont la seule donnée réelle
disponible, mais `african_mythology_V2` est un projet vivant dont le front et le moteur
d'images continuent d'évoluer.

Options : (a) importer le corpus une fois, et le projet Django devient la source de vérité,
V2 devenant un consommateur de l'API ; (b) importer en lecture seule, comme jeu de
démonstration, V2 restant autonome ; (c) ne rien importer et repartir d'une base vide.

**Décision : option (a).** Un importateur idempotent charge le corpus initial, puis
`afrimyth-api` et PostgreSQL deviennent l'unique source de vérité. Le JSON reste une archive
d'origine, sans synchronisation retour.

---

## 7. Stratégie Django pour le tronc commun et les profils

Le principe métier est acquis : un tronc `Entity` et des données propres aux divinités, héros
et créatures. Trois implémentations Django restent possibles.

| Option | Requêtes | Migrations | Serializers | Performance et contraintes |
|---|---|---|---|---|
| Héritage multi-table (`Divinity(Entity)`, etc.) | Jointure implicite vers la table enfant ; requêtes polymorphes moins lisibles | Couplées à l'héritage Django, plus délicates à faire évoluer | Un serializer par sous-classe, détection du type nécessaire | Correct à ce volume, mais magie ORM et risque de requêtes supplémentaires |
| Table unique avec tous les champs nullables | Lecture simple sans jointure | Simples au début, table vite large et remplie de colonnes sans objet | Un gros serializer avec validation conditionnelle | Lecture rapide, mais contraintes métier faibles et évolutions confuses |
| `Entity` + profils `OneToOne` explicites | Jointure seulement quand le détail du type est demandé ; `select_related` maîtrisable | Tables indépendantes, ajouts ciblés par type | Un serializer commun et trois serializers de profil composables | Une jointure sur le détail, mais contraintes claires et comportement ORM explicite |

**Recommandation : `Entity` + profils `OneToOne` explicites.** C'est le meilleur compromis
entre simplicité, validation et évolutivité. Contrairement à l'héritage multi-table, les
relations ne sont pas cachées par Django. Contrairement à la table unique, un héros ne porte
pas vingt colonnes de créature à `NULL`. Pour éviter une abstraction prématurée, la V1 garde
trois profils concrets et une validation simple : le profil présent doit correspondre à
`entity_type`.

Cette recommandation est validée dans l'ADR 0002.

---

## 8. Ordre de travail après la phase 0

1. Construire le premier incrément exécutable : projet Django, utilisateur, `Entity`, un profil,
   migrations et tests essentiels.
2. À partir de ce modèle qui tourne, préciser les premiers endpoints et poursuivre par petits
   incréments testés.

Pas de contrat REST exhaustif, d'internationalisation, de moteur générique de workflow ni de
pipeline complet d'import avant que le premier modèle soit exécutable.

---

## 9. Note sur le dossier `_audit/`

Les trois dépôts sont clonés dans `_audit/` à la racine de `django-portfolio`. Ce dépôt n'a
pas de `.gitignore` — l'instruction était de n'en ajouter la ligne que s'il en existait un, je
n'en ai donc pas créé. Conséquence : `_audit/` apparaît en fichiers non suivis dans
`git status`, et contient environ 3 Mo de médias et une base SQLite. À ignorer ou à supprimer
avant le prochain commit.
