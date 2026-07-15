# Production Environment Startup Guide

Follow these steps to safely initialize, transition, and run your Docker container environment in production.

---

## Step 1: Stop Existing Containers
To avoid port conflicts (especially on ports `80` and `443` for Traefik) and container name collisions, shut down your old mono-compose setup first:
```bash
# Run this from the directory where your old docker-compose.yml is running
docker compose down
```

---

## Step 2: Initialize the Shared Network
Traefik and your applications communicate using a shared, external Docker network. Create it using:
```bash
docker network create traefik_proxy
```

---

## Step 3: Set Up Let's Encrypt Storage
Traefik stores automatic SSL certificates in a file named `acme.json` which must have strictly locked-down permissions (`600`):

```bash
# 1. Create the persistent folder in your repository root
mkdir -p letsencrypt

# 2. Initialize the file
touch letsencrypt/acme.json

# 3. Secure file permissions (required by Traefik; won't boot without this)
chmod 600 letsencrypt/acme.json
```

---

## Step 4: Configure Your Domain Name
Ensure you have a `.env` file in the repository root directory defining your domain name:
```env
DOMAINNAME=delesio.com
PUID=1000
PGID=1000
TZ=America/New_York
```
*(Make sure to adjust the values to match your system users and timezone if necessary.)*

---

## Step 5: Start the Stacks (In Order)

### 1. Launch Traefik (Root Directory)
Launch your reverse proxy first. This handles domain routing and SSL generation.
```bash
docker compose up -d
```
You can monitor the logs to verify it starts up without errors:
```bash
docker compose logs -f traefik
```

### 2. Launch Media Server Stacks (PMS)
Once Traefik is running, spin up your media stack:
```bash
cd pms
docker compose up -d
```

### 3. Launch Leagues Mono App
Spin up your Leagues application and database/redis services:
```bash
cd ../leagues
docker compose up -d
```

---

## Step 6: Verify Access
Your applications will now be routing through Traefik v2 and secured with auto-provisioned SSL. You can access them at:
- **Traefik Dashboard**: `https://traefik.yourdomain.com` *(Protected by basic auth)*
- **Leagues App**: `https://leagues.yourdomain.com`
- **SABnzbd**: `https://downloads.yourdomain.com`
- **Jellyfin**: `https://jellyfin.yourdomain.com`
- **Tautulli**: `https://plexstats.yourdomain.com`
- **Home Assistant**: `https://ha.yourdomain.com`
- **Sonarr**: `https://tv.yourdomain.com`
- **Radarr**: `https://movies.yourdomain.com`
- **Readarr**: `https://books.yourdomain.com`
- **Ubooquity**: `https://opds.yourdomain.com`
- **Prowlarr**: `https://index.yourdomain.com`
- **Overseerr**: `https://request.yourdomain.com`
- **Komga**: `https://comics.yourdomain.com`
