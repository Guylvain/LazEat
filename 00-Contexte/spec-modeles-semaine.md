# Modèles de semaine — spécification

Source de vérité pour `src/lib/modeles-semaine.ts` et `src/lib/evenements.ts`.
Remplace toute inférence faite depuis les maquettes HTML.

---

## 1. Types d'événements et créneaux générés

Un événement n'est pas une barre sur un calendrier : **c'est une règle qui produit des créneaux repas.**

```yaml
types_evenement:
  - id: salle-de-sport
    intensite: elevee
    genere:
      - creneau: recharge          # collation courte avant l'effort
        offset_min: -15            # avant le début
        pilier: true
      - creneau: petit-dej         # vrai repas après la séance du matin
        offset_min: +25            # après la fin
        pilier: true

  - id: rugby-entrainement
    intensite: elevee
    genere:
      - creneau: collation
        offset_min: -75
        pilier: true
      - creneau: post-entrainement
        offset_min: +30

  - id: rugby-encadrement
    intensite: moderee
    genere:
      - creneau: collation
        offset_min: -75

  - id: rugby-match
    intensite: elevee
    genere:
      - creneau: avant-match
        offset_min: -120
        pilier: true
      - creneau: recharge
        offset_min: +180           # pendant, dans le sac
      - creneau: post-entrainement
        offset_min: +30
        pilier: true

  - id: cours
    intensite: nulle
    genere:
      - creneau: dejeuner
        heure: "12:30"
        pilier: true
        contrainte: transportable   # gamelle

  - id: entreprise
    intensite: nulle
    genere:
      - creneau: dejeuner
        heure: "12:30"
        pilier: true
        contrainte: transportable

  - id: teletravail
    intensite: nulle
    genere:
      - creneau: dejeuner
        heure: "13:00"
        pilier: true
        contrainte: aucune          # cuisine possible

  - id: soiree-libre                # pas d'événement le soir
    genere:
      - creneau: soir-cuisine
        heure: "19:00"
```

**Règle du petit-déjeuner.** Un créneau `petit-dej` est généré chaque jour à `heure_premier_evenement − 60 min`, sauf si un événement `salle-de-sport` en produit déjà un — auquel cas on ne double pas. Sans événement du tout, il est fixé à 10h.

**Règle du repas du soir.** Si aucun créneau n'existe après 18h, on ajoute `soir-cuisine` à 19h. Un jour sans événement le soir reste un jour où l'on dîne.

**Déduplication.** Deux créneaux du même type à moins de 90 minutes d'écart fusionnent, en gardant le plus contraint des deux.

**Journée à forte charge.** Deux événements `intensite: elevee` le même jour : tous les créneaux générés deviennent `pilier: true`.

---

## 2. Semaine d'alternance

```yaml
id: alternance
nom: Semaine d'alternance
jours:
  lundi:
    mode: presentiel
    evenements:
      - {type: trajet,             debut: "08:30", fin: "09:30"}
      - {type: entreprise,         debut: "09:30", fin: "17:30"}
      - {type: trajet,             debut: "17:30", fin: "18:30"}
      - {type: rugby-entrainement, debut: "20:00", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  mardi:
    mode: presentiel
    evenements:
      - {type: trajet,             debut: "08:30", fin: "09:30"}
      - {type: entreprise,         debut: "09:30", fin: "17:30"}
      - {type: trajet,             debut: "17:30", fin: "18:30"}
      - {type: rugby-encadrement,  debut: "19:00", fin: "20:30"}
      - {type: rugby-entrainement, debut: "20:30", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  mercredi:
    mode: distanciel
    evenements:
      - {type: trajet,          debut: "06:55", fin: "07:20"}
      - {type: salle-de-sport,  debut: "07:20", fin: "09:30"}
      - {type: trajet,          debut: "09:30", fin: "09:55"}
      - {type: teletravail,     debut: "10:00", fin: "18:00"}
      - {type: soiree-libre,    debut: "19:00", fin: "20:00"}   # batch cooking

  jeudi:
    mode: presentiel
    evenements:
      - {type: trajet,            debut: "08:30", fin: "09:30"}
      - {type: entreprise,        debut: "09:30", fin: "17:30"}
      - {type: trajet,            debut: "17:30", fin: "18:30"}
      - {type: rugby-encadrement, debut: "19:00", fin: "20:30"}
      - {type: trajet,            debut: "20:30", fin: "21:00"}
      - {type: soiree-libre,      debut: "21:15", fin: "22:00"}

  vendredi:
    mode: distanciel
    evenements:
      - {type: trajet,             debut: "06:55", fin: "07:20"}
      - {type: salle-de-sport,     debut: "07:20", fin: "09:30"}
      - {type: trajet,             debut: "09:30", fin: "09:55"}
      - {type: teletravail,        debut: "10:00", fin: "18:00"}
      - {type: rugby-entrainement, debut: "20:00", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  samedi:
    mode: libre
    evenements:
      - {type: soiree-libre, debut: "20:00", fin: "21:00"}

  dimanche:
    mode: libre
    evenements:
      - {type: trajet,      debut: "12:30", fin: "13:00"}
      - {type: rugby-match, debut: "13:00", fin: "19:00"}
      - {type: trajet,      debut: "19:00", fin: "19:30"}
    conditionnel: match_dimanche
```

