# Phase 5 — Incrément 1 : import complet, préparation sans écriture réelle

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses consignes, les ADR
0002 à 0005 et la phase 5 de la roadmap du dépôt voisin django-portfolio.
Phase 4 clôturée : ne pas refaire ses revues ou revenir sur l'admin. La frontière
acceptée est dans `docs/notes/admin-et-interface-editoriale.md` : back-office
technique, future interface éditoriale séparée, aucun de ces chantiers ici.

## Livrable strict

Étendre la commande existante `import_mythology_pilot` pour importer le corpus
complet avec une option explicite `--all`. Sans cette option, conserver le
pilote historique Oya/Anansi/Mokele-mbembe et ses garanties. Réutiliser le code
existant, sans recopier le fichier ou créer un moteur d'import générique.

Source externe, autorisée en lecture seule :
`C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json`.
Vérifiée par Codex : liste de 44 figures, SHA-256
`6D9C94CD21FCB8629D19B2C2401D31DD289C0499DD802F3564FCDFBB3A1A58BF`.
Vérifier l'état réel plutôt que supposer ce nombre si le fichier a changé.
Lire ce JSON ne permet ni de l'éditer, ni de copier le corpus/code/images
de `_audit` dans le dépôt API. Chemin uniquement via --input, jamais en dur
dans la commande. Ne pas lire de secret ou .env.

Contrat de la simulation :
`python manage.py import_mythology_pilot --input "CHEMIN_JSON" --all --dry-run`.
Tout --dry-run reste annulé transactionnellement ; sans --all, comportement
pilote inchangé. Pour cette intervention, écritures **en base de tests seulement** :
pas d'import, même simulé, dans la base de développement avant revue Codex.
L'import réel est l'étape suivante, pas un moyen de préparer la démonstration.

## Données, répétition et préservation

- Reprendre les correspondances déjà validées : type/profil exact, tableaux,
  pays/aires culturelles/ethnies, domaines en union, corrections orthographiques
  explicites. Pas de géocodage, déduction scientifique, fusion culturelle ou
  nouvelle règle métier. Signaler les valeurs non couvertes et les ambiguïtés
  plutôt que supprimer des données ou inventer un rattachement.
- Créer les référentiels nécessaires de façon déterministe ; aucune copie
  complète du JSON dans des fixtures. Tests autonomes sur quelques cas synthétiques
  représentant les différences réelles découvertes dans les 44 fiches.
- Nouvelles fiches en draft sans propriétaire/revue, avec le profil correspondant.
  Pas de publication, affectation automatique à un compte ou validation éditoriale.
- Résoudre variantes et relations après disponibilité des fiches de l'appel ;
  préserver les cibles absentes sous to_name. Pas de relation inverse automatique,
  de rapprochement flou, d'effacement de doublons éditoriaux ou de cycle interdit
  nouveau. Une cible supprimée ne devient pas une référence active par l'import.
- Conserver les exclusions, dont rendering et origin.pantheon, dans un bilan
  explicite. Ne pas ajouter de champs au modèle pour absorber un champ source.
- Réexécution sans doublon ni écrasement : identité déjà présente réutilisée
  pour résoudre les liens des nouvelles fiches, fiche existante intégralement
  préservée (contenu, profil, owner, status, notes et collections). Aucune
  restauration ou remise en draft d'une fiche existante, même supprimée.
- Conflit d'identité ou correspondance ambiguë : erreur claire et transaction
  annulée, pas de suffixe de slug, merge, --force, upsert ou réparation implicite.
  Signaler les profils manquants/incohérents déjà présents sans les corriger.
  Une mise à jour des anciennes relations du pilote doit être proposée à part,
  jamais cachée derrière l'idempotence de l'import.
- Une transaction par appel couvre tous les apports, référentiels compris.
  Ne pas appliquer les restrictions de complétude d'une publication à un brouillon.

Le bilan terminal doit rester lisible : lues/sélectionnées/créées/ignorées,
relations résolues ou citées par nom, transformations/exclusions et conflits.
Les 44 fiches du corpus ne signifient pas 44 lignes au total dans une base
qui contient déjà le pilote et des démonstrations. Aucun nouveau fichier
de bilan permanent ou système de synchronisation n'est demandé.

## Vérifications proportionnées et restrictions

Annoncer brièvement les écarts entre pilote et corpus et les fichiers concernés,
puis coder seulement l'extension nécessaire. Si une décision de transformation
manque, présenter les cas exacts avant de la prendre. Pas de refonte de l'API,
admin, serializers/views, architecture, JWT ou infrastructure de tests.
Pas de migration, bibliothèque, nouvelle interface ou chantier de phase 6.

Tests : --all, mode pilote conservé, deux passes, source incomplète/ambiguë,
rollback global, dry-run sans persistance, réexécution sans doublon, fiches
existantes préservées. Réutiliser les cas du pilote au lieu de tout recopier.
Contrôler que l'extension ne change pas les garanties type/profil ou la suppression.
Ne pas lancer des batteries de tests rôle/JWT sans modification correspondante.

Pendant le développement, tests ciblés et relances du seul contrôle affecté.
Avant livraison : check, absence de migration, Ruff/format/mypy sur le périmètre,
validation du schéma seulement si le contrat API a effectivement été touché
(ce qui n'est pas attendu). Un unique passage complet final ; ne pas faire
précéder ce passage de toutes les anciennes suites. Pas de suite concurrente,
relance d'une commande silencieuse ou messages/lectures d'attente en boucle.
Le passage précédent prenait environ vingt minutes : attendre son résultat
réel, rapport final non présenté comme terminé pendant qu'il reste en cours.

## Remise limitée

Rapport court : `docs/reports/phase-5-increment-1-import-complet.md`, copie
identique scratchpad `rapport-pour-codex-phase-5-increment-1-import-complet.md`.
Indiquer transformations réellement couvertes, limites/décisions requises,
commandes/résultats et durées, fichiers modifiés, aucune écriture réelle.
README anglais actualisé seulement pour l'option et les limites de l'import ;
docstrings anglaises courtes, commentaires internes français utiles.

Pas de guide Postman/ORM, document pédagogique, ADR, roadmap, export du corpus,
nouveau compte, démonstration en base, rapport secondaire, commit ou push.
Codex assure revue et documentation différée, simulation autorisée ensuite,
puis l'import réel après validation de l'auteur. S'arrêter à la remise :
ne pas lancer l'import complet réel ni les étapes suivantes de sa propre initiative.
