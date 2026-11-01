# Incrément 7 — correction ciblée de l'ordre des verrous

À exécuter **après** la fin du passage complet déjà lancé pour l'incrément 7.
Ne pas l'interrompre ni lancer une seconde suite contre la même base.
Travaille dans `C:\Users\heric\dev\afrimyth-api`, selon ses consignes.
Préserver l'implémentation et les modifications non committées existantes.

## Constat de revue Codex

Le PATCH verrouille sa fiche dans `PrivateEntityUpdateSerializer.update()`,
puis appelle `lock_reference()` qui verrouille la cible `variant_of`.
Deux PATCH autorisés et simultanés peuvent donc suivre des ordres opposés :

- requête 1 : modifier A en désignant B comme référence, verrou A puis B ;
- requête 2 : modifier B en désignant A comme référence, verrou B puis A.

Chacune peut attendre la ligne détenue par l'autre : interblocage possible,
transaction annulée par PostgreSQL et erreur non gérée. Les cycles indirects
ne sont pas interdits dans le modèle actuel ; ne pas inventer une nouvelle
règle métier pour éviter ce problème. Le rapport affirme « aucun cycle
possible », ce que cet ordre ne garantit pas. Constat issu de lecture du
code, pas d'un essai multithread déjà exécuté par Codex.

## Correction attendue, et rien d'autre

Établir une discipline cohérente d'acquisition des lignes concernées, dans
un ordre déterministe partagé par les chemins API pertinents (création,
PATCH, restauration et suppression), plutôt que « fiche puis référence ».
Revalider sous verrou la fiche et sa référence ; ne pas utiliser un état
périmé ni acquérir tardivement une nouvelle ligne dans un ordre contradictoire.
Expliquer sobrement la solution retenue avant de la coder.

Conserver tous les droits, réponses, corps, filtres et protections de variantes
du prompt initial. Pas de suppression physique, nouvelle contrainte de cycle,
verrou global, moteur de retry, migration, bibliothèque, refactoring général
ou réparation de données. Si la correction exige une décision de contrat ou
un élargissement, le signaler avant de l'implémenter.

Ajouter une régression ciblée pour les affectations croisées et, selon le
besoin, la restauration liée. Privilégier un scénario transactionnel
multiconnexion borné par un timeout lorsque nécessaire pour démontrer l'absence
d'interblocage ; ne pas créer une batterie de stress ni prétendre qu'une
simulation séquentielle prouve une concurrence réelle. Aucun test ne doit
rester bloqué indéfiniment.

Ne relancer que les nouveaux tests et les contrôles directement affectés
(création/PATCH/suppression/restauration pertinents, typage, lint/format).
Pas de seconde suite complète automatique ; justifier d'abord tout besoin
transversal d'en relancer une. Pas de changement de documentation publique
sans changement effectif nécessitant cette documentation.

Actualiser **le rapport existant**
`docs/reports/phase-4-increment-7-soft-delete-restore.md` : résultat du premier
passage complet, constat Codex, correction réelle, vérifications complémentaires
et limites honnêtes. Retirer l'affirmation absolue non démontrée. Synchroniser
la copie scratchpad habituelle. Aucun nouveau rapport, guide, ADR, roadmap,
livrable pédagogique, compte ou donnée de développement ; aucun commit/push.
