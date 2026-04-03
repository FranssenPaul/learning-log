# Learning Log — esbuild & Vite

**Date :** 2026-04-03
**Tags :** `tooling` `bundler` `build` `javascript` `vite` `esbuild`
**Statut :** première lecture — concepts fondamentaux

---

## 1. Le problème de départ — pourquoi des outils de build ?

Quand tu écris du JavaScript moderne, tu utilises :
- des **modules** (`import/export`)
- du **TypeScript**
- du **JSX** (React)
- des bibliothèques tierces (`node_modules`)
- du CSS avancé (SCSS, CSS Modules…)

**Les navigateurs ne comprennent pas tout ça directement.**

Il faut donc *transformer* et *assembler* le code avant de le livrer. C'est le travail d'un **bundler** (outil de mise en paquets).

> **Analogie — le restaurant :** Ton code source, c'est la cuisine avec les ingrédients bruts (modules séparés, TypeScript, JSX). Le navigateur, c'est le client à table. Le bundler, c'est le cuisinier qui transforme, assemble et présente le plat fini.

---

## 2. esbuild — le moteur de course

### Ce que c'est

**esbuild** est un **bundler et transpileur JavaScript/TypeScript**, écrit en **Go** (pas en JavaScript).

Il fait deux choses fondamentales :
1. **Transpiler** — convertir TypeScript, JSX, syntaxe ES moderne → JavaScript que les navigateurs comprennent
2. **Bundler** — prendre des dizaines/centaines de fichiers séparés et les assembler en un seul (ou quelques) fichier(s)

### Pourquoi Go ? La clé de sa vitesse

La plupart des outils de build JS (Webpack, Rollup, Babel…) sont eux-mêmes écrits en JavaScript, et tournent dans Node.js. C'est un peu comme demander à un traducteur de se traduire lui-même.

Go est un langage compilé, avec une gestion native du **parallélisme**. esbuild peut exploiter tous les cœurs de ton CPU simultanément.

> **Analogie — la scierie :** Webpack, c'est un menuisier habile qui coupe les planches une par une, à la main. esbuild, c'est une scierie industrielle avec dix lames en parallèle. Le résultat est le même — des planches prêtes à l'emploi — mais la vitesse est sans comparaison. On parle de **10x à 100x plus rapide** que les bundlers traditionnels.

### Ce qu'esbuild fait (et ne fait pas)

| ✅ Fait | ❌ Ne fait pas (ou peu) |
|---|---|
| Transpiler TS/JSX | Hot Module Replacement (HMR) dev avancé |
| Bundler ES Modules | Optimisations CSS complexes |
| Tree-shaking | Plugins riches (ecosystème limité) |
| Minification | Analyse fine du bundle (visualiseur) |
| Source maps | |

esbuild est **un moteur**, pas un véhicule complet. C'est une pièce d'infrastructure, pas un outil clé-en-main pour le développement quotidien.

### Usage direct (CLI)

```bash
# Transpiler un fichier
esbuild src/index.tsx --bundle --outfile=dist/bundle.js

# Avec options courantes
esbuild src/index.tsx \
  --bundle \
  --minify \
  --sourcemap \
  --target=es2020 \
  --outfile=dist/bundle.js
```

### Usage en Node.js (API)

```javascript
import * as esbuild from 'esbuild'

await esbuild.build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  minify: true,
  outfile: 'dist/bundle.js',
})
```

---

## 3. Vite — l'atelier complet du développeur moderne

### Ce que c'est

**Vite** (prononcé "veet", *vite* en français) est un **outil de développement frontend complet**, créé par Evan You (le créateur de Vue.js).

Vite n'est **pas** un bundler. C'est un **dev server + build tool** qui :
- gère le serveur de développement avec HMR ultra-rapide
- orchestre la production via Rollup (+ bientôt via Rolldown)
- fournit une configuration prête à l'emploi (React, Vue, TypeScript…)

> **Analogie — l'atelier de menuiserie :** esbuild est la scie à ruban — un outil précis et puissant. Vite, c'est l'atelier complet : la scie (esbuild en sous-main), l'établi, le rangement, l'éclairage, et un assistant qui range les copeaux pendant que tu travailles.

### L'insight fondamental de Vite : deux modes distincts

Vite a compris que **développement** et **production** sont deux problèmes différents :

#### Mode développement — les ES Modules natifs

En dev, le navigateur moderne comprend les `import` nativement (ES Modules). Vite **n'assemble rien**. Il sert les fichiers *tels quels*, en transformant uniquement ce qui en a besoin (TS → JS, JSX → JS) à la demande, via esbuild.

Résultat : le serveur démarre **instantanément**, quel que soit la taille du projet.

