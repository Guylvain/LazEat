# Rapport de session 2bis — remise en état avant Supabase

Document de passation. À fournir à la session Claude Code qui travaille sur
`LazEat-app`. Rédigé depuis le coffre, sans aucune écriture dans le dépôt de
l'app : tout ce qui suit est un diagnostic vérifié, pas un correctif appliqué.

Date du diagnostic : 31 juillet 2026. `HEAD` = `f609461`, arbre propre,
aligné sur `origin/main`.

---

## 0. Pourquoi cette session existe

Les sessions 1 et 2 étaient déclarées terminées. `npm run build`,
`npm run validate` et `npm run verifier-modeles` passaient tous. Le journal
technique les documentait en détail.

Et pourtant, **aucune recette n'a jamais été chargée dans le build de
production**. Tout ce qui dépend d'une recette affectée — liste de courses,
chaînage, feuille de sélection — n'a donc jamais été exercé une seule fois
en conditions réelles.

C'est le fil rouge de ce rapport et la leçon à retenir : **les trois
commandes de vérification ne traversent aucun des chemins cassés.**
`validate.ts` et `verifier-modeles-semaine.ts` tournent sous `tsx`, hors du
bundler ; ils ne voient jamais `import.meta.glob`. `tsc` ne voit pas les
contextes d'empilement CSS. `vite build` ne voit pas qu'un glob ne matche
rien. Trois voyants verts, trois angles morts.

---

## 1. Bug — la bibliothèque est vide au build (extglob non supporté)

**Symptôme.** Bibliothèque à « 0 recette », feuille de sélection sans section
« À cuisiner », planning à « 0/4 repas planifiés ». Sur Vercel **et** sur
`npm run build` en local. **Mais pas sur `npm run dev`.**

**Cause racine.** `src/lib/recettes.ts` :

```ts
import.meta.glob('/data/recettes/!(_gabarit-recette|inventaire-socle-recettes).md', …)
```

Vite 8 n'interprète pas la négation extglob `!(a|b)` dans `import.meta.glob`.
Il attend des motifs négatifs **séparés, préfixés par `!`, dans un tableau**.
Le motif ne matche donc rien, la carte de modules est vide, et
`getAllRecettes()` renvoie `[]` — sans lever, sans avertir.

**Le piège dev/build.** Le serveur Vite résout le glob à la volée à chaque
requête et tolère l'extglob ; le build le résout via Rollup, qui ne le
tolère pas. D'où un dev parfaitement vert et une production vide. C'est le
profil de bug le plus coûteux possible.

**Preuve.** Dans le `dist/` du 30 juillet :

| Chaîne cherchée | Occurrences | Origine |
|---|---|---|
| `Bol yaourt` | 0 | recette — absente |
| `Chili con carne` | 0 | recette — absente |
| `Riz basmati` | 1 | référentiel — présent |
| `yaourt-grec` | 3 | référentiel — présent |

Le référentiel d'ingrédients passe par un **import statique**
(`import referentielBrut from '../../data/referentiel-ingredients.yaml?raw'`),
pas par un glob. C'est pour ça que lui seul survit au build.

**Correctif.**

```ts
const modules = import.meta.glob(
  [
    '/data/recettes/*.md',
    '!/data/recettes/_gabarit-recette.md',
    '!/data/recettes/inventaire-socle-recettes.md',
  ],
  { eager: true, query: '?raw', import: 'default' },
) as Record<string, string>;
```

**Correctif complémentaire, non négociable.** Le vrai défaut n'est pas le
motif, c'est le silence. Ajouter une garde bruyante :

```ts
if (Object.keys(modules).length === 0) {
  throw new Error(
    "Aucune recette chargée : le motif de import.meta.glob ne matche aucun fichier. " +
    "Vérifier que data/recettes/ est peuplé et que le motif est valide pour Vite."
  );
}
```

Sans cette garde, le même bug reviendra à la première modification du motif,
et repassera inaperçu exactement de la même façon.

**Vérification.** Après `npm run build` :

```bash
grep -c "Bol yaourt" dist/assets/*.js   # doit être > 0
```

Ne pas se contenter de `npm run dev` : ce bug est invisible en dev par
construction.

---

