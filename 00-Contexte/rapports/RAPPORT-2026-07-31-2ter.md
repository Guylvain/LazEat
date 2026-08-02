# Session 2ter — 2026-07-31

Ramassage de ce qui s'est accumulé pendant le test manuel de la session
2bis (validée), plus la remontée de la fonctionnalité "portions restantes"
depuis la phase 4. Treize commits, un par livrable (A1 à A5, B, puis six
pour la Partie C, dont le découpage est expliqué plus bas).

## Ce qui a été fait

**A1 — jour de batch cooking explicite** (`3f23fd6`). `Evenement` gagne un
champ `batch?: boolean`. `MODELE_COURS` et `MODELE_ALTERNANCE` le portent
tous deux sur leur `soiree-libre` de mercredi. `premierJourBatch()`
(devenu `jourBatch()`) ne déduit plus rien du mode du jour : il cherche
l'événement marqué `batch: true`. En semaine de cours, l'ancienne règle
tombait sur samedi (tous les jours de semaine étant `presentiel` par
défaut) — dernier soir cuisinable, chaînage de production impossible.

**A2 — `verifier-modeles-semaine.ts` étendu à `MODELE_COURS`** (`63f10fe`).
Le script vérifiait uniquement `MODELE_ALTERNANCE` ; c'est ce qui a laissé
passer A1 pendant deux sessions. Généralisé en une fonction
`verifierModele()` appelée pour les deux modèles, chacun avec sa propre
table de conformité.

**A3 — conseil du petit-déjeuner corrigé** (`1f513c3`).
`CONSEIL_CRENEAU['petit-dej']` : « Assemblage, 5 min max. » → « Express ou
posé, 15 min max. », aligné sur `budgetTempsActif: 15`.

**A4 — points de `SelecteurSemaine` alignés sur `EnTeteJour`** (`d39c421`).
`planifiesParJour` (`PlanningPage.tsx`) exclut désormais les créneaux
`fixe`, comme le fait déjà le ratio "N/M repas planifiés" depuis la 2bis.

**A5 — échelle de note 1-4, type `Verdict` fusionné** (`fa6b306`).
`VerdictSchema` (nouveau, dans `recette.schema.ts`) devient la source
unique : `RecetteFrontmatterSchema.note` et `Verdict`
(`types/bilan.ts`, avant dupliqué en dur) en dérivent tous deux.

**B — sélecteur de portions jusqu'à 10** (`6312e28`). `RecetteDetail` ne
passe plus `portions_max_reel` comme `max` du sélecteur mais une constante
`PORTIONS_MAX_SELECTEUR = 10`. Au franchissement de `portions_max_reel`,
un bandeau affiche `note_scaling` si présent, sinon un message générique.
`portions_max_reel` n'est touché nulle part dans les données.

**Partie C — portions restantes**, six commits :

1. `e2edfdf` — entité `PortionStockee` (`types/portionStockee.ts`) et
   persistance (`lirePortionsStockees`/`ecrirePortionsStockees`).
2. `790c17b` — `lib/portionsStockees.ts` : `calculerDlc` (selon
   `qualite_j2`, §3.3-3.4), `enregistrerPortionsRestantes`,
   `consommerPortionStockee`, `estCouvertParStock`.
3. `fc9b1ac` — `EcranBilan` pose désormais « Il t'en reste ? » après le
   verdict (0 à 3, jamais obligatoire), puis « Où ça va ? » (frigo/congélateur)
   si `qualite_j2` le permet.
4. `207d87c` — **le point d'architecture** : `calculerListeCourses` accepte
   `PortionsStockees` et saute la création d'un `BesoinNonCouvert` quand une
   portion stockée de la recette productrice existe.
5. `e8d981d` — `ChoixCreneau` gagne la variante `portion-stockee`,
   `affecterPortionStockee` (symétrique à `affecterRecette`).
6. `2a90543` — section "Déjà prêt au frigo" dans `FeuilleSelection` (avant
   "À cuisiner"), `CarteOption` en variante ambrée, `CarteRepas` rend
   l'état `portion-stockee` (déjà prévu dans `EtatCarteRepas` et DESIGN.md,
   jamais produit avant cette session).

Le découpage en six suit les livrables logiques du point d'architecture
(demandé avant tout code, l'utilisateur a choisi l'option liée) plutôt
qu'un découpage arbitraire — chacun compile et passe `verifier-modeles`
seul.

