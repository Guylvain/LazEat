# Projet — Système de planification alimentaire personnel (Obsidian)

> Document de contexte. Destiné à être placé à la racine du coffre Obsidian et fourni comme brief à Cowork / Claude Code pour toute session de développement.

---

## 1. Contexte et raison d'être

### 1.1 Profil utilisateur

Utilisateur unique : Liam, 25 ans, étudiant en Master Data Engineering & IA en alternance, joueur et éducateur de rugby en club.

Le système n'est pas un produit générique. Il est conçu pour **une seule personne**, avec des contraintes horaires précises et des objectifs nutritionnels issus d'un bilan sanguin réel. Toute décision de conception doit être arbitrée par « est-ce que ça sert ce cas précis ? » et non par « est-ce que c'est une bonne fonctionnalité en général ? ».

### 1.2 Le problème à résoudre

L'utilisateur saute des repas — certains jours un seul repas, souvent de la restauration rapide — alors qu'il s'entraîne ou encadre quatre à cinq soirs par semaine. Le déficit énergétique qui en résulte est la cause probable d'une fatigue chronique.

La cause n'est pas un manque de connaissances ni de motivation, mais une **friction de décision et de logistique** : à 22h30 en rentrant d'entraînement, la question « qu'est-ce que je mange » n'a pas de réponse préparée, donc la réponse par défaut est la livraison.

Le système existe pour **déplacer toutes les décisions alimentaires vers le samedi ou le dimanche**, moment où l'utilisateur a du temps et de l'énergie mentale, afin que la semaine ne comporte plus que de l'exécution.

### 1.3 Objectifs nutritionnels dérivés du bilan sanguin (23/07/2026)

Trois cibles concrètes qui doivent influencer le moteur de suggestion :

| Cible | Constat biologique | Levier alimentaire |
|---|---|---|
| **Remonter les folates (B9)** | 3,30 ng/ml pour une norme à 3,89 | Légumineuses, petits pois, chou-fleur, agrumes, mâche |
| **Faire redescendre les triglycérides** | 1,50 g/L, ×4 en deux ans | Réduire sucres rapides et alcool ; oméga-3 (poissons gras, noix, huile de colza) |
| **Couvrir la dépense énergétique** | Volume d'entraînement élevé, repas sautés | Densité glucidique suffisante, régularité des prises |

Ces cibles se traduisent en **tags nutritionnels** portés par les recettes et les ingrédients, utilisés comme bonus de score dans le matching.

### 1.4 Contraintes de goût

À encoder directement dans la base d'ingrédients, sous forme d'un champ `statut_personnel`.

**Exclus** : épinards (à retester ponctuellement, jamais en pilier), brocoli, choux de Bruxelles, avocat, miel.

**Conditionnels** : œufs — acceptés uniquement intégrés à une préparation (omelette garnie, riz sauté, tortilla, shakshuka), jamais seuls ou nature.

**À tester** : sardines et maquereaux — jamais goûtés, a priori réticent. Introduction prévue par préparations à goût atténué (rillettes) avant toute recette où le poisson est central.

**Substitutions actées** : chou-fleur remplace brocoli et choux de Bruxelles ; petits pois, mâche et agrumes compensent l'absence d'épinards ; saumon et noix couvrent les oméga-3 si les sardines sont rejetées.

### 1.5 Contraintes d'emploi du temps

Deux types de semaine alternent, plus une variante du dimanche.

**Semaine de cours** — École 9h10 à 17h10, du lundi au vendredi.

**Semaine d'alternance** — Entreprise lundi, mardi et jeudi de 9h30 à 17h30, avec une heure de trajet dans chaque sens. Télétravail mercredi et vendredi (susceptible de bouger).

**Rugby, identique dans les deux cas :**

- Lundi 20h – 22h : entraînement personnel
- Mardi 19h – 20h30 : encadrement des cadets, puis 20h30 – 22h : entraînement personnel
- Jeudi 19h – 20h30 : encadrement des cadets
- Vendredi 20h – 22h : entraînement personnel
- Dimanche 13h – 19h : match, uniquement certaines semaines

