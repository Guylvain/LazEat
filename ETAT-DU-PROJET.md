# LazEat — État du projet

Document de reprise. À fournir en début de toute nouvelle conversation pour repartir sans rejouer l'historique.

Dernière mise à jour : fin de session 2.

---

## 1. À qui ça sert et pourquoi

Liam, 25 ans, master Data Engineering & IA en alternance à l'EFREI, joueur et éducateur de rugby en club, basé en Île-de-France.

**Le problème.** Il s'entraîne ou encadre cinq soirs par semaine et saute des repas — certains jours un seul, souvent livré. Le déficit énergétique qui en résulte explique une fatigue chronique.

La cause n'est ni le manque de connaissances ni la motivation, mais une **friction de décision** : à 22h30 en rentrant d'entraînement, « qu'est-ce que je mange » n'a pas de réponse préparée, donc la réponse par défaut est la livraison. 204,82 € sur Uber Eats en juillet 2026, soit environ quatre commandes de 50 € réparties sur plusieurs repas.

**Ce que l'app fait.** Elle déplace toutes les décisions alimentaires vers le week-end, pour que la semaine ne comporte plus que de l'exécution.

Application **personnelle, mono-utilisateur**. Toute décision se tranche par « est-ce que ça sert ce cas précis ? », jamais par « est-ce une bonne fonctionnalité en général ».

---

## 2. Les objectifs biologiques

Issus d'un bilan sanguin du 23/07/2026. Ils pilotent les tags nutritionnels des recettes.

| Cible | Constat | Levier |
|---|---|---|
| Remonter les folates | 3,30 ng/ml pour une norme > 3,89 | Légumineuses, petits pois, chou-fleur, agrumes, mâche |
| Baisser les triglycérides | 1,50 g/L, ×4 en deux ans | Moins de sucres rapides et d'alcool, plus d'oméga-3 |
| Couvrir la dépense | 5 séances/semaine, repas sautés | Régularité, densité glucidique |

Tout le reste du bilan était normal : NFS, foie, reins, thyroïde, ferritine, glycémie, sérologies. La CRP est passée de 56,80 à 1,40 en deux ans.

**Le juge de paix du projet** : refaire doser folates et triglycérides vers novembre 2026. Si la B9 repasse au-dessus de 3,89 et les triglycérides sous 1,20, le système a fonctionné — quelle que soit l'élégance du code.

Point non résolu : la fatigue n'est pas expliquée par le bilan. Vitamine D et ferritine méritent un dosage si elle persiste.

---

## 3. Contraintes réelles

### Emploi du temps

**Deux types de semaine alternent.**

*Semaine de cours* — école 9h10 à 17h10, du lundi au vendredi. Des jours de distanciel sont possibles mais pas encore fixés.

*Semaine d'alternance* — entreprise lundi, mardi, jeudi de 9h30 à 17h30, avec une heure de trajet dans chaque sens. Télétravail mercredi et vendredi, avec salle de sport de 7h20 à 9h30 ces deux matins.

**Rugby, identique dans les deux cas** : entraînement lundi 20h-22h, encadrement des cadets mardi 19h-20h30 puis entraînement personnel 20h30-22h, encadrement jeudi 19h-20h30, entraînement vendredi 20h-22h, match dimanche 13h-19h certaines semaines.

**Le mardi est le jour critique** : quatorze heures séparent le déjeuner du repas suivant si aucune collation n'est planifiée. **Le vendredi est le jour le plus chargé** : salle le matin, rugby le soir.

### Goûts

**Exclus** — épinards, brocoli, choux de Bruxelles, avocat, miel.
**Conditionnel** — œufs acceptés uniquement intégrés à une préparation, jamais seuls.
**À tester** — sardines et maquereaux, jamais goûtés.
**Substitutions actées** — chou-fleur pour brocoli et choux de Bruxelles ; petits pois, mâche et agrumes pour les épinards ; saumon et noix si les sardines échouent.

### Matériel

Four, micro-ondes, quatre plaques, mixeur, bouilloire, grille-pain, poêles, casseroles, plats à four. Congélateur pouvant accueillir **environ 12 boîtes** — c'est la vraie limite du batch cooking, pas le temps.

Manquent encore : une balance de cuisine et des boîtes supplémentaires.

---

## 4. Où en est le projet

### Deux dépôts GitHub privés

**`LazEat`** — coffre Obsidian, source de vérité des données.

```
00-Contexte/    contexte projet, specs, chartes, maquettes
01-Recettes/    35 recettes + _gabarit-recette.md + inventaire
02-Ingredients/ referentiel-ingredients.yaml (157 entrées) + README
03-Semaines/    vide
04-Suivi/       vide
assets/         vide
```

