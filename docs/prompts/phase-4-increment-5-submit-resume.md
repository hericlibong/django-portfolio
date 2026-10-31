# Phase 4 — Incrément 5 : soumission, retrait et reprise

Travaille dans `C:\Users\heric\dev\afrimyth-api`. Lis `AGENTS.md`,
`CLAUDE.md`, les ADR 0002 à 0005 et la phase 4 de la roadmap du dépôt voisin
`django-portfolio`. L'incrément 4 (4A/4B) est validé et publié ; point de
départ API `e3d2781`. Ne pas refaire sa revue ou ses exercices. Préserver
toute modification préexistante sans la réinitialiser ou l'embarquer hors sujet.

## 1. Livrable strict : trois actions, pas un workflow complet

POST uniquement, sous `/api/manage/entities/{slug}/` :

| Action | Route | Transition autorisée |
|---|---|---|
| Soumettre | `submit/` | draft → submitted |
| Retirer | `withdraw/` | submitted → draft |
| Reprendre | `resume/` | rejected → draft |

Succès : 200, représentation complète du détail privé existant, références
filtrées selon l'appelant. GET/PUT/PATCH/DELETE sur ces nouvelles routes ne
sont pas ouverts. Une action répétée après réussite est refusée en 400 pour
état de départ incorrect, sans nouvelle date ni autre écriture.

Corps absent ou objet JSON vide `{}` accepté ; tout objet non vide ou corps
JSON non objet donne 400. Aucun champ status, date, acteur, note de revue,
profil ou contenu métier accepté. Ne pas ignorer silencieusement un payload.
Pas de serializer d'entrée contenant des champs fictifs ; documenter les
deux formes vides acceptées dans OpenAPI.

## 2. Droits : appliquer l'ADR 0002, pas le droit de PATCH

Trois acteurs possibles pour chacune des trois actions : contributeur
propriétaire ; relecteur ; administrateur. Relecteur et administrateur
peuvent agir sur une fiche sans propriétaire. Aucun transfert de propriété.
Garder JWT uniquement, groupes en base, cumul des rôles, superutilisateur
actif administrateur ; is_staff seul ne donne aucun rôle métier.

Ordre de contrôle : authentification/rôle, sélection privée de la fiche,
droit de l'acteur, puis état de départ et validation du corps.

- JWT absent/invalide/expiré : 401 ; aucun groupe métier : 403.
- Fiche inconnue, non lisible ou supprimée : même 404, même pour l'admin
  sur une fiche supprimée. Aucun nom ou état masqué divulgué.
- Fiche lisible mais acteur non autorisé : 403.
- Acteur autorisé, mauvais statut de départ ou payload invalide : 400,
  message anglais explicite, aucune mutation.

**Piège à éviter :** `CanEditEntity`, `is_privately_editable` et
`locked_record()` protègent les écritures ordinaires. Ils refusent justement
la modification d'un submitted par son propriétaire ; ils ne peuvent donc
pas décider tels quels du droit de withdraw. Distinguer explicitement les
actions de transition, sans affaiblir ces protections sur les anciennes routes.
Ne pas donner à tout appelant authentifié un droit de transition par défaut.

## 3. État enregistré et atomicité

Conventions de cet incrément (à suivre, pas à réinventer pendant le code) :

- submit : status=submitted, submitted_at=timezone.now(), reviewed_at=null,
  reviewed_by=null et review_notes="". Une nouvelle soumission commence une
  nouvelle revue ; aucune décision ancienne ne doit sembler la concerner.
- withdraw : status=draft et submitted_at=null ; les champs de revue restent
  inchangés. L'action est un retrait, pas un rejet.
- resume : status=draft et submitted_at=null ; conserver reviewed_at,
  reviewed_by et review_notes du rejet précédent pour guider les corrections.
  Ils seront réinitialisés à la prochaine soumission.
- updated_at est actualisé uniquement au succès ; propriétaire, type,
  champs métier, profil, référentiels, sources/relations, marques de
  suppression et created_at sont inchangés.

Ces champs représentent l'état courant et la dernière revue conservée, pas
un historique exhaustif. Ne créer ni table d'audit ni nouveaux champs.
L'incrément 6 renseignera les décisions de modération ; ne pas le devancer.

Transaction atomique : retrouver puis verrouiller l'entité, relire sa visibilité,
ses droits et son statut sur l'état verrouillé, avant toute mutation. Deux
soumissions concurrentes ne doivent pas réussir toutes les deux. Une demande
refusée ne modifie aucun champ, y compris dates et notes. Réutiliser les
fonctions de politique existantes lorsque leur sens convient ; pas de validation
contre un objet périmé, pas de capture générale transformant toute erreur en 400.

La soumission n'est pas une publication ni une certification documentaire.
Ne pas inventer de seuil de complétude (description obligatoire, nombre de
sources, etc.) ou de réparations implicites : aucun n'a été décidé. Ne pas
remplir automatiquement des données métier ou créer un profil au passage.

