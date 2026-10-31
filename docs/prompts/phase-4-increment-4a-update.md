# Phase 4 — Incrément 4A : modification contrôlée

**ADR 0005 acceptée par l'auteur le 30 octobre 2026. Implémenter uniquement le lot 4A.**

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis ses consignes, les
ADR 0002 à 0005 et la phase 4 de la roadmap du dépôt voisin `django-portfolio`.
L'incrément 3 est validé et publié. Ne pas refaire sa revue ou son guide.

## Livrable

Ouvrir seulement `PATCH /api/manage/entities/{slug}/` : modifier les champs
métier, le profil existant et les rattachements, sans changer propriétaire,
type ou statut. Réponse 200 au format du détail privé. GET et POST existants
inchangés ; PUT, DELETE, transitions et écritures publiques restent fermés.
Les sources/relations constituent le lot 4B : leur écriture reste refusée ici.

## Droits, selon les décisions existantes

JWT et groupes évalués comme aujourd'hui. Contributeur : ses seuls draft et
rejected ; relecteur : tous les draft, rejected et submitted ; administrateur
et superutilisateur actif : tous les statuts non supprimés. `is_staff` seul
n'accorde rien ; les groupes se cumulent et restent lus en base.

Sans JWT utilisable : 401 ; compte sans rôle : 403. Fiche supprimée,
inexistante ou non lisible : même 404. Fiche lisible mais non modifiable : 403.
Appliquer ces droits à la fiche effectivement modifiée, pas seulement à la
route. Évaluer le droit sur l'état verrouillé dans la transaction d'écriture
pour ne pas valider un statut puis sauvegarder après son changement concurrent.

## Contrat du PATCH

- Champ absent : valeur conservée. Liste fournie : remplacement de cette
  seule liste ; `[]` la vide. `null` seulement sur les champs déjà nullables.
- Champs métier autorisés comme à la création, sauf `entity_type`, refusé
  selon ADR 0005. UUID inchangé ; slug modifiable avec son unicité conservée.
- Profil sous la clé du type actuel, modification partielle de ses champs.
  Profil absent ou `{}` : aucun effacement. Refuser null, mauvais profil,
  identifiant ou rattachement de profil. Conserver exactement le bon profil.
  Une ancienne fiche incohérente ne doit pas être réparée implicitement :
  refus 400 explicite sans mutation ; signaler ce cas dans le rapport.
- Refuser explicitement les champs serveur, inconnus ou différés comme à la
  création, y compris `created_by`, `status`, marques de revue/suppression,
  identifiants, dates techniques, sources et relations. Pas de transfert de
  propriété, même pour l'administrateur.
- Rattachements par identifiants existants ; vérifier principal/secondaire
  sur l'état final, en combinant valeurs actuelles et champs du PATCH.
  Les collections omises restent intactes. Pas de création de référentiels.
- `variant_of` reste limité aux cibles privées lisibles et non supprimées ;
  refus uniforme des cibles interdites/supprimées/inconnues. Refuser l'auto-lien.
- Garder les validations acquises : tableaux réellement textuels sans
  conversion numérique, longueurs, choix, normalisation de threat_level,
  unicité du slug et de (name, entity_type), sans conflit avec la fiche elle-même.

Toutes les modifications, profil et M2M compris, sont atomiques : une erreur
annule l'ensemble. Réutiliser les validations et la représentation existantes
sans recopier tout le serializer de création. Une fonction métier concrète
est acceptable ; pas de framework ou hiérarchie anticipée. Ne jamais masquer
une erreur inattendue derrière une capture générale en 400.

## Vérifications et remise

Tests ciblés rôle × propriétaire × statut, patch partiel et champs omis,
profil/type, injections, tableaux, doublons, état final principal/secondaire,
références interdites et atomicité après mutation de l'entité ou du profil.
Conserver les tests de lecture/création et adapter seulement les anciens
refus de PATCH privé. Vérifier que PUT/DELETE et les écritures publiques
restent refusés. Pas de tests redondants ou assertions sur toute l'architecture.

Actualiser OpenAPI et README anglais pour PATCH, ses entrées partielles,
200/400/401/403/404 et ses limites ; conserver ReDoc et les réglages existants.
Docstrings anglaises courtes, commentaires seulement pour les choix utiles.
Ne pas gonfler les modules avec la répétition des guides ou du contrat.

Exécuter check, makemigrations --check --dry-run, validation spectacular
--validate --fail-on-warn, pytest, Ruff, format et mypy. Après correction,
relancer seulement les contrôles affectés. Ne pas lire ou afficher les secrets.
Ne pas changer le schéma, l'import, les comptes, les groupes ni les données
de développement pour préparer une démonstration.

La fiche fictive `demo-create-hero-a` existe pour les futurs essais de Codex.
Ne pas la modifier automatiquement, ni toucher aux quatre fiches du corpus.

Rapport factuel court : `docs/reports/phase-4-increment-4a-update.md` ; copie
identique dans le scratchpad `rapport-pour-codex-phase-4-increment-4a-update.md`.
Indiquer fichiers, contrat, résultats réels, limites et état Git. Donner le
chemin de la copie à la fin. Pas de guide Postman ni de clôture documentaire
demandés à Claude : Codex s'en charge. Rien coché, committé ou poussé avant
revue et exercices ; ne pas lancer le lot 4B.
