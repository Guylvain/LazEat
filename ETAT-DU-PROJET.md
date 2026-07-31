# LazEat — État du projet

Document de reprise. À fournir en début de toute nouvelle conversation pour repartir sans rejouer l'historique.

Dernière mise à jour : fin de session 3a, 31 juillet 2026.

---

## 1. À qui ça sert et pourquoi

Liam, 25 ans, master Data Engineering & IA en alternance à l'EFREI, joueur et éducateur de rugby en club, basé en Île-de-France.

**Le problème.** Il s'entraîne ou encadre cinq soirs par semaine et saute des repas — certains jours un seul, souvent livré. Le déficit énergétique qui en résulte explique une fatigue chronique.

La cause n'est ni le manque de connaissances ni la motivation, mais une **friction de décision** : à 22h30 en rentrant d'entraînement, « qu'est-ce que je mange » n'a pas de réponse préparée, donc la réponse par défaut est la livraison. 204,82 € sur Uber Eats en juillet 2026.

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

Tout le reste du bilan était normal. La CRP est passée de 56,80 à 1,40 en deux ans.

**Le juge de paix du projet** : refaire doser folates et triglycérides vers novembre 2026. Si la B9 repasse au-dessus de 3,89 et les triglycérides sous 1,20, le système a fonctionné — quelle que soit l'élégance du code.

Point non résolu : la fatigue n'est pas expliquée par le bilan. Vitamine D et ferritine méritent un dosage si elle persiste.

---

## 3. Contraintes réelles

### Emploi du temps

**Semaine de cours** — école 9h10 à 17h10, du lundi au vendredi. Présentiel intégral par défaut ; les jours de distanciel ne sont pas encore fixés.

**Semaine d'alternance** — entreprise lundi, mardi, jeudi de 9h30 à 17h30, une heure de trajet dans chaque sens. Télétravail mercredi et vendredi, salle de sport de 7h20 à 9h30 ces deux matins.

**Rugby, identique dans les deux cas** : entraînement lundi 20h-22h, encadrement des cadets mardi 19h-20h30 puis entraînement personnel 20h30-22h, encadrement jeudi 19h-20h30, entraînement vendredi 20h-22h, match dimanche 13h-19h certaines semaines.

**Le mardi est le jour critique** : quatorze heures séparent le déjeuner du repas suivant sans collation planifiée. **Le vendredi est le plus chargé** : salle le matin, rugby le soir.

### Goûts

**Exclus** — épinards, brocoli, choux de Bruxelles, avocat, miel.
**Conditionnel** — œufs acceptés uniquement intégrés à une préparation, jamais seuls.
**À tester** — sardines et maquereaux, jamais goûtés.
**Substitutions actées** — chou-fleur pour brocoli et choux de Bruxelles ; petits pois, mâche et agrumes pour les épinards ; saumon et noix si les sardines échouent.

### Matériel

Four, micro-ondes, quatre plaques, mixeur, bouilloire, grille-pain, poêles, casseroles, plats à four. Congélateur d'**environ 12 boîtes** — c'est la vraie limite du batch cooking, pas le temps.

Manquent encore : une balance de cuisine et des boîtes supplémentaires.

---

## 4. Où en est le projet

### Deux dépôts GitHub privés

**`LazEat`** — coffre Obsidian, source de vérité des données et des specs.

```
00-Contexte/    contexte, specs, chartes, maquettes, prompts de session
  rapports/     un rapport par session Claude Code sur l'app
01-Recettes/    41 recettes + _gabarit-recette.md + inventaire
02-Ingredients/ referentiel-ingredients.yaml (165 entrées)
03-Semaines/    vide
04-Suivi/       vide
```

**`LazEat-app`** — PWA React, déployée sur Vercel. `data/` est une copie du coffre, via `npm run sync-vault`.

### Stack

Vite · React · TypeScript · Tailwind v4 · vite-plugin-pwa · Zod · gray-matter · persistance localStorage derrière une interface asynchrone · Vercel Hobby.

### Sessions livrées

| Session | Contenu |
|---|---|
| 1 | Échafaudage PWA, types Zod, parseur markdown, bibliothèque, mode cuisine, planning en lecture seule |
| 2 | Persistance, feuille de sélection, planning modifiable, liste de courses agrégée, chaînage, barre d'onglets |
| **2bis** | Remise en état : bibliothèque vide au build, réactivité inter-écrans, feuille recouverte par la barre |
| **2ter** | Jour de batch explicite, sélecteur à 10 portions, **portions restantes** et section « Déjà prêt au frigo » |
| **2quater** | `collation-trajet` enfin généré, règle des efforts enchaînés, fusion additive des portions |
| **3a** | **Dates réelles**, chaîne créneau → cuisine → bilan → créneau, Planning s'ouvre sur aujourd'hui |

