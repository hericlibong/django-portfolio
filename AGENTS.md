# Consignes de pilotage — Django Portfolio

## Répartition Codex / Claude CLI

Règle validée par l'auteur du projet :

- Codex assure la conception, les explications pédagogiques, la préparation
  des prompts et la revue des rapports et du code.
- Codex prend directement en charge les guides et exercices Postman/ORM,
  la documentation d'accompagnement, les mises à jour de roadmap, les petites
  corrections et la clôture Git. Ne pas déléguer ces tâches à Claude CLI
  lorsqu'elles peuvent être réalisées dans ce cadre.
- Codex évalue l'impact des corrections relevées. Il applique directement les
  corrections légères, localisées et conformes aux décisions déjà validées,
  puis effectue les vérifications proportionnées et en garde une trace dans
  le rapport de l'incrément concerné.
- Les tâches complètes et les corrections importantes ou impactantes sont
  confiées à Claude CLI au moyen d'un prompt lisible par l'auteur avant son
  exécution. L'auteur transmet ce prompt à Claude CLI.

Les prochains prompts cantonnent Claude CLI à l'implémentation nécessaire,
à ses tests et vérifications, ainsi qu'à un rapport factuel court. Les
annotations OpenAPI ou autres éléments indissociables du contrat implémenté
restent dans ce périmètre ; les guides pédagogiques séparés sont préparés
par Codex après revue. Ne pas ajouter de livrables secondaires systématiques.

L'évaluation porte sur le risque et le périmètre, pas seulement sur le nombre
de lignes : une correction qui change le schéma, les décisions métier, les
permissions, le contrat public ou plusieurs composants reste du ressort de
Claude CLI. Une correction de validation circonscrite, de documentation ou de
test, sans changement de ces décisions, peut être prise en charge par Codex.

Les échanges, les prompts et la documentation interne restent en français ;
le produit et sa documentation publique suivent l'ADR 0003.
Les docstrings du code sont en anglais, y compris les docstrings internes.
Appliquer cette règle au nouveau code ; la traduction des docstrings françaises
existantes est reportée à une passe documentaire dédiée, sans changement fonctionnel.

## Vérifications utiles et proportionnées

- Pendant l'implémentation, exécuter les tests ciblés sur le comportement
  travaillé. Prévoir un seul passage complet final par état fonctionnel,
  sans enchaîner systématiquement toutes les suites ciblées, toutes les
  régressions puis toute la suite sur le même code inchangé.
  Ne pas lancer séparément les anciennes suites juste avant la suite complète,
  sauf modification pertinente ou échec identifié à vérifier entre les deux.
- Après correction, relancer seulement les contrôles affectés. Un nouveau
  passage complet doit être motivé par le risque ou des changements transverses,
  pas par une modification de rapport, de commentaire ou de format.
- La revue Codex contrôle l'utilité des tests : conserver les comportements
  distincts, la sécurité, l'intégrité et les contrats publics ; supprimer les
  vrais doublons et remplacer les assertions sur des détails d'implémentation
  lorsqu'une vérification de comportement apporte la même garantie.
- Ne pas fixer de quota arbitraire de lignes ou de tests. Les cas paramétrés
  doivent couvrir des risques distincts, pas multiplier artificiellement
  les décomptes. Consigner les simplifications et les résultats réutilisés
  dans le rapport de l'incrément, avant sa clôture.

## Exercices pédagogiques dans le terminal

La revue doit aussi contrôler la sobriété du code : réutiliser les validations
des bibliothèques, éviter les contrôles et fonctions auxiliaires redondants,
et ne garder que les commentaires qui expliquent un choix utile. Ne pas
supprimer une protection de sécurité ou un test de contrat au nom de la concision.

Proposer des commandes courtes à exécuter une par une. Éviter les boucles et
les blocs indentés, y compris `with`, même dans IPython. Prévoir une commande
de nettoyage ciblée lorsqu'un exercice crée des données de démonstration.

## Clôture Git automatique après validation

L'auteur autorise durablement les commits et les pushes normaux des incréments
validés vers les deux dépôts existants `hericlibong/afrimyth-api` et
`hericlibong/django-portfolio`. Ils font partie de la clôture : ne pas demander
une confirmation supplémentaire ni créer un aller-retour de prompt dédié.
Codex peut exécuter ces opérations secondaires directement.

Conserver l'identité `héric Libong <hericlibong@gmail.com>` comme auteur et
committer, sans trailer d'attribution d'assistant. Indexer explicitement les
fichiers du périmètre, effectuer un contrôle Git proportionné puis un push
normal. Réutiliser les résultats de tests déjà acquis si le code n'a pas
changé. Éviter les rapports répétitifs pour chaque commande Git ; indiquer
simplement le résultat et les SHA dans la clôture.

Cette autorisation ne couvre pas une publication d'implémentation en attente
de revue, un push forcé, une réécriture d'historique, un changement de
visibilité ou la suppression d'un dépôt. Signaler uniquement les vrais
obstacles, par exemple un conflit distant ou un problème d'authentification.
Préserver les modifications existantes et respecter les instructions propres
à chaque dépôt ; cette autorisation utilisateur remplace les anciennes
demandes systématiques de confirmation pour les commits et pushes validés.
