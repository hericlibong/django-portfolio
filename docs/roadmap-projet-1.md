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
- [ ] Trancher la gestion des ethnies : principale seule, principale + secondaires, ou M2M.
- [ ] Trancher la gestion des pays : principal + secondaires, M2M seul, ou texte libre.
- [ ] Choisir l'emplacement unique des domaines et réconcilier les quatre divergences connues.
- [ ] Confirmer que l'API devient la source de vérité après l'import initial du corpus V2.
- [ ] Confirmer la stratégie `Entity` + profils `OneToOne` explicites.
- [ ] Valider le workflow et la matrice de permissions déjà proposés.
- [ ] Enregistrer les décisions retenues dans des ADR courtes.

**Livrable attendu :** décisions bloquantes consignées, schéma initial suffisamment stable
pour écrire les modèles sans chercher à figer tout le futur contrat REST.

---

## Phase 1 — Premier incrément exécutable

Objectif : obtenir rapidement une application qui démarre et une première tranche métier qui
fonctionne de bout en bout.

**Estimation : 1 à 2 jours.**

- [ ] Créer le dépôt autonome avec le nom validé en phase 0.
- [ ] Initialiser Django, Django REST Framework et la configuration PostgreSQL.
- [ ] Configurer pytest et pytest-django avant les premiers tests.
- [ ] Configurer Ruff et mypy dès l'initialisation, avec des règles utiles mais compatibles
  avec un démarrage rapide.
- [ ] Créer `CLAUDE.md` et `AGENTS.md` avec les règles communes du projet et seulement les
  différences propres à chaque agent.
- [ ] Créer les apps minimales `accounts` et `mythology`.
- [ ] Déclarer le modèle utilisateur personnalisé dès la première migration.
- [ ] Créer `Entity` et `DivinityProfile` avec leur relation `OneToOne`.
- [ ] Ajouter les migrations et l'administration Django minimale.
- [ ] Exposer un endpoint de santé.
- [ ] Exposer une première lecture liste/détail des entités.
- [ ] Ajouter les premiers tests de modèles et d'API.

**Livrable attendu :** dépôt installable, base PostgreSQL migrable, administration accessible,
endpoint de santé et première ressource `Entity` consultable avec tests, Ruff et mypy au vert.

---

## Phase 2 — Modèle métier complet de la V1

Objectif : compléter le schéma nécessaire au corpus sans ajouter les fonctions reportées.

**Estimation : 3 à 5 jours, import pilote inclus.**

L'audit a parcouru les 44 entités et a déjà identifié les valeurs composites, les relations
non résolues et les divergences de domaines. Cela suffit pour proposer un schéma initial, mais
pas pour garantir qu'aucune contrainte ou longueur de champ ne devra être corrigée. Le pilote
est donc volontairement placé dans cette phase, avant de construire toute l'API autour du
modèle.

- [ ] Ajouter `HeroProfile` et `CreatureProfile`.
- [ ] Ajouter les référentiels validés : régions culturelles, pays, ethnies et domaines.
- [ ] Ajouter les relations entre entités, y compris les relations dont la cible n'existe pas
  encore dans le corpus.
- [ ] Ajouter les sources documentaires.
- [ ] Ajouter à `Entity` la propriété, le statut et les champs de revue.
- [ ] Imposer la cohérence entre `entity_type` et le profil associé.
- [ ] Ajouter les contraintes, index et tests de validation utiles.
- [ ] Vérifier les requêtes avec `select_related` et `prefetch_related` sans optimisation
  prématurée supplémentaire.
- [ ] Écrire le squelette de la commande d'import et charger une entité réelle de chaque type.
- [ ] Confronter ce pilote aux contraintes, longueurs, valeurs nulles, caractères spéciaux et
  relations vers des cibles absentes ; corriger le modèle avant de le considérer stable.

**Livrable attendu :** modèle relationnel complet de la V1, migrations propres et suite de
tests démontrant les principales contraintes métier, validé par le passage en base de trois
entités réelles représentatives du corpus.

---

## Phase 3 — API publique de consultation

Objectif : rendre les données exploitables par un client avant d'ajouter toute la gestion des
contributions.

**Estimation : 2 à 3 jours.**

- [ ] Stabiliser les serializers de liste et de détail.
- [ ] Exposer les routes liste et détail avec des UUID et des slugs stables.
- [ ] Ajouter pagination, recherche et filtres réellement utiles.
- [ ] Ne rendre publiquement visibles que les entités `published`.
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
