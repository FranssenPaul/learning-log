# Pédagogie des fractions — séquence de remédiation (16 ans)

tags: #pédagogie #fractions #remédiation #droite-numérique #visualisation

---

## Contexte

Remédiation sur les fractions pour des élèves de 16 ans n'ayant pas acquis certaines bases.
Contrainte centrale : le contenu doit être **didactiquement régressif mais contextuellement adulte**.
Un élève de 16 ans sait qu'il n'a pas les bases — le dispositif ne doit pas l'humilier.

---

## Pourquoi le rond (camembert) est insuffisant seul

Le rond capture une seule facette de la fraction : *partie d'un tout continu*.

| | Rond | Droite numérique |
|---|---|---|
| Fraction > 1 | bricolage (2 ronds) | naturel |
| Équivalence | invisible | même point |
| Addition | fusion de deux objets | déplacement |
| Lien avec les décimaux | aucun | immédiat |
| Lien avec les négatifs | impossible | naturel plus tard |

La droite est une analogie qui **grandit avec l'élève** — elle reste valide en 3e, 2nde, algèbre.
Le rond s'arrête aux fractions simples.

**À garder** : le rond comme déclencheur initial, pas comme outil de travail prolongé.

---

## Erreurs classiques à 16 ans (théorèmes-en-acte incorrects)

- `½ + ⅓ = 2/5` → addition séparée des numérateurs et dénominateurs
- `¾ > ⅘` → "4 est plus grand que 3 donc…"
- fraction = toujours < 1 → conception "partie d'un tout" trop exclusive
- le dénominateur "c'est le nombre de parts" → mécanique sans sens du rapport

Ces élèves *savent* découper un gâteau en 4. Repartir de là ne déstabilise pas la conception erronée, ça la contourne momentanément.

---

## Principe de la droite numérique interactive

La droite est un **instrument de mesure**, pas une figure géométrique.

- La fraction est une **position**, pas un découpage
- Le dénominateur **commande les graduations** — l'élève choisit combien de divisions
- L'addition est un **déplacement** vers la droite (comme pour les entiers)
- L'équivalence devient un **constat visuel** : 2/4 et ½ atterrissent au même point

### Pourquoi laisser l'élève choisir les graduations

Si on affiche directement la droite en sixièmes, l'élève exécute sans comprendre.
Il doit pouvoir essayer "cinquièmes" → voir que ½ ne tombe pas pile → comprendre pourquoi sixièmes fonctionne.
**Le conflit cognitif est dans l'outil, pas dans le discours.**

---

## Séquence de l'outil interactif

### Qui gradue quoi

| Étape | Graduations |
|---|---|
| 1. Rond → Droite | l'élève choisit |
| 2. Droite → Fraction | données (forcément) |
| 3. Fraction → Droite | l'élève choisit |
| 4. Addition guidée | données (dénominateur commun) |
| 5. Addition avancée | l'élève choisit |

### Étape 1 — Rond → Droite
L'élève voit un rond découpé, il choisit les graduations de la droite et place le point.
*Installe : la fraction comme valeur, pas comme découpage*

**Choix pédagogique** : à 16 ans, afficher les graduations d'emblée serait infantilisant.
On propose un sélecteur "en combien de parts tu divises la droite ?" — c'est lui qui décide.
Si il choisit 8 au lieu de 4, le point tombe sur 2/8 → **l'équivalence émerge naturellement**,
sans en faire une étape didactique explicite.

### Étape 2 — Droite → Fraction
Un point est placé sur une droite déjà graduée, l'élève lit et écrit la fraction.
*Installe : lire une position, comprendre le rôle du dénominateur*

Les graduations sont forcément données ici — on ne peut pas montrer un point sans contexte.
Geste cognitif différent de l'étape 1 : **lire** les graduations plutôt que les construire.
Les deux sens se renforcent mutuellement.