**Conséquence structurelle majeure** : il n'existe aucun dîner à 20h du lundi au vendredi. Le mardi d'alternance est le point critique, avec environ quatorze heures entre le déjeuner et le prochain vrai repas si aucune collation n'est planifiée.

---

## 2. Vision produit

Une application locale, hébergée dans un coffre Obsidian, qui transforme le rituel du dimanche en un planning de semaine complet, une liste de courses agrégée, et un guide d'exécution pas à pas — tout en maintenant un inventaire à jour du frigo et des placards.

**En une phrase** : choisir une fois pour toute la semaine, pour n'avoir plus qu'à suivre.

---

## 3. Parcours utilisateur détaillés

Trois parcours distincts, séparés dans le temps et dans le lieu d'usage.

### 3.1 Parcours A — Planification (samedi ou dimanche, sur ordinateur)

1. L'utilisateur ouvre la vue « Nouvelle semaine ».
2. Il sélectionne le **type de semaine** : cours ou alternance.
3. Il coche ou non l'option **« Match dimanche »**.
4. Le planning se génère : une vue calendrier, une ligne par jour, un **bloc par créneau de repas**, chaque bloc préremplli avec son horaire théorique et son type de créneau.
5. Il peut **modifier les événements** de la semaine avant de continuer : décaler un entraînement, marquer un jour de télétravail déplacé, ajouter un imprévu, supprimer un créneau. Le planning s'ajuste.
6. Il clique sur un bloc de repas. L'interface de **matching** s'ouvre.
7. Le matching propose des recettes une par une, sous forme de cartes : photo, nom, temps de préparation, liste d'ingrédients, tags. L'utilisateur valide ou rejette.
   - **Rejet** → la carte suivante s'affiche.
   - **Validation** → la recette est affectée au bloc, les ingrédients sont ajoutés à la liste de courses, retour au planning.
   - Une option **« Mange dehors »** est toujours disponible et n'ajoute aucun ingrédient.
8. Il répète pour autant de blocs qu'il le souhaite. **La planification partielle est un cas d'usage normal** : planifier seulement les trois premiers jours doit fonctionner sans dégrader le reste du système.

### 3.2 Parcours B — Courses (avant et pendant, sur téléphone)

1. Avant de partir, l'utilisateur ouvre l'onglet **« Courses »** : une checklist agrégée à partir de toutes les recettes validées.
2. Il indique ce qu'il possède déjà. Idéalement, le système a **déjà déduit automatiquement** le contenu connu des placards, et cette étape ne sert qu'à corriger les écarts.
3. En magasin, il coche les articles au fur et à mesure. La liste doit être **groupée par rayon** pour éviter les allers-retours.
4. Au retour, la validation de la liste **incrémente le stock** de « Mes Placards » avec les quantités achetées.

### 3.3 Parcours C — Exécution (au moment du repas, sur téléphone ou tablette)

1. À l'heure du repas, ou au moment du batch cooking du dimanche, l'utilisateur reclique sur le bloc concerné.
2. La **recette se lance en mode cuisine** : étapes détaillées une par une, minuteurs intégrés, quantités ajustées au nombre de portions visé.
3. En fin de recette, il valide via un bouton **« Recette terminée »**.
4. Cette validation déclenche deux effets :
   - **Décrément** des ingrédients consommés dans « Mes Placards ».
   - Si la recette était un batch, **création de portions stockées** (voir section 5.4).
5. Le bloc passe au statut « fait ». Optionnellement, une note de satisfaction est demandée, qui alimente le scoring futur.

---

## 4. Modèle de données

Le modèle repose sur cinq entités principales et trois entités dérivées. C'est la partie du projet à figer en premier : tout le reste en découle.

### 4.1 Ingrédient (référentiel canonique)

L'entité la plus importante et la plus sous-estimée. **Aucune recette ne doit écrire un ingrédient en texte libre** — chaque ligne d'ingrédient pointe vers un identifiant du référentiel. C'est la seule façon d'agréger correctement une liste de courses et de décrémenter un stock.

