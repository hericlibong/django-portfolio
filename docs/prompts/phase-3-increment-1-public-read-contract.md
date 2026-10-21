# Prompt Claude CLI — Phase 3, incrément 1 : visibilité et contrat de lecture

Travaille dans `C:\Users\heric\dev\afrimyth-api`.
Pilotage : `C:\Users\heric\dev\django-portfolio`.
Lis les instructions locales, les ADR 0002/0003, la phase 3 de la roadmap et
`docs/notes/revue-editoriale-corpus-pilote.md`. Inspecte l'état Git des deux
dépôts. Les modifications documentaires déjà présentes dans le pilotage
appartiennent à Codex ; préserve-les.

Objectif : une API publique en lecture seule avec des réponses stables,
exposant exclusivement les fiches publiées non supprimées et leurs références
autorisées. Pagination/filtres/recherche arriveront à l'incrément 2,
OpenAPI/Swagger/ReDoc à l'incrément 3. Pas de CRUD ni de workflow maintenant.

## 1. Règle de visibilité publique

Une fiche est visible si et seulement si :

`status == EntityStatus.PUBLISHED` et `deleted_at is None`.

Applique cette règle à la liste et au détail, pour les utilisateurs anonymes
comme connectés, y compris un superutilisateur : ces routes sont publiques,
pas des routes privées de gestion. Une fiche non publique et un slug inconnu
renvoient le même HTTP 404, sans motif détaillant son existence ou son statut.

Centralise la sélection dans une petite fonction réutilisable (par exemple
`public_entities()` dans `mythology/selectors.py`) et sépare si utile la
préparation du queryset de détail. Cette fonction est explicitement demandée
par la roadmap ; pas de moteur de permissions ou de repository générique.
Le contrat permettra d'ajouter ensuite une sélection privée en phase 4 sans
réécrire les serializers liste/détail.

Garde `Entity.objects` complet, `Entity.active` filtrant seulement la suppression,
et l'administration sur son queryset complet. Ne change pas les managers pour
implémenter la visibilité éditoriale. Aucune modification des données existantes.

## 2. Références imbriquées : même visibilité

Pour le détail d'une fiche publique :

- `variant_of` : objet synthétique existant si la référence est publique ;
  sinon `null`. Ne révèle ni son identifiant, slug, nom ou type.
- `relations` : garde les liens dont la cible résolue est publique, ainsi
  que les liens déjà non résolus (`to_entity=null`, `to_name` renseigné).
  Omet entièrement un lien visant une cible résolue non publique, y compris
  son `id`, ses notes et son type. Ne le transforme pas en lien non résolu
  en recopiant le nom de la cible masquée.
- Une cible publique ne donne accès qu'à la référence synthétique à quatre
  champs existante : aucune imbrication récursive du détail.

Ces règles portent sur les références structurées. Elles ne promettent pas
de censurer automatiquement les noms figurant dans le récit public ou dans
une relation réellement non résolue : la revue éditoriale traite le texte.
Le `to_name` saisi comme cible absente n'est pas rapproché automatiquement
d'une fiche privée homonyme.

Le choix technique t'appartient : préchargement filtré des relations sortantes,
jointure de la variante et garde de sérialisation, ou préparation explicite
d'attributs publics. Assure la cohérence avec la sélection centralisée et évite
un chemin de repli utilisant les relations non filtrées. Si le serializer
nécessite un queryset préparé, documente et teste ce contrat d'utilisation.
Ne mutile pas les instances en base pour masquer une référence.

## 3. Contrat et langue publique

Préserve les clés actuelles de la liste compacte et du détail. Les trois clés
de profil restent présentes avec leurs valeurs habituelles ; `variant_of`
reste objet ou `null`, et les collections restent des listes, éventuellement
vides. Les sources et référentiels de la fiche publique gardent leur forme.
N'expose pas les comptes, emails, identités utilisateur, notes de revue,
statuts internes ou champs de suppression. Les écritures HTTP restent refusées.

Conserve `/api/entities/` et `/api/entities/{slug}/`, les noms des routes et
les UUID déjà exposés. Ne renomme pas les slugs existants et n'ajoute pas
un nouveau système de routes pour anticiper les filtres ou la phase 4.

Normalise les éléments publics en anglais conformément à l'ADR 0003 :

- libellés de choix et `verbose_name`/`verbose_name_plural` du domaine ;
- `help_text`, descriptions destinées aux clients et erreurs de validation ;
- erreurs standard Django/DRF rendues par l'API, notamment les réponses 404/405.

