# Prompt à donner à Claude Code — session 3c

Correctifs du rituel, Supabase, journal de dépenses.

---

Tu travailles sur `LazEat-app`. Lis, dans cet ordre :

1. `CLAUDE.md` et `DESIGN.md` à la racine
2. `../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — neuf pièges, chacun a
   déjà coûté du temps. **Le §8 s'applique à toi directement.**
3. `../LazEat/00-Contexte/DECISION-MULTI-UTILISATEUR.md` — **contrainte de
   schéma obligatoire pour la partie B**
4. `../LazEat/00-Contexte/FEUILLE-DE-ROUTE.md`
5. Les cinq rapports de `../LazEat/00-Contexte/rapports/`

Le coffre est la source de vérité des données et des specs. Tu le lis, tu n'y
écris jamais — sauf ton rapport de fin de session.

## A — Trois correctifs du rituel

Trouvés en usage réel, sur l'écran livré en session 3b.

**A1 — l'en-tête recouvre le haut de la feuille.** Le titre du créneau est
coupé en deux horizontalement. L'en-tête du rituel est en
`sticky top-0 z-[60]` dans `RituelPage` ; `FeuilleSelection` est en
`fixed inset-x-0 bottom-0 z-50 max-h-[88vh]` via portail vers `document.body`.
Deux systèmes de positionnement qui ne se coordonnent pas : la feuille monte
jusqu'à 88 % de la hauteur de fenêtre sans savoir qu'un en-tête occupe le
haut.

La session 3b a résolu le bon problème — l'en-tête doit rester lisible
**pendant** qu'on regarde les propositions — mais en le passant au-dessus elle
a créé le recouvrement inverse. **Réserver la place, ne pas superposer** : en
mode rituel, la feuille doit être bornée sous l'en-tête.

**A2 — le jour n'est jamais affiché.** `FeuilleSelection` montre
`{creneau.heure} · {CONSEIL_CRENEAU[...]}`, jamais le jour. Sans conséquence
dans l'usage normal, où `EnTeteJour` est juste au-dessus. **Bloquant dans le
rituel**, qui saute d'un bout à l'autre de la semaine : « 22:30 » tout seul ne
permet pas de décider d'un repas.

`CreneauCalcule` porte déjà `jour`, et `dateDuJour()` existe depuis la 3a.
Propose-moi la formulation — « Mardi 22:30 » ou « Mardi 5 août · 22:30 » — en
tenant compte du fait que la ligne porte déjà le conseil du créneau.

**A3 — la largeur sur grand écran.** La feuille reste une colonne étroite
(`max-w-lg mx-auto`) sous un en-tête pleine largeur. L'app est pensée mobile
d'abord, mais `CLAUDE.md` dit que l'ordinateur sert au tableau de bord, et le
rituel du dimanche se fait plutôt assis. **Propose, ne tranche pas seul** :
soit on assume un écran de téléphone, soit l'en-tête s'aligne sur la largeur
de la feuille.

## B — Supabase

Remplacer `PersistanceLocalStorage` par une implémentation Supabase, derrière
l'interface `Persistance` **inchangée**. Cette interface est asynchrone depuis
la session 2 précisément pour que ce soit une substitution et non une
réécriture des appelants.

### B1 — Le hors-ligne, à traiter AVANT tout le reste

**C'est le risque principal de cette session, et il est facile à manquer.**

Aujourd'hui `localStorage` fonctionne sans réseau. L'application est une PWA
installable dont le coffre-fort d'usage est précisément le pire endroit pour
le réseau :

- **La liste de courses s'utilise en magasin**, où le signal est souvent
  mauvais ou absent.
- **Le mode cuisine s'utilise à 22h30 en rentrant d'entraînement**, téléphone
  posé sur un plan de travail.

Une migration naïve vers Supabase casse les deux usages principaux de
l'application. Ce serait une régression bien plus grave que le gain apporté
par la synchronisation.

**Présente-moi un plan pour le hors-ligne avant d'écrire une ligne de code
Supabase.** Cache local plus synchronisation, bibliothèque local-first,
lecture optimiste — je veux comprendre le compromis, pas recevoir une
solution.

### B2 — Le schéma porte `user_id` dès le premier jour

**Contrainte non négociable**, motivée dans
`DECISION-MULTI-UTILISATEUR.md` :

- Chaque table à portée personnelle porte un `user_id` **dès sa création**.
- La sécurité par ligne (RLS) est activée **dès le premier jour**.
- Un seul utilisateur existe.

Raison : une colonne et une règle coûtent zéro au moment où le schéma s'écrit,
et coûtent une migration sur données vivantes ensuite. C'est la seule fenêtre
où c'est gratuit.

**Ne construis aucune fonctionnalité multi-utilisateur** : pas de gestion de
comptes, pas de changement d'utilisateur, pas de partage, pas de planning
commun, **aucune méthode ajoutée à l'interface `Persistance`**.

**Une nuance importante et volontaire** : la RLS a besoin d'une session
authentifiée pour identifier l'utilisateur. Un **écran de connexion unique**
est donc nécessaire et ne contredit pas ce qui précède. Fais-le minimal — lien
magique ou e-mail et mot de passe — et propose-moi lequel avant de coder.

### B3 — Ce qui migre

Cinq familles de données vivent aujourd'hui dans `localStorage` :
`SemainePlanifiee`, `ListeCourses`, `Bilans`, `PortionsStockees`,
`Preferences`. Les clés sont plates (`lazeat:semaine:${id}`,
`lazeat:liste-courses:${id}`, …).

**Prévois la reprise des données existantes.** L'utilisateur a des semaines,
des bilans et des portions stockées réels dans son navigateur. Les perdre à la
bascule serait inacceptable — les bilans en particulier sont irremplaçables,
c'est le carburant du scoring de la phase 5.

### B4 — Le pub-sub n'est pas dans le contrat

`ecouterChangementsPersistance` est une commodité de réactivité, délibérément
hors de l'interface `Persistance` (journal des sessions 1-2, §2). Mais
`PlanningPage`, `CoursesPage` et `BarreOnglets` en dépendent depuis la session
2bis, et les écrans ne sont jamais démontés.

**Il doit continuer de fonctionner à l'identique.** Si tu envisages de le
remplacer par le temps réel de Supabase, dis-le-moi avant : ça change le
comportement hors-ligne et ça touche trois écrans.

## C — Journal de dépenses

Photo du ticket, magasin, montant total, saisis à la main. **Pas d'OCR** —
décision prise, les libellés de tickets sont un projet en soi.

La photo passe par Supabase Storage, donc cette partie vient après B.

Volontairement remonté avant le module stock auquel il appartient
logiquement : sa valeur vient de l'accumulation. Plus il démarre tôt, plus la
comparaison avec les **204,82 € dépensés sur Uber Eats en juillet 2026** sera
parlante.

## Hors périmètre

- Toute fonctionnalité multi-utilisateur au-delà du schéma (voir B2)
- Placards, stock, décrément d'ingrédients — sessions 4a et 4b
- Scoring, rotation, « repartir de la semaine dernière » — session 5a
- Palette d'événements, créneau `diner-a-deux` — session 6
- Photos personnelles des recettes : possibles une fois Storage en place,
  mais signale-le plutôt que de les construire

## Méthode

- **A est cadré, commence par là** : c'est petit et l'utilisateur s'en sert
  dimanche prochain.
- **B1 et B2 demandent un plan validé avec moi avant tout code.** Pas de
  schéma écrit avant que le hors-ligne soit tranché.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles` après chaque
  commit. **Jamais `npm run dev` seul** pour ce qui touche au chargement des
  données (`PIEGES-ENVIRONNEMENT.md` §2).
- Les secrets Supabase passent par des variables d'environnement, jamais
  commités. Rappelle-moi ce qu'il faut poser sur Vercel.
- Distingue toujours ce que tu as **exercé** de ce que tu as seulement
  **relu**.
- Pour tout garde-fou ajouté, vérifie-le **en le laissant échouer** (§4).
  Note que `git stash` ne marche pas sur un correctif déjà commité — utilise
  `git checkout <commit>^ -- <fichier>` (piège trouvé en session 3b).
- Tu ne peux pas ouvrir de navigateur (§7). Livre une liste de vérification
  manuelle numérotée.

## Fin de session — pousser fait partie du livrable

Termine par `git push`, puis clos ton rapport sur la sortie réelle de :

```
git log --oneline -n <nombre de commits de la session>
git status -sb
```

`git status -sb` doit afficher `## main...origin/main` **sans** `[ahead N]`.

## Rapport de session

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-3c.md`, seul fichier que
tu as le droit d'écrire dans le coffre. Même structure que les cinq
précédents, plus la liste de vérification manuelle et la preuve du push.
