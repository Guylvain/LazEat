# Modèles de semaine — spécification

Source de vérité pour `src/lib/modeles-semaine.ts` et `src/lib/evenements.ts`.
Remplace toute inférence faite depuis les maquettes HTML.

---

## 1. Types d'événements et créneaux générés

Un événement n'est pas une barre sur un calendrier : **c'est une règle qui produit des créneaux repas.**

> **Les noms de créneaux ci-dessous sont les identifiants exacts de
> l'énumération `Creneau`.** Les versions abrégées employées jusqu'au
> 31/07/2026 — `collation`, `recharge`, `post-entrainement` — n'existent nulle
> part dans le code et ont produit une erreur de correspondance décrite après
> ce bloc. Ne jamais réintroduire d'abréviation ici.

```yaml
types_evenement:
  - id: salle-de-sport
    intensite: elevee
    genere:
      - creneau: recharge-express  # courte, dans le sac, avant l'effort
        offset_min: -15            # avant le début
        pilier: true
      - creneau: petit-dej         # vrai repas après la séance du matin
        offset_min: +25            # après la fin
        pilier: true

  - id: rugby-entrainement
    intensite: elevee
    genere:
      - creneau: collation-trajet  # mangée dans les transports en rentrant
        offset_min: -75
        pilier: true
      - creneau: post-entrainement-rapide
        offset_min: +30

  - id: rugby-encadrement
    intensite: moderee
    genere:
      - creneau: collation-trajet
        offset_min: -75

  - id: rugby-match
    intensite: elevee
    genere:
      - creneau: avant-match
        offset_min: -120
        pilier: true
      - creneau: recharge-express
        offset_min: +180           # pendant, dans le sac
      - creneau: post-entrainement-rapide
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

**Règle des efforts enchaînés.** Quand un `rugby-entrainement` commence moins de 30 minutes après la fin d'un autre événement, son créneau amont **n'est pas** une `collation-trajet` à −75 : c'est une `recharge-express` à **−15**, sur le modèle de `salle-de-sport`.

Sans cette règle, le mardi produit une collation à 19h15 — soit **en plein milieu de l'encadrement des cadets, de 19h à 20h30**. Un créneau où l'on ne peut physiquement pas manger. Le bon moment est 20h15, entre la fin de l'encadrement et le début de l'entraînement personnel, sac de sport à la main : c'est un ravitaillement court, pas un repas de trajet.

C'est ce créneau de 20h15 que visent `recharge-abricots-compote`, `recharge-dattes-amandes` et `boisson-recuperation`, écrites après la session 2 pour combler ce trou (`ETAT-DU-PROJET.md` §11).

**Règle du petit-déjeuner.** Un créneau `petit-dej` est généré chaque jour à `heure_premier_evenement − 60 min`, sauf si un événement `salle-de-sport` en produit déjà un — auquel cas on ne double pas. Sans événement du tout, il est fixé à 10h.

**Règle du repas du soir.** Si aucun créneau n'existe après 18h, on ajoute `soir-cuisine` à 19h. Un jour sans événement le soir reste un jour où l'on dîne.

**Règle du batch cooking.** Le créneau de batch de la semaine est **désigné explicitement** par le champ `batch: true` sur un événement `soiree-libre` du modèle. Son `soir-cuisine` devient `batch`. Un seul par semaine.

Il ne doit **pas** être déduit du `mode` du jour. La règle « premier jour distanciel ou libre portant une soiree-libre », implémentée en session 1, tombe juste en semaine d'alternance — mercredi y est distanciel — mais désigne **samedi** en semaine de cours, où les cinq jours sont présentiel par défaut.

Or samedi est le dernier soir cuisinable de la semaine. Le créneau de batch ne peut donc jamais se trouver en amont d'un consommateur, et **le chaînage de production devient structurellement impossible** : `pate-a-pizza-maison` ne peut plus alimenter aucune garniture, `riz-cuit-en-quantite` aucun plat de riz. Le chaînage refuse à juste titre d'insérer une recette productrice après son consommateur ; c'est la désignation du jour qui est fautive, pas lui.

Constaté en test le 31/07/2026 : en semaine de cours, aucune garniture de pizza ne pouvait déclencher le chaînage.

**Défaut de correspondance constaté le 31/07/2026, à corriger dans le code.** `creneauxDepuisEvenement()` traduit **à la fois** `collation` et `recharge` en `recharge-express`. Conséquence : **aucun événement ne produit jamais de créneau `collation-trajet`.** Le type existe pourtant partout ailleurs — énumération, budget, conseil, mode de sélection, filtre de matching, préférences — et huit recettes le déclarent.

Effet concret : le mardi 17h45, entre le travail et le club, l'app propose des abricots secs et une boisson de récupération. `tartines-collation-trajet`, dont le texte dit « Recette du mardi. Préparée le matin, mangée à 17h30 dans les transports », n'apparaît pas — elle déclare `collation-trajet`, un créneau qui n'existe jamais.

Les deux types ne sont pas interchangeables : `collation-trajet` se prépare la veille et se mange sans couverts dans les transports ; `recharge-express` ne se prépare pas et vit en permanence dans le sac de sport.

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
      - {type: soiree-libre,    debut: "19:00", fin: "20:00", batch: true}

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

Identifiants exacts de l'énumération `Creneau`, comme en §1. `npm run verifier-modeles` compare désormais **le type et l'heure** de chaque créneau, plus seulement leur nombre — c'est ce trou qui avait laissé passer la confusion `collation-trajet` / `recharge-express`.

| Jour | Nombre | Détail |
|---|---|---|
| Lundi | 4 | `petit-dej` 7h30 · `dejeuner` 12h30 · `collation-trajet` 18h45 · `post-entrainement-rapide` 22h30 |
| **Mardi** | 5 | `petit-dej` 7h30 · `dejeuner` 12h30 · `collation-trajet` 17h45 · **`recharge-express` 20h15** · `post-entrainement-rapide` 22h30 |
| Mercredi | 4 | `recharge-express` 7h05 · `petit-dej` 9h55 · `dejeuner` 13h · **`batch` 19h** |
| Jeudi | 4 | `petit-dej` 7h30 · `dejeuner` 12h30 · `collation-trajet` 17h45 · `soir-cuisine` 21h15 |
| Vendredi | 5 | `recharge-express` 7h05 · `petit-dej` 9h55 · `dejeuner` 13h · `collation-trajet` 18h45 · `post-entrainement-rapide` 22h30 |
| Samedi | 3 | `petit-dej` 10h · `dejeuner` 13h · `soir-cuisine` 20h |
| Dimanche | 4 | `petit-dej` 8h30 · `avant-match` 11h · `recharge-express` 16h · `post-entrainement-rapide` 19h30 |

Deux lignes portent l'essentiel de ce qui a été corrigé le 31/07/2026.

**Mercredi** produit `batch`, pas `soir-cuisine` — la ligne disait encore `soir-cuisine` alors que la règle du batch était déjà appliquée et vérifiée. C'est ce genre d'écart entre le texte et le comportement qui rend une table de conformité inutile.

**Mardi** est le seul jour à porter les deux types de collation, et c'est ce qui les distingue : `collation-trajet` à 17h45 se mange dans les transports entre le travail et le club, `recharge-express` à 20h15 sort du sac de sport entre l'encadrement et l'entraînement. Un repas de trajet à 20h15 n'aurait aucun sens, et une barre de céréales à 17h45 ne tiendrait pas jusqu'à 22h30.

---

## 3. Semaine de cours

Deux réserves : les créneaux de salle ne sont pas encore fixés sur ce type de semaine, et les jours de distanciel ne sont pas connus. Le modèle reste donc en présentiel intégral, la bascule par jour permettant de l'ajuster à la main.

**C'est précisément ce présentiel intégral qui cassait le batch cooking** tant que le jour était déduit du mode. Mercredi porte `batch: true` ici comme en alternance, indépendamment de son mode.

### Créneaux attendus — contrôle de conformité

Absente jusqu'au 31/07/2026 : `npm run verifier-modeles` ne contrôlait que la semaine d'alternance, ce qui explique que le problème du jour de batch soit passé inaperçu. Le script couvre désormais les deux modèles, types et heures compris.

| Jour | Nombre | Détail |
|---|---|---|
| Lundi | 4 | `petit-dej` 7h20 · `dejeuner` 12h30 · `collation-trajet` 18h45 · `post-entrainement-rapide` 22h30 |
| Mardi | 5 | `petit-dej` 7h20 · `dejeuner` 12h30 · `collation-trajet` 17h45 · `recharge-express` 20h15 · `post-entrainement-rapide` 22h30 |
| **Mercredi** | **3** | `petit-dej` 7h20 · `dejeuner` 12h30 · **`batch` 19h** |
| Jeudi | 4 | `petit-dej` 7h20 · `dejeuner` 12h30 · `collation-trajet` 17h45 · `soir-cuisine` 21h15 |
| Vendredi | 4 | `petit-dej` 7h20 · `dejeuner` 12h30 · `collation-trajet` 18h45 · `post-entrainement-rapide` 22h30 |
| Samedi | 3 | `petit-dej` 10h · `dejeuner` 13h · `soir-cuisine` 20h |
| Dimanche | 4 | `petit-dej` 8h30 · `avant-match` 11h · `recharge-express` 16h · `post-entrainement-rapide` 19h30 |

Table confirmée par exécution le 31/07/2026, elle n'est plus une déduction.

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
      - {type: soiree-libre, debut: "19:00", fin: "20:00", batch: true}

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
