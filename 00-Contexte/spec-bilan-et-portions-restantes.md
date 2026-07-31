# Spécification — Bilan de recette et portions restantes

Complément au document de contexte. Deux fonctionnalités qui partagent un seul écran, déclenché à la validation d'une recette.

---

## 1. Le principe : un seul écran, quinze secondes

Après le bouton **« Recette terminée »**, un écran unique apparaît. Il pose trois questions maximum, toutes répondables au doigt, sans clavier.

```
┌──────────────────────────────────┐
│  Chili con carne — terminé       │
│                                  │
│  C'était comment ?               │
│   ☹️      😐      🙂      🤩      │
│                                  │
│  Il t'en reste ?                 │
│   [ 0 ]  [ 1 ]  [ 2 ]  [ 3 ]     │
│                                  │
│  📷 Ajouter ta photo             │
│                                  │
│         [ Valider ]              │
└──────────────────────────────────┘
```

**Règle de conception non négociable** : si cet écran dépasse quinze secondes ou demande du texte libre obligatoire, il sera abandonné en trois semaines et les deux fonctionnalités mourront avec lui. Tout doit être optionnel sauf le verdict.

---

## 2. Suivi des goûts

### 2.1 Pourquoi « pas aimé » ne suffit pas

Un verdict négatif seul est une donnée inexploitable. Deux rejets identiques peuvent appeler des actions opposées :

- « Trop long à faire » → la recette est bonne, elle est juste mal placée dans le planning. Il faut la déplacer du mardi au jeudi, pas la supprimer.
- « Goût pas terrible » → la recette ou un de ses ingrédients ne passe pas. Là il faut la sortir.

D'où une **deuxième question conditionnelle**, affichée uniquement si le verdict est ☹️ ou 😐 :

```
Qu'est-ce qui n'allait pas ?
[ Le goût ]  [ La texture ]
[ Trop long ]  [ Pas assez copieux ]
[ Trop de vaisselle ]  [ J'ai raté ]
```

Un tap, pas de saisie. Un champ de commentaire libre reste disponible mais facultatif.

### 2.2 Effets sur le modèle

À chaque validation :

```yaml
note: 2                          # 1 à 4
nb_executions: +1
derniere_execution: 2026-08-05
```

Et une entrée dans l'historique :

```yaml
- recette_id: chili-con-carne
  date: 2026-08-05
  verdict: 2
  raison: gout
  commentaire: null
  creneau: soir-cuisine
```

### 2.3 Règle de transition de statut

**Un mauvais avis ne tue pas une recette.** Tu peux être fatigué, avoir raté une étape, ou avoir manqué d'un ingrédient. Supprimer sur un seul essai fera fondre la base pour de mauvaises raisons.

| Condition | Nouveau statut |
|---|---|
| Note ≥ 3 une fois | `validee` |
| Note ≤ 2 une fois | reste `a-tester`, malus de score temporaire |
| Note ≤ 2 **deux fois consécutives** | `rejetee` |
| Raison = « trop long » uniquement | statut inchangé, **retrait du créneau concerné** |

Ce dernier cas est important : une recette jugée trop longue le mardi soir n'est pas mauvaise, elle est mal placée. Le système retire `post-entrainement-rapide` de ses `creneaux_compatibles` et la garde pour le jeudi.

### 2.4 Remontée au niveau de l'ingrédient

C'est ici que le suivi devient réellement utile. Le système observe les rejets pour raison de **goût** et cherche les ingrédients communs.

> Trois recettes rejetées pour le goût contiennent toutes de la feta.
> → Proposition : passer `feta` en `statut_personnel: exclu` ?

Le système **propose**, il ne décide pas. Mais c'est ce mécanisme qui transforme une simple notation de recettes en véritable cartographie de tes goûts — et qui aurait détecté tout seul le problème des épinards.

Seuil suggéré : trois occurrences minimum, et uniquement pour la raison « goût ».

