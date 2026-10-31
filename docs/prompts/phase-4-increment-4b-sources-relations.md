# Phase 4 — Incrément 4B : écritures des sources et relations

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses consignes,
les ADR 0002 à 0005 et la phase 4 de la roadmap du dépôt voisin
`django-portfolio`. Le code 4A a été revu et ses exercices Postman terminés.
Des changements peuvent rester non committés : les préserver, ne pas refaire
4A ni sa clôture. Ne rien committer ou pousser pendant cette intervention.

## Livrable et routes retenues

Ouvrir les écritures individuelles des enfants d'une fiche :

| Méthode | Route sous `/api/manage/entities/` | Réponse |
|---|---|---|
| POST | `{slug}/sources/` | 201, source créée |
| PATCH | `{slug}/sources/{id}/` | 200, source modifiée |
| DELETE | `{slug}/sources/{id}/` | 204 sans corps |
| POST | `{slug}/relations/` | 201, relation créée |
| PATCH | `{slug}/relations/{id}/` | 200, relation modifiée |
| DELETE | `{slug}/relations/{id}/` | 204 sans corps |

Le slug identifie l'entité porteuse ; les identifiants des enfants sont
les identifiants entiers déjà présents dans leurs représentations.
Réutiliser les représentations de lecture existantes pour les réponses.
La lecture reste dans le GET privé de l'entité : pas de nouvelles listes
ou routes GET d'enfants, pas de pagination additionnelle. PUT reste fermé.
Le DELETE retire uniquement la ligne Source ou EntityRelation concernée,
jamais une Entity, un profil ou la fiche cible. Pas de corbeille d'enfants
ni d'historique à inventer. Ces suppressions sont des éditions de contenu,
pas l'ouverture de la suppression d'entités.

POST et PATCH de l'entité gardent leur contrat 4A : `sources`, `relations`,
`relations_from` et `relations_to` imbriqués restent explicitement refusés.
Adapter leur message pour orienter vers les nouvelles routes, sans annoncer
que ces écritures sont encore indisponibles.

## Sécurité : droits de la fiche porteuse

JWT uniquement, groupes relus selon le mécanisme existant. Réutiliser la
sélection privée et `is_privately_editable`, sans nouvelle matrice :
contributeur sur ses draft/rejected ; relecteur sur tous les
draft/rejected/submitted ; administrateur sur tous les statuts non supprimés.
`is_staff` seul n'accorde rien ; superutilisateur actif et cumul inchangés.

Sans JWT utilisable : 401 ; sans rôle métier : 403 ; parent absent, supprimé
ou non lisible : 404 ; parent lisible mais non modifiable : 403.
Chercher l'enfant dans ce parent, jamais par son seul id : enfant inexistant
ou appartenant à une autre fiche, même 404. Aucun transfert de rattachement.
Pour une relation existante, conserver aussi le filtre `private_relations` :
une relation vers une cible masquée reste inaccessible, en 404, même si son
id est deviné. Aucune fuite du nom, de l'UUID ou du statut de la cible.

Une opération atomique par requête ; verrouiller la fiche porteuse puis
réévaluer ses droits sur cet état, avant de charger/modifier son enfant.
Relire l'état courant de l'enfant dans cette transaction pour le PATCH.
Ne pas changer le statut, propriétaire, type, profil ou marques éditoriales
du parent. Les écritures d'enfants n'ajoutent pas de mise à jour automatique
de `Entity.updated_at`, comme les éditions directes actuelles des enfants.
Pas d'ETag, de verrouillage optimiste ou de promesse générale hors API.

## Contrats d'entrée

**Source** : uniquement `label`, `kind`, `url`, `accessed_at`, `notes`.
POST exige label et kind ; autres valeurs selon le modèle. PATCH partiel :
champs omis conservés, chaîne vide acceptée seulement si le modèle le permet,
null uniquement pour accessed_at. Respecter choix, longueurs, label non blanc,
format URL/date. Réutiliser les validations Django/DRF ; pas de requête réseau
pour vérifier une URL. Pas d'unicité ajoutée : les doublons de sources sont
permis par la décision existante.

