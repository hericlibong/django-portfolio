# Administration technique et future interface éditoriale

Décision de l'auteur confirmée le 1er novembre 2026, à la clôture de la phase 4.

## Frontière maintenue, sans nouveau chantier

Django Admin est réservé à l'administrateur technique de l'API : comptes,
groupes et maintenance. Il n'est pas l'interface de contribution des utilisateurs.
Les contributeurs, relecteurs et administrateurs éditoriaux auront ultérieurement
une interface simple, utilisant les routes de l'API et leurs droits existants
pour créer, corriger et faire évoluer les fiches. Ne pas construire cette
interface dans la phase 5 ; ne pas rouvrir cette séparation comme une décision
en attente à chaque incrément. Son emplacement dans la livraison future reste
à planifier, sans élargir automatiquement la V1 API.

## Limites actuelles du back-office, acceptées pour avancer

Entity et le profil spécialisé sont deux lignes liées. L'API les crée ensemble.
La création seule dans le formulaire Entity de Django Admin peut laisser le
profil absent ; changer le type peut laisser l'ancien profil incompatible.
Un profil vide n'est pas un profil absent. L'API refuse les modifications
incohérentes plutôt que de réparer des données implicitement.

L'admin autorise aussi une suppression physique, contrairement à DELETE dans
l'API, qui ne fait que poser les marques de suppression logique.

Ces limites ne sont pas niées ni corrigées par la seule confiance accordée à
l'administrateur. L'auteur accepte leur report, avec usage éditorial normal par
l'API, et autorise la clôture de phase 4 sans nouveau développement ou exercice.
Ce choix n'est pas une certification de préparation complète à la production.

## Solutions simples à traiter séparément, pas maintenant

Avant d'élargir ou d'utiliser régulièrement les formulaires d'entités de
l'admin, prévoir des protections localisées de ModelAdmin/formulaires :

- retirer la création d'entités de l'admin pour la laisser à l'API, ou, si
  cette création devient un besoin explicite de maintenance, y réutiliser la
  création atomique du couple entité/profil ; aucun profil rempli n'est exigé ;
- empêcher le changement ordinaire de type d'une fiche existante ; une
  reclassification nécessitera une opération explicite distincte ;
- retirer ou encadrer la suppression physique d'entités, individuelle et en
  lot, plutôt que de la confondre avec la suppression logique de l'API.

Ce sont des solutions proposées à arbitrer et tester dans cette tâche future,
pas des protections déjà implémentées. Pas de changement du modèle ou de moteur
de workflow nécessaire pour les seules restrictions de formulaires.
Les données incohérentes éventuellement déjà créées relèvent d'une correction
ciblée et autorisée, pas d'une réparation automatique cachée.

## Suite immédiate

Phase 4 clôturée ; aucun Postman supplémentaire imposé, le parcours complet
est couvert automatiquement. Phase 5 engagée : étendre l'import pilote au
corpus de 44 figures, puis effectuer l'import réel après revue et simulation.
Les limites admin ne sont pas un prétexte à élargir l'import ou à créer l'UI.
