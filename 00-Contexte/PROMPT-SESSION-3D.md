# Prompt à donner à Claude Code — session 3d

Session courte : remplacer le lien magique par un mot de passe.

---

Tu travailles sur `LazEat-app`. Lis `CLAUDE.md`, puis
`../LazEat/00-Contexte/PIEGES-ENVIRONNEMENT.md` — **le §9 s'applique à toi**.

Le coffre `../LazEat` est la source de vérité. Tu le lis, tu n'y écris jamais,
sauf ton rapport de fin de session.

## Pourquoi

La session 3c a livré une connexion par **lien magique**. À l'usage réel, ce
choix ne tient pas :

- Le service d'e-mail intégré de Supabase est bridé à quelques envois par
  heure. La limite a été atteinte dès la première mise en service, entre les
  essais en local, sur Vercel et sur téléphone.
- Supabase documente ce service comme réservé aux tests. Le garder imposerait
  de brancher un vrai SMTP (Resend, Brevo, Mailgun) — une dépendance externe
  de plus pour une application personnelle.
- Le lien doit être ouvert **sur le bon appareil**, sinon la session s'ouvre
  au mauvais endroit.
- Chaque environnement (local, Vercel, réseau local) exige sa propre URL de
  redirection déclarée côté Supabase.

Le lien magique résout un problème que ce projet n'a pas : éviter à des
utilisateurs de retenir un mot de passe. **Application personnelle,
mono-utilisateur, installée en PWA, où l'on se connecte une fois par appareil
et plus jamais ensuite** — `CLAUDE.md`, « est-ce que ça sert ce cas précis ? ».

## Ce qu'il faut faire

**Remplacer `signInWithOtp` par `signInWithPassword`** dans `ConnexionPage`.
Deux champs, e-mail et mot de passe, un bouton.

**Aucune inscription dans l'application.** Pas de `signUp`, pas de formulaire
de création de compte, pas de réinitialisation de mot de passe. Le compte est
créé à la main dans le tableau de bord Supabase, une fois. Un seul utilisateur
existe (`DECISION-MULTI-UTILISATEUR.md`).

Raison technique, pas seulement de périmètre : `signUp` déclenche un e-mail de
confirmation, donc exactement la limite qu'on cherche à fuir.

**Messages d'erreur explicites.** Un mot de passe faux doit le dire. Ne laisse
pas remonter l'erreur brute de Supabase, qui parle d'« invalid login
credentials » en anglais.

**Ne touche pas** à `persistSession: true` ni à `autoRefreshToken: true` dans
`supabaseClient.ts` : c'est ce qui fait qu'on ne se reconnecte pas à chaque
ouverture, et donc ce qui garde le hors-ligne fonctionnel.

**Vérifie qu'il existe un moyen de se déconnecter**, ne serait-ce que discret.
S'il n'y en a pas, propose-moi où le mettre plutôt que de le placer d'autorité.

## Hors périmètre

- Toute fonctionnalité multi-utilisateur (`DECISION-MULTI-UTILISATEUR.md`)
- Inscription, mot de passe oublié, changement de mot de passe dans l'app
- SMTP personnalisé — c'est précisément ce qu'on évite
- Tout le reste de la feuille de route

## Méthode

- Session courte et cadrée, tu peux y aller sans plan préalable.
- Un commit, message en français, préfixe conventionnel.
- `npm run build`, `npm run validate`, `npm run verifier-modeles`.
- Distingue ce que tu as **exercé** de ce que tu as **relu**. Tu ne pourras
  pas exercer la connexion réelle : pas d'identifiants dans cet environnement.
- Tu ne peux pas ouvrir de navigateur (`PIEGES-ENVIRONNEMENT.md` §7). Livre
  une liste de vérification manuelle.

## Fin de session

`git push`, puis clos ton rapport sur la sortie réelle de :

```
git log --oneline -n 1
git status -sb
```

`## main...origin/main` **sans** `[ahead N]`.

## Rapport

`../LazEat/00-Contexte/rapports/RAPPORT-<AAAA-MM-JJ>-3d.md`, seul fichier que
tu as le droit d'écrire dans le coffre. Structure habituelle.

**Ajoute au rapport la marche à suivre côté tableau de bord Supabase** pour
créer le compte et poser le mot de passe — l'utilisateur en aura besoin
immédiatement après ton commit.
