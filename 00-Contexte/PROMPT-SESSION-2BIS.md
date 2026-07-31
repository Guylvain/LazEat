# Prompt à donner à Claude Code — session 2bis

Copier-coller le bloc ci-dessous dans une session Claude Code ouverte sur
`LazEat-app`.

---

Tu travailles sur `LazEat-app`, la PWA de planification alimentaire. Lis
`CLAUDE.md` et `DESIGN.md` à la racine avant toute chose.

**Commence par lire `../LazEat/00-Contexte/RAPPORT-SESSION-2BIS.md`.** Il
contient le diagnostic complet de cette session : trois bugs avec leur cause
racine vérifiée, deux manques de périmètre, et le protocole de test. Il a été
rédigé depuis le coffre après inspection du code et du `dist/` — les causes
racines sont établies, pas supposées. Ne les rediagnostique pas, vérifie-les
et corrige.

## Périmètre

Cette session est une remise en état **avant** la session 3 (Supabase, dates
réelles, journal de dépenses). Tu ne construis rien de la session 3. Tu ne
touches pas au moteur de matching par score, au stock, ni aux portions
stockées : ils dépendent de données qui n'existent pas encore.

Le coffre `../LazEat` est la source de vérité des **données**. Tu peux le
lire, tu n'y écris jamais — une autre session y travaille en parallèle. Si tu
as besoin d'une modification de donnée, signale-la, ne la fais pas.

## Ce qu'il y a à faire, dans cet ordre

**1. Bug du glob des recettes** (`src/lib/recettes.ts`). L'extglob
`!(a|b).md` n'est pas interprété par Vite au build : zéro recette dans le
bundle de production, alors que `npm run dev` fonctionne. Corriger le motif
en tableau avec négations préfixées `!`, **et ajouter une garde qui lève si
la carte de modules est vide**. Le défaut de fond n'est pas le motif, c'est
le silence.

**2. Bug de réactivité inter-écrans** (`src/pages/CoursesPage.tsx` **et**
`src/pages/PlanningPage.tsx`). Les onglets ne sont jamais démontés, et seul
`BarreOnglets` s'abonne à `ecouterChangementsPersistance`. Les deux pages
sont donc figées à l'état du démarrage. Le bug est symétrique : `Courses`
rate les affectations faites depuis `Planning`, et `Planning` rate le créneau
inséré par `accepterChainage()`. Corriger les deux.

**3. Bug d'empilement de la feuille de sélection**
(`src/components/planning/FeuilleSelection.tsx`). `EcranOnglet` est un
`position: fixed` sans `z-index`, donc il crée un contexte d'empilement qui
enferme la feuille : son `z-50` ne se compare jamais au `z-30` de la barre.
Sortir la feuille par un portail vers `document.body`. Ne pas se contenter de
monter le `z-index` d'`EcranOnglet`.

**4. Dérouler le protocole de test du §6 du rapport**, en entier. C'est le
seul livrable qui rend les sessions 1 et 2 réellement validées — elles ne
l'ont jamais été. Attention : `npm run build`, `npm run validate` et
`npm run verifier-modeles` ne traversent aucun des trois chemins cassés. Ne
les prends pas pour une preuve.

**5. `defaut-reconduit`** (§5 du rapport). `MODE_SELECTION_CRENEAU` existe et
`modeSelection` est peuplé sur chaque créneau, mais rien ne le lit. Implémenter
la reconduction : préférences persistées (une recette par défaut par type de
créneau), préaffectation à la génération de la semaine, et sortie des créneaux
`fixe` du décompte « N/M repas planifiés ».

**6. Écran de bilan, verdict seul** (§4 du rapport). **Ne commence pas avant
d'avoir tranché avec moi la question de stockage du §4** : le gabarit met
`note` et `nb_executions` dans le frontmatter, mais la PWA ne peut pas écrire
dans le coffre. Pose-moi la question, propose ta lecture, attends ma réponse.

## Méthode

- Exige de valider le plan avec moi avant d'écrire du code sur les points 5
  et 6. Les points 1 à 3 sont des correctifs cadrés, tu peux y aller.
- Un commit par correctif, message en français, préfixe conventionnel.
- Ne me dis jamais qu'un correctif fonctionne sans l'avoir exercé. Pour tout
  ce qui touche au chargement des données, la vérification passe par un vrai
  `npm run build` puis une inspection de `dist/`, pas par `npm run dev`.
- Si tu trouves un fichier modifié que tu n'as pas écrit, arrête-toi et
  signale-le : deux incidents d'écriture concurrente ont déjà eu lieu sur ce
  projet.

## Rapport de session — obligatoire

À la fin de chaque session, écris un rapport dans
`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-<nom-court>.md`.

**C'est le seul fichier que tu as le droit d'écrire dans le coffre.** Il est
lu par la session qui travaille sur les données, et par toute session future
qui reprendra le projet sans contexte.

Structure attendue :

```markdown
# Session <nom> — <date>

## Ce qui a été fait
Un paragraphe par livrable. Commit associé.

## Ce qui a été vérifié, et comment
La commande ou le geste exact, et son résultat. Distinguer ce qui a été
*exercé* de ce qui a seulement été *relu*.

## Ce qui a échoué ou reste non vérifié
Sans euphémisme. Un test non fait est un test non fait.

## Décisions prises et leur raison
Uniquement ce qui n'est pas déductible en lisant le code.

## Pièges découverts
Ce qui ferait retomber une session future dans le même trou.

## Demandes vers le coffre
Modifications de données nécessaires que tu n'as pas faites, avec la raison.

## Ce qui reste ouvert
```

Écris ce rapport pour quelqu'un qui n'a aucun contexte. Si un fait est
déductible en lisant le fichier concerné, ne le répète pas — c'est la
convention déjà suivie par `JOURNAL-SESSIONS-1-2.md`.

Signale-moi le chemin du rapport quand il est écrit.
