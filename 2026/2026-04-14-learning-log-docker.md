# Learning Log — Docker

**Date :** 2026-04-14
**Outil / Mécanisme :** Docker + Docker Compose
**Couche fullstack :** Infra — transversal à toutes les couches
**Projet concerné :** POSEN, Think.Play (architecture Nginx + FastAPI + PostgreSQL)
**Connexions :** Nginx (reverse proxy), FastAPI (backend containerisé), PostgreSQL (données persistées), Git (workflow CI/CD futur)

---

## 🔍 Quel problème cet outil résout-il ?

> "Mon code tourne sur ma machine mais pas sur la tienne."

Docker résout le problème de l'**environnement** : emballer une application avec tout ce dont elle a besoin (runtime, dépendances, configuration) dans une unité portable et reproductible. L'environnement devient du code — versionnable, partageable, déployable.

Pour POSEN et Think.Play : garantir que l'application tourne de façon identique sur le ThinkCentre à Cotonou et sur ma machine de développement à Bruxelles.

---

## 🏗️ Niveau architectural — ce que je dois posséder seul

### Les objets fondamentaux

- **Image** — un instantané figé d'un environnement. Read-only. Construite depuis un Dockerfile. C'est le *moule* — elle ne tourne pas, elle ne change pas.

- **Container** — une instance en cours d'exécution d'une image. Éphémère par nature — tout ce qui est écrit dedans disparaît à sa suppression. C'est le *coulage* du moule.

- **Dockerfile** — le script de construction d'une image. Décrit les couches successives : OS de base, installation des dépendances, copie du code, commande de démarrage.

- **Volume** — mécanisme de persistance des données. Stocké sur l'hôte, monté dans le container. Survit aux suppressions de containers. Indispensable pour les bases de données.

- **Network** — réseau virtuel entre containers. Dans un Compose, les services se trouvent par leur **nom de service** — pas par IP. Isolation par défaut entre projets Compose différents.

- **Docker Compose** — orchestrateur multi-containers. Décrit dans `docker-compose.yml` l'ensemble des services, leurs relations, leurs volumes, leurs networks. Un seul fichier pour toute l'architecture.

### Le schéma mental

```
docker-compose.yml
       │
       ├── service: nginx
       │     image: nginx:alpine
       │     ports: 80:80
       │     depends_on: api
       │
       ├── service: api
       │     build: ./backend        ← Dockerfile dans ./backend
       │     volumes: ./backend:/app  ← bind mount pour dev (hot reload)
       │     depends_on: db
       │     environment: DATABASE_URL
       │
       └── service: db
             image: postgres:15
             volumes: postgres_data:/var/lib/postgresql/data  ← volume nommé
             environment: POSTGRES_PASSWORD

volumes:
  postgres_data:    ← déclaré ici = géré par Docker = persiste entre compose down/up
```

**Le flux d'une requête dans cette architecture :**
```
Client (navigateur / tablette Android)
    ↓ HTTP :80
Nginx (container)          ← sert le frontend statique
    ↓ proxy_pass :8000     ← redirige /api/* vers FastAPI
FastAPI (container)        ← traite la logique métier
    ↓ SQLAlchemy
PostgreSQL (container)     ← stocke les données
    ↓ volume nommé
Disque hôte                ← données persistées ici
```

### Les décisions architecturales clés

**Pourquoi trois containers séparés plutôt qu'un seul ?**
Séparation des responsabilités. Chaque container fait une chose. On peut redémarrer FastAPI sans toucher PostgreSQL. On peut mettre à jour Nginx sans reconstruire l'image Python. La scalabilité et la maintenance sont indépendantes.

**Bind mount vs volume nommé — quand choisir ?**
- **Bind mount** (`./code:/app`) : pour le développement — le code local est reflété en temps réel dans le container. Hot reload possible. Ne pas utiliser en production.
- **Volume nommé** (`postgres_data:/var/lib/postgresql/data`) : pour les données qui doivent persister. Géré par Docker, indépendant du système de fichiers hôte. À utiliser pour les bases de données, en dev comme en prod.

