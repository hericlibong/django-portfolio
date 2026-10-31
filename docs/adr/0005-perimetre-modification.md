# ADR 0005 — Périmètre de la modification ordinaire

- **Statut :** accepté par l'auteur le 30 octobre 2026
- **Date :** 30 octobre 2026
- **Portée :** phase 4, incrément 4 ; aucune implémentation effectuée ici

## Décision validée

Le PATCH ordinaire ne change pas `entity_type`. Le profil correspondant au
type existant peut être enrichi, sans changer son rattachement ou son identité.
La présence de `entity_type` dans un PATCH est refusée explicitement en 400.

Motif : passer d'une divinité à un héros ne se réduit pas à changer une valeur.
Cela exige de décider du devenir des données de culte et du nouveau profil.
Aucune conversion ni suppression implicite de profil n'est acceptable.
Ce changement pourra faire l'objet d'une opération de reclassement définie
séparément ; elle n'est pas implémentée par anticipation.

Cette limite ne fige pas les classifications mythologiques et n'impose pas
qu'une variante ait le même type que sa figure de référence (ADR 0002).

## Découpage validé

- **4A :** PATCH de l'entité, du profil existant et des rattachements aux
  référentiels ; permissions, atomicité et protection des champs de plateforme.
- **4B :** écritures des sources et relations, avec contrat dédié avant
  implémentation. Elles restent dans l'incrément 4, comme annoncé précédemment.

Les droits par rôle, propriétaire et statut ne sont pas rediscutés : la matrice
de l'ADR 0002 reste applicable. La suppression et les transitions de statut
restent dans leurs incréments respectifs.
