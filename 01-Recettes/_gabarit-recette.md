---
# ─── IDENTITÉ ───────────────────────────────────────────
id: slug-unique-en-kebab-case
nom: Nom affiché de la recette
photo_placeholder: assets/photos-recettes/slug.jpg
photo_perso: null            # rempli automatiquement après la 1re exécution validée

# ─── NATURE DE LA PRODUCTION ────────────────────────────
type_production: repas       # repas | composant
# "composant" = la recette produit une base réutilisable (pâte, sauce, riz cuit)
# qui ne se mange pas seule et alimente d'autres recettes.

# ─── FAMILLE ────────────────────────────────────────────
famille: null                # regroupe les VARIANTES entre elles
# Relation horizontale, pas hiérarchique. Sert à la rotation :
# le moteur ne propose pas deux recettes de la même famille
# à deux jours d'intervalle.
# Familles existantes : pizza, creme-lardons, riz-saute,
# salade-fraiche, sandwich, mijote, petit-dej, street-food

# ─── CRÉNEAUX ───────────────────────────────────────────
creneaux_compatibles:
  - petit-dej                # 5 min, assemblage
  - dejeuner                 # gamelle, doit être bon froid ou réchauffable
  - collation-trajet         # transportable, sans couverts
  - recharge-express         # zéro prépa, vit dans le sac de sport
  - post-entrainement-rapide # 12 min actif max
  - soir-cuisine             # 30 min actif
  - batch                    # 60 min, 3-4 portions
  - avant-match              # digestible, pauvre en fibres et en gras
  - diner-a-deux             # se cuisine à deux, postes parallèles

# ─── TEMPS ──────────────────────────────────────────────
temps_actif: 0               # minutes réellement passées à travailler — SERT DE FILTRE
temps_total: 0               # inclut pousses, cuissons passives, marinades

# ─── PORTIONS ───────────────────────────────────────────
portions_base: 2
portions_max_reel: 4         # limite matérielle (taille de poêle, de plat)
note_scaling: null           # ex. "épices ×1,5 seulement au-delà de 4 portions"

# ─── CONSERVATION ───────────────────────────────────────
batchable: false
portions_max_batch: null
qualite_j2: correcte         # excellente | correcte | a-eviter
congelable: false
duree_conservation_frigo: null   # en jours

# ─── CONTRAINTES ────────────────────────────────────────
bon_froid: false             # se mange froid sans être dégradé
transportable: false         # se mange debout, sans couverts
ustensiles_requis:
  - four
  - plaque

# ─── REGISTRE ET CUISINE À DEUX ─────────────────────────
registre: quotidien          # quotidien | convivial | soigne
# "quotidien"  : repas de semaine, efficace, sans cérémonie
# "convivial"  : repas partagé, décontracté, se mange à plusieurs
# "soigne"     : vrai dîner, on prend le temps, on dresse

cuisine_a_deux:
  adapte: false              # la recette se parallélise-t-elle réellement ?
  niveau: non                # excellent | bon | limite | non
  postes:                    # les deux flux de travail, pour l'affichage scindé
    - nom: Nom du poste 1
      etapes: [1, 2]
    - nom: Nom du poste 2
      etapes: [3, 4, 5]
  moment_partage: "Le moment où les deux postes se rejoignent."
# Critère : une recette est compatible si elle se découpe en deux postes
# qui avancent EN MÊME TEMPS sans se gêner. Une poêle unique et dix minutes
# d'étapes séquentielles = niveau "non", même si le plat est excellent.

# ─── NUTRITION ──────────────────────────────────────────
tags_nutritionnels:
  - folates
  - omega-3
  - proteines
  - glucides-lents

# ─── SUIVI ──────────────────────────────────────────────
statut: a-tester             # a-tester | validee | rejetee
note: null                   # 1 à 5, saisie après exécution
derniere_execution: null
nb_executions: 0
source: null                 # origine de la recette, si applicable
---

# Nom de la recette

Une à trois phrases : ce que c'est, pourquoi elle est dans la base, à quel moment de la semaine elle sert.

## Ingrédients

Chaque ligne pointe vers un `id` du référentiel. **Jamais de texte libre** — c'est ce qui permet l'agrégation de la liste de courses et le décrément du stock.