Après submit, le contributeur conserve la lecture mais ne peut modifier ni
la fiche, ni ses sources/relations. Le relecteur garde les droits d'écriture
submitted déjà prévus. Après withdraw/resume, les droits ordinaires de draft
redeviennent applicables, sans nouveau mécanisme parallèle.

## 4. Restrictions impératives de périmètre et de sobriété

- **Interdit d'anticiper l'incrément 6/7 :** aucune publication, aucun rejet,
  aucune réouverture published → draft, suppression/restauration d'entité.
- Aucun PATCH ordinaire de status ou marques éditoriales ; aucun changement
  du contrat public, de création, PATCH, sources ou relations hors intégration
  strictement nécessaire des trois actions.
- Aucun changement de modèle/migration, import, comptes/groupes, JWT,
  administration Django ou données de développement. Garder 20 minutes en
  développement et 5 hors debug ; ne lire ni afficher de secrets ou .env.
- Aucun moteur de workflow/FSM, bibliothèque supplémentaire, registre de
  transitions futures, base générique, repository, signal ou hook anticipé.
  Une petite fonction métier concrète partagée est légitime, pas une plateforme.
- Ne pas réorganiser globalement les serializers/views ni traduire les anciens
  modules. Leur découpage est un sujet distinct. Si un petit module métier de
  transitions évite de grossir les serializers, l'utiliser sans architecture
  supplémentaire ; les vues restent des adaptateurs courts.
- Réutiliser validateurs et représentations existants ; pas de double validation
  sans risque concret. Docstrings anglaises courtes ; commentaires français
  seulement pour expliquer un choix non évident. Pas de longue dissertation
  en tête de module ou de contrat recopié dans chaque méthode.

## 5. Tests utiles et plan d'exécution obligatoire

Avant de coder, annoncer brièvement les fichiers envisagés et les contrôles
prévus. Si une décision incompatible ou un besoin hors périmètre apparaît,
le signaler plutôt que l'implémenter par défaut. Ne pas attendre un accord
pour les étapes normales déjà autorisées par ce prompt.

Tester les trois transitions, leurs droits distincts, les états interdits,
les horodatages et notes, la répétition sans mutation, le corps refusé et la
lecture filtrée. Utiliser des données synthétiques et les fabriques existantes.
Vérifier en particulier withdraw par le propriétaire d'un submitted,
relecteur sur une fiche sans propriétaire, autres contributeurs masqués,
fiches supprimées, et réévaluation sur l'état verrouillé.

Un parcours ciblé doit montrer que submit bloque PATCH et écritures d'enfants
pour le contributeur, puis que withdraw rétablit ces droits. Vérifier aussi
la reprise d'un rejet et l'effacement de l'ancienne revue lors de la nouvelle
soumission. Ne pas recopier la matrice 4A complète pour chaque route ni
retester toute la mécanique JWT. Couvrir les risques distincts, pas un quota
de cas ou les détails d'organisation interne.

**Ordre des vérifications :**
1. Pendant l'implémentation : nouveaux tests ciblés et éventuels anciens
   tests précis directement affectés par un changement.
2. Avant le passage final : check, makemigrations --check --dry-run,
   spectacular --validate --fail-on-warn ; Ruff/format et mypy sur le périmètre.
3. Un seul passage complet final de pytest sur l'état fonctionnel livré.
   **Ne pas lancer séparément les anciennes suites création/PATCH/lecture/
   enfants juste avant.** Elles sont déjà incluses dans ce passage complet.
4. Après correction : uniquement les contrôles affectés. Un nouveau passage
   complet doit être justifié par un changement transversal ou un risque précis,
   pas par une retouche de rapport, docstring, README ou format.

Ne pas lancer plusieurs suites en parallèle contre la même base de tests.
Ne pas relancer une commande en cours parce qu'elle reste silencieuse.
Si un contrôle est lent, indiquer lequel et son état observable, sans
annoncer un blocage ou une réussite non vérifiés. Garder une trace concise
des relances et de leur raison dans le rapport final.

## 6. Documentation et remise, rien de plus

Actualiser seulement les passages concernés du README et OpenAPI en anglais :
trois routes, corps vide, droits, états de départ, dates/revue, réponse privée,
erreurs et limites. Ne pas annoncer une modération/publication déjà disponible.

Rapport factuel **court**, sans répétition du prompt :
`docs/reports/phase-4-increment-5-submit-resume.md`. Fichiers touchés, décisions
concrètes, résultats réellement exécutés (dont durée de la suite complète),
raisons des relances éventuelles, limites/blocages et état Git. Copie identique
dans le scratchpad : `rapport-pour-codex-phase-4-increment-5-submit-resume.md`.
Fournir les deux chemins à la fin.

**Ne produire aucun guide Postman/ORM, collection exportée, nouvelle ADR,
roadmap, rapport secondaire ou démo en base. Ne rien cocher, committer ni
pousser.** Codex assure revue, corrections légères, pédagogie, exercices et
clôture après validation de l'auteur. Terminer après la livraison de cet
incrément ; ne pas enchaîner sur la modération ou un refactoring général.
