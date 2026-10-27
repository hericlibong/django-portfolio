# ADR 0004 — Conventions de rôles pour la lecture privée

- **Statut :** accepté, conventions validées explicitement par l'auteur
- **Date :** 24 octobre 2026
- **Portée :** phase 4, incrément 2 ; complète la matrice de l'ADR 0002

## Objectif

Créer les groupes Django `contributor`, `reviewer`, `administrator` et une
consultation privée selon le rôle et la propriété. Ne pas changer la lecture
publique, créer d'inscription publique ou ouvrir les écritures métier.

La matrice existante décide déjà des droits de lecture. Les trois conventions
d'affectation ci-dessous sont validées avant leur implémentation.

## Conventions retenues

| Cas | Décision validée | Motif |
|---|---|---|
| Compte connecté sans groupe métier | Pas d'accès aux routes privées ; consultation publique conservée | Se connecter ne doit pas attribuer implicitement le rôle de contributeur |
| Compte superutilisateur | Traité comme administrateur métier même sans groupe ; `is_staff` seul ne donne aucun rôle métier | Conserver un accès d'administration de secours, sans assimiler accès à l'admin Django et droits sur le corpus |
| Plusieurs groupes métier | Droits cumulés ; dans la matrice actuelle, le rôle le plus élevé suffit | Ne pas créer de conflit de rôles ni de champ `role` en doublon ; les permissions sont évaluées depuis les groupes en base |

Les groupes sans rapport avec ces trois rôles sont sans effet sur les droits
métier. Aucun groupe n'est automatiquement attribué aux comptes déjà présents.
La migration crée seulement les groupes ; les affectations de démonstration
seront explicites dans l'administration.

## Traduction de la matrice, sans nouvelle décision métier

- Contributeur : fiches publiées et ses propres `draft`, `submitted`, `rejected`.
- Relecteur et administrateur : toutes les fiches non supprimées, y compris
  celles sans propriétaire. Le contributeur ne possède pas implicitement une
  fiche importée dont `created_by` vaut `null`.
- Les suppressions logiques restent exclues de ces lectures ordinaires, même
  pour l'administrateur ; leur restauration sera traitée à l'incrément prévu.
- Sur les routes publiques, tout appelant continue à ne voir que le contenu
  publié non supprimé, même s'il est relecteur ou superutilisateur.
- Dans la lecture privée, une référence structurée ne peut révéler qu'une
  cible lisible par l'appelant. Masquer une cible non autorisée ne doit pas
  recopier son nom ailleurs. La politique publique existante reste inchangée.

## Suite

Implémentation confiée à Claude CLI par le prompt
`docs/prompts/phase-4-increment-2-private-read.md`, après lecture par l'auteur.
La validation de cette ADR ne vaut pas validation d'une implémentation :
aucun code ni migration n'est exécuté pendant sa préparation.
