# Consignes de pilotage — Django Portfolio

## Répartition Codex / Claude CLI

Règle validée par l'auteur du projet :

- Codex assure la conception, les explications pédagogiques, la préparation
  des prompts et la revue des rapports et du code.
- Codex évalue l'impact des corrections relevées. Il applique directement les
  corrections légères, localisées et conformes aux décisions déjà validées,
  puis effectue les vérifications proportionnées et en garde une trace dans
  le rapport de l'incrément concerné.
- Les tâches complètes et les corrections importantes ou impactantes sont
  confiées à Claude CLI au moyen d'un prompt lisible par l'auteur avant son
  exécution. L'auteur transmet ce prompt à Claude CLI.

L'évaluation porte sur le risque et le périmètre, pas seulement sur le nombre
de lignes : une correction qui change le schéma, les décisions métier, les
permissions, le contrat public ou plusieurs composants reste du ressort de
Claude CLI. Une correction de validation circonscrite, de documentation ou de
test, sans changement de ces décisions, peut être prise en charge par Codex.

Les échanges, les prompts et la documentation interne restent en français ;
le produit et sa documentation publique suivent l'ADR 0003.

Aucun commit ni push sans demande explicite. Préserver les modifications
existantes et respecter les instructions propres à chaque dépôt.
