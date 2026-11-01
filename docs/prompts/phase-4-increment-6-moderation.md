# Phase 4 — Incrément 6 : décisions de modération

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, les ADR 0002 à 0005 et la phase 4 de la roadmap du dépôt voisin
`django-portfolio`. L'incrément 5 est validé et publié : départ API `87da2e9`,
961 tests collectés après revue (964 réussites avant simplification de trois
doublons). Préserver les changements préexistants ; aucune réinitialisation.

## 1. Périmètre : trois décisions seulement

Ajouter trois actions POST sous `/api/manage/entities/{slug}/` :

| Route | Transition | Acteurs autorisés |
|---|---|---|
| `publish/` | submitted → published | reviewer, administrator |
| `reject/` | submitted → rejected | reviewer, administrator |
| `reopen/` | published → draft | administrator uniquement |

Succès : 200, détail privé existant, références filtrées pour l'appelant.
GET/PUT/PATCH/DELETE sur ces routes restent fermés. Répéter une décision après
son succès donne 400 pour mauvais statut de départ, sans changer dates ou notes.
Ne pas autoriser draft → published, rejected → published ou published → rejected.
Les routes submit/withdraw/resume de l'incrément 5 restent inchangées.

## 2. Entrées, dates et revue : contrat précis

### Publication

Corps absent, `{}`, ou seul champ `review_notes` facultatif. Exemple :
`{"review_notes": "Reviewed and approved."}`. Note vide admise ; omission =
chaîne vide enregistrée, pas conservation implicite d'une ancienne note.
`null` n'est pas une note valide. Appliquer les validations normales de texte.

Écrire uniquement : status=published, reviewed_at=timezone.now(),
reviewed_by=compte appelant authentifié, review_notes=note validée et updated_at.
Conserver submitted_at tel qu'il était, même null sur une ancienne fiche
submitted préparée manuellement : ne pas inventer une date historique.

### Rejet

Objet JSON portant `review_notes` obligatoire, texte non vide et non composé
d'espaces. Exemple : `{"review_notes": "Please clarify the source of this claim."}`.
Corps absent, `{}`, note vide/blanche ou null : 400. Un motif aide le
contributeur à corriger ; ne pas inventer de code ou catégorie de rejet.

Écrire uniquement : status=rejected, reviewed_at=timezone.now(),
reviewed_by=compte appelant, review_notes=motif validé et updated_at.
Conserver submitted_at. Le propriétaire lit le rejet et utilise resume pour
revenir en draft ; resume conserve la revue, submit suivant l'efface comme
prévu dans l'incrément 5. Ne pas changer ces règles acquises.

### Réouverture

Corps absent ou `{}` seulement, comme withdraw/resume. Écrire status=draft,
submitted_at=null et updated_at. Conserver reviewed_at, reviewed_by et
review_notes de la publication précédente pour le travail de correction ;
la prochaine soumission les réinitialisera. Ne pas enregistrer l'admin qui
rouvre comme auteur d'une nouvelle revue : rouvrir n'est pas rendre une revue.
Pas de champ d'audit ou d'historique ajouté.

### Règles communes

Les notes ne sont reçues que sur publish/reject. Tout autre champ est refusé
explicitement : status, reviewed_at, reviewed_by, submitted_at, created_by,
contenu métier et clés inconnues ne sont jamais ignorés. Un corps JSON non
objet est refusé. Réutiliser les validateurs DRF/Django et le refus de clés
existant ; pas de double validation ou infrastructure de payload générique.

Propriétaire, type, contenu métier, profil, référentiels, sources/relations,
created_at et marques de suppression restent inchangés. Acteur et dates
proviennent toujours du serveur ; aucune modification partielle d'une demande
refusée, y compris updated_at. Ne pas ajouter de seuil de complétude : cette
publication est une décision humaine, pas un moteur de certification.

## 3. Sécurité, concurrence et visibilité

