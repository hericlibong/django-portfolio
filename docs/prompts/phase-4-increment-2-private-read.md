# Phase 4 — Incrément 2 : groupes et lecture privée

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, la phase 4 de la roadmap et les ADR 0002, 0003 et 0004 du dépôt
voisin `django-portfolio`. L'incrément JWT est validé et publié.

## Objectif et limites

Créer les trois groupes et permettre une lecture privée selon rôle et
propriété. Conserver les routes et réponses publiques existantes inchangées.
Pas de CRUD, transition de statut, suppression/restauration, inscription,
nouvelle durée JWT, champ `role` ou moteur générique de permissions.

Code Django/DRF simple : pas de validation JWT réécrite, de contrôle doublé
sans justification ni de base abstraite spéculative. Réutiliser la pagination,
la recherche et les filtres existants plutôt que les recopier. Les permissions
de liste doivent filtrer réellement le queryset : une permission objet seule
ne protège pas les listes. Les nouvelles docstrings et le produit public sont
en anglais ; commentaires, guide pédagogique et rapport restent en français.

## 1. Groupes et conventions validées

Crée par migration de données `contributor`, `reviewer`, `administrator`,
avec les modèles historiques et dépendances appropriées. Réutilise les groupes
déjà présents ; ne change ni leurs membres ni les comptes existants. Une marche
arrière ne doit pas supprimer des groupes préexistants ou leurs affectations
humaines ; une opération inverse sans effet, expliquée, convient.
Applique cette migration et vérifie-la. Ne distribue pas de permissions
Django globales inutilisées : les règles métier sont celles de l'ADR.

Conventions à appliquer sans nouvel arbitrage :

- un compte sans groupe métier est refusé sur les routes privées ;
- `is_staff` seul n'accorde aucun rôle métier ;
- un superutilisateur actif est traité comme administrateur même sans groupe ;
- plusieurs groupes cumulent leurs droits ; les groupes étrangers sont ignorés ;
- les rôles viennent de la base à chaque requête, pas de claims JWT ou d'un
  cache durable. Un changement de groupe doit agir sans réémettre le jeton.

## 2. Deux nouvelles routes, en lecture seule

- `GET /api/manage/entities/` : liste privée paginée ;
- `GET /api/manage/entities/{slug}/` : détail privé.

JWT d'accès requis sur ces routes, comme sur `me` ; une session seule ne
suffit pas. Permission globale de rôle puis sélection des fiches autorisées :

| Appelant | Fiches lisibles sur ces routes |
|---|---|
| Anonyme ou JWT refusé | 401 |
| Authentifié sans rôle métier | 403, même si la fiche demandée est publiée |
| Contributeur | Toutes les publiées et ses propres `draft`, `submitted`, `rejected` |
| Relecteur, administrateur, superutilisateur | Toutes les fiches non supprimées |

Les suppressions logiques sont exclues pour tous de ces lectures ordinaires.
Un propriétaire nul n'appartient à aucun contributeur ; les relecteurs et
administrateurs peuvent néanmoins lire ces fiches. Dans le détail, une fiche
non autorisée et un slug inexistant renvoient le même 404. Ne révèle pas son
statut ou son existence. La liste et son `count` ne comptent que les fiches
autorisées. Les filtres ne doivent jamais élargir cette sélection.

Conserve les 20 éléments par page, l'ordre stable, les cinq paramètres de
navigation et leur validation ; ils ne s'appliquent toujours qu'à la liste.
Toutes les écritures restent indisponibles pour tous les rôles, sans ajout
d'action administrative cachée. Le contrat public reste strictement public,
même si son appelant détient un jeton de relecteur ou de superutilisateur.

## 3. Réponses privées et références imbriquées

La liste privée reprend les cinq champs compacts et ajoute `status` et
`created_by` (identifiant du propriétaire ou null). Le détail reprend les
champs métier existants et ajoute `status`, `created_by`, `submitted_at`,
`reviewed_at`, `reviewed_by` (identifiant ou null), `review_notes`.
Pas de sérialisation complète des comptes, email, mot de passe ou permissions.
Les réponses publiques n'acquièrent aucun de ces champs.