## 2. Bug — la barre d'onglets recouvre la feuille de sélection

**Symptôme.** La feuille de sélection est correctement affichée, mais son
bas est recouvert par la barre d'onglets. Le texte « Mange dehors — aucun
ingrédient aux courses » est coupé en deux.

**Cause racine.** Les `z-index` sont pourtant justes : barre `z-30`, voile
`z-40`, feuille `z-50`. Le problème est le **contexte d'empilement**.

Dans `src/components/layout/AppShell.tsx`, `EcranOnglet` est un
`position: fixed` **sans `z-index`** :

```tsx
className="fixed inset-x-0 top-0 overflow-y-auto"
style={{ display: actif ? 'block' : 'none', bottom: 'calc(58px + env(safe-area-inset-bottom))' }}
```

Un élément `position: fixed` crée son propre contexte d'empilement. La
feuille de sélection, rendue à l'intérieur de `PlanningPage`, est donc
enfermée dedans : son `z-50` ne se compare qu'à ses frères **internes**,
jamais à la barre. La barre, elle, est un frère direct d'`EcranOnglet` dans
le contexte racine, à `z-30` — et passe donc au-dessus de **tout** le
contenu de l'onglet, feuille comprise.

**Ce que `c34db14` a corrigé, et ce qu'il n'a pas corrigé.** Ce commit a
traité le padding bas du contenu défilant, ce qui était un vrai problème
distinct. Il n'a pas touché à l'empilement. Les deux symptômes se
ressemblaient ; les causes n'avaient rien à voir.

**Correctif recommandé — portail.** `spec-navigation.md` dit que la feuille
« recouvre la barre pendant qu'elle est ouverte ». C'est une modale : elle
doit sortir de l'arbre de l'onglet.

```tsx
import { createPortal } from 'react-dom';

// dans FeuilleSelection, envelopper le voile + le panneau :
return createPortal(
  <>
    {/* voile z-40 */}
    {/* panneau z-50 */}
  </>,
  document.body,
);
```

Le portail place la feuille dans le contexte racine, où `z-40` et `z-50` se
comparent enfin réellement au `z-30` de la barre.

**Correctif alternatif, à éviter.** Donner un `z-index` élevé à
`EcranOnglet` ferait passer tout le contenu de l'onglet au-dessus de la
barre. Le contenu est certes borné par `bottom: calc(58px + safe-area)`,
donc rien ne déborderait visuellement aujourd'hui — mais on rendrait la
barre masquable par n'importe quel élément futur de la page. Le portail
règle la cause, celui-ci déplace le symptôme.

**Vérification.** Ouvrir la feuille de sélection et faire défiler jusqu'en
bas : la section « Autre » et l'option « Mange dehors » doivent être
entièrement lisibles, la barre d'onglets masquée derrière le voile.

---

## 3. Bug — les écrans d'onglet ne se rafraîchissent jamais

**Symptôme observé.** Sur `/courses`, la page affiche « 0 article restant ·
0 recette couverte » et « Aucun repas planifié cette semaine », **alors que
la pastille de l'onglet Courses affiche 6 dans la même capture**. Deux
chiffres contradictoires calculés depuis la même donnée.

**Cause racine.** `AppShell` garde les trois onglets **montés en
permanence** (`display: none`, jamais démontés). C'est une décision assumée
du journal §2, pour conserver filtres et position de défilement.

Conséquence non anticipée : un écran jamais démonté n'est jamais rechargé
non plus. Or **seul `BarreOnglets` s'abonne à
`ecouterChangementsPersistance`** — vérifié, c'est le seul fichier de `src/`
à l'appeler.

| Composant | Jamais démonté | S'abonne au pub-sub | Conséquence |
|---|---|---|---|
| `BarreOnglets` | oui | **oui** | à jour — la pastille dit vrai |
| `CoursesPage` | oui | non | figée à l'état du démarrage |
| `PlanningPage` | oui | non | figée aussi (voir ci-dessous) |

`CoursesPage` charge la semaine une seule fois, au démarrage de l'app,
quand elle était encore vide. Son `useEffect` ne dépend que de `modeleId`.
Affecter une recette depuis le Planning ne déclenche donc **rien** côté
Courses.

