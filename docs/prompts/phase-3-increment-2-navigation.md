# Phase 3 — Incrément 2 : navigation dans la collection publique

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, puis la phase 3 de la roadmap du dépôt voisin `django-portfolio`.
L'incrément 1 est validé : conserve sa visibilité publique et ses protections
des références imbriquées. Ne recommence pas sa revue.

## Objectif et périmètre

Rendre `GET /api/entities/` praticable avec un corpus important : pages,
recherche textuelle et filtres combinables. L'API reste publique et en lecture
seule. Aucun changement de modèle, de corpus, de statut de développement,
de workflow ou de permissions. Swagger/ReDoc reste pour l'incrément 3.

Implémente simplement avec Django/DRF ; pas de moteur de recherche externe,
d'abstraction générique ni de dépendance ajoutée sans nécessité. Les messages
et le README publics sont en anglais ; commentaires et rapport en français.

## 1. Pagination

- Pagination par numéro de page : `?page=2`, 20 fiches par page, taille fixe.
- Réponse de liste : `count`, `next`, `previous`, `results`, selon DRF.
  `count` compte uniquement les fiches publiques correspondant à la recherche.
- Les éléments de `results` gardent les champs du serializer de liste.
  Le détail et l'endpoint de santé ne changent pas de forme.
- Ordre déterministe par `name`, puis `id` pour départager les noms égaux.
- Conserver filtres et recherche dans les liens de navigation ; conserver le
  comportement standard DRF pour les pages inexistantes et les erreurs.

C'est un changement assumé du contrat de liste : documente le remplacement
du tableau brut par cette enveloppe et adapte les tests existants concernés.

## 2. Recherche et filtres — contrat attendu

Ces paramètres sélectionnent la **liste seulement** : ils ne doivent pas
modifier la disponibilité du détail `/api/entities/{slug}/`.

| Paramètre | Sens |
|---|---|
| `search` | Recherche DRF insensible à la casse dans `name`, `category`, `cultural_role`, `story_description` ; pas dans les notes internes, sources ou fiches liées |
| `entity_type` | Valeur exacte `divinity`, `hero` ou `creature` |
| `country` | Code ISO à deux lettres, normalisé en majuscules ; pays principal **ou** secondaire |
| `cultural_area` | Slug de l'aire ; rattachement principal **ou** secondaire à cette aire **ou à l'un de ses descendants** |

Les paramètres se combinent par ET ; les deux rattachements d'un même filtre
se combinent par OU. Aucun doublon quand plusieurs rattachements correspondent.
Un paramètre vide ne filtre pas. Un type inconnu ou un code pays mal formé
renvoie 400 avec un message anglais. Un code pays bien formé mais absent,
ou un slug d'aire absent, renvoie une liste vide : ne pas divulguer de données
privées via les résultats ou `count`.

Pour les aires, utilise la hiérarchie réellement enregistrée dans `parent`,
sans coder les noms des régions. `central-africa` doit inclure une fiche liée
à `congo-basin`, et cela doit aussi fonctionner à plusieurs niveaux. Une
traversée simple suffit ; éviter une boucle infinie si la hiérarchie contient
un cycle, sans refonte du modèle. Ne déduire aucun pays à partir d'une aire.

Exemples à documenter :

- `/api/entities/?entity_type=divinity`
- `/api/entities/?country=CD&entity_type=creature`
- `/api/entities/?cultural_area=central-africa`
- `/api/entities/?search=anansi`
- `/api/entities/?entity_type=hero&page=2`

## 3. Vérifications et livrable

Tests autonomes avec des fiches synthétiques publiées dans la base de test ;
ne publie pas les brouillons réels pour illustrer ces fonctions.

Couvre pagination vide et multi-pages, ordre stable avec noms égaux,
conservation des paramètres dans les liens, recherche et absence de résultat,
chaque filtre, rattachements secondaires, descendants à plusieurs niveaux,
absence de doublons et combinaison des paramètres. Vérifie les entrées
invalides, l'exclusion des fiches privées/supprimées des résultats **et du
compte**, et l'absence d'effet des filtres de liste sur le détail.

Mesure les requêtes utiles : la pagination peut ajouter un comptage, et la
hiérarchie son propre coût ; expliquer ces coûts et vérifier l'absence de
requêtes par entité supplémentaire, sans figer un budget artificiel.
Conserve les garanties de visibilité imbriquée et les refus d'écriture.

Exécute `python manage.py check`, `python manage.py makemigrations --check --dry-run`,
`pytest`, `ruff check .`, `ruff format --check .` et `mypy .`.
Après correction, ne relance que les contrôles affectés. Pas de secret affiché.

Actualise le README anglais avec les paramètres, leur portée, la pagination,
un exemple de réponse et les erreurs attendues. Crée le rapport français
`docs/reports/phase-3-increment-2-navigation.md`, avec copie identique dans
le scratchpad : `rapport-pour-codex-phase-3-increment-2-navigation.md`.
Indique fichiers touchés, contrat exact, résultats réellement exécutés,
nombre de tests, requêtes mesurées, limites et état Git. Termine ta réponse
par le chemin absolu de la copie temporaire.

Ne coche pas la roadmap et ne committe ni ne pousse avant la revue.
Codex effectuera la clôture après validation, sans prompt Git supplémentaire.
