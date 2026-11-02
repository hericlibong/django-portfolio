# Roadmap — Projet 1 : API REST African Mythology

## Objectif

Construire une API Django REST autonome qui devienne la source de données destinée à
`african_mythology_V2`. La première version doit démontrer un CRUD authentifié, des permissions
au niveau objet, un workflow de modération, une documentation OpenAPI et un import reproductible
du corpus V2.

L'intégration effective avec le frontend V2 est hors de la V1 : elle viendra après la livraison
de l'API autonome.

## Règles de conduite

- Avancer par incréments exécutables, pas par longues phases de spécification.
- Ne construire que ce qui sert le prochain livrable.
- Une phase est terminée lorsque ses critères sont remplis et ses tests passent ; elle n'est pas
  soumise à des cycles de revalidation sans fait nouveau.
- Reporter explicitement les améliorations non indispensables au lieu de les intégrer « au cas
  où ».
- Mettre à jour cette roadmap au fil du travail en cochant les tâches réellement terminées.

---

## Phase 0 — Décisions structurantes express

Objectif : prendre uniquement les décisions qui empêchent d'écrire les premières migrations.

**Estimation : 0,5 à 1 jour.**

- [x] Choisir le nom du dépôt autonome : `hericlibong/afrimyth-api`, distinct de
  `AfricanPantheonAPI` et `AfrMyth_API`.
- [x] Trancher la gestion des ethnies : ethnie principale facultative + ethnies secondaires.
- [x] Trancher la gestion des pays : pays principal facultatif + pays secondaires.
- [x] Trancher les aires régionales, transafricaines et diasporiques, ainsi que les variantes
  culturelles d'une même figure : `CulturalArea` + `variant_of`.
- [x] Choisir l'emplacement unique des domaines : `Entity.domains`, avec réconciliation à
  l'import.
- [x] Confirmer que l'API et PostgreSQL deviennent la source de vérité après l'import initial
  du corpus V2.
- [x] Confirmer la stratégie `Entity` + profils `OneToOne` explicites.
- [x] Valider le workflow et la matrice de permissions déjà proposés.
- [x] Enregistrer les décisions retenues dans l'ADR 0002.

**Livrable attendu :** décisions bloquantes consignées, schéma initial suffisamment stable
pour écrire les modèles sans chercher à figer tout le futur contrat REST.

---

## Phase 1 — Premier incrément exécutable

Objectif : obtenir rapidement une application qui démarre et une première tranche métier qui
fonctionne de bout en bout.

**Estimation : 1 à 2 jours.**

- [x] Créer le dépôt autonome avec le nom validé en phase 0.
- [x] Initialiser Django, Django REST Framework et la configuration PostgreSQL.
- [x] Configurer pytest et pytest-django avant les premiers tests.
- [x] Configurer Ruff et mypy dès l'initialisation, avec des règles utiles mais compatibles
  avec un démarrage rapide.
- [x] Créer `CLAUDE.md` et `AGENTS.md` avec les règles communes du projet et seulement les
  différences propres à chaque agent.
- [x] Créer les apps minimales `accounts` et `mythology`.
- [x] Déclarer le modèle utilisateur personnalisé dès la première migration.
- [x] Créer `Entity` et `DivinityProfile` avec leur relation `OneToOne`.
- [x] Ajouter les migrations et l'administration Django minimale.
- [x] Exposer un endpoint de santé.
- [x] Exposer une première lecture liste/détail des entités.
- [x] Ajouter les premiers tests de modèles et d'API.

**Livrable attendu :** dépôt installable, base PostgreSQL migrable, administration accessible,
endpoint de santé et première ressource `Entity` consultable avec tests, Ruff et mypy au vert.

### Avancement — incrément 1 validé le 13 octobre 2026

Le socle technique de `afrimyth-api` est opérationnel : Python 3.12, Django 5.2 LTS et Django
REST Framework, PostgreSQL 16 avec base et rôle applicatifs en place et migrations appliquées,
modèle utilisateur personnalisé `accounts.User` déclaré dès la première migration, endpoint
public de santé, premiers tests et outillage qualité (pytest, Ruff, mypy) au vert.

Rapport détaillé : `docs/reports/phase-1-increment-1.md` du dépôt `afrimyth-api`.

### Avancement — incrément 2 validé le 13 octobre 2026

La première tranche métier fonctionne de bout en bout : `Entity` — UUID en clé primaire, slug
unique, contrainte d'unicité `(name, entity_type)` — et `DivinityProfile` relié en `OneToOne`
avec suppression en cascade, migration générée par Django et appliquée sur PostgreSQL 16,
administration Django minimale, et API publique en lecture seule exposant
`GET /api/entities/` et `GET /api/entities/{slug}/`, le profil étant imbriqué dans le détail et
chargé en une seule requête.

La vérification manuelle en navigateur est concluante : administration accessible, création
d'une `Entity` et de son `DivinityProfile`, liste et détail visibles, profil correctement
imbriqué. 24 tests passent, Ruff et mypy sont au vert.

Rapport détaillé : `docs/reports/phase-1-increment-2.md` du dépôt `afrimyth-api`.

### Phase 1 terminée et validée le 13 octobre 2026

Toutes les tâches de la phase 1 sont remplies et son livrable est atteint. Les prochains
travaux relèvent de la phase 2 : profils `Hero` et `Creature`, référentiels, relations entre
entités et pilote d'import.

Deux points restent provisoires par construction et seront traités dans les phases prévues :
l'API est ouverte en `AllowAny` et strictement en lecture seule — authentification, rôles,
permissions objet, CRUD et modération arrivent en phase 4 — et aucun statut éditorial n'existe
encore, donc aucune entité n'est filtrée à la lecture.

---

## Phase 2 — Modèle métier complet de la V1

Objectif : compléter le schéma nécessaire au corpus sans ajouter les fonctions reportées.

**Estimation : 3 à 5 jours, import pilote inclus.**

L'audit a parcouru les 44 entités et a déjà identifié les valeurs composites, les relations
non résolues et les divergences de domaines. Cela suffit pour proposer un schéma initial, mais
pas pour garantir qu'aucune contrainte ou longueur de champ ne devra être corrigée. Le pilote
est donc volontairement placé dans cette phase, avant de construire toute l'API autour du
modèle.

- [x] Ajouter `HeroProfile` et `CreatureProfile`.
- [x] Ajouter les référentiels validés : régions culturelles, pays, ethnies et domaines.
- [x] Ajouter les relations entre entités, y compris les relations dont la cible n'existe pas
  encore dans le corpus.
- [x] Ajouter les sources documentaires.
- [x] Ajouter à `Entity` la propriété, le statut et les champs de revue.
- [x] Imposer la cohérence entre `entity_type` et le profil associé via le service de création.
  La garantie couvre les appels au service ; les autres chemins d'écriture et changements
  de type devront être encadrés en phase 4.
- [x] Ajouter les contraintes, index et tests de validation utiles.
- [x] Vérifier les requêtes avec `select_related` et `prefetch_related` sans optimisation
  prématurée supplémentaire.
- [x] Écrire le squelette de la commande d'import et charger une entité réelle de chaque type.
  Préparer les quatre régions continentales du périmètre, les sous-régions et la diaspora
  selon la précision de la section 3 de l'ADR 0002 ; ne pas confondre `kind` et niveau
  hiérarchique.
- [x] Confronter ce pilote aux contraintes, longueurs, valeurs nulles, caractères spéciaux et
  relations vers des cibles absentes ; corriger le modèle avant de le considérer stable.

### Frontière avec les données de rendu

Cette phase complète les faits mythologiques et **n'introduit aucune donnée de génération de
médias**. Aucun champ `prompt` n'est ajouté à `Entity` ni à ses profils. `prompt_canon`, les
variantes de prompt, la matrice de styles et les métadonnées d'images rejoindront le module de
rendu déjà prévu — première phase de la future roadmap d'intégration V2, hors V1 — et non la
phase 2.

Tant que ces données ne sont pas importées, le corpus V2 reste leur source conservée ; l'API ne
deviendra leur source de vérité qu'après cet import.