**Pourquoi les services se trouvent par nom dans Compose ?**
Docker Compose crée un réseau virtuel automatique. Dans ce réseau, chaque service est résolvable par son nom de service. `api` peut joindre `db` avec `postgresql://db:5432/...` — pas besoin d'IP.

**`docker compose up` vs `docker compose up --build` :**
Sans `--build`, Docker utilise les images existantes même si le Dockerfile a changé. Avec `--build`, il reconstruit les images depuis les Dockerfiles. Toujours utiliser `--build` après modification d'un Dockerfile ou des dépendances.

---

## ⚙️ Niveau implémentation — ce que je sais faire

### Patterns essentiels

```bash
# Pattern 1 — Démarrer l'architecture complète
docker compose up -d          # -d = detached, tourne en arrière-plan
docker compose up -d --build  # après changement de Dockerfile

# Pattern 2 — Arrêter sans perdre les données
docker compose down           # arrête et supprime les containers, GARDE les volumes
docker compose down -v        # supprime aussi les volumes — DANGER pour les données

# Pattern 3 — Lire les logs
docker compose logs -f         # tous les services, en continu
docker compose logs -f api     # un seul service

# Pattern 4 — Entrer dans un container
docker compose exec api bash   # shell dans le container api
docker compose exec db psql -U postgres  # psql directement

# Pattern 5 — Reconstruire un seul service
docker compose up -d --build api   # reconstruit seulement api, pas db ni nginx

# Pattern 6 — Voir l'état des containers
docker compose ps              # état de tous les services du projet
docker ps                      # tous les containers sur la machine

# Pattern 7 — Inspecter un volume
docker volume ls               # liste tous les volumes
docker volume inspect postgres_data  # détails d'un volume
```

```dockerfile
# Pattern 8 — Dockerfile Python minimal pour FastAPI
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
# Copier requirements AVANT le code = cache Docker préservé si seul le code change

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
# 0.0.0.0 obligatoire — pas 127.0.0.1 qui serait inaccessible depuis l'extérieur du container
```

### Pièges courants

- **`--host 127.0.0.1` dans le container** → le service est inaccessible depuis l'extérieur du container. Toujours `0.0.0.0` pour un service qui doit être joint par d'autres containers ou par l'hôte.

- **Modifier le code sans `--build`** → si le code est COPYé dans l'image (prod), les changements ne sont pas pris en compte sans reconstruction. En dev avec bind mount, pas de problème.

- **`docker compose down -v` sur la base de données** → supprime le volume et toutes les données. À ne jamais faire en production sans backup.

- **Ordre de démarrage avec `depends_on`** → `depends_on` garantit l'ordre de démarrage des containers, pas que le service *à l'intérieur* est prêt. PostgreSQL peut démarrer avant que son serveur soit prêt à accepter des connexions. Solution : healthcheck ou retry dans le code applicatif.

- **Nom de service ≠ nom du container** → dans le réseau Compose, on utilise le nom du service (clé dans `docker-compose.yml`). Le nom du container (visible dans `docker ps`) est différent et ne doit pas être utilisé pour la communication inter-services.

---

## 🔗 Connexions avec d'autres outils

```
Git (versionne Dockerfile et docker-compose.yml)
       ↓
Docker Build (construit les images depuis Dockerfile)
       ↓
Docker Compose (orchestre les containers)
       ├── Nginx container    ← sert le frontend, proxy vers FastAPI
       ├── FastAPI container  ← logique métier Python
       └── PostgreSQL container ← données persistées via volume
              ↓
         Disque hôte (volumes nommés)
```

**Ce que Docker reçoit :** le code source, les Dockerfiles, les fichiers de config
**Ce que Docker produit :** des containers en cours d'exécution, isolés, communicants
**Ce sur quoi Docker délègue :** le contenu applicatif (FastAPI gère les routes, PostgreSQL gère les données)

---

## 📍 Où j'en suis