`calculerListeCourses` n'a jamais été en cause. L'agrégation, l'arrondi au
conditionnement, le groupage par rayon et le chaînage sont probablement
corrects — on ne leur a simplement jamais passé une semaine à jour. **Ils
restent non vérifiés**, pas cassés.

**Le chaînage de production en découle directement.** `BandeauChainage` se
calcule depuis cette même liste vide : il ne pouvait structurellement rien
proposer. Ce n'est pas un bug du chaînage.

**Le bug est symétrique et concerne aussi le Planning.**
`accepterChainage()` dans `CoursesPage` appelle
`persistance.ecrireSemaine(nouvelle)` — il insère `pate-a-pizza-maison` dans
la semaine. `PlanningPage`, jamais démontée et non abonnée, ne verra jamais
apparaître ce créneau. L'utilisateur accepte le chaînage, retourne au
Planning, et ne voit rien. **Corriger uniquement `CoursesPage` laisserait la
moitié du bug en place.**

**Correctif — abonner les deux pages, sur le modèle de `BarreOnglets`.**

```ts
// CoursesPage
useEffect(() => {
  let annule = false;
  const id = idSemaineCourante(modeleId);

  async function recharger() {
    const [s, l] = await Promise.all([
      chargerSemaine(modeleId),
      chargerListeCoursesPersistee(id),
    ]);
    if (annule) return;
    setSemaine(s);
    setListeCourses(l);
  }

  void recharger();
  const arreter = ecouterChangementsPersistance(() => void recharger());
  return () => { annule = true; arreter(); };
}, [modeleId]);
```

Même traitement sur `PlanningPage` autour de `chargerSemaine(modeleId)`.

**Point de vigilance — pas de boucle, mais un risque de scintillement.**
`definirEtat()` écrit dans la persistance, ce qui notifie l'abonné, ce qui
relit. La relecture n'écrit pas : il n'y a donc pas de boucle, et l'état
converge puisqu'on relit exactement ce qu'on vient d'écrire. En revanche
l'écriture optimiste locale est suivie d'un aller-retour asynchrone qui peut
produire un très bref scintillement sur la case cochée. Si c'est visible en
usage réel, ignorer les notifications émises par la page elle-même — ne pas
retirer la mise à jour optimiste, qui est ce qui rend la coche instantanée
en magasin.

**Vérification.** Le test est précisément celui qui a révélé le bug :
affecter une recette depuis le Planning **sans recharger la page**, passer
sur l'onglet Courses, et vérifier que le nombre affiché sur la page est
**égal** à celui de la pastille. Puis accepter un chaînage depuis Courses et
vérifier que le créneau apparaît côté Planning, toujours sans rechargement.

---

## 4. Manque — l'écran de bilan, dû en phase 2

`spec-bilan-et-portions-restantes.md` §6 place l'écran de bilan **en phase
2**, « même réduit au seul verdict », avec cette justification explicite :

> Sans données de goût accumulées dès le départ, le scoring de la phase 5
> n'aura rien à exploiter.

Il n'existe nulle part dans `src/`. C'est le maillon manquant de toute la
boucle de rétroaction : rien ne crée jamais de portion stockée, donc la mise
en avant des restes dans le matching n'a structurellement aucune donnée à
afficher, aujourd'hui comme dans six mois.

### Une décision d'architecture à trancher d'abord

Le gabarit place `note`, `nb_executions` et `derniere_execution` dans le
**frontmatter de la recette**. Or les recettes sont des fichiers markdown
servis en lecture seule au navigateur : **la PWA ne peut pas écrire dans le
coffre.**

Ces champs ne peuvent donc pas être la destination du bilan. Deux voies :

| Voie | Conséquence |
|---|---|
| **Stocker le bilan dans la persistance**, indexé par `recette_id`, à part du markdown | Le frontmatter devient une valeur *initiale*, la persistance fait foi à l'exécution. Fonctionne dès aujourd'hui en localStorage, migre naturellement vers Supabase. |
| **Écrire dans le markdown** via un script hors ligne | Demande une écriture serveur ou une resynchronisation manuelle du coffre. Incompatible avec un usage depuis le téléphone. |

