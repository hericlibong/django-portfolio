# Phase 4 — Incrément 7 : suppression logique et restauration

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lire `AGENTS.md`,
`CLAUDE.md`, ADR 0002 à 0005 et phase 4 de la roadmap du dépôt voisin
`django-portfolio`. L'incrément 6 est validé et publié : départ API `55353da`,
1 056 tests collectés après revue (1 061 réussites avant retrait de doublons).
Préserver toute modification préexistante, ne pas réinitialiser le travail.

## 1. Livrable strict : deux opérations

| Opération | Route | Réponse |
|---|---|---|
| DELETE | `/api/manage/entities/{slug}/` | 204 sans corps |
| POST | `/api/manage/entities/{slug}/restore/` | 200, détail privé existant |

DELETE est une **suppression logique** : aucune ligne Entity, profil, source,
relation ou collection n'est physiquement supprimée. Ne jamais appeler
Entity.delete(), QuerySet.delete() ou leur cascade pour cette opération.
Les DELETE existants de sources/relations gardent leur sens et leur contrat.

Corps absent ou `{}` seulement ; toute clé ou corps JSON non objet : 400,
pas d'entrée silencieusement ignorée. Pas de serializer à champs fictifs.
PUT reste fermé sur le détail ; restore accepte POST uniquement.
Ni suppression publique, ni route de purge, corbeille, liste ou GET de fiches
supprimées, ni paramètre include_deleted. Pas d'actions en lot.

## 2. Droits : matrice de l'ADR 0002, distincte du PATCH

Suppression d'une fiche non supprimée :
- contributor : seulement ses draft/rejected ;
- reviewer : tous les draft/rejected, pas submitted/published ;
- administrator : tous les statuts, y compris sans propriétaire.

Restauration : administrator uniquement, quel que soit le statut conservé.
Garder JWT seul, groupes en base/cumul, superutilisateur actif administrateur,
is_staff seul sans droit. Ne pas attribuer de groupe ou permission à un compte.

**Attention :** le droit de PATCH autorise un reviewer à modifier submitted,
pas à le supprimer. CanEditEntity/is_privately_editable et locked_record()
ne peuvent donc pas gouverner tels quels DELETE. Adapter explicitement ces
deux seules opérations, sans élargir les anciennes écritures ou transitions.

DELETE : 401 sans JWT utilisable ; 403 sans rôle ; sélection privée habituelle
avec 404 identique si inconnue, supprimée ou non lisible ; 403 si lisible mais
suppression interdite. Refuser le corps seulement après visibilité et droit.
Un second DELETE donne 404, sans remplacer l'acteur/date du premier.

Restore : authentifier et vérifier le rôle administrateur **avant** de
rechercher une fiche supprimée. Tout compte non administrateur reçoit 403
pour les slugs existants comme inconnus, sans renseignement d'existence.
Pour l'admin, recherche étroite par slug avec Entity.objects (manager complet) :
inexistante = 404 ; existante mais non supprimée = 400, sans mutation, y compris
après une restauration déjà réussie. Ne pas exposer globalement ce queryset
complet aux listes, GET, PATCH, enfants ou transitions de la vue privée.

## 3. Colonnes et visibilité

DELETE écrit seulement deleted_at=timezone.now(), deleted_by=compte appelant,
updated_at. Restore efface seulement deleted_at/deleted_by et actualise
updated_at. Statut, propriétaire, soumission/revue, contenu, profil, références
et enfants sont conservés. Les dates/acteurs viennent du serveur, jamais du Body.
Les deux managers existants restent inchangés : objects complet, active filtré.

Une fiche supprimée disparaît du public ET des lectures privées ordinaires,
même pour l'admin, et n'accepte aucune ancienne écriture ou transition.
Les références structurées pointant vers elle sont masquées selon les filtres
existants, sans copier son nom dans to_name. Aucun enfant ne devient orphelin.

Restore conserve le statut antérieur : si c'était published, la fiche redevient
publique ; sinon elle reste privée selon les droits existants. La restauration
n'est ni reopen ni une nouvelle publication, et ne remplace pas le relecteur.
Retourner le détail privé habituel, sans imposer de nouveaux champs dans les
représentations publiques ou privées. Pas d'historique ou nouvel indicateur.

## 4. Protection des variantes et atomicité

Conformément à l'ADR 0002, refuser DELETE en 400 si la figure possède une
variante directe **active** : variant_of pointe vers elle et deleted_at est
null, quel que soit le statut/owner/visibilité de cette variante. Une variante
logiquement supprimée ne bloque pas. Ne pas révéler le nom, l'id ou le nombre
des variantes privées dans l'erreur ; simple message anglais de protection.

Conserver aussi cet invariant à la restauration : si la fiche restaurée est
elle-même une variante et que sa référence est toujours supprimée, refuser
en 400 sans mutation ; l'admin doit restaurer la référence d'abord. Ne pas
effacer variant_of ou restaurer automatiquement d'autres fiches. Les simples
relations sortantes/entrantes ne constituent pas ce blocage, réservé à variant_of.

Transaction et verrou : relire visibilité, droit, état et protection des
variantes sur l'état effectivement protégé avant mutation. Une erreur laisse
tous les champs inchangés, y compris updated_at et marques de suppression.
Deux opérations concurrentes ne doivent pas réécrire les marques déjà posées.

