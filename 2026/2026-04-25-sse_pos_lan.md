# Appliquer SSE à un POS en LAN

> Objectif : concevoir une architecture robuste pour un **système de caisse restaurant/bar en réseau local**, avec **SSE pour les notifications**, **HTTP pour les données**, **event log pour la reprise**, et **file locale pour les coupures réseau**.

---

## 1. Contexte du POS

Le système visé est un POS local pour restaurant ou bar :

- un serveur local dans l’établissement ;
- des tablettes ou téléphones pour les serveurs ;
- un écran cuisine ou bar ;
- un poste caisse ;
- un réseau Wi-Fi local dédié ;
- une tolérance aux micro-coupures réseau ;
- une utilisation possible même si internet est coupé.

Le problème principal :

```txt
Tous les appareils doivent voir un état cohérent des commandes,
sans perdre d’action critique.
```

Exemples d’actions critiques :

- créer une commande ;
- ajouter un plat ;
- envoyer une commande en cuisine ;
- marquer un plat comme prêt ;
- encaisser ;
- annuler ou corriger une ligne.

---

## 2. Choix architectural

Architecture retenue :

```txt
SSE = notification serveur → clients
HTTP = actions et lecture des données
/sync = rattrapage des événements manqués
event_log = historique fiable
IndexedDB = file d’attente locale côté tablette
UUID = idempotence des actions critiques
PostgreSQL = source de vérité
```

En résumé :

```txt
SSE avertit.
HTTP agit et récupère.
La base de données décide.
Le journal d’événements permet de rattraper.
```

---

## 3. Schéma global

```txt
                  ┌────────────────────────────┐
                  │ Serveur POS local           │
                  │ FastAPI + PostgreSQL        │
                  └──────────────┬─────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
      ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
      │ Tablette 1   │   │ Écran cuisine│   │ Caisse       │
      │ serveur      │   │ KDS          │   │ centrale     │
      └──────────────┘   └──────────────┘   └──────────────┘

Flux principaux :

client → serveur : HTTP POST /orders, /items, /payments
serveur → client : SSE /stream
client → serveur : HTTP GET /sync?since_event_id=...
client → serveur : HTTP GET /orders/{id}
```

---

## 4. Pourquoi ne pas envoyer toute la donnée dans SSE ?

On pourrait envoyer toute la commande dans le flux SSE.

Mais ce n’est pas l’option la plus robuste.

On préfère envoyer un signal léger :

```json
{
  "type": "order_updated",
  "order_id": "ord_123",
  "event_id": 184
}
```

Puis le client décide :

```http
GET /orders/ord_123
```

ou :

```http
GET /sync?since_event_id=180
```

Cela permet :

- de garder SSE léger ;
- d’éviter de dépendre d’un message temps réel pour l’état complet ;
- de mieux gérer les coupures ;
- de relire les données depuis une route HTTP fiable ;
- de déboguer plus facilement.

---

## 5. Rôles des composants

## 5.1 HTTP REST

HTTP sert aux actions métier.

Exemples :

```http
POST /orders
POST /orders/{order_id}/items
POST /orders/{order_id}/send-to-kitchen
POST /orders/{order_id}/void
POST /payments
GET  /orders/{order_id}
GET  /tables
GET  /sync?since_event_id=123
```

Pour un POS, on évite souvent les vrais `DELETE`.

On préfère :

```txt
void, cancel, close, refund, correction
```

Parce qu’une caisse doit garder un historique.

---

## 5.2 SSE `/stream`

SSE sert aux notifications.

Endpoint :

```http
GET /stream
```

Exemple de message :

```txt
id: 184
event: sync_hint
data: {"event_id":184,"type":"order_updated","entity":"order","entity_id":"ord_123"}

```

Le client reçoit :

```js
source.addEventListener("sync_hint", async (event) => {
  const hint = JSON.parse(event.data);
  await syncFrom(hint.event_id);
});
```

---

## 5.3 `/sync`

`/sync` est le filet de sécurité.

Exemple :

```http
GET /sync?since_event_id=180
```

Réponse :

```json
{
  "server_event_id": 184,
  "events": [
    {
      "id": 181,
      "type": "order_created",
      "entity": "order",
      "entity_id": "ord_123",
      "payload": {"table_id": "T7"}
    },
    {
      "id": 182,
      "type": "order_item_added",
      "entity": "order",
      "entity_id": "ord_123",
      "payload": {"item_id": "burger", "quantity": 2}
    }
  ]
}
```

Le client applique les événements dans l’ordre, puis met à jour :