| Champ | Type | Rôle |
|---|---|---|
| `id` | slug | Clé stable, ex. `pois-chiches-conserve` |
| `nom` | texte | Libellé affiché |
| `categorie` | enum | féculent, légumineuse, protéine, légume, fruit, matière grasse, laitier, épice, placard |
| `rayon` | enum | Tri de la liste de courses : fruits-légumes, frais, surgelé, épicerie, boucherie, poissonnerie |
| `unite_base` | enum | g, ml, pièce |
| `poids_moyen_piece` | nombre | Uniquement si `unite_base = pièce`. Ex. 1 oignon ≈ 150 g. Indispensable aux conversions |
| `conditionnement` | nombre | Format d'achat courant, ex. riz vendu par 1000 g. Sert à arrondir la liste de courses |
| `emplacement` | enum | frigo, congélateur, placard, corbeille à fruits, étagère à épices — alimente les sous-vues de « Mes Placards » |
| `duree_conservation` | jours | Optionnel, pour alerter sur les périssables |
| `tags_nutritionnels` | liste | folates, omega-3, protéines, glucides-lents |
| `statut_personnel` | enum | aimé, neutre, à-tester, exclu |
| `substituts` | liste d'`id` | Alternatives acceptables |

### 4.2 Recette

| Champ | Type | Rôle |
|---|---|---|
| `id` | slug | Clé stable |
| `nom` | texte | |
| `photo` | chemin | Image locale dans le coffre |
| `creneaux_compatibles` | liste | Voir section 4.3 |
| `temps_actif` | minutes | Temps réellement passé aux fourneaux. **C'est ce chiffre qui filtre**, pas le temps total |
| `temps_total` | minutes | Inclut cuisson passive, repos, marinade |
| `portions_base` | entier | |
| `batchable` | booléen | La recette supporte-t-elle d'être multipliée et conservée ? |
| `portions_max_batch` | entier | |
| `bon_froid` | booléen | Critère décisif pour les gamelles du midi |
| `transportable` | booléen | Se mange sans couverts, dans un train — critère du mardi |
| `ingredients` | liste | `{ingredient_id, quantite, unite, optionnel, substituts}` |
| `etapes` | liste | `{titre, texte, minuteur_secondes}` |
| `ustensiles` | liste | Pour signaler ce qui demande un four, un mixeur, etc. |
| `tags_nutritionnels` | liste | Hérités ou déclarés |
| `statut` | enum | à-tester, validée, rejetée |
| `note` | 1 à 4 | Renseignée après exécution. Échelle paire, voir `spec-bilan-et-portions-restantes.md` §2.3 — corrigé le 31/07/2026, ce document disait 1 à 5 |
| `derniere_execution` | date | Alimente la rotation |
| `nb_executions` | entier | |

### 4.3 Créneau (type de repas)

Constantes du système. Chaque créneau porte ses propres contraintes, qui deviennent des filtres durs dans le matching.

| Créneau | Budget temps actif | Contraintes | Exemples |
|---|---|---|---|
| `petit-dej` | 5 min | Assemblage uniquement | Yaourt, fruit, noix |
| `dejeuner` | 0 min sur place | `bon_froid` ou réchauffable ; issu d'un batch ou d'un reste | Gamelle |
| `collation-trajet` | 5 min la veille | `transportable`, sans couverts | Tartines, banane |
| `recharge-express` | 0 min | Non périssable, vit dans le sac de sport | Abricots secs, compote |
| `post-entrainement-rapide` | 12 min | Réchauffage ou poêlée express | Poêlée post-rugby |
| `soir-cuisine` | 30 min | Vraie cuisine, quantités doublées | Sardines-patates |
| `batch` | 60 min | `batchable = true`, 3 à 4 portions | Curry de pois chiches |
| `avant-match` | 20 min | Digestible, pauvre en fibres et en gras | Pâtes, riz |
| `libre` | — | Option « Mange dehors » | — |

### 4.4 Semaine planifiée

| Champ | Type |
|---|---|
| `date_debut` | date (lundi) |
| `type` | cours \| alternance |
| `match_dimanche` | booléen |
| `evenements` | liste d'événements, préremplis par le type de semaine puis modifiables |
| `slots` | liste de créneaux planifiés |

