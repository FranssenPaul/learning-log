# Learning Log — Brousseau & les situations didactiques

**Date :** 2026-04-12  
**Thème :** Didactique des mathématiques — Théorie des Situations Didactiques (TSD)  
**Source :** Échange avec Claude / réflexion personnelle  
**Connexions :** Hofstadter, pédagogie par analogie, Think.Play

---

## 🧩 Point de départ — Le paradoxe de la transmission

Brousseau part d'un inconfort fondamental :

> **Si l'enseignant explique trop bien, l'élève n'apprend pas vraiment.**

Ce qu'on appelle "bien expliquer" peut précisément court-circuiter la construction du sens. L'élève reçoit un savoir clé-en-main — correct, clair, bien formulé — mais qui ne lui *appartient* pas. Il peut le reproduire, pas le mobiliser.

**L'analogie qui m'aide :** lire la solution d'un bug vs debugger soi-même. Même si je comprends la solution lue, je n'ai pas parcouru le chemin. Le chemin, c'est le savoir.

---

## 🏗️ Le concept central : la Situation Didactique

Brousseau propose de penser l'enseignement non comme une **transmission** mais comme une **ingénierie de situations**. On ne transmet pas un savoir — on conçoit un environnement dans lequel l'élève est *contraint* de le construire.

Trois phases successives :

### 1. Situation d'action
L'élève agit sur un **milieu** (problème, jeu, dispositif). Il développe des **stratégies implicites**, souvent intuitives, pas encore verbalisables.

- Il fait, il tâtonne, il essaie
- L'enseignant n'intervient pas
- C'est "bruyant" cognitivement — et c'est voulu

> *Analogie code :* écrire du code exploratoire avant de comprendre l'architecture. Le tâtonnement n'est pas du temps perdu — c'est le travail réel.

### 2. Situation de formulation
L'élève doit **communiquer** ses stratégies à d'autres (binôme, groupe). Cette contrainte l'oblige à :

- Expliciter ce qu'il faisait intuitivement
- Trouver des mots, une structure, une forme transmissible
- Distinguer ce qu'il *sait* de ce qu'il *croit* savoir

> La verbalisation est une épreuve de vérité. On ne peut pas mentir au langage très longtemps.

### 3. Situation de validation
L'élève doit **convaincre** — prouver, débattre, réfuter. C'est ici que le **conflit cognitif** joue à plein.

Une contradiction apparaît :
- entre sa représentation et le comportement réel du milieu
- ou entre son argument et celui d'un pair

Il doit *résoudre* cette contradiction — ce qui oblige une **restructuration** du schème initial.

---

## 🌍 Le milieu didactique (*milieu*)

C'est le concept-clé de toute la théorie.

Le milieu, c'est **tout ce qui résiste à l'élève** : le problème lui-même, les données, les outils, les camarades. Un bon milieu est **rétroactif** — il donne des feedbacks naturels, sans que l'enseignant soit l'arbitre.

**Exemple classique :** la *course à 20* (qui atteint 20 en ajoutant 1 ou 2 à tour de rôle). L'élève découvre la stratégie gagnante (multiples de 3) par l'échec répété face au milieu — pas par explication. Le milieu *parle*.

