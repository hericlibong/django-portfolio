# Phase 5 — Incrément 2 : simulation sur la base existante

## Objectif unique

Travaille dans `C:\Users\heric\dev\afrimyth-api`, selon ses consignes.
L'auteur demande explicitement cette intervention à Claude CLI.
La commande de l'incrément 1 est livrée. Il faut maintenant déterminer ce
qu'elle ferait dans la base locale déjà utilisée pour les exercices.
Ce n'est pas une nouvelle implémentation ni l'import réel.

Lis le rapport existant :
`docs/reports/phase-5-increment-1-import-complet.md`.
Réutilise les résultats acquis : 1 126 tests passants et simulation sur base
de tests vide. Ne recommence ni ces tests ni cette simulation.

## Source et commande autorisées

Source externe en lecture seule :
`C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json`.
Empreinte attendue SHA-256 :
`6D9C94CD21FCB8629D19B2C2401D31DD289C0499DD802F3564FCDFBB3A1A58BF`.
Nombre attendu : 44 entrées. Si la source diffère, signaler et arrêter
avant simulation ; ne pas la corriger ou la recopier dans le dépôt API.

Utilise l'environnement Python existant et les réglages locaux habituels,
sans ouvrir, afficher ou recopier les valeurs de `.env`. Le chargement
normal des réglages par Django est autorisé. Ne pas changer de serveur,
de base, de configuration ou d'identifiants pour contourner un accès refusé.
Si la base locale n'est pas accessible, rapporter cet obstacle et arrêter.

La seule opération d'import autorisée est :

```text
python manage.py import_mythology_pilot --input "C:\Users\heric\dev\django-portfolio\_audit\african_mythology_V2\src\data\mythology_data.json" --all --dry-run
```

Exécuter une seule fois. Conserver impérativement `--dry-run`.
Avant l'appel, confirmer par lecture ciblée que le chemin de simulation
annule bien sa transaction. Si cette garantie manque, arrêter sans exécuter.

## Contrôle avant / après, sans conservation de données privées

1. Relever l'état des données métier existantes : décomptes, identités des
   fiches du corpus déjà présentes et éventuels conflits de nom/type/slug.
   Comparer les identités réellement lues, pas une liste supposée de fiches.
2. Calculer en mémoire une empreinte déterministe du contenu des tables
   mythologiques, profils, référentiels, relations, sources et associations
   M2M, avec les marques éditoriales et de suppression. Aucun export de base,
   compte, mot de passe ou jeton ; ne pas conserver les contenus dans un fichier.
3. Exécuter la simulation puis relever à nouveau les mêmes décomptes et
   empreintes. Indiquer s'ils sont identiques. Le contrôle porte sur les
   données persistées ; des séquences PostgreSQL peuvent avancer malgré le
   rollback et ne doivent pas être réinitialisées pour masquer ce comportement.
4. Rendre un bilan concret : entrées lues, créations tentées puis annulées,
   fiches existantes ignorées/préservées, relations prévues et avertissements.
   Les chiffres d'une base vide ne sont pas transposables à cette base.

Un petit script temporaire hors dépôt est permis si nécessaire pour réunir
ces contrôles ; pas d'outil générique, nouveau test ou nouvelle commande.
Ne pas modifier de données par l'ORM, l'admin ou l'API. Ne pas lancer de
migration, nettoyage, affectation, restauration ou publication.
L'auteur évite les modifications concurrentes pendant cette manipulation ;
si les empreintes divergent, signaler et analyser avant toute autre action,
sans réparer, effacer ou relancer automatiquement.

## Traitement des limites déjà connues

`Congo` sans code, ethnonymes/domaines distincts, descendants non convertis,
absence de variantes et liens antérieurs du pilote préservés sont des limites
documentées, pas des demandes de nouvelles fonctionnalités.
Ne pas inventer de décision éditoriale, fusionner des lignes ou réparer un
profil existant. Un conflit qui fait réellement échouer la commande est
rapporté avec les identités concernées ; aucune correction dans ce lot.

## Restrictions et remise

- Aucun code applicatif modifié, refactoring, correction admin, UI ou dépendance.
- Aucun pytest, Ruff/mypy/OpenAPI ou contrôle global relancé : le code ne change pas.
- Aucun import sans `--dry-run`, même si la simulation réussit.
- Aucun commit, push, mise à jour de roadmap, guide ou livrable supplémentaire.
- Un seul rapport court :
  `docs/reports/phase-5-increment-2-simulation-base-existante.md`.
  Donner une copie identique dans le scratchpad sous le nom
  `rapport-pour-codex-phase-5-increment-2-simulation-base-existante.md`.

Le rapport indique les chiffres réels, résultat avant/après, commande,
durée, avertissements et éventuel blocage. Conclure simplement : import réel
préparable en conservant les limites connues, ou conflit précis à résoudre.
Ne pas déclarer que les 44 fiches sont disponibles dans l'API : toutes les
créations de cet appel doivent être annulées.
S'arrêter après la remise ; l'incrément 3 reste une opération distincte.