La configuration actuelle est `LANGUAGE_CODE="fr-fr"` : passe la langue par
défaut du produit à l'anglais pour les messages standards. Ne change pas le
fuseau horaire et n'ajoute pas de middleware de langue ni d'architecture i18n.
Les commentaires, docstrings internes, prompts et rapports restent français.
Ne traduis pas le corpus ni ne corriges ses classifications éditoriales ici.

Génère la migration de métadonnées si Django détecte les changements de
libellés/choix ; pas de réécriture des migrations anciennes et pas de
renommage des valeurs techniques stockées. Mets à jour les assertions qui
vérifiaient les anciens libellés français. Les `__str__` destinés à afficher
des objets dans l'interface doivent rester cohérents avec la langue publique.

## 4. Tests à adapter et à compléter

Les tests d'API antérieurs préparaient souvent des brouillons accessibles et
un test documentait la visibilité de cibles supprimées : ces attentes sont
désormais obsolètes. Remplace-les par les nouveaux comportements.

Ne change pas le statut par défaut des modèles, du service, des factories
globales ou de l'import pour rendre ces tests verts. Prépare explicitement
des fiches publiées dans les tests qui exercent la lecture publique. Les
tests d'import vérifient toujours la création en `draft` ; pour tester ensuite
leur lecture publique, publie-les uniquement dans la base de test.

Couvre les combinaisons pertinentes :

- racines `draft`, `submitted`, `rejected`, `published`, supprimées ou non ;
- seul un `published` non supprimé apparaît ; 404 générique sinon ;
- utilisateur anonyme ou connecté, dont superutilisateur : pas de contournement
  de la règle publique et administration conservant ses accès ;
- variante publique/non publique/supprimée, avec contenu source public ;
- relations mélangées : cible publique, privée, supprimée et nom non résolu ;
  vérifier le JSON brut, pas seulement une longueur de liste ;
- aucun nom/slug/id/notes du lien masqué dans la réponse structurée ; aucun
  basculement automatique en `to_name` pour réexposer une cible cachée ;
- références publiques sans récursion, clés stables et champs internes absents ;
- messages publics anglais pour erreurs pertinentes ; écritures toujours refusées ;
- `objects` et `active` conservent leur portée ; corpus de développement intact.

Conserve les tests de chargement : mesure et explique le coût réel du détail
entièrement relié et teste zéro requête pendant la sérialisation après
évaluation du queryset réellement utilisé par la vue. Aucun accès de base par
référence masquée. Réutilise les tests utiles au lieu de les dupliquer.
Ne fige pas un nombre arbitraire de requêtes avant de mesurer le choix technique.

## 5. Données de développement et documentation

Les quatre fiches actuellement en base sont en brouillon. Après cet incrément,
la liste publique peut donc légitimement être vide et les URL du pilote en
404. Ne publie ni Oya, Anansi, Mokele-mbembe ou Shango, ne leur attribue pas
de propriétaire et ne les répare pas pour maintenir la démonstration.

Actualise le README public en anglais : règle de visibilité, liste/détail,
références masquées et comportement attendu d'une base ne contenant que des
brouillons. N'affirme pas que JWT, le workflow, les filtres ou Swagger existent.
Actualise les docstrings devenues fausses et la règle de fonctions métier
de `CLAUDE.md` seulement si nécessaire pour mentionner cette sélection explicite.

## 6. Vérifications et rapport

Exécute sur la version finale : `python manage.py check`,
`python manage.py makemigrations --check --dry-run`, `pytest`, `ruff check .`,
`ruff format --check .` et `mypy .`. Si une migration est générée, applique-la
et vérifie son état. Après correction, relance uniquement les contrôles affectés.
N'affiche aucune valeur du `.env`. Vérifie par lecture que les fiches de
développement ont gardé leurs statuts et données éditoriales.

Crée un rapport français :
`docs/reports/phase-3-increment-1-public-read-contract.md`, avec copie identique
dans ton scratchpad : `rapport-pour-codex-phase-3-increment-1-public-read-contract.md`.

Indique les fichiers modifiés, la sélection publique et son contrat réutilisable,
les règles appliquées aux références imbriquées, les tests et leur nombre
réel de cas, le budget mesuré de requêtes, les migrations éventuelles et la
préservation des données. Distingue contrôles exécutés, limites et travaux
reportés. Termine par l'état Git des deux dépôts et donne le chemin absolu
de la copie temporaire dans ta réponse.

Ne coche pas la roadmap, ne committe et ne pousse pas avant notre revue.
Codex effectuera la clôture après validation, sans prompt Git supplémentaire.
