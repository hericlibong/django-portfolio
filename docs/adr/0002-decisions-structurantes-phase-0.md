# ADR 0002 — Décisions structurantes de la phase 0

- **Statut :** accepté
- **Portée :** modèle initial de `afrimyth-api`

## Contexte

Cette ADR rassemble les décisions qui doivent être prises avant les premières migrations.
Chaque décision est ajoutée après validation explicite. La roadmap n'est cochée qu'une fois la
décision consignée ici.

## 1. Gestion des ethnies

- **Statut :** validée
- **Décision :** une entité possède une `primary_ethnicity` facultative et peut posséder zéro ou
  plusieurs `secondary_ethnicities`.

### Justification

Le corpus contient des entités associées à plusieurs cultures, tandis que la matrice de styles
requiert un ancrage culturel principal. Un champ unique ferait perdre les appartenances
secondaires et transformerait des libellés comme `Multi-ethnic` en fausses ethnies. Un M2M seul
obligerait à ajouter un mécanisme séparé pour désigner l'ancrage principal.

### Conséquences

- `Entity.primary_ethnicity` sera une clé étrangère nullable vers `Ethnicity`.
- `Entity.secondary_ethnicities` sera une relation plusieurs-à-plusieurs facultative vers
  `Ethnicity`.
- Une même ethnie ne devra pas être présente à la fois comme principale et secondaire.
- L'API distinguera explicitement l'ethnie principale des ethnies secondaires.
- L'import devra transformer les valeurs vagues comme `Multi-ethnic` en informations
  exploitables ou les signaler, plutôt que de créer automatiquement une ethnie de ce nom.

## 2. Gestion des pays et des origines géographiques multiples

- **Statut :** validée
- **Décision :** une entité possède un `primary_country` facultatif et peut posséder zéro ou
  plusieurs `secondary_countries`.

### Justification

Plusieurs figures sont présentes dans plusieurs pays. Un champ texte composé empêcherait les
filtres fiables, tandis qu'une relation multiple seule ne permettrait pas d'indiquer l'ancrage
géographique principal utilisé pour présenter l'entité.

### Conséquences

- `Entity.primary_country` sera une clé étrangère nullable vers `Country`.
- `Entity.secondary_countries` sera une relation plusieurs-à-plusieurs facultative vers
  `Country`.
- Seuls de véritables pays seront enregistrés dans `Country`.
- Le premier pays indiqué par le corpus deviendra l'ancrage principal par défaut, avec
  signalement des cas ambigus dans le rapport d'import.
- Un pays ne devra pas être présent à la fois comme principal et secondaire.
- `Pan-African`, `Congo Basin` et `Diaspora` ne seront pas transformés en pays. Leur
  classification culturelle et géographique fait l'objet de la décision suivante.

## 3. Aires culturelles et variantes d'une même figure

- **Statut :** validée
- **Décision :** les classifications régionales, transafricaines et diasporiques sont portées
  par un référentiel `CulturalArea`. Les variantes culturelles sont des `Entity` autonomes
  reliées facultativement à une figure de référence par `variant_of`.

### Justification

`Congo Basin`, `Pan-African` et `Diaspora` ne décrivent pas le même phénomène. Le premier est
une aire africaine, le deuxième une portée transrégionale et le troisième une transmission
hors d'Afrique. Par ailleurs, une même figure peut avoir selon les régions un récit, une
apparence et même une qualification différente. Fusionner toutes ces formes dans une seule
fiche ferait disparaître ces particularités.

### Conséquences

- `CulturalArea` est un modèle Django de référence, et non un type d'entité ou un profil métier.
- Il porte au minimum `name`, `slug`, `kind` (`african_region`, `transregional`, `diasporic`) et
  un `parent` facultatif pour la hiérarchie.
- Une entité possède une aire culturelle principale facultative et peut être associée à des
  aires secondaires.
- `Entity.variant_of` est une clé étrangère facultative et autoréférentielle.
- Chaque variante conserve ses propres pays, ethnies, aires, récit, apparence et profil.
- Le `entity_type` d'une variante est décidé indépendamment de celui de sa figure de référence :
  une forme régionale peut être une divinité alors qu'une autre est une créature.
- La fiche générale de Mami Wata est classée initialement comme `creature`. Une variante ne sera
  classée `divinity` que si son contexte culturel le justifie explicitement.