**`LazEat-app`** — PWA React, déployée sur Vercel.

```
CLAUDE.md              contexte permanent des sessions
DESIGN.md              système visuel, fait foi pour les valeurs
BRIEF-SESSION-*.md     périmètres
BUGS-SESSION-1.md      rapport de bugs
data/                  copie du coffre, via npm run sync-vault
  recettes/            35 .md
  referentiel-ingredients.yaml
  contexte/            copie de 00-Contexte
src/
scripts/               sync-vault.mjs, validate.ts, verifier-modeles.ts
```

Les deux dépôts sont côte à côte. `npm run sync-vault` recopie le coffre vers `data/`.

### Stack

Vite · React · TypeScript · Tailwind v4 · vite-plugin-pwa · Zod · gray-matter · polices auto-hébergées via fontsource · persistance localStorage derrière une interface asynchrone · déploiement Vercel Hobby.

### Sessions livrées

**Session 1** — échafaudage PWA installable et hors-ligne, types dérivés de schémas Zod, parseur markdown avec validation, bibliothèque de recettes avec filtres et recherche, détail avec sélecteur de portions, mode cuisine avec minuteurs à timestamp absolu et Wake Lock, planning en lecture seule.

**Correctifs post-session 1** — polices (`--font-display` pointait vers `Fraunces` alors que fontsource déclare `Fraunces Variable`), interpolation des ingrédients qui perdait le nom, modèles de semaine reconstruits depuis une spec dédiée, conformité du mode cuisine à la charte, `--verbose` sur validate.

**Session 2** — persistance derrière `src/lib/persistance.ts`, feuille de sélection, planning modifiable, liste de courses agrégée avec chaînage de production, barre d'onglets de navigation.

### Ce qui fonctionne et a été vérifié

`npm run validate` passe à zéro erreur sur 35 recettes et 157 ingrédients. `npm run verifier-modeles` reproduit les comptes de créneaux attendus. `tsc --noEmit` et `npm run build` passent.

---

## 5. Les règles métier à ne jamais violer

**Une recette `type_production: composant` n'apparaît jamais dans le matching d'un créneau repas.** Elle ne sort que sur un créneau `batch`, ou par chaînage automatique. Concerne `pate-a-pizza-maison` et `riz-cuit-en-quantite`.

**Un ingrédient `provenance: produit` ne va jamais sur la liste de courses.** On n'achète pas un pâton de pizza ni du riz cuit. S'il en manque, l'app propose d'insérer la recette productrice au planning — c'est le chaînage de production. Sans cette règle, la liste devient absurde dès la première semaine.

**Les créneaux repas sont calculés depuis les événements, jamais stockés.** Seules les affectations sont persistées. Ça garantit qu'ajouter un événement fait apparaître ses repas sans resynchronisation manuelle.

**Toucher un créneau l'épingle.** Le recalcul ne déplace jamais ce que l'utilisateur a décidé.

**Créer une portion stockée ne retouche pas au stock d'ingrédients** — ils ont déjà été décrémentés à la validation de la recette. C'est l'erreur qui ferait diverger l'inventaire.

**Un sous-groupe d'ingrédients est toujours additif.** Une alternative se décrit en prose dans les notes, sinon l'agrégation achète les deux versions.

**Hiérarchie des sources en cas de contradiction** : `DESIGN.md` pour les valeurs, les maquettes pour le rendu, les briefs pour le périmètre et les règles métier. Quand un brief contredit `DESIGN.md` sur une valeur, le brief a tort.

---

## 6. Le système visuel

Détail complet dans `DESIGN.md`, rendu dans `charte-graphique-v3.html`.

**Palette** — `cream #EEF0E8` · `white #FCFCF9` · `moss #33574A` · `moss-deep #244035` · `amber #E2A33F` · `sage #7E8F84` · `sage-dark #5F6F65` · `mist #DBE0D5`

**Typographie** — Fraunces en display avec `SOFT 100`, `WONK 1` et `opsz` suivant la taille ; Figtree pour le texte. Exception unique : `WONK 0` sur le minuteur du mode cuisine.

**Deux règles structurantes** :

*Arrondi pour ce qui se touche, angle vif pour ce qui se mesure.* Un bloc de trente minutes avec un rayon en pilule se lit plus court qu'il n'est.

*L'ambre ne porte jamais de texte sur fond clair* — 2,1:1 mesuré. Sur `moss-deep` il passe à 5,1:1.

**Le texte d'une carte de repas change de nature selon l'état.** Vide, elle affiche le créneau et son conseil : c'est de l'aide à la décision. Remplie, elle affiche la recette et ses métadonnées : c'est ce qui sert en cuisine.

---

## 7. Les données