```
[x] Niveau architectural compris
[x] Schéma mental stable (je peux le dessiner sans support)
[x] Patterns essentiels maîtrisés
[x] Utilisé sur un projet réel (Think.Play, POSEN)
[ ] Devient transparent (charge libérée) ← en cours
```

---

## 🔁 Retrieval Practice

> Utilisation : papier, sans support, 5 minutes avant chaque session impliquant Docker.
> Objectif : quand le niveau 1 ne demande plus d'effort → Docker est automatisé.

### Niveau 1 — Objets et commandes (à automatiser complètement)

1. Quelle est la différence fondamentale entre une image et un container ?
2. Quelle commande pour démarrer tous les services en arrière-plan ?
3. Quelle commande pour voir les logs du service `api` en continu ?
4. Comment entrer dans un shell dans le container `api` ?
5. Quelle est la différence entre `docker compose down` et `docker compose down -v` ?
6. Pourquoi doit-on spécifier `--host 0.0.0.0` dans uvicorn ?
7. Comment un service `api` appelle-t-il le service `db` dans le réseau Compose ?
8. Quelle commande pour reconstruire seulement le service `api` sans toucher `db` ?
9. Quelle est la différence entre un bind mount et un volume nommé ?
10. Quand utilise-t-on `--build` avec `docker compose up` ?

### Niveau 2 — Architecture et décisions (à comprendre structurellement)

1. Pourquoi sépare-t-on Nginx, FastAPI et PostgreSQL en trois containers distincts plutôt qu'un seul ?
2. Pourquoi les données PostgreSQL doivent-elles être dans un volume nommé et non dans le container lui-même ?
3. `depends_on: db` garantit-il que PostgreSQL est *prêt* à accepter des connexions quand `api` démarre ? Pourquoi ?
4. Pourquoi copie-t-on `requirements.txt` *avant* le reste du code dans le Dockerfile ?
5. Dans quel cas un bind mount est-il préférable à un volume nommé, et pourquoi ne l'utilise-t-on pas en production ?

### Niveau 3 — Diagnostic et conception (charge germane)

1. Ton container `api` redémarre en boucle. Quelle est ta stratégie de diagnostic, étape par étape ?
2. FastAPI ne peut pas joindre PostgreSQL au démarrage malgré `depends_on`. Que se passe-t-il probablement, et comment le résoudre ?
3. Tu déploies sur le ThinkCentre à Cotonou. Quelles différences dans ta configuration Docker entre l'environnement de dev (Bruxelles) et la production (Cotonou) ?
4. Comment gères-tu les secrets (mots de passe, clés API) dans ton `docker-compose.yml` sans les committer dans Git ?

---

## ❓ Questions ouvertes

- Comment gérer les migrations Alembic au démarrage du container `api` — dans le Dockerfile, dans le CMD, ou dans un script d'entrée ?
- À quel moment passer de Docker Compose à un orchestrateur plus robuste (Portainer, Kubernetes léger) pour POSEN en production à Cotonou ?
- Comment gérer les backups automatiques du volume PostgreSQL dans un contexte de production à faible infrastructure ?

---

## 📚 Références

- **Documentation officielle :** https://docs.docker.com/compose/
- **Dockerfile best practices :** https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
- **Sweller, J. (1988)** — *Cognitive Load During Problem Solving* — la théorie qui explique pourquoi automatiser Docker libère de la mémoire de travail pour l'architecture
- **Vers Nginx** — comment Nginx s'intègre comme reverse proxy dans cette architecture
- **Vers PostgreSQL** — volumes, backups, connexions depuis SQLAlchemy

---

## 📝 Journal des sessions

### 2026-04-14
**Ce qui était encore de la charge extrinsèque :** l'ordre exact des flags dans `docker compose up`, la distinction bind mount / volume nommé sous la pression du projet
**Ce qui est devenu automatique :** le schéma mental Nginx → FastAPI → PostgreSQL, la communication inter-services par nom
**Ajout au retrieval :** question sur `depends_on` vs readiness — piège non évident
