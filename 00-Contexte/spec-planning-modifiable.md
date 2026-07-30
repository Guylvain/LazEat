# Spécification — Planning modifiable

Résout la question ouverte n°4 du document de contexte : que devient le type de semaine quand on modifie un événement ?

---

## 1. Le principe : modèle plus surcharges

**Une semaine n'est jamais une copie du modèle. C'est une référence au modèle, plus une liste de différences.**

```yaml
# 03-Semaines/2026-W35.md
date_debut: 2026-08-24
modele: cours              # cours | alternance | (modèles personnalisés)
match_dimanche: false

surcharges:
  - jour: mercredi
    action: changer_mode
    valeur: distanciel

  - jour: mercredi
    action: ajouter
    evenement: salle-de-sport
    debut: "17:30"
    duree: 90

  - jour: vendredi
    action: supprimer
    evenement: rugby-entrainement
    motif: "Repos avant match"
```

### Pourquoi pas une copie

Si modifier une semaine dupliquait tout le modèle, alors :

- Améliorer le modèle plus tard ne profiterait qu'aux semaines futures
- Impossible de savoir ce qui a été modifié et ce qui est standard
- Aucun retour en arrière possible

Avec des surcharges, le modèle reste la source de vérité et chaque semaine ne stocke que son écart. Un bouton **« Revenir au modèle »** est disponible par jour et pour la semaine entière.

---

## 2. Le mode du jour

Chaque jour porte un `mode`, qui est bien plus qu'une étiquette : il **reconfigure les créneaux repas**.

| Mode | Trajets | Conséquences sur les repas |
|---|---|---|
| `presentiel` | oui | Déjeuner en gamelle, `transportable` ou `bon_froid` requis. Petit-déj expédié |
| `distanciel` | non | Déjeuner cuisiné possible. Petit-déj tranquille. Soirée libérée plus tôt |
| `libre` | non | Aucune contrainte |
| `absent` | — | Vacances, déplacement. Aucun créneau généré |

Basculer mercredi de `presentiel` à `distanciel` produit automatiquement quatre effets :

1. Les blocs de trajet disparaissent du planning
2. Le créneau déjeuner perd sa contrainte `transportable` et gagne l'accès aux recettes qui demandent une cuisson
3. Le petit-déjeuner glisse plus tard et passe de `petit-dej` expédié à repas complet
4. Le créneau du soir démarre plus tôt

Aucune de ces quatre choses n'est saisie à la main.

---

## 3. Les créneaux repas sont calculés, jamais stockés

**C'est la décision architecturale centrale de cette spec.**

```
créneaux du jour = f(événements, mode, règles)
```

Si les créneaux étaient stockés en dur, chaque modification d'événement obligerait à les resynchroniser manuellement — et ils dériveraient au premier oubli. En les recalculant à chaque affichage, ajouter une séance de salle fait apparaître ses deux repas associés sans aucune intervention.

Le planning devient **génératif** au lieu d'être rédigé à la main.


---

## 3 bis. Les modèles arrivent déjà remplis

Un modèle n'est pas un squelette vide. Il embarque **ses créneaux repas déjà positionnés, avec leurs horaires et leurs valeurs par défaut**. Charger une semaine ne demande donc aucune reconstruction : tout est là, seules les recettes des repas principaux restent à choisir.

```yaml
# 00-Modeles/cours.md
id: cours
nom: Semaine de cours
evenements:
  - jour: [lun, mar, mer, jeu, ven]
    type: cours
    debut: "09:10"
    fin: "17:10"
  - jour: lun
    type: rugby-entrainement
    debut: "20:00"
  # ...

creneaux_par_defaut:
  - jour: [lun, mar, mer, jeu, ven]
    type: petit-dej
    heure: "07:45"
    mode_selection: defaut-reconduit
    recette_defaut: bol-yaourt-banane-noix
  - jour: [lun, mar, mer, jeu, ven]
    type: dejeuner
    heure: "12:30"
    mode_selection: matching
  # ...
```

---

## 3 ter. Tous les créneaux ne méritent pas un matching

**C'est ici que se trouve le vrai gain de temps.** Une semaine complète compte environ vingt-cinq créneaux. Swiper vingt-cinq fois chaque dimanche est une corvée que tu abandonneras au bout de trois semaines.

