# Revue éditoriale du corpus pilote

- Date : 21 octobre 2026
- Origine : remarques de l'auteur après consultation des fiches API du pilote
- Statut : points à examiner, sans nouvelle décision de schéma

## Portée de la validation précédente

La phase 2 a validé la transformation et la restitution des données du corpus,
pas la véracité, l'exhaustivité ou la classification scientifique de leurs
contenus. Les trois fiches ont été importées en `draft`, sans propriétaire
attribué et sans sources documentaires. Leur présence provisoire dans l'API
ne vaut pas approbation éditoriale.

L'API et PostgreSQL deviennent la source de vérité opérationnelle des données
gérées ; cela ne signifie pas que chaque assertion importée est établie.
Le travail éditorial doit documenter les interprétations et leurs sources.

## Liste des points à revoir

| Fiche / sujet | Question ou observation | Suite attendue |
|---|---|---|
| Oya, Anansi, Mokele-mbembe | Des corrections de contenu sont attendues ; les motifs détaillés ne sont pas encore tous fournis | Maintenir les fiches en brouillon jusqu'à revue ; compléter cette note avec les observations sourcées |
| Anansi | L'auteur signale une classification pouvant varier entre héros, créature et divinité selon les contextes | Documenter les traditions concernées avant de confirmer un type principal ou de créer des variantes culturelles ; ne pas changer le type ni dupliquer les fiches sur cette seule remarque |
| Mokele-mbembe | L'aire Congo Basin est renseignée mais aucun pays ne l'est ; l'auteur souhaite examiner les rattachements à la République du Congo et à la RDC | Rechercher les pays effectivement concernés par cette figure et cette version du récit ; renseigner un pays principal et/ou des pays secondaires si les références le justifient ; conserver l'aire culturelle |
| Analyse et ambiguïtés | Le récit seul peut ne pas suffire à expliquer les classifications et les variantes | Évaluer un contenu analytique distinct, avec l'idée de champ évoquée comme « story analytics » ; préciser public visé, contenu, sources et différence avec le récit avant toute modification de modèle |

Ces observations sont des questions éditoriales à instruire, pas des faits
confirmés par une recherche documentaire effectuée pendant cette intervention.
Ne pas attribuer de pays à une figure par simple déduction du nom de son aire.

## « À revoir » et workflow existant

L'ADR 0002 prévoit quatre statuts : `draft`, `submitted`, `published`, `rejected`.
Il n'y a pas de statut technique supplémentaire `to_review` à ce stade.

- À corriger avant soumission : conserver `draft`.
- Prête pour examen par un relecteur : passer à `submitted` via l'action de
  soumission prévue en phase 4.
- Retour par un relecteur avec corrections demandées : `rejected` et
  `review_notes`, puis retour en `draft` pour correction, selon les transitions
  et permissions déjà validées.
- Un propriétaire peut retirer son `submitted` vers `draft` ; les transitions
  ne sont pas réservées au seul administrateur. Un contributeur ne publie ni
  ne rejette lui-même une soumission selon la matrice actuelle.

Les champs existent, mais les actions et permissions HTTP arriveront en
phase 4. Les fiches importées ont `created_by=null` : il ne faut pas présumer
un propriétaire humain parce que la commande a été lancée depuis son poste.
Leur prise en charge éditoriale et, si nécessaire, leur attribution explicite
devront être traitées par une personne habilitée.

La présente note garde les remarques du pilote. Elle ne crée pas de workflow,
de journal d'audit, de champ d'analyse ou de mécanisme de signalement. Le cas
d'une fiche déjà publiée signalée « à revoir » reste à distinguer du circuit
de soumission : la V1 n'a pas de proposition de modification séparée, et la
réouverture `published` → `draft` est réservée à l'administrateur.

## Recherches et URL — phase 3

Le détail `/api/entities/{slug}/` identifie une fiche ; les paramètres de la
collection `/api/entities/` sélectionneront un ensemble. Les exemples
ci-dessous sont des contrats à préciser lors de l'implémentation, pas des
filtres déjà disponibles :

- `/api/entities/?entity_type=divinity`
- `/api/entities/?entity_type=creature&country=CD`
- `/api/entities/?cultural_area=central-africa`
- `/api/entities/?search=anansi&page=2`

Le filtre pays doit documenter s'il considère les rattachements principaux
et secondaires ; le besoin exprimé justifie les deux. Le filtre de grande
région doit préciser la prise en compte des sous-régions : Congo Basin est
enfant de Central Africa. Il ne doit pas transformer une aire en pays ou
inventer une origine manquante. Éviter les doublons dans les résultats des
relations multiples. Les détails de pagination et les paramètres exacts
seront fixés et documentés en phase 3.

Un filtre ne peut pas retrouver un rattachement absent de la base. L'amélioration
de la recherche et l'enrichissement éditorial sont donc complémentaires.

## Moments de reprise

- Phase 3 : paramètres de filtre, pagination, hiérarchie des aires, documentation
  de leur portée, visibilité publique limitée aux fiches publiées. Ne pas
  publier les brouillons du pilote automatiquement pour conserver une démonstration.
- Phase 4 : actions de soumission/retrait/rejet/correction, permissions,
  propriété et retour de revue.
- Avant publication éditoriale des fiches : classification d'Anansi, pays de
  Mokele-mbembe, sources et corrections détaillées des trois fiches.
- Champ analytique éventuel : discussion sur exemples concrets avant une
  décision de modèle ; ce point ouvert ne rouvre pas à lui seul la phase 2.
