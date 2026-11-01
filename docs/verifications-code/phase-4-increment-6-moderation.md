# Incrément 6 — explications du code de modération

Document pédagogique de Codex, 1er novembre 2026. Lecture différée facultative
avant Postman : les explications ne sont plus déroulées bloc par bloc dans
le chat. Ce document ne remplace pas le rapport de revue technique de l'API,
`docs/reports/phase-4-increment-6-moderation.md`. Les exercices sont validés
le 1er novembre 2026 ; l'incrément est terminé.

## Explication du fonctionnement principal

L'incrément 6 **complète les transitions existantes**, sans créer un second
mécanisme. Voici les trois changements essentiels.

### 1. Les droits dépendent maintenant de l'action

Dans [mythology/transitions.py](C:/Users/heric/dev/afrimyth-api/mythology/transitions.py) :

```python
PUBLISH: frozenset({REVIEWER, ADMINISTRATOR}),
REJECT: frozenset({REVIEWER, ADMINISTRATOR}),
REOPEN: frozenset({ADMINISTRATOR}),
```

La fonction `may_transition()` reçoit désormais **l'action demandée** :

```python
if roles & ALLOWED_ROLES[action]:
    return True
```

La propriété n'autorise que les anciennes actions `submit`, `withdraw` et
`resume`. **Un contributeur ne peut donc pas publier son propre travail ;
un relecteur ne peut pas rouvrir une publication.**

### 2. Les notes sont validées avant la décision

Dans [mythology/serializers.py](C:/Users/heric/dev/afrimyth-api/mythology/serializers.py),
deux petits serializers distinguent les entrées :

```python
# Publication : note facultative, vide autorisé
review_notes = serializers.CharField(required=False, allow_blank=True)

# Rejet : motif obligatoire, vide refusé
review_notes = serializers.CharField()
```

DRF retire les espaces périphériques : un motif composé uniquement d'espaces
est donc refusé.

Les autres clés sont explicitement rejetées. Le client ne peut pas fournir
son propre `reviewed_by`, sa date de revue ou le statut souhaité.

### 3. La décision enregistre la revue côté serveur

Dans `apply_transition()` :

```python
elif action in DECISIONS:
    entity.status = (
        EntityStatus.PUBLISHED if action == PUBLISH else EntityStatus.REJECTED
    )
    entity.reviewed_at = timezone.now()
    entity.reviewed_by = actor
    entity.review_notes = review_notes
```

**Le serveur fournit l'acteur et la date** ; seule la note vient du corps
validé. La date de soumission est conservée.

Pour `reopen`, la branche de retour au brouillon est réutilisée : statut
`draft`, `submitted_at = None`, dernière revue conservée. Une nouvelle
soumission effacera cette ancienne revue.

Dans [mythology/views.py](C:/Users/heric/dev/afrimyth-api/mythology/views.py),
`run_transition()` garde l'enchaînement déjà expliqué : **verrou → droit →
statut de départ → validation du corps → sauvegarde ciblée**.

Enfin, **aucun nouveau code de visibilité publique n'est nécessaire** : le
filtre existant expose les fiches `published`. Publier les rend visibles ;
rouvrir les masque à nouveau.

## Complément : où la vue valide la note

`validated_review_notes()` choisit `PublishNotesSerializer` ou
`RejectNotesSerializer` via les deux entrées de `DECISION_SERIALIZERS`.
Elle appelle `is_valid(raise_exception=True)`, puis récupère la note validée.
Une publication sans note enregistre une chaîne vide, pas une ancienne note.
Pour submit/withdraw/resume/reopen, elle garde le contrat sans entrée : un
corps absent ou `{}` est accepté ; toute autre donnée est refusée.

La validation intervient dans la transaction, après la sélection privée,
le droit propre à l'action et le statut de départ. Un appelant refusé ne
reçoit donc pas un diagnostic sur son motif ou sur l'état d'une fiche masquée.
`update_fields` limite l'écriture aux champs éditoriaux concernés et à
`updated_at`, sans toucher au propriétaire, au type, au profil ou aux enfants.

## Complément : les routes et la réponse

Les trois méthodes publish/reject/reopen du viewset privé suivent le schéma
des routes submit/withdraw/resume : `@action(detail=True, methods=["post"],
...)` ajoute une action au détail identifié par slug. Elles délèguent à
`run_transition()` ; les annotations `@extend_schema` décrivent l'API sans
accorder de permission. Publish/reject déclarent leur serializer de notes ;
reopen ne déclare pas de corps d'entrée.

La réponse 200 est le détail privé existant, avec références filtrées pour
l'appelant. La lecture publique ne contient jamais les notes ou acteurs de
revue, même quand une publication vient de rendre la fiche visible.

## Limites à garder en tête

- La propriété seule ne permet aucune décision de modération. La matrice
  autorise toutefois un relecteur à publier sa propre fiche : aucune règle
  supplémentaire d'auto-revue ou de double validation n'a été décidée.
- Il n'existe pas d'historique exhaustif : le modèle conserve l'état courant
  et la dernière revue selon les transitions prévues.
- Les simulations de concurrence des tests prouvent la relecture de l'état ;
  elles ne constituent pas des essais multithread de PostgreSQL.
- Suppression et restauration d'entité restent hors de l'incrément 6.

## Pratique validée le 1er novembre 2026

Sur la seule fiche fictive demo-create-hero-a : point de départ, soumission
et refus de publication par le contributeur, refus d'un rejet sans motif,
rejet motivé, reprise et resoumission, publication par le relecteur et
visibilité publique, reopen refusé au relecteur, puis réouverture par
l'administrateur et disparition publique. La dernière réponse privée confirme
la conservation de la revue (relecteur 5) et du propriétaire 3. L'absence des
champs de revue dans le public reste une garantie automatiquement testée,
sans confirmation manuelle détaillée distincte. État final : draft conservé,
sans création d'enfant ni changement des fiches du corpus. Les objectifs
de chaque contrôle ont été annoncés avant les manipulations.
