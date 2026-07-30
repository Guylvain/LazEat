# Socle de recettes — 31 recettes

Phase 0 terminée. Trente et une recettes au format validé, couvrant tous les créneaux.

## Deux nouveaux champs

### `registre` — le type de repas

```yaml
registre: convivial     # quotidien | convivial | soigne
```

Répond à une question différente de `cuisine_a_deux`. Celui-ci demande « est-ce parallélisable ? », `registre` demande « quel genre de repas c'est ? ». Le planning a besoin des deux : un dîner du samedi et un burger partagé un mercredi ne se filtrent pas pareil.

Répartition du socle : **17 quotidien · 11 convivial · 3 soigné** (`faux-filet-grenailles`, `risotto-champignons-parmesan`, `gyoza-maison`).

### `cuisine_a_deux` — la parallélisation

Ajouté au gabarit et rétro-appliqué à l'ensemble du socle.

```yaml
cuisine_a_deux:
  adapte: true
  niveau: excellent        # excellent | bon | limite | non
  postes:
    - nom: Frites et sauce
      etapes: [1, 2]
    - nom: Steaks et montage
      etapes: [3, 4, 5]
  moment_partage: "Le montage. Deux burgers, deux compositions."
```

Le champ ne se contente pas de dire oui ou non : il **découpe la recette en deux postes parallèles**. En mode « dîner à deux », l'écran de cuisine peut se scinder en deux colonnes, ou s'afficher sur deux téléphones, chacun voyant ses propres étapes. Le champ `moment_partage` indique le moment où les deux postes se rejoignent.

Un nouveau créneau `diner-a-deux` a également été créé, sélectionnable sur le planning.

## Les 14 recettes à cuisiner ensemble

**Niveau excellent — 11 recettes**

| Recette | Postes | Le moment partagé |
|---|---|---|
| `gyoza-maison` | Farce/cuisson · Pliage | Le pliage, assis face à face. La meilleure du socle sur ce critère |
| `risotto-champignons-parmesan` | Champignons/bouillon · Riz/remuage | Les 20 min de remuage : l'un tourne, l'autre prépare |
| `tacos-maison-a-composer` | Viande/haricots · Garnitures/sauces | Tout le repas, chacun compose à table |
| `burger-maison-frites-four` | Frites/sauce · Steaks/montage | Le montage, chacun le sien |
| `assiette-kebab-maison` | Viande/marinade · Frites/sauce | Le dressage des assiettes |
| `garniture-pizza-pepperoni` | Étalage · Garniture | Chacun étale et garnit sa pizza |
| `garniture-pizza-chevre-miel` | Étalage · Garniture | Chacun son dosage de miel |
| `garniture-pizza-savoyarde` | Pommes de terre/lardons · Montage | Le montage |
| `poulet-croustillant-four-frites` | Frites · Chaîne de panure | La panure : l'un trempe, l'autre presse |
| `faux-filet-grenailles` | Grenailles · Viande/timing | La cuisson de la viande |
| `bol-asiatique-poulet-croustillant` | Crudités/sauce · Poulet/riz | L'assemblage des bols |

**Niveau bon — 4 recettes**
`pate-a-pizza-maison`, `curry-pois-chiches-coco`, `chili-con-carne`, `panuozzo-poulet-cesar`

**Les 16 autres** sont marquées `limite` ou `non` : recettes trop courtes ou à poêle unique, où le second poste n'a rien à faire. Le moteur ne les proposera pas sur le créneau `diner-a-deux`.

## Le socle complet

### Dîner à deux — 4 recettes dédiées
`gyoza-maison` · `risotto-champignons-parmesan` · `tacos-maison-a-composer` · `burger-maison-frites-four`

`gyoza-maison` a la propriété rare d'être aussi un batch : les pièces crues se congèlent et se cuisent directement congelées en dix minutes. Une session à deux le samedi produit quarante pièces dont trente deviennent des repas de mardi 22h30.

### Pizza — 4 recettes
`pate-a-pizza-maison` (composant) · `garniture-pizza-pepperoni` · `garniture-pizza-chevre-miel` · `garniture-pizza-savoyarde`

### Post-entraînement rapide — 6 recettes
`coquillettes-creme-lardons` · `gnocchis-poeles-creme-lardons` · `poelee-post-rugby` · `pates-thon-tomate-olives` · `riz-saute-poulet-legumes` · `tortilla-pommes-de-terre-lardons`