- Une destination diasporique précise est préférée au terme générique `Diaspora` lorsqu'elle
  est connue ; les pays restent enregistrés séparément dans les champs géographiques validés.

## 4. Emplacement unique des domaines

- **Statut :** validée
- **Décision :** les domaines sont associés à `Entity` pour les trois types, au moyen d'une
  relation plusieurs-à-plusieurs vers le référentiel `Domain`. Aucun doublon n'est conservé
  dans les profils propres aux types.

### Justification

Le corpus renseigne `attributes.domains` pour les 44 entités : les domaines divins de Shango,
les thèmes narratifs d'Anansi ou les associations symboliques de Mokele-mbembe peuvent ainsi
être recherchés de manière transverse. Le doublon `type_specific.divinity.domains`, rempli
pour quatre divinités seulement, diverge de la liste commune et ne doit pas devenir un second
emplacement dans la base.

### Conséquences

- `Domain` est un référentiel partagé et `Entity.domains` une relation plusieurs-à-plusieurs.
- L'API expose un seul champ `domains`, quel que soit le type de l'entité.
- L'import utilise `attributes.domains` comme base et lui ajoute sans perte les valeurs
  supplémentaires de `type_specific.divinity.domains`.
- Les erreurs certaines, comme `Ceteris` pour `Cemeteries`, sont corrigées.
- Les synonymes ou nuances discutables, comme `Change` et `Transformation`, sont conservés et
  signalés pour revue éditoriale plutôt que fusionnés automatiquement.
- Les quatre divergences connues concernant Mami Wata, Oya, Ogun et Eshu sont traitées dans le
  rapport d'import.

## 5. Corpus V2 et source de vérité

- **Statut :** validée
- **Décision :** le corpus V2 est importé une fois dans PostgreSQL. Après cet import,
  `afrimyth-api` et sa base deviennent l'unique source de vérité pour les données métier.

### Justification

Maintenir simultanément le fichier JSON et PostgreSQL comme sources modifiables créerait des
divergences et imposerait une synchronisation bidirectionnelle sans valeur pour la V1. La
nouvelle API doit précisément remplacer le fichier comme système de gestion des données.

### Conséquences

- Le fichier `mythology_data.json` est conservé comme source historique de l'import initial.
- Les corrections et nouvelles entités sont ensuite enregistrées uniquement via l'API, son
  administration ou ses services métier.
- Aucun mécanisme de synchronisation retour vers le JSON n'est développé.
- La commande d'import est idempotente pour les installations et démonstrations, mais elle ne
  doit pas écraser silencieusement des modifications réalisées ensuite dans PostgreSQL.
- Lors de la future intégration, `african_mythology_V2` lira les données depuis l'API au lieu de
  son fichier JSON local.

## 6. Stratégie d'implémentation du tronc commun et des profils

- **Statut :** validée
- **Décision :** le modèle utilise une table commune `Entity` et trois profils explicites
  `DivinityProfile`, `HeroProfile` et `CreatureProfile`, chacun relié à `Entity` par un
  `OneToOneField`.

### Justification

Une table unique accumulerait de nombreuses colonnes nulles et affaiblirait les règles propres
à chaque type. L'héritage multi-table de Django masquerait les jointures et rendrait les
serializers polymorphes moins explicites. Les relations `OneToOne` conservent la séparation
métier tout en restant simples à interroger et à faire évoluer.

### Conséquences

- `Entity` porte tous les champs communs et `entity_type`.
- Chaque profil contient uniquement les champs propres à son type.
- Une entité doit posséder exactement le profil correspondant à `entity_type`.
- Cette cohérence est garantie par le service de création et couverte par des tests ; elle ne
  repose pas sur une abstraction polymorphe supplémentaire.
- Les vues de détail utilisent `select_related` pour maîtriser la jointure vers le profil.
- Une variante reliée par `variant_of` peut avoir un type et un profil différents de ceux de sa
  figure de référence.

## 7. Workflow de contribution et matrice de permissions

- **Statut :** validée
- **Décision :** l'entité elle-même suit un workflow simple. Aucun modèle générique de
  contribution ou moteur de workflow supplémentaire n'est créé dans la V1.

### Rôles

Les rôles `contributor`, `reviewer` et `administrator` sont des groupes Django. Il n'existe pas
de champ `role` en doublon sur l'utilisateur.

### Statuts et transitions