### Créneaux attendus — contrôle de conformité

| Jour | Nombre | Détail |
|---|---|---|
| Lundi | 4 | petit-déj 7h30 · déjeuner 12h30 · collation 18h45 · post-entraînement 22h30 |
| Mardi | 5 | petit-déj 7h30 · déjeuner 12h30 · collation 17h45 · recharge 20h15 · post-entraînement 22h30 |
| **Mercredi** | **4** | **recharge 7h05 · petit-déj 9h55 · déjeuner 13h · soir-cuisine 19h** |
| Jeudi | 4 | petit-déj 7h30 · déjeuner 12h30 · collation 17h45 · soir-cuisine 21h15 |
| Vendredi | 5 | recharge 7h05 · petit-déj 9h55 · déjeuner 13h · collation 18h45 · post-entraînement 22h30 |
| Samedi | 3 | petit-déj 10h · déjeuner 13h · soir-cuisine 20h |
| Dimanche | 4 | petit-déj 8h30 · avant-match 11h · recharge 16h · post-entraînement 19h30 |

Le mercredi doit produire **quatre** créneaux, pas deux. C'est le test qui échoue aujourd'hui.

---

## 3. Semaine de cours

Deux réserves : les créneaux de salle ne sont pas encore fixés sur ce type de semaine, et les jours de distanciel ne sont pas connus. Le modèle reste donc en présentiel intégral, la bascule par jour permettant de l'ajuster à la main.

```yaml
id: cours
nom: Semaine de cours
jours:
  lundi:
    mode: presentiel
    evenements:
      - {type: trajet,             debut: "08:20", fin: "09:10"}
      - {type: cours,              debut: "09:10", fin: "17:10"}
      - {type: trajet,             debut: "17:10", fin: "17:50"}
      - {type: rugby-entrainement, debut: "20:00", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  mardi:
    mode: presentiel
    evenements:
      - {type: trajet,             debut: "08:20", fin: "09:10"}
      - {type: cours,              debut: "09:10", fin: "17:10"}
      - {type: trajet,             debut: "17:10", fin: "18:30"}
      - {type: rugby-encadrement,  debut: "19:00", fin: "20:30"}
      - {type: rugby-entrainement, debut: "20:30", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  mercredi:
    mode: presentiel
    evenements:
      - {type: trajet,       debut: "08:20", fin: "09:10"}
      - {type: cours,        debut: "09:10", fin: "17:10"}
      - {type: trajet,       debut: "17:10", fin: "17:50"}
      - {type: soiree-libre, debut: "19:00", fin: "20:00"}   # batch cooking

  jeudi:
    mode: presentiel
    evenements:
      - {type: trajet,            debut: "08:20", fin: "09:10"}
      - {type: cours,             debut: "09:10", fin: "17:10"}
      - {type: trajet,            debut: "17:10", fin: "18:30"}
      - {type: rugby-encadrement, debut: "19:00", fin: "20:30"}
      - {type: trajet,            debut: "20:30", fin: "21:00"}
      - {type: soiree-libre,      debut: "21:15", fin: "22:00"}

  vendredi:
    mode: presentiel
    evenements:
      - {type: trajet,             debut: "08:20", fin: "09:10"}
      - {type: cours,              debut: "09:10", fin: "17:10"}
      - {type: trajet,             debut: "17:10", fin: "17:50"}
      - {type: rugby-entrainement, debut: "20:00", fin: "22:00"}
      - {type: trajet,             debut: "22:00", fin: "22:30"}

  samedi:
    mode: libre
    evenements:
      - {type: soiree-libre, debut: "20:00", fin: "21:00"}

  dimanche:
    mode: libre
    evenements:
      - {type: trajet,      debut: "12:30", fin: "13:00"}
      - {type: rugby-match, debut: "13:00", fin: "19:00"}
      - {type: trajet,      debut: "19:00", fin: "19:30"}
    conditionnel: match_dimanche
```

---

## 4. Effet de la bascule présentiel / distanciel

Basculer un jour ne change pas ses événements de rugby ni de salle. Il agit sur trois choses :

1. **Les trajets domicile-travail disparaissent** — ceux qui encadrent un événement `cours` ou `entreprise`. Les trajets liés au sport ou à la salle restent.
2. **L'événement `cours` ou `entreprise` devient `teletravail`** — donc le déjeuner passe de 12h30 contraint `transportable` à 13h sans contrainte.
3. **Le petit-déjeuner glisse** puisqu'il est calculé depuis le premier événement de la journée.

Le mouvement inverse rétablit l'état du modèle. Aucun autre jour n'est affecté.