## Ce qui a été vérifié, et comment

**Exercé réellement** (code réel contre données réelles, scripts via
`vite.createServer({server:{middlewareMode:true}}).ssrLoadModule(...)`,
supprimés après usage) :

- A1 : `npm run verifier-modeles` (les deux modèles, tous verts) et un
  script direct sur `genererCreneauxSemaine(MODELE_COURS, {})` — je n'ai
  **pas** rejoué l'ancien bug avant correctif (la régression était déjà
  démontrée par le test manuel de la 2bis, décrite dans le prompt), j'ai
  seulement vérifié que le résultat après correctif est correct.
- A2 : sortie complète des 7 jours de `MODELE_COURS` tracée à la main
  d'après les règles déjà en code, puis confirmée par exécution —
  concordance totale sur les comptes et sur les heures citées par la table
  du coffre, à une exception près (voir "Demandes vers le coffre").
- A3 : `recettesPourCreneau` exécuté contre les 41 vraies recettes pour un
  créneau petit-dej réel — 6 recettes, `tortilla-petits-pois-comte`
  présente à 15 min pile. La comparaison de `matching.ts` était déjà `<=`,
  pas de correctif nécessaire là — vérifié, pas supposé.
- A5 : `npm run validate` après resserrement du schéma — 0 erreur sur les
  41 recettes réelles (toutes à `note: null` actuellement, donc aucune
  migration de donnée nécessaire).
- Partie C, DLC : `calculerDlc` exécuté contre `tortilla-petits-pois-comte`
  (qualite_j2 `excellente`, `duree_conservation_frigo: 3`) et des variantes
  `correcte`/`a-eviter` — dates conformes à §3.3-3.4 dans les trois cas,
  congélateur à 90 jours. `joursAvantDlc` testé sur les trois bornes
  (avant/le jour même/après).