Chaque **slot** contient : jour, type de créneau, heure, source (`recette_id`, `portion_stockee_id`, ou `dehors`), et statut (`vide`, `planifié`, `cuisiné`, `mangé`, `sauté`).

### 4.5 Placard (état du stock)

Une ligne par ingrédient présent : `{ingredient_id, quantite, unite, emplacement, date_achat, dlc}`.

Les vues « Mon Frigo », « Mon Congélateur », « Mes Épices », « Mes Fruits », « Mon Placard » ne sont **pas des entités séparées** : ce sont des filtres sur le champ `emplacement`. Cela évite de dupliquer la donnée et de la désynchroniser.

### 4.6 Entités dérivées

**Liste de courses** — Générée, jamais saisie à la main. Voir la règle d'agrégation en 5.2.

**Portion stockée** — Voir section 5.4.

**Historique de matching** — Chaque validation ou rejet est journalisé : `{recette_id, creneau, decision, date}`. C'est le carburant du scoring à moyen terme.

---

## 5. Règles métier

### 5.1 Moteur de matching

Le matching se fait en deux temps : un filtre dur, puis un score.

**Filtres durs — une recette est éliminée si :**

- Le créneau demandé n'est pas dans ses `creneaux_compatibles`
- Son `temps_actif` dépasse le budget du créneau
- Elle contient un ingrédient au `statut_personnel = exclu` sans substitut déclaré
- Son statut est `rejetée`

**Score sur les recettes restantes :**

| Critère | Effet |
|---|---|
| Note personnelle élevée | Bonus fort |
| Cuisinée il y a moins de 14 jours | Malus fort — la rotation prime sur la préférence |
| Ingrédients déjà présents en stock | Bonus proportionnel |
| Tag nutritionnel prioritaire (folates, oméga-3) | Bonus modéré |
| Statut `à-tester` | Bonus léger, pour forcer la découverte |
| Rejetée récemment pour ce créneau | Malus |

L'interface présente les cartes par score décroissant, avec quatre ou cinq propositions avant de reboucler.

### 5.2 Agrégation de la liste de courses

1. Rassembler les lignes d'ingrédients de toutes les recettes planifiées et non encore cuisinées.
2. Convertir chaque quantité vers l'`unite_base` de l'ingrédient — d'où l'importance du champ `poids_moyen_piece`.
3. Sommer par `ingredient_id`.
4. Soustraire le stock connu dans « Mes Placards ».
5. Arrondir au `conditionnement` supérieur. Un besoin de 1,5 boîte de pois chiches devient 2 boîtes.
6. Grouper par `rayon`.

L'étape 5 génère mécaniquement du surplus, qui retourne au stock. C'est normal et voulu — c'est ainsi que le placard se constitue.

### 5.3 Décrément du stock

Le décrément se déclenche à la **validation de la recette**, pas à l'achat ni à la planification. Une recette planifiée mais jamais cuisinée ne doit rien consommer.

Cas particuliers à traiter explicitement :

- Ingrédient consommé alors que le stock est à zéro → autoriser le passage en négatif ou marquer un écart, mais **ne jamais bloquer l'utilisateur en cuisine**. L'ergonomie prime sur l'intégrité des données à cet instant.
- Épices et huiles → candidats à être exclus du décrément et gérés en simple « présent / bientôt fini », car les quantités réelles sont impossibles à suivre.

### 5.4 Portions stockées — le mécanisme central

**Ce point est le plus important du système et le plus facile à manquer.**

Le batch cooking crée un décalage entre le moment où l'on cuisine et le moment où l'on mange. Un curry préparé le mercredi soir alimente le déjeuner du jeudi, le repas du dimanche soir et une gamelle du lundi suivant. Si le système ne modélise que « recette → repas », il proposera de racheter les ingrédients et de recuisiner à chaque fois.

Il faut donc une entité intermédiaire :

