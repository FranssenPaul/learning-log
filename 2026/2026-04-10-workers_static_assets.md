# Learning log — Comprendre Workers Static Assets avec Vite, Wrangler et un Worker

## 1. L’idée générale

Dans une application web moderne, il y a souvent deux grandes parties :

- **le front** : ce que le navigateur charge et exécute
- **la logique serveur** : ce qui reçoit des requêtes et construit des réponses

Dans l’écosystème Cloudflare Workers, on peut faire cohabiter les deux dans un même projet :

- un **Worker** pour la logique dynamique
- des **static assets** pour les fichiers front

L’objectif de **Workers Static Assets** est simple :

> déployer un Worker et, en même temps, les fichiers statiques de l’application.

Cloudflare sert alors les fichiers front au navigateur, et le Worker gère la partie dynamique ou les routes spéciales.

---

## 2. Qu’est-ce qu’un asset ?

Dans ce contexte, un **asset** est un **fichier statique**.

Typiquement :

- `index.html`
- `style.css`
- `main.js`
- une image
- une icône
- une police
- un fichier SVG déjà prêt

On dit **statique** parce que le fichier est servi tel quel au navigateur.

Exemples :

- un fichier HTML affiché dans le navigateur
- une feuille CSS
- un fichier JavaScript côté client
- une image PNG ou SVG

En revanche, le code de ton Worker **n’est pas** un asset :

- `worker/index.ts` est du **code exécuté**
- `dist/index.html` est un **asset servi**

La distinction importante est donc :

- **asset** = fichier servi
- **Worker** = code exécuté

---

## 3. Pourquoi “Workers Static Assets” est utile ?

Sans cela, on pourrait imaginer :

- un hébergement pour le front
- un autre service pour l’API

Avec Workers Static Assets, Cloudflare permet d’avoir :

- le front statique
- le Worker
- le déploiement
- la plateforme de diffusion

… dans un même ensemble.

C’est très intéressant pour apprendre à faire de **vraies petites applis publiables**, parce que cela rapproche beaucoup du modèle “produit complet” :

- une interface utilisateur
- une logique métier
- un point d’entrée HTTP
- un déploiement unique

---

## 4. Où se place le Worker dans tout ça ?

Le Worker est la partie qui reçoit une requête et construit une réponse.

Exemples de rôles possibles :

- lire des paramètres d’URL
- valider des entrées
- générer un SVG dynamiquement
- renvoyer du JSON
- gérer certaines routes d’API

Dans un cas plus général, le Worker peut par exemple :

- recevoir des paramètres comme `title`, `theme` ou `format`
- produire dynamiquement une ressource ou une réponse adaptée
- renvoyer du HTML, du JSON, du texte ou un SVG

Exemple mental :

- `/` → la page front
- `/api/preview?title=Bonjour&theme=light` → réponse générée par le Worker

---

## 5. Le front, dans ce modèle

Le front est ce que le navigateur télécharge.

Il contient par exemple :

- une page HTML
- du CSS
- du JavaScript ou TypeScript compilé
- des images

Ce front peut :

- afficher un formulaire
- laisser l’utilisateur saisir des paramètres
- appeler le Worker avec `fetch()`
- afficher la réponse renvoyée

Le front n’est pas obligé de produire lui-même le résultat final.

Deux approches sont possibles :

### Approche A — génération côté front
Le navigateur calcule ou construit directement le résultat.

### Approche B — génération côté Worker
Le navigateur demande au Worker de générer la réponse.

L’approche B est souvent très formatrice parce qu’elle apprend :

- la structure d’une API
- la validation côté serveur
- la production d’une réponse web exploitable par d’autres

---

## 6. Le rôle de Vite

**Vite** est un outil de développement et de build pour le front.

Il sert principalement à deux choses :

### En développement
Il lance un serveur local rapide pour travailler sur l’interface.

### En production
Il construit le front et produit un dossier final prêt à être servi.

Avec Vite, l’idée importante est la suivante :

- tu écris ton **code source front**
- Vite le transforme en **build final**

Ce build final va généralement dans `dist/`.

---

## 7. La différence entre `src`, `public` et `dist`

C’est un point essentiel.

### `src/`
C’est le **code source**.

On y met généralement :

- le TypeScript du front
- les modules métier
- le CSS importé dans le front
- les composants, si on en utilise

Par exemple, `src/` peut contenir :

- `main.ts`
- `style.css`
- `lib/api.ts`
- `lib/render.ts`

### `public/`
C’est un dossier spécial pour des fichiers statiques qu’on veut copier tels quels.

On l’utilise surtout pour :

