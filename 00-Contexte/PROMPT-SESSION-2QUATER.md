# Prompt à donner à Claude Code — session 2quater

Dernière session avant la session 3. Copier-coller le bloc ci-dessous dans
une session Claude Code ouverte sur `LazEat-app`.

---

Tu travailles sur `LazEat-app`. Lis `CLAUDE.md` et `DESIGN.md` à la racine,
puis les deux rapports de session dans `../LazEat/00-Contexte/rapports/`.

Le coffre `../LazEat` est la source de vérité des données et des specs. Tu le
lis, tu n'y écris jamais — sauf ton rapport de fin de session.

Session courte : un bug de correspondance trouvé en relisant la session
2ter, plus les deux décisions qu'elle laissait ouvertes, désormais tranchées
côté coffre.

## A — Le créneau `collation-trajet` n'est jamais généré

**Le bug.** `creneauxDepuisEvenement()` (`evenements.ts`) traduit **à la
fois** `collation` et `recharge` de la spec en `recharge-express`. Résultat :
aucun événement ne produit jamais de créneau `collation-trajet`.

Le type existe pourtant partout ailleurs — `Creneau`, `CONTRAINTES_CRENEAU`,
`CONSEIL_CRENEAU`, `MODE_SELECTION_CRENEAU`,
`CONTRAINTE_SUPPLEMENTAIRE_PAR_CRENEAU`, les préférences de reconduction — et
**huit recettes du coffre le déclarent**. Il est mort depuis la session 1.

**Effet concret.** Le mardi 17h45, entre le travail et le club, l'app propose
des abricots secs et une boisson de récupération. `tartines-collation-trajet`,
dont le texte dit « Recette du mardi. Préparée le matin, mangée à 17h30 dans
les transports », n'apparaît pas.

**Origine.** La spec §1 employait des noms abrégés — `collation`, `recharge`,
`post-entrainement` — qui n'existent pas dans l'énumération. La
correspondance a été faite à la main et deux types distincts ont fusionné.

`spec-modeles-semaine.md` §1 a été corrigée : elle porte désormais les
identifiants **exacts** de l'énumération. Reporte-les fidèlement. Les deux
types ne sont pas interchangeables — `collation-trajet` se prépare la veille
et se mange sans couverts dans les transports, `recharge-express` ne se
prépare pas et vit dans le sac de sport.

**Nouvelle règle à implémenter, §1 de la spec : les efforts enchaînés.**
Quand un `rugby-entrainement` commence moins de 30 minutes après la fin d'un
autre événement, son créneau amont n'est pas une `collation-trajet` à −75
mais une `recharge-express` à **−15**.

Sans elle, le mardi produit un créneau à 19h15 — en plein milieu de
l'encadrement des cadets, de 19h à 20h30. Un moment où l'on ne peut pas
manger. Le bon horaire est 20h15, entre l'encadrement et l'entraînement
personnel.

**Sur la demande n° 1 du rapport 2ter.** Elle proposait d'aligner la table de
la spec (20h15) sur le code (19h15). C'est l'inverse : la table a raison.
`ETAT-DU-PROJET.md` §11 confirme que `recharge-abricots-compote`,
`recharge-dattes-amandes` et `boisson-recuperation` ont été écrites pour ce
créneau de 20h15. Deux documents indépendants disent 20h15, et 19h15 est
matériellement impossible.

**Étendre `verifier-modeles-semaine.ts`** pour qu'il contrôle aussi le
**type** de chaque créneau, pas seulement leur nombre et quelques horaires.
C'est ce trou qui a laissé passer le bug : les comptes étaient justes, les
types faux. Le mardi doit produire `collation-trajet` à 17h45 **et**
`recharge-express` à 20h15.

## B — Fusion additive des portions restantes

Le rapport 2ter signale que répondre deux fois à « Il t'en reste ? » pour la
même recette **remplace** l'entrée précédente.

`spec-bilan-et-portions-restantes.md` §3.2 a été mise à jour : le modèle à un
seul lot par recette est **validé et conservé**, mais la fusion doit être
**additive** — additionner les portions, retenir la **DLC la plus proche**.

Écraser fait disparaître des portions réellement présentes au frigo, soit
l'inverse exact de ce que la fonctionnalité existe pour éviter.

## C — L'urgence de DLC se dit, elle ne se colore pas

Décision confirmée, `spec-bilan-et-portions-restantes.md` §3.8 mise à jour :
ton choix de la session 2ter était le bon. `DESIGN.md` ne définit qu'un
accent, `amber`, et interdit d'inventer une valeur ; un code vert / orange /
rouge en créerait deux de toutes pièces.

Rien à changer. C'est écrit ici pour qu'aucune session future ne rouvre le
sujet.

## D — Ce qui n'a jamais été vu à l'écran

La session 2ter a livré A4, la partie B et toute la partie C **sans qu'aucun
de ces écrans soit observé dans un navigateur** — l'extension Chrome de ton
environnement ne joint pas `localhost`, piège déjà documenté.

Ne retente pas. Prépare plutôt, à la fin de ton rapport, une **liste de
vérification numérotée** que l'utilisateur déroulera à la main, couvrant au
minimum :

- les points de `SelecteurSemaine` alignés sur le ratio N/M
- le sélecteur de portions jusqu'à 10 et le bandeau `note_scaling` au
  franchissement de `portions_max_reel`
- les trois étapes de l'écran de bilan, dont le cas `qualite_j2: a-eviter`
  (avertissement affiché, congélation masquée, DLC forcée à 1 jour)
- la section « Déjà prêt au frigo » en tête de la feuille de sélection
- l'état `portion-stockee` d'une carte de repas
- le mardi : `collation-trajet` à 17h45 avec `tartines-collation-trajet`
  proposée, `recharge-express` à 20h15 avec les recettes de recharge

Une ligne par test, avec le geste exact et le résultat attendu. C'est le
livrable qui permettra de clore la phase 2 pour de bon.

## Méthode

- Les points A et B sont cadrés, tu peux y aller. Valide un plan avec moi si
  la règle des efforts enchaînés t'oblige à retoucher la structure de
  `creneauxDepuisEvenement` plus profondément que prévu.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate` et `npm run verifier-modeles` après
  chaque commit.
- Distingue toujours ce que tu as **exercé** de ce que tu as seulement
  **relu**. C'est ce que les deux rapports précédents ont bien fait et c'est
  ce qui les rend utilisables.
- Si tu trouves un fichier modifié que tu n'as pas écrit, arrête-toi et
  signale-le.

## Rapport de session — obligatoire

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-2quater.md`, seul fichier
que tu as le droit d'écrire dans le coffre. Même structure que les deux
précédents, plus la liste de vérification du point D.

Signale-moi le chemin quand il est écrit.
