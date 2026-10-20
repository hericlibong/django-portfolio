# Prompt Claude CLI — Phase 2, incrément 5 : propriété et état éditorial

Travaille dans `C:\Users\heric\dev\afrimyth-api`.
Dépôt de pilotage : `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales et inspecte l'état Git avant d'agir.

Références : ADR 0002, section 7 (statuts, permissions et managers), ADR 0003
(langues) et roadmap. Les incréments précédents sont validés et publiés.

Objectif : préparer les champs de plateforme de `Entity`. L'API reste en
lecture seule. Les groupes et permissions, les services de transition, le
CRUD et la restauration appartiennent à la phase 4 ; ils ne sont pas demandés
maintenant. Le filtrage éditorial des seules fiches publiées appartient à la
phase 3.

## 1. Champs à ajouter à `Entity`

| Champ | Définition |
|---|---|
| `created_by` | FK `settings.AUTH_USER_MODEL`, `SET_NULL`, `null=True`, `blank=True` |
| `status` | `CharField(max_length=20, choices=EntityStatus.choices, default=EntityStatus.DRAFT)` |
| `submitted_at` | `DateTimeField(null=True, blank=True)` |
| `reviewed_at` | `DateTimeField(null=True, blank=True)` |
| `reviewed_by` | FK `settings.AUTH_USER_MODEL`, `SET_NULL`, `null=True`, `blank=True` |
| `review_notes` | `TextField(blank=True)` |
| `deleted_at` | `DateTimeField(null=True, blank=True)` |
| `deleted_by` | FK `settings.AUTH_USER_MODEL`, `SET_NULL`, `null=True`, `blank=True` |

`EntityStatus` est une classe `TextChoices` avec les valeurs et libellés
anglais : `draft` / `Draft`, `submitted` / `Submitted`, `published` /
`Published`, `rejected` / `Rejected`. Pas de statut `deleted` : la suppression
logique est portée séparément par `deleted_at`.

Utilise des `related_name` distincts et lisibles pour les trois relations
utilisateur. Le propriétaire est facultatif pour les données historiques et
les imports. Supprimer un utilisateur ne doit supprimer aucune fiche.

Ne remplis automatiquement aucun auteur, relecteur ou horodatage métier.
`created_at` et `updated_at` conservent leur comportement existant. Ne crée ni
`role` sur l'utilisateur, ni `is_deleted`, ni second champ de vérification
redondant. Les groupes Django seront créés en phase 4.

La validation Django vérifie les choix de statut ; elle ne constitue pas une
machine à états. Aucun `save()` surchargé ou signal pour gérer les transitions.
Ne pose pas de contraintes liant maintenant acteur, date et statut : supprimer
un utilisateur doit pouvoir laisser une date historique avec un acteur nul.

## 2. Managers et API

Conformément à l'ADR, déclare explicitement :

- `objects = models.Manager()` en premier : manager complet, comprenant les
  fiches supprimées logiquement ;
- `active` comme second manager, filtrant uniquement
  `deleted_at__isnull=True`.

Garde le manager complet comme manager par défaut. Ne filtre pas le manager
de base ni les relations implicitement. N'ajoute pas de bibliothèque de soft
delete ni de comportement de suppression personnalisé.

La vue d'entités utilise maintenant `Entity.active` pour la liste et le
détail, en conservant tous les `select_related` et `prefetch_related` actuels.
Une fiche racine marquée supprimée ne figure plus dans la liste et son URL
renvoie 404. Le filtrage de `active` ne dépend pas de `status`.

Conserve les clés actuelles des serializers liste/détail. Les nouveaux champs
de plateforme restent internes pour cet incrément : n'expose pas les comptes,
leurs emails ou noms, les notes privées de relecture, ni les champs de
suppression dans l'API publique. Le contrat destiné aux contributeurs sera
préparé avec les permissions. Aucun endpoint ni méthode HTTP d'écriture ajouté.

Limite à consigner clairement : le manager `active` filtre les fiches racines
de la vue, pas toutes les cibles imbriquées (`variant_of`, relations). La
visibilité cohérente des cibles selon statut/suppression doit être traitée
avant la livraison publique de phase 3 ; n'affirme pas que le manager règle
à lui seul la confidentialité de tout le graphe.

## 3. Migration et administration

Génère une migration unique sans modifier les précédentes, avec dépendance
vers le modèle utilisateur via le mécanisme Django approprié.
Applique-la à PostgreSQL de développement.

La fiche `shango` existante reçoit `status=draft` ; les autres nouveaux champs
sont vides ou nuls. Ne la publie pas automatiquement, ne lui invente pas de
propriétaire et ne réécris pas ses autres données. La vue reste provisoirement
accessible pour les fiches actives quel que soit leur statut : le filtrage
sur `published` viendra en phase 3.

Dans `EntityAdmin`, affiche le statut et le propriétaire, ajoute les filtres
utiles et conserve le queryset complet. Les fiches supprimées logiquement
doivent rester consultables dans l'administration. Organise les nouveaux
champs sobrement ; les horodatages de soumission, revue et suppression ainsi
que `deleted_by` sont en lecture seule. L'administration peut encore servir
à préparer manuellement des statuts et données de test ; cela ne constitue
pas une implémentation de la matrice de permissions ni du workflow.

N'ajoute pas d'actions de publication, suppression logique ou restauration,
ni de gestion des rôles. Ne modifie pas `delete()` ou les suppressions
physiques historiques dans ce seul incrément ; leurs limites seront rappelées
pour la phase 4. Les exercices techniques utiliseront l'ORM sur la base de test.

## 4. Tests ciblés

Couvre au minimum :

- valeurs par défaut et quatre statuts acceptés par `full_clean()` ; valeur
  inconnue refusée, sans prétendre que la base limite les choix ;
- attribution explicite des trois acteurs, horodatages et notes ;
- suppression d'un utilisateur : fiche conservée, FK concernées mises à nul ;
- `objects` complet et manager par défaut complet, `active` excluant uniquement
  les lignes dont `deleted_at` est renseigné ;
- fiche active en `draft`, `submitted`, `published` ou `rejected` toujours
  accessible à ce stade ; fiche supprimée absente de la liste et détail 404 ;
- accès administratif aux lignes supprimées via le queryset de `EntityAdmin` ;
- pas de fuite des nouveaux champs internes dans les deux représentations API ;
- sérialisation préchargée sans requête supplémentaire et non-régression des
  profils, référentiels, relations et sources. Le budget du détail reste
  normalement de sept requêtes, puisque les nouveaux champs ne sont pas exposés.

Le marquage `deleted_at` des tests est une préparation explicite par ORM,
pas un nouveau service métier. N'écris pas de tests de transitions ou de
permissions qui ne sont pas implémentées. Ne crée aucune donnée persistante
de démonstration dans la base de développement.

## 5. Rapport et sortie

Exécute sur la version finale : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`,
`python manage.py showmigrations mythology`, `pytest`, `ruff check .`,
`ruff format --check .` et `mypy .`. Après correction éventuelle, relance
uniquement les contrôles affectés. N'affiche pas le contenu du `.env`.

Crée un rapport français dans le dépôt API :
`docs/reports/phase-2-increment-5-editorial-fields.md`, puis sa copie identique
dans le scratchpad : `rapport-pour-codex-phase-2-increment-5-editorial-fields.md`.

Donne les fichiers changés, la migration, les managers effectifs, les règles
réellement garanties, les contrôles et le nombre de cas exécutés, l'état Git
final et les divergences éventuelles. Liste les points restant à la phase 3
(visibilité publique, y compris cibles imbriquées) et à la phase 4 (rôles,
propriété automatique, transitions, permissions, suppression et restauration).
Indique le chemin absolu du rapport temporaire dans ta réponse.

Ne coche pas la roadmap et ne committe/pousse pas cette implémentation avant
la revue. Codex effectuera la clôture après validation, sans prompt Git séparé.
