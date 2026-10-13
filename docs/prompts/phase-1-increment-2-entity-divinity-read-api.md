# Prompt proposé à Claude CLI — Phase 1, incrément 2

Tu travailles dans `C:\Users\heric\dev\afrimyth-api`.

Avant toute modification, lis :

- `CLAUDE.md` et `AGENTS.md` dans ce dépôt ;
- `C:\Users\heric\dev\django-portfolio\docs\roadmap-projet-1.md` ;
- `C:\Users\heric\dev\django-portfolio\docs\adr\0002-decisions-structurantes-phase-0.md` ;
- la section 4.3 de
  `C:\Users\heric\dev\django-portfolio\docs\modele-donnees.md`.

Implémente uniquement le deuxième incrément de la phase 1 : un premier modèle
métier exécutable, son administration et une API en lecture seule. Ne complète
pas encore tout le modèle de la phase 2.

## 1. Modèles

Dans l'app `mythology`, crée `Entity`, tronc commun à toutes les figures, avec
les champs stables nécessaires à cette première tranche :

- `id` : UUID comme clé primaire, généré automatiquement et non éditable ;
- `slug` : unique ;
- `name` : chaîne de 255 caractères ;
- `entity_type` : choix `divinity`, `hero`, `creature` ;
- `category` : sous-titre éditorial facultatif ;
- `gender` : choix `male`, `female`, `neutral`, `androgynous`, `fluid` ;
- `cultural_role` : facultatif ;
- `alignment` : facultatif ;
- `story_description` : texte facultatif ;
- `characteristics`, `physical_signs`, `symbols`, `power_objects` et
  `symbolic_animals` : `ArrayField` de chaînes, avec `default=list` et
  `blank=True` ;
- `manifestations` : texte facultatif ;
- `created_at` et `updated_at`.

Ajoute une contrainte d'unicité sur `(name, entity_type)` et un index utile sur
`entity_type`. N'ajoute pas de logique de génération de slug complexe : le slug
reste explicite pour l'instant et l'administration peut le préremplir depuis le
nom.

Crée `DivinityProfile` avec :

- un `OneToOneField` vers `Entity`, `on_delete=CASCADE`,
  `related_name="divinity_profile"` ;
- `cult_offerings`, `cult_taboos`, `cult_rituals`, `cult_places` et
  `cult_festivals`, tous en `ArrayField(CharField(max_length=100))`,
  `default=list`, `blank=True`.

Une instance `DivinityProfile` ne doit référencer qu'une `Entity` dont
`entity_type == "divinity"`. Mets en place une validation Django simple et
testée, sans créer de moteur de service générique. N'essaie pas encore de
garantir automatiquement qu'une `Entity` possède exactement un profil : la
création métier complète arrivera avec les services et les autres profils.

Ne crée pas encore :

- `Country`, `Ethnicity`, `CulturalArea`, `Domain` ou une app de référentiels ;
- `HeroProfile`, `CreatureProfile` ou `EntityRelation` ;
- les champs de workflow, de propriété, de revue ou de suppression logique ;
- les images, données de rendu ou fonctions d'import.

Ces éléments appartiennent aux phases suivantes de la roadmap.

## 2. Migration et administration

- Génère la migration avec Django et applique-la sur PostgreSQL.
- Enregistre `Entity` et `DivinityProfile` dans l'administration Django.
- Fournis une administration simple et utile : colonnes principales, filtres
  par type et recherche par nom ; préremplissage du slug depuis le nom.
- Évite toute abstraction ou personnalisation d'administration non nécessaire.

## 3. Première API liste/détail en lecture seule

Expose :

- `GET /api/entities/` : collection compacte contenant au minimum `id`, `slug`,
  `name`, `entity_type` et `category` ;
- `GET /api/entities/{slug}/` : détail d'une entité avec ses champs communs et,
  lorsqu'il existe, son `divinity_profile` imbriqué.

Utilise Django REST Framework de manière conventionnelle. Les deux endpoints
sont `AllowAny` pour rendre cette première tranche observable. Il n'existe pas
encore de statut éditorial : le filtrage sur `published` sera ajouté lorsque le
workflow existera. L'API est strictement en lecture seule : aucun POST, PUT,
PATCH ou DELETE. Cette lecture seule est **provisoire et limitée à cet
incrément** : la phase 4 ajoutera les comptes, rôles, permissions objet, le CRUD
et les transitions de modération validées dans l'ADR 0002. Ne devance pas cette
phase ici.

La vue de détail doit charger le profil avec `select_related` afin d'éviter une
requête supplémentaire. Ne mets en place ni pagination, ni recherche HTTP, ni
filtres, ni OpenAPI dans cet incrément : ils appartiennent à la phase 3.

Conserve le rendu HTML navigable fourni par défaut par Django REST Framework en
développement, afin que les routes puissent être inspectées depuis un
navigateur, en plus de leur représentation JSON. Documente dans le rapport les
étapes d'une vérification manuelle future : créer une divinité et son profil
dans l'administration, lancer le serveur, ouvrir la liste puis le détail. Ne
crée pas automatiquement de superutilisateur et n'ajoute pas de donnée de
démonstration persistante.

Les champs de cet incrément sont un socle, pas la déclaration que le modèle
final serait terminé. Ils décrivent des faits mythologiques réutilisables
(récit, caractéristiques, apparence, symboles, manifestations et culte). Les
données propres à la génération de médias — prompts, variantes, paramètres et
métadonnées d'image, de son ou de vidéo — doivent rester séparées et hors de cet
incrément. Signale explicitement cette frontière dans le rapport.

## 4. Tests

Ajoute des tests couvrant au minimum :

- création d'une `Entity` de type divinité et de son profil OneToOne ;
- refus par la validation d'un `DivinityProfile` rattaché à un autre type ;
- contrainte d'unicité `(name, entity_type)` ;
- suppression en cascade du profil lorsque l'entité est supprimée ;
- réponse compacte de la liste ;
- détail avec profil imbriqué ;
- détail d'une entité sans profil ;
- `404` pour un slug inconnu ;
- refus des méthodes d'écriture sur les endpoints de lecture.

## 5. Vérifications et rapport

Exécute et fais réussir :

- `python manage.py check` ;
- `python manage.py makemigrations --check --dry-run` ;
- `python manage.py migrate` ;
- `pytest` ;
- `ruff check .` ;
- `ruff format --check .` ;
- `mypy .`.

Crée `docs/reports/phase-1-increment-2.md` selon la convention du dépôt. Le
rapport doit notamment expliquer les champs retenus, la relation OneToOne, les
routes exposées, les résultats exacts des commandes et les tâches reportées.

Ne committe et ne pousse rien. Termine par un résumé concis destiné à la revue
de code, avec la liste des fichiers touchés et tout arbitrage ou blocage réel.