**35 recettes** au format markdown avec frontmatter YAML validé par Zod. Chaque recette porte son type de production, sa famille, ses créneaux compatibles, ses temps, ses portions, sa tenue au lendemain, son registre, son aptitude à la cuisine à deux avec découpage en postes, ses tags nutritionnels et son historique.

Six d'entre elles remplacent directement des commandes en livraison identifiées dans son historique Uber Eats : pizza, poulet croustillant, panuozzo, gnocchis, bol asiatique, kebab, burger.

**157 ingrédients** dont environ 45 stubs de substituts à compléter. Les champs `conditionnement` sont des estimations — ils doivent être vérifiés en faisant les courses une fois avec le fichier ouvert. C'est le principal travail de contenu restant.

**Couverture par créneau** : soir-cuisine 22 · dîner à deux 12 · déjeuner 11 · post-entraînement 10 · collation-trajet 8 · batch 6 · recharge-express 4 · petit-déj 3 · avant-match 2.

Les créneaux `petit-dej` et `avant-match` sont les plus pauvres — deux ou trois recettes chacun. À étoffer si le matching paraît répétitif.

---

## 8. Ce qui reste ouvert

**Les dates réelles.** Le planning est abstrait : « lundi » n'est rattaché à aucune date. Conséquence, l'état « faite » d'un créneau n'est pas déclenchable, l'historique « cuisinée il y a 12 jours » n'est pas calculable, et il n'existe pas de raccourci vers le repas du soir. **Décision de session 3.**

**L'accès rapide au repas de ce soir.** À 22h30, le chemin fait quatre gestes. C'est trop pour le moment précis où l'app doit gagner contre la livraison. Dépend des dates réelles.

**La synchronisation entre appareils.** localStorage garde les données sur l'appareil : planifier sur l'ordinateur puis faire ses courses avec le téléphone ne fonctionne pas. En attendant Supabase, tout se fait depuis le téléphone. L'interface de persistance est asynchrone précisément pour que la bascule soit une substitution, pas une réécriture.

**Les conditionnements du référentiel** restent des estimations.

**Le rendu visuel n'a jamais été vérifié par l'agent** — il n'a pas accès à un navigateur. Toute vérification d'affichage passe par l'utilisateur.

---

## 9. Feuille de route

| Session | Contenu | État |
|---|---|---|
| 1 | Échafaudage, bibliothèque, mode cuisine, planning lecture seule | fait |
| 2 | Sélection, liste de courses, chaînage, navigation | fait |
| **3** | **Supabase, dates réelles, journal de dépenses** | à venir |
| 4 | Placards, stock, validation du panier après courses | |
| 5 | Portions stockées, écran de bilan après cuisine | |
| 6 | Scoring, rotation, suivi du gaspillage | |

La session 3 regroupe trois choses liées : passer à Supabase règle d'un coup la synchronisation entre appareils et le stockage des photos, et débloque le journal de dépenses. Les dates réelles y sont parce qu'elles conditionnent l'historique et l'état « faite ».

**Le journal de dépenses est volontairement remonté** en session 3 alors qu'il appartient logiquement au module stock. Sa valeur vient de l'accumulation dans le temps : plus il démarre tôt, plus la comparaison avec les 204,82 € d'Uber Eats sera parlante. Photo du ticket, magasin, montant total, saisis à la main. **Pas d'OCR** — décision prise, les libellés de tickets sont un projet en soi.

**L'ordre des dépendances**, plus important que les numéros : Supabase avant les photos, les dates avant l'historique, les placards avant les portions stockées, et l'usage réel avant le scoring.

---

## 10. Méthode de travail

Une seule session agentique à la fois sur un même dossier. Deux incidents d'écriture concurrente se sont déjà produits, produisant des fichiers en double et des clés YAML dupliquées.

Commiter avant et après chaque session. Un commit par livrable ou par bug.

Exiger le plan avant le code, et surveiller la dérive de périmètre — un agent propose spontanément des modules hors sujet.

Le poste de travail est sous Windows, avec le Bureau synchronisé par OneDrive. Cela a déjà cassé `fs.cpSync` par un `EPERM` sur `chmod`. Éviter les opérations touchant aux permissions ; déplacer les dépôts hors du Bureau serait préférable.

---

## 11. La prochaine chose à faire

Vérifier visuellement le résultat de la session 2 : affecter une recette et recharger pour contrôler la persistance, ouvrir l'onglet Courses, tester le chaînage en affectant une pizza pepperoni sans pâton disponible.

Puis récupérer les trois recettes de recharge express créées après la session 2, qui comblent un créneau qui était à zéro — le mardi 20h15 entre l'encadrement et l'entraînement personnel n'avait aucune proposition.

Ensuite, session 3.