La phase 2 prépare en revanche les entrées dont ce futur moteur aura besoin, parce qu'elles sont
elles-mêmes des faits mythologiques : l'ethnie, qui sert de clé à la matrice de styles, ainsi que
les signes physiques, les symboles et les objets de pouvoir.

Décision consignée dans [`ADR 0002`](adr/0002-decisions-structurantes-phase-0.md), section 8.

### Avancement — incrément 1 de la phase 2, le 15 octobre 2026

`HeroProfile` et `CreatureProfile` sont livrés, reliés à `Entity` par un `OneToOneField` en
cascade, avec validation du type dans `clean()`, administration, serializers de lecture et
imbrication dans le détail d'une entité. Le détail expose désormais les trois clés de profil,
chargées en une seule requête, celle du type portant l'objet et les deux autres valant `null`.
39 tests passent, Ruff et mypy sont au vert.

Rapport détaillé : `docs/reports/phase-2-increment-1.md` du dépôt `afrimyth-api`.

### Avancement — incrément 2 de la phase 2 validé en revue

Les quatre référentiels `CulturalArea`, `Country`, `Ethnicity` et `Domain` sont implémentés,
administrables et associés à `Entity`. Le détail API expose les trois ancrages facultatifs
et les quatre collections ; la liste reste compacte. La migration 0003 a été appliquée.
Les corrections légères de revue ont été intégrées : normalisation et validation des codes
pays, test de sérialisation sans requête après préchargement et clarification du budget
de cinq requêtes sur le détail. Les 73 tests, Ruff et mypy passent ; les contrôles Django
ne détectent ni problème ni migration supplémentaire.

La distinction entre grandes régions, sous-régions et diaspora est consignée dans l'ADR 0002,
section 3. Les données de référence seront créées à l'import pilote ; le regroupement pour
la carte sera réalisé lors de l'intégration V2. Le corpus n'est pas encore importé.

Rapport : `docs/reports/phase-2-increment-2-referentiels.md` du dépôt `afrimyth-api`.
Clôture documentaire préparée ; commits en attente d'exécution par Claude CLI.

### Avancement — incrément 3 validé le 18 octobre 2026

`Entity.variant_of` relie les variantes culturelles autonomes à leur référence, avec
protection contre la suppression physique et l'auto-référence. `EntityRelation` porte les
liens de parenté, de conjoint, d'alliance et d'opposition, vers une fiche existante ou un
nom cité. Plusieurs conjoints distincts sont autorisés ; les doublons exacts sont refusés.
Le détail API expose la référence synthétique et les relations sortantes, sans récursion.
La migration 0004 est appliquée ; Claude a validé 110 cas de test, Ruff et mypy.
Le détail coûte six requêtes et la sérialisation préchargée n'en déclenche aucune.

La revue pédagogique est terminée. Les limites connues restent documentées : absence de
résolution automatique par nom et de relations inverses automatiques, cycles indirects
non détectés, suppression logique à venir. Les tâches générales de contraintes et de
vérification des requêtes restent ouvertes pour les prochains modèles.

Rapport : `docs/reports/phase-2-increment-3-relations.md` du dépôt `afrimyth-api`.
Prochaine tâche : ajout des sources documentaires.

### Avancement — incrément 4 validé le 18 octobre 2026

`Source` rattache plusieurs références documentaires à une fiche, avec les six catégories
validées, une URL et une date de consultation facultatives. Administration, migration 0005
et exposition en lecture dans le détail API sont livrées. La liste reste compacte.
Claude a validé 140 cas de test et les contrôles Django, Ruff et mypy ; le détail coûte
sept requêtes, avec zéro requête pendant la sérialisation préchargée.

La revue pédagogique et l'exercice ORM sont validés : création, validation, consultation
et suppression d'une référence fictive. L'auteur a confirmé la suppression de la source
de démonstration et la liste vide des sources de Shango. La fiabilité documentaire reste
une appréciation éditoriale, distincte des validations de saisie.

Rapport : `docs/reports/phase-2-increment-4-sources.md` du dépôt `afrimyth-api`.
Prochaine tâche : propriété, statut éditorial et champs de revue de `Entity`.

### Avancement — incrément 5 validé le 20 octobre 2026

`Entity` porte désormais le propriétaire, les quatre statuts éditoriaux, les dates et acteurs
de revue ainsi que les champs de suppression logique. Le manager `objects` reste complet ;
`active` filtre uniquement `deleted_at`. La vue utilise `active`, tandis que l'administration
conserve l'accès à toutes les fiches. Aucun champ interne n'est ajouté aux réponses publiques.

La migration 0006 est appliquée. Claude a validé 171 cas de test, les contrôles Django, Ruff
et mypy. Le budget du détail reste de sept requêtes. La revue technique et pédagogique est
validée ; aucun changement fonctionnel n'a suivi ces contrôles. Les transitions, permissions,
attributions automatiques et opérations de suppression/restauration restent en phase 4.
La visibilité des cibles imbriquées doit être traitée en phase 3, comme indiqué ci-dessous.

Rapport : `docs/reports/phase-2-increment-5-editorial-fields.md` du dépôt `afrimyth-api`.
Prochaine tâche : cohérence entre le type de l'entité et son profil métier.

### Avancement — incrément 6 validé le 20 octobre 2026

`create_entity_with_profile` crée l'entité et le seul profil correspondant dans une même
transaction, avec validation des données et annulation des créations en cas d'échec.
Aucun nouveau modèle ni migration. Claude a validé 205 cas de test et les contrôles qualité.
La revue technique et pédagogique est terminée, avec exercice ORM et constat du refus d'un
doublon. Aucun comportement n'a changé après ces contrôles.

Cette garantie ne couvre ni l'ORM direct ni les changements de type ultérieurs. L'ancienne
fiche de démonstration Shango, sans profil lors de l'inventaire, n'a pas été réparée par cet
incrément. Le service sera utilisé par l'import pilote, puis le CRUD en phase 4.

Rapport : `docs/reports/phase-2-increment-6-entity-profile-consistency.md` du dépôt `afrimyth-api`.
Prochaine étape : préparer l'import pilote de trois entités réelles et confronter leurs
données aux contraintes et validations du modèle ; les vérifications transverses restantes
de phase 2 seront consolidées à cette occasion.

### Clôture de la phase 2 — incrément 7 validé le 21 octobre 2026

Le pilote a importé Oya (divinité), Anansi (héros) et Mokele-mbembe (créature), avec leurs
profils, référentiels et relations. La simulation est annulée et la relance ne duplique
ni n'écrase les fiches. Shango, fiche historique hors pilote, a été préservée.
Les corrections de revue couvrent les cibles du pilote déjà présentes lors d'un import
partiel et le signalement d'un parent inattendu sur une région racine.

Les contraintes existantes conviennent aux trois fiches : aucun nouvel index, changement
de modèle ou migration n'est justifié par leurs données. Les cas synthétiques couvrent
notamment Unicode, valeurs nulles, dépassements de longueur et résolution en deux passes.
Codex a validé 257 cas de test, Ruff, le formatage, mypy et les contrôles Django. Le détail
reste à sept requêtes ; sa sérialisation préchargée n'en déclenche aucune.

L'auteur a vérifié les trois réponses API en HTTP 200 : domaines réconciliés et culte d'Oya,
profil de héros d'Anansi, profil de créature de Mokele-mbembe, pays nul et aire Congo Basin.
La phase 2 est terminée dans son périmètre. Les limites restent explicites : Shango sans
profil n'est pas réparée ; les liens d'Oya sont non résolus ; la visibilité publique et les
autres chemins d'écriture restent à encadrer en phases 3 et 4. L'import complet reste en
phase 5. La normalisation des libellés publics anglais est bien prévue dans la V1, en phase 3.

Rapport : `docs/reports/phase-2-increment-7-pilot-import.md` du dépôt `afrimyth-api`.
Prochaine phase : API publique de consultation.

**Livrable attendu :** modèle relationnel complet de la V1, migrations propres et suite de
tests démontrant les principales contraintes métier, validé par le passage en base de trois
entités réelles représentatives du corpus.

---

## Phase 3 — API publique de consultation

Objectif : rendre les données exploitables par un client avant d'ajouter toute la gestion des
contributions.

**Estimation : 2 à 3 jours.**

