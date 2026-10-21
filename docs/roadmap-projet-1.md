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

Objectif : réaliser le cœur démonstratif du projet portfolio.

**Estimation : 3 à 5 jours, dont 0,5 à 1 jour de reprise de la phase 3.**

- [ ] Ajouter l'authentification JWT avec renouvellement de jeton.
- [ ] Créer par migration les groupes `contributor`, `reviewer` et `administrator`.
- [ ] Attribuer automatiquement `created_by` à la création.
- [ ] Implémenter création, modification et suppression selon la matrice de permissions.
- [ ] Implémenter les actions explicites de soumission, publication, rejet et réouverture.
- [ ] Interdire la modification directe du statut par un `PATCH` ordinaire.
- [ ] Ajouter les tests croisés rôle × propriétaire × statut × action.
- [ ] Vérifier que les réponses refusées utilisent correctement `401`, `403` ou `404`.

**Livrable attendu :** CRUD authentifié complet avec workflow de modération démontrable et
matrice de permissions intégralement couverte par les tests.

---

## Phase 5 — Import maîtrisé du corpus V2

Objectif : charger les 44 entités sans faire du fichier JSON la nouvelle base de données.

**Estimation : 1 à 2 jours.**

- [ ] Compléter la commande amorcée et validée sur trois entités en phase 2.
- [ ] Rendre la commande d'import idempotente.
- [ ] Créer les référentiels nécessaires pendant l'import de façon déterministe.
- [ ] Résoudre les relations en deux passes et conserver les cibles absentes sous forme de nom.
- [ ] Appliquer les décisions prises sur les ethnies, les pays et les domaines.
- [ ] Signaler explicitement dans le rapport d'import les champs non importés, dont
  `origin.pantheon`.
- [ ] Importer les 44 entités après validation du pilote.
- [ ] Tester une seconde exécution pour prouver l'absence de doublons.

**Livrable attendu :** commande reproductible important les 44 entités, rapport lisible des
transformations et omissions, relations conservées et seconde exécution sans doublon.

---

## Phase 6 — Qualité et livraison portfolio

Objectif : rendre le projet reproductible, lisible et présentable publiquement.

**Estimation : 2 à 3 jours.**

- [ ] Ajouter `Dockerfile` et `docker-compose.yml` pour l'API et PostgreSQL.
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

**Livrable attendu :** dépôt public démarrant en une commande, CI verte, documentation complète,
démonstration reproductible du workflow et version `v1.0.0` publiable dans le portfolio.

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

- [ ] les 44 entités sont importables sans doublon ;
- [ ] les lectures publiques et le CRUD authentifié sont documentés ;
- [ ] la matrice de permissions est entièrement testée ;
- [ ] le workflow de contribution fonctionne de bout en bout ;
- [ ] PostgreSQL, Docker et la CI sont opérationnels ;
- [ ] aucune dépendance à l'application V2 ou à Vertex AI n'est nécessaire pour lancer l'API ;
- [ ] une personne externe peut installer, tester et comprendre le projet à partir du README.
