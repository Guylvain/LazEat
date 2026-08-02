# Pièges d'environnement

Ce que la machine, le navigateur ou l'outillage font dans ton dos. Chacun de
ces pièges a déjà coûté du temps sur ce projet — plusieurs plus d'une fois.

**À fusionner dans `CLAUDE.md` de `LazEat-app`.** Ce fichier vit dans le
coffre pour être lisible par toute session, y compris celles qui n'ouvrent
que le coffre.

---

## 1. Le service worker ressert un build périmé

**Le plus coûteux à ce jour.** `vite-plugin-pwa` est configuré en
`registerType: 'autoUpdate'` **et** `devOptions: { enabled: true }` — donc le
service worker tourne aussi en développement, précache le build, et continue
de le servir après une recompilation.

**Comment ça se manifeste** : tu corriges quelque chose, tu rebuild, et
l'application se comporte exactement comme avant. Aucune erreur nulle part.

Constaté le 31/07/2026 : cinq tests de la checklist de clôture déclarés en
échec alors que le code et le `dist/` étaient corrects — le navigateur
exécutait le build d'une session antérieure.

**Règle : avant tout test manuel, purger.** DevTools → Application → Service
Workers → *Unregister*, puis Storage → *Clear site data*, puis rechargement
forcé.

**Et pour trancher en dix secondes** : ouvrir la même URL en navigation
privée. Pas de service worker, pas de cache. Si le comportement y est
correct, c'était le cache.

**Piste de correction** : passer `devOptions.enabled` à `false`. Le service
worker n'a rien à faire en développement, et il ne teste rien qu'on ne
puisse tester sur un `preview`.

## 2. `npm run dev` ne prouve rien sur le chargement des données

`src/lib/recettes.ts` utilise `import.meta.glob`. Le serveur Vite le résout à
la volée à chaque requête, le build le résout via Rollup — **et les deux
n'acceptent pas les mêmes motifs**. Un glob peut donc fonctionner
parfaitement en développement et ne matcher aucun fichier en production, sans
la moindre erreur.

C'était le bug de la bibliothèque vide : `npm run dev` vert, Vercel vide,
`npm run build` réussissant sans broncher.

**Règle** : toute vérification touchant au chargement des recettes ou du
référentiel passe par un vrai `npm run build`, puis une inspection de
`dist/` :

```
grep -c "Bol yaourt" dist/assets/*.js
```

Jamais par `npm run dev` seul.

## 3. Les trois commandes de vérification ont des angles morts

`npm run build`, `npm run validate` et `npm run verifier-modeles` peuvent
toutes passer au vert sur une application entièrement cassée. Elles l'ont
fait.

| Commande | Ce qu'elle ne voit pas |
|---|---|
| `tsc` / `npm run build` | Un glob qui ne matche rien · les contextes d'empilement CSS · une classe Tailwind pointant vers un token inexistant |
| `npm run validate` | Tout ce qui passe par le bundler : elle tourne sous `tsx`, hors de Vite |
| `npm run verifier-modeles` | Ce qu'elle ne compare pas explicitement — elle a longtemps vérifié le *nombre* de créneaux sans leur *type*, ce qui a laissé passer la confusion `collation-trajet` / `recharge-express` |

**Règle** : un voyant vert n'est jamais une preuve qu'une chose non testée
fonctionne. Distinguer systématiquement ce qui a été **exercé** de ce qui a
été **relu**.

## 4. Un garde-fou doit être vérifié en le laissant échouer

Un test de non-régression qui passe ne dit rien tant qu'on ne l'a pas vu
échouer sur le bug qu'il est censé attraper.

**Règle** : `git stash push -- <le fichier du correctif>`, relancer le test,
vérifier qu'il échoue **à l'endroit attendu**, restaurer. Trente secondes,
et ça distingue « le test passe parce que le code est bon » de « le test
passe quoi qu'il arrive ».

Appliqué en session 2quater, à reproduire pour tout correctif accompagné
d'un garde-fou.

## 5. Les fins de ligne polluent les diffs

Sous Windows, un éditeur ou OneDrive réécrit les fichiers en CRLF après un
commit en LF. `git status` affiche alors des fichiers modifiés dont le
contenu est identique — six fichiers, 399 insertions et 399 suppressions,
pour rien.

Le danger n'est pas le bruit : c'est qu'une **vraie** modification s'y cache.

**Règle** : `.gitattributes` avec `* text=auto eol=lf` à la racine, posé
**avant** tout `git add`. Puis `git add --renormalize .`.

