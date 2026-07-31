# Prompt à donner à Claude Code — session 3a

**Boucler l'exécution.** Dates réelles, chaîne créneau → cuisine → bilan →
créneau, et le raccourci vers le repas de ce soir.

Supabase et le journal de dépenses passent en session 3b, après. La raison est
en tête de ce fichier, pas au milieu : cette session **change le modèle de
données** (dates, réservation contre consommation). Construire un schéma
Supabase par-dessus un modèle qu'on est en train de modifier obligerait à le
migrer deux fois.

---

Tu travailles sur `LazEat-app`. Lis, dans cet ordre :

1. `CLAUDE.md` et `DESIGN.md` à la racine
2. `../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — huit pièges qui ont
   chacun déjà coûté du temps sur ce projet. **Le §8 s'applique à toi
   directement.**
3. Les trois rapports de `../LazEat/00-Contexte/rapports/`

Le coffre `../LazEat` est la source de vérité des données et des specs. Tu le
lis, tu n'y écris jamais — sauf ton rapport de fin de session.

La phase 2 est close : la checklist manuelle de clôture est passée en entier,
un seul écart, qui est le point A ci-dessous.

## Ce que cette session doit rendre possible

> « Mercredi soir, je rentre. J'ouvre l'app, elle sait qu'on est mercredi,
> elle me montre le repas de ce soir, je touche, je cuisine. À la fin je dis
> comment c'était et le créneau passe à "fait". »

Rien de tout ça ne fonctionne aujourd'hui, et c'est le problème d'origine du
projet — 22h30, qu'est-ce que je mange.

## A — Réserver n'est pas consommer

**Le bug, trouvé au test manuel.** `PlanningPage` appelle
`consommerPortionStockee()` **au moment de l'affectation**. `viderCreneau()`
supprime l'affectation sans rien restituer. Résultat : affecter décrémente,
désaffecter ne rend rien, réaffecter décrémente encore. En basculant trois
fois sur le même créneau, le stock du frigo se vide sans qu'un seul repas ait
été mangé.

**La spec dit déjà l'inverse**, à deux endroits :

- `spec-planning-modifiable.md` §7 : « un créneau supprimé **libère la
  portion qui lui était réservée**, qui redevient disponible pour un autre
  jour ». Le mot est *réservée*.
- `contexte-projet-meal-planner.md` §5.3 : « le décrément se déclenche à la
  **validation de la recette**, pas à l'achat ni à la planification ».

**Attendu.** Affecter une portion la **réserve**. Vider le créneau la libère.
Seule la validation après cuisine la consomme — ce qui n'est possible qu'une
fois le point C posé, d'où l'ordre de cette session.

Second symptôme du même défaut : la carte du planning n'était pas rafraîchie,
parce qu'elle reflétait un état qui n'aurait pas dû changer à cet instant.

## B — Les dates réelles

**Commence par lire `idSemaineCourante()` dans `semaine.ts`.** Une partie du
travail est déjà faite et ce n'est pas évident : la fonction calcule la
**semaine ISO réelle** depuis `new Date()`, et l'identifiant de semaine vaut
déjà `cours:2026-S31`. L'application est donc déjà ancrée dans le calendrier.

Ce qui manque est plus petit qu'il n'y paraît :

- une fonction pure `jour de semaine + id de semaine → date calendaire`
  (lundi de la semaine ISO, plus n jours)
- « aujourd'hui » : quel `JourSemaine`, et `null` si la semaine affichée n'est
  pas la semaine courante
- l'affichage de la date réelle sur l'en-tête du jour

**Ne construis pas un modèle de dates plus gros que ça.** Résiste à
l'envie de faire porter une date à chaque créneau : la date se **dérive** de
l'identifiant de semaine et du jour, elle ne se stocke pas. Stocker
introduirait une seconde source de vérité à resynchroniser, exactement ce que
`spec-planning-modifiable.md` §3 interdit pour les créneaux eux-mêmes.

**Point de vigilance.** `CreneauCalcule.id` vaut `${jour}-${type}-${index}` et
sert de clé de persistance aux affectations. Il est déjà cantonné à une
semaine, puisque les affectations vivent dans une `SemainePlanifiee` indexée
par l'identifiant de semaine. **N'y touche pas** : le journal des sessions 1-2
§6 avertit qu'un changement de format orpheline silencieusement toutes les
affectations existantes.

## C — La chaîne créneau → cuisine → bilan → créneau

**Le cœur de la session.**

La route est `/recettes/:id/cuisine`. Elle porte la recette, jamais le
créneau. En sortant du mode cuisine, l'écran de bilan peut écrire la note et
le nombre d'exécutions, mais il ne peut **rien dire au créneau d'où l'on
vient**. C'est exactement pourquoi l'état « faite » existe dans
`EtatCarteRepas` et dans `DESIGN.md` depuis la session 1 sans avoir jamais
été déclenché une seule fois.

**Attendu.** Le mode cuisine sait de quel créneau il vient, et à la validation :

1. le créneau passe à l'état **« faite »**
2. la portion stockée réservée est **consommée** (le point A devient vrai)
3. le bilan est enregistré comme aujourd'hui

Lancer le mode cuisine depuis la bibliothèque, sans créneau, doit continuer de
fonctionner — le bilan s'enregistre, aucun créneau n'est touché.

**Ne décrémente pas le stock d'ingrédients.** Le moteur de stock au grain de
l'ingrédient reste hors périmètre (`CLAUDE.md`). Cette session pose le
déclencheur, pas le décrément.

**Valide un plan avec moi avant de coder cette partie.** C'est la seule qui
touche au routage et à la forme des affectations.

## D — Le repas de ce soir

`spec-navigation.md`, « Point laissé ouvert » : à 22h30 le chemin fait quatre
gestes — Planning, trouver le bon jour, toucher le créneau, lancer le mode
cuisine. C'est trop pour le moment précis où l'app doit gagner contre la
livraison.

Une fois B posé, le raccourci devient possible. **Propose-moi une option, ne
choisis pas seul** : l'onglet Planning qui s'ouvre sur aujourd'hui, une carte
mise en avant, ou autre chose. Le critère est le nombre de gestes à 22h30,
pas l'élégance.

## Hors périmètre — ne construis pas

- **Supabase** — session 3b, après cette session, sur un modèle stabilisé
- **Journal de dépenses** — session 3b
- **Décrément du stock d'ingrédients** — demande le moteur de placards
- **Scoring et rotation** — demandent des données d'usage qui n'existent pas
  encore ; cette session commence justement à en produire
- **Suggestion automatique d'une portion dont la DLC approche**
  (`spec-bilan` §3.7) — devient possible avec B, mais reste hors périmètre
  ici. Signale-le si tu penses que c'est presque gratuit une fois B fait

## Méthode

- Le point A est cadré, tu peux y aller — mais il ne devient complet qu'avec
  C. Le point C demande un plan validé avec moi avant tout code.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles` après
  chaque commit.