- un favicon
- un `robots.txt`
- un fichier qui doit garder son nom exact
- un asset qu’on ne veut pas importer via le code

Important : `public/` n’est pas forcément “tout le front”.

Dans un projet Vite propre, le front source vit surtout dans `src/`.

### `dist/`
C’est le **résultat du build**.

On n’écrit normalement pas `dist/` à la main.

C’est Vite qui le produit quand on lance le build.

Donc :

- `src/` = source
- `public/` = fichiers statiques copiés tels quels
- `dist/` = sortie finale prête à être déployée

---

## 8. Pourquoi `dist/` est important avec Cloudflare

Quand tu utilises Vite, le front final n’est pas directement `src/`.

Cloudflare doit servir la **sortie du build**, donc le plus souvent :

- `dist/`

C’est pourquoi, dans une architecture Vite + Worker, il faut raisonner comme ceci :

1. tu développes ton front dans `src/`
2. Vite construit ce front
3. le résultat va dans `dist/`
4. Cloudflare sert ce résultat comme assets statiques

Autrement dit :

> `dist/` est ce que Cloudflare sert, pas `src/`

---

## 9. Et `public/` alors ?

`public/` reste utile, mais il ne faut pas le confondre avec le dossier final de déploiement.

Avec Vite :

- `public/` contient certains fichiers statiques source
- ces fichiers sont copiés dans `dist/` au build

Donc on peut voir cela comme :

- `public/` = dépôt de certains fichiers bruts
- `dist/` = package final prêt à servir

---

## 10. Le rôle de Wrangler

**Wrangler** est la CLI officielle de Cloudflare Workers.

Elle sert notamment à :

- développer localement
- lire la configuration du projet
- déployer le Worker
- déployer les assets statiques

Dans un projet moderne avec le plugin Vite de Cloudflare, le flux typique est souvent :

- `vite dev` pour le développement
- `vite build` pour construire le front
- `wrangler deploy` pour déployer

Wrangler reste donc central, même si le développement local quotidien passe plutôt par Vite.

---

## 11. Le plugin Cloudflare pour Vite

Le plugin `@cloudflare/vite-plugin` permet d’intégrer Vite et le runtime Workers.

Son intérêt est fort :

- ton développement front reste confortable
- ton code Worker s’intègre au même projet
- le runtime local se rapproche du comportement réel de production

C’est donc un bon choix pour un projet où tu veux :

- une petite interface propre
- un Worker qui génère ou renvoie une réponse dynamique
- un déploiement cohérent

---

## 12. `wrangler.jsonc` : pourquoi ce fichier existe

`wrangler.jsonc` est le fichier de configuration du projet Cloudflare.

### Pourquoi `jsonc` ?
Le `c` signifie **Comments**.

Cela veut dire qu’on peut utiliser une variante de JSON plus agréable à lire, notamment avec commentaires.

### Son rôle
Il sert à déclarer des informations comme :

- le nom du projet
- la date de compatibilité
- le point d’entrée du Worker
- d’autres options Cloudflare si besoin

Pour un nouveau projet, Cloudflare recommande aujourd’hui `wrangler.jsonc` plutôt que `wrangler.toml`.

---

## 13. Pourquoi ne pas partir sur Workers Sites ?

Historiquement, Cloudflare proposait aussi **Workers Sites**.

Mais pour les nouveaux projets, l’orientation actuelle est :

- **Workers Static Assets**
- pas **Workers Sites**

Autrement dit, pour un projet neuf, mieux vaut partir directement sur le modèle moderne.

---

## 14. Structure de projet recommandée

Une structure saine serait par exemple :

```text
mon-projet/
  index.html
  package.json
  tsconfig.json
  vite.config.ts
  wrangler.jsonc

  public/
    favicon.svg

  src/
    main.ts
    style.css
    lib/
      api.ts
      parser.ts
      render.ts
      types.ts

  worker/
    index.ts
```

### Comment lire cette structure ?

- `index.html` : point d’entrée du front Vite
- `src/` : code source du front et logique réutilisable
- `public/` : assets bruts à copier tels quels
- `worker/index.ts` : point d’entrée du Worker
- `dist/` : sera généré automatiquement au build

---

## 15. Flow général du projet

### En développement
1. tu écris ton interface dans `src/`
2. tu écris ton Worker dans `worker/`
3. tu lances le projet localement avec Vite

### Au build
4. Vite produit `dist/`

### Au déploiement
5. Cloudflare déploie :
   - le Worker
   - les assets statiques issus du build

Ce flow est très intéressant pédagogiquement parce qu’il t’oblige à distinguer :

- le code source
- le build final
- le rôle du serveur
- le rôle du navigateur