```txt
last_seen_event_id = 184
```

---

## 6. Le journal d’événements

La table centrale :

```sql
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    type TEXT NOT NULL,
    entity_type TEXT NOT NULL,
    entity_id TEXT NOT NULL,
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_events_id ON events(id);
CREATE INDEX idx_events_entity ON events(entity_type, entity_id);
```

Exemples d’événements :

```txt
order_created
order_item_added
order_item_removed
order_sent_to_kitchen
kitchen_item_started
kitchen_item_ready
order_closed
payment_recorded
order_voided
```

Le champ `id` est très important.

Il sert de version globale :

```txt
1, 2, 3, 4, 5, ...
```

C’est mieux qu’un simple timestamp, car l’ordre est clair et stable.

---

## 7. Écriture robuste avec UUID

Le problème classique :

```txt
La tablette envoie POST /orders.
Le serveur enregistre la commande.
Mais la réponse HTTP se perd.
La tablette réessaie.
Sans protection, on crée deux commandes.
```

Solution : chaque action critique reçoit un UUID côté client.

```json
{
  "client_action_id": "4fffb7e6-d985-45c7-b865-3a4d6f4e8808",
  "device_id": "tablet-03",
  "table_id": "T7",
  "items": []
}
```

Le serveur garde une table des actions déjà traitées :

```sql
CREATE TABLE processed_client_actions (
    client_action_id UUID PRIMARY KEY,
    device_id TEXT NOT NULL,
    request_hash TEXT NOT NULL,
    response JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Si la même action revient :

```txt
Le serveur ne la rejoue pas.
Il renvoie la même réponse que la première fois.
```

C’est l’idempotence.

---

## 8. Flux complet : création d’une commande

### Étape 1 — La tablette prépare l’action

Avant même d’envoyer au serveur, la tablette stocke en local :

```json
{
  "client_action_id": "uuid-123",
  "type": "create_order",
  "status": "pending",
  "payload": {
    "table_id": "T7"
  }
}
```

Stockage conseillé côté navigateur :

```txt
IndexedDB
```

### Étape 2 — La tablette envoie en HTTP

```http
POST /orders
Idempotency-Key: uuid-123
Content-Type: application/json
```

Body :

```json
{
  "client_action_id": "uuid-123",
  "table_id": "T7"
}
```

### Étape 3 — Le serveur écrit en transaction

Dans une même transaction :

```txt
1. vérifier client_action_id
2. créer la commande
3. créer un événement order_created
4. enregistrer la réponse liée à client_action_id
5. commit
```

### Étape 4 — Le serveur répond

```json
{
  "ok": true,
  "order_id": "ord_123",
  "event_id": 184
}
```

### Étape 5 — Le serveur notifie en SSE

```txt
id: 184
event: sync_hint
data: {"type":"order_created","entity":"order","entity_id":"ord_123","event_id":184}

```

### Étape 6 — Les autres clients rattrapent

Les autres appareils reçoivent le signal et font :

```http
GET /sync?since_event_id=183
```

ou :

```http
GET /orders/ord_123
```

---

## 9. Démarrage d’une tablette

Quand une tablette démarre :

```txt
1. lire last_seen_event_id depuis IndexedDB/localStorage
2. afficher l’état local en cache si disponible
3. appeler GET /sync?since_event_id=last_seen_event_id
4. appliquer les événements manqués
5. ouvrir EventSource('/stream')
6. relancer la file pending_actions[]
```

Important : ouvrir SSE ne suffit pas.

Il faut d’abord se synchroniser.

---

## 10. Reconnexion après coupure Wi-Fi

Quand le Wi-Fi revient :

```txt
1. EventSource se reconnecte automatiquement
2. le client appelle /sync?since_event_id=last_seen_event_id
3. le client rejoue ses pending_actions[]
4. les UUID empêchent les doublons
5. l’écran redevient cohérent
```

Si le client reçoit un événement avec un trou :

```txt
client avait 180
il reçoit directement 184
```

Alors il ne fait pas confiance au signal seul :

```http
GET /sync?since_event_id=180
```

---

## 11. Stratégie côté client

Pseudo-code TypeScript :

```ts
type SyncHint = {
  event_id: number;
  type: string;
  entity: string;
  entity_id: string;
};

let lastSeenEventId = Number(localStorage.getItem("lastSeenEventId") ?? "0");

