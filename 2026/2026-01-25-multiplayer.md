# Learning Log — Multiplayer Pong Architecture

## Core idea

Phaser alone is not enough for real multiplayer. You need a backend.

---

## Recommended classroom setup

For a local classroom network:

- backend runs on your machine
- phones connect over local Wi-Fi
- internet is not required

Recommended backend:

- **FastAPI + WebSockets**

---

## Authoritative server model

The server should own the truth.

- clients send input only
- server simulates ball, paddles, and score
- server broadcasts the current state

This avoids cheating and keeps the game consistent for both players.

---

## Why FastAPI works well

- async by default
- clean WebSocket support
- reliable for small LAN games
- easy to extend later with lobby or admin features

Key rules:

- keep the game loop async
- never block the event loop
- keep game state on the server

---

## Alternative

Cloudflare Workers + Durable Objects can work for internet multiplayer, but they add more cost and complexity.

Simple rule:

- LAN-only: FastAPI on your PC
- internet multiplayer: Workers + Durable Objects