| Depuis | Vers | Acteur autorisé |
|---|---|---|
| `draft` | `submitted` | propriétaire, relecteur, administrateur |
| `submitted` | `draft` | propriétaire, relecteur, administrateur |
| `submitted` | `published` | relecteur, administrateur |
| `submitted` | `rejected` | relecteur, administrateur |
| `rejected` | `draft` | propriétaire, relecteur, administrateur |
| `published` | `draft` | administrateur uniquement |

Le retour `submitted` → `draft` est un retrait de soumission, pas un rejet : il permet notamment
au propriétaire de reprendre immédiatement une contribution envoyée par erreur. Le statut
n'est jamais modifiable par un `PATCH` ordinaire. Chaque transition passe par une action
explicite de l'API.

### Matrice de permissions

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
| Soumettre une entité | Non | Propriétaire seulement | Oui | Oui |
| Retirer sa soumission | Non | Propriétaire seulement | Oui | Oui |
| Publier ou rejeter | Non | Non | Oui | Oui |
| Rouvrir une entité publiée | Non | Non | Non | Oui |

### Conséquences

- `Entity` porte `created_by`, `status`, `submitted_at`, `reviewed_at`, `reviewed_by` et
  `review_notes`.
- Les permissions combinent rôle, propriété et statut de l'objet.
- La matrice fait l'objet de tests paramétrés couvrant les combinaisons utiles.
- Une proposition de modification séparée pour une entité déjà publiée reste hors V1.

### Note — suppression logique et managers Django

- `Entity.variant_of` utilise `on_delete=PROTECT`.
- Le service refuse également la suppression logique d'une figure de référence tant qu'elle
  possède des variantes actives.
- Une suppression renseigne `deleted_at` et `deleted_by` ; aucun champ `is_deleted` redondant
  n'est ajouté.
- `objects = models.Manager()` reste le manager complet et est déclaré en premier sur le
  modèle. L'administration Django et les opérations de restauration l'utilisent.
- Un second manager `active` filtre explicitement `deleted_at__isnull=True`. Les vues de l'API
  utilisent `Entity.active`.
- Le manager par défaut n'exclut donc jamais silencieusement des lignes dans l'administration,
  la restauration ou le parcours des relations.
- La restauration est réservée à l'administrateur. La suppression physique n'est pas exposée
  par l'API V1.

## 8. Séparation des faits mythologiques et des données de rendu

- **Statut :** validée
- **Décision :** `Entity` et ses profils ne portent que des faits mythologiques. Les données
  destinées à la génération de médias sont regroupées dans un module de rendu distinct, livré
  après la V1.

### Justification

Dans le corpus V2, `appearance.image_generation_prompt` et le bloc `rendering` cohabitent avec
les faits mythologiques au sein du même objet. Ce sont pourtant deux natures différentes : le
récit, les signes physiques ou les symboles d'une figure sont des faits documentés, alors qu'un
prompt est une instruction adressée à un moteur d'images, dépendante du modèle de génération, de
son style et de sa version. Les mélanger ferait entrer dans le schéma relationnel une donnée qui
change au rythme d'un outil externe, et obligerait à migrer les tables métier chaque fois que ce
moteur évolue.

### Conséquences

- Aucun champ `prompt` isolé n'est ajouté à `Entity` ni à l'un de ses profils, à aucun moment de
  la V1.
- `prompt_canon`, le prompt négatif, les variantes de prompt par style, la matrice de styles et
  les métadonnées d'images, de sons ou de vidéos seront intégrés dans le module de rendu déjà
  prévu par la roadmap, et non dans la phase 2.
- Tant que ces données ne sont pas importées, le corpus V2 reste leur source conservée.
  `afrimyth-api` ne deviendra leur source de vérité qu'après cet import, par symétrie avec la
  décision 5 sur les données mythologiques.
- La phase 2 doit néanmoins préparer les **entrées** dont ce futur moteur aura besoin, parce
  qu'elles sont elles-mêmes des faits mythologiques : l'ethnie — clé de la matrice de styles —
  ainsi que les signes physiques, les symboles et les objets de pouvoir, déjà portés par
  `Entity`.
- Cette décision ne déplace pas la phase de rendu dans la phase 2 : elle fixe seulement la
  frontière, afin qu'aucun incrément intermédiaire n'introduise par commodité un champ de
  prompt dans les tables métier.
