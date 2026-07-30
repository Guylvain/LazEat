---
id: pate-a-pizza-maison
nom: Pâte à pizza maison
photo_placeholder: assets/photos-recettes/pate-a-pizza-maison.jpg
photo_perso: null

type_production: composant

famille: pizza

creneaux_compatibles:
  - batch
  - soir-cuisine
  - diner-a-deux

temps_actif: 22
temps_total: 210

portions_base: 4             # 1 portion = 1 pâton = 1 pizza de 28-30 cm
portions_max_reel: 8
note_scaling: >
  Le sel et la levure suivent la farine proportionnellement.
  Au-delà de 8 pâtons, pétrir en deux fois : la masse devient
  ingérable à la main.

batchable: true
portions_max_batch: 8
qualite_j2: excellente       # la pâte est meilleure après 24-72 h de fermentation froide
congelable: true
duree_conservation_frigo: 3

bon_froid: false
transportable: false
ustensiles_requis:
  - four
  - plaque
  - grand-saladier
  - balance

registre: convivial

cuisine_a_deux:
  adapte: true
  niveau: bon
  postes:
    - nom: Pétrissage et pousses
      etapes: [1, 2, 3, 4]
    - nom: Boulage et cuisson
      etapes: [5, 6, 7, 8, 9]
  moment_partage: "L'étalage des pâtons à la main, chacun le sien."

tags_nutritionnels:
  - glucides-lents

statut: a-tester
note: null
derniere_execution: null
nb_executions: 0
source: null
---

# Pâte à pizza maison

Recette pivot du système. La pizza est le plat le plus commandé en livraison, et le mercredi soir — jour de craquage identifié — devient le jour de production. Une session produit quatre pâtons : un ou deux cuits le soir même, le reste congelé pour couvrir le mardi suivant, créneau critique de la semaine.

Coût de revient : environ 0,30 € le pâton, contre 12,50 € la pizza livrée même en offre promotionnelle.

## Ingrédients

| ingredient_id | quantité | unité | optionnel | substituts |
|---|---|---|---|---|
| `farine-t65` | 600 | g | non | `farine-t55`, `farine-00` |
| `eau` | 380 | ml | non | — |
| `sel-fin` | 15 | g | non | — |
| `levure-boulangere-seche` | 3 | g | non | `levure-boulangere-fraiche` (9 g) |
| `huile-olive` | 20 | ml | non | — |

Hydratation à 63 %, volontairement modérée : une pâte plus hydratée donne un meilleur résultat mais devient difficile à manipuler sans pratique.

### Garniture, par pizza — recette séparée

| ingredient_id | quantité | unité |
|---|---|---|
| `tomates-concassees` | 60 | g |
| `mozzarella` | 125 | g |
| `origan-seche` | 1 | g |

## Étapes

### 1. Réveiller la levure
`minuteur: 300`

Délayer `{levure-boulangere-seche}` dans `{eau}` tiède, autour de 25 °C. Laisser reposer sans intervenir.

### 2. Mélanger
`minuteur: 180`

Verser `{farine-t65}` dans un grand saladier, ajouter `{sel-fin}` sur un côté et `{huile-olive}`. Incorporer l'eau et mélanger à la main jusqu'à disparition de la farine sèche. La pâte colle à ce stade, c'est normal.

### 3. Pétrir
`minuteur: 480`

Pétrir sur le plan de travail en étirant et repliant. **Ne pas ajouter de farine** même si la pâte colle : elle devient lisse d'elle-même après quelques minutes. Elle est prête quand elle s'étire sans se déchirer.

### 4. Première pousse
`minuteur: 7200`

Mettre en boule dans le saladier huilé, couvrir d'un torchon humide, laisser à température ambiante jusqu'à doublement du volume. Aucune intervention.

### 5. Diviser et bouler
`minuteur: 300`

Dégazer doucement, diviser en 4 parts égales d'environ 255 g. Bouler chaque part en ramenant les bords sous la boule pour tendre la surface.

### 6. Deuxième pousse
`minuteur: 3600`

Disposer les pâtons espacés sur un plateau fariné, couvrir. **Congeler ici les pâtons non utilisés le jour même.**

### 7. Préchauffer à fond
`minuteur: 1800`

Placer la plaque **retournée** dans le tiers inférieur du four. Chauffer à 250-275 °C, chaleur classique de préférence. La plaque doit être brûlante : elle remplace la masse thermique d'une pierre ou d'un acier.

### 8. Étaler et garnir
`minuteur: 300`

Étaler un pâton **à la main** sur du papier cuisson, en pressant du centre vers l'extérieur et en préservant un bourrelet de 2 cm. Garnir légèrement.

### 9. Cuire
`minuteur: 540`

Faire glisser la pizza avec son papier sur la plaque brûlante. Au bout de 3 minutes, retirer le papier d'un coup sec — la pâte finit de cuire au contact direct. Sortir quand le bourrelet est coloré et le fromage boursouflé.

## Notes

**Ne jamais utiliser de rouleau.** Il écrase les bulles de gaz produites par la fermentation et donne une pâte plate et dense. C'est l'erreur qui ruine le plus de pizzas maison.

**Mozzarella** — l'égoutter et la couper en dés au moins 30 minutes avant. Sinon elle rend son eau pendant la cuisson et détrempe la pâte.

**Sauce** — tomates concassées crues, simplement salées et relevées d'origan. Une sauce déjà cuite recuit au four et devient amère.

**Congélation** — huiler légèrement chaque pâton, l'emballer individuellement à plat. Trois mois de conservation. Sortir le matin pour le soir, décongélation à température ambiante, jamais au micro-ondes.

**Version supérieure** — après le boulage, placer les pâtons au frigo 24 à 72 heures au lieu d'une heure à température ambiante. La fermentation lente développe nettement plus de goût. Sortir 2 heures avant d'étaler.

**Équipement** — la plaque retournée et préchauffée donne un résultat correct. Un acier à pizza (40-60 €) reste la seule amélioration matérielle qui vaille, à envisager après un mois de pratique.
