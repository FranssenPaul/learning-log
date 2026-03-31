# Solution POS : Architecture robuste en LAN avec WebSocket

## Contexte et objectif

Créer un système de caisse (Point of Sale) destiné aux restaurants, bars et boîtes de nuit moyens en Afrique francophone.  
Les contraintes principales sont :

- **Fiabilité** : fonctionnement continu même sans connexion internet.
- **Simplicité** : installation clé en main, maintenance réduite.
- **Coût maîtrisé** : modèle de location mensuelle, matériel inclus.

L’application repose sur un serveur local (LAN) et des tablettes en prise de commande, avec un temps réel grâce aux WebSockets.

---

## Pourquoi WebSocket plutôt que du polling ?

### Analogie : le livreur vs le client qui vient toutes les heures

- **Polling** : le client envoie une requête toutes les X secondes pour savoir si quelque chose a changé.  
  → Équivalent à appeler le restaurant toutes les 5 minutes pour demander si la commande est prête. Ça surcharge le serveur et crée un délai systématique.

- **WebSocket** : une fois la connexion établie, le serveur peut **pousser** l’information dès qu’elle est disponible.  
  → C’est le livreur qui sonne à la porte quand la commande arrive. Immédiat, efficace, et sans requêtes inutiles.

En LAN, la latence est inférieure à 10 ms et la connexion est stable. Le WebSocket exploite au mieux ces conditions.

---

## Architecture technique

### 1. Composants

- **Serveur unique** (mini‑PC, NUC, Raspberry Pi 5 avec SSD)  
  - IP fixe (ex. 192.168.1.50)  
  - Docker pour l’isolation et la mise à jour  
  - Base de données PostgreSQL (centralisée)  
  - Application FastAPI (API REST + WebSockets)  
  - Worker d’envoi de logs vers un serveur distant (optionnel, internet sortant uniquement)

- **Tablettes / téléphones** (Android / iPad)  
  - Connexion Wi‑Fi au LAN, IP dynamique (DHCP)  
  - Navigateur web (simple raccourci écran d’accueil, pas de PWA pour éviter la contrainte HTTPS)  
  - Stockage local (IndexedDB) pour la file d’attente hors ligne

- **Réseau**  
  - Switch Gigabit et point d’accès Wi‑Fi dédié  
  - Aucun accès internet pour les tablettes (pas de passerelle par défaut)  
  - Le serveur peut avoir une seconde connexion internet (pour la remontée de logs), sans forwarding IP vers le LAN

### 2. Fiabilité renforcée

#### Heartbeat applicatif (ping / pong)

- Toutes les 10-15 secondes, la tablette envoie un `ping` au serveur.
- Le serveur répond immédiatement par un `pong`.
- Si le client ne reçoit pas de `pong` après deux tentatives, il déclenche une reconnexion.
- Si le serveur ne reçoit pas de `ping` pendant 40 secondes, il ferme la connexion côté serveur.

#### Reconnexion automatique avec backoff exponentiel

- En cas de coupure (Wi‑Fi instable, redémarrage du serveur), le client tente de se reconnecter avec un délai croissant (1s, 2s, 4s… jusqu’à 30s).
- À la reconnexion, le client renvoie son `device_id` et synchronise les commandes non envoyées.

#### Système d’acknowledgment

Chaque message critique (création de commande, modification, paiement) est accompagné d’un identifiant unique généré côté client (`idempotency_key`).

- Le serveur accuse réception avec le même identifiant.
- Si le client n’a pas reçu l’ack dans un délai (ex. 5 secondes), il réessaie l’envoi.
- Le serveur ignore les doublons en vérifiant l’identifiant.

#### Offline‑first et file d’attente locale

- La tablette stocke les commandes en local (IndexedDB) avant même de les envoyer.
- Si le WebSocket est coupé, les commandes s’empilent.
- Dès la reconnexion, la file est rejouée automatiquement.

**Analogique** : comme un carnet de commandes papier. Le serveur note la commande, et quand le réseau revient, il la saisit en caisse.

---

## Déploiement avec Docker

Le serveur est empaqueté dans une image Docker contenant :

- FastAPI (avec Uvicorn)
- PostgreSQL
- Un worker séparé pour l’envoi des logs (si internet sortant)

### Avantages

