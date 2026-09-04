# Traefik Docker Proxy & Application Orchestration

This repository serves as the production configuration for running multi-service Docker container ecosystems securely and independently under a unified **Traefik v2** reverse proxy.

This architecture splits a previously monolithic docker-compose file into modular, decoupled micro-stacks. Each stack runs independently, communicating via a shared external network, while maintaining isolated storage, networks, and configurations.

---

## Repository Structure

```
.
├── docker-compose.yml     # Root Traefik v2 service configuration
├── README.md              # Repository Overview & Documentation
├── start.md               # Step-by-step production startup guide
├── bass/
│   └── docker-compose.yml # Bass Lessons Stack (Node.js backend, React frontend, PostgreSQL)
├── leagues/
│   └── docker-compose.yml # Leagues web app Stack (Spring Boot, MySQL, Redis)
└── pms/
    └── docker-compose.yml # Plex Media Server (PMS) Stack (11 media services + Plex)
```

---

## Complete List of Orchestrated Services

All external routing domains use the base domain configured via `${DOMAINNAME:-delesio.com}` in your environment config.

### 1. Reverse Proxy Stack (Root)
| Service Name | Container Name | Host Port | Routing Host Rule | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Traefik** | `traefik-v2` | `80`, `443` | `traefik.yourdomain.com` | Unified Edge Router & Let's Encrypt auto-resolver |

### 2. Media Server Stack (`pms/`)
| Service Name | Container Name | Host Port | Routing Host Rule | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Plex** | `plex-v2` | `Host Mode` | Direct IP / Port `32400` | Primary Plex Media Server |
| **SABnzbd** | `sabnzbd-v2` | `7070` | `downloads.yourdomain.com` | Usenet downloader client |
| **Jellyfin** | `jellyfin-v2` | `8096` | `jellyfin.yourdomain.com` | Alternative media server |
| **Tautulli** | `tautulli-v2` | `8181` | `plexstats.yourdomain.com` | Plex server monitoring & analytics |
| **Home Assistant** | `homeassistant-v2` | None | `ha.yourdomain.com` | Smart home automation dashboard |
| **Sonarr** | `sonarr-v2` | None | `tv.yourdomain.com` | Automated television show manager |
| **Radarr** | `radarr-v2` | `7878` | `movies.yourdomain.com` | Automated movie manager |
| **Readarr** | `readarr-v2` | `8787` | `books.yourdomain.com` | Automated ebook manager |
| **Speakarr** | `speakarr-v2` | `8788` | `audiobooks.yourdomain.com` | Automated audiobook manager (Readarr profile) |
| **Ubooquity** | `ubooquity-v2` | `2202`, `2203` | `opds.yourdomain.com` | Ebook and comic book content server |
| **Prowlarr** | `prowlarr-v2` | `9696` | `index.yourdomain.com` | Unified torrent and usenet indexer manager |
| **Overseerr** | `overseerr-v2` | `5055` | `request.yourdomain.com` | Media request system for Plex/Jellyfin |
| **Komga** | `komga-v2` | `8082` | `comics.yourdomain.com` | Modern comic and manga server |

### 3. Leagues Application Stack (`leagues/`)
| Service Name | Container Name | Host Port | Routing Host Rule | Description |
| :--- | :--- | :--- | :--- | :--- |
| **App** | `leagues-app-v2` | `8083` | `leagues.yourdomain.com` | Spring Boot Monolith application |
| **MySQL** | `leagues-mysql-v2` | None | *Internal Only* | Leagues relational storage database |
| **Redis** | `leagues-redis-v2` | `6379` | *Internal Only* | Leagues active session and data cache |

### 4. Bass Lessons Stack (`bass/`)
| Service Name | Container Name | Host Port | Routing Host Rule | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend** | `bass_lesson_frontend_prod` | None | `bass.yourdomain.com` | React production user interface |
| **Backend** | `bass_lesson_backend_prod` | None | `bass.yourdomain.com/api` | Node.js Express server backend API |
| **DB** | `bass_lesson_db_prod` | None | *Internal Only* | PostgreSQL relational database |

---

## 🛠️ Post-Deployment Application Customizations

Certain services require minor manual configurations inside their respective interfaces to function perfectly due to third-party outages.

### 📚 Readarr & Speakarr Metadata Resolution Fix
By default, Readarr/Speakarr relies on the `api.bookinfo.club` metadata proxy server to fetch books, authors, and metadata. Because this official server has been decommissioned, searching or mapping books will fail with a `Name does not resolve` DNS error in the logs.

Follow these steps to point both **Readarr** and **Speakarr** to the active community-maintained backup server:

1. Open your browser and go to your service dashboard:
   * **Readarr**: `https://books.yourdomain.com`
   * **Speakarr**: `https://audiobooks.yourdomain.com`
2. Once logged in, go to the address bar and manually append **`/settings/development`** to the end of your URL (e.g. `https://audiobooks.yourdomain.com/settings/development`).
3. Scroll down and locate the **"Metadata Provider Source"** field (currently set to `https://api.bookinfo.club`).
4. Replace that value with the active community backup endpoint:
   👉 **`https://api.bookinfo.pro`**
5. Scroll and click **Save** at the top or bottom of the page.

---

## Quick Start Guide

Before deploying these stacks in production, you must initialize the shared external network and secure Let's Encrypt key permissions.

For full, step-by-step deployment instructions, please refer to:
👉 **[start.md](start.md)**

---

## Domain and Environment Configuration

To run these stacks, define the following variables in a `.env` file in each stack's folder (or in the root folder):
- `DOMAINNAME`: Your top-level domain name (e.g., `delesio.com`).
- `PUID` / `PGID`: User ID and Group ID permissions for volume mounting (usually `1000`).
- `TZ`: Your local timezone (e.g., `America/New_York`).
