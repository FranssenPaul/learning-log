# Docker Internals — Session 2

> 22 février 2026

---

## Image, container, environment

- an image is a frozen filesystem
- a container lives as long as its PID 1 lives
- environment variables let one image behave differently across environments

---

## UV + Docker

Using `uv.lock` and copying dependency files first helps Docker cache builds efficiently.

Important detail in containers:

```bash
--host 0.0.0.0
```

without it, the app may only be reachable from inside the container.

---

## Isolation and limits

Namespaces isolate what the container sees.
Cgroups limit what it can consume.

Simple model:

```text
namespaces -> isolation
cgroups    -> resource limits
filesystem -> packaged runtime
```

---

## Network and volumes

Docker gives containers:

- an internal network
- service-name DNS resolution
- persistent volumes for stateful data

That makes it possible to keep PostgreSQL private while still allowing FastAPI to reach it.
