# Phase 6 — Incrément 1 : démarrage local Docker et PostgreSQL

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis AGENTS.md et
CLAUDE.md, puis la phase 6 de la roadmap du dépôt django-portfolio.
Ce prompt autorise Docker/Compose, auparavant hors périmètre. Il n'autorise
ni CI, couverture, refonte, nouvelle interface ni déploiement en production.
Phase 5 terminée, HEAD attendu au départ `b7560dc` (vérifier l'état réel).

## Livrable strict

Une installation locale reproductible : Dockerfile, docker-compose.yml,
.dockerignore et exemple d'environnement Docker sans secrets réels.
Deux services seulement : API Django et PostgreSQL, avec un volume propre
pour les données. Réutiliser Python 3.12 et PostgreSQL 16, déjà employés par
le projet ; pas de changement opportuniste de versions ou dépendances.

Le serveur Django de développement est acceptable pour ce lot local,
explicitement présenté comme tel. Pas de promesse de déploiement production,
reverse proxy, Kubernetes, Celery, Redis ou framework de scripts de lancement.
Si un besoin indispensable manque, le signaler avant d'élargir le périmètre.

## Isolement obligatoire

- Ne pas ouvrir/copier le `.env` des exercices ni réutiliser ses secrets.
  Garder un `.env.docker` distinct, ignoré par Git, préparé à partir d'un
  `.env.docker.example` à valeurs fictives. Les commandes Compose doivent
  choisir cet environnement explicitement, sans dépendance implicite au
  `.env` local. Un fichier de secrets Docker local peut être créé avec des
  valeurs aléatoires adaptées, sans les afficher ni les mettre au rapport.
- Relier Django au service PostgreSQL via le réseau Compose et le nom de
  service, jamais localhost ou host.docker.internal. Ne pas publier le port
  PostgreSQL sur Windows. Projet Compose/volume distincts du PostgreSQL local.
- Publier l'API sur `127.0.0.1:8001` pour conserver l'API des exercices sur
  8000 ; si le port est occupé, signaler sans arrêter de service personnel.
- La base locale afrimyth et ses 45 fiches ne doivent subir aucune connexion,
  migration, import, sauvegarde/restauration ou nettoyage pendant ce lot.
- Ne pas embarquer `.env*` réels, `.venv`, caches, `.git`, sauvegardes, rapports
  temporaires, corpus ou images externes dans le contexte/image. Conserver
  les exemples nécessaires sans vrai secret. Ne pas ajouter le JSON V2.
- Aucun compte ni corpus créés automatiquement au démarrage. L'API doit
  démarrer avec une base métier vide et pouvoir redémarrer sans perdre les
  tables ou réinitialiser les données existantes du volume.

## Comportement attendu

Installer uniquement les dépendances requises par l'application dans l'image.
Prévoir une attente bornée de disponibilité de PostgreSQL via le healthcheck
Compose, puis l'application des migrations avant le service HTTP. Employer
les mécanismes existants plutôt qu'une boucle maison ou un outil d'attente.
Assurer l'accès à health, Swagger, au schéma et aux ressources statiques
nécessaires au mode de lancement retenu. Ne pas changer les règles API/JWT
ou les réglages de la session locale pour faciliter la démonstration.

Mettre à jour la seule section Docker nécessaire dans le README anglais :
prérequis, préparation du fichier local, commande de démarrage, URL 8001,
commande d'arrêt sans suppression du volume, accès administratif facultatif
via commande manuelle. Les guides détaillés seront faits par Codex.
Actualiser seulement la mention Docker hors périmètre dans CLAUDE.md pour
éviter une contradiction ; CI et couverture restent au prochain incrément.
Nouvelles docstrings anglaises courtes, commentaires français utiles.

## Prérequis Windows et opérations manuelles

Constat Codex du 2 novembre 2026 : Docker Desktop est installé, CLI 27.4.0,
Compose v2.31.0-desktop.2, WSL par défaut en version 2 ; moteur Linux Docker
indisponible au contrôle. L'auteur doit ouvrir Docker Desktop et attendre
qu'il indique son moteur prêt avant les essais.

Vérifier l'état réel avec des commandes courtes. Si le moteur reste
indisponible, poursuivre la préparation des fichiers mais rapporter les essais
non exécutés. Ne pas boucler, réinstaller, mettre à jour ou modifier WSL/BIOS,
Windows, réseau, droits système ou services personnels. Une éventuelle action
interactive requise doit être décrite précisément avec son message réel.
L'auteur n'a pas à installer PostgreSQL sur Windows pour ces conteneurs.

## Vérifications proportionnées

1. Validation Compose sans sortie de valeurs sensibles (`config --quiet`
   ou équivalent). Pas de dump de l'environnement ni de logs de secrets.
2. Si moteur disponible : construction et démarrage isolés, DB healthy,
   migrations réellement appliquées, HTTP 200 et JSON attendu de health,
   Swagger et schéma accessibles, consultation publique vide en 200.
3. Redémarrage des services et contrôle du maintien des tables/migrations
   dans le même volume. Ne pas créer de donnée de démonstration pour cela.
   Arrêt/redémarrage simples ; jamais `down -v`, purge, reset ou suppression
   de volume. Ne pas relancer l'import du corpus.

Pas de nouveau test d'existence de Dockerfile, de test qui recopie du YAML,
ni de suite rôle/JWT. Les essais d'intégration conteneur sont les contrôles
pertinents si le code applicatif ne change pas : ne pas relancer les 1 126
tests pour des fichiers Docker et de documentation seuls. Si un changement
Python indispensable est autorisé, justifier et vérifier seulement son risque.
Pas de passage complet demandé par défaut dans ce lot.

## Remise

Un rapport court : `docs/reports/phase-6-increment-1-docker.md`, copie
scratchpad identique `rapport-pour-codex-phase-6-increment-1-docker.md`.
Lister fichiers, commandes/résultats réels, éventuels essais bloqués,
services laissés démarrés ou arrêtés et seules actions manuelles encore
nécessaires. Aucun secret ni valeur de fichier local dans le rapport.

Ne pas prétendre terminé un démarrage non essayé. Pas de guide séparé,
ADR, roadmap, nouvelle dépendance non justifiée, commit, push ou tag.
S'arrêter à la remise ; Codex assure revue, pratique et clôture Git.