---

## 16. Ce qu’il faut retenir absolument

### Idée 1
Un **asset** est un fichier statique servi au navigateur.

### Idée 2
Le **Worker** est du code exécuté pour répondre à des requêtes.

### Idée 3
Avec **Vite**, le front source vit surtout dans `src/`.

### Idée 4
`public/` n’est pas le build final : c’est un dossier de fichiers statiques source particuliers.

### Idée 5
`dist/` est la sortie finale du build, donc le front prêt à être servi.

### Idée 6
**Wrangler** gère la configuration et le déploiement Cloudflare.

### Idée 7
Le combo **Vite + Cloudflare Vite plugin + Worker + Static Assets** est un excellent point de départ pour apprendre à construire des applis utiles et publiables.

---

## 17. Application directe à un mini-projet

Dans un mini-projet de ce type, tu peux viser ceci :

- une petite UI avec formulaire
- un Worker qui reçoit des paramètres
- une réponse générée dynamiquement
- un aperçu dans la page
- éventuellement une URL partageable

Cela t’apprendra simultanément :

- le front
- le HTTP
- la validation
- la génération de contenu
- le déploiement réel

C’est exactement le type de mini-projet qui apprend à construire un outil réellement réutilisable par d’autres.

---

## 18. Résumé ultra court

Workers Static Assets = **des fichiers statiques de front déployés avec un Worker**.

Dans ton stack :

- **Vite** construit le front
- **`dist/`** contient le build final
- **Cloudflare** sert ce build comme assets statiques
- **le Worker** gère la logique dynamique
- **Wrangler** configure et déploie l’ensemble

---

## 19. Questions de révision

1. Quelle est la différence entre un asset statique et le code d’un Worker ?
2. Pourquoi dit-on qu’un fichier comme `index.html` est statique ?
3. Quel est le rôle principal d’un Worker dans une application Cloudflare ?
4. Pourquoi Workers Static Assets est-il pratique pour déployer une application web complète ?
5. Quelle différence faut-il faire entre la partie front et la logique serveur ?
6. Dans une architecture avec Vite, à quoi sert le dossier `src/` ?
7. Quel type de fichiers place-t-on plutôt dans `public/` ?
8. Pourquoi ne faut-il pas considérer `public/` comme le build final ?
9. Quel est le rôle du dossier `dist/` ?
10. Pourquoi Cloudflare sert-il généralement `dist/` plutôt que `src/` ?
11. Que fait Vite en développement ?
12. Que fait Vite en production ?
13. Quel est le rôle de Wrangler dans un projet Cloudflare Workers ?
14. Pourquoi `wrangler.jsonc` est-il important ?
15. Que signifie le `c` dans `jsonc` ?
16. Que permet le plugin `@cloudflare/vite-plugin` ?
17. En quoi l’approche où le navigateur demande au Worker de générer la réponse est-elle pédagogique ?
18. Donne un exemple de route servie comme asset statique.
19. Donne un exemple de route qui serait traitée dynamiquement par un Worker.
20. Pourquoi est-il utile de bien distinguer le code source, le build final et la réponse HTTP ?
21. Quelle est la différence entre “fichier servi” et “code exécuté” ?
22. Pourquoi le couple Vite + Worker + Static Assets est-il une bonne base d’apprentissage ?
23. Si tu modifies un fichier dans `src/`, pourquoi faut-il ensuite reconstruire le projet pour la production ?
24. Quel est le flux général entre écriture du front, build et déploiement ?
25. Si tu devais résumer ce document en 5 phrases, quels seraient les points essentiels ?

---

## Sources officielles utiles

- Cloudflare Workers — Static Assets  
  <https://developers.cloudflare.com/workers/static-assets/>

- Cloudflare Workers — Static Assets with the Vite plugin  
  <https://developers.cloudflare.com/workers/vite-plugin/reference/static-assets/>

- Cloudflare Workers — Wrangler configuration  
  <https://developers.cloudflare.com/workers/wrangler/configuration/>

- Cloudflare Workers — Vite plugin  
  <https://developers.cloudflare.com/workers/vite-plugin/>

- Cloudflare Workers — Get started with the Vite plugin  
  <https://developers.cloudflare.com/workers/vite-plugin/get-started/>

- Cloudflare Workers — Workers Sites deprecation guidance  
  <https://developers.cloudflare.com/workers/configuration/sites/start-from-existing/>

- Vite — Building for Production  
  <https://vite.dev/guide/build>

- Vite — Static Asset Handling  
  <https://vite.dev/guide/assets>

- Vite — Build Options (`outDir = dist`)  
  <https://vite.dev/config/build-options>
