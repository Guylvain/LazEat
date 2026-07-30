---
id: faux-filet-grenailles
nom: Faux-filet et pommes grenailles
photo_placeholder: assets/photos-recettes/faux-filet-grenailles.jpg
photo_perso: null

type_production: repas

famille: null

creneaux_compatibles:
  - soir-cuisine
  - diner-a-deux

temps_actif: 25
temps_total: 35

portions_base: 2
portions_max_reel: 3
note_scaling: >
  ATTENTION — portions dissociées. La viande ne se réchauffe pas,
  les grenailles sont excellentes le lendemain. Cuisiner 1 pièce
  de viande par personne présente, mais toujours 500 g de grenailles
  minimum : le surplus devient un accompagnement pour le repas suivant.

batchable: false
portions_max_batch: null
qualite_j2: a-eviter
congelable: false
duree_conservation_frigo: 1

bon_froid: false
transportable: false
ustensiles_requis:
  - poele
  - casserole

registre: soigne

cuisine_a_deux:
  adapte: true
  niveau: excellent
  postes:
    - nom: Grenailles
      etapes: [2, 3]
    - nom: Viande et timing
      etapes: [1, 4, 5]
  moment_partage: "La cuisson de la viande, à surveiller à deux pendant que les grenailles rissolent."

tags_nutritionnels:
  - proteines
  - glucides-lents

statut: validee
note: null
derniere_execution: null
nb_executions: 0
---

# Faux-filet et pommes grenailles

Plat déjà réalisé, intégré au socle. Recette du jeudi soir ou du samedi.

**Cas particulier du modèle de données** : c'est la première recette où les portions se dissocient. Le faux-filet réchauffé devient gris et coriace, alors que les grenailles repassées à la poêle sont meilleures le lendemain. Une pièce de viande par convive, mais toujours 500 g de grenailles.

## Ingrédients

| ingredient_id | quantité | unité | optionnel | substituts |
|---|---|---|---|---|
| `faux-filet` | 2 | piece | non | `entrecote`, `bavette` |
| `pommes-grenailles` | 500 | g | non | `pommes-de-terre-rattes` |
| `ail` | 4 | piece | non | — |
| `thym-seche` | 2 | g | non | `herbes-provence` |
| `beurre-demi-sel` | 30 | g | non | `beurre-doux` |
| `huile-olive` | 20 | ml | non | — |
| `sel-fin` | 4 | g | non | — |
| `poivre-noir` | 2 | g | non | — |

## Étapes

### 1. Sortir la viande
`minuteur: 1800`

Sortir `{faux-filet}` du frigo trente minutes avant de cuire. Une viande à température ambiante cuit uniformément ; sortie du frigo, elle reste froide au centre pendant que l'extérieur brûle.

### 2. Précuire les grenailles
`minuteur: 720`

Couper `{pommes-grenailles}` en deux. Les faire démarrer douze minutes dans l'eau bouillante salée, puis égoutter et laisser sécher deux minutes à l'air.

### 3. Rissoler les grenailles
`minuteur: 720`

Poêle à feu moyen-vif avec `{huile-olive}`. Verser les grenailles face coupée vers le bas, sans y toucher pendant quatre minutes. Ajouter `{ail}` en chemise et `{thym-seche}`, puis retourner régulièrement jusqu'à belle coloration. Réserver au chaud.

### 4. Saisir la viande
`minuteur: 300`

Poêle très chaude. Sécher `{faux-filet}` au papier absorbant, saler et poivrer généreusement. Saisir sans bouger : deux minutes par face pour un steak de 2 cm saignant, trois pour à point. Ajouter `{beurre-demi-sel}` en fin de cuisson et arroser la viande à la cuillère.

### 5. Repos
`minuteur: 300`

**Sortir la viande et la laisser reposer cinq minutes** sur une assiette tiède avant de couper. Non négociable : sans repos, tout le jus s'échappe à la découpe.

## Notes

**Sécher la viande avant de la saisir.** Une surface humide produit de la vapeur au lieu d'une croûte.

**Le repos de cinq minutes** est l'étape que tout le monde saute et qui fait la plus grande différence sur le résultat final.

**Les grenailles du lendemain** se repassent trois minutes à la poêle. Elles accompagnent parfaitement des œufs ou un reste de poulet.