Ne pas se contenter d'un exists() sans coordination : une création, un PATCH
de variant_of ou une restauration concurrente pourrait autrement réintroduire
une variante active sous une référence que DELETE vient de supprimer.
Vérifier les chemins d'affectation API existants ; seuls les ajustements
localisés indispensables à la synchronisation de ces écritures sont autorisés
ici. Choisir un ordre de verrous cohérent, ne pas introduire de verrou global,
réparation de corpus, moteur de retry ou migrations pour cela. Signaler un vrai
besoin plus large au lieu de l'implémenter en silence. La garantie vise les
chemins API coordonnés, pas les écritures arbitraires SQL/admin/import hors API.
Ne pas prétendre qu'un test simulé constitue un essai multithread.

## 5. Restrictions impératives

- Pas de suppression physique d'entité, suppression de compte, purge,
  nouvelle permission/rôle, changement de type, nouveau statut deleted,
  audit/history, notification, worker ou fonctionnalité de l'incrément 8.
- Aucun modèle/migration, nouvelle bibliothèque ou réorganisation globale
  de serializers/views/schema. Préserver les six transitions éditoriales.
- Réutiliser les filtres et fonctions pertinents ; une petite fonction
  métier concrète suffit. Pas de repository, FSM, classe de base générique,
  registre de futurs workflows ou duplication des matrices existantes.
- Ne toucher ni au corpus, ni aux comptes/groupes, ni à la fiche fictive pour
  préparer une démo. Toute écriture de vérification reste dans la base de tests.
- Aucun secret/.env lu ou affiché ; JWT inchangé (20 min en développement,
  5 hors debug). Pas de changement d'infrastructure de tests pour accélérer.
- Docstrings anglaises courtes, commentaires français expliquant les seuls
  choix utiles. Pas de longues explications répétées dans chaque fichier.

## 6. Vérifications utiles et ordre d'exécution

Annoncer brièvement fichiers et contrôles avant de coder. Signaler tout
écart au contrat avant une implémentation hors périmètre ; pas de demande
supplémentaire pour les étapes normales déjà autorisées.

Tests synthétiques orientés comportements :
- Suppression autorisée et refus reviewer/submitted et propriétaire/published ;
  dates/acteur serveur, statut et contenu/enfants conservés, second DELETE 404.
- GET public/privé et anciennes écritures inaccessibles après suppression,
  références masquées ; pas de fuite sur slug ou variante cachée.
- Restore admin seulement, mauvaise répétition sans mutation, restoration
  published de nouveau publique et brouillon restant privé, revue conservée.
- Référence bloquée par une variante active même non publiée ou non lisible ;
  variante supprimée non bloquante ; ordre de restauration référence/variante.
- Corps/champs interdits, réévaluation sous verrou, invariant face aux
  affectations variant_of concurrentes. Ne pas tester seulement la fonction
  de permission : vérifier l'intégration des routes et de leurs querysets.
- OpenAPI : méthodes exactes, 204 sans corps, 200 privé, JWT, droits,
  erreurs, statut conservé et distinction avec les DELETE d'enfants.

Réutiliser les fabriques ; pas de comptes inutilisés ou matrices intégrales
JWT × rôle × statut × corps recopiées pour chaque route. Écarter les cas heureux
déjà couverts des matrices de refus. Aucun quota de tests ; conserver les
protections de sécurité et les risques réellement distincts.

1. Pendant le développement : nouveaux tests ciblés, anciens tests précis
   seulement si directement affectés. Après correction locale, relancer le
   seul contrôle concerné, pas toute la suite OpenAPI par réflexe.
2. Avant passage final : check, makemigrations --check --dry-run, spectacular
   --validate --fail-on-warn, Ruff/format et mypy sur le périmètre affecté.
3. Un unique passage complet final pytest. Ne pas lancer séparément les
   anciennes suites lecture/création/PATCH/enfants/transitions/modération juste
   avant : elles y sont incluses. Justifier toute seconde suite complète par
   un risque transversal, pas une retouche de rapport ou format.

Le passage complet précédent prenait environ dix-huit minutes : annoncer
son départ, attendre le résultat réel avec l'outil adapté. Pas de relance
d'une commande silencieuse, de suites parallèles contre la même base, de
boucles de lecture ou messages répétitifs d'attente. Vérification ponctuelle
du processus/résultat permise si nécessaire. Ne pas annoncer une tâche finie
alors qu'une commande ou un monitor attend encore le résultat.

## 7. Documentation et remise, sans livrable secondaire

README/OpenAPI anglais : DELETE logique, restore, droits, état conservé,
visibilité, protection variantes et réponses. Actualiser les phrases obsolètes
annonçant DELETE comme indisponible, sans modifier le contrat des enfants.
La restauration d'un published redevient publique : le dire clairement.

Rapport **court et factuel**, daté de l'intervention réelle :
`docs/reports/phase-4-increment-7-soft-delete-restore.md`. Fichiers, choix
concrets, résultats réellement exécutés, durée complète, relances justifiées,
limites et état Git. Copie identique dans le scratchpad :
`rapport-pour-codex-phase-4-increment-7-soft-delete-restore.md`.
Donner les deux chemins après consignation du résultat complet ; si celui-ci
est en attente, le dire explicitement au lieu de déclarer la livraison terminée.

**Pas de guide Postman/ORM, explication pédagogique, export de collection,
nouvelle ADR, roadmap, rapport secondaire, commit ou push.** Codex assure la
revue technique, les petites corrections, les explications différées dans
`docs/verifications-code/`, la liste des contrôles Postman et leurs objectifs,
puis exercices et clôture après validation. Ne pas ouvrir l'incrément 8.