Les serializers publics masquent aujourd'hui les références non publiques :
élargir seulement le queryset ne suffit donc pas. Ajoute une représentation
privée explicite en réutilisant les serializers communs ; ne copie pas tous
leurs champs et ne déclenche pas le mode privé via un paramètre HTTP.

Sur le détail privé, `variant_of` n'est exposé que si la cible est lisible par
l'appelant, sinon null. Une relation résolue vers une cible interdite est
entièrement omise, sans recopier le nom dans `to_name`. Les relations réellement
non résolues restent lisibles comme auparavant. Les cibles restent des
références compactes, jamais des détails récursifs. Les objets supprimés
restent masqués également dans ces références.

Prépare les jointures/préchargements pour ne pas faire une requête par fiche
ou cible, ni relire les groupes pour chaque élément. Partage des fonctions
concrètes de sélection/politique si utile, pas de framework. Une représentation
privée sans appelant valide doit rester restrictive ; aucun repli permissif.
Ne supprime pas les protections de référence du chemin public pour simplifier.

## 4. Tests, documentation et vérifications

Tests paramétrés sur rôle × propriétaire × statut, couvrant aussi sans groupe,
`is_staff` seul, groupes étrangers/multiples, superutilisateur sans groupe,
propriétaire nul et fiches supprimées. Vérifie 401/403/404, liste/count/détail,
les références imbriquées autorisées/interdites, le changement de groupe avec
le même JWT et l'impossibilité de changer la politique par un paramètre client.

Couvre le nouveau contrat de champs sans fuite sur le public, la migration
des groupes et sa non-destruction, la navigation réutilisée et les refus
d'écriture. Réutilise les suites existantes pour les garanties déjà acquises,
sans multiplier des tests identiques. Mesure les requêtes sur des fiches et
relations multiples pour déceler un N+1, sans budget arbitraire fixé d'avance.

Actualise OpenAPI et le README anglais : routes privées, sécurité Bearer,
champs éditoriaux privés et erreurs ; distinction entre lecture publique et
privée. Adapte les anciens tests qui affirmaient que seul `me` était protégé,
sans affaiblir les vérifications du contrat public. Préserve ReDoc fixé et
le démarrage corrigé par `TypingPatchConfig`. Actualise `CLAUDE.md` pour ne
plus annoncer qu'aucun accès privé n'existe après cet incrément.

Exécute `check`, `makemigrations --check --dry-run`, validation spectacular
avec `--validate --fail-on-warn`, pytest, Ruff, vérification de format et mypy.
Vérifie la migration appliquée ; après correction, seuls les contrôles affectés
sont à relancer. Ne lis ni n'affiche aucun secret du `.env` ou de Postman.

## 5. Guide pédagogique et rapport

Prépare un court guide français pour deux contributeurs et un relecteur :
affectation manuelle des groupes dans l'administration, fiches de démonstration
appartenant à chacun, comparaison des listes/détails publics et privés dans
Postman. Il s'agit d'étapes proposées, pas d'essais prétendument réalisés.
Les comptes et fiches de démonstration seront créés par l'auteur avec notre
accompagnement : ne les crée pas automatiquement en développement, ne change
pas les propriétaires ou statuts du corpus, ni les droits du compte existant.
Prévoir un nettoyage ciblé sans boucle ni bloc indenté si un exercice ORM
est proposé. Aucune donnée ou compte de démonstration dans une migration.

Ajoute les chemins des principaux extraits à expliquer : migration, contrôle
du rôle, sélection des objets et rendu des références. Les tests restent
autonomes avec comptes et données synthétiques. Les jetons restent dans le
Local Vault ; ne pas fournir d'export contenant des secrets.

Crée `docs/reports/phase-4-increment-2-private-read.md` et une copie identique
dans le scratchpad : `rapport-pour-codex-phase-4-increment-2-private-read.md`.
Indique fichiers, contrat, résultats réellement exécutés, nombre de cas,
requêtes mesurées, migration, préservation des comptes/corpus et état Git des
deux dépôts. Distingue les limites et les exercices manuels restant à faire.
Termine ta réponse par le chemin absolu de la copie temporaire.

Ne coche pas la roadmap, ne committe ni ne pousse avant la revue et les
exercices. Codex effectuera la clôture après validation, sans prompt Git séparé.
