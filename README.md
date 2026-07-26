# Traefik Docker Proxy & Application Orchestration

This repository serves as the production configuration for running multi-service Docker container ecosystems securely and independently under a unified **Traefik v2** reverse proxy.

This architecture splits a previously monolithic docker-compose file into modular, decoupled micro-stacks. Each stack runs independently, communicating via a shared external network, while maintaining isolated storage, networks, and configurations.

---

## Repository Structure

```
.
├── docker-compose.yml     # Root Traefik v2 service configuration
├── start.md               # Step-by-step production startup guide
├── leagues/
│   └── docker-compose.yml # Leagues web app (Spring Boot), MySQL, & Redis
└── pms/
    └── docker-compose.yml # Plex Media Server (PMS) stack (11 services + Plex)
```

---

## Architectural Highlights

### 1. Unified External Routing Network (`traefik_proxy`)
All web-facing containers connect to a pre-created, external Docker bridge network named `traefik_proxy`. Traefik listens on this network and routes incoming subdomain traffic (HTTP/HTTPS) to the appropriate containers.

### 2. Isolated Application Databases (`leagues-network`)
The **Leagues** application is connected to two networks:
- `leagues-network` (private, internal bridge): Used for private database (MySQL) and cache (Redis) transactions.
- `traefik_proxy` (external bridge): Used to expose only the Spring Boot web app to Traefik.
The MySQL and Redis containers are kept off the external network, ensuring they are completely secure from external exposure.

### 3. Shared Security Middlewares
Instead of replicating security configurations (SSL redirection, HSTS headers, XSS filters, frame blocking) on each individual service, we define a global `secure-headers` middleware on the root Traefik container. Other services safely reference this middleware dynamically as `secure-headers@docker`.

---

## Quick Start Guide

Before deploying this stack in production, you must initialize the external network and secure Let's Encrypt key permissions.

For full, step-by-step deployment and rollback instructions, please refer to:
👉 **[start.md](start.md)**

---

## Domain and Environment Configuration

To run these stacks, define the following variables in a `.env` file in each stack's folder (or in the root folder):
- `DOMAINNAME`: Your top-level domain name (e.g., `delesio.com`).
- `PUID` / `PGID`: User ID and Group ID permissions for volume mounting (usually `1000`).
- `TZ`: Your local timezone (e.g., `America/New_York`).