```
Portion stockée
  ├─ recette_id       (d'où elle vient)
  ├─ date_cuisson
  ├─ portions_restantes
  ├─ dlc_estimee
  └─ emplacement      (frigo | congélateur)
```

**Règles associées :**

- Valider une recette `batchable` crée N portions stockées et décrémente les ingrédients **une seule fois**.
- Un slot peut être rempli soit par une recette à cuisiner, soit par une **portion stockée existante**. Le matching doit proposer les portions disponibles **en priorité absolue**, avant toute nouvelle recette.
- Consommer une portion la décrémente sans toucher au stock d'ingrédients.
- Une portion qui approche sa DLC doit remonter en tête des suggestions.

Cette entité est ce qui distingue un vrai outil de batch cooking d'un simple carnet de recettes.

### 5.5 Gestion des écarts d'inventaire

Un inventaire de placard **dérive toujours** : consommation hors recette, quantités approximatives, aliments jetés, courses d'appoint non saisies. C'est la raison principale d'abandon de ce type d'outil.

Deux garde-fous à prévoir dès la conception :

1. Un **rituel de recalage hebdomadaire**, intégré au parcours du dimanche : une vue rapide où l'on corrige les quantités manifestement fausses, en trente secondes.
2. Une **tolérance assumée** : la liste de courses propose, elle n'impose pas. Un écart d'inventaire doit produire au pire un achat en double, jamais un blocage.

---

## 6. Architecture technique

### 6.1 Contrainte de fond

Obsidian n'offre nativement ni interface de swipe, ni écriture transactionnelle. Le projet impose donc un choix entre trois voies.

### 6.2 Option A — Tout en Markdown natif

Recettes en notes avec propriétés YAML, planning en note hebdomadaire, requêtes via **Bases** (natif) ou **Dataview**, actions via **Templater** et **Buttons**.

*Avantages* : aucun code, portable, robuste, fonctionne sur mobile immédiatement.
*Limites* : pas de swipe, uniquement des listes cliquables. Le décrément automatique est difficile à fiabiliser.

### 6.3 Option B — Plugin Obsidian sur mesure (recommandée)

Données en Markdown avec frontmatter, interface développée en TypeScript comme plugin Obsidian, avec des vues personnalisées pour le matching, le planning et le mode cuisine.

*Avantages* : contrôle total de l'ergonomie, vrai swipe, mode cuisine plein écran, logique métier centralisée et testable.
*Limites* : demande un vrai développement. Cela dit, le profil de l'utilisateur (data engineering, TypeScript accessible) rend cette voie réaliste.

**C'est l'option recommandée**, à condition de respecter une règle : **les données restent en Markdown lisible**. Le plugin est une couche d'interface, jamais un format propriétaire. Si le plugin est abandonné un jour, le coffre reste exploitable.

### 6.4 Option C — Application web externe

Une application React lisant et écrivant les fichiers du coffre.

*Limites* : complexité de synchronisation, rupture avec l'usage mobile d'Obsidian, deux sources de vérité. **Déconseillée** sauf besoin d'un déploiement multi-appareils sortant du cadre Obsidian.

### 6.5 Question de l'état transactionnel

Le stock des placards et les portions stockées sont des données **à écriture fréquente et à faible valeur documentaire**. Les stocker dans du frontmatter Markdown est fragile.

Recommandation : un **fichier JSON unique** (`data/stock.json`, `data/portions.json`) géré par le plugin, tandis que recettes et ingrédients restent en Markdown. On conserve la lisibilité là où elle sert, et la robustesse là où elle compte.

---

## 7. Structure de coffre proposée

```
Coffre-Nutrition/
├── 00-Contexte/
│   ├── contexte-projet.md          ← ce document
│   ├── profil-nutritionnel.md      ← objectifs issus du bilan sanguin
│   └── contraintes-horaires.md     ← les deux types de semaine
├── 01-Recettes/
│   ├── _template-recette.md
│   ├── curry-pois-chiches.md
│   ├── poelee-post-rugby.md
│   └── ...
├── 02-Ingredients/
│   ├── _template-ingredient.md
│   └── ...                         ← ou un fichier unique si le référentiel reste modeste
├── 03-Semaines/
│   ├── _template-semaine.md
│   ├── 2026-W32.md
│   └── ...
├── 04-Suivi/
│   ├── courses.md                  ← liste active
│   ├── placards.md                 ← vue générée
│   └── historique.md
├── data/
│   ├── stock.json
│   └── portions.json
├── assets/
│   └── photos-recettes/
└── .obsidian/plugins/meal-planner/ ← si option B
```

