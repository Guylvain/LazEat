# Décision — le multi-utilisateur

Prise le 31 juillet 2026, avant la session 3c (Supabase).

**Question posée.** Liam veut aujourd'hui l'application pour lui seul, mais
envisage que sa copine et sa sœur aient chacune un compte, avec leur planning,
leurs semaines et leurs goûts — plus la possibilité de plannings communs.
Faut-il appliquer cette logique dès maintenant ou plus tard ?

---

## La décision

**Ne rien construire du multi-utilisateur. Mais ne pas construire le
mono-utilisateur dans le schéma Supabase.**

La nuance porte toute la décision.

### Ce que la session 3c doit faire

- Chaque table à portée personnelle porte un `user_id` **dès sa création**.
- La sécurité par ligne (RLS) est activée **dès le premier jour**.
- Un seul utilisateur existe : Liam.

### Ce que la session 3c ne doit pas faire

- Aucune interface de compte, d'inscription ou de connexion multiple.
- Aucun changement d'utilisateur.
- Aucun partage, aucun planning commun.
- **Aucune méthode supplémentaire sur l'interface `Persistance`** *au titre du
  multi-utilisateur*.

> **Précision ajoutée le 2 août 2026.** Cette dernière ligne visait le
> périmètre de la session 3c : ne pas introduire de méthode liée aux comptes,
> au partage ou au changement d'utilisateur. Le rapport de la 3c l'a lue comme
> une interdiction universelle, ce qu'elle n'est pas.
>
> Une donnée qui doit vivre hors-ligne **doit** passer par `Persistance` : c'est
> le seul chemin qui écrit dans le cache local avant le réseau. La session 4a a
> donc légitimement ajouté `lirePlacard`/`ecrirePlacard`, le décrément de stock
> se déclenchant à 22h30 au retour d'entraînement — le moment que `CLAUDE.md`
> décrit comme le problème central du projet.
>
> Le critère n'est pas « ajouter une méthode est interdit », c'est : **cette
> donnée doit-elle survivre à une absence de réseau ?** Le journal de dépenses
> répond non — envoyer une photo suppose le réseau de toute façon. Le stock
> répond oui.

---

## Pourquoi maintenant, et seulement ça

Le coût aujourd'hui est quasi nul : une colonne et une règle, au moment précis
où le schéma s'écrit de toute façon.

Le coût si on ne le fait pas : une migration sur des données vivantes, la
sécurité par ligne à rétro-installer, et toutes les clés de persistance à
réécrire.

**C'est cette asymétrie qui décide, pas une préférence.** Et c'est le seul
moment où elle est gratuite — après la 3c, elle ne le sera plus jamais.

Le reste est écarté par la règle de `CLAUDE.md` : « est-ce que ça sert ce cas
précis ? ». Construire trois comptes avant qu'un seul utilisateur ait vécu une
semaine complète, c'est exactement ce que cette règle existe pour empêcher.

---

## Ce qui était déjà fait sans le savoir

Le plus difficile du multi-utilisateur est de séparer **ce qu'une recette
est** de **ce que j'en pense**.

La session 2ter l'a déjà tranché : `note`, `nb_executions` et
`derniere_execution` ont quitté le frontmatter pour la persistance, indexés
par `recette_id`, le markdown ne gardant qu'une valeur initiale.

Ce n'était pas pour préparer le multi-utilisateur — c'était parce que la PWA
ne peut pas écrire dans le coffre. Mais la couche « état personnel par-dessus
donnée canonique » existe déjà, et c'est exactement la forme qu'il faut.

Il suffira d'indexer par `(user_id, recette_id)` au lieu de `recette_id`.

---

## Le reliquat : `statut_personnel`

Un seul endroit mélange encore le personnel et le canonique : le champ
`statut_personnel` du référentiel d'ingrédients.

`miel: exclu`, `sardines: a-tester` — c'est le goût de Liam, écrit dans une
donnée partagée par construction. Le jour où un second compte existe, cette
exclusion devient la sienne aussi.

**Non corrigé aujourd'hui, délibérément** : le déplacer demande de retoucher
le filtre de matching et le référentiel entier, pour ne rien servir tant qu'un
seul utilisateur existe. Signalé dans `_gabarit-recette.md` pour que personne
ne bâtisse davantage dessus entre-temps.

Le jour venu, la migration est mécanique : `statut_personnel` sort du YAML
partagé et devient une table `(user_id, ingredient_id, statut)`.

---

## Le vrai problème difficile : le planning commun

Les comptes ne sont pas la partie compliquée. Le partage l'est, et il pose des
questions sans bonne réponse théorique :

- **Un dîner partagé, quels goûts le filtrent ?** L'intersection des
  exclusions ? Le socle rétrécit vite : Liam exclut le miel, quelqu'un d'autre
  exclut autre chose, et il ne reste plus grand-chose.
- **Les restes vont dans quel frigo ?** Les portions stockées sont attachées à
  un utilisateur ou à un foyer ?
- **Les ingrédients sur quelle liste de courses ?**
- **Un repas commun apparaît-il dans les deux plannings, ou dans un seul ?**
- Les objectifs biologiques sont personnels — celui de Liam vise les folates
  et les oméga-3. Un repas partagé suit lesquels ?

Aucune ne se tranche sur le papier. Elles se répondent en essayant.

**Le socle a déjà commencé à y penser.** `cuisine_a_deux` découpe onze
recettes en deux postes parallèles, avec l'idée explicite de deux téléphones
affichant chacun ses étapes. Le créneau `diner-a-deux` existe. Le projet
anticipe déjà **deux personnes qui cuisinent** — il lui manque seulement deux
comptes.

---

## Ce qui ferait changer cette décision

Un seul critère, et il n'est pas technique : **si quelqu'un d'autre veut
réellement l'utiliser dans les semaines qui viennent**, pas dans six mois.

Alors l'authentification passe en 3c avec le schéma, et ça devient une vraie
session avec son propre périmètre.

Tant que la réponse est « peut-être un jour », le schéma ouvert suffit.
