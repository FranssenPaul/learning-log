# Git — Branches, Flow et Pull Requests
*learning-log — gestion de version*

---

## Le problème que les branches résolvent

Sans branches, `main` est constamment en danger. Tu codes une nouvelle idée, ça casse quelque chose, et ta version stable est compromise. Les branches résolvent ça en donnant un espace de travail **isolé** — tu expérimentes sans jamais toucher ce qui fonctionne.

> 💡 **Analogie** : `main` est le document officiel signé. Tu ne griffonnes pas dessus pour tester une idée. Tu fais une photocopie, tu travailles dessus, et seulement quand c'est propre tu mets à jour le document officiel.

---

## Les trois zones à comprendre

Avant même de parler de branches, il faut avoir en tête que Git travaille dans trois zones simultanément :

```
Ton disque dur      →    Index (staging)    →    Repo local    →    GitHub (distant)
  (working dir)          git add                 git commit         git push
```

Chaque branche existe dans ces zones. Quand tu crées une branche, tu crées un **pointeur** vers un état du repo — pas une copie des fichiers. C'est pour ça que basculer entre branches est instantané.

---

## Les commits, HEAD et les branches — sous le capot

### Un commit, c'est quoi exactement ?

Un commit est un **snapshot complet** de ton projet à un instant donné — pas un diff, pas un patch, mais une photo entière. Git calcule un identifiant unique (un hash SHA-1 comme `a3f7c2d9...`) qui représente cet état exact. Si un seul octet change dans un fichier, le hash est différent.

Ces snapshots sont stockés dans le dossier caché **`.git/objects/`** à la racine de ton projet. Tu n'interagis jamais directement avec ce dossier — c'est la base de données interne de Git.

### Le graphe de commits

Chaque commit connaît son **parent** (le commit précédent). Ça crée une chaîne orientée dans le temps :

```
●──────────●──────────●──────────●
a3f7c2     b1d4e9     c8a2f1     d6b3e0
(init)     (login)    (style)    (fix)

← chaque commit pointe vers son parent →
```

Git remonte cette chaîne pour reconstruire l'historique. `git log` fait exactement ça : il part de ta position actuelle et suit les parents jusqu'à l'origine.

### Une branche : juste un pointeur

Une branche n'est **pas** une copie des fichiers. C'est un petit fichier texte dans `.git/refs/heads/` qui contient le hash du dernier commit de cette branche, et rien d'autre.

```
.git/refs/heads/main              → contient "d6b3e0..."
.git/refs/heads/feature/mon-idee  → contient "c8a2f1..."
```

Créer une branche = créer ce petit fichier de quelques octets. C'est pour ça que c'est instantané, et que changer de branche ne déplace aucun fichier — Git remplace juste le contenu de ton working dir par le snapshot pointé.

### HEAD — où tu te trouves en ce moment

`HEAD` est un pointeur spécial qui indique **ta position courante dans le graphe**. Il pointe vers une branche (et non directement vers un commit dans le cas normal).

```
HEAD → main → d6b3e0...
```

Quand tu fais `git switch feature/mon-idee`, HEAD bascule :

```
HEAD → feature/mon-idee → c8a2f1...
```

Quand tu fais un **nouveau commit**, la branche courante avance automatiquement vers ce nouveau commit, et HEAD suit :

```
Avant :  HEAD → feature/mon-idee → c8a2f1...
Après :  HEAD → feature/mon-idee → e9d4f2...  ← nouveau commit
                                       ↑
                              (parent : c8a2f1)
```

### Vue d'ensemble

```
                            HEAD
                              ↓
              feature/mon-idee     main
                     ↓               ↓
●──────────●──────────●             ●
a3f7c2     b1d4e9     c8a2f1        d6b3e0
```

`main` et `feature/mon-idee` partagent les deux premiers commits — ce ne sont pas des copies, c'est le même objet dans `.git/objects/`. La branche ne duplique rien, elle pointe différemment.

---

## `git switch` — la commande dédiée aux branches

