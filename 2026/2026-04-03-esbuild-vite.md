# Learning Log — Comprendre esbuild, Vite et Rollup simplement

**Date :** 2026-04-03
**Tags :** `tooling` `build` `javascript` `vite` `esbuild` `rollup`
**Statut :** note de compréhension — version simplifiée

---

## 1. Le point de départ

Quand on débute avec le développement web moderne, on rencontre très vite des noms comme :

- `esbuild`
- `Vite`
- `Rollup`
- `Webpack`
- `Babel`

Au début, cela peut donner l’impression qu’il faut comprendre toute une usine avant même d’écrire une page web.

En réalité, l’idée de fond est assez simple :

> tu écris du code source pour travailler confortablement, puis des outils le préparent pour qu’il soit facile à utiliser dans le navigateur.

Autrement dit :

- toi, tu écris dans une forme pratique pour développer
- les outils transforment cela dans une forme pratique pour exécuter

---

## 2. Pourquoi a-t-on besoin de ces outils ?

Quand on construit une application web, on ne travaille pas toujours avec un seul fichier JavaScript.

On utilise souvent :

- plusieurs fichiers séparés
- des `import` et `export`
- du TypeScript
- parfois du JSX
- des dépendances installées avec `npm`
- des fichiers CSS, images, polices, etc.

Le navigateur moderne comprend déjà beaucoup de choses, mais il ne suffit pas toujours de lui envoyer le projet brut tel quel.

Il faut souvent :

- transformer certains fichiers
- organiser les dépendances
- préparer les fichiers pour la production
- réduire le poids final

On peut résumer cela ainsi :

- **écrire le projet** est une chose
- **préparer le projet pour le navigateur** en est une autre

---

## 3. L’idée de “build”

Le mot **build** désigne simplement la préparation du projet avant sa version finale.

Faire un build, c’est par exemple :

- prendre les fichiers source
- les transformer si besoin
- regrouper ce qui doit l’être
- produire un dossier final prêt à être servi

On peut imaginer ceci :

- `src/` = ce que tu écris
- `dist/` = ce que l’outil prépare pour la version finale

Le build n’est donc pas “de la magie compliquée”.
C’est surtout une étape de préparation.

---

## 4. esbuild, expliqué simplement

### Ce que c’est

**esbuild** est un outil très rapide qui sert à **transformer** et parfois **assembler** ton code JavaScript ou TypeScript.

Son rôle principal est :

- lire tes fichiers source
- les convertir si nécessaire
- produire des fichiers directement utilisables

Par exemple, il peut :

- transformer du TypeScript en JavaScript
- transformer du JSX en JavaScript
- regrouper plusieurs modules
- minifier le résultat final

### Pourquoi on en parle autant ?

Parce qu’il est connu pour sa **vitesse**.

L’idée importante ici n’est pas de retenir dans quel langage il est écrit en interne, mais de comprendre ceci :

> esbuild est surtout apprécié parce qu’il fait très vite un travail de transformation que d’autres outils faisaient plus lentement.

### Comment le voir mentalement ?

Tu peux voir esbuild comme :

- un **moteur très rapide**
- un **outil de transformation**
- une **brique technique**

Ce n’est pas forcément l’outil complet que l’on utilise directement tous les jours pour toute l’expérience de développement.

---

## 5. Vite, expliqué simplement

### Ce que c’est

**Vite** est un outil pensé pour rendre le développement frontend plus simple et plus fluide.

Il aide surtout à deux moments :

- **pendant le développement**
- **au moment de produire la version finale**

Autrement dit, Vite ne sert pas seulement à transformer du code.
Il sert aussi à rendre le travail quotidien plus confortable.

### Pourquoi Vite plaît autant ?

Parce qu’il donne une sensation de légèreté :

- le projet démarre vite
- les changements apparaissent vite
- la configuration reste souvent raisonnable

### Comment le voir mentalement ?

Tu peux voir Vite comme :

- un **atelier de travail**
- un **organisateur**
- un **outil de développement complet**

Là où esbuild ressemble plutôt à un moteur spécialisé, Vite ressemble davantage à un environnement de travail prêt à l’emploi.

---

## 6. La différence essentielle entre esbuild et Vite

Si on simplifie beaucoup :

- **esbuild** transforme très vite du code
- **Vite** s’appuie sur des outils comme esbuild pour offrir une bonne expérience de développement

Donc :

- esbuild est plutôt une **brique**
- Vite est plutôt un **outil complet**

Une manière simple de s’en souvenir :

> esbuild aide à faire le travail technique rapidement  
> Vite aide le développeur à travailler confortablement

---

## 7. Pourquoi on dit souvent que Vite utilise esbuild

Cette phrase peut être un peu trompeuse si on ne précise pas le contexte.