---

## 3. Portions restantes

### 3.1 Le cas que le modèle ne couvrait pas

Le modèle initial ne créait des portions stockées que pour les recettes **planifiées comme batch**. Or le cas le plus fréquent est le surplus non prévu : tu cuisines pour deux, tu manges une part, il en reste une.

Sans capture de ce surplus, il devient invisible, il est oublié au fond du frigo, et il finit à la poubelle. C'est aussi une des principales sources de dérive d'inventaire.

### 3.2 Entité créée

Une réponse ≥ 1 à « Il t'en reste ? » crée automatiquement :

```yaml
recette_id: chili-con-carne   # ← la clé : une seule entrée par recette
origine: restes               # batch | restes
type_contenu: repas           # repas | composant
date_production: 2026-08-05
portions_restantes: 1
dlc_estimee: 2026-08-09
emplacement: frigo            # frigo | congelateur
```

**Un seul lot par recette, pas un identifiant par lot.** Décision du
31/07/2026, en remplacement du `id: ps-AAAAMMJJ-NNN` que décrivait cette
spec. Le stockage est indexé par `recette_id`, sur le modèle de `Bilans`.

Conséquence : deux lots de la même recette avec des DLC différentes ne sont
pas représentables. Assumé — c'est un cas de bord, et la migration vers une
liste de lots reste possible sans rien casser en aval si l'usage réel le
réclame.

**Mais la fusion doit être additive, jamais un remplacement.** Répondre une
seconde fois à « Il t'en reste ? » pour une recette dont il reste déjà des
portions doit **additionner** les portions et **retenir la DLC la plus
proche**. Écraser l'entrée précédente ferait disparaître des portions
réellement présentes au frigo — l'inverse exact de ce que cette
fonctionnalité existe pour éviter.

`contenant_id` reste hors périmètre tant que la notion de boîtes n'existe
pas (§3.5).

### 3.3 La DLC est calculée, jamais saisie

`dlc_estimee = date_production + duree_conservation_frigo` — champ déjà présent sur chaque recette du socle.

Si tu choisis « congélateur », la durée passe à 90 jours. Tu ne tapes jamais de date.

Ce choix évite le piège classique : demander une DLC à l'utilisateur garantit qu'il ne la renseignera pas.

### 3.4 Interaction avec `qualite_j2`

Le champ `qualite_j2` gouverne le comportement de cette question.

| `qualite_j2` | Comportement |
|---|---|
| `excellente` | Question posée normalement. DLC = 3 à 4 jours. Congélation proposée |
| `correcte` | Question posée. DLC = 2 jours |
| `a-eviter` | Question posée **avec avertissement** : « Ce plat se conserve mal. » DLC forcée à 1 jour, congélation masquée |

Concrètement, si tu déclares un reste de gnocchis crème lardons, le système accepte mais te signale qu'il faut les manger le lendemain midi, pas plus tard.

### 3.5 La contrainte des boîtes devient calculable

Tu as douze boîtes. Chaque portion stockée en occupe une. Le système sait donc combien sont libres.

- Avant de lancer un batch de 4 portions, il vérifie qu'il reste 4 contenants.
- S'il n'en reste que 2, il propose de réduire le batch ou signale qu'il faut d'abord vider le frigo.
- À la consommation d'une portion, le contenant est libéré.

C'est la traduction en données d'une contrainte physique réelle, celle qui limite vraiment ton batch cooking.

### 3.6 Pas de double décrément

**Point de vigilance technique.** Les ingrédients sont décrémentés à la validation de la recette. La portion stockée est une **transformation de stock déjà consommé**, pas un ajout.

Créer une portion stockée ne doit donc jamais toucher au stock d'ingrédients. Consommer cette portion non plus. C'est l'erreur qui ferait diverger l'inventaire en quelques semaines.

### 3.7 Priorité absolue dans le matching