JWT uniquement, groupes évalués en base, cumul des rôles, superutilisateur
actif administrateur et is_staff seul sans droit : conventions inchangées.
La propriété n'accorde jamais publish/reject au contributeur ; le rôle reviewer
n'accorde jamais reopen. Relecteur/admin agissent aussi sur les fiches sans
propriétaire. Ne pas inventer une interdiction d'auto-revue ou un système
à deux validateurs : la matrice validée ne l'exige pas.

Ordre : authentification/rôle, sélection privée, droit propre à l'action,
statut de départ puis corps. Codes : 401 sans JWT utilisable ; 403 sans rôle
ou acteur refusé sur une fiche lisible ; même 404 pour inconnue, non lisible
ou supprimée (même admin) ; 400 pour mauvais statut ou données invalides.
Un acteur refusé ne doit pas recevoir un diagnostic sur le corps ou le statut
d'une fiche qu'il n'est pas autorisé à modérer.

Réutiliser le verrou et la transaction des transitions : visibilité, droit
de l'acteur et statut sont contrôlés sur l'état relu sous verrou, avant toute
mutation. Deux décisions concurrentes publish/reject sur la même soumission
ne peuvent pas réussir toutes les deux. Pas d'ETag, de version ou de promesse
générale pour les écritures hors API. Ne pas cacher toute exception derrière 400.

**Attention sécurité :** `may_transition()` admet actuellement le propriétaire
pour submit/withdraw/resume. Il faut distinguer explicitement les droits selon
l'action ; ne pas réutiliser ce droit général pour la modération. De même,
le droit de PATCH ne décide pas d'une publication. L'ajout des trois actions
dans les routes exemptées de `CanEditEntity` doit être compensé par le droit
de transition spécifique, jamais par un accès authentifié permissif.

Après publish : GET public liste/détail voit la fiche via la sélection publique
existante, sans exposer les champs éditoriaux ou les notes. Après reject :
la fiche reste absente du public. Après reopen : elle disparaît du public.
La visibilité des références structurées suit les règles déjà en place :
aucun nom de cible masquée recopié en to_name, aucun champ de revue public,
aucune influence du jeton ou d'un paramètre pour obtenir un rendu privé.
Ne pas réécrire les selectors publics ; les nouveaux statuts doivent suffire.

## 4. Restrictions impératives : ne pas élargir la tâche

- Pas de suppression/restauration d'entité (incrément 7), changement de type,
  PATCH ordinaire du statut/revue, inscription, notification, file de tâches,
  historique, nouvel acteur ou permission Django à attribuer.
- Aucun modèle, migration, bibliothèque, import, compte/groupe ou donnée de
  développement modifié. Ne pas publier la fiche fictive pour une démo :
  toutes les écritures de vérification restent dans la base de tests.
- Ne pas lire/afficher les secrets ou .env. Ne pas changer JWT, configuration
  de sécurité ou infrastructure de tests pour gagner du temps : garder
  l'accès de 20 minutes en développement, 5 hors debug.
- Réutiliser et étendre sobrement `transitions.py` et l'orchestration existante.
  Pas de FSM, registre générique de futurs workflows, repository, nouvelle
  classe de base ou suite de petits helpers qui ne servent qu'une fois.
- Ne pas découper globalement serializers/views/schema : la question de
  maintenance est reportée à une tâche distincte. Ajouter seulement les petits
  serializers d'entrée réellement nécessaires aux notes, sans déplacer tout
  le code. Préserver le contrat vide des trois premières actions.
- Docstrings anglaises courtes, commentaires français pour les choix utiles.
  Pas de longs blocs de documentation répétés en tête de chaque fichier.

## 5. Tests et exécution : pas de cumul rituel

Avant de coder, annoncer brièvement les fichiers envisagés et les contrôles.
Signaler une contradiction ou un vrai besoin hors périmètre avant de l'implémenter.
Ne pas demander une validation supplémentaire pour les étapes normales autorisées.

Tests orientés risques distincts, données synthétiques et fabriques existantes :

