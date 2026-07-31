# Journal technique — sessions 1 et 2

Document de référence, pas un journal chronologique. Objectif : revenir en
arrière si quelque chose casse, et ne pas re-trancher une décision déjà
prise faute de savoir qu'elle l'a été. Écrit pour un agent sans contexte —
si un fait est déductible en lisant le fichier concerné, il n'est pas
répété ici ; seul ce qui ne l'est pas y figure.

---

## 1. Inventaire du code

### `src/schemas/` — le socle

- **`recette.schema.ts`** — schémas Zod du frontmatter et du corps d'une
  recette (`RecetteFrontmatterSchema`, `LigneIngredientSchema`,
  `GroupeIngredientsSchema`, `EtapeSchema`, `RecetteSchema`). Toutes les
  unions littérales (`Creneau`, `Registre`, `TypeProduction`…) sont
  définies ici en `z.enum`. **Critique** : `_gabarit-recette.md` du coffre
  fait foi sur sa forme — n'importe quel champ ajouté aux recettes sans
  être ajouté ici sera silencieusement ignoré (Zod non strict, voir §2).
- **`ingredient.schema.ts`** — schéma du référentiel d'ingrédients :
  `IngredientStubSchema` (id + nom + `a_completer: true`) et
  `IngredientCompletSchema`, unis par `z.union` (pas
  `z.discriminatedUnion`, voir §2). **Critique** : toute lecture
  d'ingrédient dans l'app passe par `estStub()` avant d'accéder aux champs
  qui n'existent que sur la variante complète.

### `src/types/` — dérivés des schémas, jamais écrits à la main

- **`recette.ts`, `ingredient.ts`** — ré-exportent les types Zod
  (`z.infer`) et le type `Recette` complet.
- **`planning.ts`** — types *non* dérivés d'un gabarit (pas de fichier
  markdown source) : `JourSemaine`, `ModeJour`, `TypeEvenement`,
  `Evenement`, `CreneauCalcule`, `ElementPlanning<C>` (générique, voir
  §6), `ModeleSemaine`. **Critique** : `CreneauCalcule` est la forme que
  manipulent `evenements.ts`, `semaine.ts`, `courses.ts` et tous les
  composants de planning — un champ ajouté ici doit être peuplé partout où
  un `CreneauCalcule` est construit (un seul endroit : `genererCreneauxJour`).