## 6. `fs.cpSync` échoue sous OneDrive

`fs.cpSync` applique un `chmod` sur chaque fichier écrit. Windows le refuse
avec `EPERM` dès que le dossier est géré par OneDrive ou surveillé par un
antivirus. `sync-vault.mjs` plantait systématiquement pour cette raison.

**Règle** : `readFileSync` + `writeFileSync`, jamais d'opération touchant aux
permissions. Déplacer les dépôts hors du Bureau serait mieux encore.

## 7. Une session agentique ne peut pas ouvrir de navigateur

L'environnement de Claude Code ne joint pas `localhost`. Confirmé sur
plusieurs sessions, plusieurs tentatives. **Ne pas re-tenter** sans raison de
penser que ça a changé.

Conséquence structurelle : **aucun rendu visuel n'est jamais vérifié par
l'agent**. Toute session qui touche à un écran doit livrer une liste de
vérification manuelle numérotée — geste exact, résultat attendu — plutôt que
d'affirmer que ça marche.

Pour exercer du vrai code contre les vraies données sans navigateur :

```js
import { createServer } from 'vite';
const server = await createServer({ configFile: 'vite.config.ts', server: { middlewareMode: true } });
await server.ssrLoadModule('/scripts/mon-diagnostic.mts');
await server.close();
```

Nécessaire parce que `recettes.ts` et `ingredients.ts` utilisent des imports
spécifiques à Vite qu'un `tsx` nu ne résout pas. Si le script touche à
`persistance.ts`, prévoir en plus un polyfill `localStorage` en mémoire.

## 8. `Clear site data` et les écritures pas encore synchronisées

**Danger fortement réduit depuis la mise en service de Supabase**, le
2 août 2026 — la synchronisation a été vérifiée de bout en bout, écriture
hors-ligne comprise. Les données vivent désormais dans Postgres ;
`localStorage` n'est qu'un cache.

**Ce qui reste vrai.** La file d'attente de synchronisation
(`lazeat:file-attente-sync`) vit dans `localStorage`. Purger le stockage
alors que des écritures n'ont pas encore été remontées — typiquement après
une session hors-ligne — les perd définitivement.

**Règle** : ne jamais purger sans être en ligne et sans avoir laissé passer
une minute. `Service Workers → Unregister` seul reste de toute façon
suffisant pour le problème du §1, et ne touche pas au stockage.

**Ce qui l'était avant, et qui vaut d'être retenu comme principe.** Entre les
sessions 2ter et 3c, les bilans, portions stockées et semaines n'existaient
**que** dans `localStorage` — le frontmatter des recettes ne contient que des
valeurs *initiales*. Le protocole de purge du §1 les aurait détruits sans
recours.

Ni le protocole ni la migration n'étaient fautifs séparément : c'est leur
rencontre qui l'était. À reconsidérer chaque fois qu'une donnée locale devient
la seule copie de quelque chose.

## 9. Commiter n'est pas pousser

Arrivé deux fois. Neuf commits des sessions 2ter et 2quater sont restés sur
la machine pendant que Vercel servait toujours la version de fin de session
2bis — et les tests manuels ont été déroulés sur cette version périmée avant
qu'on s'en aperçoive.

**Règle** : `git push` fait partie du livrable, pas du rangement. Une session
qui se termine sans push n'est pas terminée.

Le rapport de session doit se clore sur la sortie de :

```
git log --oneline -n <nombre de commits de la session>
git status -sb
```

`git status -sb` affiche `## main...origin/main` sans `[ahead N]` quand tout
est poussé. C'est la preuve, pas l'intention.

## 9. Une seule session agentique à la fois par dépôt

Deux incidents d'écriture concurrente ont déjà produit des fichiers en double
et des clés YAML dupliquées dans le référentiel.

**Règle** : une session par dossier. Si une session trouve un fichier modifié
qu'elle n'a pas écrit, elle s'arrête et le signale au lieu de l'écraser.

---

## Le protocole minimal avant un test manuel

1. `git push` — sinon Vercel sert une version antérieure
2. `npm run build` puis `npm run preview`
3. Purger le service worker, ou ouvrir en navigation privée
4. **Vérifier d'abord un repère connu** avant de dérouler quoi que ce soit :
   sur mardi en semaine de cours, le sélecteur doit afficher **4 points** et
   un ratio **sur 4**. Cinq points ou un ratio sur 3 signifient que tu
   regardes un build antérieur à la session 2quater — inutile de tester quoi
   que ce soit d'autre.
