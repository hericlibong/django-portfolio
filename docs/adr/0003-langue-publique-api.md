# ADR 0003 — Langue de travail interne et langue publique de l'API

- **Statut :** accepté
- **Date :** 16 octobre 2026
- **Portée :** `afrimyth-api`, son contrat public, son corpus et sa documentation
- **Résout :** la question de la langue du contenu, reportée par l'[ADR 0001](0001-decisions-metier-reportees.md)

## Contexte

L'ADR 0001 reportait la question « quelle langue employer pour le contenu ? »,
au motif que le corpus V2 est en anglais et peut être importé tel quel. Le
projet avance maintenant vers un contrat public : profils métier livrés,
endpoints de lecture exposés, schéma OpenAPI prévu en phase 3. Les noms de
champs, les valeurs techniques et les descriptions qu'ils porteront seront
difficiles à renommer une fois publiés.

Dans le même temps, le travail de conception et de pilotage se fait en
français : roadmap, ADR, rapports d'incrément, prompts destinés aux assistants
de code, échanges pédagogiques. Mélanger les deux registres sans règle
explicite produirait un dépôt incohérent, où un champ anglais porterait un
`help_text` français lu par les clients de l'API.

La décision ci-dessous ne choisit donc pas « une langue » : elle sépare la
**langue de travail** de la **langue du produit**.

## Décision

Le français reste la langue de travail interne. L'anglais est la langue
canonique du produit public et du corpus.

### Restent en français

- les échanges pédagogiques avec l'auteur du projet ;
- les prompts destinés aux assistants de code ;
- la roadmap et les documents de pilotage ;
- les ADR et les rapports internes ;
- les commentaires et les docstrings internes.

### Doivent être en anglais

- les identifiants du code ;
- les noms des champs exposés par l'API ;
- les valeurs techniques de l'API ;
- les descriptions publiques des champs et des fonctionnalités ;
- les messages d'erreur destinés aux clients de l'API ;
- le futur schéma OpenAPI et ses interfaces Swagger ou ReDoc ;
- le README public du dépôt API ;
- le corpus canonique, notamment les noms, les descriptions et les contenus
  éditoriaux importés ;
- les éléments d'interface destinés au public.

### Pas d'internationalisation maintenant

L'anglais est la langue canonique de l'API et du corpus. Une interface
bilingue pourra être réalisée ultérieurement, mais **aucune architecture i18n
n'est ajoutée maintenant** : ni tables de traduction, ni champs suffixés par
langue, ni activation de `django.middleware.locale`, ni `gettext` sur les
chaînes publiques. Introduire cette mécanique avant qu'un besoin de
publication bilingue soit confirmé coûterait sur chaque modèle et chaque
serializer, pour une fonctionnalité dont le périmètre n'est pas arrêté.

## Justification

Trois raisons, dans l'ordre d'importance.

**Le corpus est déjà anglais.** Les 44 entités de V2 sont rédigées en anglais,
y compris les récits et les listes d'attributs. Les importer tels quels évite
une campagne de traduction qui n'apporterait aucune valeur à la V1 et
introduirait des écarts de sens dans des contenus culturels.

**Un contrat public se renomme mal.** Un nom de champ, une valeur de choix ou
une clé de réponse entrent dans le code des clients. Fixer l'anglais avant la
publication du schéma OpenAPI évite une rupture de compatibilité ultérieure.

**La langue de travail n'a pas à contaminer le produit.** Écrire les
commentaires et les rapports en français sert la compréhension de l'auteur et
ne coûte rien aux clients de l'API, qui ne les lisent jamais.

## Conséquences

- L'import pilote puis l'import complet du corpus V2 conservent les contenus
  canoniques anglais ; aucune traduction automatique n'est appliquée.
- Les ADR, la roadmap, les prompts et les rapports existants **ne sont pas
  traduits**. Les anciennes ADR et la roadmap restent en français.
- Les fichiers sources existants ne sont pas traduits dans l'intervention qui
  acte cette décision. Seul le README public de `afrimyth-api` passe en
  anglais, parce qu'il est la porte d'entrée publique du dépôt.
- Le README bilingue de `django-portfolio` reste inchangé : c'est un document
  de portfolio, pas un produit.

### Dette identifiée — libellés publics encore en français

Certains éléments écrits en français sont techniquement visibles côté public et
devront être normalisés en anglais **avant** de stabiliser le contrat public et
la documentation OpenAPI :

- les `verbose_name` et `verbose_name_plural` des modèles ;
- les libellés des classes de choix — `EntityType`, `Gender`, `ThreatLevel` ;
- les `help_text`, par exemple celui de `Entity.category` ;
- les futurs messages d'erreur de validation, aujourd'hui rédigés en français
  dans `clean()`.

Ces libellés n'ont aucun effet sur les clients actuels : l'API de lecture
expose des valeurs techniques, pas des libellés, et l'administration Django
n'est pas un produit public. Cette normalisation **ne doit pas retarder la
phase 2**. Elle est inscrite comme tâche non cochée de la roadmap, à traiter
avant la stabilisation de l'API publique.

## Portée de cette décision

Cette ADR ne remet en cause aucune décision de l'ADR 0002. Elle ne modifie pas
le modèle de données, ne renomme aucun champ existant — les identifiants sont
déjà en anglais — et n'ajoute aucune dépendance.
