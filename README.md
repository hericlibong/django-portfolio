# django-portfolio

**[Français](#version-française) · [English](#english-version)**

---

## Version française

Point d'entrée vers dix projets Django hébergés dans des dépôts séparés.
Chaque projet isole une compétence backend précise et se lit indépendamment des autres.

### Objectif

Ce dépôt ne contient aucun code applicatif. Il sert d'index à un parcours d'apprentissage Django construit par paliers, du CRUD authentifié jusqu'à une plateforme de données. Chaque projet répond à une question technique identifiée en amont — comment isoler les données d'un client, comment traiter une tâche longue sans bloquer la requête, comment garantir qu'un webhook rejoué ne facture pas deux fois — et se matérialise dans un dépôt autonome, avec ses tests, sa configuration et sa documentation. L'ordre est cumulatif : les projets tardifs réutilisent les acquis des précédents, et le dixième applique une finition de niveau production à l'ensemble.

### Projets

★ = projet destiné au portfolio public. Les autres dépôts restent publics mais ne sont pas mis en avant.

| #  | Projet                                    | Ce qu'il démontre                                                                     | Stack                                                  | Statut       | Lien    |
|----|-------------------------------------------|---------------------------------------------------------------------------------------|--------------------------------------------------------|--------------|---------|
| 1  | ★ API REST CRUD authentifiée              | Modélisation, authentification par jeton, permissions au niveau objet, schéma OpenAPI   | Django, DRF, SimpleJWT, drf-spectacular, PostgreSQL     | ⬜ Planifié  | à venir |
| 2  | Application multi-utilisateurs avec rôles | Groupes et permissions natives, cloisonnement des vues par rôle, admin personnalisée    | Django, `django.contrib.auth`, PostgreSQL              | ⬜ Planifié  | à venir |
| 3  | Import/export de données métier           | Lecture CSV/Excel, validation ligne à ligne, rapport d'erreurs exploitable par l'usager | Django, pandas, openpyxl, PostgreSQL                   | ⬜ Planifié  | à venir |
| 4  | ★ Backend avec tâches asynchrones         | Découplage des traitements longs, files d'attente, tâches planifiées, reprise sur échec | Django, Celery, Redis, celery beat                     | ⬜ Planifié  | à venir |
| 5  | Paiement intégré                          | Parcours de paiement en mode test, réception de webhooks, idempotence des traitements   | Django, Stripe (test), PostgreSQL                      | ⬜ Planifié  | à venir |
| 6  | Multi-tenant                              | Isolation des données par organisation, filtrage systématique, tests d'étanchéité       | Django, PostgreSQL, middleware dédié                   | ⬜ Planifié  | à venir |
| 7  | Django + front découplé                   | Frontière nette API/interface, rendu partiel ou client autonome, gestion des états      | DRF, HTMX ou SPA React                                 | ⬜ Planifié  | à venir |
| 8  | Recherche avancée                         | Recherche plein texte, tolérance aux fautes de frappe, pondération et classement        | Django, PostgreSQL (`tsvector`, `pg_trgm`)             | ⬜ Planifié  | à venir |
| 9  | ★ Plateforme data avec visualisation      | Ingestion de sources externes, agrégations ORM, restitution graphique                   | Django, PostgreSQL, Celery, bibliothèque de graphiques | ⬜ Planifié  | à venir |
| 10 | Finition production                       | Conteneurisation, chaîne CI/CD, supervision et journalisation — appliqué aux projets ci-dessus | Docker, GitHub Actions, gunicorn, nginx, Sentry | ⬜ Planifié  | à venir |

### Standards appliqués

Ces règles valent pour les dix dépôts, sans exception négociée projet par projet.

- **Tests** — pytest et pytest-django, couverture mesurée, tests de permissions et de cas limites systématiques.
- **Intégration continue** — GitHub Actions : lint, typage et tests à chaque push et chaque pull request.
- **Docker** — `Dockerfile` et `docker-compose.yml` fonctionnels ; le projet démarre en une commande.
- **Typage** — annotations de type sur le code applicatif, vérification par mypy.
- **Linting et formatage** — ruff pour l'analyse statique et le formatage, configuration partagée entre les dépôts.
- **Commits** — convention Conventional Commits, historique lisible, une intention par commit.
- **Documentation** — README par projet : objectif, décisions d'architecture, installation, lancement des tests.

### Méthode de travail

Le parcours est conduit en orchestration d'agents IA. La conception reste humaine : découpage du domaine, choix du modèle de données, arbitrages d'architecture et critères d'acceptation sont définis avant toute génération de code. Les agents produisent ensuite l'implémentation sur la base de ces spécifications, par itérations courtes et délimitées. Chaque sortie passe une relecture manuelle avant commit — vérification de la logique métier, des cas non couverts et de la cohérence avec l'existant ; rien n'est intégré sans avoir été lu. Les décisions d'architecture significatives sont consignées dans le dépôt concerné, avec l'alternative écartée et la raison de l'écart. La chaîne d'intégration continue sert de garde-fou automatisé, non de substitut à cette relecture.

### Contact

- GitHub — [github.com/hericlibong](https://github.com/hericlibong)
- Site — [hericlibong.github.io](https://hericlibong.github.io)
- LinkedIn — *(URL à compléter)*

---

## English version

Index of ten Django projects, each living in its own repository.
Every project isolates one backend skill and can be read on its own.

### Purpose

This repository holds no application code. It indexes a Django learning path built in stages, from an authenticated CRUD API up to a data platform. Each project answers a technical question chosen beforehand — how to keep one tenant's data out of another's reach, how to run a long job without holding the request open, how to make sure a replayed webhook does not charge a customer twice — and ships as a standalone repository with its own tests, configuration and documentation. The sequence is cumulative: later projects build on what earlier ones established, and the tenth applies production-grade hardening across the set.

### Projects

★ marks the projects intended for the public portfolio. The remaining repositories are public but not featured.

| #  | Project                            | What it demonstrates                                                                   | Stack                                            | Status      | Link |
|----|------------------------------------|----------------------------------------------------------------------------------------|--------------------------------------------------|-------------|------|
| 1  | ★ Authenticated CRUD REST API      | Data modelling, token authentication, object-level permissions, generated OpenAPI schema | Django, DRF, SimpleJWT, drf-spectacular, PostgreSQL | ⬜ Planned  | TBA  |
| 2  | Multi-user application with roles  | Native groups and permissions, role-scoped views, customised admin site                  | Django, `django.contrib.auth`, PostgreSQL        | ⬜ Planned  | TBA  |
| 3  | Business data import/export        | CSV and Excel parsing, row-level validation, error report an end user can act on         | Django, pandas, openpyxl, PostgreSQL             | ⬜ Planned  | TBA  |
| 4  | ★ Backend with asynchronous tasks  | Long jobs moved off the request cycle, queues, scheduled work, retry on failure           | Django, Celery, Redis, celery beat               | ⬜ Planned  | TBA  |
| 5  | Integrated payments                | Test-mode checkout flow, webhook handling, idempotent processing                          | Django, Stripe (test mode), PostgreSQL           | ⬜ Planned  | TBA  |
| 6  | Multi-tenancy                      | Per-organisation data isolation, enforced query scoping, leak-proofing tests              | Django, PostgreSQL, dedicated middleware         | ⬜ Planned  | TBA  |
| 7  | Django with a decoupled front end  | Clear API/UI boundary, partial rendering or standalone client, state handling             | DRF, HTMX or a React SPA                         | ⬜ Planned  | TBA  |
| 8  | Advanced search                    | Full-text search, typo tolerance, result weighting and ranking                            | Django, PostgreSQL (`tsvector`, `pg_trgm`)       | ⬜ Planned  | TBA  |
| 9  | ★ Data platform with dashboards    | External source ingestion, ORM aggregations, visual reporting                             | Django, PostgreSQL, Celery, charting library     | ⬜ Planned  | TBA  |
| 10 | Production hardening               | Containerisation, CI/CD pipeline, monitoring and logging — applied to the projects above  | Docker, GitHub Actions, gunicorn, nginx, Sentry  | ⬜ Planned  | TBA  |

### Engineering standards

The same rules apply to all ten repositories; none is waived on a per-project basis.

- **Tests** — pytest with pytest-django, measured coverage, permission and edge-case tests as a baseline.
- **Continuous integration** — GitHub Actions running lint, type checks and tests on every push and pull request.
- **Docker** — working `Dockerfile` and `docker-compose.yml`; the project starts with a single command.
- **Typing** — type annotations across application code, checked with mypy.
- **Linting and formatting** — ruff for static analysis and formatting, with configuration shared across repositories.
- **Commits** — Conventional Commits, readable history, one intent per commit.
- **Documentation** — a README per project covering purpose, architectural decisions, setup and how to run the tests.

### Working method

The work is run as AI agent orchestration. Design stays human: domain breakdown, data model, architectural trade-offs and acceptance criteria are settled before any code is generated. Agents then implement against those specifications in short, bounded iterations. Every output is reviewed by hand before it is committed — business logic, uncovered cases and consistency with existing code — and nothing is merged unread. Architectural decisions of consequence are recorded in the relevant repository, together with the option that was rejected and why. The CI pipeline acts as an automated safety net, not as a replacement for that review.

### Contact

- GitHub — [github.com/hericlibong](https://github.com/hericlibong)
- Website — [hericlibong.github.io](https://hericlibong.github.io)
- LinkedIn — *(URL to be added)*
