# Checklist de clôture de la phase 2

À dérouler à la main. **Aucun de ces écrans n'a jamais été observé en
fonctionnement** — les sessions 2ter et 2quater ont livré du code vérifié par
script, jamais dans un navigateur. C'est le dernier livrable manquant avant la
session 3.

Compte 20 à 30 minutes. Note ce qui casse, ne t'arrête pas au premier écart.

---

## Préparation

```
git push
npm run build
npm run preview
```

Le build doit passer. S'il échoue, arrête-toi là.

**Puis purger le service worker** — DevTools → Application → Service Workers →
*Unregister*, puis Storage → *Clear site data*, puis rechargement forcé. Ou
plus simple : ouvrir en **navigation privée**.

Sans ça, le service worker ressert un build antérieur et **tous les résultats
sont faux**. C'est arrivé le 31/07/2026 : cinq tests déclarés en échec alors
que le code était correct. Voir `PIEGES-ENVIRONNEMENT.md` §1.

- [ ] **Test 0 — le repère.** Planning, semaine de cours, **mardi**. Regarder
      le sélecteur des sept jours et le compteur sous le nom du jour.
      → **4 points** sous le M, et un ratio **sur 4** (« 0/4 repas planifiés »).

      **5 points ou un ratio sur 3 = tu regardes un vieux build.** Purge à
      nouveau et ne déroule rien tant que ce test ne passe pas : le reste
      n'aurait aucune valeur.

---

## A — Les créneaux du mardi

Le bloc le plus important : il valide le correctif qui a occupé toute la
session 2quater. Ouvre **Planning**, sélectionne **mardi**.

- [ ] **A1 — `collation-trajet` à 17h45.** Ouvrir le créneau de 17h45.
      → Il s'appelle **« Collation trajet »**, et **`Tartines de collation pour le trajet`** figure dans les propositions.
      *C'est le test qui compte : cette recette était invisible depuis la session 1.*

- [ ] **A2 — `recharge-express` à 20h15.** Ouvrir le créneau suivant.
      → Il s'appelle **« Recharge express »**, il est à **20h15** et non 19h15, et propose les recettes de recharge (abricots-compote, dattes-amandes, boisson de récupération).

- [ ] **A3 — lundi reste une collation.** Aller sur **lundi**, ouvrir le créneau de **18h45**.
      → Il s'appelle **« Collation trajet »**, pas « Recharge express ».
      *Confirme que la règle des efforts enchaînés ne déborde pas sur tous les entraînements.*

- [ ] **A4 — mercredi est le jour de batch.** Aller sur **mercredi**, regarder le créneau du soir (19h).
      → Il s'appelle **« Batch cooking »**, en semaine de cours **comme** en semaine d'alternance.

- [ ] **A5 — les points du sélecteur.** Sur mardi, compter les points sous l'initiale du jour dans le bandeau des sept jours.
      → Leur nombre est **égal au dénominateur** du « N/M repas planifiés » affiché sous le nom du jour.

---

## B — Le sélecteur de portions

Ouvrir **Recettes**, puis une fiche.

- [ ] **B1 — jusqu'à 10.** Ouvrir `Tortilla petits pois et comté` (`portions_max_reel` = 4). Appuyer sur `+` sans s'arrêter.
      → Le sélecteur monte jusqu'à **10**, jamais bloqué à 4.

- [ ] **B2 — l'avertissement au franchissement.** Sur la même recette, passer de 4 à 5.
      → Un bandeau apparaît : « Une poêle de 24 cm tient 4 œufs. Au-delà, passer au plat à four. » Redescendre à 4 → il disparaît.

- [ ] **B3 — le repli générique.** Ouvrir `Bol yaourt banane noix` (`note_scaling` vide, `portions_max_reel` = 4), dépasser 4.
      → Un message **générique** apparaît. Ni bandeau vide, ni plantage.

---

## C — L'écran de bilan

- [ ] **C1 — l'enchaînement.** Lancer le mode cuisine sur `Tortilla petits pois et comté`, aller au bout, terminer. Choisir un verdict.
      → L'écran enchaîne sur **« Il t'en reste ? »**, il ne revient pas à la fiche.

- [ ] **C2 — la congélation est proposée.** Répondre **2**.
      → Un choix **Frigo / Congélateur** apparaît. (`qualite_j2: excellente`)

- [ ] **C3 — le cas « se conserve mal ».** Refaire l'opération sur `Tartines saumon fumé, fromage blanc citron` (`qualite_j2: a-eviter`).
      → Un avertissement **« Ce plat se conserve mal »** s'affiche, et répondre ≥ 1 **ne propose pas** le congélateur — frigo seul, DLC à 1 jour.

- [ ] **C4 — « Passer ».** Sur l'écran « Il t'en reste ? », toucher **Passer**.
      → Retour direct, **aucune** portion créée.

---

## D — Les portions stockées

Suite directe de C2 : tu as 2 portions de tortilla au frigo.

- [ ] **D1 — la section apparaît.** Planning → un créneau **petit-déjeuner** ou **déjeuner** → ouvrir la feuille de sélection.
      → Une section **« Déjà prêt au frigo »** est en tête, **avant** « À cuisiner », sur fond ambré, avec la DLC formulée en toutes lettres (« à manger avant le… »), jamais un point de couleur.

- [ ] **D2 — la consommation.** Toucher cette carte.
      → La feuille se ferme, la carte de repas passe à l'état **portion stockée** (étiquette « au frigo », nombre de portions, DLC).

- [ ] **D3 — la fusion additive.** Refaire le mode cuisine sur la **même** recette, répondre **1** à « Il t'en reste ? ». Rouvrir un créneau compatible.
      → La section affiche le **cumul**, pas 1. La première réponse n'a pas été écrasée.
      *C'est le correctif de la session 2quater : l'écrasement faisait disparaître des portions réellement au frigo.*

- [ ] **D4 — la section disparaît à zéro.** Consommer toutes les portions restantes.
      → « Déjà prêt au frigo » n'apparaît plus.

---

## E — Vercel

- [ ] **E1 — la bibliothèque.** Ouvrir l'URL Vercel après déploiement.
      → **41 recettes**.
      Si elle est vide alors que le local marche : service worker périmé. DevTools → Application → Service Workers → Unregister, puis rechargement forcé.

- [ ] **E2 — le mardi.** Refaire A1 et A2 sur Vercel.

---

## Si tout passe

La phase 2 est close. Restent alors, dans l'ordre de dépendance :

**Session 3** — Supabase, dates réelles, journal de dépenses. Les dates réelles
débloquent la suggestion automatique des portions dont la DLC approche
(`spec-bilan` §3.7), aujourd'hui impossible faute de savoir si un créneau tombe
avant ou après une DLC.

**Avant de la lancer** : `npm run sync-vault` côté app. `data/contexte/` est en
retard de quatre fichiers, et il est commité précisément pour qu'une session
sans accès au coffre puisse lire les specs.