- Partie C, persistance : `localStorage` simulé en mémoire (polyfill
  minimal dans le script, pas dans le code de l'app) pour exercer
  `enregistrerPortionsRestantes` → `consommerPortionStockee` →
  `estCouvertParStock` hors navigateur — création, décrément, suppression
  à zéro, et confirmation qu'une réponse "0" n'écrit rien.
- Partie C, chaînage (le point d'architecture) : `garniture-pizza-pepperoni`
  planifiée sans `pate-a-pizza-maison` sur un vrai créneau de
  `MODELE_COURS` — `BesoinNonCouvert` présent pour `paton-pizza` sans
  stock, absent avec un stock de `pate-a-pizza-maison`. C'est exactement
  le scénario décrit dans le point d'architecture du prompt.
- Partie C, créneau : `affecterPortionStockee` + `appliquerAffectations`
  exécutés — `etat: 'portion-stockee'`, recette, portions et `dlcEstimee`
  tous corrects.
- Partie C, "Déjà prêt" : `portionsCompatiblesCreneau` exécuté contre les
  vraies données — `tortilla-petits-pois-comte` proposée sur `petit-dej`,
  absente de `recharge-express` (pas transportable pour ce créneau-là) ;
  la liste se vide après consommation totale.
- Après chaque commit : `npx tsc --noEmit`, `npm run build` (production
  réelle, pas seulement `dev`), `npm run validate`, `npm run
  verifier-modeles`.

**Seulement relu / raisonné, jamais vu à l'écran** : A4 (le rendu des
points de `SelecteurSemaine`), la Partie B (le sélecteur à 10 et le
bandeau d'avertissement), et toute la Partie C côté écran — les trois
étapes de `EcranBilan` (dont l'avertissement `a-eviter`), la section "Déjà
prêt au frigo" dans `FeuilleSelection`, la carte ambrée de `CarteOption`,
l'état `portion-stockee` de `CarteRepas`. La logique de données qui les
alimente est exercée (voir ci-dessus) ; leur rendu ne l'est pas.
L'extension Chrome de cet environnement ne joint pas `localhost` — piège
déjà documenté dans `RAPPORT-2026-07-31-2bis.md`, pas retenté cette
session faute de raison de penser que ça a changé.

## Ce qui a échoué ou reste non vérifié

Rien n'a échoué à proprement parler, mais tout le rendu visuel de cette
session repose sur du code qui compile et une logique de données exercée
par script — jamais sur une observation à l'écran. C'est la même réserve
que la 2bis avait posée sur son propre écran de bilan, étendue ici à
davantage d'écrans (A4, B, et toute la Partie C).

Deux zones précises restent non vérifiées même par script, faute de
scénario à leur portée cette session :

- Le passage `qualite_j2: a-eviter` de `EcranBilan` (avertissement affiché,
  congélation masquée, DLC forcée à 1 jour) : le calcul de DLC est exercé
  (voir ci-dessus), l'enchaînement des trois étapes de l'écran ne l'est que
  par lecture du code.
- Le comportement de `note_scaling` quand il vaut `null` sur une vraie
  recette (message générique) : aucune des 41 recettes réelles inspectées
  n'a ce champ à `null`, donc le repli n'a pas été exercé contre une
  vraie donnée — seulement contre le code.

## Décisions prises et leur raison

**A2, écart "20h15" non corrigé.** La table de `spec-modeles-semaine.md`
§3 indique "recharge 20h15" pour mardi ; l'implémentation produit 19h15.
Même écart déjà présent — et déjà toléré — dans la table §2 de
l'alternance, vérifiée conforme en session 2. Ce n'est donc pas une
régression d'A1 ni une erreur de mon script : signalé ci-dessous plutôt
que "corrigé" d'un côté ou de l'autre, comme demandé par A2.

**A5, schéma comme source unique.** `Verdict` (avant dupliqué en dur dans
`types/bilan.ts`) dérive maintenant de `VerdictSchema`
(`recette.schema.ts`), pas l'inverse — le gabarit de recette "fait foi"
(CLAUDE.md), c'est donc le schéma qui valide les vraies données qui doit
porter la définition.

**Partie C, point d'architecture : option liée, retenue par l'utilisateur.**
Les deux options et leurs coûts ont été présentées avant tout code ; la
liaison a été choisie. Portée volontairement limitée à une **vérification
binaire de présence** (`estCouvertParStock`), pas à une comptabilité fine
par quantité d'ingrédient : le moteur de stock au grain de l'ingrédient
est explicitement hors périmètre avant la phase 3 (CLAUDE.md). Conséquence
assumée : une portion stockée de composant n'est **jamais décrémentée
automatiquement** quand la recette qui la consomme est cuisinée — voir
"Demandes vers le coffre".

**`PortionStockee`, un seul lot actif par recette.** Le §3.2 de la spec
montre un identifiant par lot (`id: ps-20260805-001`) ; j'ai indexé
`PortionsStockees` par `recette_id` à la place, sur le modèle explicite de
`Bilans` demandé par le prompt ("l'entité PortionStockee persistée, sur le
modèle de Bilans"). Répondre deux fois à "Il t'en reste ?" pour la même
recette remplace l'entrée précédente au lieu de l'additionner. Plus simple,
mais incapable de représenter deux lots de la même recette avec des DLC
différentes en même temps — acceptable pour l'usage réel (le congélateur
sert justement à ça), signalé ci-dessous.

**Code couleur DLC → formulation, pas palette.** §3.8 demande vert/orange/
rouge. DESIGN.md ne définit qu'un seul accent (`amber`) et CLAUDE.md
interdit d'inventer une couleur. Traduit en texte
(« à manger aujourd'hui » / « demain » / « avant le [date] »), le seul
signal binaire réellement exprimable sans couleur inventée
(`dlcUrgente`) régissant l'usage du fond ambré déjà licencié pour ce type
de carte (DESIGN.md, "Portion stockée").

**`EcranBilan`, trois étapes au lieu d'un seul écran.** Le mockup de la
spec (§1) montre tout sur un seul écran avec un bouton "Valider" final ;
l'écran existant (2bis) navigue immédiatement au tap du verdict. Gardé ce
principe : verdict → "Il t'en reste ?" (skippable) → "Où ça va ?"
(seulement si portions ≥ 1 et congélation disponible). Deux taps
supplémentaires au pire, jamais obligatoires au-delà du verdict.

## Pièges découverts

- **Toute vérification touchant à `persistance.ts` hors navigateur exige
  un polyfill `localStorage`.** `vite.createServer(...).ssrLoadModule(...)`
  suffit pour `import.meta.glob`, mais le code de persistance appelle
  `localStorage` directement (frontière système assumée, CLAUDE.md) — sans
  polyfill le script plante. Technique utilisée cette session pour
  `lib/portionsStockees.ts`, pas nécessaire avant (rien ne touchait encore
  la persistance dans les scripts de vérification précédents).
- **La section "Déjà prêt au frigo" n'a pas eu besoin de nouvelle règle
  pour exclure les composants.** `portionsCompatiblesCreneau` filtre sur
  `typeContenu === 'repas'`, qui suffit : un composant (pâton, riz cuit)
  n'a simplement pas de portion `repas` à proposer sur un créneau-repas. La
  règle CLAUDE.md ("un composant n'apparaît jamais dans le matching d'un
  créneau repas") n'a donc pas eu besoin d'être répliquée ici, elle
  découle du typage des données.
- **`CreneauAffecte` a un champ `dlcEstimee` obligatoire (`string | null`),
  pas optionnel.** Les quatre branches d'`appliquerAffectations` devaient
  toutes l'assigner explicitement — TypeScript l'a exigé au premier `tsc`,
  aucune branche oubliée n'aurait compilé. À reproduire pour toute future
  extension de `ChoixCreneau` : préférer un champ obligatoire (même
  `| null`) à un champ optionnel quand toutes les branches doivent en
  décider une valeur, ça transforme un oubli silencieux en erreur de build.

## Demandes vers le coffre

- **`spec-modeles-semaine.md` §3, ligne "recharge 20h15" (mardi, semaine de
  cours).** L'implémentation produit 19h15 — cohérent avec §2 (alternance),
  qui porte la même valeur pour le même calcul et est déjà considérée
  conforme. À corriger dans la table plutôt que dans le code, sauf si
  20h15 était en fait la valeur voulue à l'origine et que c'est
  l'implémentation qui devrait changer (aucun élément ne l'indique).
- **`PortionStockee`, un lot par recette plutôt qu'un lot par identifiant.**
  Simplification de session 2ter (voir "Décisions prises"), qui s'écarte
  du schéma YAML à `id: ps-...` de §3.2. À valider : soit la spec est mise
  à jour pour documenter ce modèle simplifié, soit une session future migre
  vers une liste de lots si le besoin de deux DLC simultanées pour une même
  recette se manifeste en usage réel.
- **DESIGN.md n'a pas de palette d'urgence (vert/orange/rouge).**
  spec-bilan-et-portions-restantes.md §3.8 en réclame une. Traduit en
  formulation cette session (voir "Décisions prises") pour respecter la
  règle "ne jamais inventer de couleur" — à trancher : ajouter des tokens
  d'urgence à DESIGN.md pour une phase future, ou confirmer que la
  formulation suffit durablement.
- **Portions stockées de composant jamais décrémentées automatiquement.**
  Une portion de `pate-a-pizza-maison` couvre le chaînage tant qu'elle
  existe, mais rien ne la décrémente quand la recette qui la consomme
  (une garniture) est réellement cuisinée — le stock réel du congélateur et
  celui de l'app peuvent diverger silencieusement sur plusieurs semaines.
  Assumé pour cette session (le moteur de stock au grain de l'ingrédient
  est phase 3, CLAUDE.md) ; à traiter explicitement quand ce moteur
  arrivera, pas oublié.

## Ce qui reste ouvert

- **Aucun écran de cette session n'a été vu fonctionner dans un vrai
  navigateur** — A4, Partie B, et toute la Partie C. Le protocole de test
  manuel qui a validé la 2bis doit être rejoué pour ces livrables avant
  session 3.
- **Décrément automatique des portions stockées de composant** (voir
  "Demandes vers le coffre") — laissé pour la phase où le moteur de stock
  d'ingrédients existera.
- **Suggestion automatique par défaut** (§3.7, DLC à deux jours proposée
  avant l'écran de matching) — explicitement hors périmètre, nécessite des
  dates réelles de planning (décision de session 3, comme indiqué par le
  prompt).
- **Contrainte des douze boîtes** (§3.5), **suivi du gaspillage** (§4),
  **remontée au niveau ingrédient** (§2.4), **raisons de rejet
  structurées** (§2.1) — hors périmètre, comme indiqué par le prompt.
- **Palette d'urgence DLC** (voir "Demandes vers le coffre").
