# Le `docker-compose.yml` expliqué

> **En une phrase** : `docker-compose.yml` est le **plan d'architecte** de ton application — il décrit quels bâtiments construire, comment ils communiquent, et ce qu'ils contiennent.

---

## L'analogie du restaurant

Imagine que ton application est un **restaurant** :

| Monde réel | Docker |
|---|---|
| Le restaurant entier | ton application |
| La salle + cuisine | les conteneurs |
| Le plan d'architecte | `docker-compose.yml` |
| Les livreurs de matières premières | les images Docker |
| Le stock de nourriture | les volumes |
| L'interphone cuisine ↔ salle | le réseau Docker interne |

---

## Le fichier décortiqué

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8000"
    environment:
      - DATABASE_URL=postgresql://paul:secret@db:5432/hellocompose
    depends_on:
      - db
    restart: always

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=paul
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=hellocompose
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  postgres_data:
```

---

## `services` — Les acteurs

`services` déclare tous les **conteneurs** qui vont tourner ensemble.

```yaml
services:
  api:   ← conteneur 1 : ton FastAPI
  db:    ← conteneur 2 : PostgreSQL
```

C'est comme la liste des **pièces du restaurant** :
- `api` = la salle où les clients arrivent
- `db` = la cuisine où la vraie magie opère

---

## `api` — Ton FastAPI

### `build: .`

```yaml
build: .
```

> *"Construis l'image depuis le Dockerfile qui est dans ce dossier."*

Le `.` c'est le **contexte de build** — le dossier courant.
Docker lit ton `Dockerfile` et construit une image sur mesure.

À l'opposé, `db` utilise `image: postgres:16-alpine` — une image toute faite, pas besoin de Dockerfile.

---

### `ports` — La porte d'entrée

```yaml
ports:
  - "8080:8000"
```

```
ton Ubuntu écoute sur 8080
        ↕  (pont Docker)
conteneur écoute sur 8000
```

**Analogie** : c'est le **numéro de rue** de ton restaurant.
- `8080` = numéro visible depuis l'extérieur (ton navigateur, curl)
- `8000` = numéro interne (uvicorn dans le conteneur)

Tu peux avoir deux restaurants sur la même rue avec des numéros différents :
```yaml
api-v1:
  ports: ["8080:8000"]   ← accessible via :8080
api-v2:
  ports: ["8081:8000"]   ← accessible via :8081
```

---

### `environment` — Les variables d'environnement

```yaml
environment:
  - DATABASE_URL=postgresql://paul:secret@db:5432/hellocompose
```

C'est comme les **instructions données au personnel** avant l'ouverture :
*"Le stock est au sous-sol, code d'accès : secret, adresse : db."*

Décomposons le DATABASE_URL :

```
postgresql://paul:secret@db:5432/hellocompose
     ↑          ↑     ↑    ↑   ↑       ↑
  protocole  user  pass  host port   base de données
```

**Le point magique : `@db`**

`db` n'est pas une IP — c'est le **nom du service** PostgreSQL dans ce fichier.
Docker a son propre DNS interne et résout automatiquement `db` → `172.18.0.2` (ou peu importe l'IP attribuée).

C'est comme une **liste de contacts interne** :
> *"Appelle 'db' — Docker sait où il habite."*

---

### `depends_on` — L'ordre de démarrage

```yaml
depends_on:
  - db
```

> *"Ne démarre pas `api` avant que `db` soit lancé."*

Sans ça, FastAPI essaierait de se connecter à PostgreSQL qui n'est pas encore prêt → erreur au démarrage.

**Attention** : `depends_on` attend que le conteneur **démarre**, pas qu'il soit **prêt**.
PostgreSQL peut prendre quelques secondes à accepter des connexions après son démarrage.
Pour du code de production, on ajoute un health check — mais pour apprendre, c'est suffisant.

---

### `restart: always` — La résilience

```yaml
restart: always
```

> *"Si le conteneur plante ou si la machine redémarre → relance automatiquement."*

| Valeur | Comportement |
|---|---|
| `no` | Ne redémarre jamais (défaut) |
| `always` | Redémarre toujours, même après reboot |
| `on-failure` | Redémarre seulement si erreur |
| `unless-stopped` | Redémarre sauf si arrêté manuellement |

Pour Cotonou avec les coupures de courant : **`restart: always` est indispensable**.

---

## `db` — PostgreSQL

### `image: postgres:16-alpine`

```yaml
image: postgres:16-alpine
```

Pas de `build`, pas de Dockerfile — on utilise directement l'image officielle PostgreSQL.

```
16       → version de PostgreSQL
alpine   → basé sur Alpine Linux (léger, ~276 Mo vs ~430 Mo pour Debian)
```

---

### `environment` — Configuration PostgreSQL

```yaml
environment:
  - POSTGRES_USER=paul
  - POSTGRES_PASSWORD=secret
  - POSTGRES_DB=hellocompose
