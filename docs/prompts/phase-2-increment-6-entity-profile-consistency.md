# Prompt Claude CLI — Phase 2, incrément 6 : création cohérente entité/profil

Travaille dans `C:\Users\heric\dev\afrimyth-api`.
Dépôt de pilotage : `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales, l'ADR 0002 section 6 et inspecte l'état Git.

Objectif : créer une entité et exactement son profil métier dans une seule
transaction, conformément à l'ADR. Aucun nouveau modèle, aucun endpoint.
L'API reste en lecture seule.

## 1. Une fonction métier explicite

Crée `mythology/services.py` avec une fonction typée et documentée :

`create_entity_with_profile(*, entity_data, profile_data=None) -> Entity`

`entity_data` contient les champs simples et clés étrangères de l'entité ;
`profile_data` contient seulement les champs du profil correspondant au type.
Un dictionnaire absent ou vide de profil doit créer le bon profil avec ses
valeurs par défaut, pas une entité sans profil.

Le type choisit le modèle de profil, sans paramètre de profil concurrent :

- `divinity` → `DivinityProfile` ;
- `hero` → `HeroProfile` ;
- `creature` → `CreatureProfile`.

Implémentation attendue, sobre :

1. vérifier le type et les données acceptées ;
2. dans `transaction.atomic()`, construire l'entité, appeler `full_clean()`,
   puis l'enregistrer ;
3. construire le profil sélectionné en imposant sa relation à cette entité,
   appeler `full_clean()`, puis l'enregistrer ;
4. retourner l'entité créée.

Ne laisse pas les données du profil remplacer `entity` ou `entity_id`, ni
imposer un `id`/`pk` existant. Cette fonction crée : elle ne doit jamais
modifier une fiche ou un profil préexistant. Refuse les champs inconnus ou
hors périmètre avec une `ValidationError` lisible, en anglais, plutôt qu'une
écriture partielle. Ne modifie pas les dictionnaires fournis par l'appelant.

Les M2M, relations entre figures et sources ne sont pas traités dans cette
fonction. Refuse leur passage dans `entity_data` et explique cette frontière.
L'import pourra l'appeler dans sa propre transaction, puis rattacher ces
collections, sans compromettre le couple entité/profil.

N'absorbe pas une exception à l'intérieur de la transaction pour retourner
une entité incomplète. Une erreur de validation ou d'enregistrement doit
annuler les deux créations et rester visible pour l'appelant.

## 2. Portée de la garantie

Le service garantit le résultat de ses propres appels réussis : une nouvelle
entité et exactement le profil correspondant. Il ne garantit pas toutes les
écritures ORM possibles, ni les changements de type après création.

Ne surcharge pas `save()`, ne crée pas de signal, de trigger PostgreSQL,
de bibliothèque polymorphe, de repository ou d'architecture de services.
Les validations existantes des profils restent utiles ; conserve-les.

L'instruction locale interdisant les couches de services anticipées ne bloque
pas cette fonction : elle est explicitement demandée ici et prévue par l'ADR.
Actualise seulement cette règle dans `CLAUDE.md` pour autoriser les fonctions
métier concrètement demandées, en conservant le refus des abstractions
spéculatives. Pas de refonte générale de ce fichier.

La fonction est une interface interne pour l'import puis le CRUD. Elle n'est
pas un serializer HTTP, une authentification ou un contrôle d'autorisation :
ne branche pas directement un JSON client arbitraire sur ses arguments.
N'ajoute pas les rôles, transitions, permissions ou l'attribution automatique
du propriétaire. Le statut par défaut reste `draft`.

## 3. Existant et administration

Aucune modification de schéma ni migration ne devrait être nécessaire.
Inspecte les données existantes en lecture seule et indique dans le rapport
si une entité manque de profil ou possède un profil incohérent. Ne répare pas
automatiquement Shango ni aucune autre fiche : aucun changement persistant
en base de développement n'est demandé ici.

L'administration existante et les écritures ORM directes peuvent encore
contourner ce service : documente-le clairement. Ne refonds pas les formulaires
admin et ne branche pas maintenant le futur CRUD. L'import pilote devra
utiliser cette fonction pour les créations ; la phase 4 devra couvrir les
autres chemins d'écriture et les changements de type.

Ne remplace pas toutes les factories existantes par le service : certains
tests ont volontairement besoin d'entités sans profil ou de données invalides
pour vérifier des limites. Ajoute les tests du service dans un module dédié.

## 4. Tests utiles

Couvre notamment :

- les trois types : entité créée, bon profil unique, aucun des deux autres ;
- données du tronc commun et du profil correctement conservées en base ;
- profil vide ou absent produisant un profil avec ses valeurs par défaut ;
- type absent/inconnu et champs de profil incompatibles refusés ;
- impossibilité de détourner le rattachement ou de réutiliser un identifiant
  existant pour écraser une ligne ;
- échec de validation du profil après enregistrement de l'entité : aucun
  résidu créé, les données préexistantes préservées ;
- unicité du nom/type ou du slug respectée ;
- données d'entrée non modifiées et champs M2M hors périmètre refusés ;
- non-régression des tests existants et du contrat de lecture.

Utilise des données fictives et la base de test. Ne crée pas de démonstration
persistante. Les tests doivent vérifier le résultat stocké, pas seulement
l'état en mémoire ou la présence d'un décorateur `atomic`.

## 5. Contrôles et rapport

Exécute sur la version finale : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`, `pytest`, `ruff check .`,
`ruff format --check .` et `mypy .`. Si une correction intervient, relance
uniquement les contrôles affectés. Ne révèle aucune valeur du `.env`.

Crée un rapport français :
`docs/reports/phase-2-increment-6-entity-profile-consistency.md` dans le dépôt API,
avec copie identique dans ton scratchpad :
`rapport-pour-codex-phase-2-increment-6-entity-profile-consistency.md`.

Indique les fichiers changés, la signature exacte et un petit exemple d'appel
avec données fictives, les garanties et limites, l'état des fiches existantes
sans les modifier, les contrôles et le nombre de cas exécutés, les divergences
éventuelles et l'état Git final. Indique le chemin absolu de la copie temporaire.

Ne coche pas la roadmap et ne committe/pousse pas cette implémentation avant
notre revue. Après validation pédagogique, Codex prend en charge directement
la clôture documentaire et Git.
