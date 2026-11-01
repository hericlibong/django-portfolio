# Phase 4 — Incrément 8 : consolidation finale, sans nouveau chantier

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, les ADR 0002 à 0005 et la phase 4 de la roadmap du dépôt voisin
`django-portfolio`. Incréments 1 à 7 validés et publiés ; départ API `f3c098f`,
1 109 tests collectés. Préserver les changements préexistants. Ne pas refaire
les incréments ni remettre leur contrat en discussion sans fait nouveau.

## 1. Objectif et limites du mandat

Vérifier que l'ensemble livré forme un parcours cohérent, compléter seulement
les garanties réellement manquantes et corriger les défauts démontrés.
**La consolidation n'autorise ni nouvelle fonctionnalité ni réécriture.**
L'absence de nouveau code métier peut être un résultat parfaitement correct.

Avant toute modification, donner un état des lieux bref dans la conversation :
point à contrôler → couverture existante → manque éventuel → correction ciblée
envisagée. Pas de nouveau fichier d'audit ou matrice exhaustive comme livrable.
Ne pas conclure à une absence de couverture sur la seule recherche d'un nom
de test : lire les scénarios existants avant d'en créer d'autres.

Seules les corrections localisées nécessaires au contrat validé et leurs
tests sont autorisés. Si un problème exige schéma/migration, nouvelle règle
métier ou permission, changement de contrat public, réarchitecture ou action
sur les données existantes, le signaler et demander l'arbitrage **avant** de
l'implémenter. Continuer les contrôles indépendants possibles ; ne pas masquer
ce problème derrière une clôture générale ou une nouvelle fonctionnalité.

## 2. Contrat acquis à conserver, pas à réinventer

- JWT et groupes : aucun droit éditorial par simple authentification/staff ;
  rôles cumulés, superutilisateur actif administrateur, groupes en base.
- Public : published et non supprimé uniquement ; aucun champ éditorial,
  note ou cible masquée divulgué, aucun mode privé activable par paramètre.
- Privé : sélection selon rôle/propriété, références filtrées, acteur serveur.
  Création en draft avec propriétaire et exactement le bon profil ; PATCH
  partiel sans transfert de propriété/type/statut, champs omis conservés.
- Sources/relations : écritures individuelles sous le bon parent, aucun accès
  transversal par id ; une seule forme de cible, pas d'auto-lien ou doublon
  interdit ; sources documentaires dupliquées permises selon le modèle.
- Contribution : submit/withdraw/resume, propriétaires ou reviewer/admin selon
  ADR 0002 ; blocage du PATCH et des enfants du contributeur en submitted.
- Modération : publish/reject reviewer/admin ; motif de rejet non blanc requis,
  publication avec note facultative ; reopen admin seul. Dates et acteurs
  serveur, revue conservée/effacée conformément aux incréments 5 et 6.
- Suppression logique selon sa matrice distincte du PATCH, restore admin seul,
  aucun effacement physique d'entité ; statut et contenu conservés. Un
  published restauré redevient public. Aucune liste de corbeille ni purge.
- Variantes : protection des références par les variantes actives, restauration
  de référence avant variante ; acquisition ordonnée des verrous sur les
  chemins API coordonnés, pas de nouvelle interdiction des cycles indirects.
- Refus : 401 sans JWT utilisable, 403 pour un rôle/droit insuffisant sur une
  ressource lisible, 404 uniforme pour absente/non lisible/supprimée selon la
  route ; restore refuse le non-admin avant recherche. Les violations de
  payload/état autorisé sont des 400, sans changement partiel ni dates déplacées.

Les anciennes conventions explicitement testées, notamment le comportement
d'un Bearer invalide sur une lecture publique, ne sont pas des nouveautés
à « corriger » ici. Une simulation séquentielle ne devient pas une preuve
multithread ; conserver la portée exacte des garanties de concurrence.

## 3. Vérifications bornées

### Parcours entre incréments et permissions

Examiner les tests existants de lecture, création, PATCH, enfants, transitions,
modération, suppression et verrous. Compléter uniquement un croisement de
droits ou un enchaînement essentiel réellement absent ; ne pas générer le
produit cartésien rôle × propriété × statut × action × corps.

Le parcours de référence est création avec profil → modification → soumission
→ refus des écritures contributeur → rejet motivé → reprise/correction →
resoumission → publication → disparition après suppression logique →
restauration publique → réouverture. Il peut être couvert par plusieurs tests
existants complémentaires ; ne pas le recopier en entier si cela n'apporte
aucune garantie supplémentaire. Si un trou d'intégration est réel, ajouter
un scénario synthétique court via les routes réelles, sans contourner les
transitions par des mises à jour directes du statut dans ce scénario.

Contrôler les 401/403/404 et la non-divulgation avec les tests présents,
sans refaire toute la batterie JWT. Conserver les protections de sécurité ;
pas de tests de structure interne, comptes inutilisés ou exclusions de champs
déjà prouvées par un ensemble exact.

### Administration Django : examen ciblé, pas une seconde application

Examiner les chemins d'écriture réellement exposés par `mythology/admin.py` :
création et modification d'entité/profil, type, état/revue et suppression.
Vérifier notamment si une écriture normale peut produire un profil absent ou
incompatible, et quelles garanties de l'API ne s'appliquent pas dans l'admin.

