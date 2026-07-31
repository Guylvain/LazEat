# Feuille de route

Vue d'ensemble : ce qui est fait, ce qui reste, et l'ordre des dépendances.
Mise à jour au 31 juillet 2026, fin de session 3b.

---

## Où on en est

L'application fait aujourd'hui le tour complet du parcours : décider le
dimanche, acheter, cuisiner, valider, garder les restes.

**Ce qui manque n'est plus une fonctionnalité, c'est de l'usage réel.** Le
scoring a besoin de données de goût, la rotation d'un historique, et les 165
conditionnements du référentiel ne se vérifieront qu'en magasin.

### Sessions livrées

| Session | Contenu | Vérifié |
|---|---|---|
| 1 | Échafaudage PWA, bibliothèque, mode cuisine, planning en lecture seule | jamais, à l'époque |
| 2 | Persistance, feuille de sélection, planning modifiable, liste de courses, chaînage, onglets | jamais, à l'époque |
| 2bis | Bibliothèque vide au build, réactivité inter-écrans, feuille recouverte | manuel, complet |
| 2ter | Jour de batch explicite, sélecteur à 10 portions, **portions restantes**, écran de bilan | manuel, complet |
| 2quater | `collation-trajet` enfin généré, efforts enchaînés, fusion additive | manuel, complet |
| 3a | **Dates réelles**, chaîne créneau → cuisine → bilan, ouverture sur aujourd'hui | manuel, complet |
| 3b | **Rituel du dimanche**, correctif de sur-réservation | manuel, complet |

Les sessions 1 et 2 n'avaient jamais été validées : aucune recette n'avait
jamais été chargée dans le build de production. Trois sessions de remise en
état ont été nécessaires pour s'en apercevoir et le corriger.

---

## Ce qui reste

### Session 3c — Correctifs du rituel, Supabase, dépenses

**Trois correctifs visuels du rituel**, trouvés à l'usage :

- L'en-tête recouvre le haut de la feuille, le titre du créneau est coupé.
  `sticky top-0 z-[60]` contre `fixed bottom-0 max-h-[88vh]` : deux systèmes
  de positionnement qui ne se coordonnent pas. Réserver la place au lieu de
  superposer.
- **Le jour n'est jamais affiché.** La feuille montre « 22:30 », jamais
  « Mardi 22:30 ». Sans importance dans l'usage normal, où `EnTeteJour` est
  juste au-dessus ; bloquant dans le rituel, qui saute d'un bout à l'autre
  de la semaine. `CreneauCalcule` porte déjà `jour`, et `dateDuJour()` existe
  depuis la 3a.
- Sur écran large, la feuille reste une colonne étroite sous un en-tête
  pleine largeur. À trancher : le rituel est-il un écran de téléphone, ou
  s'aligne-t-on sur la largeur de la feuille ?

**Supabase** — remplace `PersistanceLocalStorage` derrière l'interface
`Persistance`, inchangée depuis la session 2 précisément pour que ce soit une
substitution et non une réécriture. Débloque la synchronisation entre
appareils, et le stockage des photos.

**Journal de dépenses** — photo du ticket, magasin, montant, saisis à la
main. **Pas d'OCR**, décision prise. Remonté volontairement ici alors qu'il
appartient au module stock : sa valeur vient de l'accumulation, plus il
démarre tôt plus la comparaison avec les 204,82 € d'Uber Eats sera parlante.

### Session 4a — Les placards

Stock par ingrédient, vues par emplacement (frigo, congélateur, épices,
fruits, placard) — qui ne sont pas des entités mais des filtres sur
`emplacement`.

Décrément à la **validation** de la recette, jamais à l'achat ni à la
planification. Autoriser le passage en négatif : ne jamais bloquer
l'utilisateur en cuisine.

Débloque : une liste de courses qui soustrait ce qu'on a déjà.

### Session 4b — Le retour de courses

Validation du panier après les courses, qui incrémente le stock. Rituel de
recalage hebdomadaire, trente secondes le dimanche — sans lui, l'inventaire
dérive et l'outil est abandonné.

Contrainte des douze boîtes : chaque portion stockée occupe un contenant,
le système sait combien sont libres et refuse un batch de 4 s'il n'en reste
que 2.

Décrément automatique des portions stockées de **composant** — un pâton
consommé par une garniture. Signalé par le rapport 2ter, laissé en attente du
moteur de stock.

### Session 5a — Le scoring