async function sync() {
  const res = await fetch(`/sync?since_event_id=${lastSeenEventId}`);

  if (res.status === 409) {
    await fullReload();
    return;
  }

  const body = await res.json();

  for (const event of body.events) {
    applyEventToLocalState(event);
    lastSeenEventId = event.id;
  }

  localStorage.setItem("lastSeenEventId", String(lastSeenEventId));
}

function startSSE() {
  const source = new EventSource("/stream");

  source.addEventListener("sync_hint", async (message) => {
    const hint: SyncHint = JSON.parse(message.data);

    if (hint.event_id > lastSeenEventId) {
      await sync();
    }
  });

  source.onerror = () => {
    console.log("SSE reconnecting...");
  };
}

async function boot() {
  renderFromLocalCache();
  await sync();
  startSSE();
  startPendingActionWorker();
}
```

---

## 12. Stratégie côté serveur FastAPI

### 12.1 Formater un événement SSE

```python
import json


def format_sse(event: str, data: dict, event_id: int | None = None) -> str:
    lines = []

    if event_id is not None:
        lines.append(f"id: {event_id}")

    lines.append(f"event: {event}")
    lines.append(f"data: {json.dumps(data)}")

    return "\n".join(lines) + "\n\n"
```

### 12.2 Endpoint `/stream` simplifié

Exemple pédagogique :

```python
import asyncio
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse

app = FastAPI()

subscribers: set[asyncio.Queue] = set()


@app.get("/stream")
async def stream(request: Request):
    queue: asyncio.Queue = asyncio.Queue()
    subscribers.add(queue)

    async def generator():
        try:
            yield ": connected\n\n"

            while True:
                if await request.is_disconnected():
                    break

                try:
                    event = await asyncio.wait_for(queue.get(), timeout=20)
                    yield format_sse(
                        event="sync_hint",
                        event_id=event["event_id"],
                        data=event,
                    )
                except asyncio.TimeoutError:
                    yield ": heartbeat\n\n"
        finally:
            subscribers.discard(queue)

    return StreamingResponse(
        generator(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
            "X-Accel-Buffering": "no",
        },
    )
```

### 12.3 Diffuser un événement

```python
async def broadcast_sync_hint(event: dict):
    dead_queues = []

    for queue in subscribers:
        try:
            queue.put_nowait(event)
        except asyncio.QueueFull:
            dead_queues.append(queue)

    for queue in dead_queues:
        subscribers.discard(queue)
```

Ce modèle fonctionne pour un prototype avec un seul processus.

Pour une vraie production avec plusieurs workers, utiliser plutôt :

```txt
PostgreSQL LISTEN/NOTIFY
Redis Pub/Sub
NATS
```

---

## 13. Endpoint `/sync`

Pseudo-code :

```python
@app.get("/sync")
async def sync(since_event_id: int = 0):
    events = await db.fetch_all(
        """
        SELECT id, type, entity_type, entity_id, payload, created_at
        FROM events
        WHERE id > :since_event_id
        ORDER BY id ASC
        LIMIT 1000
        """,
        {"since_event_id": since_event_id},
    )

    if too_old(since_event_id):
        return JSONResponse(
            status_code=409,
            content={"error": "full_resync_required"},
        )

    server_event_id = events[-1]["id"] if events else since_event_id

    return {
        "server_event_id": server_event_id,
        "events": [dict(e) for e in events],
    }
