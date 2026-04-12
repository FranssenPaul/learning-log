# SSH — Authentification par paire de clés

**Date** : 2026-03-29
**Tags** : `ssh` `cryptographie` `authentification` `github` `réseau` `asymétrique`

---

## Le problème fondamental

Quand tu te connectes à GitHub (ou n'importe quel serveur SSH), le serveur doit répondre à une question :

> *"Est-ce vraiment Paul ?"*

Le mot de passe classique répond : *"Il connaît le secret partagé."*
SSH répond : *"Il possède la clé privée correspondant à la clé publique qu'il m'a donnée."*

La différence cruciale : **la clé privée ne quitte jamais ta machine**.

---

## La métaphore du cadenas

- Tu fabriques une paire : **cadenas** (`id_ed25519.pub`) + **clé** (`id_ed25519`)
- Tu donnes le **cadenas ouvert** au serveur — il peut le fermer, mais pas l'ouvrir
- Tu gardes la **clé** chez toi — elle ne voyage jamais sur le réseau

Quand tu te connectes, le serveur te lance un défi :
*"Prouve que tu peux ouvrir ce cadenas."*
Ta machine ouvre le cadenas. Le serveur vérifie. Connexion établie.

---

## Ce qui se passe cryptographiquement

L'algorithme utilisé ici est **Ed25519** (courbe elliptique, 256 bits).
C'est de la **cryptographie asymétrique** : ce que l'une chiffre, seule l'autre peut déchiffrer.

```
Ta machine                          Serveur (GitHub)
──────────                          ────────────────
id_ed25519      (clé privée)        id_ed25519.pub  (dans ton compte)

1. "Je suis Paul"                ──────────────────────────►
2. Challenge : nombre aléatoire  ◄──────────────────────────
3. Signe le challenge            
   avec ta clé privée            ──────────────────────────►
4.                                  Vérifie la signature
                                    avec ta clé publique
                                    Si ça correspond → OK ✓
```

**Signer** = transformer un message avec ta clé privée de façon que n'importe qui
ayant ta clé publique puisse vérifier que c'est bien toi qui l'as produit.

---

## Ce qui se passe sous le capot réseau

### 1. Établissement de la connexion TCP

Avant même de parler SSH, TCP fait son travail :

```
Client (sirocco)          Serveur (GitHub :22)
        │                        │
        │──── SYN ──────────────►│
        │◄─── SYN-ACK ───────────│
        │──── ACK ──────────────►│
        │                        │
        │   connexion TCP établie │
```

Le port **22** est le port standard SSH.

### 2. Négociation SSH (handshake)

Les deux parties s'accordent sur les algorithmes à utiliser :

```
Client                          Serveur
  │── SSH_MSG_KEXINIT ─────────►│   "voici les algos que je supporte"
  │◄─ SSH_MSG_KEXINIT ──────────│   "voici les miens"
  │                              │   → ils choisissent l'intersection
```

À ce stade, ils négocient :
- L'algorithme d'**échange de clés** (ex: `curve25519-sha256`)
- L'algorithme de **chiffrement symétrique** (ex: `chacha20-poly1305`)
- L'algorithme de **signature** (ex: `ssh-ed25519`)

### 3. Échange de clés Diffie-Hellman (ou ECDH)

C'est ici que la **magie du secret partagé** se produit :

```
Client génère un nombre aléatoire a  →  calcule A = g^a mod p
Serveur génère un nombre aléatoire b →  calcule B = g^b mod p

Client envoie A au serveur
Serveur envoie B au client

Client calcule  : K = B^a mod p
Serveur calcule : K = A^b mod p
→ K est identique des deux côtés, sans jamais avoir été transmis !
```

Ce secret partagé **K** sert à dériver la **clé de session symétrique**.
Tout le reste de la communication est chiffré avec cette clé.

> C'est le principe du **Perfect Forward Secrecy** : même si quelqu'un enregistre
> le trafic aujourd'hui et obtient ta clé privée demain, il ne pourra pas déchiffrer
> la session — car K était éphémère.

### 4. Authentification du serveur

Avant de t'authentifier, **tu vérifies que le serveur est bien GitHub**.
C'est le rôle du fichier `~/.ssh/known_hosts`.

Lors de ta première connexion, tu as vu :
```
The authenticity of host 'github.com' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting?
```

En répondant `yes`, tu as **fait confiance à cette empreinte** et elle a été
enregistrée dans `known_hosts`. Lors des connexions suivantes, SSH vérifie
que l'empreinte correspond — si elle change, c'est une alarme (attaque MITM possible).

### 5. Authentification du client (ta paire de clés)

C'est ici qu'intervient ta paire de clés :

```
Client                                    Serveur
  │── "Je veux m'authentifier             │
  │    avec la clé id_ed25519" ──────────►│
  │                                        │
  │◄── Challenge (nombre aléatoire) ───────│
  │                                        │
  │ Signe le challenge avec id_ed25519     │
  │── Signature ──────────────────────────►│
  │                                        │
  │                  Vérifie avec ta clé   │
  │                  publique enregistrée  │
  │◄── SSH_MSG_USERAUTH_SUCCESS ───────────│
```

### 6. Session chiffrée

À partir de là, tout le trafic (commandes git, données) est chiffré
avec la clé de session symétrique négociée à l'étape 3.

---

## Les fichiers en jeu

| Fichier | Rôle |
|---|---|
| `~/.ssh/id_ed25519` | Clé **privée** — ne quitte jamais ta machine |
| `~/.ssh/id_ed25519.pub` | Clé **publique** — tu la donnes aux serveurs |
| `~/.ssh/known_hosts` | Empreintes des serveurs connus (anti-MITM) |
| `~/.ssh/authorized_keys` | Clés publiques autorisées à se connecter *vers* ta machine |

---

## Pourquoi c'est plus solide qu'un mot de passe

| | Mot de passe | SSH |
|---|---|---|
| Transmis sur le réseau | Oui (même chiffré) | **Jamais** |
| Bruteforcable | Oui | Non (256 bits) |
| Phishable | Oui | Non |
| Révocable par machine | Non | Oui (supprimer la clé publique) |

---

## La passphrase — couche de sécurité locale

```bash
ssh-keygen -t ed25519 -C "commentaire"
# Enter passphrase: ****
```

- La clé privée est **chiffrée sur disque** avec la passphrase
- Si quelqu'un vole ton fichier `id_ed25519`, il ne peut pas l'utiliser sans elle
- `ssh-agent` la mémorise en RAM pour ne pas la retaper à chaque fois

---

## Procédure complète sur une nouvelle machine

```bash
# 1. Vérifier si une clé existe déjà
ls -la ~/.ssh/

# 2. Générer une nouvelle paire
ssh-keygen -t ed25519 -C "nom-machine"

# 3. Afficher la clé publique à copier sur GitHub
cat ~/.ssh/id_ed25519.pub

# 4. Tester la connexion
ssh -T git@github.com
# → Hi FranssenPaul! You've successfully authenticated...

# 5. Cloner
git clone git@github.com:FranssenPaul/repo.git
```

---

## À retenir

> La clé privée **signe**, la clé publique **vérifie**.
> La clé privée ne voyage jamais. Le serveur n'a pas besoin de la connaître pour te reconnaître.
> C'est l'essence de la cryptographie asymétrique.