| ingredient_id | quantité | unité | optionnel | substituts |
|---|---|---|---|---|
| `farine-t65` | 600 | g | non | `farine-t55` |

## Étapes

Une étape par bloc. Le `minuteur` est en secondes, et doit être présent dès qu'il y a une attente, même passive.

### 1. Titre court de l'étape
`minuteur: 300`

Texte de l'instruction. Les quantités s'écrivent en référence `{ingredient_id}` pour être recalculées automatiquement selon le nombre de portions.

## Notes

Variantes, erreurs classiques, conseils de conservation, ce qui change si on double les quantités.

---

# Dépendances entre recettes

Il n'existe **aucune relation parent-enfant entre recettes**. Une seule relation compte : la dépendance à un composant, et elle s'exprime **entièrement par la ligne d'ingrédient**.

```
pate-a-pizza-maison  ──produit──▶  paton-pizza  ──consommé par──▶  garniture-pizza-*
riz-cuit-en-quantite ──produit──▶  riz-cuit     ──consommé par──▶  poelee-post-rugby
                                                                    riz-saute-poulet-legumes
                                                                    bol-asiatique-poulet-croustillant
```

Les garnitures de pizza ne sont pas des sous-recettes de la pâte. Ce sont des recettes qui consomment un pâton, exactement comme la poêlée post-rugby consomme du riz cuit. Aucun champ supplémentaire n'est nécessaire : la ligne `paton-pizza` dans le tableau d'ingrédients dit déjà tout.

Le champ `famille` capture l'autre relation, celle que `recette_parente` tentait maladroitement d'exprimer : les trois garnitures sont des **variantes entre elles**, une relation de fratrie et non de filiation.

---

# Référentiel d'ingrédients — schéma

Chaque ingrédient est une note dans `02-Ingredients/`, ou une ligne d'un fichier unique tant que le référentiel reste sous cent entrées.

```yaml
id: farine-t65
nom: Farine T65
categorie: placard          # féculent | légumineuse | protéine | légume | fruit
                            # matière-grasse | laitier | épice | placard
rayon: epicerie             # fruits-legumes | frais | surgele | epicerie
                            # boucherie | poissonnerie
unite_base: g               # g | ml | piece
poids_moyen_piece: null     # obligatoire si unite_base = piece (ex. oignon → 150)
conditionnement: 1000       # format d'achat courant, sert à arrondir la liste
emplacement: placard        # frigo | congelateur | placard | corbeille-fruits
                            # etagere-epices
suivi_stock: precis         # precis | binaire | ignore
                            # "binaire" pour épices et huiles : présent ou à racheter
duree_conservation: null
tags_nutritionnels: []
statut_personnel: neutre    # aime | neutre | a-tester | exclu
substituts:
  - farine-t55
```

## Règle absolue — ingrédients produits

Un ingrédient `provenance: produit` ne doit **jamais** apparaître sur la liste de courses. On n'achète pas un pâton de pizza ni du riz cuit.

S'il en manque au moment de planifier :

1. L'app ne l'ajoute pas au caddie.
2. Elle propose d'**insérer une session de la recette productrice** en amont dans le planning.

Exemple concret : tu sélectionnes « pizza pepperoni » pour mercredi soir, aucun pâton n'est en stock. L'app ne met pas « 1 pâton » sur ta liste Monoprix — elle te propose d'ajouter `pate-a-pizza-maison` au créneau du mercredi après-midi, et n'ajoute aux courses que la farine, la levure et le sel.

C'est le chaînage de production. Sans cette règle, la liste de courses devient absurde dès la première semaine.

Ingrédients produits actuellement au référentiel : `paton-pizza` (par `pate-a-pizza-maison`), `riz-cuit` (par `riz-cuit-en-quantite`).

## Statuts personnels déjà connus

**exclu** — `epinards` (à retester ponctuellement, jamais en pilier), `brocoli`, `choux-bruxelles`, `avocat`, `miel`

**a-tester** — `sardines-conserve`, `maquereaux-conserve`

**conditionnel** — `oeufs` : acceptés uniquement intégrés à une préparation, jamais seuls ou nature. À traiter comme un `aime` avec une contrainte au niveau des recettes, pas de l'ingrédient.
