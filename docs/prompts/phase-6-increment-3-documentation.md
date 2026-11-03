# Phase 6 — Incrément 3 : documentation et langue

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis AGENTS.md, CLAUDE.md,
l'ADR 0003 et la phase 6 de la roadmap de django-portfolio. Le dernier
incrément a validé Docker et GitHub Actions sur `dbb44f3`.

## README destiné à une personne extérieure

Retravaille le README existant en anglais. Commence par expliquer ce qu'est
Afrimyth : un projet culturel à vocation encyclopédique consacré aux
mythologies africaines et diasporiques. Explique ce qu'une fiche peut
documenter (figure, contexte culturel, relations, références), puis le rôle
de l'API dans la consultation et la contribution. Distingue cette ambition
du périmètre effectivement livré. Évite les affirmations d'exhaustivité,
d'autorité éditoriale ou de disponibilité publique du corpus.

Organise ensuite le parcours en sections distinctes :

1. **Découvrir** : à qui sert le projet, ce qu'il contient et ce qu'un lecteur
   peut consulter. Présente brièvement les rôles et le passage du brouillon à
   la publication, avec des mots accessibles.
2. **Installer** : prérequis techniques, répertoire de travail, configuration
   Docker, commandes de démarrage et d'arrêt, santé et Swagger sur 8001.
   Garde le démarrage hors Docker comme parcours distinct. Explique qu'une
   installation neuve a sa propre base vide et que l'import V2 nécessite
   un fichier externe. Appuie-toi sur les commandes déjà validées.
3. **Essayer** : proposer des exemples courts et reproductibles avec des
   données fictives. Indiquer les prérequis et le résultat attendu pour la
   lecture publique, la contribution, la relecture et l'administration.
   Utiliser les routes et permissions réellement livrées. Aucun identifiant
   réel ni corpus copié dans le dépôt.
4. **Comprendre le code** : résumer l'architecture, pointer vers OpenAPI
   et expliquer comment exécuter les contrôles. Conserver les détails
   techniques utiles en référence sans les imposer comme première lecture.

## Langue et vérifications

Traduire en anglais les docstrings françaises restantes dans le code et les
tests. Préserver leur sens et garder les commentaires internes et rapports
en français selon l'ADR 0003. Relever les libellés publics encore en français.
Une traduction localisée sans effet sur les règles peut être proposée avec
les vérifications ciblées correspondantes. Signaler séparément toute
modification qui toucherait le contrat public.

Vérifier les exemples contre les routes, paramètres et permissions du code.
Pour les docstrings, comparer l'AST hors docstrings et contrôler Ruff sur
les fichiers touchés. Réutiliser la CI verte comme référence et éviter un
passage complet local pour des changements documentaires. Aucun changement
de Docker, du workflow, des données métier ou du corpus.

## Remise

Un rapport court dans `docs/reports/phase-6-increment-3-documentation.md`,
avec copie identique dans le scratchpad. Indiquer comment un lecteur extérieur
peut comprendre le projet et suivre le premier parcours, les fichiers modifiés
et les vérifications réellement effectuées. Codex assure la revue éditoriale,
les ajustements et la clôture. Aucun commit, push ou tag par Claude.