- **`semaine.ts`** — `AffectationCreneau`, `ChoixCreneau`,
  `SemainePlanifiee` (l'état réellement persisté).
- **`courses.ts`** — `EtatArticle`, `ListeCourses` (persisté — uniquement
  l'état coché, jamais les quantités, voir §2).

### `src/lib/markdown/` — parseur, aucune dépendance à Vite ni à React

- **`frontmatter.ts`** — sépare le YAML du corps via `gray-matter`, ne
  valide rien (`parseFrontmatterEtCorps`).
- **`sections.ts`** — découpe le corps en sections `## Titre`
  (`decouperSectionsH2`), retrouve une section par préfixe de titre (les
  titres varient : "Ingrédients" vs "Ingrédients — environ 40 pièces").
- **`ingredientsTable.ts`** — parse les tables markdown d'ingrédients en
  `GroupeIngredients[]`, gère les sous-groupes `### Titre` et l'annotation
  "recette séparée" en nettoyant les backticks des cellules.
- **`etapes.ts`** — parse `### N. Titre` + `` `minuteur: N` `` optionnel en
  `Etape[]`.
- **`recetteMarkdown.ts`** — orchestrateur (`parseCorpsRecette`), échoue
  bruyamment si "Ingrédients" ou "Étapes" est absent ou vide.

### `src/lib/` — logique métier

- **`recettes.ts`** — charge les 31 recettes via
  `import.meta.glob(..., {eager:true, query:'?raw'})`, valide chacune avec
  Zod, met en cache. **Critique et piège** : cet import est spécifique à
  Vite ; ce module ne peut pas être importé par un script `tsx` nu hors
  du serveur Vite (voir §6).
- **`ingredients.ts`** — même mécanique pour le référentiel YAML
  (`?raw` + `yaml.parse`). Même piège.
- **`scaling.ts`** — `scalerQuantite`, `formaterQuantite` (n'affiche
  jamais le mot "piece", voir §3 bug session 1), `interpolerInstruction`
  (remplace `` `{ingredient_id}` `` par la quantité mise à l'échelle **et**
  le nom pluralisé/élidé de l'ingrédient).
- **`temps.ts`, `heure.ts`** — formatage de durée et arithmétique sur des
  chaînes `"HH:mm"` (`versMinutes`, `versHeureTexte`, `ajouterMinutes`).
  Utilisées partout où une heure est calculée.
- **`creneaux.ts`** — `CONTRAINTES_CRENEAU` (budget de temps actif +
  contraintes transportable/bon-froid par créneau), `NOM_CRENEAU`,
  `CONSEIL_CRENEAU`, `MODE_SELECTION_CRENEAU`, `NOM_JOUR` (minuscules,
  distinct de celui d'`EnTeteJour.tsx`, voir §6). **Critique** : la table
  de budgets pilote à la fois `CreneauCalcule.budgetTempsActif` (généré
  par `evenements.ts`) et le filtre de `matching.ts` — la modifier change
  ce qui apparaît dans le planning ET dans la feuille de sélection.
- **`modeles-semaine.ts`** — `MODELE_COURS`, `MODELE_ALTERNANCE` :
  transcription littérale de `spec-modeles-semaine.md`, `JOURS_SEMAINE`.
  **Critique** : source unique des événements de la semaine ; toute
  modification change le nombre de créneaux générés partout (planning,
  courses, badge de la barre d'onglets, `verifier-modeles-semaine.ts`).
- **`evenements.ts`** — `genererCreneauxJour` (pur, un jour), 
  `genererCreneauxSemaine` (les sept jours, décide du jour de batch
  cooking, voir §2), `evenementsAffiches`, `bandeDe`, `regrouperParBande`.
  **Le plus critique du dépôt** : toute la génération de créneaux passe
  par ces deux fonctions. Ne jamais dupliquer une boucle sur les sept
  jours ailleurs (voir §6).
- **`matching.ts`** — `recettesPourCreneau` : les filtres durs de la
  feuille de sélection (créneau compatible, budget de temps, jamais un
  composant sauf pour `batch`, statut non rejeté, contrainte booléenne
  supplémentaire par créneau), triés par temps actif croissant.
- **`filtresRecettes.ts`** — filtres de la bibliothèque (recherche,
  créneau, registre, cuisine à deux, temps actif max) — indépendant de
  `matching.ts`, pas de scoring dans les deux cas.
- **`semaine.ts`** — pont entre le pur (`CreneauCalcule`) et le persisté
  (`SemainePlanifiee`) : `chargerSemaine`, `idSemaineCourante`,
  `affecterRecette`, `affecterMangeDehors`, `viderCreneau`,
  `definirSurchargeMode` (transitions d'état pures, immutables),
  `appliquerAffectations` (superpose l'état réel sur des créneaux purs) et
  le type `CreneauAffecte`. **Critique** : `CreneauAffecte.id` est la clé
  de persistance des affectations (voir §2 et §6).
- **`courses.ts`** — `calculerListeCourses` : agrégation multi-recettes,
  conversion d'unité, exclusion des `provenance: produit`, arrondi au
  conditionnement, groupage par rayon, détection du chaînage de
  production (`BesoinNonCouvert`), `compterArticlesRestants`. **Critique
  et dense** : la fonction la plus longue du dépôt, plusieurs règles
  imbriquées (voir §2).
- **`persistance.ts`** — interface `Persistance` (4 méthodes async),
  implémentation `PersistanceLocalStorage`, singleton `persistance`, et
  un pub-sub (`ecouterChangementsPersistance`) pour la réactivité
  inter-écrans (badge de la barre d'onglets). **Critique** : seul point
  d'entrée vers `localStorage` — aucun composant ne doit y accéder
  directement.
- **`modeleActif.tsx`** — `ModeleActifProvider`/`useModeleActif`, un
  `Context` React pour partager `modeleId` entre Planning, Courses et la
  barre d'onglets.

### `src/components/planning/`

- **`SelecteurSemaine.tsx`** — 7 pastilles jour, points par créneau
  (ambre si affecté).
- **`EnTeteJour.tsx`** — nom du jour (a sa propre `NOM_JOUR` capitalisée,
  voir §6), mode, compteur "N/M repas planifiés".
- **`ToggleModeJour.tsx`** — `<select>` présentiel/distanciel/libre/absent.
- **`BandeCreneaux.tsx`** — une bande (matin/après-midi/soir), route vers
  `CarteRepas` ou `LigneEvenement`.
- **`LigneEvenement.tsx`** — ligne d'événement non-repas ; ses `Record`
  couleur/libellé doivent lister *toutes* les valeurs de `TypeEvenement`
  (erreur TS sinon).
- **`CarteRepas.tsx`** — les 5 états de `DESIGN.md` (4 atteignables :
  vide/planifiée/mange-dehors/faite jamais déclenché) plus le gradient
  pilier.
- **`CarteOption.tsx`** — carte enrichie dans la feuille (vignette,
  ligne d'ingrédients tronquée, étiquettes, historique).
- **`FeuilleSelection.tsx`** — panneau bas modal, sections "À cuisiner" /
  "Autre".

### `src/components/courses/`

- **`LigneArticle.tsx`** — checklist à 3 états, cycle au tap.
- **`BandeauChainage.tsx`** — deux variantes strictes (créneau libre trouvé
  → bouton Accepter ; sinon → message à trois volets, jamais de bouton
  "affecter quand même").

### `src/components/navigation/` et `src/components/layout/`

- **`BarreOnglets.tsx`** — barre fixe bas, 3 onglets, pastille Courses
  réactive via `ecouterChangementsPersistance`.
- **`AppShell.tsx`** — **le plus critique de la navigation**. Garde
  Planning/Recettes/Courses montés en permanence (`display:none`, pas de
  démontage) pour la mémoire de filtres/scroll ; route le détail de
  recette et le mode cuisine via un vrai `<Outlet/>`. Voir §2 et §6.

### `src/components/recettes/`

- **`RecetteCard.tsx`, `RecetteFiltres.tsx`, `PortionSelector.tsx`** —
  bibliothèque et sélecteur de portions, sans état partagé.
- **`RecetteDetail.tsx`** — détail complet, bouton retour `navigate(-1)`
  (gère bibliothèque et planning comme origine sans distinction).
- **`cuisine/ModeCuisine.tsx`** — orchestrateur du mode cuisine (une étape
  à la fois), interpole l'instruction avec accentuation des quantités.
- **`cuisine/EtapeCuisine.tsx`, `cuisine/Minuteur.tsx`** — rendu d'une
  étape et du minuteur (Fraunces `'WONK' 0` exclusif).
- **`cuisine/useMinuteur.ts`** — minuteur à timestamp absolu (voir §2).
- **`cuisine/useWakeLock.ts`** — Wake Lock défensif (voir §2).

### `src/components/ui/` et `src/components/icons/`

- **`Button.tsx`, `Pill.tsx`, `TexteRiche.tsx`** — primitives partagées ;
  `TexteRiche` ne gère que `**gras**`/`*italique*`, pas de markdown
  complet.
- **`icons/*.tsx`** — SVG dessinés à la main, `strokeWidth` variable selon
  contexte (2 par défaut, 1,5 pour la barre d'onglets).

### `src/pages/` et racine

- **`PlanningPage.tsx`** — orchestrateur du planning : charge/persiste
  `SemainePlanifiee`, calcule les 7 jours via `genererCreneauxSemaine`,
  pilote `FeuilleSelection`.
- **`CoursesPage.tsx`** — orchestrateur de `/courses` : charge
  semaine + `ListeCourses`, calcule via `calculerListeCourses`, pilote le
  chaînage.
- **`BibliothequePage.tsx`, `RecetteDetailPage.tsx`,
  `ModeCuisinePage.tsx`** — pages simples, peu de logique propre.
- **`router.tsx`** — un seul layout route (`AppShell`) ; `/planning`,
  `/recettes`, `/courses` ont `element: null` (voir §2).
- **`main.tsx`** — importe le polyfill Buffer *avant* tout le reste,
  ordre obligatoire (voir §2).
- **`polyfills/buffer.ts`** — voir §2.
- **`styles/index.css`** — tous les tokens `@theme` (couleurs, tailles,
  rayons, ombres). **Critique et silencieux** : une classe Tailwind
  utilisant un token qui n'existe pas ne casse jamais la compilation,
  elle disparaît juste du CSS généré (voir §6).

### `scripts/`

- **`validate.ts`** (`npm run validate`) — valide `data/recettes/*.md` et
  le référentiel, résumé par défaut, `--verbose` pour le détail (voir §5).
- **`verifier-modeles-semaine.ts`** (`npm run verifier-modeles`) — garde-fou
  de non-régression sur les comptes de créneaux de la semaine d'alternance
  (voir §5).
- **`sync-vault.mjs`** (`npm run sync-vault`) — copie manuelle (pas
  `fs.cpSync`, voir §2) du coffre Obsidian vers `data/`.
- **`generate-icons.mjs`** — usage ponctuel, génère les PNG PWA depuis les
  SVG de `public/`. Pas dans le flux de `npm run build`.

---

## 2. Décisions techniques et leur raison

### Le polyfill Buffer

`gray-matter` appelle `Buffer.from()` sans condition (dans son moteur
YAML interne), et Vite ne polyfille plus `Buffer` côté navigateur depuis
sa v3. Sans polyfill, toute lecture de recette plante au premier appel de
`gray-matter` — pas une erreur au build, une exception à l'exécution.
`src/polyfills/buffer.ts` pose `globalThis.Buffer` depuis le paquet
`buffer` (le shim npm, pas le module Node), et **doit être importé avant
tout le reste** dans `main.tsx` : n'importe quel module chargé avant lui
qui référence `Buffer` de façon statique lèverait avant que le polyfill
soit posé.

### `'Fraunces Variable'` et la variante `/full.css`

`@fontsource-variable/fraunces` expose plusieurs variantes ; seule
`/full.css` porte les axes `SOFT`/`WONK`/`opsz` utilisés partout dans
`DESIGN.md` (la variante par défaut n'expose que `wght`). Mais cette
variante **déclare la police sous le nom `'Fraunces Variable'`**, pas
`'Fraunces'` — c'était le bug de fonte de la session 1 (commit `482f591`) :
`--font-display` pointait vers un nom qui ne correspondait à rien de
chargé, fallback silencieux sur Georgia, aucune erreur nulle part. Si une
session future change de fournisseur de police ou de variante, vérifier
le nom réellement déclaré dans le CSS livré, pas celui du paquet npm.

### Zod non strict sur le frontmatter, union stub/complet plutôt que `discriminatedUnion`

`RecetteFrontmatterSchema` n'est pas `.strict()` : les clés inconnues
(ex. `roles_a_deux`, résiduel sur certaines recettes, remplacé par
`cuisine_a_deux` mais pas toujours nettoyé) sont ignorées plutôt que de
faire échouer la validation. La consigne "échouer bruyamment" du gabarit
porte sur un champ **manquant** ou **hors énumération**, pas sur une clé
en trop — sinon chaque note Obsidian mal nettoyée casse `npm run
validate` pour une raison sans rapport avec les données réelles.

Pour le référentiel d'ingrédients, `IngredientEntrySchema` est un
`z.union([IngredientStubSchema, IngredientCompletSchema])`, pas un
`z.discriminatedUnion('a_completer', ...)`. Raison structurelle : les
entrées complètes réelles n'ont **pas la clé `a_completer` du tout** (ni
`false` ni absente au sens Zod — absente au sens JS), or un
`discriminatedUnion` exige que le champ discriminant soit présent des
deux côtés de l'union pour pouvoir trancher. `estStub()` (un simple
`'a_completer' in ingredient`) fait ce travail à la place.

### Types dérivés par `z.infer`, jamais écrits à la main

Tous les types de domaine (`Recette`, `Creneau`, `IngredientComplet`…)
sont `z.infer<typeof Schema>`, jamais une interface parallèle. Un type
écrit à la main peut diverger silencieusement du schéma qui valide
réellement les données (quelqu'un ajoute un champ au schéma, oublie le
type ⇒ TypeScript ne voit rien, seul du JS non typé y accède). Avec
`z.infer`, le schéma est l'unique source de vérité ; le type suit
automatiquement.

### Minuteur à timestamp absolu, pas un décompte `setInterval`

`useMinuteur` calcule `Date.now() + secondes*1000` une seule fois au
démarrage et recalcule le temps restant par différence à chaque tick,
plutôt que de décrémenter un compteur à chaque intervalle. Un
`setInterval` qui décrémente perd le compte dès que l'onglet passe en
arrière-plan (throttle navigateur, voire suspension complète sur mobile) :
au retour au premier plan, le minuteur affiché est en avance sur le temps
réel. Le timestamp absolu ne peut jamais désynchroniser — il suffit de
recalculer `fin_prévue - maintenant` au réveil.

### Wake Lock défensif, réacquisition sur `visibilitychange`

L'API Wake Lock est mal supportée en PWA installée (notamment iOS), et
elle est **toujours relâchée automatiquement** par le navigateur quand
l'app passe en arrière-plan (visio, notification, verrouillage d'écran).
`useWakeLock` : (1) vérifie `'wakeLock' in navigator` avant tout appel,
(2) avale silencieusement l'erreur si `request()` échoue ou est refusé —
le mode cuisine doit rester pleinement utilisable sans l'API, (3) réécoute
`visibilitychange` pour redemander le verrou dès que l'app redevient
visible, puisque le verrou précédent a été implicitement libéré.

### `tsx` pour `validate.ts`, pourquoi un `.mjs` ne suffisait pas

`validate.ts` importe directement les modules TypeScript de `src/`
(`RecetteFrontmatterSchema`, `parseCorpsRecette`…) pour valider les
données avec **le même code que l'application exécute**, plutôt que de
réimplémenter un parseur séparé qui pourrait diverger. Un `.mjs` ne peut
pas importer un `.ts` sans étape de compilation préalable ; `tsx`
transpile à la volée sans build séparé. `sync-vault.mjs`, à l'inverse,
n'importe aucun module `src/` (juste `node:fs`/`node:path`) : un `.mjs`
brut suffit, pas besoin de `tsx`.

### Copie manuelle dans `sync-vault.mjs`, pas `fs.cpSync`

`fs.cpSync` applique un `chmod` sur chaque fichier écrit. Sous Windows,
si le dossier de destination est géré par OneDrive (ou un antivirus actif),
ce `chmod` échoue avec `EPERM` dès le premier fichier — le script plantait
entièrement, systématiquement. `readFileSync` + `writeFileSync` ne
touchent jamais aux permissions et copient tout aussi bien.

### Séparation créneaux calculés / affectations persistées

`genererCreneauxJour`/`genererCreneauxSemaine` (evenements.ts) sont
**pures** : elles ne connaissent ni `localStorage` ni aucune affectation,
elles ne produisent que la structure du jour (quels créneaux, à quelle
heure, avec quel budget) toujours à l'état `vide`. `appliquerAffectations`
(semaine.ts) superpose l'état réel par-dessus, en dehors de toute logique
de génération. Raison : le modèle de semaine (CLAUDE.md) dit explicitement
que les créneaux sont **calculés, jamais stockés** — mélanger génération et
persistance rendrait impossible de recalculer proprement le planning si le
modèle change (ex. une surcharge de mode) sans perdre ou dupliquer les
affectations existantes. `CreneauAffecte` (extension de `CreneauCalcule`)
est la forme réunifiée, construite à la volée pour l'affichage, jamais
persistée telle quelle.

### Interface de persistance asynchrone malgré une implémentation localStorage synchrone

`Persistance` (4 méthodes, toutes `Promise`) est appelée partout comme si
elle pouvait être distante, alors que `PersistanceLocalStorage` est
100% synchrone en interne. But explicite (BRIEF-SESSION-2.md) : le passage
à Supabase en session 3 doit être une substitution d'implémentation
(nouvelle classe qui implémente la même interface), pas une réécriture de
tous les appelants qui devraient sinon apprendre à gérer une latence
réseau après coup.

### Ordre de calcul du petit-déjeuner : après le déjeuner et le soir

`genererCreneauxJour` calcule le petit-déjeuner **en dernier**, après
avoir déjà résolu le déjeuner (y compris son repli si absent) et le repas
du soir. Ce n'était pas le cas dans une version intermédiaire (commit
`2347c49`) : la règle "premier événement − 60 min" plaçait le petit-déj
du dimanche de match à 11h30, **après** l'avant-match de 11h — un
non-sens (déjeuner avant le petit-déj). Corrigé (`902cf46`) en ajoutant un
second plancher, "premier créneau repas déjà généré − 150 min", dont le
minimum avec le premier terme donne l'heure retenue. Ce second terme ne
peut être calculé qu'après que les autres créneaux du jour existent déjà
— d'où l'ordre. Toucher à cet ordre sans comprendre cette dépendance
réintroduit le bug.

### Le proxy `type_production` pour les sous-groupes d'ingrédients

Trois recettes ont un second tableau d'ingrédients sous un titre `### `.
Deux sont de vrais sous-composants (sauce d'accompagnement des gyoza,
sauce blanche maison de la salade concombre-tomates-mozza — cette
dernière un temps mal gérée, voir bug §3) ; un troisième
(`pate-a-pizza-maison`, "Garniture, par pizza — recette séparée")
documente les ingrédients d'une **autre** recette pour référence, déjà
comptés quand cette autre recette est affectée. Rien dans le schéma ne
distingue formellement les deux cas ; `_gabarit-recette.md` documente
maintenant la règle en toutes lettres (une alternative se décrit en
prose dans les Notes, jamais dans un tableau ; le second tableau annoté
"recette séparée" est réservé aux `composant`). `groupesAAgreger()`
(courses.ts) s'appuie sur `type_production` comme proxy — repas : tous
les groupes, composant : seulement le principal — et ça tombe juste sur
les trois cas connus sans inventer de champ de schéma. Si un futur
composant a un vrai second groupe additif (pas une référence), ce proxy
sera insuffisant et il faudra alors un champ de schéma dédié.

### Les contraintes booléennes en filtres durs séparés du budget de temps

`contexte-projet-meal-planner.md` §4.3 donne un budget "0 min" pour
`dejeuner` et `recharge-express`. Pris au pied de la lettre comme
`temps_actif <= budget`, ce filtre excluait toute recette (aucune ne se
prépare en 0 minute). En réalité, ce "0" décrit une contrainte de
**préparation en amont** (la veille, le week-end), pas un temps de
cuisson sur place — la vraie contrainte de ces créneaux est d'être
mangeables sans effort immédiat. `CONTRAINTES_CRENEAU` porte maintenant
des budgets réels (30 min pour déjeuner, 10 pour recharge-express…), et
`matching.ts` ajoute une contrainte booléenne à part
(`CONTRAINTE_SUPPLEMENTAIRE_PAR_CRENEAU`) — `bon_froid` OU `transportable`
pour déjeuner, `transportable` seul pour collation-trajet et
recharge-express, `batchable` pour batch, `cuisine_a_deux.adapte` pour
dîner à deux. Séparer les deux évite qu'un budget de temps large et
permissif masque une vraie contrainte de fond.

### Autres décisions notables (non listées explicitement mais à connaître)

- **Id de semaine** = `` `${modeleId}:${isoWeek}` `` (ex.
  `alternance:2026-S31`). Changer de modèle dans le sélecteur revient à
  regarder une semaine différente, pas une vue alternative de la même
  semaine — cohérent avec le fait que les deux modèles décrivent des
  réalités mutuellement exclusives (on n'est jamais "en cours" et "en
  alternance" la même semaine calendaire).
- **`modeleId` partagé via `Context`** (`modeleActif.tsx`) plutôt qu'un
  `useState` local par page : la pastille de la barre d'onglets doit
  savoir quelle semaine regarder sans dépendre de quel écran est monté.
- **Pub-sub dans `persistance.ts`**, pas une méthode de l'interface
  `Persistance` : c'est une commodité de réactivité UI (le badge doit se
  mettre à jour depuis n'importe quel écran sans rechargement), pas un
  primitif de stockage — donc hors du contrat que Supabase devra
  implémenter.
- **`ListeCourses` ne persiste que l'état coché**, jamais les quantités
  ni les regroupements : ceux-ci sont toujours recalculés en direct
  depuis `SemainePlanifiee` + les recettes, pour ne jamais afficher un
  état périmé si une recette ou ses portions changent après que
  certains articles ont déjà été cochés.
- **`genererCreneauxSemaine` centralise le jour de batch cooking** :
  avant, trois appelants (PlanningPage, courses.ts,
  verifier-modeles-semaine.ts) bouclaient chacun sur
  `genererCreneauxJour` jour par jour ; rien ne garantissait qu'ils
  choisissent le même "premier jour distanciel/libre avec soiree-libre"
  pour la conversion en `batch`. Centralisé pour qu'un seul calcul fasse
  foi partout.
- **`AppShell` garde les 3 écrans d'onglet montés en permanence**
  (`display:none`, pas de démontage React) : c'est la seule façon de
  tenir "chaque onglet conserve son état" (filtres de la bibliothèque,
  position de défilement, jour sélectionné) sans réintroduire un état
  global dupliqué. Le détail de recette et le mode cuisine restent des
  routes normales via l'`Outlet` — leur état ne doit *pas* survivre à une
  navigation, contrairement aux trois onglets racine.
- **`router.tsx` déclare `/planning`, `/recettes`, `/courses` avec
  `element: null`** : ces chemins doivent exister dans l'arbre de routes
  pour que React Router les matche (et que `useLocation()` fonctionne
  dans `AppShell`), mais leur élément réel est rendu à la main par
  `AppShell`, pas par le routeur.

---

## 3. Bugs rencontrés et corrigés

Format : symptôme observé → cause racine → correctif → commit. Les bugs de
données sont inclus : ils reviennent plus facilement que les bugs de code,
parce que rien ne les empêche mécaniquement de réapparaître à la prochaine
recette ajoutée sur le même modèle.

**Police Fraunces jamais chargée** (`482f591`)
Symptôme : titre d'étape et minuteur du mode cuisine en serif générique
(Georgia) au lieu de Fraunces. Cause : `--font-display` référençait
`'Fraunces'`, mais la variante `/full.css` du paquet déclare la police
sous `'Fraunces Variable'` — aucun nom ne correspondait, fallback
silencieux, aucune erreur. Correctif : renommer le token. Voir §2 pour le
détail.

**Nom d'ingrédient perdu dans les instructions interpolées** (`24f7e73`)
Symptôme : "Mélanger avec la moitié de 300 g, 10 g…" — la quantité
s'affichait sans le nom de l'ingrédient. Cause : `interpolerInstruction`
ne remplaçait le token que par la quantité mise à l'échelle, jamais par
le nom. Correctif : ajout du nom (via `getIngredientById`), avec élision
de "de" devant voyelle/h muet, pluralisation du premier mot pour les
ingrédients comptés en `piece`, et suppression totale du mot "piece" de
l'affichage (y compris dans la liste d'ingrédients de `RecetteDetail`,
touchée par le même correctif de `formaterQuantite`).

**Modèles de semaine incomplets — mercredi à 2 créneaux au lieu de 4**
(`2347c49`, corrigé une seconde fois par `902cf46`)
Symptôme : en semaine d'alternance, mercredi (jour distanciel avec séance
de salle) n'affichait que petit-déjeuner et cuisine du soir, sans aucun
événement. Cause : `evenements.ts` n'avait jamais de branche pour les
types d'événement `salle-de-sport`/`teletravail`, et
`modeles-semaine.ts` ne les modélisait pas non plus pour mercredi/vendredi
— les deux étaient absents en même temps, chacun masquant l'autre.
Correctif : transcription complète des deux modèles depuis
`spec-modeles-semaine.md`, réécriture de la génération avec règles de
repli pour petit-déj/déjeuner/soir et déduplication. Le premier correctif
avait un défaut (petit-déj du dimanche de match à 11h30, après
l'avant-match) rattrapé par le second — voir §2, "ordre de calcul du
petit-déjeuner". `902cf46` restaure aussi le champ `pilier`, retiré par
erreur dans une itération intermédiaire du premier correctif.

**Mode cuisine visuellement décalé de la charte** (`cc15c6e`)
Symptôme : titre d'étape, minuteur, interlignage et opacité de
l'instruction ne correspondaient pas à `charte-graphique-v3.html`. Cause :
une fois la police chargée (bug précédent), les valeurs `opsz` codées en
dur (40 au lieu de 60 pour le titre, 120 au lieu de 144 pour le minuteur)
et l'interlignage/opacité génériques (`text-body-xl`, 1.6/.90) ne
matchaient pas les valeurs spécifiques au mode cuisine (1.5/.94). Correctif :
overrides ciblés + nouveau token `--text-cuisine-instruction`.

**`eau` casse l'agrégation de la liste de courses avec un `NaN`**
(trouvé et corrigé dans `70ad33d`, avant tout commit séparé)
Symptôme : quantité affichée `NaN ml` dès qu'une recette contenant de
l'eau était affectée. Cause : `eau` a `suivi_stock: ignore` et
`conditionnement: 0` dans le référentiel ; le code ne traitait
explicitement que `suivi_stock: binaire`, laissant `ignore` tomber dans
le chemin normal — `Math.ceil(x / 0)` = `Infinity`, `Infinity * 0` =
`NaN`. Correctif : exclusion complète des ingrédients `ignore` de la
liste (ni achat, ni "à vérifier" — rien à suivre).

**`sync-vault.mjs` plantait systématiquement sous Windows** (`0be1f62`)
Symptôme : `npm run sync-vault` échouait à chaque exécution avec `EPERM`
sur le premier fichier écrit. Cause : `fs.cpSync` applique un `chmod` sur
la destination, refusé par Windows dès que le dossier est géré par
OneDrive ou surveillé par un antivirus actif — le cas du dossier `data/`
sur la machine de développement. Correctif : copie manuelle
`readFileSync`/`writeFileSync`, qui ne touche jamais aux permissions ; au
passage, le contexte se synchronise en bloc vers `data/contexte/` plutôt
que fichier par fichier à la racine de `data/` (voir §2).

**Sauce blanche maison comptée deux fois** (`555c26d`)
Symptôme : l'agrégation de `salade-concombre-tomates-mozza` donnait 11
lignes au lieu de 6-7 attendues. Cause : la recette portait un second
tableau "Sauce blanche maison — deux minutes" décrivant une
**alternative** à l'ingrédient `sauce-blanche` du tableau principal, pas
un ajout — l'algorithme (type_production `repas` → tous les groupes,
volontairement, voir §2) achetait donc la sauce du commerce ET les
ingrédients pour la faire soi-même. Correctif : la préparation maison
passe en prose dans les Notes ; `_gabarit-recette.md` documente
désormais explicitement qu'une alternative ne se met jamais dans un
tableau. **Non résolu** : après correction, l'agrégation donne 7 lignes
(3 articles + 4 "à vérifier"), pas les 6 attendus par l'utilisateur — la
donnée et le code ont été vérifiés sans trouver de règle qui justifierait
d'en retirer une huitième. Question posée, jamais tranchée depuis (voir
§6).

**Budgets de créneaux à "0 min" excluant toutes les recettes** (`67f32ca`)
Symptôme : feuille de sélection vide pour `dejeuner` et
`recharge-express`. Cause : lecture littérale de
`contexte-projet-meal-planner.md` §4.3, qui donne "0 min" pour ces deux
créneaux — en réalité une contrainte de préparation en amont, pas un
temps sur place. Voir §2 pour le correctif complet (nouvelle table de
budgets + contraintes booléennes séparées).

**Composant exclu même pour `batch`** (`67f32ca`, même commit que le
bug précédent, cause différente)
Symptôme : `batch` ne proposait que 4 recettes au lieu de 6. Cause : le
filtre "jamais un composant" de la feuille de sélection était appliqué
sans exception, alors que CLAUDE.md dit explicitement qu'un composant
"ne sort que sur un créneau `batch` ou par chaînage automatique" — `batch`
est l'exception documentée, pas une exclusion totale. Correctif :
`type_production === 'repas' || creneau.type === 'batch'`.

**Barre d'onglets recouvrant le bas du contenu** (`c34db14`)
Symptôme : rapporté par l'utilisateur sans capture, contenu caché sous la
barre. Cause confirmée : le détail de recette (`RecetteDetail`, rendu via
l'`Outlet` d'`AppShell`) n'avait **aucun** mécanisme de dégagement — les
trois onglets racine, eux, étaient déjà bornés par leur conteneur fixe
(`bottom: calc(58px + safe-area)`), donc probablement déjà corrects, mais
non vérifiables visuellement (pas d'accès navigateur dans cet
environnement). Correctif : token utilitaire `.pb-barre` appliqué aux
quatre écrans défilants, plus la zone sûre ajoutée au padding bas de la
feuille de sélection elle-même par précaution.

---

## 4. Limites assumées

Chaque point ci-dessous est un choix délibéré, pas un oubli — casé dans
`## Ce qu'il ne faut pas construire` des deux briefs. Ce qui débloque
chacun est précisé pour éviter qu'une session future improvise une
solution partielle par méconnaissance du blocage réel.

- **État de carte "faite" jamais déclenché.** Le type (`EtatCarteRepas`)
  et le rendu visuel existent (opacité réduite, titre barré, étiquette
  "fait"), mais aucun code ne produit jamais cet état. Raison :
  `spec-bilan-et-portions-restantes.md` décrit "faite" comme la
  conséquence d'une **validation de recette après cuisine** (note, photo,
  portions restantes, décrément du stock) — un bouton "Marquer fait"
  isolé maintenant créerait un chemin parallèle à démonter en session 4,
  et perdrait les données de goût que ce flux est censé capturer. Débloqué
  par : l'écran de bilan après cuisine (explicitement hors périmètre
  session 2, voir liste ci-dessous).
- **Planning sans dates réelles.** `JourSemaine` est un jour de semaine
  abstrait ("lundi"), jamais rattaché à une date calendaire. Conséquence :
  impossible de savoir automatiquement si "aujourd'hui" est passé pour
  marquer un créneau comme faite, impossible de raccourcir le chemin vers
  "le repas de ce soir" (`spec-navigation.md`, "Point laissé ouvert").
  Débloqué par : une décision de modèle de données en session 3 — soit la
  semaine porte une date de début et les jours deviennent des dates, soit
  le modèle reste abstrait ; explicitement non tranché avant cette
  session-là.
- **Sélection en swipe.** La feuille de sélection est une liste, pas un
  swipe plein écran. Raison assumée dans `DESIGN.md` : la liste permet de
  comparer plusieurs options d'un coup d'œil pour remplir dix créneaux
  d'affilée, ce que le swipe interdit par construction ; le swipe reste
  "possible en phase 5" mais n'est pas un manque à combler maintenant.
- **Scoring et rotation.** `recettesPourCreneau` ne fait que filtrer et
  trier par temps actif — aucune notion de "quand cuisinée pour la
  dernière fois" n'influence l'ordre ou l'inclusion. Débloqué par : des
  données d'usage réelles (nombre d'exécutions, dates), qui n'existent pas
  encore puisque l'app n'a pas encore été utilisée en conditions réelles.
- **Portions stockées (état "stock" de la carte, section "Déjà prêt au
  frigo" de la feuille).** Dépend d'un système de placards/stock qui
  n'existe pas. Débloqué par : le moteur de stock, explicitement
  repoussé à la phase 3 par CLAUDE.md — il dépend du référentiel
  d'ingrédients (qui existe depuis la session 1) mais aussi de données de
  goût collectées en usage réel.
- **`diner-a-deux` jamais généré.** Aucun événement ne produit ce créneau
  cette session — décision explicite du brief 2 : il ne peut venir que
  d'un événement ajouté à la main, prévu en phase 3. Le filtre de
  matching (`cuisine_a_deux.adapte`) et la constante `budgetTempsActif`
  existent déjà, prêts à être exercés dès qu'un tel événement existera.
- **Hors périmètre des deux sessions, sans détail supplémentaire ici**
  (repoussé à la phase 3, listé dans les briefs) : moteur de matching par
  score, décrément de stock, écran de bilan après cuisine, suivi de
  budget et tickets de caisse, notion de magasins, authentification et
  Supabase.

---

## 5. Vérification

Aucun framework de test (`npm test` n'existe pas) — décision assumée,
pas un oubli : projet personnel mono-utilisateur, CLAUDE.md décourage les
dépendances lourdes non justifiées. La vérification passe par le
type-checker, deux scripts de garde-fou sur les données/la génération, et
des scripts `tsx` jetables écrits à la volée pendant le développement pour
exercer le code réel contre les vraies données (voir §6 pour la technique,
utile pour toute session future qui doit diagnostiquer un comportement
sans navigateur).

| Commande | Contrôle | Sortie saine | Échec connu |
|---|---|---|---|
| `npm run build` | `tsc` (type-check complet) puis `vite build` | `built in Xs`, code de sortie 0 | Toute erreur TypeScript ; un avertissement `eval` de `gray-matter` et un avertissement de taille de chunk apparaissent systématiquement, **pré-existants, pas un échec** |
| `npm run validate` | Frontmatter et corps de chaque recette contre les schémas Zod ; chaque référence d'ingrédient (principale et substitut) existe dans le référentiel ; `produit_par` pointe vers une recette qui existe | `N recette(s), M ingrédient(s)… 0 erreur(s), K avertissement(s)` — les avertissements (référence vers un stub `a_completer: true`) sont attendus et non bloquants | Code de sortie 1 si au moins une erreur ; `--verbose` affiche chaque avertissement au lieu du seul résumé |
| `npm run verifier-modeles` | Nombre de créneaux générés par jour pour la semaine d'alternance (4,5,4,4,5,3,4), heure du petit-déj samedi/dimanche (10h00/08h30), et que mercredi produit bien un `batch` | Une ligne `✓` par jour + une ligne `✓ mercredi : batch à …` | Code de sortie 1 si un compte, une heure ou le jour de batch diverge — regarder d'abord si `modeles-semaine.ts` ou la règle de repli dans `evenements.ts` a changé |
| `npm run sync-vault` | Recopie `../LazEat/00-Contexte`, `01-Recettes`, `02-Ingredients/referentiel-ingredients.yaml` vers `data/` | `N fichiers` par section, "Synchronisation terminée" | Échoue immédiatement si `../LazEat` (le coffre Obsidian, dépôt frère) n'existe pas à côté de ce dépôt — jamais appelé par le build Vercel |
| `npm run dev` / `npm run preview` | Pas un contrôle — serveur de dev / aperçu du build | — | — |

`npm run build` et `npm run validate` sont les deux à lancer
systématiquement avant de considérer un changement terminé ; 
`npm run verifier-modeles` seulement si `modeles-semaine.ts` ou
`evenements.ts` a été touché.

---

## 6. Pièges pour une session future

- **`ingredients.ts` et `recettes.ts` ne s'exécutent pas hors de Vite.**
  Ils utilisent `import.meta.glob` et l'import `?raw`, spécifiques au
  bundler. Un script `tsx scripts/mon-diagnostic.mts` qui importe (même
  indirectement, via `courses.ts` ou `semaine.ts`) l'un de ces deux
  modules plante avec `ERR_UNKNOWN_FILE_EXTENSION` ou
  `ERR_MODULE_NOT_FOUND`. Pour exécuter du vrai code applicatif contre les
  vraies données depuis un script jetable, passer par le serveur Vite en
  mode programmatique :
  ```js
  import { createServer } from 'vite';
  const server = await createServer({ configFile: 'vite.config.ts', server: { middlewareMode: true } });
  await server.ssrLoadModule('/scripts/mon-diagnostic.mts');
  await server.close();
  ```
  Technique utilisée à plusieurs reprises en session 2 pour vérifier
  l'agrégation de courses et les effectifs de `recettesPourCreneau` contre
  les 31 vraies recettes plutôt que de le déduire par lecture de code.

- **`CreneauCalcule.id`** (format `` `${jour}-${type}-${index}` ``) est la
  clé de persistance des affectations (`AffectationCreneau.creneauId`).
  Il n'est stable que si le même `(jour, type, index)` est reproduit à
  chaque génération. Actuellement, changer le mode d'un jour
  (présentiel/distanciel/…) ne change **jamais** l'ensemble des créneaux
  générés (seulement deux booléens sur le déjeuner), donc les id restent
  stables à travers les bascules — mais si une session future fait
  dépendre le *type* ou le *nombre* de créneaux du mode, les affectations
  existantes se retrouveront orphelines silencieusement (l'ancien id
  n'existera simplement plus, sans erreur).

- **La conversion `soir-cuisine` → `batch`** (règle du premier jour
  distanciel/libre avec `soiree-libre`) mute le type du créneau **avant**
  que son `id` soit calculé, à l'intérieur de `genererCreneauxJour`. Si
  cette règle est un jour déplacée en post-traitement après construction
  des `CreneauCalcule`, il faut aussi recalculer l'`id` — sinon il
  continuera à contenir "soir-cuisine" alors que le type affiché est
  "batch", et toute affectation faite sur ce créneau après le changement
  ne retombera plus sur le même id à la génération suivante.

- **Ne jamais boucler sur `genererCreneauxJour` jour par jour dans un
  nouvel appelant.** Toujours `genererCreneauxSemaine`, qui décide une
  seule fois quel jour devient le `batch` de la semaine. Trois appelants
  dupliquaient cette boucle avant d'être unifiés en session 2 — la
  duplication est exactement ce qui a permis que la règle du batch
  cooking diverge silencieusement entre écrans.

- **Deux tables `NOM_JOUR` distinctes existent, volontairement.**
  `creneaux.ts` exporte une version minuscule (ajoutée en session 2 pour
  les messages de chaînage) ; `EnTeteJour.tsx` a sa propre constante
  locale capitalisée (session 1, pour le titre de l'en-tête). Ne pas les
  unifier sans vérifier la casse attendue à chaque usage.

- **`CreneauCalcule.transportableRequis`/`bonFroidRequis` ne sont
  consommés nulle part.** Ils sont peuplés par `genererCreneauxJour` (via
  `CONTRAINTES_CRENEAU` et le mode du jour pour le déjeuner) mais aucun
  composant ni aucune fonction ne les lit — vestiges de la session 1,
  en attente du moteur de matching de phase 3. Ne pas les confondre avec
  `matching.ts` → `CONTRAINTE_SUPPLEMENTAIRE_PAR_CRENEAU`, un mécanisme
  entièrement différent ajouté en session 2, qui lit des champs de la
  **recette** (`bon_froid`, `transportable`, `batchable`,
  `cuisine_a_deux.adapte`), pas du créneau.

- **La ligne d'ingrédients de `RecetteDetail` et l'aperçu de
  `CarteOption` n'appliquent pas la règle repas/composant des groupes
  d'ingrédients.** Seule `courses.ts` (`groupesAAgreger`) le fait, parce
  que c'est le seul endroit où ignorer la règle coûte réellement quelque
  chose (achat en double). Les deux autres affichages aplatissent tous
  les groupes sans distinction — comportement délibérément différent, pas
  un oubli à harmoniser.

- **Une classe Tailwind qui référence un token `@theme` inexistant ne
  casse jamais la compilation** — elle disparaît silencieusement du CSS
  généré. `npm run build` réussit, le rendu est juste faux (pas de rayon,
  pas de couleur). Après tout ajout de token dans `index.css`, vérifier
  dans `dist/assets/index-*.css` (après un build) que la classe attendue
  est bien présente si un doute existe, en particulier pour les valeurs
  arbitraires avec virgules (`bg-[linear-gradient(...)]`) — elles
  fonctionnent (confirmé par inspection du CSS généré, les virgules n'ont
  pas besoin d'échappement, seuls les espaces doivent devenir des
  `_`), mais ça vaut la peine de vérifier plutôt que de supposer.

- **`recharge-express` n'a aucune recette compatible dans les données
  actuelles** — confirmé en session 2, ce n'est pas un bug de filtre : la
  feuille de sélection s'ouvrira vide sur ce créneau tant qu'aucune
  recette n'aura `recharge-express` dans `creneaux_compatibles`. Ne pas
  chercher un bug de code si ce créneau reste vide.

- **Le compte de 7 lignes pour `salade-concombre-tomates-mozza` n'a
  jamais été confirmé par l'utilisateur** (qui en attendait 6, voir bug
  §3). Le code a été vérifié correct contre les données actuelles ; si ce
  nombre doit changer, ce sera par une modification de données (retirer
  un ingrédient, changer un `suivi_stock`), pas par un correctif de
  `courses.ts`.

- **`data/contexte/` est suivi par git**, pas ignoré — y compris son
  sous-dossier `99-Archives`. Contrairement à ce que le nom pourrait
  suggérer, ce n'est pas un dossier de build local à exclure : c'est la
  copie de référence du contexte, commitée pour qu'une session sans accès
  au coffre Obsidian (`../LazEat`) puisse quand même lire les specs.

- **Deux clés `localStorage` distinctes partagent le même id de
  semaine** : `lazeat:semaine:${id}` et `lazeat:liste-courses:${id}`.
  Vider l'une dans les devtools ne vide pas l'autre — une liste de
  courses avec des cases cochées peut survivre à une semaine planifiée
  remise à zéro, et inversement.

- **`noUncheckedIndexedAccess` est activé** (`tsconfig.json`). Tout accès
  indexé (`array[i]`, `record[clé]`) est typé `T | undefined`, d'où les
  nombreux `!` dans le code existant sur des accès jugés sûrs par
  construction (ex. après un `.find()` dont l'existence vient d'être
  vérifiée juste avant). Ne pas les retirer par réflexe de nettoyage —
  ils sont presque toujours nécessaires, pas oubliés.