```

Ces variables sont **comprises par l'image PostgreSQL officielle**.
Au premier démarrage, PostgreSQL :
1. Crée l'utilisateur `paul` avec le mot de passe `secret`
2. Crée la base `hellocompose`
3. Donne tous les droits à `paul` sur `hellocompose`

C'est comme un **formulaire d'inscription automatique** — tu remplis les cases, PostgreSQL fait le reste.

---

### `volumes` — La mémoire persistante

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

```
nom du volume       chemin dans le conteneur
postgres_data   :   /var/lib/postgresql/data
```

**Sans volume :**
```
conteneur arrêté → tout effacé → tes données perdues 😱
```

**Avec volume :**
```
conteneur arrêté → données sur ton Ubuntu → conteneur redémarre → données intactes ✅
```

**Analogie** : C'est le **disque dur externe** branché sur ton conteneur.
Le conteneur peut mourir, le disque dur survit.

Sur ton Ubuntu, les données vivent ici :
```
/var/lib/docker/volumes/hello-compose_postgres_data/_data/
```

---

## `volumes` (section racine) — La déclaration

```yaml
volumes:
  postgres_data:
```

Cette section **déclare** les volumes utilisés dans le fichier.
Sans cette déclaration, Docker refuserait de démarrer.

C'est comme la **liste des ressources** dans un plan de construction :
*"Ce projet utilise un espace de stockage nommé `postgres_data`."*

---

## Le réseau invisible

Tu ne vois pas de section `networks` dans ce fichier — et pourtant les deux conteneurs se parlent.

Docker Compose crée automatiquement un **réseau privé** pour tous les services :

```
réseau hello-compose_default
├── api  → 172.18.0.2
└── db   → 172.18.0.3
```

Les deux conteneurs se voient par leur nom de service.
PostgreSQL n'est **jamais exposé à l'extérieur** — seulement `api` sur le port 8080.

```
Internet / ton Ubuntu
        ↓ port 8080
       [api]
        ↓ réseau interne Docker (port 5432)
       [db]  ← invisible depuis l'extérieur
```

---

## Résumé visuel

```
docker-compose.yml
│
├── services
│   ├── api ──────────────────────────────────────────────┐
│   │   ├── build: .          (construit depuis Dockerfile)│
│   │   ├── ports: 8080:8000  (porte vers l'extérieur)    │
│   │   ├── environment       (DATABASE_URL avec @db)      │
│   │   ├── depends_on: db    (attends que db démarre)     │
│   │   └── restart: always   (résilience)                 │
│   │                                                      │
│   └── db ───────────────────────────────────────────────┘
│       ├── image: postgres:16-alpine  (image toute faite)
│       ├── environment                (user/pass/db)
│       ├── volumes: postgres_data     (données persistantes)
│       └── restart: always            (résilience)
│
└── volumes
    └── postgres_data  (déclaration du volume)
```

---

## Les commandes clés

```bash
# Démarrer tout (avec build si nécessaire)
docker compose up

# Démarrer en arrière-plan
docker compose up -d

# Voir les logs
docker compose logs
docker compose logs api
docker compose logs -f db   # en temps réel

# Arrêter sans supprimer
docker compose stop

# Arrêter et supprimer les conteneurs
docker compose down

# Arrêter, supprimer conteneurs ET volumes (⚠️ perd les données !)
docker compose down -v

# Reconstruire l'image après modification du code
docker compose up --build
```

---

> **À retenir** : Le `docker-compose.yml` ne remplace pas le `Dockerfile` — il l'orchestre.
> `Dockerfile` = recette d'une image. `docker-compose.yml` = partition d'un orchestre.
