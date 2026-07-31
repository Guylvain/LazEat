# Prompt à donner à Claude Code — session 2ter

Copier-coller le bloc ci-dessous dans une session Claude Code ouverte sur
`LazEat-app`. À faire **avant** la session 3.

---

Tu travailles sur `LazEat-app`. Lis `CLAUDE.md` et `DESIGN.md` à la racine,
puis `../LazEat/00-Contexte/rapports/RAPPORT-2026-07-31-2bis.md` — le rapport
de la session précédente, qui liste ce qui reste ouvert.

La session 2bis est validée : le protocole de test a été déroulé à la main
par l'utilisateur, tout passe. Cette session ramasse ce qui s'est accumulé
pendant ce test, plus une fonctionnalité qui remonte de la phase 4.

Le coffre `../LazEat` est la source de vérité des **données et des specs**.
Tu le lis, tu n'y écris jamais — sauf ton rapport de fin de session. Une
autre session y travaille en parallèle.

## Partie A — cinq correctifs courts

**A1. Le jour de batch cooking est mal désigné.**
`premierJourBatch()` dans `evenements.ts` cherche « le premier jour
`distanciel` ou `libre` portant une `soiree-libre` ». En semaine
d'alternance ça tombe sur mercredi et c'est juste ; en **semaine de cours**,
les cinq jours de semaine sont `presentiel` par défaut, donc la règle
désigne **samedi** — le dernier soir cuisinable de la semaine.

Conséquence : le créneau de batch ne peut jamais se trouver en amont d'un
consommateur, et **le chaînage de production est structurellement impossible
en semaine de cours**. `pate-a-pizza-maison` ne peut alimenter aucune
garniture, `riz-cuit-en-quantite` aucun plat de riz. Constaté en test.

`courses.ts` n'est pas en cause : il refuse à juste titre d'insérer une
recette productrice après son consommateur.

`spec-modeles-semaine.md` a été corrigée côté coffre : le jour de batch est
désormais **désigné explicitement** par `batch: true` sur l'événement
`soiree-libre`, mercredi dans les deux modèles, indépendamment du mode.
Porter ça dans `modeles-semaine.ts` et `evenements.ts`.

**A2. `verifier-modeles-semaine.ts` ne teste que la semaine d'alternance.**
C'est ce qui a laissé passer A1 pendant deux sessions. L'étendre à
`MODELE_COURS`. `spec-modeles-semaine.md` §3 porte désormais une table de
conformité pour ce modèle — **attention, ses horaires sont déduits des
règles, pas vérifiés par exécution**. Si l'implémentation diverge, vérifie
laquelle a raison avant de trancher, et signale-le dans ton rapport pour
que la spec soit corrigée côté coffre.

**A3. Le conseil du créneau petit-déjeuner est faux.**
`CONSEIL_CRENEAU['petit-dej']` vaut `'Assemblage, 5 min max.'` alors que
`CONTRAINTES_CRENEAU['petit-dej'].budgetTempsActif` vaut 15. Le créneau
compte maintenant six recettes, dont trois entre 10 et 15 minutes — celles
des matins de mercredi et vendredi au retour de la salle. Proposition :
`'Express ou posé, 15 min max.'`

Vérifie au passage que `tortilla-petits-pois-comte` (15 min pile) apparaît
bien sur ce créneau : si elle manque, la comparaison de `matching.ts` est
stricte au lieu d'être `<=`.

**A4. Deux compteurs divergent.** Le ratio « N/M repas planifiés »
d'`EnTeteJour` exclut désormais les créneaux `fixe`, mais les points de
`SelecteurSemaine` les comptent toujours. Aligner.

**A5. L'échelle de note est tranchée.** `_gabarit-recette.md` et
`contexte-projet-meal-planner.md` ont été corrigés côté coffre : c'est
**1 à 4**, pas 1 à 5. Raison : les transitions de statut de
`spec-bilan-et-portions-restantes.md` §2.3 sont écrites « note ≥ 3 →
validee » et « note ≤ 2 → reste a-tester », ce qui ne partage proprement
qu'une échelle paire ; sur 1 à 5 le point milieu validerait une recette
jugée quelconque.

Aligner `RecetteFrontmatterSchema.note` sur 1-4 et fusionner le type
`Verdict` avec ce champ — ils décrivent la même chose.

## Partie B — le sélecteur de portions

`RecetteDetail` passe `max={recette.portions_max_reel}`, ce qui bloque le
sélecteur à 2, 3 ou 4 selon la recette.

**Le sélecteur doit monter à 10 pour toutes les recettes.**
`portions_max_reel` devient un **seuil d'avertissement**, plus un plafond.

Ne pas réécrire `portions_max_reel` à 10 dans les données : ce champ veut
dire « limite matérielle, taille de poêle ou de plat ». Une tortilla pour 10
ne tient pas dans une poêle de 24 cm, deux faux-filets sont deux pièces.
Le mettre à 10 partout transformerait une information vraie en mensonge et
supprimerait le seul endroit qui sait qu'il faut une deuxième poêle.

