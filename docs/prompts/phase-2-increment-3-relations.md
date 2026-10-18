# Prompt Claude CLI — Phase 2, incrément 3 : variantes et relations

Implémente cet incrément dans `C:\Users\heric\dev\afrimyth-api`.
Le dépôt de pilotage est `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales applicables et inspecte l'état Git initial.

Références : ADR 0002 (variantes et protection des figures de référence),
ADR 0003 (langues), section `EntityRelation` de `docs/modele-donnees.md` et
roadmap du dépôt de pilotage. Le document d'audit contient aussi des propositions
historiques : les décisions validées de l'ADR priment.

Objectif : relier les figures entre elles, y compris lorsque la figure citée
n'a pas encore de fiche. Livrer modèles, migration, administration, API de
lecture et tests dans un seul incrément. Rester sur du Django explicite.

## 1. Variantes culturelles

Ajoute à `Entity` :

- `variant_of` : `ForeignKey("self", null=True, blank=True,
  on_delete=PROTECT, related_name="variants")`.

Une variante reste une entité autonome : ses pays, ethnies, aires, textes et
profils ne sont ni hérités ni copiés automatiquement. Son `entity_type` peut
différer de celui de la figure de référence. N'impose pas de contrainte
d'égalité des types.

Interdis qu'une entité soit sa propre référence, avec une `CheckConstraint`
en base et une validation `clean()` donnant un message anglais lisible.
La relation est facultative : aucun changement des données existantes.
La suppression physique d'une référence encore utilisée doit lever
`ProtectedError`, conformément à l'ADR 0002.

Ne construis pas de parcours récursif ou de détection générale des cycles.
Consigne la limite concernant les cycles indirects. La protection contre une
suppression logique arrivera avec les champs et services correspondants.

## 2. Relations entre figures

Crée `EntityRelation` dans `mythology`, avec :

| Champ | Définition |
|---|---|
| `from_entity` | FK `Entity`, `CASCADE`, `related_name="relations_from"` |
| `to_entity` | FK `Entity`, nullable et facultative, `CASCADE`, `related_name="relations_to"` |
| `to_name` | `CharField(max_length=255, blank=True)`, défaut textuel vide |
| `kind` | `TextChoices` : `parent`, `child`, `consort`, `ally`, `enemy` |
| `notes` | `TextField(blank=True)` |

Les valeurs, les libellés de choix et les nouveaux messages destinés au
produit sont en anglais ; les commentaires et explications internes en français.

Le sens est toujours lu depuis `from_entity` : une ligne A, `parent`, B
signifie « B est un parent de A » ; A, `child`, B signifie « B est un enfant
de A ». Documente ce sens dans le modèle, le rapport et les tests pour éviter
une inversion lors de l'import. Ne convertis pas un descendant lointain en
enfant direct : les cas ambigus seront signalés à l'import.

Contraintes et validation :

- exactement une cible : soit `to_entity` renseigné et `to_name` vide, soit
  `to_entity` nul et `to_name` non vide ; poser une `CheckConstraint` ;
- unicité `(from_entity, to_entity, kind)` pour les liens résolus ;
- unicité `(from_entity, to_name, kind)` lorsque `to_entity` est nul, afin
  d'éviter les doublons exacts non résolus ; pas de rapprochement flou ;
- interdire un lien résolu vers soi-même, en base et dans `clean()` ;
- dans `clean()`, refuser un nom cible composé uniquement d'espaces ;
- produire des erreurs compréhensibles dans l'administration ; distinguer
  dans le rapport les garanties en base des validations de formulaire.

Ne crée pas automatiquement de relation inverse ni de résolution par nom :
`parent`/`child` ou les relations de conjoint et d'alliance ne génèrent aucune
seconde ligne. Les noms sont culturellement ambigus ; ils ne doivent pas
déclencher de fusion automatique de fiches. Plusieurs types de relation
peuvent concerner la même paire de figures.

## 3. Migration et administration

Génère une migration pour les deux ajouts, sans réécrire les migrations
existantes. Applique-la à PostgreSQL de développement et préserve les données
présentes. Ne crée pas de démonstration persistante ni de migration de données.

Ajoute `variant_of` aux champs d'autocomplétion de l'administration de
`Entity`. Enregistre `EntityRelation` avec une présentation simple : source,
type, cible résolue ou nom, recherche pertinente, filtre par `kind` et
autocomplétion sur les deux FK. Pas d'interface personnalisée complexe.

## 4. API de lecture

La liste reste exactement aussi compacte qu'aujourd'hui. Le détail ajoute :

- `variant_of` : représentation synthétique de la référence, ou `null` ;
- `relations` : liste des relations sortantes (`relations_from`).

Une référence d'entité imbriquée contient uniquement `id`, `slug`, `name`,
`entity_type`. Chaque relation expose `id`, `kind`, `to_entity` (cet objet
synthétique ou `null`), `to_name` et `notes`. Une cible résolue conserve
`to_name=""`. Sans lien : `variant_of=null`, `relations=[]`.

N'imbrique pas le détail complet des cibles : cela provoquerait des récursions.
N'ajoute pas les variantes inverses, les relations entrantes ou des endpoints
autonomes à ce stade. L'API reste strictement en lecture seule.

Dans le queryset de détail, joins `variant_of` avec `select_related` et
précharge les relations sortantes avec un queryset `EntityRelation` joignant
`to_entity`. Conserve le préchargement existant des profils et référentiels.
Choisis un ordre déterministe pour les relations et teste cet ordre.

Adapte le test de budget du détail au coût réel : une collection sortante
préchargée doit normalement ajouter une requête, pas une par cible. Maintiens
le test qui impose zéro requête pendant la sérialisation après évaluation du
queryset réel de la vue. Enrichis-le avec une variante, plusieurs relations
résolues et une cible absente. Ne présente pas le seul budget global d'une
fiche comme une preuve de la présence de chaque préchargement.

## 5. Tests et limites

Couvre les cas utiles : variante d'un autre type, référence facultative,
refus d'auto-référence, protection à la suppression, relations résolues et
non résolues, exclusivité des cibles, doublons, auto-lien, nom blanc,
différents types entre une même paire et absence de création inverse.
Teste les contraintes en base avec `IntegrityError` dans un bloc atomique,
et la portée de `full_clean()` séparément. Vérifie les JSON liste/détail,
les collections vides, l'absence de récursion et le chargement des cibles.

Les liens avec `CASCADE` suivent le schéma déjà proposé. Signale leur portée
en cas de suppression physique ; l'API V1 utilisera la suppression logique
ultérieurement. N'implémente pas celle-ci dans cet incrément.

Hors périmètre : `Source`, utilisateurs/rôles, workflow, services d'écriture,
import pilote ou complet, rendu et prompts, carte et endpoints supplémentaires.

## 6. Vérification et rapport

Exécute une fois sur la version finale : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`,
`python manage.py showmigrations mythology`, `pytest`, `ruff check .`,
`ruff format --check .` et `mypy .`. Relance uniquement les contrôles affectés
si une correction est nécessaire. N'affiche aucune valeur du `.env`.

Crée un rapport français dans le dépôt API :
`docs/reports/phase-2-increment-3-relations.md`.
Copie-le à l'identique dans ton scratchpad :
`rapport-pour-codex-phase-2-increment-3-relations.md`.

Le rapport doit donner les fichiers changés, les règles et leurs garanties,
le sens des relations avec un exemple, un JSON représentatif, les résultats
des contrôles et le nombre réel de cas exécutés, le budget de requêtes,
les limites et divergences, ainsi que l'état Git final des deux dépôts.
Indique le chemin absolu du rapport temporaire dans ta réponse.

Cet incrément doit encore être relu : ne coche pas la roadmap, ne committe
pas et ne pousse pas l'implémentation maintenant. Après validation, Codex
prendra en charge directement la clôture Git, sans prompt séparé.