Historiquement, Git utilisait `git checkout` pour tout : changer de branche, restaurer des fichiers, inspecter des commits. Trop de responsabilités pour une seule commande.

Depuis Git 2.23, **`git switch`** est dédié exclusivement aux branches :

```bash
git switch main                  # basculer sur une branche existante
git switch -c feature/mon-idee  # créer ET basculer (-c = create)
```

`git checkout` continue de fonctionner, mais `git switch` exprime clairement l'intention. C'est la commande à préférer.

---

## Le nommage des branches

Un nom de branche est du texte libre pour Git. Par convention, on utilise un **préfixe** suivi d'un `/` pour exprimer l'intention :

| Préfixe | Intention | Exemple |
|---------|-----------|---------|
| `feature/` | nouvelle fonctionnalité | `feature/websocket-sync` |
| `fix/` | correction de bug | `fix/login-crash` |
| `hotfix/` | correction urgente en production | `hotfix/paiement-bloque` |
| `refactor/` | réécriture sans changer le comportement | `refactor/api-routes` |
| `docs/` | documentation | `docs/readme-posen` |
| `test/` | expérimentation, prototype | `test/offline-mode` |

Le `/` n'a aucune signification pour Git — c'est un séparateur visuel, comme un dossier. Sur un projet solo, un nom simple sans préfixe suffit largement : `websocket-sync`, `offline-mode`, `schema-v2`.

---

## La Pull Request — demande de fusion

Une **Pull Request** (PR) est une demande formelle de fusionner une branche dans une autre. Sur GitHub, elle sert à :

- **Visualiser** le diff entre la branche et `main`
- **Documenter** pourquoi ce changement a été fait
- **Valider** avant d'intégrer (review, tests)
- **Merger** proprement avec un historique traçable

En équipe, quelqu'un d'autre approuve avant le merge. Seul sur un projet, tu fais la PR et tu la merges toi-même — ça devient un **checkpoint personnel** et une trace dans l'historique GitHub.

> 💡 **Analogie** : la PR c'est le bon de commande avant livraison. Même si tu es seul, tu signes le bon avant de valider — ça force une relecture et ça laisse une trace de la décision.

GitHub appelle les deux rôles dans une PR :
- **base** : la branche cible (`main`) — là où tu veux fusionner
- **compare** : ta branche de travail — ce que tu apportes

---

## `gh` — le CLI officiel de GitHub

**Git** et **GitHub** sont deux choses distinctes. Git est le système de versioning qui tourne en local. GitHub est la plateforme distante qui héberge les repos. Pour interagir avec GitHub depuis le terminal, il faut un outil supplémentaire : **`gh`**, le CLI officiel de GitHub.

Sans `gh`, tu dois ouvrir le navigateur pour créer un repo, ouvrir une PR, la merger. Avec `gh`, tout ça se fait en ligne de commande — sans quitter le terminal.

### Ce que `gh` permet

```bash
# Créer un repo GitHub depuis un dossier local (et le lier)
gh repo create mon-projet --private --source=. --push

# Ouvrir une Pull Request
gh pr create --base main

# Lister les PRs ouvertes
gh pr list

# Merger une PR
gh pr merge

# Voir le statut d'une PR
gh pr status
```

### Relation entre `git` et `gh`

```
git  →  gère l'historique local (commits, branches, index)
gh   →  parle à l'API GitHub (repos, PRs, issues, releases)
```

`git push` envoie tes commits sur GitHub, mais ne crée pas de PR. C'est `gh pr create` qui ouvre la PR — c'est une opération côté GitHub, pas côté Git.

### Installation et authentification

`gh` s'installe séparément de Git (`apt install gh` sur Ubuntu, ou via le site officiel). Au premier lancement, `gh auth login` ouvre un flux d'authentification pour lier ton compte GitHub.

---

## Le flow complet