Au franchissement du seuil, afficher un avertissement. Le texte existe déjà
sur beaucoup de recettes : c'est `note_scaling`, le champ prévu pour ça
(« une poêle de 24 cm tient 4 œufs, au-delà passer au plat à four »).
Prévoir un message générique quand `note_scaling` est `null`.

L'utilisateur n'est jamais bloqué. 10 est cohérent avec sa contrainte
physique réelle : le congélateur tient environ 12 boîtes.

## Partie C — portions restantes

Remontée de la phase 4. Lis `spec-bilan-et-portions-restantes.md` §3 en
entier avant de commencer.

L'écran de bilan livré en 2bis pose le verdict seul. Il doit aussi poser
**« Il t'en reste ? »** avec 0 / 1 / 2 / 3, et créer l'entité correspondante.

### Ce qui est dans le périmètre

- La question, après le verdict, optionnelle comme tout le reste de l'écran
  sauf le verdict lui-même (§1 : quinze secondes maximum, aucun texte libre
  obligatoire — au-delà l'écran sera abandonné en trois semaines)
- L'entité `PortionStockee` persistée, sur le modèle de `Bilans`
- **La DLC est calculée, jamais saisie** (§3.3) :
  `date_production + duree_conservation_frigo`, présent sur chaque recette.
  Congélateur : 90 jours. L'utilisateur ne tape jamais de date
- Le comportement selon `qualite_j2` (§3.4) : `excellente` → question
  normale, congélation proposée ; `correcte` → DLC 2 jours ; `a-eviter` →
  avertissement « ce plat se conserve mal », DLC forcée à 1 jour,
  congélation masquée
- Une section **« Déjà prêt au frigo »** en tête de la feuille de sélection,
  avant les recettes à cuisiner. Nom, portions restantes, jours avant DLC,
  emplacement. Code couleur DLC (§3.8) : vert au-delà de 2 jours, orange à
  1 jour, rouge le jour même
- Choisir une portion la consomme et la décrémente

C'est cette section qui porte toute la valeur. À 22h30 en rentrant
d'entraînement, lire « il te reste 1 portion de chili, à manger avant
vendredi » vaut mieux que chercher une recette. Sans elle, la question du
bilan fait saisir dans le vide, ce qui est pire que de ne rien demander.

### Ce qui n'est PAS dans le périmètre, et pourquoi

- **La suggestion automatique par défaut** (§3.7 : une portion dont la DLC
  arrive dans deux jours proposée avant même l'écran de matching). Les
  créneaux du planning n'ont **aucune date réelle** : l'app ne peut pas
  savoir si un créneau tombe avant ou après une DLC. C'est la décision de
  la session 3. La section « Déjà prêt » suffit d'ici là
- **La contrainte des douze boîtes** (§3.5) : demande une notion de
  contenants qui n'existe pas
- **Le suivi du gaspillage** (§4) et la remontée au niveau ingrédient
  (§2.4) : phase 5
- **Les raisons de rejet structurées** (§2.1) : phase 3

### Point d'architecture à me signaler AVANT de coder

Une portion stockée d'une recette `type_production: composant` et un
ingrédient `provenance: produit` décrivent **la même chose** : « j'ai un
pâton au congélateur ». Il y a deux façons de le représenter, et le
chaînage de production (`courses.ts`, `BesoinNonCouvert`) s'appuie
aujourd'hui sur la seconde en supposant le stock toujours vide.

Si tu crées des portions stockées de `pate-a-pizza-maison` sans les relier
au chaînage, l'app proposera d'insérer une session de pâte alors qu'un pâton
existe. Si tu les relies, tu touches à un module qui vient d'être vérifié.

**Ne tranche pas seul.** Expose-moi les deux options et leur coût, et
attends ma réponse. C'est exactement le genre de choix qui se paye par une
réécriture en session 4.

## Méthode

- Les points A1 à A5 et B sont cadrés : tu peux y aller. **La partie C
  demande un plan validé avec moi avant tout code**, à cause du point
  d'architecture ci-dessus.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build` et `npm run validate` après chaque commit ;
  `npm run verifier-modeles` obligatoire, A1 et A2 y touchent directement.
- Ne me dis jamais qu'un correctif fonctionne sans l'avoir exercé.
  Distingue systématiquement ce que tu as **exercé** de ce que tu as
  seulement **relu** — c'est ce que la session 2bis a bien fait et c'est ce
  qui a rendu son rapport utile.
- Pose un `.gitattributes` s'il n'y en a pas : les fins de ligne CRLF ont
  pollué les diffs de la session précédente.
- Si tu trouves un fichier modifié que tu n'as pas écrit, arrête-toi et
  signale-le.

## Rapport de session — obligatoire

À la fin, écris un rapport dans
`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-2ter.md`. **C'est le
seul fichier que tu as le droit d'écrire dans le coffre.**

Même structure que le rapport de la session 2bis, qui est le modèle à
suivre : ce qui a été fait · ce qui a été vérifié et comment · ce qui a
échoué ou reste non vérifié · décisions prises et leur raison · pièges
découverts · demandes vers le coffre · ce qui reste ouvert.

Écris-le pour quelqu'un sans aucun contexte. Si un fait est déductible en
lisant le fichier concerné, ne le répète pas.

Signale-moi le chemin du rapport quand il est écrit.
