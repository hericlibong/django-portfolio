# Incrément 7 — suppression logique, restauration et verrous

Explications pédagogiques de Codex, 1er novembre 2026. Lecture différée :
ce fichier remplace l'explication bloc par bloc dans le chat. Les exercices
Postman sont validés le 1er novembre 2026 ; le rapport technique est dans le dépôt API :
`docs/reports/phase-4-increment-7-soft-delete-restore.md`.

## 1. Supprimer ne signifie pas effacer la ligne

Dans [mythology/deletion.py](C:/Users/heric/dev/afrimyth-api/mythology/deletion.py),
`apply_deletion()` n'appelle jamais delete :

```python
entity.deleted_at = timezone.now()
entity.deleted_by = actor
entity.save(update_fields=["deleted_at", "deleted_by", "updated_at"])
```

Le serveur enregistre qui supprime et quand. Le statut reste draft, submitted,
rejected ou published : aucun statut deleted n'est ajouté. Profil, sources,
relations et référentiels sont conservés. Le 204 signifie que l'opération
a réussi sans corps de réponse, pas qu'une ligne a disparu physiquement.
Les DELETE d'une source ou d'une relation, eux, gardent leur suppression
physique individuelle ; il faut distinguer les deux types de ressource.

## 2. Le droit n'est pas celui de modifier

`may_delete()` reprend la matrice décidée :
- contributeur sur ses seuls draft/rejected ;
- relecteur sur tous les draft/rejected ;
- administrateur sur tous les statuts non supprimés.

Un relecteur peut modifier un submitted, mais pas le supprimer : réutiliser
le droit de PATCH serait incorrect. Dans les permissions, destroy/restore
font donc partie des actions qui appliquent leur propre contrôle, sans
désactiver les protections des anciennes écritures ou transitions.

## 3. DELETE sélectionne la fiche, verrouille puis contrôle

Dans [mythology/views.py](C:/Users/heric/dev/afrimyth-api/mythology/views.py),
`destroy()` utilise d'abord le slug pour sélectionner une fiche privée lisible.
Il ouvre une transaction, relit cette fiche sous verrou avec le filtre privé,
contrôle `may_delete()`, refuse un Body non vide et cherche les variantes
actives avant d'appeler `apply_deletion()`.

Une fiche masquée ou déjà supprimée donne 404 ; une fiche lisible mais non
supprimable donne 403. Les valeurs d'acteur/date ne viennent jamais du Body.
Une exception annule l'écriture de la transaction.

Les lectures habituelles filtrent déjà deleted_at : après DELETE, même un
administrateur ne retrouve pas la fiche par les GET privés ordinaires.
Un second DELETE donne donc 404, sans changer les marques initiales.

## 4. Restaurer : administrateur avant recherche

`restore()` vérifie d'abord le rôle administrateur. Sans ce rôle, même
réponse 403 pour un slug existant ou inconnu : l'API ne révèle pas la corbeille.
Pour l'administrateur seulement, la recherche étroite par slug utilise
`Entity.objects`, le manager complet, car le filtre privé exclut les supprimés.
Cela n'élargit pas les autres routes et ne crée aucune liste de corbeille.

Après verrouillage, une fiche non supprimée donne 400. Si c'est une variante,
sa référence doit déjà être active : on restaure la référence avant la variante.
Puis `apply_restoration()` écrit :

```python
entity.deleted_at = None
entity.deleted_by = None
entity.save(update_fields=["deleted_at", "deleted_by", "updated_at"])
```

Le statut est conservé. **Restaurer un published le rend de nouveau public ;
restaurer un draft le laisse privé.** Ce n'est pas une réouverture éditoriale,
qui ferait passer un published en draft, ni une nouvelle décision de revue.
Les dates et notes de revue ainsi que le propriétaire restent intacts.

## 5. Une variante active protège sa référence

`has_active_variant()` cherche une variante directe avec deleted_at null,
quel que soit son statut ou sa visibilité. Sa présence bloque la suppression
de la référence en 400 ; le message ne donne pas de nom ou identifiant privé.
Une variante déjà supprimée logiquement ne bloque pas. Les simples relations
entre figures ne bloquent pas non plus : cette protection concerne variant_of.

Ce contrôle doit être coordonné avec les écritures : sinon une variante
pourrait être créée juste après la recherche et avant la suppression.

## 6. Pourquoi l'ordre des verrous a été corrigé

La première version verrouillait la fiche puis sa future référence. Deux
PATCH croisés pouvaient prendre A puis B et B puis A et s'attendre mutuellement.
La correction utilise `locked_in_pk_order()` : les lignes nécessaires sont
acquises ensemble dans l'ordre de leur clé primaire, identique pour tous.

```python
locked = Entity.objects.select_for_update().filter(pk__in=wanted).order_by("pk")
```

Le PATCH prend ainsi sa fiche et la référence fournie ; restore prend la
fiche et sa référence ; create prend la référence. DELETE, qui vérifie les
variantes, tient le verrou de la fiche de référence. Les vérifications finales
portent sur les objets verrouillés, pas ceux lus avant la transaction.

Ce n'est pas un moteur de workflow ou un verrou global. Les cycles indirects
de variantes restent possibles selon le modèle existant ; leur interdiction
n'est pas ajoutée pour contourner un problème technique.

Un test à deux connexions réelles et attente bornée couvre les PATCH croisés.
Les autres tests de fenêtre entre validation et verrou sont des simulations,
pas une preuve que tous les entrelacements PostgreSQL ont été exercés.

## 7. Limites et pratique ciblée

Pas de purge, de journal d'événements, d'ETag ou de nouveau champ. Les garanties
de coordination concernent les chemins API, pas le SQL direct ou l'admin.
Une restauration efface les marques de suppression ; elle ne conserve pas
un historique exhaustif des suppressions passées.

Pratique réalisée sur demo-create-hero-a uniquement : lecture initiale,
DELETE propriétaire en 204, GET et second DELETE en 404, restore contributeur
en 403, restauration par l'administrateur avec fiche accessible, et répétition
en 400. La fiche est conservée restaurée ; aucun nouvel enfant ou compte,
aucune purge et aucun changement du corpus. La conservation détaillée des
colonnes, les variantes, le cas published et la concurrence restent couverts
automatiquement, sans être annoncés comme tous exécutés manuellement.
