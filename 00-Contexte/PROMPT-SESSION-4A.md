# Prompt à donner à Claude Code — session 4a

**Les placards.** Stock par ingrédient, décrément à la validation, liste de
courses qui soustrait ce qu'on a déjà.

---

Tu travailles sur `LazEat-app`. Lis, dans cet ordre :

1. `CLAUDE.md` et `DESIGN.md` à la racine
2. `../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — neuf pièges, chacun a
   déjà coûté du temps. **Le §9 s'applique à toi directement.**
3. `../LazEat/00-Contexte/contexte-projet-meal-planner.md` §4.5, §5.2, §5.3
   et §8.2 — c'est la spec de cette session
4. `../LazEat/00-Contexte/DECISION-MULTI-UTILISATEUR.md`
5. Les six rapports de `../LazEat/00-Contexte/rapports/`

Le coffre est la source de vérité des données et des specs. Tu le lis, tu n'y
écris jamais — sauf ton rapport de fin de session.

Supabase est en production depuis le 2 août 2026, vérifié de bout en bout, y
compris le fonctionnement hors-ligne et la reprise des écritures en attente.

## A — L'entité Placard

Une ligne par ingrédient présent :
`{ingredient_id, quantite, unite, emplacement, date_achat, dlc}`
(`contexte-projet-meal-planner.md` §4.5).

**Les vues ne sont pas des entités.** « Mon Frigo », « Mon Congélateur »,
« Mes Épices », « Mes Fruits », « Mon Placard » sont des **filtres sur le
champ `emplacement`** de l'ingrédient, jamais des tables ou des listes
séparées. La spec est explicite : dupliquer la donnée la désynchroniserait.

`emplacement` existe déjà sur chaque ingrédient du référentiel.

## B — Le décrément

**À la validation de la recette. Jamais à l'achat, jamais à la
planification.** C'est la même règle que pour les portions stockées, corrigée
en session 3a après un bug : affecter réservait, seule la validation
consommait. Le déclencheur existe déjà — la chaîne créneau → cuisine → bilan
livrée en 3a.

Une recette planifiée mais jamais cuisinée ne consomme rien.

**Ne jamais bloquer l'utilisateur en cuisine** (§5.3). Un ingrédient consommé
alors que le stock est à zéro passe en négatif ou marque un écart, mais
n'empêche jamais de continuer. L'ergonomie prime sur l'intégrité des données
à cet instant précis.

### `suivi_stock` gouverne tout

Le référentiel porte déjà ce champ sur chaque ingrédient, à trois valeurs
(§8.2) :

| Valeur | Comportement attendu |
|---|---|
| `precis` | Décrément au gramme. Viandes, légumes, féculents |
| `binaire` | Présent / à racheter. Jamais de quantité. Épices, huiles |
| `ignore` | Hors du système. Ni stock, ni décrément, ni alerte |

**Piège documenté.** `eau` porte `suivi_stock: ignore` **et**
`conditionnement: 0`. En session 2, un chemin qui ne traitait explicitement
que `binaire` a laissé `ignore` tomber dans le calcul normal :
`Math.ceil(x / 0)` = `Infinity`, puis `NaN` affiché dans la liste de courses.
Le même piège attend le décrément. Traite les trois valeurs explicitement.

## C — La liste de courses soustrait le stock

C'est ce que débloque cette session, et l'étape 4 de la règle d'agrégation
(§5.2) : sommer, convertir, **soustraire le stock connu**, arrondir au
conditionnement supérieur, grouper par rayon.

L'arrondi génère mécaniquement du surplus, qui retourne au stock. C'est voulu
— c'est ainsi que le placard se constitue.

## D — La table Supabase — attention

**Piège spécifique à ce projet, et il va te mordre.** Le projet a le
déclencheur « automatic RLS » activé : toute nouvelle table du schéma `public`
reçoit la sécurité par ligne **automatiquement**, mais **sans aucune règle
d'accès**.

Une table avec RLS active et zéro politique refuse tout. Tu obtiendras
`permission denied` sur une table qui existe pourtant, avec des colonnes
correctes, et rien dans le code applicatif ne l'expliquera.

Écris donc, dans `supabase/schema.sql`, sur le modèle exact des six tables
existantes :

- `user_id uuid not null references auth.users (id) default auth.uid()`
- les quatre politiques `select` / `insert` / `update` / `delete`, toutes
  conditionnées à `auth.uid() = user_id`

Le fichier est idempotent (`drop policy if exists` avant chaque `create`) —
garde cette propriété, l'utilisateur le ré-exécute à la main dans l'éditeur
SQL du tableau de bord.

**Le schéma ne peut pas être exécuté depuis ce dépôt** — signale-le dans ton
rapport et donne à l'utilisateur la marche à suivre, comme l'a fait la
session 3c.

## E — Où ça vit dans l'interface

`spec-navigation.md` défend trois onglets et argumente contre un quatrième.
« Mes Placards » a pourtant besoin d'un endroit.

**Propose-moi une option, ne tranche pas seul.** Et signale-le si tu crées une
surface nouvelle : deux besoins en attente y logeraient naturellement — la
**déconnexion**, absente depuis la session 3d, et l'**écran de réglages des
recettes par défaut**, manquant depuis la 2ter. Ne les construis pas de ta
propre initiative, mais dis-le-moi.

## Hors périmètre — ne construis pas

- **Validation du panier après courses** et **rituel de recalage
  hebdomadaire** — session 4b
- **Contrainte des douze boîtes**, contenants — session 4b
- **Décrément automatique des portions stockées de composant** (un pâton
  consommé par une garniture) — demandé par le rapport 2ter, session 4b
- **Scoring, rotation, bonus « ingrédients en stock »** — session 5a, et
  bloqué par l'absence de données d'usage réelles
- **Photos personnelles des recettes** — possible depuis Supabase Storage,
  mais signale-le plutôt que de le construire

## Méthode

- **Valide un plan avec moi avant tout code.** Cette session touche au schéma
  de base de données et à la liste de courses, deux choses déjà en
  production.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles` après chaque
  commit. **Jamais `npm run dev` seul** pour ce qui touche au chargement des
  données (§2).
- Distingue toujours ce que tu as **exercé** de ce que tu as seulement
  **relu**.
- Pour tout garde-fou ajouté, vérifie-le **en le laissant échouer** (§4).
  `git stash` ne marche pas sur un correctif déjà commité — utilise
  `git checkout <commit>^ -- <fichier>`.
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

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-4a.md`, seul fichier que
tu as le droit d'écrire dans le coffre. Structure habituelle, plus la liste de
vérification manuelle, la marche à suivre côté Supabase, et la preuve du push.