Personne ne veut choisir son petit-déjeuner sept fois par semaine. Chaque créneau porte donc un mode de sélection :

| Mode | Comportement | Créneaux concernés |
|---|---|---|
| `matching` | Écran de swipe, choix hebdomadaire | déjeuner, soir-cuisine, post-entraînement, batch, dîner à deux |
| `defaut-reconduit` | Une recette par défaut, reconduite automatiquement. Modifiable d'un tap, jamais demandée | petit-déj, collation-trajet, repas d'avant-match |
| `fixe` | Jamais demandé, jamais affiché comme à planifier | recharge-express (sac de sport) |

Résultat concret sur une semaine d'alternance :

- **Créneaux totaux** : 26
- **Reconduits ou fixes** : 16 — rien à faire
- **À swiper réellement** : 10

Le rituel du dimanche passe d'une corvée de vingt minutes à environ cinq minutes.

Les valeurs par défaut se règlent une fois, dans les préférences, et se changent au fil de l'eau : si tu remplaces trois fois de suite ton petit-déj par un porridge, l'app propose de changer la valeur par défaut.

---

## 3 quater. Recalcul incrémental, jamais de remise à zéro

Ajouter ou supprimer un événement **ne réinitialise pas la semaine**. L'opération est une fusion en trois temps :

1. Recalculer la structure de créneaux à partir des événements
2. La comparer à la structure existante
3. Reporter les affectations sur les créneaux qui subsistent

### Identité stable des créneaux

Chaque créneau porte un identifiant qui survit au recalcul :

```
2026-W35-mer-dejeuner-1
= semaine · jour · type de créneau · rang d'apparition
```

Le rang est **attribué à la création et jamais renuméroté**. Sans ça, supprimer la première séance d'une journée qui en compte deux ferait glisser les rangs et perdre les affectations.

### Règle d'épinglage

Un créneau existe dans deux états :

| État | Comportement au recalcul |
|---|---|
| `auto` | Position et horaire recalculés librement |
| `epingle` | Figé. Ni déplacé ni supprimé sans confirmation |

**Toucher un créneau l'épingle.** Lui affecter une recette, déplacer son horaire ou modifier son type suffit. Le recalcul ne déplace donc jamais quelque chose que tu as décidé toi-même.

### Les trois cas de la fusion

| Situation | Comportement |
|---|---|
| Créneau conservé | Affectation préservée, aucune action |
| Créneau créé | Apparaît vide, ou avec sa valeur par défaut si `defaut-reconduit` |
| Créneau supprimé alors qu'une recette y était affectée | **Jamais de suppression silencieuse.** L'app signale l'orphelin et propose de le réaffecter, de le rendre à la liste de courses, ou de libérer la portion stockée qui lui était réservée |

Exemple : tu ajoutes une séance de salle le mercredi à 17h30. Deux créneaux apparaissent — collation à 17h15, repas à 19h20. Le déjeuner de midi et le batch du soir, déjà planifiés, ne bougent pas d'un millimètre.

---

## 3 quinquies. Repartir de la semaine précédente

Deuxième levier contre la replanification : le bouton **« Repartir de la semaine dernière »**.

Il reprend l'intégralité des affectations de la semaine précédente, puis applique le score de rotation :

- Les créneaux `defaut-reconduit` sont repris tels quels, sans commentaire
- Les repas principaux sont repris mais **marqués à revoir** si la recette date de moins de quatorze jours
- Les portions stockées disponibles sont réaffectées en priorité

Tu arrives donc sur une semaine déjà complète, avec trois ou quatre blocs signalés en orange à reconsidérer. C'est plus rapide que de repartir de zéro, sans pour autant manger la même chose toutes les semaines.

---

## 4. Bibliothèque de types d'événements

Tu n'écris pas un événement libre : tu piochesdans une palette. Chaque type porte ses conséquences alimentaires.

```yaml
id: salle-de-sport
nom: Salle de sport
duree_defaut: 150
intensite: elevee              # nulle | moderee | elevee
couleur: gym
genere_creneaux:
  - type: collation-pre-effort
    offset: -15                # minutes avant le début
    obligatoire: true
    note: "Court et sucré. Partir à jeun sur 2 h de séance coûte du muscle."
  - type: repas-post-effort
    offset: +20                # minutes après la fin
    obligatoire: true
    contraintes: [proteines, glucides-lents]
```