Quand tu ouvres un créneau de repas, l'ordre de proposition est :

1. **Portions stockées compatibles**, DLC la plus proche en premier
2. Recettes à cuisiner, classées par score

Une portion dont la DLC arrive dans deux jours ou moins est proposée en **suggestion par défaut**, avant même l'écran de matching. Pas de swipe à faire : l'app affiche directement « Il te reste 1 portion de chili, à manger avant vendredi ».

Une portion de composant — un pâton de pizza congelé, du riz cuit — n'est jamais proposée sur un créneau `post-entrainement-rapide` de douze minutes, puisqu'elle demande encore du travail.

### 3.8 Affichage dans « Mes Placards »

Nouvelle section, distincte des ingrédients bruts :

```
Mes Placards
├── Mes plats prêts      ← portions stockées
├── Mon Frigo
├── Mon Congélateur
├── Mes Fruits
├── Mes Épices
└── Mon Placard
```

Chaque plat prêt affiche : nom, nombre de portions, jours restants avant DLC, emplacement.

**L'urgence se dit, elle ne se colore pas.** Décision du 31/07/2026, en remplacement du code couleur vert / orange / rouge que décrivait cette spec. `DESIGN.md` ne définit qu'un seul accent, `amber`, et interdit explicitement d'inventer une valeur — trois couleurs d'urgence en créeraient deux de toutes pièces et casseraient un système entier bâti sur un accent unique.

L'urgence passe donc par la formulation : « à manger aujourd'hui », « à manger demain », « à manger avant le [date] ». Un seul signal binaire, `dlcUrgente`, gouverne l'emploi du fond ambré déjà prévu pour ce type de carte.

C'est aussi plus lisible : « à manger demain » se comprend sans apprendre une convention, ce qui n'est pas le cas d'un point orange.

---

## 4. Bénéfice dérivé : suivi du gaspillage

Une portion stockée dont la DLC est dépassée sans consommation déclenche une question simple à la prochaine ouverture : **mangée ou jetée ?**

Cela produit gratuitement une métrique utile :

- Nombre de portions jetées par mois
- Valeur estimée du gaspillage, croisée avec l'historique de prix des tickets
- Détection d'un sur-batch systématique : si tu jettes régulièrement, tu cuisines trop grand, et le système peut proposer de réduire `portions_base` sur les recettes concernées

C'est aussi un signal de qualité indirect. Une recette bien notée mais dont les restes finissent toujours à la poubelle est une recette que tu apprécies sur le moment mais que tu ne veux pas remanger. Information impossible à obtenir par la notation seule.

---

## 5. Impact sur le planning

Ces deux fonctionnalités créent une boucle de rétroaction qui n'existait pas :

```
Planifier → Cuisiner → Bilan → Portions stockées
     ↑                    ↓            ↓
     └──── Scoring ←── Goûts ──────────┘
```

Concrètement, après quatre à six semaines d'usage :

- Le matching propose en priorité ce que tu aimes réellement, pas ce que je supposais que tu aimerais
- Le planning se remplit en partie tout seul, les créneaux étant préaffectés aux portions disponibles
- La liste de courses rétrécit, parce qu'elle tient compte de ce qui existe déjà sous forme de plats prêts

C'est le moment où le système cesse d'être un carnet de recettes pour devenir un vrai outil.

---

## 6. Phasage

| Phase | Contenu |
|---|---|
| 2 | Écran de bilan, verdict seul. Alimente `note` et `nb_executions` |
| 3 | Raisons de rejet structurées, transitions de statut, photo perso |
| 4 | Portions restantes, DLC calculée, contrainte des contenants |
| 5 | Remontée au niveau ingrédient, suivi du gaspillage |

L'écran doit exister **dès la phase 2**, même réduit au seul verdict. Sans données de goût accumulées dès le départ, le scoring de la phase 5 n'aura rien à exploiter.
