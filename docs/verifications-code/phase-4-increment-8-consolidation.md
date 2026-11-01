# Incrément 8 — explication de la consolidation

Codex, 1er novembre 2026. Phase 4 clôturée avec accord explicite de l'auteur,
sans exercice supplémentaire. Ce document est destiné à une lecture différée.

## Aucun nouveau code métier

Les droits et comportements sont déjà implémentés aux incréments 1 à 7.
Le seul ajout est un test dans
[test_private_deletion.py](C:/Users/heric/dev/afrimyth-api/mythology/tests/test_private_deletion.py),
qui enchaîne les routes réelles là où les anciens tests préparaient directement
une fiche déjà supprimée dans la fabrique de test.

## Ce que le nouveau scénario vérifie

Il crée une fiche avec son profil via POST, la soumet, la publie, vérifie sa
visibilité publique, la supprime via DELETE, constate son absence publique et
privée, puis appelle restore et enfin reopen. Les statuts ne sont pas forcés
par une mise à jour directe en base dans ce scénario.

```python
assert api(admin).delete(detail_url(slug)).status_code == 204
assert client.get(public).status_code == 404
assert api(admin).get(detail_url(slug)).status_code == 404

restauree = api(admin).post(action("restore"))
assert restauree.status_code == 200
assert restauree.json()["status"] == EntityStatus.PUBLISHED
```

L'intérêt est l'articulation entre les opérations : restore doit effacer les
marques que le vrai DELETE a écrites et rendre la fiche utilisable par les
routes privées de modération. Le test vérifie aussi deleted_at/deleted_by
effacés, propriétaire et relecteur conservés, puis disparition publique après
reopen. Il ne réécrit pas toute la matrice de permissions.

## Administration et limites

Le changement de `mythology/admin.py` n'est qu'un commentaire plus exact.
Les formulaires permettent toujours une Entity sans profil spécialisé et une
suppression physique. Ces limites sont reportées avec accord de l'auteur,
car ce back-office est réservé à la maintenance technique, pas à la saisie
éditoriale des utilisateurs. Décision et solutions futures dans
[la note dédiée](C:/Users/heric/dev/django-portfolio/docs/notes/admin-et-interface-editoriale.md).

## Validation

1 110 tests réussis, sans relance complète. Les exercices de phase 4 ont déjà
été validés au fil des incréments. Aucun nouveau développement, correction
de corpus ou test Postman supplémentaire n'est imposé pour cette clôture.