La première voie est la seule compatible avec le déploiement Vercel et avec
la bascule Supabase de la session 3. Elle implique d'accepter que le
frontmatter et l'état réel divergent — et de le documenter dans
`_gabarit-recette.md`, sinon la prochaine session s'y perdra.

**Périmètre minimal pour la 2bis** : le verdict seul (quatre émojis), écrit
dans la persistance, plus `nb_executions` et `derniere_execution`. Les
raisons de rejet structurées, la photo et les portions restantes attendent
leurs phases respectives.

---

## 5. Manque — `defaut-reconduit`, dû en phase 1

`spec-planning-modifiable.md` §8 place les modes de sélection en **phase 1**.
État réel :

- `MODE_SELECTION_CRENEAU` existe dans `creneaux.ts` et distribue
  correctement `matching` / `defaut-reconduit` / `fixe`
- `evenements.ts` peuple bien `modeSelection` sur chaque `CreneauCalcule`
- **aucun code ne lit jamais ce champ**

C'est la promesse centrale de la spec qui n'est pas tenue : « 26 créneaux
dont 16 sans rien à faire, le rituel du dimanche passe de vingt minutes à
cinq ». Aujourd'hui les 26 créneaux sont à remplir à la main, petit-déjeuner
compris, sept fois par semaine.

**Ce qu'il manque.**

1. Un magasin de préférences dans la persistance : une recette par défaut
   par type de créneau (`spec-planning` §3bis montre
   `recette_defaut: bol-yaourt-banane-noix`).
2. À la génération de la semaine, préaffecter les créneaux
   `defaut-reconduit` avec leur valeur par défaut, sans les compter comme
   « à planifier ».
3. Les créneaux `fixe` (`recharge-express`) sortent du décompte « N/M repas
   planifiés » — ils ne sont jamais demandés.

**Lien avec le travail en cours sur le coffre.** Les créneaux
`defaut-reconduit` sont exactement `petit-dej`, `collation-trajet` et
`avant-match` — les deux créneaux que je suis en train d'étoffer de 3 à 6 et
de 2 à 6 recettes. La reconduction n'a d'intérêt que si la valeur par défaut
peut tourner ; c'est cohérent de livrer les deux ensemble.

**Note sur le compteur.** `recharge-express` n'a aucune recette compatible
dans les données (journal §6, confirmé). Une fois les créneaux `fixe` sortis
du décompte, ce n'est plus un problème visible.

---

## 6. Protocole de test — à dérouler après correction

Aucune de ces vérifications n'est couverte par `npm run build`,
`npm run validate` ni `npm run verifier-modeles`. Elles doivent être faites
à la main, dans cet ordre.

| # | Test | Attendu |
|---|---|---|
| 1 | `npm run build` puis `grep -c "Bol yaourt" dist/assets/*.js` | > 0 |
| 2 | `npm run preview`, onglet Recettes | 35 recettes, filtres et recherche actifs |
| 3 | Ouvrir la feuille de sélection, défiler jusqu'en bas | « Mange dehors » entièrement lisible, barre masquée |
| 4 | Affecter une recette, **sans recharger**, passer sur Courses | Page et pastille affichent le **même** nombre |
| 5 | Recharger la page (F5) | L'affectation survit |
| 6 | Affecter `garniture-pizza-pepperoni` sans pâton | Bandeau proposant `pate-a-pizza-maison` ; farine aux courses, pas de « 1 pâton » |
| 7 | Accepter le chaînage, retourner au Planning **sans recharger** | Le créneau `pate-a-pizza-maison` est apparu |
| 8 | Liste de courses | Groupée par rayon, quantités arrondies au conditionnement |
| 9 | Mode cuisine sur une recette à minuteur | Plein écran sans barre, minuteur qui décompte |
| 10 | Onglet arrière-plan 2 min pendant un minuteur, puis revenir | Le temps restant est juste (timestamp absolu) |

Le test 4 est le plus important : c'est la contradiction pastille/page qui a
révélé le bug 3, et c'est le seul qui vérifie réellement la réactivité
inter-écrans.

---

## 7. Anomalies de données trouvées dans le coffre

Indépendantes des bugs de l'app. Elles n'empêchent rien aujourd'hui mais
fausseront le matching dès qu'il aura un score.