---

## 8. Points de conception non résolus

À trancher avant ou pendant le développement.

1. **Conversion d'unités.** Le champ `poids_moyen_piece` résout le cas des ingrédients comptables, mais pas les mesures approximatives type « une poignée » ou « un filet d'huile ». Faut-il les interdire dans les recettes, ou les autoriser avec une quantité nulle et les exclure du décrément ?

2. **Granularité du suivi de stock.** Suivre les épices au gramme est irréaliste. Proposition : un champ `suivi_stock` sur l'ingrédient, à trois valeurs — `precis` (viande, légumes, féculents), `binaire` (épices, huiles : présent ou à racheter), `ignore`.

3. **Photos des recettes.** Deux stratégies : photographier ses propres plats au fil de l'eau, ou utiliser des images libres de droits. La première est plus lente mais plus motivante et sans risque juridique. Prévoir un placeholder propre en attendant.

4. **Source de vérité du planning.** Si l'utilisateur modifie un événement de la semaine, le type de semaine devient-il caduc, ou s'agit-il d'une surcharge conservée à part ?

5. **Recettes pour une personne.** Cuisiner pour un seul convive génère des restes d'ingrédients frais. Le système doit-il activement suggérer des recettes qui réutilisent un ingrédient entamé ?

---

## 9. Questions ouvertes à l'utilisateur

À renseigner avant la phase 1.

1. Qu'est-ce que **Dispatch** dans ton contexte ? L'outil n'est pas identifié, et son rôle dans l'architecture reste à préciser.
2. Quelle version d'Obsidian, et quels plugins sont déjà installés ? Dataview, Bases, Templater sont-ils en place ?
3. Es-tu prêt à développer un plugin en TypeScript, ou faut-il rester sur du no-code au départ ?
4. Le swipe se fait sur téléphone ou sur ordinateur ? Cela change le design de l'interaction.
5. Combien de recettes veux-tu en base au démarrage ? En dessous de vingt-cinq, le matching n'a pas grand-chose à proposer.
6. Le suivi du budget de courses doit-il être intégré ?
7. Congélateur disponible, et de quelle taille ? Cela conditionne l'ampleur du batch cooking.
8. Cuisines-tu uniquement pour toi ?
9. Veux-tu suivre un indicateur de résultat — sensation d'énergie aux entraînements, poids, régularité des repas — pour vérifier que le système fonctionne ?

---

## 10. Roadmap par phases

L'ordre est important : chaque phase doit être utilisable seule.

**Phase 0 — Fondations.** Figer le format de recette et le référentiel d'ingrédients. Saisir vingt à vingt-cinq recettes couvrant tous les créneaux. Aucune interface. *C'est la phase la plus ingrate et la plus déterminante.*

**Phase 1 — Planning et courses manuels.** Template de semaine, sélection manuelle des recettes, liste de courses agrégée par requête. Pas de swipe, pas de stock. **Cette phase délivre déjà l'essentiel de la valeur** — elle résout le problème réel, qui est de décider à l'avance.

**Phase 2 — Interface de matching.** Le swipe, les cartes, l'affectation automatique aux blocs.

**Phase 3 — Placards.** Stock, décrément, vues par emplacement, rituel de recalage.

**Phase 4 — Portions stockées.** Le mécanisme de batch complet.

**Phase 5 — Scoring.** Historique, rotation, suggestions selon le stock disponible.

**Recommandation** : vivre au moins trois semaines complètes en phase 1 avant d'écrire la moindre ligne de plugin. Cette période révélera quelles recettes tiennent réellement le choc du mardi à 22h30 — information impossible à obtenir autrement, et qui conditionne toute la suite.
