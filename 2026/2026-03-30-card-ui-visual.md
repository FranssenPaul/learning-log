# Anatomie Visuelle d'une Card UI

## Objectif

Ce document complete [VOCABULAIRE_CARDS_UI.md](/home/franssen/Ducit/VOCABULAIRE_CARDS_UI.md).

Ici, l'idee n'est pas seulement de definir les mots, mais de voir comment ils se placent dans une card.

## Vue d'ensemble

Voici une card fictive tres simple :

```text
+--------------------------------------------------+
| EYEBROW                                          |
| Title principal                                  |
| Petite description pour expliquer la card.       |
|                                                  |
| [ Contenu principal / champ / liste / image ]    |
|                                                  |
| [Action principale]   [Action secondaire]        |
+--------------------------------------------------+
```

On peut la lire comme ceci :

- `EYEBROW` : petit texte de contexte
- `Title principal` : titre principal
- `Petite description...` : texte explicatif
- `[Contenu principal ...]` : body ou content
- `[Action principale] [Action secondaire]` : actions, souvent dans le footer ou en bas du content

## Version annotee

```text
+--------------------------------------------------+
| Eyebrow                                          |
| Title                                            |
| Description                                      |
|                                                  |
| Body / Content                                   |
|                                                  |
| Actions                                          |
+--------------------------------------------------+
```

## 1. Eyebrow

L'`eyebrow` est le petit texte situe tout en haut.

Exemple :

```text
Chapitre actif
```

Son role :

- introduire la section
- donner une categorie
- poser un contexte discret

Il est souvent :

- petit
- en capitales
- plus leger visuellement que le titre

## 2. Title

Le `title` est le titre principal de la card.

Exemple :

```text
Mot de passe
```

Son role :

- nommer clairement la card
- dire ce que l'utilisateur regarde

Visuellement, c'est souvent l'element textuel le plus fort.

## 3. Description

La `description` explique ce que fait la card ou ce que l'utilisateur doit comprendre.

Exemple :

```text
Le mot de passe sert seulement a ouvrir une session securisee.
```

Son role :

- clarifier l'usage
- rassurer
- guider

## 4. Body ou Content

Le `body` ou `content` est la zone principale de la card.

C'est l'endroit ou se trouve l'interaction ou l'information utile.

Exemples :

- un input
- un menu deroulant
- une liste
- un graphique
- un bloc de texte

Dans une card de connexion :

```text
[ champ mot de passe ]
```

Dans une card de chapitre :

```text
[ select : Fonction carre ]
```

## 5. Actions

Les `actions` sont les elements qui permettent d'agir.

Exemples :

```text
[Se connecter]
[Nouvelle conversation]
[Annuler]
```

On distingue souvent :

- `primary action`
- `secondary action`

## 6. Footer

Le `footer` est une zone basse reservee a des actions ou informations secondaires.

Toutes les cards n'en ont pas.

Parfois, les actions sont visuellement en bas sans qu'on parle formellement de footer.

Donc :

- si une zone basse est clairement separee, on parle volontiers de `footer`
- sinon, on peut simplement parler de `actions` en bas de card

## Exemple 1 : card de connexion

```text
+--------------------------------------------------+
| Acces                                            |  <- Eyebrow
| Mot de passe                                     |  <- Title
| Ouvre une session securisee.                     |  <- Description
|                                                  |
| [ input mot de passe ]                           |  <- Body / Content
|                                                  |
| [ Se connecter ]                                 |  <- Primary action
| Session active sur cet appareil.                 |  <- Meta / Status text
+--------------------------------------------------+
```

Lecture :

- `Acces` = eyebrow
- `Mot de passe` = title
- la phrase d'explication = description
- le champ = body
- le bouton = primary action
- le texte d'etat = meta ou status text

## Exemple 2 : card de selection de chapitre

```text
+--------------------------------------------------+
| Chapitre actif                                   |  <- Eyebrow
|                                                  |
| [ Fonction carre v ]                             |  <- Body / Content
|                                                  |
| D'autres chapitres pourront etre ajoutes.        |  <- Description
+--------------------------------------------------+
```

Lecture :

- `Chapitre actif` = eyebrow
- le select = body
- le texte d'explication = description

Ici, il n'y a pas forcement de `title` separe.

C'est important : une card n'est pas obligee d'avoir tous les elements.

## Exemple 3 : card d'action simple

```text
+--------------------------------------------------+
| Conversation                                     |  <- Eyebrow
|                                                  |
| [ Nouvelle conversation ]                        |  <- Action
+--------------------------------------------------+
```

Lecture :

- `Conversation` = eyebrow
- `Nouvelle conversation` = action

Ici, la card est volontairement tres legere.

## Toutes les cards n'ont pas la meme anatomie

Une erreur frequente quand on apprend est de croire qu'une card doit toujours contenir :

- eyebrow
- title
- description
- body
- footer

En realite, une card peut etre :

- tres riche
- tres compacte
- purement informative
- purement orientee action

L'important est surtout de comprendre les briques possibles.

## Exemple de comparaison

### Card complete

```text
Eyebrow
Title
Description
Body
Actions
Footer
```

### Card minimale

```text
Title
Action
```

### Card informative

```text
Eyebrow
Title
Description
```

### Card utilitaire

```text
Eyebrow
Body
Action
```

## Comment lire une card quand tu designes

Quand tu regardes une card, pose-toi ces questions :

1. Quel est son role principal ?
2. Quel est le texte qui donne le contexte ?
3. Quel est le texte qui nomme vraiment la section ?
4. Quel est le contenu utile ?
5. Quelle est l'action principale ?
6. Y a-t-il une information secondaire ou un etat ?

Ces questions t'aident a reconnaitre naturellement :

- eyebrow
- title
- description
- content
- actions
- meta

## Application a Ducit

Dans ton interface actuelle, tu peux lire les cartes comme ceci.

### Card `Chapitre actif`

- Eyebrow : `Chapitre actif`
- Content : le select
- Description : le texte qui explique l'ajout futur de chapitres

### Card `Mot de passe`

- Eyebrow : `Acces`
- Title : `Mot de passe`
- Description : l'explication sur la session securisee
- Content : l'input
- Action : `Se connecter`
- Status text : le texte d'etat

### Card `Conversation`

- Eyebrow : `Conversation`
- Action : `Nouvelle conversation`

## Resume tres simple

Si tu veux une lecture ultra rapide d'une card :

```text
[Contexte]      -> Eyebrow
[Nom principal] -> Title
[Explication]   -> Description
[Zone utile]    -> Content / Body
[Boutons]       -> Actions
```

Quand tu identifies bien ces pieces, tu comprends beaucoup mieux :

- comment structurer une interface
- comment nommer proprement les zones
- comment rendre une card plus claire visuellement
