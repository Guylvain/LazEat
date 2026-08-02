# Prompt à donner à Claude Code — session 4a-bis

Quatre correctifs trouvés en usage réel. Session courte.

---

Tu travailles sur `LazEat-app`. Lis `CLAUDE.md`, `DESIGN.md`, puis
`../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — **le §9 s'applique à toi**.

Le coffre est la source de vérité. Tu le lis, tu n'y écris jamais, sauf ton
rapport de fin de session.

Aucun de ces quatre points n'est apparu en relisant du code. Tous viennent
d'une utilisation réelle, et trois portent sur des livrables déclarés
« relus, non exercés » par les sessions qui les ont produits.

## A — L'en-tête du rituel coupe encore le haut de la feuille

**Diagnostic établi, correctif d'une ligne.**

La session 3c a ajouté `hauteurReservee`, mesurée dans `RituelPage` par un
`ResizeObserver` :

```js
setHauteurEnTete(entree.contentRect.height)
```

**`contentRect` exclut le rembourrage.** L'en-tête est déclaré `pt-6 pb-4`,
soit **40 pixels** que la mesure ne compte pas. La feuille réserve donc 40
pixels de trop peu, son sommet glisse sous l'en-tête, et ce qui disparaît est
précisément la première ligne : le nom du créneau et le « Mardi 22:30 » que
la même session venait d'ajouter (§A2). Deux symptômes, une cause.

Mesurer la boîte de bordure :

```js
entree.borderBoxSize?.[0]?.blockSize ?? entree.target.getBoundingClientRect().height
```

Vérifie qu'aucun autre appel à `contentRect` ne traîne ailleurs avec la même
hypothèse.

## B — Le chaînage se tait quand le besoin est couvert

**Comportement juste, communication absente.**

`calculerListeCourses` sort silencieusement de la boucle de chaînage dans
trois cas : la recette productrice est déjà planifiée, une portion stockée la
couvre (`estCouvertParStock`), ou aucun consommateur n'existe.

Constaté en usage : `Bol asiatique poulet croustillant` planifiée, aucun riz
sur la liste de courses, aucun message. La cause était trois portions de
`Riz cuit en quantité` au congélateur — donc la bonne décision. Mais rien à
l'écran ne le disait, et l'utilisateur a cherché un bug pendant une
demi-heure.

**Le silence est une mauvaise réponse quand la raison est intéressante.**

Fais remonter la couverture au lieu de l'avaler. Une ligne du genre
« Riz cuit — couvert par 3 portions au frigo », ou « déjà prévu mercredi ».

**Propose-moi la forme, ne tranche pas seul.** Deux points à peser : ça ne
doit pas encombrer la liste de courses, qui se lit en magasin au pouce ; et
les deux causes n'ont pas la même valeur informative — une portion stockée
est une vraie découverte, une recette déjà planifiée se voit sur le planning.

## C — Le formulaire des placards

Deux défauts, un certain et un à reproduire.

**Certain** : le champ quantité est prérempli à `0`, qu'il faut effacer avant
chaque saisie. Champ vide au départ, ou sélection du contenu à la prise de
focus.

**À reproduire** : l'utilisateur rapporte devoir cliquer ailleurs avant que
« Enregistrer » ne fonctionne — le premier clic semble sans effet. Le code de
`FormulairePrecis` paraît pourtant correct : `onChange` met l'état à jour à
chaque frappe, le bouton lit cet état, aucun `onBlur`, aucun bouton imbriqué.

**Ne devine pas une cause.** Cherche les explications plausibles — perte de
focus sur mobile qui consomme le premier appui, remontage du composant,
propagation d'événement — et dis-moi ce que tu trouves. Si tu ne peux pas
conclure, dis-le et propose une formulation robuste qui rende le problème
impossible quelle qu'en soit la cause.

## D — Aucun moyen de vider une semaine

Seul `viderCreneau` existe, un créneau à la fois. Repartir de zéro demande
aujourd'hui autant de gestes qu'il y a de créneaux remplis, ou de basculer
sur l'autre modèle pour changer d'identifiant de semaine.

Ajoute un vidage global de la semaine affichée.

**Deux exigences.** Une confirmation explicite — c'est destructeur, et le
rituel du dimanche peut représenter dix décisions. Et ça ne doit toucher
**que** les affectations : ni les surcharges de mode, ni les portions
stockées, ni les bilans, ni le placard.

Sur l'emplacement, propose-moi une option.

## Hors périmètre

- Tout ce qui relève de la session 4b : validation du panier après courses,
  rituel de recalage, contenants, décrément des portions de composant
- Le scoring et la rotation — session 5a
- Toute nouvelle fonctionnalité non listée ci-dessus

## Méthode

- A est entièrement cadré, vas-y. B, C et D demandent un échange avec moi
  avant ou pendant.
- Un commit par livrable, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles` après chaque
  commit.
- Distingue ce que tu as **exercé** de ce que tu as seulement **relu**. Trois
  des quatre points de cette session viennent de livrables déclarés « relus,
  non exercés » — c'est la troisième fois qu'un raisonnement juste sur le
  principe se révèle faux dans le détail.
- Tu ne peux pas ouvrir de navigateur (§7). Livre une liste de vérification
  manuelle numérotée.

## Fin de session

`git push`, puis clos ton rapport sur la sortie réelle de :

```
git log --oneline -n <nombre de commits>
git status -sb
```

`## main...origin/main` **sans** `[ahead N]`.

## Rapport

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-4a-bis.md`, seul fichier
que tu as le droit d'écrire dans le coffre. Structure habituelle.
