# POS Infrastructure — Session 2

> 22 février 2026

---

## Target architecture

```text
Tablets -> Nginx -> FastAPI -> PostgreSQL
```

- Nginx serves the frontend and proxies API traffic
- FastAPI handles business logic
- PostgreSQL stays on the internal Docker network

---

## Remote access

SSH is the foundation for remote administration.

Typical flow:

```bash
ssh-keygen -t ed25519
ssh-copy-id paul@192.168.1.100
ssh paul@192.168.1.100
```

VS Code Remote SSH can then turn a distant server into your working environment.

---

## Deployment choices

Two main contexts:

- a local HP EliteDesk in Cotonou for the offline-first POS
- a VPS in Europe for public projects

Tailscale can bridge remote access securely.

---

## Good practices

- dockerize early
- use `restart: always` in production
- persist real data in volumes
- keep databases off the public internet
- keep each project isolated with its own network and storage