### Découpage d'implémentation validé

1. **Visibilité et contrat de lecture** : fiches publiées non supprimées,
   protection des références imbriquées, stabilité des réponses et libellés publics anglais.
2. **Navigation** : pagination, recherche et filtres par type, pays et aire culturelle,
   avec portée explicite des rattachements secondaires et sous-régions.
3. **Documentation et validation finale** : OpenAPI, Swagger/ReDoc, exemples et erreurs,
   puis consolidation des tests d'intégration du contrat obtenu.

L'API reste en lecture seule. Les brouillons du pilote attendent leur revue éditoriale ;
ils ne sont pas publiés automatiquement pour maintenir une démonstration. Les tests de
lecture publique utilisent des fiches explicitement publiées dans leur base dédiée.

- [x] Normaliser en anglais, avant la stabilisation de l'API publique, tous les libellés, aides
  et messages exposés aux clients : `verbose_name`, libellés des classes de choix, `help_text`
  et messages d'erreur de validation. Langue canonique fixée par
  [`ADR 0003`](adr/0003-langue-publique-api.md).
- [x] Stabiliser les serializers de liste et de détail.
- [x] Exposer les routes liste et détail avec des UUID et des slugs stables.
  L'UUID est retourné dans `id` ; la route de détail utilise le slug, sans seconde route UUID.
- [x] Ajouter pagination, recherche et filtres réellement utiles.
  Besoins relevés après le pilote : type d'entité, pays principaux/secondaires et aire
  culturelle (avec portée des sous-régions explicitée). Voir les exemples et questions
  éditoriales dans [la note de revue du pilote](notes/revue-editoriale-corpus-pilote.md).
- [x] Ne rendre publiquement visibles que les entités `published`.
- [x] Appliquer également les règles de visibilité aux cibles imbriquées (`variant_of`
  et cibles des relations) : une fiche supprimée ou non publiée ne doit pas être divulguée
  par le détail d'une autre fiche. Le manager `active` ne filtre que les objets racines.
- [x] Centraliser la sélection des objets visibles dans des fonctions réutilisables.
  La phase 4 conservera la forme des réponses ; si elle expose des références privées,
  elle devra aussi adapter explicitement les gardes publiques des serializers.
- [x] Produire le schéma OpenAPI avec drf-spectacular.
- [x] Ajouter des exemples de réponses et documenter les erreurs principales.
- [x] Tester les requêtes, filtres, données imbriquées et règles de visibilité.

### Avancement — incrément 1 validé le 21 octobre 2026

La consultation publique ne montre que les fiches publiées et non supprimées,
y compris pour un administrateur connecté. Les références imbriquées masquées
ne divulguent pas de fiche privée. Les libellés et messages publics sont en anglais.
Claude a validé 286 cas de test et les contrôles qualité ; la revue de Codex
n'a apporté que des précisions documentaires, sans changement fonctionnel.

L'auteur a vérifié la liste publique vide, le 404 d'Oya encore en brouillon
et sa présence dans l'administration, puis validé l'explication pédagogique.
Les brouillons du pilote n'ont pas été publiés pour cette démonstration.

Rapport : `docs/reports/phase-3-increment-1-public-read-contract.md` dans `afrimyth-api`.
Prochain incrément : pagination, recherche et filtres combinables sur la collection publique.
La tâche transverse des tests reste ouverte jusqu'à la consolidation de la phase.

### Avancement — incrément 2 validé après revue et exercices pratiques

La collection publique est paginée par 20 fiches compactes, recherchable et
filtrable par type, pays principal/secondaire et aire culturelle avec ses
descendants. Les paramètres se combinent sans doublons. Les filtres de liste
n'affectent pas le détail. Aucun modèle ni migration n'a été ajouté.

Claude a validé 346 cas de test et les contrôles qualité. Codex a renforcé
les tests anti-doublons et de combinaison, puis validé les 60 cas de navigation,
Ruff, le formatage et mypy sur le fichier modifié ; le code applicatif n'a
pas changé pendant cette revue.

L'auteur a validé les explications du code : pagination, recherche, garde du
détail, filtres `Q`/`distinct()` et sélection des sous-régions. Les réponses
vides et erreurs attendues ont été vérifiées, puis deux fiches fictives
publiées ont permis de vérifier recherche et filtre par type. L'auteur
confirme l'exercice et son nettoyage ; les brouillons du corpus sont préservés.
La pagination multi-pages, les filtres pays et la hiérarchie restent couverts
par les tests automatiques, sans démonstration manuelle supplémentaire.

Rapport : `docs/reports/phase-3-increment-2-navigation.md` dans `afrimyth-api`.
Prochain incrément : OpenAPI, Swagger/ReDoc, exemples et validation finale.
La phase 3 n'est pas encore terminée.

### Clôture de la phase 3 — incrément 3 validé après essais Swagger/ReDoc

OpenAPI décrit la liste paginée, le détail par slug et la santé, leurs
paramètres, structures imbriquées, valeurs nulles, exemples synthétiques et
erreurs. Swagger UI et ReDoc consomment le même schéma et sont accessibles
sans connexion. Les données et routes métier restent en lecture seule.

La revue a confirmé et testé le correctif d'un import DRF prématuré qui
empêchait la lecture des réglages. L'essai navigateur a également révélé
une incompatibilité de ReDoc `@latest` avec le gabarit : la version 2.5.3
est maintenant fixée et son rendu a été vérifié.

Claude a exécuté 398 tests et les contrôles globaux ; Codex a ajouté deux
tests de régression et exécuté les contrôles ciblés concernés, dont les
53 tests OpenAPI. Le total attendu devient 400, sans nouvelle exécution
annoncée de la suite complète. Les résultats acquis sont réutilisés.

L'auteur a validé la revue pédagogique (configuration, routes, annotations,
exemples), le rendu Swagger et ses requêtes réelles : liste vide en 200,
code pays invalide en 400 puis valide en 200, 404 identiques pour Oya en
brouillon et un slug inconnu. Après correction, il a confirmé le chargement
ReDoc et les paramètres, profils et références nullables documentés.
Aucun brouillon du corpus n'a été publié pendant ces exercices.

Rapport : `docs/reports/phase-3-increment-3-openapi.md` dans `afrimyth-api`.
La phase 3 est terminée. Prochaine phase : authentification, CRUD et modération.
La documentation et les descriptions actuellement « read-only » devront
évoluer avec les opérations réellement livrées en phase 4, avec leurs tests.

**Livrable attendu :** API de consultation documentée dans Swagger/ReDoc, utilisable sans
authentification pour les entités publiées et couverte par des tests d'intégration.

**Reprise prévue en phase 4 : 0,5 à 1 jour**, incluse dans l'estimation de cette phase. Elle
porte sur les classes de permission, l'élargissement conditionnel du queryset aux objets
privés et les tests associés. La forme des serializers reste stable ; une politique privée
des références imbriquées nécessitera d'adapter leurs gardes publiques. Pendant la phase 3,
les statuts sont posés par les fixtures, l'administration et
l'import pilote ; les transitions HTTP arrivent en phase 4.

---

## Phase 4 — Authentification, CRUD et modération

**Terminée le 1er novembre 2026**, après revue, tests et exercices des
incréments 1 à 7, puis consolidation de l'incrément 8. Limites du back-office
acceptées et reportées ; aucun nouvel exercice requis pour la clôture.

Objectif : réaliser le cœur démonstratif du projet portfolio.

**Estimation : 3 à 5 jours, dont 0,5 à 1 jour de reprise de la phase 3.**

### Découpage validé — incréments courts et pédagogiques

L'estimation reste indicative : cette phase mérite du temps pour comprendre
et éprouver les permissions. Aucun code de phase 4 n'est lancé par ce plan.
La matrice et les transitions de l'ADR 0002 restent la référence ; chaque
prompt précisera le contrat nécessaire à son incrément, sans nouvelle
architecture générique de rôles ou de workflow.