Il faut la comprendre de manière simple :

- Vite utilise `esbuild` pour certaines transformations rapides
- cela aide beaucoup pendant le développement
- ce n’est pas forcément toute l’histoire du build final

L’idée importante n’est donc pas de mémoriser l’architecture interne exacte dès le début, mais de comprendre ceci :

> Vite s’appuie sur la rapidité de esbuild pour rendre le développement agréable.

---

## 8. Et Rollup alors ?

### Ce que c’est

**Rollup** est un autre outil de build.

Son rôle est de prendre plusieurs fichiers et dépendances, puis de construire un résultat final propre pour la production.

### Pourquoi Vite en parle ?

Parce que, dans sa logique classique :

- Vite est très orienté confort et rapidité en développement
- Rollup est utilisé pour le build de production

Donc, si on simplifie :

- **esbuild** aide beaucoup pour aller vite
- **Rollup** aide à produire une version finale bien préparée

### Comment comprendre Rollup simplement ?

Tu peux voir Rollup comme un **assembleur de version finale**.

Son travail est de :

- rassembler ce qu’il faut
- organiser les fichiers de sortie
- optimiser le résultat de production

Il est moins utile à retenir comme “nom technique à connaître absolument” que comme idée :

> pour la production, on veut un résultat final propre, cohérent et optimisé.

Et Rollup est l’outil historiquement utilisé par Vite pour cela.

---

## 9. Une image simple pour retenir les trois

Si on prend l’analogie d’un atelier :

- **esbuild** = un outil très rapide qui coupe et prépare les pièces
- **Rollup** = l’outil qui assemble soigneusement le produit final
- **Vite** = l’atelier bien organisé qui te permet de travailler facilement du début à la fin

Ou encore :

- esbuild = la vitesse
- Rollup = l’assemblage final
- Vite = l’expérience globale de travail

---

## 10. Développement et production : pourquoi ce n’est pas la même chose ?

C’est une idée très importante.

Quand tu développes, tu veux surtout :

- voir rapidement tes changements
- corriger sans attendre
- garder un cycle de travail fluide

Quand tu mets en production, tu veux surtout :

- des fichiers propres
- un résultat stable
- un code plus léger
- une sortie prête à être déployée

Donc les besoins ne sont pas exactement les mêmes.

C’est pour cela qu’un outil comme Vite peut utiliser une logique très rapide en développement, puis une logique plus orientée build final en production.

---

## 11. Un exemple très simple

Imaginons un mini-projet web avec :

- `index.html`
- `src/main.ts`
- `src/message.ts`
- `src/style.css`

Pendant que tu développes :

- tu modifies `main.ts`
- tu enregistres
- Vite met à jour très vite ce qu’il faut

Au moment du build :

- l’outil prépare une version finale
- il produit par exemple un dossier `dist/`
- ce dossier contient ce qui sera réellement servi en production

L’idée à retenir n’est pas le détail technique de chaque étape.
L’idée à retenir est :

- **en développement**, on cherche la fluidité
- **en production**, on cherche une sortie propre

---

## 12. Quand parler directement de esbuild ?

Il est utile de connaître `esbuild` si tu veux comprendre qu’il existe des outils très spécialisés dans la transformation rapide du code.

Mais, dans la pratique d’un projet frontend classique, tu peux très bien raisonner ainsi :

- j’utilise **Vite** pour travailler
- Vite s’occupe d’utiliser les bons outils en interne

Autrement dit, tu n’as pas forcément besoin de manipuler esbuild directement pour profiter de ses avantages.

---

## 13. Quand Vite est un bon choix

Vite est souvent un bon choix quand tu veux :

- créer une application frontend moderne
- travailler avec une boucle de développement rapide
- garder une configuration assez simple
- avoir un outil déjà bien adopté

Pour un apprentissage progressif, Vite est intéressant parce qu’il permet de travailler sans devoir comprendre dès le premier jour tous les détails internes du build.

---

## 14. Ce qu’il faut retenir absolument

### Idée 1
Le **build** est la préparation du projet pour sa version finale.

### Idée 2
**esbuild** est surtout un outil de transformation rapide.

### Idée 3
**Vite** est un outil de développement plus global, pensé pour rendre le travail confortable.

### Idée 4
**Rollup** est l’outil historiquement utilisé par Vite pour construire la version de production.

### Idée 5
Le développement et la production n’ont pas exactement les mêmes besoins.

### Idée 6
Il n’est pas nécessaire de tout comprendre en profondeur tout de suite pour utiliser Vite intelligemment.

---

## 15. À explorer ensuite

- [ ] **Le mot “bundler”**
  Comprendre ce mot simplement aide beaucoup. Un bundler sert à rassembler et organiser plusieurs fichiers pour produire un résultat final plus facile à distribuer.