**Clés YAML dupliquées dans `referentiel-ingredients.yaml`.** `compote-a-boire`,
`amandes` et `dattes` apparaissent chacun **deux fois**, avec des valeurs
divergentes — `compote-a-boire` a `conditionnement: 4` puis `12`. C'est la
signature des incidents d'écriture concurrente documentés dans
`ETAT-DU-PROJET.md` §10. Le parseur YAML garde la dernière occurrence
silencieusement.

**`miel` proposé comme substitut alors qu'il est `exclu`.** L'entrée
`sirop-erable` déclare `substituts: [miel]`. Le filtre dur de
`contexte-projet` §5.1 élimine une recette contenant un ingrédient exclu
« sans substitut déclaré » — mais rien n'empêche un **substitut** d'être
lui-même exclu.

**`bol-yaourt-banane-noix` recommande le miel en prose.** Section Notes,
« Version match : ajouter du miel ou du sirop ».

**La famille `petit-dej` va bloquer la rotation.** `bol-yaourt-banane-noix`
et `porridge-avoine-banane` portent `famille: petit-dej`. La règle de
rotation — « le moteur ne propose pas deux recettes de la même famille à
deux jours d'intervalle » — rend impossible de remplir cinq petits-déjeuners
par semaine si les six recettes du créneau partagent une famille. Il faut
des familles fines (`bol-frais`, `bouillie-chaude`, `tartine`, `pancake`,
`tortilla`) et retirer `petit-dej` des deux recettes existantes. `famille`
regroupe des **variantes entre elles**, pas un créneau — le gabarit est
explicite là-dessus.

**`inventaire-socle-recettes.md` est périmé.** Il annonce 31 recettes et
« petit-dej 2 » ; le coffre en contient 35 et le créneau en a 3. À
régénérer.

---

## 7 bis. Demandes vers l'app, issues du travail sur les données

Ajouté après l'écriture des sept recettes de petit-déjeuner et d'avant-match.

**Le conseil affiché sur le créneau `petit-dej` est devenu faux.**
`CONSEIL_CRENEAU['petit-dej']` vaut `'Assemblage, 5 min max.'` alors que
`CONTRAINTES_CRENEAU['petit-dej'].budgetTempsActif` vaut **15**. Les deux se
contredisaient déjà ; avec les trois nouvelles recettes de 10, 12 et 15
minutes — celles des matins de mercredi et vendredi au retour de la salle —
le texte devient activement trompeur. Il s'affiche sur chaque carte de repas
vide et dans l'en-tête de la feuille de sélection.

Proposition : `'Express ou posé, 15 min max.'`

**`panuozzo-poulet-cesar` ne peut pas apparaître en collation-trajet.** Son
`temps_actif` est de 20 minutes, le budget du créneau est de 15. Elle
déclare pourtant `collation-trajet` dans ses créneaux. Soit le budget est
trop bas, soit la déclaration est de trop — mais aujourd'hui la donnée dit
une chose et le filtre en fait une autre, silencieusement. Anomalie
préexistante, pas introduite par cette session.

**`tortilla-petits-pois-comte` est exactement à la limite** (15 min pour un
budget de 15). Elle passe si le filtre est `temps_actif <= budget`, ce que
dit le journal. À vérifier au moment du test : si elle n'apparaît jamais sur
un créneau de petit-déjeuner, c'est que la comparaison est stricte.

---

## 8. Ordre de traitement recommandé

1. **Bug 1** — sans recettes, rien d'autre n'est testable.
2. **Bug 3** — débloque la liste de courses et le chaînage, jamais exercés.
3. **Bug 2** — cosmétique mais visible à chaque ouverture de la feuille.
4. **Protocole de test complet** — c'est ici que les sessions 1 et 2
   deviennent réellement validées, pour la première fois.
5. **Manque 5** (`defaut-reconduit`) — plus gros gain d'usage par ligne de
   code du projet.
6. **Manque 4** (écran de bilan) — trancher d'abord la question du stockage
   du §4, puis livrer le verdict seul.

Les points 1 à 4 sont la condition d'entrée de la session 3. Bâtir Supabase
par-dessus une liste de courses jamais exercée reviendrait à migrer du code
dont personne ne sait s'il fonctionne.