```

Si un client est trop en retard et que les anciens événements ont été nettoyés :

```txt
Réponse 409 full_resync_required
```

Le client recharge alors l’état complet :

```http
GET /snapshot
```

---

## 14. Gestion des pannes

| Problème | Ce qui se passe | Solution |
|---|---|---|
| Signal SSE perdu | Le client n’est pas averti immédiatement | `/sync` au retour ou sur prochain signal |
| Wi-Fi coupé | EventSource reconnecte quand possible | `/sync` + pending actions |
| Réponse POST perdue | Le client ne sait pas si l’action est passée | retry avec même UUID |
| Action envoyée deux fois | Risque de doublon | table `processed_client_actions` |
| Serveur redémarre | connexions SSE coupées | reconnexion automatique + `/sync` |
| Client trop ancien | événements supprimés | `409 full_resync_required` + snapshot |
| Plusieurs workers | événements en mémoire non partagés | Redis ou PostgreSQL `LISTEN/NOTIFY` |
| Proxy bufferise | événements reçus en retard | désactiver buffering |

---

## 15. Nettoyage des événements

Tu peux garder les événements pendant :

```txt
7 jours
14 jours
30 jours
```

Pour un restaurant, 7 à 14 jours peuvent suffire pour la synchronisation.

Mais pour l’audit de caisse, il faut garder les données métier plus longtemps.

Différence importante :

```txt
events = journal technique de synchronisation
orders/payments/audit_log = historique métier et légal
```

Donc on peut nettoyer `events`, mais pas supprimer les traces de caisse importantes.

---

## 16. Sécurité et réseau local

Même en LAN, il faut prévoir :

- authentification des appareils ;
- rôles : serveur, cuisine, caisse, manager ;
- session ou token ;
- limitation des actions sensibles ;
- journal des corrections et annulations ;
- sauvegardes automatiques.

Pour une PWA, attention : certaines fonctions modernes du navigateur exigent un **contexte sécurisé**.

Sur un LAN, cela peut impliquer :

```txt
HTTPS local avec certificat installé
ou application installée différemment
ou usage sans service worker au début
```

IndexedDB fonctionne dans le navigateur, mais les service workers/PWA offline sont plus stricts.

---

## 17. Recommandation de version 1

Pour une première version solide :

```txt
Backend : FastAPI
DB : PostgreSQL
Frontend : React + Zustand
Local cache : IndexedDB
Notifications : SSE
Actions : HTTP POST avec client_action_id
Sync : GET /sync?since_event_id=N
Déploiement : Docker Compose sur serveur local
Réseau : routeur + point d’accès POS dédié
```

Ne commence pas avec trop de sophistication.

Version 1 réaliste :

```txt
1 serveur FastAPI
1 PostgreSQL
1 endpoint SSE
1 endpoint /sync
1 event_log
1 outbox IndexedDB
```

C’est déjà une base professionnelle.

---

## 18. Séquence recommandée d’implémentation

### Étape 1 — REST simple

Implémenter :

```txt
POST /orders
GET /orders/{id}
POST /orders/{id}/items
```

Sans SSE au début.

### Étape 2 — Event log

À chaque modification, écrire une ligne dans `events`.

### Étape 3 — `/sync`

Permettre aux clients de demander :

```txt
Donne-moi tout depuis mon dernier event_id.
```

### Étape 4 — SSE

Ajouter `/stream` qui envoie uniquement des `sync_hint`.

### Étape 5 — Client local

Ajouter :

```txt
last_seen_event_id
cache local
pending_actions[]
```

### Étape 6 — Idempotence

Ajouter `client_action_id` et la table `processed_client_actions`.

### Étape 7 — Tests de panne

Tester volontairement :

- coupure Wi-Fi ;
- redémarrage serveur ;
- double clic ;
- refresh tablette ;
- retard de 10 minutes ;
- coupure pendant un POST.

---

## 19. Tests concrets à faire dans ton laboratoire

### Test 1 — Notification simple

1. ouvrir tablette A et écran cuisine ;
2. créer une commande sur A ;
3. vérifier que la cuisine reçoit le signal SSE ;
4. vérifier que la cuisine recharge via `/sync`.

### Test 2 — Signal perdu

1. couper le Wi-Fi de la cuisine ;
2. créer 3 commandes ;
3. rallumer le Wi-Fi ;
4. vérifier que `/sync` récupère les 3 commandes.

### Test 3 — Double POST

1. envoyer deux fois le même `client_action_id` ;
2. vérifier qu’une seule commande est créée ;
3. vérifier que la même réponse est renvoyée.

### Test 4 — Redémarrage serveur

1. ouvrir 3 clients ;
2. redémarrer FastAPI ;
3. vérifier que les clients se reconnectent ;
4. vérifier qu’ils appellent `/sync`.

### Test 5 — Client trop ancien

1. simuler un `last_seen_event_id` très ancien ;
2. supprimer les vieux événements ;
3. appeler `/sync` ;
4. vérifier que le serveur répond `409 full_resync_required`.

---

## 20. Modèle mental final

```txt
Une action critique ne dépend jamais de SSE.
```

Elle passe par HTTP :

```txt
POST + UUID + transaction DB
```

SSE sert à dire :

```txt
Quelque chose a changé, synchronise-toi.
```

`/sync` sert à garantir :

```txt
Même si tu as raté des signaux, tu peux tout récupérer.
```

La base de données sert à garantir :

```txt
La vérité officielle est centralisée et transactionnelle.
```

La file locale sert à garantir :

```txt
Une coupure réseau ne fait pas disparaître l’action de l’utilisateur.
```

---

# Retrieval practice — Questions POS LAN

## A. Architecture générale

1. Dans cette architecture POS, quel est le rôle exact de SSE ?
2. Pourquoi les actions critiques passent-elles par HTTP POST et non par SSE ?
3. À quoi sert `/sync` ?
4. Pourquoi faut-il un `event_log` ?
5. Pourquoi préfère-t-on `event_id` à un timestamp pour synchroniser ?
6. Pourquoi la base de données reste-t-elle la source de vérité ?

## B. Robustesse

7. Une tablette perd le Wi-Fi pendant 5 minutes. Que doit-elle faire à la reconnexion ?
8. Un signal SSE `order_updated` est perdu. Pourquoi ce n’est pas grave ?
9. Le serveur redémarre. Que doivent faire les clients ?
10. La tablette envoie une commande, mais ne reçoit jamais la réponse HTTP. Quel mécanisme évite le doublon ?
11. Pourquoi faut-il stocker l’action localement avant de l’envoyer ?
12. Que faire si `/sync` indique que le client est trop ancien ?

## C. Modélisation

13. Quelles colonnes minimales faut-il dans une table `events` ?
14. À quoi sert la table `processed_client_actions` ?
15. Pourquoi éviter un vrai `DELETE` sur les commandes ?
16. Cite trois types d’événements utiles pour un POS restaurant.
17. Quelle différence fais-tu entre `events` et `audit_log` ?

## D. Scénarios

18. Deux serveurs ajoutent un plat à la même table presque en même temps. Que doit garantir le serveur ?
19. La cuisine reçoit directement l’événement `184` alors qu’elle avait `180`. Que doit-elle faire ?
20. Une tablette a 20 actions en attente dans IndexedDB. Dans quel ordre doit-elle les rejouer ?
21. Une requête POST est rejouée avec le même UUID mais un body différent. Comment le serveur devrait-il réagir ?
22. Pourquoi un système avec plusieurs workers FastAPI nécessite Redis ou PostgreSQL `LISTEN/NOTIFY` ?

## E. Décision technique

23. Pourquoi SSE est-il plus naturel que WebSocket dans cette architecture ?
24. Dans quel cas WebSocket redeviendrait-il intéressant ?
25. Pourquoi garder un polling lent de sécurité peut quand même être utile ?
26. Pourquoi tester les coupures réseau est aussi important que tester le code normal ?

---

# Corrigé rapide

1. Avertir les clients qu’une ressource a changé.  
2. SSE est serveur → client ; HTTP POST donne réponse, statut, transaction et retry propre.  
3. Récupérer les événements manqués depuis un dernier id connu.  
4. Garder une trace ordonnée des changements.  
5. Un entier croissant donne un ordre stable ; les timestamps peuvent être ambigus.  
6. Elle garantit l’état officiel et les transactions.  
7. Appeler `/sync`, rejouer les pending actions, rouvrir SSE.  
8. `/sync` ou la reconnexion rattrape les événements manqués.  
9. Reconnexion SSE + `/sync`.  
10. `client_action_id` / `Idempotency-Key`.  
11. Pour ne pas perdre l’action si le réseau tombe.  
12. Faire un full reload ou charger un snapshot.  
13. `id`, `type`, `entity_type`, `entity_id`, `payload`, `created_at`.  
14. Empêcher de traiter deux fois la même action client.  
15. Une caisse doit garder l’historique des corrections et annulations.  
16. `order_created`, `order_item_added`, `payment_recorded`.  
17. `events` sert à synchroniser ; `audit_log` sert à tracer légalement/métier.  
18. Sérialiser les écritures correctement et produire des événements ordonnés.  
19. Appeler `/sync?since_event_id=180`.  
20. Dans l’ordre de création locale, sauf règles métier particulières.  
21. Refuser, par exemple avec `409 Conflict` ou `422`, car un UUID doit représenter une seule action.  
22. La mémoire d’un worker n’est pas partagée avec les autres.  
23. Le besoin principal est serveur → client.  
24. Si les clients doivent envoyer un flux temps réel continu au serveur : chat, jeu, collaboration live.  
25. Pour détecter un client silencieux ou une connexion bloquée malgré SSE.  
26. Parce que la robustesse réelle apparaît surtout dans les pannes.

---

# Résumé final pour tes notes

```txt
Architecture POS LAN recommandée :

HTTP POST + UUID       = actions fiables
PostgreSQL             = source de vérité
Event log              = ordre global des changements
SSE                    = notification légère serveur → client
/sync                  = rattrapage des événements manqués
IndexedDB pending queue = tolérance aux coupures côté client
```

Phrase à retenir :

```txt
SSE rend le POS réactif ; /sync et la DB le rendent fiable.
```
