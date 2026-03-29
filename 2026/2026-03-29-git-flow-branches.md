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