> **Analogie — le restaurant encore :** En dev, Vite ne cuisine pas tout le menu à l'avance. Il prépare chaque plat *à la commande*, au moment où le navigateur demande le fichier. Pas de mise en place longue, service immédiat.

#### Mode production — Rollup (bundler mature)

Pour la production, il faut assembler, optimiser, minifier. Vite utilise **Rollup**, un bundler mature avec un excellent tree-shaking et un écosystème de plugins riche.

> ⚠️ **Pourquoi pas esbuild en production ?** esbuild est moins mature pour certaines optimisations avancées (code splitting complexe, CSS modules…). Vite préfère la robustesse de Rollup pour le build final. *Note : Rolldown (un Rollup réécrit en Rust) est en cours de développement et deviendra le futur moteur de Vite.*

### Architecture interne de Vite

```
┌─────────────────────────────────────────┐
│                  VITE                   │
│                                         │
│  ┌──────────────┐  ┌─────────────────┐  │
│  │  Dev Server  │  │  Build (prod)   │  │
│  │              │  │                 │  │
│  │  esbuild     │  │  Rollup         │  │
│  │  (transform) │  │  (bundle+opt)   │  │
│  │              │  │                 │  │
│  │  HMR natif   │  │  Tree-shaking   │  │
│  └──────────────┘  └─────────────────┘  │
│                                         │
│  Plugin API unifiée (Rollup-compatible) │
└─────────────────────────────────────────┘
```

### Démarrer avec Vite

```bash
npm create vite@latest mon-projet -- --template react-ts
cd mon-projet
npm install
npm run dev    # serveur dev instantané
npm run build  # build production via Rollup
```

### Configuration (vite.config.ts)

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
})
```

---

## 4. Comparaison directe — esbuild vs Vite

| Critère | esbuild | Vite |
|---|---|---|
| **Nature** | Bundler/transpileur bas niveau | Outil de dev complet |
| **Langage** | Go | JavaScript (Node.js) |
| **Vitesse** | Extrêmement rapide | Très rapide (grâce à esbuild en dev) |
| **Dev server** | Non (à construire soi-même) | Oui, avec HMR |
| **HMR** | Non | Oui, natif |
| **Config** | Minimale, API simple | Configuration riche |
| **Plugins** | Écosystème limité | Riche (compatible Rollup) |
| **Cas d'usage** | Build scripts, CLI tools, libs | Applications web (SPA, SSR…) |
| **Courbe d'apprentissage** | Faible | Modérée |

> **Analogie finale — moteur vs voiture :** esbuild est un moteur de voiture de course — puissant, nu, à intégrer soi-même dans un châssis. Vite est une voiture complète qui *utilise* un moteur rapide (esbuild) pour le quotidien, et un moteur plus sophistiqué (Rollup) pour la course officielle (production).

---

## 5. Quand choisir quoi ?

### Choisir esbuild directement si :
- tu construis un **outil CLI** ou un **script de build custom**
- tu veux intégrer un bundler dans ta propre infrastructure
- tu construis une **bibliothèque** (pas une app avec UI)
- tu as besoin d'un contrôle total et minimal

### Choisir Vite si :
- tu développes une **application web** (React, Vue, Svelte…)
- tu veux un **DX (Developer Experience) optimal** out-of-the-box
- tu as besoin de HMR, de gestion des assets, de plugins
- → **C'est le choix par défaut pour 95% des projets web modernes**

---

## 6. Connexions avec mes projets

### Think.Play
Le frontend React de Think.Play *devrait* être bootstrappé avec Vite (si ce n'est pas déjà le cas). Le HMR en développement est précieux quand on teste les composants WebSocket et quiz en temps réel.

### paul.craft()
Pour les visualisations mathématiques (JSXGraph, p5.js…), Vite gère parfaitement les imports de bibliothèques tierces et la production de bundles optimisés.

### POSEN
Le frontend React du POS bénéficierait de Vite pour le dev, avec un build Rollup optimisé pour la distribution offline-first.

---

## 7. À explorer ensuite

- [ ] **Rollup** — comprendre le bundler de production derrière Vite
- [ ] **Rolldown** — le futur de Vite (réécrit en Rust)
- [ ] **Vite + FastAPI** — intégration du build Vite avec un backend Python
- [ ] **vite-plugin-pwa** — pour l'offline-first dans POSEN
- [ ] **vitest** — framework de test qui réutilise la config Vite

---

## Résumé en une phrase

> **esbuild est le moteur.** C'est ce qui transforme et assemble ton code à une vitesse inédite.
> **Vite est le véhicule.** Il embarque ce moteur pour le dev, ajoute tout ce dont un développeur a besoin, et utilise Rollup pour les longs trajets en production.