Le back-office utilise les permissions Django et reste un outil de confiance :
ne pas y dupliquer automatiquement toute la matrice JWT/groupes, ni confondre
un superutilisateur de maintenance avec un contributeur API. Les limites déjà
assumées hors API ne deviennent pas un motif pour créer ici un workflow admin,
des actions de modération, une corbeille ou des signaux.

Corriger une incohérence localisée contre une garantie déjà décidée seulement
si le comportement attendu est non ambigu, avec un test de formulaire/route
admin pertinent. Si le traitement impose de choisir qui peut éditer un statut,
reclasser un type, réparer un profil ou supprimer physiquement depuis l'admin,
exposer le cas et la décision requise plutôt que fixer une nouvelle politique
en silence. Ne pas réparer la base ou changer les comptes pour le démontrer.
Ne pas prétendre que les verrous API couvrent les écritures admin/SQL/import.

### Contrat documentaire

Contrôler les passages concernés de README/OpenAPI : opérations réellement
ouvertes, formes d'entrée et réponses, droits, notes/dates, public/privé,
suppression/restauration et limites. Corriger seulement les contradictions
factuelles, pas réécrire les textes pour le style ou remplir chaque route
d'exemples supplémentaires. Laisser visibles les limites effectives.

## 4. Interdictions explicites

- Aucun lancement de phase 5 : ni import des 44 entités, reprise du corpus,
  enrichissement documentaire, moteur de rendu ou nettoyage de données.
- Aucun endpoint, rôle, modèle, champ, migration ou dépendance nouveau ; pas
  d'inscription, logout/révocation, rotation JWT, limitation de débit, moteur
  de workflow, historique, audit permanent, notification ou worker.
- Aucun découpage général de serializers/views/schema, renommage en masse,
  harmonisation stylistique, traduction des docstrings existantes ou chantier
  « qualité » opportuniste. Leur maintenance reste une tâche distincte.
- Aucun changement d'infrastructure de tests pour gagner du temps, pas de
  nouveau système de fixtures, benchmark, couverture chiffrée ou stress test.
- Ne pas modifier JWT : 20 minutes en développement et 5 hors debug sont
  conservées pour les exercices. Le retour du réglage de développement à 5
  avant finalisation de l'API reste suivi par Codex, pas anticipé ici.
- Aucun secret/.env lu ou affiché, mot de passe ou jeton exporté ; aucune
  écriture dans la base de développement, compte/groupe/fiche fictive/corpus.
- Aucune suppression de protection ou de test utile au nom de la concision.
  Nouveaux commentaires/docstrings courts : anglais pour les docstrings et
  le produit, français pour les explications internes nécessaires.

## 5. Ordre obligatoire des contrôles

1. Lire l'existant et annoncer les seuls écarts réels. Pas de batteries de
   commandes pour « voir si tout passe » avant de cibler le travail.
2. Si correction : tests nouveaux/affectés seulement, puis lint/format/typage
   sur le périmètre. Pas de relance complète à chaque retouche.
3. Avant le résultat final : check, makemigrations --check --dry-run,
   spectacular --validate --fail-on-warn, Ruff/format et mypy. Le contrôle final
   peut être global une fois ; ne pas enchaîner versions ciblées/globales
   répétées sur le même état sans justification.
4. **Un seul passage complet final de pytest** pour l'état livré de la phase.
   Ne pas lancer séparément toutes les anciennes suites juste avant : elles
   y sont incluses. Si le premier passage échoue, corriger et relancer les
   contrôles affectés ; justifier tout second passage complet par un risque
   transversal, pas par une retouche de documentation ou un test obsolète.

Les 1 109 cas ont un coût réel : annoncer le lancement final puis attendre le
résultat avec le mécanisme adapté. Aucune suite concurrente contre la même
base, relance d'une commande silencieuse, boucle de lecture de fichier ou
messages d'attente répétés. Aucune réussite déclarée avant la sortie réelle.
Consigner l'ordre, les résultats et les raisons des relances, sans recopier
la sortie de toutes les commandes. Un contrôle en attente ou échoué doit
rester explicitement indiqué, pas remplacé par une promesse de réussite.

## 6. Livrables limitatifs et arrêt

Uniquement : éventuelles corrections nécessaires, tests utiles, mises à jour
ciblées README/OpenAPI et **un seul rapport factuel court** :
`docs/reports/phase-4-increment-8-consolidation.md`.

Rapport : garanties déjà couvertes/réutilisées, écarts réels et corrections,
résultats exécutés avec durée complète, limites/décisions en attente, fichiers
touchés et état Git. Pas de répétition du prompt ou d'affirmation « phase 4
validée » : la validation finale appartient à l'auteur après revue et pratique.
Une limite hors périmètre n'est pas automatiquement un blocage ; la distinguer
d'un écart qui empêche réellement la clôture.

Copie identique dans le scratchpad :
`rapport-pour-codex-phase-4-increment-8-consolidation.md`. Donner les deux
chemins après le résultat réel. Ne pas antidater le rapport.

**Aucun guide Postman/ORM, collection exportée, document pédagogique, ADR,
roadmap, fichier d'audit séparé, checklist supplémentaire, compte de démo,
commit ou push.** Codex prépare les explications dans `docs/verifications-code/`,
annonce le plan Postman avec ses objectifs, conduit les exercices nécessaires
et clôture après validation. Ne cocher aucune roadmap, ne clore aucune phase
et ne commencer aucun autre chantier après la remise.
