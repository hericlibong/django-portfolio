# Phase 6 — Incrément 2 : couverture et GitHub Actions

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis AGENTS.md, CLAUDE.md et le découpage de phase 6 dans la roadmap voisine. Vérifie l'état Git au départ (référence `9699006`). Ce lot ouvre la CI et la couverture. Il porte sur la configuration et les vérifications associées.

## Livrable

- Ajouter pytest-cov aux dépendances de développement. Mesurer la couverture du code applicatif avec la suite existante, en excluant tests et migrations de la mesure. Documenter le périmètre, le résultat et le seuil choisi à partir de cette mesure. Retenir un seuil raisonnable sous la mesure initiale, avec une marge expliquée.
- Ajouter un workflow GitHub Actions unique sur les pull requests et les pushes vers `main`. Utiliser Python 3.12 et PostgreSQL 16 comme service de CI. Configurer des identifiants réservés à cette base éphémère. Fournir à Django ses variables de test sans dépendre d'un fichier `.env` réel, du corpus V2, de Docker Desktop ou de la base Windows.
- Exécuter Ruff (lint et format), mypy, `manage.py check`, `makemigrations --check --dry-run`, la validation OpenAPI avec `spectacular --validate --fail-on-warn` et pytest avec couverture. Faire échouer la CI si un contrôle échoue. Réutiliser les configurations existantes et donner au workflow les permissions minimales.
- Documenter brièvement dans le README anglais la commande locale de couverture et les contrôles de CI. Actualiser la mention CI/couverture dans CLAUDE.md pour refléter le périmètre livré.

## Vérifications et limites

Vérifier les versions et la syntaxe des actions retenues dans leur documentation officielle. Effectuer les contrôles rapides, puis un seul passage complet local avec couverture sur une base de test isolée. Utiliser son résultat pour fixer le seuil. Vérifier ce seuil sur le rapport déjà produit, sans rejouer pytest pour ce réglage seul. Si le passage complet échoue, diagnostiquer et cibler la vérification suivante sur le problème constaté.

Préserver le code métier, Docker et les données locales. Aucun changement des règles d'authentification, des permissions ou du schéma métier. Aucun nouveau test uniquement destiné à relever le pourcentage. Si un défaut applicatif bloque la CI, le signaler avec la reproduction et l'impact précis. Garder un seul environnement de CI, sans matrice ni service supplémentaire. Utiliser la suite locale comme preuve locale. L'exécution distante sera vérifiée par Codex après revue et publication.

## Remise

Produire un rapport court dans `docs/reports/phase-6-increment-2-ci-couverture.md`, avec copie identique dans le scratchpad sous le nom `rapport-pour-codex-phase-6-increment-2-ci-couverture.md`. Indiquer les fichiers, les commandes et résultats réels, le pourcentage mesuré, le seuil et sa justification, ainsi que tout obstacle restant. Indiquer séparément le statut de la CI distante. Arrêter à la remise. Codex prend en charge la roadmap, la revue et la clôture Git. Aucun commit, push, tag ou livrable secondaire par Claude.
