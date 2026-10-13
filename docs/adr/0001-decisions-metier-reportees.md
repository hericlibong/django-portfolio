# ADR 0001 — Décisions métier reportées

- **Statut :** accepté
- **Portée :** première version de l'API Django REST

## Contexte

Quatre questions relevées pendant l'audit ne bloquent pas la création des premières tables.
Les traiter maintenant retarderait le premier incrément sans réduire son risque technique.
Elles sont donc consignées ici pour ne pas être redécouvertes au milieu de l'implémentation.

## Décisions reportées

| Question | Pourquoi elle est reportée | Moment de reprise |
|---|---|---|
| Faut-il distinguer les héros mythiques des figures historiques ? | La V1 peut conserver les trois types existants. Si nécessaire, un champ de qualification pourra être ajouté sans réorganiser les profils. | Avant l'enrichissement éditorial du corpus ou l'ajout de filtres sur l'historicité |
| Faut-il conserver l'`alignment` inspiré du jeu de rôle ? | Garder temporairement un champ nullable n'impose aucune structure supplémentaire. La décision est éditoriale, pas nécessaire au fonctionnement de l'API. | Avant de publier ce champ dans l'API publique stable |
| Quelle langue employer pour le contenu ? | Le corpus V2 est en anglais et peut être importé tel quel. Une stratégie multilingue ne doit être ajoutée que lorsqu'un besoin de publication bilingue est confirmé. | Avant toute fonctionnalité d'internationalisation ou campagne de traduction |
| Comment représenter le genre des créatures ? | Un champ nullable accepte le corpus actuel sans introduire une taxonomie prématurée. | Avant de figer les choix de valeurs exposés dans le schéma public |

## Conséquence

Ces sujets ne doivent pas bloquer le modèle initial, le premier endpoint ni l'import pilote.
Toute décision définitive ultérieure fera l'objet d'une nouvelle ADR si elle modifie le contrat
public ou les règles métier.
