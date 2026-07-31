# Prompt à donner à Claude Code — session 3b

**Le rituel du dimanche.** Séparer le moment où l'on décide du moment où l'on
exécute.

Supabase et le journal de dépenses passent en session 3c. La raison est en
tête : la friction constatée à l'usage est un problème d'écran, pas
d'appareil. Supabase ne change rien de visible tant que tout se fait depuis
un seul terminal ; le rituel, lui, sert dès dimanche prochain.

---

Tu travailles sur `LazEat-app`. Lis, dans cet ordre :

1. `CLAUDE.md` et `DESIGN.md` à la racine
2. `../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — neuf pièges qui ont
   chacun déjà coûté du temps. **Le §8 s'applique à toi directement.**
3. `../LazEat/00-Contexte/spec-planning-modifiable.md` §3ter et §3quinquies
4. `../LazEat/00-Contexte/contexte-projet-meal-planner.md` §3.1, « Parcours A »
5. Les quatre rapports de `../LazEat/00-Contexte/rapports/`

Le coffre est la source de vérité des données et des specs. Tu le lis, tu n'y
écris jamais — sauf ton rapport de fin de session.

## Le problème, constaté à l'usage

**Un seul écran fait deux métiers opposés.**

Le dimanche soir, l'utilisateur **décide** : il veut une vue d'ensemble,
comparer, repérer les trous, remplir une dizaine de créneaux d'affilée, et
sortir avec une liste de courses. Beaucoup d'information d'un coup.

Le mercredi à 22h30, il **exécute** : une seule chose, zéro décision, un
geste. Une seule information.

Ces deux besoins demandent l'inverse l'un de l'autre. Aujourd'hui c'est le
même onglet Planning, le même parcours jour par jour à travers sept
pastilles. La session 3a a réglé le côté exécution — Planning s'ouvre sur
aujourd'hui — mais ça n'aide pas le dimanche, où il faut justement parcourir
toute la semaine.

**Et la promesse de la spec n'est pas tenue.** `spec-planning-modifiable.md`
§3ter annonce « le rituel du dimanche passe d'une corvée de vingt minutes à
environ cinq minutes ». La reconduction automatique existe depuis la session
2ter, mais le rituel lui-même n'existe pas : planifier revient à se promener
dans l'écran qui sert aussi à exécuter. Le « Parcours A » de
`contexte-projet-meal-planner.md` §3.1 commence par « il ouvre la vue
*Nouvelle semaine* » — cette vue n'existe pas.

## A — Le rituel

### Ne construis pas une nouvelle interface de sélection

`FeuilleSelection` existe, fonctionne, et affiche déjà les recettes
compatibles avec leur historique et leurs étiquettes. Le rituel est **un
contrôleur qui l'enchaîne** sur une file de créneaux, avec une progression et
la liste de courses au bout.

Si tu te retrouves à dessiner un nouveau composant de carte ou un système de
swipe, tu as pris le mauvais chemin — reviens me voir.

`DESIGN.md` assume d'ailleurs explicitement le choix de la liste contre le
swipe : elle permet de comparer plusieurs options d'un coup d'œil pour
remplir dix créneaux d'affilée, ce que le swipe interdit par construction.

### La file

Seuls les créneaux qui demandent réellement une décision : ceux en
`modeSelection: 'matching'` et encore vides. Les `defaut-reconduit` déjà
préaffectés et les `fixe` en sont exclus — c'est tout l'intérêt.

**Mesure le nombre réel et rapporte-le.** Mon décompte à la main donne 13
créneaux à décider sur 27 en semaine de cours avec match le dimanche. La spec
annonce 10 sur 26, chiffre plus ancien. Ne recopie ni l'un ni l'autre :
compte-les par exécution sur les deux modèles, avec et sans match, et mets le
résultat dans ton rapport.

Ordre chronologique, lundi vers dimanche. C'est ainsi qu'on pense sa semaine.

### Le déroulé

- Un point d'entrée depuis Planning. **Propose-moi son emplacement et son
  libellé, ne choisis pas seul** — `spec-navigation.md` défend trois onglets
  et argumente contre un quatrième, à toi de voir si le point d'entrée est un
  bouton, une carte en tête d'écran, ou autre chose.
- Une progression visible : où j'en suis, combien il reste.
- **Passer doit être de premier ordre**, pas un échec. `contexte-projet` §3.1
  est explicite : « la planification partielle est un cas d'usage normal ».
  Planifier trois jours et s'arrêter doit fonctionner sans dégrader le reste.
- Revenir en arrière sur le créneau précédent.
- **Au bout, la liste de courses.** C'est la récompense et ce qui referme la
  boucle : on ne sort pas du rituel sur un écran vide, on en sort avec ce
  qu'on doit acheter.

### Ce que le rituel ne fait pas

Il ne modifie pas les événements de la semaine, ne bascule pas les modes de
jour, ne touche pas au modèle. Il ne fait que remplir des créneaux. Tout le
reste continue de passer par l'écran Planning.

## B — La sur-réservation

`portionsCompatiblesCreneau` ne filtre que sur `portionsRestantes > 0`. Elle
ne tient pas compte des portions **déjà réservées sur un autre créneau** de la
semaine.

Conséquence : une portion de chili placée mercredi est encore proposée jeudi.
Deux créneaux réclament une portion qui n'existe qu'une fois. Le compte
persisté reste exact — ce n'est pas une corruption — mais la section « Déjà
prêt au frigo » ment sur ce qui est réellement disponible, soit exactement ce
qu'elle existe pour éviter.

**Ce point devient critique avec A** : c'est pendant le rituel, en enchaînant
dix créneaux d'affilée, que la même portion sera proposée plusieurs fois.

La fonction doit recevoir la `SemainePlanifiee` et soustraire les portions
déjà réservées sur des créneaux non encore marqués `faite`.

## Hors périmètre — ne construis pas

- **Supabase et le journal de dépenses** — session 3c
- **« Repartir de la semaine dernière »** (`spec-planning` §3quinquies) —
  suppose un historique de semaines qui n'existe pas encore. Signale-le si tu
  penses que c'est presque gratuit une fois A fait
- **Le scoring** — la file est en ordre chronologique, les recettes restent
  triées par temps actif. Aucune notion de score
- **La modification des événements** (palette, ajout, suppression) — phase 3
  de `spec-planning` §8, hors sujet ici
- **Le décrément du stock d'ingrédients** — demande le moteur de placards

## Méthode

- Le point B est cadré, tu peux y aller. **Le point A demande un plan validé
  avec moi avant tout code**, en particulier le point d'entrée et la forme de
  la progression.
- Commence par B : il est petit, et il rend A correct dès le premier essai.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles` après chaque
  commit. **Jamais `npm run dev` seul** pour ce qui touche au chargement des
  données (`PIEGES-ENVIRONNEMENT.md` §2).
- Distingue toujours ce que tu as **exercé** de ce que tu as seulement
  **relu**.
- Pour tout garde-fou ajouté, vérifie-le **en le laissant échouer** (§4).
- Tu ne peux pas ouvrir de navigateur (§7). Livre une liste de vérification
  manuelle numérotée.

## Fin de session — pousser fait partie du livrable

Termine par `git push`, puis clos ton rapport sur la sortie réelle de :

```
git log --oneline -n <nombre de commits de la session>
git status -sb
```

`git status -sb` doit afficher `## main...origin/main` **sans** `[ahead N]`.
C'est la preuve, pas l'intention.

## Rapport de session

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-3b.md`, seul fichier que
tu as le droit d'écrire dans le coffre. Même structure que les quatre
précédents, plus la liste de vérification manuelle, le décompte réel des
créneaux à décider, et la preuve du push.
