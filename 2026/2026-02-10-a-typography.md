# Learning Log - Gestion des tailles typo et CSS/Tailwind

Date: 2026-02-10

## 1) Pourquoi cette note

Ce projet combine:
- du markdown rendu avec KaTeX (`$...$` et `$$...$$`)
- des tailles pilotees par YAML (`prompt_text_size`, `prompt_math_size`)
- Tailwind pour l'UI
- du CSS metier pour le rendu des prompts

L'objectif est d'avoir des tailles previsibles, lisibles, et faciles a regler.

## 2) Unites CSS utiles

- `px`: unite fixe ecran (pixel CSS).
- `rem`: relatif a la taille de police racine (`html`).
- `em`: relatif a la taille de police du contexte courant (parent/element).
- `vw`: 1% de la largeur du viewport.

Regle pratique:
- utiliser `rem` pour des bases stables.
- utiliser `vw` pour la fluidite responsive.
- utiliser `em` pour des variantes locales (ex: directives markdown).

## 3) `clamp(min, ideal, max)`

`clamp()` borne une taille fluide:
- minimum garanti
- valeur fluide (souvent en `vw`)
- maximum garanti

Exemple:
```css
font-size: clamp(1.35rem, 2.6vw, 2.4rem);
```

Lecture:
- jamais plus petit que `1.35rem`
- suit `2.6vw` entre les bornes
- jamais plus grand que `2.4rem`

## 4) Ce qu'on a etabli dans le projet

- `prompt_text_size` et `prompt_math_size` utilisent maintenant une echelle coherente.
- Les tailles `sm` a `9xl` sont supportees.
- Le display math (`$$...$$`) est regle via `.katex-display`.
- Le inline math (`$...$`) suit `.katex`.

Fichiers cle:
- `frontend/src/index.css`
- `backend/games/markdown_questions.py`
- `backend/games/log_questions/README.md`
- `MARKDOWN_QUESTIONS_PLAYBOOK.md`

## 5) Pourquoi du CSS specifique en plus de Tailwind

Tailwind est excellent pour:
- layout
- spacing
- composants UI

Mais ici, du CSS metier est preferable pour:
- cibler KaTeX (`.katex`, `.katex-display`)
- appliquer une echelle semantique dynamique (`game-prompt--text-*`, `game-prompt--math-*`)
- garder le mapping YAML -> rendu simple et robuste

## 6) Interference Tailwind vs CSS custom

Pas de probleme majeur avec la structure actuelle:
- `@import "tailwindcss";` est en haut de `frontend/src/index.css`
- les regles custom arrivent apres
- les selecteurs metier sont explicites (`.game-prompt ...`)

Donc, a specificite egale, les regles custom definies apres Tailwind gagnent.

## 7) Bonne pratique retenue

- Garder Tailwind pour l'UI et le layout.
- Garder le CSS metier pour la typographie de prompt et KaTeX.
- Garder des echelles lisibles (`sm..9xl`) cote backend (validation) et frontend (styles).
- Preferer `clamp()` pour un rendu responsive stable.

## 8) Points de vigilance

- Ne pas fixer `html { font-size: ... }` sans besoin fort (accessibilite/zoom utilisateur).
- Eviter des ecarts trop grands entre texte et maths qui cassent la hierarchie visuelle.
- Documenter toute nouvelle taille dans `backend/games/log_questions/README.md` et `MARKDOWN_QUESTIONS_PLAYBOOK.md`.
