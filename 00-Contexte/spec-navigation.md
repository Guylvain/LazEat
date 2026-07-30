# Navigation — spécification

À fusionner dans `DESIGN.md`, section Composants.

---

## Le modèle

**Barre d'onglets en bas**, trois destinations. C'est le seul schéma qui reste atteignable au pouce sur un téléphone tenu d'une main, et l'app est installée en PWA : elle doit se comporter comme une app native, pas comme un site.

```
┌────────────────────────────┐
│                            │
│        contenu             │
│                            │
├────────────────────────────┤
│   Planning  Recettes  ⌾Courses │
└────────────────────────────┘
```

| Onglet | Route | Rôle |
|---|---|---|
| **Planning** | `/planning` | Écran par défaut au lancement. C'est le hub : on y décide |
| **Recettes** | `/recettes` | Parcourir, chercher, cuisiner sans passer par le planning |
| **Courses** | `/courses` | La liste, consultée en magasin |

---

## Les écrans qui masquent la barre

**Le mode cuisine** (`/recettes/:id/cuisine`) est un mode immersif. Aucune barre d'onglets : on ne navigue pas pendant qu'on cuisine, et chaque pixel compte sur un téléphone posé sur un plan de travail. Il porte son propre bouton de retour en haut à gauche.

**La feuille de sélection** est une modale, pas une route. Elle recouvre la barre pendant qu'elle est ouverte.

**Le détail d'une recette** (`/recettes/:id`) garde la barre et se comporte comme un empilement : un bouton retour en haut ramène à l'écran précédent, qu'on vienne de la bibliothèque ou du planning.

---

## Comportement

**Retour à la racine.** Toucher l'onglet déjà actif remonte en haut de la liste, ou revient à la racine si on est dans un sous-écran.

**Mémoire de position.** Chaque onglet conserve son état : revenir sur Recettes après un détour par Planning doit retrouver les filtres et la position de défilement.

**Pastille sur Courses.** Le nombre d'articles restant à acheter, affiché en `amber` sur `moss-deep`. Rien si la liste est vide ou entièrement cochée.

**Bouton retour matériel Android.** Il remonte la pile, il ne quitte pas l'app depuis un sous-écran.

---

## Rendu

Fond `white`, ombre portée vers le haut : `0 -2px 12px rgba(36, 64, 53, .07)`.
Hauteur 58 px, **plus `env(safe-area-inset-bottom)`** — sans ça, la barre passe sous l'indicateur d'accueil des iPhone en mode PWA installée.

Chaque onglet : icône de 24 px au-dessus d'un libellé en 10,5 px, graisse 600.
Actif en `moss`, inactif en `sage`. Aucun fond, aucun soulignement : seule la couleur distingue.
Zone tactile de 58 px de haut sur toute la largeur de l'onglet.

Icônes dessinées à la main en SVG, trait de 1,5 px, dans `components/icons/` :
- **Planning** — une grille de calendrier
- **Recettes** — un livre ouvert, ou une casserole
- **Courses** — une liste cochée, ou un panier

Rayon : la barre est un élément qu'on touche, mais elle est ancrée au bord de l'écran. **Rayon nul**, avec un filet supérieur `1px solid var(--mist)`. Les rayons sont réservés aux éléments qui flottent.

---

## Point laissé ouvert

L'accès rapide au repas du soir n'est pas résolu. À 22h30 en rentrant d'entraînement, le chemin actuel fait quatre gestes : Planning, trouver le bon jour, toucher le créneau, lancer le mode cuisine.

Le raccourci évident serait un écran qui ouvre directement sur « ce soir » — mais il suppose que le planning soit rattaché à des dates réelles, ce qui reste la décision de session 3. À reprendre à ce moment-là, pas avant.

En attendant, l'onglet Planning s'ouvre sur le jour sélectionné en dernier, ce qui limite les dégâts.