**Propriétés d'un bon milieu :**
- Résiste (sinon pas d'apprentissage)
- Rétroagit (l'élève peut lire ses erreurs)
- Est *a-didactique* : le feedback vient du problème, pas de l'enseignant

> *Analogie :* un bon kata de code, un test unitaire qui échoue, un contre-exemple en maths — ce sont des milieux. Ils résistent et rétroagissent.

---

## 📜 Le contrat didactique

C'est le côté plus subtil — et plus sombre — de la théorie.

Il existe un **contrat implicite** entre enseignant et élève :

- L'élève s'attend à ce que l'enseignant lui fournisse les clés du succès
- L'enseignant s'attend à ce que l'élève joue le jeu attendu

Ce contrat **parasite** l'apprentissage : l'élève cherche à *deviner ce que l'enseignant veut* plutôt qu'à s'affronter au problème réel. Il lit les signaux sociaux au lieu de lire le milieu.

**Exemples de contrat didactique en action :**
- L'élève recopie la démarche du prof sans la comprendre (et ça marche à l'évaluation)
- En voyant que l'exercice est dans le chapitre "Pythagore", l'élève sait qu'il faut utiliser Pythagore — sans avoir besoin de comprendre *pourquoi*
- L'élève demande "c'est bon ?" à chaque ligne, cherchant la validation plutôt que la vérité

**La dévolution** : moment clé où l'enseignant *transfère* la responsabilité du problème à l'élève. Si la dévolution réussit, l'élève prend le problème comme *sien*. Si elle échoue, il attend.

> La dévolution, c'est rompre le contrat de façon assumée et bienveillante.

---

## 🏛️ L'institutionnalisation

Après que l'élève a construit la connaissance dans la situation, l'enseignant intervient pour :

- **Nommer** ce qui a été découvert
- **Formaliser** : donner la forme culturelle, mathématiquement rigoureuse
- **Situer** dans le corpus du savoir : "ce que vous venez de faire, ça s'appelle..."

Sans institutionnalisation → la connaissance reste locale, personnelle, non transférable.  
Sans les phases précédentes → l'institutionnalisation est vide, un nom sans substrat.

> *Analogie :* apprendre le mot "récursion" avant d'avoir jamais eu besoin de calculer quelque chose récursivement. Le mot existe, le concept non.

---

## 🗺️ Vue d'ensemble — Le flux

```
Ingénierie du milieu (enseignant en amont)
           ↓
   [Situation d'action]
   L'élève agit, tâtonne, développe des stratégies implicites
           ↓
   [Situation de formulation]
   L'élève verbalise, communique, structure
           ↓
   [Situation de validation]
   L'élève argumente, réfute — conflit cognitif → restructuration
           ↓
   [Institutionnalisation]
   L'enseignant nomme, formalise, situe dans le savoir culturel
```

---

## 🔗 Connexion avec Hofstadter

Pour Hofstadter, comprendre *c'est* trouver une analogie qui fait tenir les choses ensemble. La cognition progresse par **friction entre représentations** : une analogie ancienne craque face à un contre-exemple, et force l'émergence d'une analogie plus fine.

Brousseau dit la même chose pédagogiquement : c'est la **résistance du milieu** qui force la restructuration des schèmes. Le conflit cognitif, c'est le moment où le modèle mental de l'élève rencontre quelque chose qu'il ne peut pas digérer — et doit muter.

**Même mouvement, deux registres :**

| Hofstadter | Brousseau |
|---|---|
| Analogie ancienne | Stratégie implicite de l'élève |
| Contre-exemple / friction | Résistance du milieu |
| Émergence d'une nouvelle analogie | Restructuration du schème |
| Compréhension | Savoir construit |

---

## 💡 Application à Think.Play

Think.Play est naturellement un **générateur de milieux** :

- Le quiz multijoueur *résiste* (mauvaise réponse = feedback immédiat)
- La compétition entre pairs crée la **situation de validation** (l'autre a répondu différemment — pourquoi ?)
- Le classement en temps réel est un milieu *rétroactif* par excellence

**Ce que Brousseau m'invite à penser :**
- Est-ce que mes questions créent un vrai milieu résistant, ou juste une vérification de restitution ?
- Est-ce que je prévois des moments de formulation (un élève explique sa réponse à la classe) ?
- Est-ce que j'institutionnalise après le jeu — ou le jeu s'arrête là ?

**Piste concrète :** après chaque round Think.Play, prévoir une phase "Pourquoi tu as répondu ça ?" — c'est la situation de formulation. Le jeu devient alors un déclencheur du milieu, pas la fin de l'activité.

---

## ❓ Questions ouvertes

- Comment concevoir un milieu numérique qui résiste vraiment, sans devenir frustrant au point de décourager ?
- La rétroaction immédiate (vrai/faux instantané) est-elle un bon milieu, ou court-circuite-t-elle la phase de validation ?
- Dans un cours de maths en Belgique, où est le contrat didactique le plus solide ? (hypothèse : autour de la notation)
- Peut-on faire de la dévolution dans un contexte d'évaluation certificative ?

---

## 📚 Pour aller plus loin

- Brousseau, G. (1998). *Théorie des situations didactiques*. La Pensée Sauvage.
- Chevallard, Y. — **transposition didactique** (le savoir savant → savoir enseigné) : complémentaire à Brousseau
- Vergnaud, G. — **schèmes et concepts-en-acte** : la psychologie cognitive derrière la TSD
- Sensevy, G. — continuation contemporaine, "jeux d'apprentissage"