| Incrément | Travail et livrable vérifiable | Exercice pratique prévu |
|---|---|---|
| 1 — Authentification JWT | Obtenir un jeton d'accès, le renouveler et vérifier son identité sur une petite route protégée. Pas d'inscription publique ni d'écriture métier. | Dans Postman : connexion, requête avec/sans jeton, jeton invalide, renouvellement. |
| 2 — Rôles et lecture privée | Créer les trois groupes par migration ; appliquer rôle et propriété à la consultation privée. Conserver la consultation publique et ses protections. | Avec deux contributeurs et un relecteur : voir son brouillon, ne pas voir celui de l'autre, vérifier la vue du relecteur. |
| 3 — Création | Créer une entité et son profil de façon atomique, attribuer le propriétaire côté serveur et encadrer les champs éditoriaux. Valider les rattachements et collections pris en charge par le contrat d'écriture. | POST autorisé avec jeton ; vérifier le propriétaire et le brouillon, puis une création invalide sans donnée partielle. |
| 4 — Modification | Modifier les champs autorisés selon la matrice, préserver la cohérence type/profil et les relations ; interdire le détournement du propriétaire ou du statut par écriture ordinaire. | PATCH de sa fiche, tentative sur celle d'un autre et tentative de modifier directement le statut. |
| 5 — Soumission et reprise | Actions explicites `draft → submitted`, retrait `submitted → draft`, reprise `rejected → draft`. Droits et dates cohérents ; pas de moteur de workflow. | Soumettre, constater le blocage de modification du contributeur, retirer puis modifier à nouveau. |
| 6 — Modération | Actions `submitted → published`, `submitted → rejected` et réouverture `published → draft` réservée à l'administrateur ; renseigner la revue. | Un relecteur publie ou rejette ; vérifier la visibilité publique ; refus d'une publication par le contributeur. |
| 7 — Suppression et restauration | Suppression logique selon la matrice, avec acteur/date ; protection d'une référence ayant des variantes actives ; restauration réservée à l'administrateur. Aucune suppression physique exposée. | Supprimer une fiche de démonstration autorisée, constater sa disparition puis sa restauration ; tester un refus. |
| 8 — Consolidation | Compléter les croisements de la matrice, vérifier les codes 401/403/404 et les chemins d'écriture de l'administration concernés ; clôturer la documentation et les essais de bout en bout. | Parcours complet avec comptes fictifs : création, soumission, revue, publication, suppression/restauration. |

Chaque incrément comprend ses tests, une revue de petits extraits de code et
un exercice. La documentation OpenAPI/README évolue dès qu'une opération est
livrée : elle n'attend pas l'incrément 8. Les droits ne seront pas laissés
ouverts temporairement en attendant une « phase de sécurité » finale.

La lecture privée nécessite d'expliciter le rendu des références imbriquées,
pas seulement d'élargir le queryset public. Les règles des utilisateurs sans
groupe, des groupes multiples et du superutilisateur devront être explicites
dans le prompt de l'incrément 2, sans confondre `is_staff` et rôle métier.
La modification du type et les écritures de profils, sources et relations
seront précisées dans les prompts 3/4 avant d'ouvrir ces chemins d'écriture ;
ne pas créer automatiquement un CRUD complet de tous les référentiels.

### Postman — support pédagogique dès l'incrément 1