**Les sessions 1 et 2 n'avaient jamais été validées.** Le test manuel du 31/07/2026 a révélé qu'aucune recette n'avait jamais été chargée dans le build de production : tout ce qui dépendait d'une recette affectée — liste de courses, chaînage, feuille de sélection — n'avait jamais été exercé. Trois sessions de remise en état ont été nécessaires.

### État de vérification

**La phase 2 est close**, protocole manuel déroulé en entier et passé. La session 3a l'est aussi, 14 tests validés à la main.

`npm run build`, `npm run validate` et `npm run verifier-modeles` passent. Ce dernier contrôle désormais **le type et l'heure** de chaque créneau sur les deux modèles, plus seulement leur nombre.

---

## 5. Les règles métier à ne jamais violer

**Une recette `type_production: composant` n'apparaît jamais dans le matching d'un créneau repas.** Elle ne sort que sur `batch` ou par chaînage. Concerne `pate-a-pizza-maison` et `riz-cuit-en-quantite`.

**Un ingrédient `provenance: produit` ne va jamais sur la liste de courses.** S'il en manque, l'app propose d'insérer la recette productrice en amont — c'est le chaînage de production.

**Les créneaux repas sont calculés depuis les événements, jamais stockés.** Seules les affectations sont persistées. **Les dates aussi se dérivent** : elles se calculent depuis l'identifiant de semaine et le jour, jamais stockées.

**Le jour de batch cooking est désigné explicitement** par `batch: true` sur un événement `soiree-libre`, jamais déduit du mode du jour. La déduction plaçait le batch le samedi en semaine de cours, rendant tout chaînage impossible.

**Toucher un créneau l'épingle.** Le recalcul ne déplace jamais ce que l'utilisateur a décidé.

**Affecter une portion la réserve, seule la validation après cuisine la consomme.** Le décrément se déclenche à la validation, jamais à la planification.

**Créer une portion stockée ne retouche pas au stock d'ingrédients** — ils ont déjà été décrémentés à la validation de la recette.

**Un sous-groupe d'ingrédients est toujours additif.** Une alternative se décrit en prose dans les notes, sinon l'agrégation achète les deux versions.

**Hiérarchie des sources** : `DESIGN.md` pour les valeurs, les maquettes pour le rendu, les briefs pour le périmètre. Quand un brief contredit `DESIGN.md` sur une valeur, le brief a tort.

---

## 6. Le système visuel

Détail dans `DESIGN.md`, rendu dans `charte-graphique-v3.html`.

**Palette** — `cream #EEF0E8` · `white #FCFCF9` · `moss #33574A` · `moss-deep #244035` · `amber #E2A33F` · `sage #7E8F84` · `sage-dark #5F6F65` · `mist #DBE0D5`

**Typographie** — Fraunces en display avec `SOFT 100`, `WONK 1` et `opsz` suivant la taille ; Figtree pour le texte. Exception unique : `WONK 0` sur le minuteur du mode cuisine.

**Trois règles structurantes** :

*Arrondi pour ce qui se touche, angle vif pour ce qui se mesure.*

*L'ambre ne porte jamais de texte sur fond clair* — 2,1:1 mesuré. Sur `moss-deep` il passe à 5,1:1.

*Un seul accent existe.* Il n'y a pas de palette d'urgence. L'urgence d'une DLC se dit en toutes lettres — « à manger demain » — jamais par une couleur inventée.

**Le texte d'une carte de repas change de nature selon l'état.** Vide, elle affiche le créneau et son conseil : c'est de l'aide à la décision. Remplie, elle affiche la recette et ses métadonnées : c'est ce qui sert en cuisine.

---

## 7. Les données

**41 recettes** au format markdown avec frontmatter validé par Zod. Sept ont été ajoutées le 31/07/2026 pour combler les deux créneaux les plus pauvres.

**Couverture par créneau** : soir-cuisine 22 · dejeuner 14 · diner-a-deux 12 · post-entrainement 10 · collation-trajet 8 · batch 7 · **petit-dej 6** · **avant-match 6** · recharge-express 3.

Le petit-déjeuner couvre désormais deux usages distincts : trois recettes express de 2 à 5 minutes pour les matins de semaine, trois de 10 à 15 minutes pour les mercredis et vendredis au retour de la salle.

**165 ingrédients**, dont 38 stubs restants. Les `conditionnement` sont des estimations à vérifier en faisant les courses une fois avec le fichier ouvert. C'est le principal travail de contenu restant.

**L'échelle de note est de 1 à 4**, pas 1 à 5. Échelle paire : les transitions de statut sont écrites « ≥ 3 → validée » et « ≤ 2 → reste à tester », ce qui ne partage proprement qu'un nombre pair de valeurs.