- **`npm run build`, jamais `npm run dev` seul**, pour toute vérification
  touchant au chargement des données (`PIEGES-ENVIRONNEMENT.md` §2).
- Distingue toujours ce que tu as **exercé** de ce que tu as seulement
  **relu**. Les trois rapports précédents l'ont fait, c'est ce qui les rend
  utilisables.
- Pour tout garde-fou ajouté, vérifie-le **en le laissant échouer**
  (`PIEGES-ENVIRONNEMENT.md` §4).
- Tu ne peux pas ouvrir de navigateur, ne perds pas de temps à essayer
  (§7). Livre une liste de vérification manuelle numérotée à la place.

## Fin de session — pousser fait partie du livrable

Neuf commits sont déjà restés non poussés sur ce projet pendant que Vercel
servait une version périmée, et des tests manuels ont été déroulés dessus
avant qu'on s'en aperçoive.

**Termine par `git push`**, puis clos ton rapport sur la sortie réelle de :

```
git log --oneline -n <nombre de commits de la session>
git status -sb
```

`git status -sb` doit afficher `## main...origin/main` **sans** `[ahead N]`.
C'est la preuve, pas l'intention. Une session qui se termine sans push n'est
pas terminée.

## Rapport de session

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-3a.md`, seul fichier que
tu as le droit d'écrire dans le coffre. Même structure que les trois
précédents : ce qui a été fait · ce qui a été vérifié et comment · ce qui a
échoué ou reste non vérifié · décisions prises et leur raison · pièges
découverts · demandes vers le coffre · ce qui reste ouvert · **la liste de
vérification manuelle** · **la preuve du push**.

Signale-moi le chemin quand il est écrit.