Postman sert à exécuter et conserver les requêtes, pas à remplacer les tests
pytest ni à porter les permissions. Utiliser des comptes et fiches fictifs,
une adresse locale et des variables pour l'URL et les identifiants de fiches.
Conserver mots de passe et jetons dans un espace local adapté (Postman Vault),
jamais dans Git, les rapports ou un export de collection. Tout export partagé
doit être nettoyé. Référence :
[Postman Vault](https://learning.postman.com/docs/use/postman-vault/use-vault-secrets).

La collection d'exercices se construit progressivement, sans infrastructure
de tests supplémentaire ni comptes de démonstration avec mots de passe
committés. Les deux premiers incréments pourront utiliser des comptes créés
explicitement dans l'administration, sans ajouter une API d'inscription.

- [x] Ajouter l'authentification JWT avec renouvellement de jeton.
- [x] Créer par migration les groupes `contributor`, `reviewer` et `administrator`.
- [x] Attribuer automatiquement `created_by` à la création.
- [x] Implémenter la création selon la matrice de permissions.
- [x] Implémenter la modification selon la matrice de permissions.
- [x] Implémenter la suppression selon la matrice de permissions.
- [x] Implémenter les actions explicites de soumission, publication, rejet et réouverture.
- [x] Interdire la modification directe du statut par un `PATCH` ordinaire.
- [x] Ajouter les tests croisés rôle × propriétaire × statut × action.
- [x] Vérifier que les réponses refusées utilisent correctement `401`, `403` ou `404`.

### Avancement — incrément 1 validé le 24 octobre 2026

Simple JWT fournit l'obtention et le renouvellement des jetons ; la route
`GET /api/auth/me/` expose uniquement l'identité de l'appelant et exige un
accès JWT. Aucun rôle, accès aux brouillons ou CRUD du corpus n'est ajouté.
Les réglages, routes et erreurs sont documentés dans OpenAPI et le README.

La revue a corrigé une réponse 500 sur renouvellement après suppression du
compte, remplacée par un refus 401 ciblé sans réécriture des contrôles JWT.
Codex a exécuté 463 tests et les contrôles qualité, puis les trois tests
affectés par les simplifications finales. Ces résultats sont réutilisés.

L'auteur a confirmé le remplacement local de la clé de signature, créé un
compte ordinaire de démonstration et préparé sa collection Postman et le
Local Vault. Il a obtenu les deux jetons, observé le refus 401 d'un accès
expiré, renouvelé cet accès puis vérifié `me` en 200 avec `id` et `username`.
Les autres cas négatifs sont couverts automatiquement, sans les présenter
comme des essais Postman effectués.

L'accès reste valable cinq minutes et le refresh un jour. Le maintien de
cinq minutes pour apprendre le mécanisme est validé ; une éventuelle durée
plus longue limitée au développement sera consignée lorsqu'elle sera décidée,
sans modifier implicitement un futur réglage de production.

Rapport : `docs/reports/phase-4-increment-1-jwt.md` dans `afrimyth-api`.
Prochain incrément : groupes Django, permissions et lecture privée.
La phase 4 n'est pas terminée.

### Avancement — incrément 2 validé le 27 octobre 2026

Les trois groupes sont créés par migration sans affectation automatique.
Les routes privées `/api/manage/entities/` et leur détail exigent un JWT et
un rôle métier, puis filtrent les fiches selon rôle et propriété. Le contrat
public est inchangé ; les références imbriquées suivent les droits de lecture.
L'ADR 0004 précise les conventions sans groupe, superutilisateur et cumul.

La revue technique de Codex a exécuté 169 tests ciblés et validé OpenAPI ; la
suite complète de 573 cas est le résultat de Claude. Une petite amélioration
demandée pendant les exercices affiche les groupes dans la liste admin des
utilisateurs, avec préchargement et quatre tests supplémentaires passants.

Essais Postman confirmés par l'auteur : refus 403 de `essai-postman` sans
groupe ; contributeur A limité à Anansi après attribution manuelle ; détail
Anansi 200 et Oya 404 ; relecteur `demo-selecteur` lisant Oya et les quatre
fiches ; collection publique vide car toutes restent `draft` ; POST privé
refusé en 405 avec un jeton valide. Les 401 dus à l'expiration et au mauvais
type de jeton ont été distingués du contrôle des permissions.

Les quatre fiches sont celles du corpus pilote, pas les fiches fictives du
guide. Anansi appartient désormais à `demo-contrib-a` ; Oya, Mokele-mbembe
et Shango restent sans propriétaire. Ne pas supprimer ces fiches lors d'un
nettoyage de démonstration. Les autres scénarios du guide ne sont pas
présentés comme effectués manuellement : ils restent couverts par les tests.

Rapport : `docs/reports/phase-4-increment-2-private-read.md` dans `afrimyth-api`.
Prochain incrément : création atomique d'une entité et de son profil via POST,
propriétaire automatique et brouillon initial. Les tâches transverses de tests
et de codes de refus restent ouvertes jusqu'à couverture des futures écritures.

### Avancement — incrément 3 validé le 30 octobre 2026

`POST /api/manage/entities/` crée une entité avec le profil correspondant à
son type, impose le compte appelant comme propriétaire et le statut `draft`.
Les référentiels existants et variantes autorisées peuvent être rattachés.
Une transaction couvre le couple entité/profil et les collections M2M.
Modification, suppression et transitions restent fermées ; les sources et
relations imbriquées sont différées. OpenAPI et README sont actualisés.

La revue a corrigé la conversion silencieuse d'un nombre en chaîne dans les
tableaux, avec quatre tests supplémentaires. Claude rapporte 691 tests
passants avant correction ; Codex a exécuté 123 tests ciblés après correction.
695 cas sont collectés, sans prétendre qu'une nouvelle suite complète a été
relancée. Avant clôture, 91 lignes de commentaires/docstrings redondants ont
été retirées ; l'AST hors docstrings confirme des instructions inchangées.

Essais Postman confirmés : création du héros fictif `demo-create-hero-a`
avec profil, statut `draft` et propriétaire 3 (`demo-contrib-a`), relecture
privée identique, détail public en 404, refus d'un statut fourni par le
client puis absence de la fiche refusée, refus d'un profil de divinité pour
un héros et refus d'un slug déjà utilisé. Les autres cas sont couverts
automatiquement, sans être présentés comme exécutés à la main.

La fiche fictive créée est conservée pour l'incrément 4. Anansi reste
attribuée au contributeur A ; les trois autres fiches du corpus restent
inchangées. Rapport : `docs/reports/phase-4-increment-3-create.md` dans
`afrimyth-api`. Prochain incrément : modification contrôlée des fiches.
La phase 4 n'est pas terminée.

Préparation de l'incrément 4, validée par l'auteur le 30 octobre : deux lots,
4A (champs, profil et rattachements) puis 4B (sources et relations). L'ADR 0005
est acceptée : le PATCH ordinaire ne change pas le type ; un reclassement
nécessiterait une opération distincte définie ultérieurement. Le prompt 4A
a été transmis à Claude CLI ; les deux lots ont ensuite été revus et validés.

### Incrément 4 — modification contrôlée, terminé le 31 octobre 2026

Lots 4A et 4B validés après revue du code et exercices Postman : PATCH des
champs métier, profil et rattachements ; POST/PATCH/DELETE individuels des
sources et relations, avec les droits de modification de la fiche porteuse.
Type, propriétaire, statut et suppression d'entité restent protégés.
Les références privées conservent les filtres de visibilité et les enfants
ne peuvent pas être atteints sous le slug d'une autre fiche.

Vérifications proportionnées : Claude rapporte 888 tests réussis avant les
corrections localisées de la revue 4B ; Codex a validé 39 tests ciblés après
ces corrections. Les contrôles 4A et JWT sont tracés dans leur rapport.
Aucune relance complète à la clôture documentaire. Les relances évitables
et les simplifications de tests sont consignées ; les consignes des deux
dépôts interdisent désormais explicitement le cumul non justifié d'anciennes
suites avec le passage complet final.

Essais 4A : modification partielle, champs omis conservés, liste vidée,
refus du statut/type/profil incompatible, contributeur B en 404, relecteur
autorisé, principal/secondaire cohérent et PUT refusé. Essais 4B : création,
modification et suppression des deux enfants, refus du doublon de relation,
de l'auto-relation et d'un mauvais parent ; erreur de source sans modification
à la relecture. Nettoyage confirmé par GET : source 2 et relation 10 supprimées,
héros fictif conservé avec sources/relations vides. Aucun changement du corpus.

Rapports du dépôt API : `docs/reports/phase-4-increment-4a-update.md` et
`docs/reports/phase-4-increment-4b-sources-relations.md`. Accès de développement
porté à 20 minutes pour les exercices ; 5 minutes hors debug conservées.
Revenir sur ce réglage avant la finalisation de l'API. La longueur des modules
serializers/views a été discutée : une séparation par responsabilités est
une piste de maintenance à évaluer, sans réorganisation effectuée ici.

Prochain incrément : **5 — soumission et reprise**. La phase 4 reste ouverte.

### Incrément 5 — soumission et reprise, terminé le 31 octobre 2026

Trois actions POST privées (submit, withdraw, resume) assurent draft → submitted,
submitted → draft et rejected → draft, selon rôle et propriété. Le propriétaire
peut retirer sa soumission malgré l'interdiction de PATCH sur submitted.
Visibilité, droit et statut sont relus sous verrou ; les corps non vides et
transitions invalides sont refusés sans mutation. Dates de soumission et marques
de revue suivent le contrat du prompt, sans nouveau seuil de complétude.

Revue : aucune correction fonctionnelle nécessaire ; deux descriptions OpenAPI
rectifiées et trois tests redondants retirés. Passage complet de Claude confirmé :
964 réussites avant ces retouches ; revue Codex : 31 tests ciblés réussis,
typage/format conformes, 961 tests collectés après simplification (pas une seconde
exécution complète). Les anciennes suites n'ont pas été cumulées inutilement
avec le passage final. Résultats réutilisés pour la clôture documentaire.

Postman confirmé : état initial, soumission, PATCH contributeur en 403,
retrait autorisé, contenu conservé puis PATCH rétabli ; corps status refusé et
retrait d'un draft refusé en 400. Rejet préparé manuellement dans l'admin sur
la seule fiche fictive, puis reprise via resume en 200. La fiche reste en draft,
propriétaire 3, catégorie « Village protector after withdrawal » et submitted_at
null. Aucun enfant supplémentaire créé, aucun changement des fiches du corpus.
Les autres croisements restent couverts automatiquement.

Rapport API : `docs/reports/phase-4-increment-5-submit-resume.md`.
Prochaine étape : **incrément 6 — modération** (publication, rejet, réouverture
réservée à l'administrateur), désormais validé et clôturé ci-dessous.
La phase 4 n'est pas terminée.

### Incrément 6 — modération, terminé le 1er novembre 2026

Publication et rejet de submitted par reviewer/administrator ; réouverture
published → draft réservée à administrator. Motif de rejet obligatoire,
note de publication facultative, dates et relecteur côté serveur ; revue
conservée à la réouverture puis effacée à la soumission suivante.

Revue technique du 31 octobre : pas de correction fonctionnelle nécessaire ;
documentation contradictoire corrigée, cinq doublons de tests retirés et une
simulation trompeuse remplacée par un contrôle de visibilité après suppression
intermédiaire. 1 061 réussites de Claude avant revue ; 31 tests ciblés réussis
après retouches, 1 056 cas collectés, sans seconde suite complète.
Rapport API : `docs/reports/phase-4-increment-6-moderation.md`.

À compter du 1er novembre, les explications pédagogiques sont conservées dans
`docs/verifications-code/phase-4-increment-6-moderation.md`. La lecture différée
du code ne bloque plus Postman ; chaque séance commence par les contrôles
prévus et leur objectif.

Postman validé le 1er novembre sur la seule fiche fictive : refus de publication
par le contributeur, rejet sans motif refusé, rejet motivé et absence publique,
reprise puis resoumission effaçant l'ancienne revue ; publication par le relecteur
et visibilité publique, reopen refusé au relecteur puis accepté à l'administrateur,
disparition publique finale. Le brouillon reste conservé, propriétaire 3,
reviewed_by 5 et note de publication conservés ; aucun nouveau compte ou enfant,
aucun changement du corpus. L'absence de champs de revue publics est couverte
automatiquement, sans confirmation manuelle détaillée distincte.

Résultats de revue réutilisés pour la clôture, sans relance complète pour la
documentation. Étape suivante : **incrément 7 — suppression logique et
restauration**, désormais validé et clôturé ci-dessous.
La phase 4 reste ouverte.

### Incrément 7 — terminé le 1er novembre 2026

DELETE logique de l'entité selon la matrice, POST restore réservé à l'admin,
contenu/statut/enfants conservés ; protection des références ayant des variantes
actives, coordination création/PATCH/restauration. Aucun effacement physique
d'entité, liste de corbeille ou nouveau statut. Restore conserve le statut :
un published restauré redevient public, un draft reste privé.

La revue a identifié un risque d'interblocage dans la première implémentation.
Claude l'a corrigé par acquisition ordonnée des lignes et ajouté un test réel
multiconnexion, sans interdiction nouvelle des cycles. Passage complet initial :
1 108 réussites et 1 ancien test attendant encore DELETE en 405 échoué ;
ce test est adapté et 236 tests affectés passent après correction. Pas de
seconde suite complète. Revue Codex : 29 tests ciblés réussis (dont le test
concurrent réel), typage/format conformes, 1 109 tests collectés. Un test de
création a été renforcé pour viser la fenêtre après validation ; comptes
inutilisés évités dans les matrices, sans perte de couverture.

Rapport API : `docs/reports/phase-4-increment-7-soft-delete-restore.md`.
Explications différées : `docs/verifications-code/phase-4-increment-7-soft-delete-restore.md`.
Le prompt correctif et son périmètre sont conservés dans `docs/prompts/`.
Postman confirmé sur la seule fiche fictive demo-create-hero-a : lecture privée
initiale, DELETE propriétaire en 204, GET et second DELETE en 404 ; restore
contributeur en 403, restauration administrateur avec fiche accessible, puis
seconde restauration refusée en 400. Aucun nouveau compte ou enfant, aucune
purge ou modification du corpus ; fiche conservée après restauration. Les
variantes, la conservation détaillée des colonnes et le cas published sont
couverts automatiquement sans être présentés comme exécutés manuellement.

Clôture documentaire et Git sans relance des suites inchangées. Prochaine étape :
**incrément 8 — consolidation et validation finale de la phase 4**. La phase 4
ne sera clôturée qu'après cette consolidation ciblée sur les écarts restants.

### Incrément 8 — consolidation terminée, phase 4 clôturée le 1er novembre 2026

Le prompt `docs/prompts/phase-4-increment-8-consolidation.md` borne le travail
aux écarts réels : réutilisation des garanties de matrice/parcours existantes,
examen ciblé des écritures admin et cohérence README/OpenAPI. Pas de fonctionnalité,
refactoring général, décision métier implicite ou démarrage de phase 5.
La consolidation ajoute un seul test d'aller-retour par les routes réelles :
création/profil → soumission → publication → suppression → restauration →
réouverture. Aucun code métier modifié ; seul un commentaire admin est corrigé.
Résultat final confirmé : **1 110 tests réussis en environ 20 min 23 s**, sans
relance. Rapport et copie scratchpad identiques. Les garanties des suites
existantes sont réutilisées, sans parcours manuel supplémentaire imposé.

L'auteur confirme la séparation déjà prévue : Django Admin pour le responsable
technique ; future interface éditoriale simple utilisant l'API selon les rôles.
Les protections des formulaires type/profil et de suppression physique dans
l'admin sont reportées à une tâche distincte, sans bloquer la clôture API.
Décision et solutions minimales conservées dans
`docs/notes/admin-et-interface-editoriale.md`. Pas d'UI développée maintenant.

Rapport API : `docs/reports/phase-4-increment-8-consolidation.md`.
Les contrôles de phase 4 sont achevés ; la suite est la phase 5, pas un
refactoring ou un nouveau cycle de revue sans fait nouveau.

**Livrable attendu :** CRUD authentifié complet avec workflow de modération démontrable et
matrice de permissions intégralement couverte par les tests.

---

## Phase 5 — Import maîtrisé du corpus V2

**Phase terminée le 2 novembre 2026.** Les 44 figures du corpus sont présentes
en base, plus une démonstration. Les huit tâches sont réalisées ; les lectures
Swagger ont été validées par l'auteur et remplacent les essais Postman.
La simulation séparée a été annulée ; aucun nouvel import n'est requis.

Objectif : charger les 44 entités sans faire du fichier JSON la nouvelle base de données.

**Estimation : 1 à 2 jours.**

### Découpage — trois incréments, sans élargissement du périmètre

Le découpage initial est conservé pour la traçabilité, avec l'incrément 2
annulé : la simulation sur corpus réel en base de tests est déjà acquise.
On passe de la commande livrée à son utilisation réelle avec sauvegarde.
L'auteur demande l'exécution de l'import par Claude CLI ; Codex conserve
le suivi documentaire, la revue et les lectures Postman. L'estimation
globale reste indicative, sans promesse de durée par incrément.

| Incrément | Travail et livrable vérifiable | Vérification pratique et raison |
|---|---|---|
| 1 — Étendre la commande | Ajouter `--all` à la commande existante, conserver le mode pilote et couvrir les différences réelles du corpus. Créer les référentiels, profils et liens selon les décisions déjà validées ; conserver les exclusions explicites. Tests en base de tests uniquement. | Revue Codex du rapport, des transformations et des tests : vérifier que les 44 fiches sont prises en charge sans modifier le contrat API ni écraser les fiches existantes. Aucun import dans la base de développement. |
| 2 — Simulation séparée, annulée | Aucun travail supplémentaire demandé. La simulation des 44 figures en base de tests est déjà acquise à l'incrément 1. | Ne pas lancer l'ancien prompt de simulation sur la base locale. |
| 3 — Importer et valider | Après validation du bilan, sauvegarder la base locale, exécuter l'import réel, puis relancer la même commande. Contrôler les profils, rattachements et relations ainsi que la préservation des fiches antérieures. Clôturer rapport et roadmap. | Dans le terminal/ORM : vérifier les fiches du corpus, puis l'absence de doublons et d'écrasement à la seconde exécution. Dans Postman : lire quelques fiches importées avec un rôle autorisé et vérifier un filtre ou une relation représentative, pour confirmer que les données sont utilisables par l'API. |

### Où nous en sommes — livraison de l'incrément 1

| Incrément | État réel | Résultat visible pour l'auteur |
|---|---|---|
| 1 — Étendre la commande | Terminé, livré et revu. Corrections documentaires locales sans changement fonctionnel ; résultats des tests réutilisés. | La commande importe toutes les figures du JSON avec `--all` et conserve le mode pilote. |
| 2 — Simulation séparée | Annulée par décision de l'auteur, sans exécution sur la base locale. | Aucun nouveau livrable attendu. |
| 3 — Importer et valider | Terminé : sauvegarde, import et relance vérifiés ; consultation validée dans l'admin et Swagger. | 40 nouvelles fiches, 4 figures du corpus préservées et une démonstration conservée : 45 au total. Liste privée et détail d'Oshun avec relation résolue vers Shango confirmés par l'auteur. |

La simulation de Claude a lu les 44 figures dans une base temporaire vide :
44 créations tentées, puis annulées. Cette base a ensuite été détruite ;
elle n'est ni un environnement à ouvrir ni une livraison à consulter dans
Postman. Elle prouve le fonctionnement sur le corpus source, pas encore la
cohabitation avec les fiches déjà présentes dans notre base de développement.

La simulation séparée n'est plus un préalable : utiliser les garanties
transactionnelles déjà testées et une sauvegarde de la base avant l'import.
Ne pas rejouer la simulation sur base vide ou toute la batterie de tests.
La revue Codex est un contrôle interne du livrable, sans nouvelle séance
pédagogique imposée à l'auteur.

Rapport du livrable :
`C:\Users\heric\dev\afrimyth-api\docs\reports\phase-5-increment-1-import-complet.md`.
Les résultats acquis sont réutilisés : 1 126 tests réussis, sans relance
nécessaire pour cette mise à jour documentaire.

L'ancien prompt de l'incrément 2 porte la mention ANNULÉ.
Le prompt actif demandé pour Claude est :
`docs/prompts/phase-5-increment-3-import-reel.md`.
Périmètre : sauvegarde, import réel, vérification de préservation et unique
réexécution, sans code ni relance des tests. Sa préparation ne signifie
pas que l'import a été exécuté.

**Résultat désormais acquis — incrément 3.** Sauvegarde PostgreSQL conservée
hors Git et catalogue vérifié. Premier import : 40 créations et 4 figures
existantes ignorées ; seconde exécution : 0 création et 44 ignorées. Claude
rapporte la préservation des cinq fiches antérieures et un état strictement
identique après le second appel. Codex a confirmé en transaction READ ONLY
les 44 identités source, les 45 fiches totales, les 40 nouveaux brouillons
avec profils cohérents, les décomptes et les limites annoncées. Aucun import
ni test relancé pendant cette vérification.

Les trois relations anciennes d'Oya restent par nom et Shango conserve son
absence préexistante de profil : ces limites sont enregistrées sans réparation
implicite. Rapport API : `docs/reports/phase-5-increment-3-import-reel.md`.
Cette livraison et la phase sont terminées. Aucune nouvelle tâche confiée
à Claude pour refaire cet import.

### Clôture — 2 novembre 2026

L'auteur a constaté les 45 fiches dans l'administration, puis s'est connecté
dans Swagger avec le relecteur. Liste privée en 200 avec `count: 45`, pagination
expliquée ; détail d'Oshun en 200 et lien `consort` vers la fiche Shango
confirmé. Ces lectures remplacent Postman, sans répétition ni publication.
Les données restent en brouillon et les limites connues sont conservées.

Revue de l'extension et des tests effectuée : pas de moteur d'import générique,
pas de migration ni changement des droits API. Les tests couvrent des risques
distincts (sélection, intégrité, rollback et préservation). Codex a raccourci
les docstrings modifiées et supprimé les docstrings redondantes des nouveaux
tests ; contrôles limités à l'équivalence AST hors docstrings et à Ruff/format.
Les 1 126 tests déjà acquis sont réutilisés, sans nouveau passage complet.
Prochaine phase : qualité et livraison portfolio, à découper avant exécution.

### Règles communes et critères de clôture

- Le JSON source reste en lecture seule. Aucune copie du corpus, des images
  ou du code de V2 dans le dépôt API ; aucun nouvel importeur générique.
- Les fiches existantes sont préservées : contenu, profil, propriétaire,
  statut, revue et collections. Pas de remise en brouillon, restauration,
  réparation ou enrichissement implicite. Les démonstrations sont conservées.
- Les nouvelles fiches sont des brouillons sans propriétaire, avec leur
  profil correspondant. L'import ne vaut ni revue éditoriale ni publication.
- Le bilan distingue les 44 entrées source des créations effectives : les
  fiches déjà présentes réduisent ce nombre, les démonstrations augmentent
  le nombre total en base. Un total global différent de 44 n'est pas une erreur.
- Les cibles absentes sont conservées par nom. Les conflits d'identité ou
  transformations non décidées sont explicités, sans rapprochement inventé.
  Seul un problème empêchant l'import correct suspend l'étape concernée.
- Les essais Postman restent courts et portent sur le corpus importé ; ne
  pas rejouer la matrice de permissions de phase 4. Un contributeur ne voit
  pas automatiquement les brouillons importés sans propriétaire : utiliser
  un compte autorisé par les règles existantes. Aucune publication de masse
  pour rendre artificiellement la collection publique visible.
- Réutiliser les contrôles acquis sur le même code. Ne pas relancer toute
  la suite pour une simulation, une opération d'import ou une mise à jour
  documentaire seule. Toute correction révélée appelle des tests proportionnés.
- Les explications différées, commandes pratiques, notes et clôtures Git
  restent du ressort de Codex. Pas de nouvelle interface, correction admin,
  refonte des serializers ou travaux Docker/CI de phase 6 dans cette phase.

Le prompt de l'incrément 1, désormais livré, est conservé ici :
`docs/prompts/phase-5-increment-1-import-complet.md`. Il couvre uniquement la
préparation et les tests, pas l'import réel de l'incrément 3. Aucun import
n'est lancé par la seule mise à jour de ce découpage.

### Suivi de réalisation

- [x] Compléter la commande amorcée et validée sur trois entités en phase 2 — option `--all` livrée.
- [x] Rendre la commande d'import idempotente — tests et seconde exécution sur la base locale réussis.
- [x] Créer les référentiels nécessaires pendant l'import de façon déterministe — implémenté, testé et exécuté lors de l'import réel.
- [x] Résoudre les relations en deux passes et conserver les cibles absentes sous forme de nom — implémenté et vérifié sur le corpus en base de tests.
- [x] Appliquer les décisions prises sur les ethnies, les pays et les domaines — correspondances explicites, valeurs ambiguës conservées sans fusion inventée.
- [x] Signaler explicitement dans le rapport d'import les champs non importés, dont
  `origin.pantheon`.
- [x] Importer les 44 entités après validation du pilote — 40 créées et 4 figures antérieures préservées ; démonstration conservée.
- [x] Tester une seconde exécution pour prouver l'absence de doublons — 0 création, 44 ignorées, état après le premier import conservé selon le relevé Claude.

Les réserves du rapport sur `Congo`, les termes proches et les descendants
ne sont pas de nouvelles tâches de développement obligatoires : l'import
préserve les distinctions, signale l'ambiguïté et n'invente pas de liens.
Compléter les relations des anciennes fiches du pilote reste distinct d'un
import qui doit les préserver. Seul un conflit empêchant effectivement
l'exécution doit être résolu avant de poursuivre.

**Livrable attendu :** commande reproductible important les 44 entités, rapport lisible des
transformations et omissions, relations conservées et seconde exécution sans doublon.

---

## Phase 6 — Qualité et livraison portfolio

**Découpage préparé le 2 novembre 2026. Aucune implémentation de phase 6
n'est lancée par ce plan.** La phase 5 est terminée ; ses résultats et ceux
de la phase 4 sont acquis, sans nouvelle validation générale du métier.

Objectif : rendre le projet reproductible, lisible et présentable publiquement.

**Estimation indicative : 2 à 3 jours**, selon l'environnement Docker et
les résultats de la première CI. Ce n'est pas un délai garanti.

### Découpage — quatre incréments avec un résultat observable

| Incrément | Travail et livrable | Pourquoi / vérification de fin | Responsable |
|---|---|---|---|
| 1 — Démarrage avec Docker | Ajouter Dockerfile, Compose API/PostgreSQL et exclusions de construction ; préciser variables et commandes nécessaires. Ne pas utiliser la base locale des exercices. | Démarrer une installation isolée, appliquer les migrations et obtenir un health check réussi ainsi que Swagger accessible. Objectif : permettre de lancer le projet sans reproduire manuellement tout l'environnement. | Claude : implémentation nécessaire et rapport court. Codex : revue, instructions de lancement et essai avec l'auteur. |
| 2 — Couverture et CI | Ajouter la mesure de couverture, choisir un seuil à partir du résultat réel et configurer GitHub Actions : Ruff/format, mypy, migrations, schéma OpenAPI et tests avec PostgreSQL de service. Conserver les réglages déjà utiles. | Une exécution complète avec couverture sert de référence, puis une CI verte sur le commit publié. Objectif : détecter les régressions automatiquement, sans batteries locales répétées. | Claude : configuration et seuls tests nécessaires. Codex : revue, publication et lecture du résultat CI. |
| 3 — Documentation et langue | Mettre à jour le README existant, compléter démarrage Docker et exemples par rôle, traduire les docstrings françaises restantes en anglais et corriger les libellés publics encore contraires à l'ADR 0003. Garder les documents internes français. | Relire le parcours d'installation et de consultation à partir des commandes déjà essayées ; vérifier l'absence de changement fonctionnel pour les docstrings et contrôler seulement les fichiers affectés. Objectif : rendre le dépôt compréhensible par une personne extérieure. | Codex : documentation, traduction sans changement fonctionnel et petites corrections locales. Toute correction importante du contrat public reçoit un prompt distinct pour Claude. |
| 4 — Revue finale et version 1.0.0 | Examiner uniquement les risques restants : secrets/fichiers suivis et contexte Docker, réglages hors debug, permissions ou requêtes touchées, cohérence des versions et limites documentées. Aligner la version du projet, clôturer le suivi et créer le tag v1.0.0 sur un commit à CI verte. | Réutiliser les garanties déjà acquises ; contrôler les écarts réellement trouvés. Vérifier CI verte et tag distant du bon commit. Objectif : livrer une version identifiable et reproductible du portfolio. | Codex : revue, rapport court et clôture Git autorisée. Claude intervient seulement si une correction fonctionnelle importante est réellement nécessaire. |

### Périmètre et ordre des travaux

- Commencer par l'incrément 1 ; aucun prompt ne lance toute la phase à la fois.
  La CI vient ensuite sur cet environnement ; la documentation se stabilise
  avant la version finale. Les comptes, brouillons et données locales existants
  ne sont pas des prérequis à la construction ou aux tests de l'application.
- Docker utilise une base et un volume distincts de la base PostgreSQL locale
  contenant les 44 figures et la démonstration. Ne pas migrer, purger ou écraser
  cette dernière. Ne pas arrêter les services personnels pour libérer un port
  sans nécessité expliquée. Pas de suppression de volumes contenant des données.
- L'image, le dépôt et la CI ne contiennent ni `.env`, sauvegarde, jeton, compte
  réel ou corpus externe. Le chemin `--input` reste facultatif pour importer
  des données : le démarrage de l'API ne dépend pas du JSON de `_audit`.
  Les tests restent autonomes sur leurs données synthétiques.
- Docker sert ici une installation locale reproductible. Déploiement chez un
  hébergeur, domaine/TLS, Kubernetes, nouvelle interface et raccordement V2
  ne font pas partie de cette phase.
- Ne pas viser arbitrairement 100 % de couverture. Mesurer d'abord, examiner
  les branches importantes manquantes et fixer un seuil motivé sans fabriquer
  des tests de détail pour atteindre un chiffre. Conserver les tests métier
  existants ; ne pas réécrire la matrice de permissions déjà validée.
- Les 1 126 tests du dernier état sont acquis. Pour ajouter la couverture,
  préférer un passage complet avec mesure à un passage sans puis un autre
  avec. Une exécution CI sur le commit final reste nécessaire ; son résultat
  est réutilisé sans imposer un nouveau passage complet local identique.
- Traduction de docstrings : pas de nouveau cycle fonctionnel ; comparer le
  code hors documentation et vérifier format/lint. Une modification d'un
  message public ou d'un comportement appelle les seuls contrôles affectés.
- Ne pas ajouter systématiquement un scanner, outil de sécurité, cache de tests,
  matrice multi-versions ou infrastructure de démonstration. Les contrôles de
  livraison doivent répondre à un risque concret du projet.
- La revue finale vérifie notamment que l'accès JWT allongé reste limité au
  développement et que les réglages hors debug suivent le contrat existant.
  Elle n'est pas une certification de production ni une nouvelle phase métier.
- Shango sans profil, anciennes relations d'Oya, ambiguïtés du corpus et
  protections Django Admin reportées restent documentés. Leur correction ne
  devient pas implicitement une condition de clôture de la phase 6 ; un fait
  nouveau réellement bloquant doit être décrit avec son impact précis.

### Suivi — cases cochées après réalisation, pas après préparation du prompt

Incrément 1 préparé : `docs/prompts/phase-6-increment-1-docker.md`.
Prérequis constatés le 2 novembre 2026 : Docker Desktop/Compose installés,
WSL 2 présent ; moteur Docker non disponible avant ouverture de Desktop.
Action manuelle initiale : ouvrir Docker Desktop et attendre son démarrage.
PostgreSQL sera fourni par Compose dans une instance isolée : aucune nouvelle
installation Windows ni manipulation de la base actuelle requise.
Les éventuelles autres opérations Windows seront précisées uniquement si un
message réel les exige.

Incrément 1 validé le 3 novembre 2026. L'auteur a confirmé le démarrage dans
Docker Desktop. Codex a vérifié PostgreSQL sain et les réponses 200 de la
santé, de Swagger, du schéma et de la collection publique vide sur le port
8001. Le rapport consigne les migrations et la persistance du volume.

- [x] Ajouter `Dockerfile` et `docker-compose.yml` pour l'API et PostgreSQL.
- [ ] Ajouter la mesure de couverture à pytest et fixer un seuil réaliste.
- [ ] Ajuster les configurations Ruff et mypy déjà utilisées depuis la phase 1, sans introduire
  un rattrapage massif en fin de projet.
- [ ] Ajouter une CI GitHub Actions : lint, typage, tests et contrôle des migrations.
- [ ] Vérifier qu'aucun secret ni fichier de développement ne se trouve dans le dépôt.
- [ ] Rédiger le README : objectif, architecture, installation, commandes, API et décisions.
- [ ] Traduire en anglais les docstrings françaises existantes dans tous les fichiers
  de code, tests compris, sans changement fonctionnel ; aligner les consignes du
  dépôt API sur l'ADR 0003. Les nouvelles docstrings sont en anglais dès maintenant.
- [ ] Documenter quelques scénarios reproductibles pour chaque rôle.
- [ ] Exécuter une revue finale ciblée sur sécurité, permissions et requêtes.
- [ ] Créer la version `v1.0.0` lorsque la CI est verte.

Les deux premières cases techniques relèvent respectivement des incréments
1 (Docker) et 2 (couverture) ; Ruff/mypy et CI relèvent aussi du 2. README,
langue et scénarios relèvent du 3. Contrôle des secrets, revue finale et
version relèvent du 4. Chaque livraison actualise ces cases et son état,
sans présenter une préparation comme une réalisation.

**Livrable attendu :** dépôt public avec démarrage documenté par Compose,
CI verte, documentation et scénarios reproductibles, version `v1.0.0`
identifiable dans le portfolio. La mise en ligne sur un hébergeur n'est
pas requise pour atteindre ce livrable.

---

## Estimation globale de la V1

| Phase | Fourchette |
|---|---:|
| 0 — Décisions structurantes | 0,5–1 jour |
| 1 — Premier incrément | 1–2 jours |
| 2 — Modèle et import pilote | 3–5 jours |
| 3 — Consultation publique | 2–3 jours |
| 4 — CRUD et modération | 3–5 jours |
| 5 — Import complet | 1–2 jours |
| 6 — Qualité et livraison | 2–3 jours |
| **Total V1** | **12,5–21 jours de travail** |

La cible est donc d'environ trois à cinq semaines de travail régulier, marge de correction
incluse, et reste nettement sous deux mois. Si une phase dépasse sa borne haute, son contenu
non indispensable est déplacé hors V1 avant d'élargir le calendrier.

---

## Hors périmètre de la V1

- Connexion effective du frontend `african_mythology_V2` à cette API.
- Génération d'images par Vertex AI.
- Internationalisation des contenus.
- Propositions de modification séparées pour les entités déjà publiées.
- Moteur générique de workflow ou journal d'audit personnalisé.
- Enrichissement automatique vers 400 entités.
- Résolution définitive des décisions métier consignées comme reportées dans l'ADR 0001.

Ces éléments pourront former une roadmap d'intégration ultérieure, après la livraison de l'API
autonome.

### Première phase de la future roadmap d'intégration V2 — Données de rendu

Cette phase est conservée intégralement, mais elle ne fait pas partie de la V1 de l'API. Elle
sera le premier incrément du chantier de raccordement à `african_mythology_V2`.

Objectif : exposer les données nécessaires au moteur visuel sans intégrer encore le frontend.

**Estimation indicative : 2 à 4 jours, hors total V1.**

- [ ] Ajouter le profil de rendu, les variantes de prompt et les styles visuels.
- [ ] Ajouter les métadonnées d'images sans imposer le stockage définitif des fichiers.
- [ ] Importer la matrice de styles utile au corpus.
- [ ] Définir une représentation API compatible avec les besoins observés dans V2.
- [ ] Tester les prompts canoniques, variantes, styles disponibles et images absentes.
- [ ] Documenter clairement la frontière entre données mythologiques et génération d'images.

**Livrable attendu :** endpoints documentés fournissant à un futur client V2 les prompts,
styles et références d'images, sans dépendance directe à Vertex AI.

## Définition globale de terminé

Le projet 1 est terminé lorsque :

- [x] les 44 entités sont importables sans doublon ;
- [x] les lectures publiques et le CRUD authentifié sont documentés — phases 3 et 4 ; installation et présentation finale à compléter en phase 6 ;
- [x] la matrice de permissions est entièrement testée — phase 4 et consolidation validées ;
- [x] le workflow de contribution fonctionne de bout en bout — phase 4 validée par tests et exercices ;
- [ ] PostgreSQL, Docker et la CI sont opérationnels ;
- [ ] aucune dépendance à l'application V2 ou à Vertex AI n'est nécessaire pour lancer l'API ;
- [ ] une personne externe peut installer, tester et comprendre le projet à partir du README.