### Étape 3 — Fraction → Droite
On donne une fraction écrite, l'élève choisit les graduations et place le point.
*Installe : le dénominateur commande les graduations*

### Étape 4 — Addition guidée
Droite affichée avec le dénominateur commun déjà en place.
L'élève place le premier terme (point de départ), puis le deuxième terme comme un bond,
et lit le résultat là où il atterrit.
*Installe : additionner = se déplacer, pas fusionner deux objets*

**Choix pédagogique** : à ce stade l'enjeu cognitif est l'addition comme déplacement,
pas encore la recherche du dénominateur commun. Surcharger les deux diluerait les deux apprentissages.

### Étape 5 — Découverte de la règle
Série d'exercices d'addition générés automatiquement.
L'élève répète le geste sur plusieurs cas → il observe les patterns :
- les graduations communes correspondent à quoi dans les dénominateurs ?
- le numérateur du résultat, comment se calcule-t-il ?

**Ce que l'enseignant n'a pas besoin de dire** :
- "on met au même dénominateur"
- "on multiplie numérateur et dénominateur par le même nombre"
- "on additionne les numérateurs"

L'élève le constate, le formule, le verbalise.
La règle n'est plus une procédure tombée du ciel — c'est une **généralisation de ce qu'il a vu**.

### Étape 6 — Addition avancée
Mêmes exercices, mais les graduations ne sont plus données.
L'élève doit d'abord trouver le dénominateur commun, graduer, puis additionner.

---

## Notes d'implémentation

- Rond à gauche, droite numérique à droite — l'élève fait lui-même le pont
- Feedback visuel : le point se cale sur la bonne position ou montre l'écart
- Variante : montrer un point sur la droite, l'élève choisit le rond correspondant (sens inverse)
- Retrait progressif du rond : la droite reste, le rond disparaît quand l'élève n'en a plus besoin
- Génération automatique d'exercices d'addition pour l'étape 5
- Technologie : SVG ou Canvas

---

## Multiplication — méthode des rectangles

### Principe

Un rectangle unitaire 1×1 représente le "tout".
On découpe dans **deux directions perpendiculaires** — une par fraction.

Pour ¾ × ⅔ :
- On découpe la largeur en 3, on en garde 2 → ⅔
- On découpe la hauteur en 4, on en garde 3 → ¾
- La zone colorée (intersection) c'est le produit

```
┌──┬──┬──┐
│██│██│  │  ← 3 rangées sur 4
│██│██│  │
│██│██│  │
├──┼──┼──┤
│  │  │  │
└──┴──┴──┘
 ←⅔→
```

- Cases colorées : 2×3 = **6** → numérateur
- Cases totales : 3×4 = **12** → dénominateur
- Résultat : 6/12 = ½

### Ce que l'élève découvre

La règle "on multiplie les numérateurs entre eux et les dénominateurs entre eux" n'est plus magique :
- numérateur = nombre de cases colorées
- dénominateur = nombre de cases totales

La commutativité est visible : retourner le rectangle donne la même zone colorée.

### Extension : multiplier par un entier

Une fois la règle établie sur fraction × fraction, poser la question :
"Et si on multiplie ¾ × 2 ?"

L'élève applique la règle → il bloque sur le dénominateur de 2.
**2 = 2/1** émerge alors comme nécessité, pas comme convention arbitraire.
C'est le même geste qu'avec l'addition : la convention arrive comme solution à un problème ressenti.

### Lien avec l'addition

| Opération | Représentation | Geste |
|---|---|---|
| Addition | Droite numérique | déplacement |
| Multiplication | Rectangle | découpage perpendiculaire |

Deux analogies distinctes, mais le même principe : **la règle émerge du visuel**, l'élève la formule lui-même.

---

## Références

- Vergnaud — théorèmes-en-acte, champs conceptuels
- Brousseau — situations didactiques, conflit cognitif
- Registres de représentation (Duval) — importance de multiplier les registres