- Succès publish/reject avec acteur/date/notes côté serveur ; reopen réservé
  admin et conservation de la revue ; propriétés et contenu non modifiés.
- Refus contributeur et reviewer/reopen, mauvais départ, répétition sans
  nouvelle écriture, suppression/visibilité et corps/champs interdits.
- Motif de rejet obligatoire et notes de publication facultatives ; notes
  conservées par resume/reopen puis effacées à la prochaine soumission.
- Parcours submit → reject → resume → submit → publish → reopen ; droits
  d'écriture après chaque changement et absence de régression de withdraw.
- GET public avant/après publication et réouverture, absence de fuite des notes
  et d'une cible non publique. Un test d'intégration ciblé suffit : ne pas
  recopier toute l'ancienne suite de visibilité publique ou JWT.
- Simulation du changement de statut ou de droit entre sélection et lecture
  verrouillée ; distinguer ces simulations d'un véritable test multithread.
- OpenAPI : routes/méthodes, entrées différentes publish/reject/reopen,
  réponses privées, erreurs et règles documentées. Éviter de tester chaque
  phrase littérale ou la structure interne des fonctions.

Pas de matrice rôles × statuts × corps recopiée intégralement pour chaque
action, pas d'assertions d'exclusion doublonnant un ensemble exact de champs,
pas de comptes inutilisés systématiquement créés. Aucun quota arbitraire de
tests ; conserver les garanties de sécurité, d'intégrité et de contrat.

**Ordre obligatoire des contrôles :**
1. Pendant le développement : nouveaux tests ciblés ; anciens tests précis
   seulement si directement affectés par une modification ou un échec identifié.
2. Avant le passage final : check, makemigrations --check --dry-run,
   spectacular --validate --fail-on-warn, Ruff/format et mypy sur le périmètre.
3. Un unique passage complet final de pytest. Ne pas lancer séparément les
   anciennes suites transitions/lecture/création/PATCH/enfants avant ce passage.
   Elles y sont incluses ; ne pas refaire toute la suite OpenAPI pour une
   simple correction locale si des tests ciblés suffisent.
4. Après correction : seulement les contrôles affectés. Justifier toute seconde
   suite complète par un risque transversal, pas par rapport/README/format.

Les passages complets prennent environ quinze minutes : annoncer leur départ
et attendre leur résultat, sans relancer une commande silencieuse, sans suite
concurrente contre la même base, sans boucles de lecture de fichiers ou messages
répétés « j'attends ». Utiliser le mécanisme d'attente disponible ; si une
notification ne suffit pas, une vérification ponctuelle du processus/résultat
est permise. Aucune réussite déclarée avant la sortie finale réelle.

## 6. Documentation et remise : strict minimum utile

Actualiser les sections concernées de README et OpenAPI en anglais : trois
routes, notes, droits, dates, réponse privée, visibilité publique et limites.
Les notes internes ne deviennent pas publiques. L'incrément 7 reste annoncé
comme non livré. Ne pas refaire la documentation des incréments précédents.

Rapport factuel court : `docs/reports/phase-4-increment-6-moderation.md`.
Fichiers, décisions concrètes, résultats réels et durée complète, raisons des
relances, limites et état Git ; pas de copie de toutes les exigences du prompt.
Copie identique dans le scratchpad :
`rapport-pour-codex-phase-4-increment-6-moderation.md`. Fournir les deux chemins
une fois le résultat complet consigné. Si la suite finale tourne encore,
dire clairement « résultat complet en attente », ne pas présenter une livraison
terminée. Ne pas antidater le rapport : date réelle de l'intervention.

**Interdits de remise :** pas de guide Postman/ORM, collection exportée, ADR,
roadmap, rapport secondaire, donnée de démonstration, commit ou push. Codex
assure revue, petites corrections, pédagogie, exercices puis clôture après
validation. Ne pas commencer l'incrément 7 ou une réorganisation générale.
