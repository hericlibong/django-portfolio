# Phase 6 — Incrément 4 : revue finale et préparation de la version

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis AGENTS.md, CLAUDE.md et
la phase 6 de la roadmap du dépôt django-portfolio. Le dernier commit publié
est `dbb44f3`. L'arbre contient les changements de l'incrément 3, dont les
corrections du README apportées par Codex. Prends cet état comme point de départ
et préserve ces changements.

## Revue ciblée

1. Vérifie le périmètre publiable en examinant les fichiers suivis, les ajouts
   prévus et les exclusions Docker/Git. Cherche les secrets réels, sauvegardes,
   corpus et fichiers locaux. Rapporte seulement les chemins concernés et le
   type de problème, sans recopier de valeur sensible.
2. Vérifie les réglages hors debug, dont l'exigence d'une clé Django et la
   durée JWT. Confirme que la durée allongée est réservée au développement.
   Vérifie les permissions et les requêtes uniquement si le diff touche leur
   comportement. Réutilise les preuves acquises dans les phases précédentes.
3. Relis le parcours d'une personne extérieure dans le README, de la
   présentation culturelle et encyclopédique à la première requête.
   Vérifie la cohérence des commandes, des rôles, des données fictives et des
   limites déjà documentées. Préserve les corrections du diagramme et des
   comptes ordinaires. Garde la présentation du projet avant l'installation.
4. Identifie les versions qui décrivent la livraison (métadonnées du projet,
   version OpenAPI et références documentaires pertinentes). Prépare leur
   alignement avec `1.0.0`, en distinguant les versions de dépendances et les
   conventions du contrat API. Signale tout changement de contrat avant de
   l'appliquer.

Effectue les corrections légères nécessaires dans ce périmètre et justifie-les.
Pour un défaut fonctionnel important, documente la reproduction et l'impact.
Les sujets reportés concernant Django Admin, Shango, Oya et les ambiguïtés du
corpus restent des limites documentées. Utilise les contrôles ciblés selon le
diff final, notamment le schéma si ses métadonnées changent. Les 1 126 tests
et la couverture de 98,20 % du dernier passage sont les preuves de référence.
Le passage complet de livraison aura lieu dans la CI après publication.

## Remise et publication

Rédige un rapport court dans `docs/reports/phase-6-increment-4-revue-finale.md`
et une copie identique dans le scratchpad. Indique les fichiers, constats,
corrections, vérifications effectuées et blocages éventuels. Aucun secret dans
le rapport. Arrête-toi à la remise, sans commit, push ni tag.

Codex relira le résultat, puis publiera les incréments 3 et 4 ensemble et
vérifiera la CI du commit exact de livraison. Le tag `v1.0.0` sera créé
seulement après une CI verte et une revue finale concluante.