- **Reproductibilité** : l’environnement est identique de la machine de développement à celle du client.
- **Mise à jour simple** : on remplace l’image Docker et on relance le conteneur.
- **Isolation** : pas de conflit avec d’autres logiciels éventuels sur le serveur.

---

## Gestion des logs et supervision

Même si le client n’a pas internet, le serveur peut avoir une connexion sortante (4G ou fibre) pour **remonter des logs anonymisés** (état du système, erreurs, volume de commandes).

- Les logs sont d’abord écrits dans PostgreSQL (table `outbox`).
- Un worker asynchrone les envoie vers un serveur distant en HTTPS.
- En cas de panne internet, les logs restent en base et sont envoyés plus tard.

Cette supervision permet de :

- détecter les pannes avant que le client n’appelle,
- surveiller l’état du parc (batteries des tablettes, uptime),
- collecter des données d’usage pour améliorer le produit.

---

## Modèle économique : location mensuelle matériel inclus

- **Client** : restaurant, bar, boîte de nuit moyen.
- **Offre** : forfait mensuel comprenant la licence logicielle + le matériel (serveur, tablettes, switch, onduleur éventuel) + le support.
- **Engagement** : contrat de 12 à 24 mois pour amortir le matériel.
- **Avantages pour le client** : pas d’investissement initial, coût prévisible, maintenance incluse.

### Pourquoi ce modèle est adapté

- Les petits commerçants ont du mal à avancer plusieurs milliers d’euros.
- La location transforme le logiciel en service, ce qui est plus facile à vendre.
- Le matériel reste la propriété du fournisseur, ce qui réduit le risque de revente à la concurrence.

---

## Gestion des paiements mobiles (Mobile Money)

L’intégration en temps réel avec les opérateurs (Orange Money, MTN, M‑Pesa) n’est pas prévue dans un premier temps. À la place, l’établissement utilise son propre compte mobile money.

### Fonctionnement

- Le caissier sélectionne « Mobile Money » comme mode de paiement.
- Il saisit le montant et éventuellement un numéro de transaction (facultatif).
- À la **clôture de caisse**, le gérant consulte le total des paiements mobiles enregistrés dans le POS.
- Il se connecte à son compte mobile money et vérifie que la somme reçue correspond.

### Avantages

- Pas de développement complexe (API, certifications).
- Pas de commission supplémentaire prélevée par un agrégateur.
- Le client reste maître de ses comptes.

### Risques et parades

| Risque | Parade |
|--------|--------|
| Fraude interne (employé qui détourne l’espèce en enregistrant mobile money) | Le POS affiche un écart à la clôture. Le gérant valide lui‑même chaque clôture et peut tracer par utilisateur. |
| Non‑conformité fiscale dans certains pays | Se renseigner localement ; si nécessaire, faire certifier le système ou utiliser un mode hybride (impression de ticket fiscal conforme). |

---

## Pourquoi cette solution est adaptée à l’Afrique francophone

- **Indépendance internet** : dans beaucoup de régions, la connexion est instable. Un système qui tourne en LAN sans dépendance du cloud est un argument de vente fort.
- **Location mensuelle** : le coût d’entrée est un frein ; la location lève cette barrière.
- **Gestion simplifiée** : les bars et boîtes de nuit n’ont pas de service informatique. L’installation clé en main et le support local sont essentiels.
- **Adaptabilité fiscale et monétaire** : l’architecture permet de paramétrer facilement les taux de TVA, les taxes locales et de gérer plusieurs devises si nécessaire.

---

## En résumé : une architecture professionnelle, robuste et pragmatique

| Composant | Choix |
|-----------|-------|
| Communication temps réel | WebSocket avec heartbeat et ack |
| Mode dégradé | Offline‑first, file d’attente locale |
| Base de données | PostgreSQL centralisée |
| Déploiement | Docker (image unique, mises à jour faciles) |
| Supervision | Logs remontés via worker asynchrone |
| Paiements mobiles | Reconnaissance à la clôture (pas d’API en temps réel) |
| Modèle commercial | Location mensuelle matériel inclus |

Cette approche combine la fiabilité des technologies modernes avec une compréhension fine des contraintes du terrain. Elle est prête à être déployée, testée et commercialisée dans les établissements moyens d’Afrique francophone.

---

**Auteur** : Paul Franssen  
**Date** : Mars 2026