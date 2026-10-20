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
- [ ] Imposer la cohérence entre `entity_type` et le profil associé.
- [ ] Ajouter les contraintes, index et tests de validation utiles.
- [ ] Vérifier les requêtes avec `select_related` et `prefetch_related` sans optimisation
  prématurée supplémentaire.
- [ ] Écrire le squelette de la commande d'import et charger une entité réelle de chaque type.
  Préparer les quatre régions continentales du périmètre, les sous-régions et la diaspora
  selon la précision de la section 3 de l'ADR 0002 ; ne pas confondre `kind` et niveau
  hiérarchique.
- [ ] Confronter ce pilote aux contraintes, longueurs, valeurs nulles, caractères spéciaux et
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

**Livrable attendu :** modèle relationnel complet de la V1, migrations propres et suite de
tests démontrant les principales contraintes métier, validé par le passage en base de trois
entités réelles représentatives du corpus.

---

## Phase 3 — API publique de consultation

Objectif : rendre les données exploitables par un client avant d'ajouter toute la gestion des
contributions.

**Estimation : 2 à 3 jours.**

- [ ] Normaliser en anglais, avant la stabilisation de l'API publique, tous les libellés, aides
  et messages exposés aux clients : `verbose_name`, libellés des classes de choix, `help_text`
  et messages d'erreur de validation. Langue canonique fixée par
  [`ADR 0003`](adr/0003-langue-publique-api.md).
- [ ] Stabiliser les serializers de liste et de détail.
- [ ] Exposer les routes liste et détail avec des UUID et des slugs stables.
- [ ] Ajouter pagination, recherche et filtres réellement utiles.
- [ ] Ne rendre publiquement visibles que les entités `published`.
- [ ] Appliquer également les règles de visibilité aux cibles imbriquées (`variant_of`
  et cibles des relations) : une fiche supprimée ou non publiée ne doit pas être divulguée
  par le détail d'une autre fiche. Le manager `active` ne filtre que les objets racines.
- [ ] Centraliser la sélection des objets visibles dans une méthode réutilisable afin que la
  phase 4 ajoute les vues privées sans réécrire les serializers de lecture.
- [ ] Produire le schéma OpenAPI avec drf-spectacular.
- [ ] Ajouter des exemples de réponses et documenter les erreurs principales.
- [ ] Tester les requêtes, filtres, données imbriquées et règles de visibilité.

**Livrable attendu :** API de consultation documentée dans Swagger/ReDoc, utilisable sans
authentification pour les entités publiées et couverte par des tests d'intégration.

**Reprise prévue en phase 4 : 0,5 à 1 jour**, incluse dans l'estimation de cette phase. Elle
porte sur les classes de permission, l'élargissement conditionnel du queryset aux objets
privés et les tests associés. Les serializers de liste et de détail ne doivent pas être
réécrits. Pendant la phase 3, les statuts sont posés par les fixtures, l'administration et
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
