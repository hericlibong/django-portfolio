# Phase 3 — Incrément 3 : OpenAPI, Swagger et ReDoc

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, l'ADR 0003 et la phase 3 de la roadmap du dépôt voisin
`django-portfolio`. Les incréments de visibilité et navigation sont validés :
documente leur fonctionnement réel, sans recommencer leur implémentation.

## Objectif et limites

Livrer un schéma OpenAPI et deux interfaces utilisant ce même schéma :
Swagger UI pour explorer et essayer l'API, ReDoc pour la lire.
Conserver les routes, formats de réponse, pagination, filtres, visibilité et
refus d'écriture actuels. Aucun modèle, migration, CRUD, JWT, nouveau filtre,
import ou changement de statut dans la base de développement.

Le prompt, les rapports et les explications internes sont en français.
Le schéma, les descriptions, exemples et README publics sont en anglais.
Toutes les nouvelles docstrings sont en anglais ; traduire les anciennes
uniquement lorsqu'elles alimentent cette documentation ou sont remplacées
dans le travail demandé. La traduction globale reste reportée.

## 1. Installation et routes documentaires

Ajoute `drf-spectacular` aux dépendances d'exécution avec une contrainte de
version compatible avec notre Django/DRF. Configure son générateur et expose :

- `/api/schema/` : schéma OpenAPI ;
- `/api/docs/` : Swagger UI ;
- `/api/redoc/` : ReDoc.

Ces trois routes doivent être consultables sans connexion, comme notre API
publique. Ne change pas les permissions par défaut du reste du projet pour
y parvenir. Pas de Swagger manuel parallèle ni de framework documentaire.
Si les interfaces chargent leurs ressources depuis un CDN, indique cette
dépendance dans le README ; pas de dispositif hors ligne sans besoin établi.

## 2. Documenter le contrat effectivement livré

Utilise l'introspection et les annotations ciblées de drf-spectacular.
N'écris pas une seconde définition exhaustive de tous les modèles.
Un petit module dédié aux annotations/exemples est acceptable si cela évite
d'encombrer les vues ; pas de découpage ou de couche générique spéculative.

Le schéma doit décrire fidèlement :

- `GET /api/health/` et sa réponse `{"status": "ok"}` ;
- la liste `GET /api/entities/` : enveloppe `count`, `next`, `previous`,
  `results`, avec cinq champs par élément et taille fixe de 20 ;
- le détail `GET /api/entities/{slug}/` : paramètre chemin **slug**, UUID
  dans `id`, profils et référentiels imbriqués, sources et relations ;
- les références et profils pouvant réellement valoir `null`, y compris
  `variant_of`, ainsi que les types des deux `SerializerMethodField`
  `variant_of` et `relations` ; ne pas modifier le JSON pour faciliter le schéma ;
- les paramètres **de liste uniquement** : `page`, `search`, `entity_type`,
  `country`, `cultural_area`, sans doublons dus à leur détection automatique ;
- les trois valeurs de type, les champs recherchés, la normalisation du code
  pays, les rattachements principaux/secondaires, les descendants d'une aire,
  la combinaison par ET et l'absence de doublons ;
- les réponses 400 de paramètres invalides et les 404 de page ou détail
  introuvable, avec les messages anglais réellement renvoyés ;
- l'accès anonyme : ne pas annoncer qu'une authentification est requise ;
- la visibilité publique et imbriquée, et le fait que les fiches du pilote
  restent en brouillon, donc absentes de cette consultation.

Ne présente aucune opération POST/PUT/PATCH/DELETE comme disponible.
Mentionne leur refus 405 dans la description/README, sans inventer des
opérations d'écriture dans le schéma. Ne publie aucun champ éditorial interne.
La génération doit fonctionner sur une base sans fiche publiée et ne doit
pas dépendre de paramètres de requête ni de données réelles pour introspecter
les vues. Ne masque pas globalement les avertissements du générateur.

## 3. Exemples, tests et vérification

Ajoute quelques exemples synthétiques clairement identifiés : liste paginée,
détail avec profil, collection vide et erreurs principales. Ils ne doivent
pas être lus dans la base de développement ni présentés comme des faits
mythologiques validés. Vérifie leur cohérence avec les serializers ; pas de
gros corpus de démonstration ajouté au dépôt.

Tests proportionnés : schéma accessible anonymement et analysable, routes
et méthodes correctes, paramètres présents sur la liste et absents du détail,
forme de pagination, références imbriquées correctement décrites, absence
de champs internes ; pages Swagger/ReDoc accessibles et branchées sur notre
schéma. Un HTTP 200 de page HTML ne prouve pas le chargement JavaScript :
distingue les contrôles automatisés de ce qui restera à vérifier au navigateur.

Conserve la suite de tests fonctionnels existante, sans créer une seconde
suite redondante pour la même visibilité. Exécute sur la version finale :

- `python manage.py check` ;
- `python manage.py makemigrations --check --dry-run` ;
- `python manage.py spectacular --validate --fail-on-warn`, en dirigeant
  la sortie du schéma vers un fichier temporaire si nécessaire ;
- `pytest`, `ruff check .`, `ruff format --check .`, `mypy .`.

Corrige les avertissements réels ; après correction, relance les contrôles
affectés seulement. Pas de secret affiché, pas de schéma généré committé par
défaut : la route et la commande le produisent à partir du code.

## 4. Documentation et compte rendu

Actualise le README anglais : liens des trois routes, commande de validation,
usage de Swagger, accès public, résultat vide attendu sans publication et
limites actuelles. La phase 3 reste en attente de validation finale par
l'auteur : ne la déclare pas achevée avant notre revue et essai navigateur.

Écris `docs/reports/phase-3-increment-3-openapi.md`, avec copie identique dans
le scratchpad sous `rapport-pour-codex-phase-3-increment-3-openapi.md`.
Indique les fichiers touchés, la version ajoutée, les annotations nécessaires,
les résultats exacts et le nombre de tests, les éventuelles limites et l'état
Git des deux dépôts. Ajoute une courte section pédagogique : rôle de la
configuration, du routage, des annotations et des exemples, avec les chemins
des principaux extraits à examiner ensemble. Pas de rapport par commande.
Termine ta réponse par le chemin absolu de la copie temporaire.

Ne coche pas la roadmap, ne committe ni ne pousse avant notre revue.
Codex prendra en charge la clôture après validation, sans prompt Git séparé.