### Palette initiale

| Événement | Durée | Intensité | Créneaux générés |
|---|---|---|---|
| `salle-de-sport` | 150 min | élevée | collation avant · repas après |
| `rugby-entrainement` | 120 min | élevée | collation avant · repas après |
| `rugby-encadrement` | 90 min | modérée | collation légère avant |
| `rugby-match` | 360 min | élevée | repas avant-match (−180 min) · collation sac · repas retour |
| `cours` | 480 min | nulle | déjeuner |
| `entreprise` | 480 min | nulle | déjeuner · trajets si `presentiel` |
| `distanciel` | 480 min | nulle | déjeuner cuisiné |
| `diner-a-deux` | 120 min | nulle | créneau forcé sur `diner-a-deux` |
| `indisponible` | variable | nulle | aucun créneau, ou « Mange dehors » |

Un événement n'est donc pas une barre colorée sur un calendrier. C'est **une règle qui produit des créneaux repas avec leurs contraintes**.

### Cas particulier — deux efforts le même jour

Le vendredi d'alternance cumule salle le matin et rugby le soir. Deux événements `intensite: elevee` sur une même journée déclenchent un marqueur **« journée à forte charge »**, qui :

- rend obligatoires tous les créneaux générés, aucun ne peut être laissé vide
- ajoute un bonus de score aux recettes taguées `glucides-lents` et `proteines`
- affiche un avertissement si moins de quatre prises alimentaires sont planifiées

---

## 5. Interface

Le planning déjà généré reste l'écran principal. Chaque jour porte deux actions.

```
┌────────────────────────────────────────┐
│  MERCREDI          présentiel  ▾       │  ← bascule du mode
│  ──────────────────────────────────    │
│  7   10   13   16   19   22            │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░                    │
│                                        │
│  + Ajouter un événement                │  ← ouvre la palette
│  ↺ Revenir au modèle                   │  ← visible si surcharges
└────────────────────────────────────────┘
```

La bascule du mode est un simple menu à trois entrées. L'ajout d'événement ouvre la palette, puis demande uniquement l'heure de début — la durée par défaut du type est proposée et modifiable.

---

## 6. Promotion d'une semaine en modèle

Tes semaines de cours n'ont pas encore de rythme stable, puisque le distanciel n'est pas fixé. La bonne approche n'est pas de deviner maintenant, mais de **laisser le motif émerger**.

Quand une configuration se répète, un bouton **« Enregistrer comme modèle »** transforme la semaine surchargée en nouveau modèle réutilisable.

Tu passerais alors de deux à quatre modèles :

- `alternance`
- `cours-presentiel`
- `cours-hybride` — avec les jours de distanciel à leur place
- `vacances` ou `semaine-legere`

C'est mieux que de re-surcharger manuellement chaque semaine, et ça évite d'inventer aujourd'hui une structure que tu ne connais pas encore.

---

## 7. Impact sur les autres modules

**Matching** — les contraintes de créneau étant recalculées, une journée passée en distanciel débloque immédiatement les recettes non transportables sur le déjeuner. Aucune resynchronisation.

**Liste de courses** — supprimer un événement supprime ses créneaux, donc les recettes qui y étaient affectées, donc leurs ingrédients. La liste se recalcule.

**Portions stockées** — un créneau supprimé libère la portion qui lui était réservée, qui redevient disponible pour un autre jour.

**Notifications** — les délais préalables (sortir un pâton du congélateur 8 h avant) se recalculent à partir de l'heure réelle du créneau, pas de l'heure théorique du modèle.

---

## 8. Phasage

| Phase | Contenu |
|---|---|
| 1 | Deux modèles remplis (créneaux + horaires + défauts), figés |
| 1 | Modes de sélection : `matching`, `defaut-reconduit`, `fixe` |
| 2 | Bascule du mode `presentiel` / `distanciel` par jour |
| 2 | Recalcul incrémental avec identité stable et épinglage |
| 3 | Palette d'événements, ajout et suppression, retour au modèle |
| 3 | Repartir de la semaine précédente |
| 4 | Journée à forte charge, contraintes renforcées |
| 5 | Promotion d'une semaine en modèle personnalisé |

La bascule de mode en phase 2 couvre déjà l'essentiel du besoin réel. La palette complète peut attendre.
