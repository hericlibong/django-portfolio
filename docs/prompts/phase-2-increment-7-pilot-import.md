# Prompt Claude CLI — Phase 2, incrément 7 : consolidation et import pilote

Travaille dans `C:\Users\heric\dev\afrimyth-api`.
Pilotage : `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales, l'ADR 0002, l'ADR 0003 et la phase 2 de la roadmap.
Inspecte l'état Git ; préserve les changements préexistants.

Objectif : livrer un seul incrément couvrant les quatre tâches restantes de
phase 2 : contraintes/tests utiles, chargement des relations, commande d'import
pilote et confrontation aux données réelles. Ne reconstruis pas l'existant.

## 1. Source et périmètre

Le corpus réel est une liste JSON de 44 figures :
`C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json`.

Ce prompt autorise explicitement sa lecture et l'import des trois fiches
ci-dessous dans PostgreSQL. L'interdiction locale de recopier `_audit` ne
bloque pas cet usage demandé : ne copie cependant pas le dossier, son code,
ses images ou le corpus complet dans le dépôt API. Le JSON reste intact et
externe ; les tests committés doivent être autonomes, avec des données
synthétiques équivalentes. Ne fais aucun appel IA ou réseau.

Sélection du pilote, noms exacts vérifiés :

- `Oya` — `Divinity`, profil de culte rempli, domaines divergents et relations ;
- `Anansi` — `Hero`, profil avec titres, réalisations, objets et héritage ;
- `Mokele-mbembe` — `Creature`, profil rempli et origine `Congo Basin`.

N'importe pas les 41 autres entités. Préserve la fiche historique `shango`,
ses données et son absence de profil ; ne la supprime pas et ne la répare pas
dans cet incrément. Aucun propriétaire, source ou fait culturel inventé.

## 2. Commande simple, transactionnelle et relançable

Crée une commande Django, par exemple :
`python manage.py import_mythology_pilot --input "CHEMIN_JSON"`.

Le chemin est fourni explicitement : aucun chemin absolu local en dur dans
le code. La sélection reste limitée aux trois noms du pilote. Vérifie que
chacun existe une seule fois dans le JSON et a le type attendu avant d'écrire.
Donne des erreurs lisibles pour fichier/JSON invalide ou sélection incomplète.

La commande crée les fiches via `create_entity_with_profile`, dans une
transaction englobant aussi les référentiels, collections et relations du
pilote. Valide les modèles créés avec `full_clean()` avant enregistrement.
Une erreur bloquante annule tout nouvel apport de cet appel, référentiels
compris. Les limites ou ambiguïtés éditoriales signalées ne sont pas assimilées
automatiquement à des erreurs fatales.

Ajoute `--dry-run` : il effectue la validation et les essais d'écriture dans
une transaction finalement annulée, puis affiche un bilan. Pas de restauration
de sauvegarde ou de base temporaire artisanale pour cette simulation.

Une relance normale ne duplique pas et n'écrase pas silencieusement les fiches :
si nom/type/slug correspondent à une fiche déjà présente, signale-la comme
ignorée, sans modifier ses champs, profils ou collections. Un conflit
d'identité non concordant est une erreur explicite, sans suffixage arbitraire
de slug. N'introduis pas de mode `--force`, de fusion ou de mise à jour globale.
Une fiche ignorée ne doit pas conduire à modifier ses référentiels ou liens.

## 3. Transformation métier

Construis un mapping lisible des clés JSON vers les modèles existants. Quelques
fonctions explicites sont acceptables ; pas de framework d'import générique.

- Convertis `entity_type` et `identity.gender` vers leurs valeurs techniques
  anglaises. Génère les slugs de façon déterministe.
- Conserve les textes anglais et les listes du tronc commun : catégorie,
  rôle culturel, alignement, récit, caractéristiques, signes physiques,
  manifestations, symboles, objets de pouvoir et animaux symboliques.
- Reprends les seuls champs métier de `type_specific` dans le bon profil.
  Les données manquantes utilisent les valeurs par défaut existantes ; ne
  transforme pas une valeur absente en fait inventé. Signale les formes
  inattendues, les valeurs nulles et les écarts de longueur au lieu de tronquer.
- Les fiches créées sont `draft`, sans propriétaire ni marque de revue ou
  suppression. Aucun statut public automatique.
- Les domaines sont l'union dédupliquée de `attributes.domains` et des domaines
  spécifiques de divinité. Applique la correction validée `Ceteris` →
  `Cemeteries` et consigne les écarts d'Oya. Ne fusionne pas des synonymes
  discutables comme `Change` et `Transformation`.
- Réutilise/crée les pays, ethnies, aires et domaines par une identité stable.
  Pas de doublon principal/secondaire ; déduplique les collections. Une valeur
  vague telle que `Multi-ethnic` est signalée, pas créée comme ethnie réelle.

Prépare les quatre grandes aires `West Africa`, `Central Africa`, `East Africa`,
`Southern Africa` (`african_region`, sans parent), ainsi que `Diaspora`
(`diasporic`, sans parent). `Pan-African`, si nécessaire, est `transregional`.
`Congo Basin` est une aire `african_region` avec parent `Central Africa`.
Pour Mokele-mbembe, utilise `Congo Basin` comme aire principale précise :
`Central Africa` est accessible par son parent, sans duplication artificielle.
Ne déduis pas de pays de cette seule indication ; `primary_country` reste nul.

Pour les autres fiches, reprends le pays et l'ethnie connus et l'aire fournie.
Le premier pays réel d'une origine multiple est principal, les suivants
secondaires. Les associations secondaires à d'autres régions ou diasporas
ne doivent pas être inventées. Une table explicite de correspondances pour
les valeurs rencontrées suffit ; pas de géocodage ou dépendance supplémentaire.
Ne réécris pas un référentiel existant contradictoire : signale le conflit.

Les champs de rendu restent hors V1 : `rendering`, prompts, images, matrice
de styles. `origin.pantheon` est également exclu du modèle cible. Consigne ces
omissions avec leur raison, sans copier les longs prompts dans le rapport.
Le corpus du pilote n'a pas de sources documentaires : laisse `sources=[]`.
Ne crée pas de variantes culturelles inexistantes dans les données.

## 4. Relations en deux passes

Crée d'abord les fiches du pilote, puis leurs relations. Le sens reste :
`parents` → `parent` (la cible est le parent de la fiche), `conjoint` → `consort`.
Ne convertis pas automatiquement `descendants` en enfant direct : signale
les cas non vides pour arbitrage, avec conservation de l'information dans le
bilan si le sens n'est pas établi.

Résous uniquement les références exactes et non ambiguës vers les fiches
identifiées du pilote. Les autres noms restent dans `to_name`. N'importe pas
une quatrième fiche pour résoudre une cible. Ne rapproche pas automatiquement
les noms de toutes les fiches préexistantes, notamment la Shango historique.
Signale les cibles laissées non résolues, même si une fiche homonyme existe.
Ne fusionne pas `Yemaya` avec un autre nom, ne crée aucun lien inverse.
Valide chaque relation et empêche les doublons exacts.

## 5. Consolidation ciblée et tests

Commence par réutiliser les contraintes et tests déjà présents. Ajoute seulement
ce que le pilote révèle ou ce qui est nécessaire à l'import, notamment :

- transformation des trois types, profils et listes ; domaines réconciliés ;
- pays principal nul pour une aire géographique, hiérarchie des aires et
  absence de doublon principal/secondaire dans les données produites ;
- profils cohérents via le service ; cibles absentes conservées ; résolution
  en deux passes démontrée avec des données synthétiques ;
- relance sans doublon et préservation d'une fiche modifiée entre deux appels ;
- dry-run sans apport persistant ; erreur sur le profil ou un lien annulant
  les créations de l'appel ; conflit de fiche préexistante non écrasée ;
- API de détail des fiches importées et préchargement à sept requêtes,
  puis zéro requête à la sérialisation après chargement ;
- listes vides, caractère Unicode, valeur nulle et dépassement de longueur :
  comportements explicitement testés, sans prétendre que tous surviennent
  dans les trois fiches réelles.

Ne rajoute pas d'index redondant sur les FK. Un index ou une contrainte
supplémentaire doit répondre à un manque constaté. Si une limite de champ
bloque une donnée réelle valide, corrige-la de manière ciblée avec une
migration générée et explique les valeurs qui le justifient. Ne modifie pas
les décisions métier et ne multiplie pas les abstractions pour finir la phase.

## 6. Exécution réelle et livrables

Quand les tests pertinents passent, relève l'état initial de développement,
exécute le dry-run, l'import réel puis une seconde exécution. Cette consigne
autorise les trois créations réelles et leurs référentiels/liens uniquement.
Vérifie la préservation des fiches existantes et les réponses API. Ne publie
pas les fiches et ne réalise pas l'intégration carte ou frontend.

Documente en anglais dans le README la commande, le chemin externe, la
sélection pilote, la simulation et le comportement de relance sans écrasement.
Garde les autres documents internes en français. Ajuste sobrement la règle
`_audit` de `CLAUDE.md` pour autoriser la lecture de ce JSON externe lors des
imports explicitement demandés, tout en interdisant la copie du dossier ou
de son code dans le dépôt API.

Exécute sur la version finale : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`, `pytest`, `ruff check .`,
`ruff format --check .`, `mypy .`. Réutilise les résultats acquis ; après une
correction, relance les seuls contrôles affectés. Aucun secret dans les sorties.

Crée `docs/reports/phase-2-increment-7-pilot-import.md` dans le dépôt API,
avec une copie identique dans le scratchpad nommée
`rapport-pour-codex-phase-2-increment-7-pilot-import.md`.

Le rapport doit permettre de valider séparément les quatre tâches de roadmap :
contraintes/tests, requêtes, commande/trois imports, confrontation réelle.
Donne les commandes et résultats, nombre de cas exécutés, noms/slugs/types
créés ou ignorés, référentiels et liens, transformations/omissions, anomalies,
absence d'écrasement, résultats de relance et de dry-run, limites et état Git.
Référence le JSON source et son empreinte SHA-256 pour identifier l'entrée.

Si une tâche reste incomplète, dis-le : ne déclare pas la phase terminée par
anticipation. Les autres chemins d'écriture et changements de type restent
à encadrer en phase 4 ; la visibilité des cibles imbriquées reste en phase 3.
L'import complet des 44 fiches et sa généralisation restent en phase 5.

Ne coche pas la roadmap, ne committe et ne pousse pas avant notre revue.
Codex fera la clôture documentaire et Git après validation. Dans ta réponse,
indique le chemin absolu du rapport temporaire.