**Le frontmatter n'est qu'une valeur initiale.** La PWA ne peut pas écrire dans le coffre : dès qu'une recette a été validée une fois, la persistance fait foi devant le markdown pour `note`, `nb_executions` et `derniere_execution`.

---

## 8. Ce qui reste ouvert

**La sur-réservation d'une portion stockée.** `portionsCompatiblesCreneau` ne filtre que sur `portionsRestantes > 0` : elle ne tient pas compte des portions déjà réservées sur un autre créneau. La même portion peut donc être proposée deux fois. Le compte reste exact, mais la section « Déjà prêt au frigo » ment sur ce qui est disponible. **Correctif en tête de la session 3b.**

**La synchronisation entre appareils.** localStorage garde les données sur l'appareil : planifier sur l'ordinateur puis faire ses courses avec le téléphone ne fonctionne pas. En attendant Supabase, tout se fait depuis le téléphone. L'interface de persistance est asynchrone précisément pour que la bascule soit une substitution, pas une réécriture.

**La suggestion automatique d'une portion dont la DLC approche.** Devenue techniquement possible avec les dates réelles, mais l'interface reste à décider : où la proposer, comment l'écarter.

**Les conditionnements du référentiel** restent des estimations.

**Le service worker en développement.** `devOptions.enabled: true` fait tourner le service worker en dev, où il ressert des builds périmés. Il a déjà invalidé une campagne de tests entière. Candidat à passer à `false`.

**Le rendu visuel n'est jamais vérifié par l'agent** — il n'a pas accès à un navigateur. Toute vérification d'affichage passe par une liste de contrôle manuelle, que chaque session doit livrer.

---

## 9. Feuille de route

| Session | Contenu | État |
|---|---|---|
| 1 | Échafaudage, bibliothèque, mode cuisine, planning lecture seule | fait |
| 2 | Sélection, liste de courses, chaînage, navigation | fait |
| 2bis · 2ter · 2quater | Remise en état, portions restantes, créneaux corrigés | fait |
| 3a | Dates réelles, chaîne d'exécution, repas de ce soir | fait |
| **3b** | **Sur-réservation, Supabase, journal de dépenses** | à venir |
| 4 | Placards, stock, validation du panier après courses | |
| 5 | Scoring, rotation, suivi du gaspillage | |

La session 3 a été coupée en deux : la 3a changeait le modèle de données (dates, réservation contre consommation), et poser un schéma Supabase par-dessus un modèle en cours de modification aurait obligé à le migrer deux fois.

**Le journal de dépenses est volontairement remonté** en 3b alors qu'il appartient logiquement au module stock. Sa valeur vient de l'accumulation : plus il démarre tôt, plus la comparaison avec les 204,82 € d'Uber Eats sera parlante. Photo du ticket, magasin, montant, saisis à la main. **Pas d'OCR** — décision prise.

**L'ordre des dépendances**, plus important que les numéros : Supabase avant les photos, les placards avant le décrément d'ingrédients, et l'usage réel avant le scoring.

---

## 10. Méthode de travail

**Lire `00-Contexte/PIEGES-ENVIRONNEMENT.md` avant toute session sur l'app.** Neuf pièges, chacun ayant déjà coûté du temps : le service worker qui ressert un build périmé, `npm run dev` qui ne prouve rien sur le chargement des données, les angles morts des trois commandes de vérification, les fins de ligne CRLF, l'impossibilité pour un agent d'ouvrir un navigateur, et le fait que commiter n'est pas pousser.

Une seule session agentique à la fois par dossier. Deux incidents d'écriture concurrente ont déjà eu lieu.

Chaque session sur l'app écrit un rapport dans `00-Contexte/rapports/`, seul fichier qu'elle a le droit d'écrire dans le coffre. Structure : ce qui a été fait · ce qui a été vérifié **et comment** · ce qui reste non vérifié · décisions et leur raison · pièges découverts · demandes vers le coffre · liste de vérification manuelle · preuve du push.

La distinction entre ce qui a été **exercé** et ce qui a été **relu** est ce qui rend ces rapports utilisables. L'exiger explicitement.

Exiger le plan avant le code sur tout ce qui touche à l'architecture, et surveiller la dérive de périmètre.

Poste sous Windows, Bureau synchronisé par OneDrive. Éviter les opérations touchant aux permissions ; déplacer les dépôts hors du Bureau serait préférable.

---

## 11. La prochaine chose à faire

Vivre une vraie semaine avec l'app. Planifier un dimanche, faire les courses avec la liste, cuisiner, valider les recettes, déclarer les restes.

C'est la seule chose qui manque désormais, et rien ne peut la remplacer : le scoring de la phase 5 a besoin de données de goût réelles, la rotation a besoin d'un historique, et les conditionnements du référentiel ne se vérifieront qu'en magasin.

Puis la session 3b.