```bash
# ── INITIALISATION (une seule fois par projet) ──────────────────

# Créer le repo GitHub depuis ton dossier local
gh repo create mon-projet --private --source=. --push
# → crée le repo, lie le dossier local, pousse main sur GitHub


# ── CYCLE DE TRAVAIL (répété pour chaque feature/fix) ───────────

# 1. Créer une branche dédiée
git switch -c feature/mon-idee

# 2. Travailler — autant de commits que nécessaire
git add .
git commit -m "description claire de ce que tu as fait"

# 3. Pousser la branche sur GitHub
git push origin feature/mon-idee

# 4. Ouvrir la Pull Request
gh pr create --base main

# 5. Merger la PR (après relecture)
gh pr merge

# 6. Revenir sur main et synchroniser le local
git switch main
git pull


# ── SI L'IDÉE EST ABANDONNÉE ────────────────────────────────────

git branch -d feature/mon-idee           # supprimer en local
git push origin --delete feature/mon-idee  # supprimer sur GitHub
```

---

## Ce que `git pull` fait à l'étape 6

Après le merge sur GitHub, ton `main` **local** est en retard sur le `main` **distant**. C'est une erreur classique que les débutants oublient.

```
GitHub (distant)   main ──●──●──●──[merge feature]
                                          ↑
Ton disque (local) main ──●──●──●         ← en retard
```

`git pull` = `git fetch` (télécharger) + `git merge` (intégrer). Il resynchronise ton local avec le distant.

---

## Pourquoi ce flow même seul ?

- `main` est **toujours stable** — tu peux revenir à un état qui fonctionne à tout moment
- Chaque branche raconte une **histoire cohérente** dans l'historique
- Si une idée ne marche pas, tu supprimes la branche — aucune trace dans `main`
- Tu prends l'habitude du workflow pro — quand tu collabores, tu n'as rien à réapprendre

Sur POSEN par exemple : `main` reste la version qui tourne au restaurant, pendant que tu développes `feature/manager-approval` ou `fix/offline-sync` en parallèle sans jamais risquer la prod.

---

## GitHub stocke aussi le graphe complet

GitHub héberge un **dépôt Git bare** — c'est-à-dire le contenu de ton `.git/` (objets, refs, historique complet), sans les fichiers de travail. C'est la même base de données que ton local, sur leurs serveurs.

Quand tu fais `git push`, Git ne transfère pas les fichiers — il transfère uniquement les **objets manquants** que GitHub n'a pas encore. Si un commit existe déjà (parce qu'il était dans un push précédent), il n'est pas renvoyé.

`git clone` télécharge l'intégralité du graphe — tous les commits, toutes les branches. Le working dir affiche `main`, mais tous les objets sont déjà dans `.git/objects/`. Tu peux immédiatement `git switch feature/mon-idee` sans rien télécharger de plus.

```
Après git clone
├── .git/objects/   ← tous les commits de toutes les branches
├── .git/refs/      ← toutes les branches (origin/main, origin/feature/...)
└── working dir     ← fichiers de la branche par défaut (main)
```

Les branches distantes apparaissent comme `origin/main`, `origin/feature/...` — ce sont des **remote-tracking branches**, des pointeurs en lecture seule qui reflètent l'état de GitHub au moment du clone ou du dernier `git fetch`.

---

## Authentification avec GitHub

Il y a deux mécanismes distincts selon l'outil :

### Pour `git push` / `git pull` — SSH

Tu génères une paire de clés sur ton PC :
- **clé privée** : reste sur ton disque (`~/.ssh/id_ed25519`), ne quitte jamais ta machine
- **clé publique** : tu la colles dans GitHub (Settings → SSH keys)

Quand tu fais `git push`, ton PC prouve son identité avec la clé privée, GitHub vérifie avec la clé publique — sans mot de passe. C'est le même principe qu'une serrure (publique) et une clé (privée).

### Pour `gh` — token OAuth

`gh auth login` ouvre un flux dans le navigateur, tu autorises l'application, et `gh` stocke un **token d'accès** dans ton trousseau système (`~/.config/gh/hosts.yml`). C'est ce token que `gh` envoie à l'API GitHub à chaque commande.

