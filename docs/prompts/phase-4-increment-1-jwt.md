# Phase 4 — Incrément 1 : connexion JWT et identité

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis `AGENTS.md`,
`CLAUDE.md`, la phase 4 de la roadmap du dépôt voisin `django-portfolio`,
et les ADR 0002/0003. La phase 3 est terminée et publiée.

## Objectif

Un utilisateur existant doit pouvoir obtenir des jetons, renouveler son accès
et consulter sa propre identité via JWT. C'est un incrément d'authentification,
pas encore de permissions métier. Aucun groupe, inscription publique, CRUD
d'entités, workflow, modification du corpus ou nouveau modèle utilisateur.

Utilise Simple JWT et les vues standard lorsque possible, sans implémenter
de mécanisme de signature maison. Choisis une version compatible avec nos
versions Django/DRF, avec contrainte dans les dépendances d'exécution.
Référence officielle :
https://django-rest-framework-simplejwt.readthedocs.io/en/stable/getting_started.html.

Prompts, commentaires internes et rapports en français ; nouvelles docstrings,
schéma OpenAPI, messages publics et README en anglais. Ne traduis pas tous les
anciens fichiers au passage. Actualise la consigne devenue obsolète qui place
encore JWT hors périmètre dans `CLAUDE.md`.

## 1. Contrat HTTP

| Route | Entrée et résultat |
|---|---|
| `POST /api/auth/token/` | `username` et `password` ; réponse standard `access` et `refresh` |
| `POST /api/auth/token/refresh/` | `refresh` ; réponse standard avec un nouvel `access` |
| `GET /api/auth/me/` | JWT d'accès dans `Authorization: Bearer …` ; réponse limitée à `id` et `username` de l'utilisateur authentifié |

Les deux routes de jetons sont accessibles sans authentification préalable.
`me` exige un JWT valide : ne pas accepter à sa place une simple session
d'administration, ni un jeton de renouvellement. Une authentification réussie
ne donne ici aucun droit supplémentaire sur les entités.

Utilise `JWTAuthentication` avec utilisateur vérifié en base, pas sa variante
stateless. Conserve le refus des utilisateurs inactifs. Garde les réponses
standard de Simple JWT autant que possible : 200 en cas de succès, 400 pour
les champs requis manquants, 401 pour des identifiants/jetons refusés. Messages
anglais, sans distinguer compte inexistant et mauvais mot de passe.

## 2. Configuration et limites de sécurité

Ajoute JWT en premier dans les classes d'authentification DRF, en conservant
la session existante pour les vues qui l'utilisent ; limite explicitement
`me` à JWT. Maintiens `IsAuthenticated` comme permission par défaut et les
exceptions publiques explicites. Préserve l'ordre des imports corrigé en
phase 3 et adapte son test de non-régression aux nouveaux défauts attendus.

Pour ce premier incrément : accès de 5 minutes, renouvellement de 1 jour,
sans rotation, blacklist ou route de déconnexion ajoutées. Consigne ce choix
et sa limite : renouveler ne révoque pas le jeton précédent ; supprimer un
jeton de Postman ne le révoque pas côté serveur. Pas de promesse de déconnexion
ou révocation immédiate. Ne désactive aucun contrôle de signature, expiration
ou activité du compte pour simplifier les exercices.

Réutilise la configuration de signature standard de la bibliothèque ; aucun
secret en dur, affichage de `.env`, mot de passe ou jeton réel dans les traces,
exemples ou rapports. Aucun rôle métier ajouté aux claims JWT. Cette étape
n'est pas une certification de sécurité production : documente notamment
HTTPS nécessaire hors localhost et la protection anti-bruteforce à reprendre
avant exposition publique des routes de connexion, sans bâtir ce dispositif ici.

Les routes publiques existantes restent consultables **sans** jeton et ne
donnent pas accès aux brouillons après connexion. Avec un en-tête Bearer
invalide, DRF peut refuser l'authentification même sur une vue `AllowAny` :
ne contourne pas ce comportement et précise la différence dans le README.
L'administration Django continue à fonctionner par session.

## 3. Documentation et tests

Actualise OpenAPI et le README avec ces trois routes, leurs entrées, réponses
et erreurs. Swagger doit proposer l'authentification Bearer pour `me`, sans
l'exiger pour la consultation publique ou l'obtention des jetons. Les POST
de connexion ne sont pas un CRUD métier : les entités restent en lecture
seule. Corrige les descriptions globales et tests « toutes les opérations
sont GET », sans affaiblir les refus d'écriture sur les entités.
Préserve la version ReDoc fixée et les descriptions publiques anglaises.

Tests avec comptes synthétiques dans la base de test : connexion valide,
erreurs d'identifiants/champs manquants, compte inactif, identité correcte,
absence de jeton, jeton malformé, expiré et mauvaise signature ; renouvellement
valide/invalide/expiré, accès et refresh non interchangeables. Construire les
expirations dans les tests, sans attente réelle. Vérifie également qu'un
compte désactivé après émission ne peut plus utiliser son accès sur `me`.

Vérifie le contrat OpenAPI et l'absence de données utilisateur sensibles dans
`me` (pas d'email, mot de passe, permissions ou flags administratifs), les
défauts réellement appliqués aux vues, la consultation publique sans jeton,
le maintien de la visibilité et des refus d'écriture sur les entités.

Exécute `check`, `makemigrations --check --dry-run`, la validation spectacular
avec `--validate --fail-on-warn`, pytest, Ruff, le contrôle de format et mypy.
Après correction, relance les seuls contrôles affectés. Aucune migration
métier n'est attendue ; signale toute nécessité imprévue avant de l'élargir.

## 4. Préparer les exercices Postman et le rapport

Prépare un guide interne français court sous `docs/`, avec les chemins des
extraits de code à relire : configuration, routes, vue/serializer de `me`,
tests et annotations. Décris les essais Postman dans l'ordre : obtenir les
jetons, appeler `me` sans puis avec accès, essayer un jeton invalide,
renouveler et réutiliser le nouvel accès. Explique accès vs renouvellement
et authentification vs autorisation.

Ne crée pas de compte dans la base de développement sans demande. Indique
comment l'auteur pourra créer un compte fictif non staff via l'administration,
avec mot de passe choisi par lui. Aucun compte ou identifiant de connexion
committé. Les comptes déjà présents et toutes les fiches restent intacts.

Pour les exercices : URL via variable `base_url`, secrets dans Postman Local
Vault ou mécanisme local adapté, pas de synchronisation/publication de jetons.
Pas d'automatisation Postman complexe, d'installation de Postman, ni d'export
de collection nécessaire à ce stade. Le guide n'est pas une preuve d'essai
manuel : distingue ce qui a été exécuté de ce que nous ferons ensemble.

Crée `docs/reports/phase-4-increment-1-jwt.md` et sa copie identique dans le
scratchpad : `rapport-pour-codex-phase-4-increment-1-jwt.md`. Indique version
ajoutée, fichiers, contrat, choix et limites, résultats réels et nombre de
tests, éventuels blocages, préservation des données et état Git des deux
dépôts. Termine la réponse par le chemin absolu de la copie temporaire.

Ne coche pas la roadmap, ne committe ni ne pousse avant notre revue et les
exercices. Codex effectuera la clôture après validation, sans prompt Git séparé.