Note personnelle, malus de rotation à 14 jours, bonus si les ingrédients sont
en stock, bonus sur les tags nutritionnels prioritaires, bonus léger pour les
recettes `a-tester`.

Débloque **« Repartir de la semaine dernière »**. Le rapport 3b a montré que
ce bouton n'est pas gratuit sans scoring : la spec le lie explicitement au
marquage « à revoir » de ce qui a été cuisiné il y a moins de 14 jours. Sans
ça il recopierait la semaine à l'identique et trahirait son intention.

Débloque aussi la **suggestion automatique** d'une portion dont la DLC
approche, techniquement possible depuis la 3a mais dont l'interface reste à
décider.

### Session 5b — La boucle de rétroaction

Raisons de rejet structurées — « trop long » retire le créneau concerné au
lieu de condamner la recette.

Remontée au niveau de l'ingrédient : trois rejets pour raison de goût
partageant un ingrédient déclenchent une proposition de le passer `exclu`.
C'est ce mécanisme qui aurait détecté tout seul le problème des épinards.

Suivi du gaspillage : une portion périmée sans consommation pose une question,
mangée ou jetée. Produit gratuitement une métrique et détecte le sur-batch.

### Session 6 — La palette d'événements

Ajouter, supprimer, déplacer un événement. Bouton « Revenir au modèle ».
Journée à forte charge. Promotion d'une semaine en modèle personnalisé.

**Débloque le créneau `diner-a-deux`, aujourd'hui inatteignable.** Douze
recettes le déclarent, il a son budget, son filtre de matching et son mode de
sélection — mais **aucun événement ne le produit**. C'est exactement le
schéma de `collation-trajet` avant la session 2quater : un créneau vivant
partout sauf là où il se génère. La différence est que celui-ci est assumé
depuis la session 2, pas accidentel.

---

## Ce qui n'appartient à aucune session

### Dette technique

| Sujet | Pourquoi ça compte |
|---|---|
| `devOptions.enabled: false` sur le service worker | Il ressert des builds périmés en développement. A déjà invalidé une campagne de tests entière |
| Fusionner `PIEGES-ENVIRONNEMENT.md` dans `CLAUDE.md` | Neuf pièges qui ont chacun coûté du temps, aujourd'hui lisibles seulement depuis le coffre |
| Écran de réglages des recettes par défaut | Aujourd'hui on ne peut changer un défaut qu'en choisissant une recette dans un vrai créneau |
| Le libellé « tu as mis 4/5 » de `planning-selection-enrichie.html` | L'échelle est passée à 1-4, la maquette fait foi pour le rendu |

### Contenu

| Sujet | Effort |
|---|---|
| **Vérifier les 165 conditionnements en magasin** | Une session de courses, le fichier ouvert. Le plus rentable de tous |
| Compléter les 38 stubs d'ingrédients | À la demande, quand ils servent réellement |
| Photos personnelles des recettes | Dépend de Supabase Storage, donc de la 3c |
| Tester les sardines et les maquereaux | Deux ingrédients `a-tester` qui portent les oméga-3 |
| Magasins préférés, prix, disponibilité | Décrit dans l'inventaire, aucune session rattachée |

---

## Les dépendances qui commandent l'ordre

Plus important que les numéros de session :

```
usage réel ──▶ données de goût ──▶ scoring (5a) ──▶ repartir de la semaine dernière
     │
     └──▶ conditionnements vérifiés ──▶ liste de courses juste

Supabase (3c) ──▶ photos perso
              └──▶ synchronisation ordinateur / téléphone

placards (4a) ──▶ décrément d'ingrédients ──▶ liste qui soustrait le stock
              └──▶ contenants (4b) ──▶ contrainte des 12 boîtes

palette d'événements (6) ──▶ créneau dîner à deux ──▶ 12 recettes débloquées
```

**La seule dépendance qu'aucune session ne peut satisfaire, c'est l'usage
réel.** Le scoring, la rotation et la remontée au niveau de l'ingrédient
attendent tous des données que seules quelques semaines vécues produiront.

C'est pourquoi la prochaine étape n'est pas une session de développement.

---

## La prochaine chose à faire

**Vivre une vraie semaine.** Planifier un dimanche avec le rituel, faire les
courses avec la liste, cuisiner, valider, déclarer les restes.

Deux questions se répondront toutes seules : est-ce que le rituel tient au-delà
de cinq minutes, et est-ce que la liste de courses est juste en magasin.

Puis la session 3c.