```
git push      →  authentification SSH    →  protocole Git
gh pr create  →  token OAuth            →  API REST GitHub
```

Les deux coexistent et sont indépendants.

---

## SSH vs HTTPS — deux protocoles de transport

L'URL de ton remote détermine le **canal de communication** utilisé :

| | SSH | HTTPS |
|---|---|---|
| URL | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| Port | 22 | 443 |
| Auth | clé publique/privée | token |
| Usage | usage quotidien recommandé | firewalls qui bloquent le port 22 |

Le contenu transféré est identique dans les deux cas — les objets Git. Seul le canal change.

```bash
git remote -v                                          # vérifier l'URL courante
git remote set-url origin git@github.com:user/repo.git  # passer en SSH
```

---

## Retrieval practice

*Réponds à voix haute ou par écrit, sans relire les notes. Reviens vérifier ensuite.*

**Fonctionnement interne**

1. Un commit, c'est un diff ou un snapshot ? Qu'est-ce que ça change concrètement ?
2. Où Git stocke-t-il physiquement les commits sur ton disque ?
3. Qu'est-ce qu'une branche, vraiment ? Que contient le fichier qui la représente ?
4. Pourquoi créer ou changer de branche est-il instantané ?
5. Qu'est-ce que HEAD ? Vers quoi pointe-t-il normalement ?
6. Que se passe-t-il exactement sur HEAD et sur la branche quand tu fais un nouveau commit ?

**Les trois zones**

7. Cite les quatre zones par lesquelles transitent les données dans Git (working dir inclus).
8. Quelle commande fait passer des fichiers du working dir vers l'index ? De l'index vers le repo local ?
9. `git push` fait passer les données de quelle zone vers quelle zone ?

**Le flow et les branches**

10. Quelle commande crée une branche *et* bascule dessus en une seule opération ?
11. Dans une Pull Request sur GitHub, que désignent les termes "base" et "compare" ?
12. Pourquoi fait-on un `git pull` sur `main` après avoir mergé une PR ?
13. `git pull` est la combinaison de quelles deux commandes ? Que fait chacune ?

**`gh` et GitHub CLI**

14. Quelle est la différence entre `git` et `gh` ? Lequel connaît GitHub, lequel ne le connaît pas ?
15. `git push` et `gh pr create` : ces deux commandes font-elles la même chose ? Dans quel ordre les utilise-t-on et pourquoi ?
16. Que fait exactement `gh pr create --base main` ? D'où vient la branche source ?
17. `--base main` est-il toujours obligatoire ? Pourquoi l'écrire quand même ?

**GitHub et le graphe distant**

18. GitHub stocke-t-il uniquement la dernière version des fichiers, ou quelque chose de plus complet ?
19. Quand tu fais `git push`, Git envoie-t-il tous les commits ou seulement les nouveaux ? Comment le sait-il ?
20. `git clone` donne-t-il uniquement la branche `main`, ou l'intégralité du graphe ? Qu'est-ce que ça implique concrètement ?
21. Qu'est-ce qu'une remote-tracking branch (`origin/main`) ? En quoi est-elle différente d'une branche locale ?

**Authentification et transport**

22. Quels sont les deux outils qui communiquent avec GitHub, et quel mécanisme d'authentification chacun utilise-t-il ?
23. Où est stockée la clé privée SSH ? Où va la clé publique ? Laquelle ne doit jamais quitter ta machine ?
24. Quelle est la différence entre SSH et HTTPS comme protocole de transport pour Git ? Quand utilise-t-on l'un ou l'autre ?
25. Comment vérifier si ton remote est configuré en SSH ou en HTTPS ? Comment le changer ?

**Scénarios**

26. Tu es sur `main`, tu veux travailler sur une nouvelle feature. Écris les commandes du flow complet, de la création de branche jusqu'au merge.
27. Tu as ouvert une branche pour tester une idée qui ne marche pas. Comment tu nettoies proprement, en local et sur GitHub ?
28. Deux branches peuvent-elles partager des commits ? Explique pourquoi ou pourquoi pas.
