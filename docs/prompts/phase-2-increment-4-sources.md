# Prompt Claude CLI — Phase 2, incrément 4 : sources documentaires

Travaille dans `C:\Users\heric\dev\afrimyth-api`.
Dépôt de pilotage : `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales et inspecte l'état Git avant d'agir. Les
référentiels, variantes et relations sont déjà validés et publiés.

Objectif : permettre de rattacher plusieurs références documentaires à une
fiche, puis de les consulter dans le détail API. C'est la tâche « Ajouter les
sources documentaires » de la phase 2, décrite dans `docs/modele-donnees.md`.

## 1. Modèle `Source`

Ajoute dans l'app `mythology`, sans nouvelle app ni abstraction :

| Champ | Définition |
|---|---|
| `entity` | `ForeignKey(Entity, on_delete=CASCADE, related_name="sources")` |
| `label` | `CharField(max_length=255)`, requis |
| `kind` | `CharField` requis avec les choix ci-dessous, sans choix implicite par défaut |
| `url` | `URLField(max_length=2048, blank=True)`, absence représentée par `""` |
| `accessed_at` | `DateField(null=True, blank=True)` |
| `notes` | `TextField(blank=True)` |

Utilise une classe `SourceKind(models.TextChoices)` :

- `book` / `Book` ;
- `article` / `Article` ;
- `encyclopedia` / `Encyclopedia` ;
- `museum` / `Museum` ;
- `oral` / `Oral tradition or testimony` ;
- `other` / `Other`.

Une source appartient à une seule entité ; une entité en possède zéro ou
plusieurs. La même référence bibliographique peut être saisie pour plusieurs
entités : pas de catalogue partagé ni de ManyToMany dans cet incrément.

Le titre, le nom du site ou la référence restent libres dans `label`. Les
pages, l'auteur, la date d'édition ou le contexte d'un témoignage peuvent être
précisés dans `notes`. Ne crée pas de champs bibliographiques supplémentaires.
Une source orale ou un livre sans URL reste valide. `accessed_at` est une date
de consultation facultative, pas une date de publication ; elle peut être
présente sans URL. Ne remplis ni date ni référence automatiquement.

Prévois un `__str__` utile, un ordre déterministe `label`, puis `id`, et le
refus d'un `label` composé uniquement d'espaces lors de `full_clean()` et dans
l'administration. Utilise les validations Django pour `kind` et le format de
l'URL. Les garanties de ces validations ne doivent pas être présentées comme
des contraintes PostgreSQL : `save()` ne déclenche pas `full_clean()`.

Ne pose pas d'unicité sur `label`, l'URL ou leur combinaison : deux éditions,
pages ou usages peuvent partager ces valeurs. Le contrôle éditorial reste
distinct de la structure de la base. Aucun accès réseau, téléchargement,
vérification de disponibilité ou liste de sites autorisés n'est demandé.

Les identifiants, nouveaux libellés et messages publics sont en anglais ;
les commentaires, le prompt et le rapport restent en français (ADR 0003).

## 2. Migration et administration

Génère une migration unique, sans toucher les précédentes. Applique-la sur
PostgreSQL de développement. Préserve les entités existantes ; ne crée pas
de sources de démonstration persistantes ni de migration de données.

Enregistre `Source` dans l'administration avec une interface simple : colonnes
utiles (`label`, `entity`, `kind`, `accessed_at`), recherche par référence et
nom d'entité, filtre par `kind`, autocomplétion de `entity` et
`list_select_related` adapté. Pas de formulaire ou d'interface complexe.

`CASCADE` retire les sources lors d'une suppression physique de leur entité.
La suppression logique viendra ultérieurement et ne déclenche pas ce mécanisme.

## 3. API de lecture

Garde la liste compacte, avec exactement les clés actuelles. Dans le détail,
ajoute `sources`, une liste d'objets avec exactement :

`id`, `label`, `kind`, `url`, `accessed_at`, `notes`.

Utilise un serializer imbriqué en lecture seule, sans réimbriquer l'entité.
Sans source : `sources=[]`. Sans URL : `url=""`. Sans date :
`accessed_at=null`. Une date renseignée est exposée au format `YYYY-MM-DD`.
L'API reste strictement en lecture seule ; aucun endpoint autonome de source.

Ajoute `sources` au `prefetch_related` du détail, en conservant les autres
jointures et préchargements. La liste ne charge pas les sources.

Adapte les tests de budget : le détail passe normalement de six à sept
requêtes, une seule requête supplémentaire pour toutes ses sources. Enrichis
le test existant imposant zéro requête pendant la sérialisation du queryset
préchargé avec plusieurs sources. Ce test prouve le préchargement ; le seul
budget global d'une fiche ne suffit pas à le prouver.

## 4. Tests et vérifications

Couvre les comportements utiles :

- plusieurs sources pour une entité ; même référence sur deux entités ;
- choix de `kind` valides et refus d'une valeur inconnue par `full_clean()` ;
- référence obligatoire, refus d'un titre blanc et URL invalide par validation ;
- source sans URL ni date, source orale, notes et date de consultation ;
- suppression en cascade depuis l'entité et conservation de celle-ci lorsqu'on
  supprime seulement une source ;
- JSON du détail, ordre déterministe, liste vide et liste d'entités inchangée ;
- budget global adapté et zéro requête pendant la sérialisation préchargée ;
- non-régression des profils, référentiels, variantes et relations.

Utilise des références fictives explicitement identifiées comme telles, avec
des noms et contenus de démonstration en anglais. N'invente pas une source
documentaire réelle ni une citation pour le corpus.

Sur la version finale, exécute : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`,
`python manage.py showmigrations mythology`, `pytest`, `ruff check .`,
`ruff format --check .` et `mypy .`. Si une correction intervient, relance
uniquement les contrôles affectés. N'affiche aucune valeur du `.env`.

## 5. Rapport et fin d'intervention

Crée dans le dépôt API `docs/reports/phase-2-increment-4-sources.md`, en français,
puis une copie identique dans ton scratchpad :
`rapport-pour-codex-phase-2-increment-4-sources.md`.

Indique les fichiers changés, les règles et leur portée réelle, un JSON
fictif représentatif, la migration appliquée et la préservation des données,
les résultats des contrôles et le nombre de cas exécutés, le budget des
requêtes, les limites ou divergences et l'état Git final. Donne le chemin
absolu du rapport temporaire dans ta réponse.

Hors périmètre : workflow, rôles, permissions, suppression logique, services
d'écriture, import, rendu, carte, authentification ou nouvelle architecture.

Ne coche pas la roadmap, ne committe pas et ne pousse pas cette implémentation
avant notre revue. Après validation pédagogique, Codex effectuera directement
la clôture documentaire, les commits et les pushes, sans prompt Git séparé.
