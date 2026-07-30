# Référentiel d'ingrédients — mode d'emploi

103 ingrédients extraits automatiquement des 31 recettes, plus 49 substituts laissés minimaux.

Le fichier `referentiel-ingredients.yaml` est **pré-rempli avec mes estimations**. Ton travail est de relire et corriger, pas de saisir.

## Le seul champ vraiment à vérifier

**`conditionnement`** — c'est le format dans lequel *tu* achètes. Je ne peux pas le deviner : riz par 500 g ou par 1 kg, œufs par 6 ou par 12, pommes de terre au sac de 2,5 kg ou à l'unité.

Ce champ sert à arrondir la liste de courses. Un besoin de 1,5 boîte de pois chiches devient 2 boîtes. S'il est faux, ta liste est fausse.

Les autres champs sont fiables ou sans conséquence si approximatifs.

## Comment procéder — 30 minutes réparties

Ne fais pas les 103 d'un coup. Procède par rayon, dans l'ordre où tu fais tes courses :

1. **Épicerie** — 53 entrées, le gros du travail
2. **Fruits et légumes** — 24 entrées
3. **Frais** — 19 entrées
4. **Boucherie** — 5 entrées
5. **Surgelé** — 2 entrées

Le plus simple : fais tes courses une fois avec le fichier ouvert sur le téléphone, et note les formats réels au fur et à mesure. Tu auras un référentiel exact au lieu d'estimations.

## Points à trancher

**`suivi_stock`** — trois valeurs. `precis` pour ce qui se pèse (viande, féculents, légumes). `binaire` pour ce qui est impossible à suivre au gramme (épices, huiles, sauces) : présent ou à racheter. `ignore` pour l'eau.

J'ai mis 68 en `precis` et 34 en `binaire`. Si le suivi te paraît trop lourd à l'usage, bascule davantage d'entrées en `binaire`. Mieux vaut un inventaire partiel et juste qu'un inventaire complet et faux.

**`statut_personnel`** — j'ai reporté ce que je savais : `miel` en `exclu`, `sardines-conserve`, `chou-chinois` et `pates-gyoza` en `a-tester`. Le reste est en `aime` ou `neutre` selon nos échanges. Corrige librement.

## Deux anomalies détectées

**`miel-ou-sirop-erable`** — cet identifiant présent dans `bol-asiatique-poulet-croustillant` est invalide : il désigne deux ingrédients. À remplacer par `sirop-erable`, qui a déjà `miel` en substitut.

**`chili-con-carne` cité comme substitut** dans `tacos-maison-a-composer` — ce n'est pas un ingrédient mais une recette. C'est en réalité le mécanisme des portions stockées : le chili disponible au frigo remplace la viande à cuisiner. À traiter côté matching, pas dans le référentiel. Ne pas créer d'entrée pour lui.

## Les deux ingrédients produits

`paton-pizza` et `riz-cuit` portent `provenance: produit`. Rappel de la règle : **ils ne doivent jamais apparaître sur une liste de courses.** S'il en manque, l'app propose d'insérer la recette productrice au planning.

## Vérifier après modification

```bash
python -c "import yaml; d=yaml.safe_load(open('referentiel-ingredients.yaml',encoding='utf-8')); print(len(d['ingredients']),'entrées valides')"
```

Si ça plante, c'est presque toujours une indentation : deux espaces avant chaque champ, un tiret devant `id`.