**Relation** : uniquement `kind`, `to_entity`, `to_name`, `notes`.
POST exige kind et exactement une cible : UUID existant en to_entity, ou
nom non blanc en to_name. Même règle sur l'état final d'un PATCH, combinant
champs actuels et fournis. Exemple de résolution explicite d'un nom :
`{"to_entity": "<uuid>", "to_name": ""}` ; inversement, fournir
`to_entity: null` et un to_name non blanc. Ne pas effacer l'autre forme
implicitement. Champ omis conservé ; aucune résolution automatique par nom.

Une cible résolue doit être privée lisible et non supprimée pour l'appelant,
sans devoir être modifiable. UUID inconnu, supprimé ou interdit : même erreur
400 de référence, sans détail révélateur. Une auto-relation est refusée.
Respecter les choix de kind et les contraintes d'unicité existantes, en
excluant la relation elle-même lors d'une modification. Conserver le sens
orienté des liens : `A — parent → B` signifie « B est un parent de A ».
Pas d'inverse, de réciproque ou de contrôle des cycles ajouté.

Refuser explicitement les clés inconnues et les champs imposés par le serveur
(`id`, `entity`, `entity_id`, `from_entity`, `from_entity_id`, etc.). Le parent
provient uniquement de l'URL. Une erreur de validation donne 400 sans mutation.
Réutiliser full_clean et les validateurs appropriés ; ne pas recopier les
contraintes du modèle dans une seconde politique. Ne pas convertir toutes
les exceptions en 400, ni laisser une violation connue de contrat en 500.

## Sobriété et vérifications

Choisir l'intégration Django/DRF la plus simple compatible avec ces routes.
Pas de bibliothèque de routes imbriquées, framework CRUD générique, couche
repository, fabrique ou hiérarchie spéculative. Factoriser seulement ce qui
est réellement partagé ; ne pas recopier les vues ou serializers d'entités.
Pas de changement de schéma, migration, import ou configuration JWT.
Conserver l'accès de 20 minutes en développement et de 5 hors debug.

Tests orientés comportements : parcours POST/PATCH/DELETE des deux enfants,
champs omis, validations du modèle, état final des cibles, unicité des relations
mais doublons documentaires permis, rattachement serveur, id sous mauvais
parent, permissions et absence de fuite. Couvrir la réévaluation sous verrou
et l'absence de mutation en cas d'échec. Réutiliser les fabriques et garanties
existantes ; ne pas refaire pour chaque route toute la batterie JWT ou tous
les cas de la matrice déjà testée. Préserver les refus d'écritures publiques,
PUT et DELETE d'entité. Contrôler le contrat OpenAPI réellement ajouté.

Pendant le travail, tests ciblés ; un seul passage complet final sur l'état
fonctionnel livré, sans ajouter systématiquement toutes les suites de
régression juste avant. Après correction, relancer les contrôles affectés ;
motiver une nouvelle suite complète par le risque, pas le format ou le rapport.
Exécuter check, contrôle d'absence de migrations, validation spectacular
sans avertissement, Ruff/format et mypy sur le périmètre affecté.

Actualiser OpenAPI et les passages concernés du README en anglais : routes,
entrées, réponses, droits, suppressions d'enfants et limites. Docstrings
anglaises courtes ; commentaires français seulement pour les choix utiles.
Ne pas créer de comptes ou données dans la base de développement, ne pas
lire les secrets et ne pas toucher au corpus ni à la fiche des exercices.

## Remise

Rapport factuel court : `docs/reports/phase-4-increment-4b-sources-relations.md`,
avec copie identique dans le scratchpad et chemin final. Indiquer le périmètre,
les choix concrets, fichiers, résultats réellement exécutés, limites et vrais
blocages. Ne pas répéter tout le prompt. Pas de guide Postman/ORM, de mise à
jour de roadmap ou d'ADR, de clôture Git ou de livrable secondaire : Codex
s'en charge après revue. Ne pas ouvrir les transitions, la suppression ou la
restauration d'entités, ni commencer l'incrément suivant.