- [ ] **La différence entre `src/` et `dist/`**
  `src/` correspond en général à ce que tu écris. `dist/` correspond à ce que l’outil prépare pour la version finale. Bien comprendre cette différence clarifie beaucoup de choses.

- [ ] **Le rôle du serveur de développement**
  Un dev server n’est pas la même chose qu’un build de production. Il sert surtout à travailler rapidement pendant que tu développes.

- [ ] **La minification**
  C’est le fait de rendre les fichiers plus compacts pour la production. Le code devient moins agréable à lire pour un humain, mais plus léger à envoyer au navigateur.

- [ ] **Le tree-shaking**
  C’est l’idée de ne garder dans le build final que ce qui est réellement utilisé. Même sans entrer dans le détail, comprendre ce principe aide à voir pourquoi certains outils de build sont utiles.

- [ ] **Les assets**
  Une application ne contient pas seulement du JavaScript. Il y a aussi des images, du CSS, des polices, des SVG. Comprendre comment un outil gère ces fichiers est une bonne suite logique.

- [ ] **Pourquoi la production demande plus de préparation**
  En développement, on veut aller vite. En production, on veut quelque chose de propre, stable et léger. Explorer cette différence permet de mieux comprendre le rôle réel des outils.

- [ ] **La configuration de Vite**
  Pas pour la mémoriser tout de suite, mais pour voir qu’elle sert à décrire quelques choix du projet : plugins, dossier de sortie, comportement du serveur, etc.

---

## 16. Questions de révision

1. Pourquoi utilise-t-on des outils de build en développement web moderne ?
2. Que veut dire le mot “build” dans ce contexte ?
3. Quelle différence simple peux-tu faire entre `src/` et `dist/` ?
4. À quoi sert esbuild, formulé avec des mots simples ?
5. Pourquoi esbuild est-il surtout connu ?
6. Pourquoi peut-on dire que esbuild est plutôt une brique qu’un outil complet de développement ?
7. À quoi sert Vite dans un projet frontend ?
8. Pourquoi Vite est-il souvent perçu comme agréable à utiliser ?
9. Quelle différence générale peux-tu faire entre esbuild et Vite ?
10. Pourquoi dit-on que Vite utilise esbuild ?
11. Quel est le rôle de Rollup, expliqué simplement ?
12. Pourquoi Vite ne traite-t-il pas forcément le développement et la production de la même manière ?
13. Qu’attend-on surtout d’un outil en phase de développement ?
14. Qu’attend-on surtout d’un outil en phase de production ?
15. Pourquoi est-il utile de distinguer vitesse de développement et qualité du build final ?
16. Dans l’analogie de l’atelier, quel rôle joue esbuild ?
17. Dans l’analogie de l’atelier, quel rôle joue Rollup ?
18. Dans l’analogie de l’atelier, quel rôle joue Vite ?
19. Pourquoi un débutant n’a-t-il pas besoin de comprendre immédiatement tous les détails internes de Vite ?
20. Que veut dire l’idée suivante : “Vite aide le développeur à travailler confortablement” ?
21. Pourquoi le concept de bundler est-il important à comprendre, même simplement ?
22. À quoi sert la minification ?
23. Que cherche à faire le tree-shaking ?
24. Pourquoi un projet frontend moderne contient-il autre chose que du JavaScript ?
25. Si tu devais résumer ce log en 5 phrases, que dirais-tu ?

---

## 17. Vocabulaire à retenir

- **Build** : étape où l’on prépare le projet pour sa version finale.
- **Code source** : fichiers que tu écris pour développer.
- **Production** : version finale du projet, prête à être mise en ligne.
- **Bundler** : outil qui rassemble et organise plusieurs fichiers pour produire une sortie finale.
- **Transpiler** : outil qui transforme un code écrit dans une syntaxe en une autre plus directement exploitable.
- **esbuild** : outil très rapide qui transforme et peut assembler du code JavaScript ou TypeScript.
- **Vite** : outil de développement frontend qui rend le travail plus fluide et prépare aussi le build.
- **Rollup** : outil utilisé pour construire une sortie de production propre et optimisée.
- **Dev server** : serveur utilisé pendant le développement pour voir rapidement les changements.
- **Asset** : fichier utilisé par le projet, par exemple une image, une feuille CSS, une police ou un SVG.
- **Minification** : réduction de la taille des fichiers pour la production.
- **Tree-shaking** : fait de retirer du build final le code non utilisé.
- **`src/`** : dossier où l’on place en général le code source.
- **`dist/`** : dossier qui contient en général la version finale produite par le build.

---

## Résumé en une phrase

> **esbuild** sert surtout à transformer le code très vite, **Rollup** sert à bien préparer la version finale, et **Vite** organise tout cela pour rendre le développement plus simple et plus fluide.
