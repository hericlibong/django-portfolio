# Phase 5 — Incrément 3 : import réel du corpus

Travaille dans `C:\Users\heric\dev\afrimyth-api` selon ses consignes.
L'auteur demande l'import réel par Claude CLI. La simulation séparée de
l'incrément 2 est abandonnée : ne pas l'exécuter.

## Objectif et périmètre

Utiliser la commande livrée pour ajouter les fiches manquantes du corpus
dans la base locale existante, en brouillon, puis vérifier la réexécution.
Les contenus incomplets ou discutables restent corrigeables après import ;
ne pas imposer une complétude de publication à un brouillon.
Le modèle, les transformations et les exclusions restent ceux déjà livrés.
Aucun changement de code ni nouvelle décision éditoriale dans cette tâche.

Réutiliser `docs/reports/phase-5-increment-1-import-complet.md` et ses
résultats : 1 126 tests passants, 44 figures transformées en base temporaire.
Ne pas relancer pytest, Ruff/mypy/OpenAPI ou une simulation préalable.

Source externe en lecture seule :
`C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json`.
SHA-256 attendu :
`6D9C94CD21FCB8629D19B2C2401D31DD289C0499DD802F3564FCDFBB3A1A58BF`.
44 entrées attendues. Si la source diffère, arrêter et signaler avant écriture.

## Exécution, dans cet ordre

1. Utiliser Python et PostgreSQL locaux déjà configurés. Le chargement
   habituel des réglages Django est autorisé ; ne pas afficher/lire en clair
   `.env`, mots de passe ou jetons. Confirmer que la cible est la base locale
   attendue ; ne pas modifier de configuration ni basculer vers une autre base.
2. Relever les identités et l'état des fiches présentes pour pouvoir vérifier
   leur préservation : champs, profils, collections et marques éditoriales.
   Un relevé déterministe en mémoire suffit ; aucun export de comptes/secrets.
3. Faire une sauvegarde PostgreSQL complète de cette base avec les outils
   existants, dans un fichier horodaté hors Git, sous
   `C:\Users\heric\dev\backups\afrimyth-api\`. Ne rien écraser. Vérifier
   la réussite, la taille non nulle et la lecture du catalogue de l'archive
   (format custom avec `pg_dump`, puis `pg_restore --list`). Ne pas restaurer
   ni créer une autre base pour ce contrôle. Aucun secret dans les arguments
   affichés ou le rapport. Si la sauvegarde échoue ou l'outil manque, arrêter
   et signaler précisément, sans installer ni improviser une sauvegarde partielle.
4. Exécuter l'import réel, une fois :

```text
python manage.py import_mythology_pilot --input "C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json" --all
```

5. En cas de succès, contrôler les fiches source présentes, les profils des
   nouvelles fiches et leur état draft sans propriétaire, les relations et
   les limites annoncées. Comparer les anciennes fiches au relevé initial :
   aucune réécriture, affectation, suppression ou restauration autorisée.
   Le total global peut dépasser 44 à cause des démonstrations ; ne pas
   exiger 41 créations si plus de trois figures du corpus existent déjà.
6. Relancer exactement la même commande une seule fois pour contrôler
   l'idempotence : zéro nouvelle fiche, aucun nouveau lien ou référentiel,
   aucun changement du contenu existant. Ne pas répéter au-delà de ce contrôle.

La commande reste transactionnelle. Si elle échoue, signaler le conflit
exact et vérifier l'absence d'apports partiels. Ne pas corriger le JSON,
les identités ou les données par l'ORM, ni contourner les validations.
Ne pas restaurer automatiquement la sauvegarde ; signaler toute anomalie
de préservation avant une action supplémentaire.

## Limites conservées et remise

`Congo` sans code ISO, termes proches non fusionnés, descendants non convertis,
absence de variantes et anciennes relations du pilote inchangées restent
documentés. Ne pas inventer de liens ni compléter les anciennes fiches.
Les données de rendu/images restent exclues. Aucune publication automatique.

Un seul rapport court : `docs/reports/phase-5-increment-3-import-reel.md`,
avec copie identique scratchpad `rapport-pour-codex-phase-5-increment-3-import-reel.md`.
Donner le chemin de sauvegarde, les chiffres avant/après et des deux appels,
le contrôle de préservation, les limites et tout obstacle réel.
Pas de dump, contenu de comptes ou secret dans le rapport.

Aucun code, migration, nettoyage, compte, UI, guide ou autre livrable ajouté.
Ne pas modifier la roadmap, committer, pousser ou déclarer la phase close.
Codex assurera le suivi et les quelques lectures Postman avec l'auteur.
S'arrêter à la remise du rapport ou au premier obstacle empêchant l'opération.
