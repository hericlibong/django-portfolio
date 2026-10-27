# Phase 4 — Incrément 3 : création d'une entité et de son profil

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses `AGENTS.md` et
`CLAUDE.md`, la phase 4 de la roadmap et les ADR 0002, 0003, 0004 du dépôt
voisin `django-portfolio`. La lecture privée de l'incrément 2 est validée.

## Objectif et périmètre

Ouvrir uniquement `POST /api/manage/entities/` : créer une entité avec son
profil, en brouillon, au nom du compte appelant. Conserver toutes les lectures
existantes et le filtrage de leurs références. Pas de PUT, PATCH, DELETE,
transition de statut, inscription ou nouveau réglage de durée JWT.

Reste sobre : serializers DRF explicites, validations Django/DRF réutilisées,
service existant plutôt que duplication. Pas de framework de permissions ou
de validation, de serializer universel, ni d'ouverture du CRUD complet.
Nouvelles docstrings, messages, exemples OpenAPI et README en anglais ;
commentaires, guide et rapport internes en français.

## Contrat de création

- JWT d'accès et rôle métier requis. Contributeur, relecteur, administrateur
  et superutilisateur actif peuvent créer. Sans JWT : 401 ; compte sans rôle,
  y compris staff seul : 403. Réutiliser les règles de l'incrément 2.
- Corps JSON plat pour les champs communs de l'entité. `name`, `slug`,
  `entity_type`, `gender` requis. Slug explicite, sans génération automatique.
- Profil sous sa clé existante : `divinity_profile`, `hero_profile` ou
  `creature_profile`. Seule la clé correspondant au type est acceptée ;
  profil absent ou `{}` : créer le bon profil avec ses valeurs par défaut.
  Un profil null ou d'un autre type est refusé en 400, pas ignoré. Ne jamais
  accepter `entity`, `entity_id`, `id` ou `pk` dans un profil.
- Autoriser les champs métier simples existants et les listes de chaînes
  selon leurs validations actuelles. Ne pas convertir silencieusement un
  mauvais type de données en valeur acceptable.
- Rattachements : clés étrangères et M2M par identifiants de référentiels
  existants, sous les noms actuels des champs. Principaux facultatifs/null,
  secondaires et domaines facultatifs/listes vides. Pas de création imbriquée
  ni de nouveau CRUD des référentiels. Refuser une valeur présente à la fois
  en principal et en secondaire pour une même dimension, selon l'ADR 0002.
- `variant_of` facultatif/null, par UUID ; sélectionner uniquement parmi les
  entités lisibles privément par cet appelant et non supprimées. Cible
  interdite, supprimée ou inexistante : erreur 400 sans distinction révélant
  son existence. Réutiliser la politique privée, ne pas la réécrire.
- Création imbriquée de `sources` et `relations` différée à l'incrément 4 :
  refuser leur présence en 400 avec un message clair, sans les ignorer.
- `created_by` est imposé côté serveur à l'appelant, `status` à `draft`.
  Aucun client, même administrateur, ne fournit acteur, statut, marques de
  revue/suppression, UUID ou dates techniques. Refuser explicitement ces
  champs en 400 lorsqu'ils sont présents, au lieu de prétendre les accepter.
  Utiliser une liste explicite de champs d'entrée, jamais le JSON brut comme
  arguments du modèle ; ne pas inventer un moteur générique de validation.

Réponse 201 : représentation du détail privé déjà existante, profil inclus.
La fiche est immédiatement accessible à son propriétaire en privé et absente
du public. Une création invalide renvoie 400, jamais des données partielles.

## Implémentation et atomicité

Réutiliser `create_entity_with_profile()` : ce service garde son rôle concret
et ses usages par l'import. Les rattachements M2M se font dans une transaction
englobant aussi le couple entité/profil. Un échec doit tout annuler.
Convertir les erreurs de validation métier Django en réponses DRF 400 lisibles,
sans capture générale masquant les bugs. Garder la protection des doublons.

Ajouter l'action de création uniquement à la vue privée ; ne pas activer les
autres écritures par l'emploi d'un ModelViewSet complet. Les vues publiques
restent sans écriture. Ne pas modifier les modèles, l'import, les comptes,
les appartenances de groupes ou les données de développement sans nécessité.
L'administration existante et sa colonne Groups restent fonctionnelles.

## Vérifications et documentation

Tester les trois types et leur profil exact, les valeurs par défaut, les
doublons, les profils incohérents, les champs serveur injectés, les tableaux
invalides, les rattachements inexistants ou redondants, les variantes
interdites, les permissions, le propriétaire et le draft imposés. Vérifier
l'atomicité sur un échec du profil après sauvegarde de l'entité et un échec
de rattachement après création du couple ; aucun objet partiel ne subsiste.
Ne pas recopier toute la suite de lecture : garder ses tests de non-régression.
Adapter les tests qui refusaient POST privé, sans affaiblir les refus des
autres méthodes ou du POST public.

Actualiser OpenAPI, exemples, sécurité, erreurs et README anglais : POST
privé disponible, lectures publiques inchangées et autres écritures différées.
Les textes ne doivent plus annoncer que toute l'API privée est read-only.
Préserver les réglages ReDoc et le démarrage TypingPatchConfig.

Exécuter check, makemigrations --check --dry-run, validation spectacular
--validate --fail-on-warn, pytest, Ruff, format et mypy. Après une correction,
relancer seulement les contrôles affectés. Ne pas lire ou afficher les secrets.

Préparer un guide Postman français court, une étape à la fois : créer une
figure fictive au slug exact `demo-create-hero-a` avec `demo-contrib-a`, lire
le résultat en privé, constater son absence en public, puis provoquer une
erreur sans création partielle. Exemples sans boucle ni bloc indenté.
Ne créer aucune donnée automatiquement dans la base de développement.
Les comptes existent déjà : `demo-contrib-a`, `demo-contrib-b`,
`demo-selecteur` (reviewer) et `essai-postman` (sans rôle). Anansi a été
attribuée manuellement au contributeur A ; les trois autres fiches du corpus
restent sans propriétaire. Ne pas les supprimer ou les réattribuer.
Prévoir un nettoyage limité à l'objet fictif effectivement créé, pas un préfixe
global. Ne jamais inclure mot de passe ou jeton dans un exemple ou un rapport.

Créer `docs/reports/phase-4-increment-3-create.md`, puis une copie identique
dans le scratchpad nommée `rapport-pour-codex-phase-4-increment-3-create.md`.
Indiquer le contrat exact, les fichiers, les contrôles exécutés, les limites,
l'état Git et les extraits à expliquer. Distinguer tests automatisés et
exercices manuels restant à faire ; terminer par le chemin de la copie.

Ne pas cocher la roadmap ni committer/pousser l'implémentation avant la revue
et les exercices. Codex prendra en charge la clôture après validation.