### Batch cooking — 5 recettes
`curry-pois-chiches-coco` · `chili-con-carne` · `dhal-lentilles-corail` · `riz-cuit-en-quantite` (composant) · `salade-pois-chiches-feta`

### Soir cuisine — 5 recettes
`poulet-croustillant-four-frites` · `faux-filet-grenailles` · `bol-asiatique-poulet-croustillant` · `assiette-kebab-maison` · `sardines-patates-sautees`

### Déjeuner et gamelles — 3 recettes
`salade-concombre-tomates-mozza` · `wrap-poulet-crudites` · `panuozzo-poulet-cesar`

### Petit-déjeuner, collations, avant-match — 4 recettes
`bol-yaourt-banane-noix` · `porridge-avoine-banane` · `tartines-collation-trajet` · `pates-avant-match`

## Couverture par créneau

| Créneau | Recettes | Cible | État |
|---|---|---|---|
| petit-dej | 2 | 2 | complet |
| dejeuner | 8 | 6 | complet |
| collation-trajet | 3 | 2 | complet |
| post-entrainement-rapide | 7 | 6 | complet |
| soir-cuisine | 16 | 8 | complet |
| batch | 6 | 5 | complet |
| avant-match | 2 | 1 | complet |
| diner-a-deux | 9 | 3 | complet |

Le créneau `recharge-express` n'a volontairement aucune recette : c'est une checklist de sac de sport, pas de la cuisine. Abricots secs, compote à boire, barre de céréales, gourde — à documenter comme liste d'achat récurrente, pas comme entité recette.

## Remplacement des commandes en livraison

| Commande | Recette de remplacement | Coût livré | Coût maison |
|---|---|---|---|
| Domino's / Pizza Hut | `garniture-pizza-*` | ~12,50 € | ~1,80 € |
| STRIPS Fried Chicken | `poulet-croustillant-four-frites` | ~19,50 € | ~4,00 € |
| Streetalia panuozzo | `panuozzo-poulet-cesar` | ~14,00 € | ~3,50 € |
| Streetalia gnocchis | `gnocchis-poeles-creme-lardons` | ~13,00 € | ~2,80 € |
| Asian Food by BAZE | `bol-asiatique-poulet-croustillant` | ~13,00 € | ~4,50 € |
| BASIS Grilled Kebab | `assiette-kebab-maison` | ~13,00 € | ~4,00 € |
| La Brigade | `burger-maison-frites-four` | ~27,00 € | ~5,00 € |

## À prévoir plus tard — Magasins préférés

### Entité `Magasin`

```yaml
id: monoprix-villejuif
nom: Monoprix Villejuif
type: supermarche        # supermarche | primeur | boucherie | surgeles
                         # epicerie-specialisee | marche
distance_minutes: 8
jours_ouverture: [lun, mar, mer, jeu, ven, sam, dim]
niveau_prix: eleve       # bas | moyen | eleve
ordre_rayons:            # ordre physique de parcours
  - fruits-legumes
  - frais
  - epicerie
  - surgele
```

### Relation ingrédient ↔ magasin

```yaml
disponibilite:
  - magasin: monoprix-villejuif
    dispo: oui
    prix_observe: 2.40
    date_prix: 2026-08-02
  - magasin: lidl-villejuif
    dispo: non
```

### Ce que ça débloque

**Répartition automatique de la liste.** Un écran devient plusieurs listes, une par magasin, avec les articles introuvables ailleurs assignés au bon endroit.

**Tri selon ton parcours physique.** Le champ `ordre_rayons` classe la liste dans l'ordre où tu traverses réellement les allées.

**Arbitrage prix contre distance.** Si un batch coûte 40 % moins cher au magasin à vingt minutes, l'app le signale sans décider à ta place.

**Historique de prix par produit.** Alimenté par tes saisies de tickets — le lien naturel avec la V1 du suivi budget, sans OCR.

**Filtre de faisabilité.** Un dimanche soir où seule l'épicerie de quartier est ouverte, l'app ne propose que des recettes réalisables avec ce qu'on y trouve.

### Question à trancher

La saisie de disponibilité par ingrédient et par magasin est fastidieuse. Deux approches : renseigner à la demande quand un produit s'avère introuvable, ou faire un relevé initial complet. La première est plus réaliste et se remplit d'elle-même à l'usage.
