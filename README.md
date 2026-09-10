# micasaestuya-infra

Docker Compose dev environment for [micasaestuya.com](https://micasaestuya.com).

## Structure

```
micasaestuya-infra/
  docker-compose.yml   # Full dev stack (all services)
  .env.example         # Environment variables template
  seed/
    mongo-init.js      # MongoDB initialization script
    regions.json       # Region data (seed)
    regions_cu.json    # Cuba regions
    regions_do.json    # Dominican Republic regions
```

## Architecture

```
  Browser
     │
     │ :3000
     ▼
┌──────────────┐         ┌──────────────────┐
│     nuxt     │         │     express      │
│   Nuxt 3     │────────▶│   Node.js API    │
│   port 3000  │         │   port 3001      │
└──────────────┘         └────────┬─────────┘
                                  │
                     ┌────────────┴────────────┐
                     │                         │
           ┌─────────┴──────────┐   ┌──────────┴─────────┐
           │       mongo        │   │        redis        │
           │     MongoDB        │   │   location cache    │
           │    port 27017      │   │     port 6379       │
           └────────────────────┘   └────────────────────┘
```

> **Note:** Nuxt runs as CSR. All API calls go from the browser to `localhost:3001`, not through Docker's internal network.

## Ports

| Port  | Service | Description        |
|-------|---------|---------------------|
| 3000  | nuxt    | Nuxt 3 dev server   |
| 24678 | nuxt    | Vite HMR websocket  |
| 3001  | express | Express REST API    |
| 27017 | mongo   | MongoDB             |
| 6379  | redis   | Redis               |

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Git

## Folder structure requirement

This Docker Compose uses relative paths to mount source code from sibling repos. **All repos must be cloned as siblings inside the same parent directory:**

```
micasaestuya/
  micasaestuya-api/     ← backend repo
  micasaestuya-web/     ← frontend repo
  micasaestuya-infra/   ← this repo
```

## Setup

### 1. Clone all repos as siblings

```bash
mkdir micasaestuya && cd micasaestuya
git clone git@github.com:marlonbdez/micasaestuya-api.git
git clone git@github.com:marlonbdez/micasaestuya-web.git
git clone git@github.com:marlonbdez/micasaestuya-infra.git
```

### 2. Configure environment variables (optional)

```bash
cp micasaestuya-infra/.env.example micasaestuya-infra/.env
# Edit .env if you need to change credentials or ports
```

Default values in `docker-compose.yml` work for local development without any changes.

### 3. Start all services

```bash
cd micasaestuya-infra
docker compose up --build
```

This starts:

- **Frontend**: http://localhost:3000 (Nuxt)
- **API**: http://localhost:3001 (Express)
- **MongoDB**: localhost:27017
- **Redis**: localhost:6379

### 4. Seed Redis (first time only)

Location autocomplete requires Redis to be populated:

```bash
docker exec express npm run redis:seed
```

## Useful commands

```bash
# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove volumes (clears MongoDB and Redis data)
docker compose down -v

# Restart a specific service
docker compose restart express

# Check service status
docker compose ps
```

## Troubleshooting

**Port already in use?** Edit `docker-compose.yml` and change the host port, e.g. `"3000:3000"` → `"3002:3000"`.

**Services won't start?**

```bash
docker compose down -v  # remove volumes and try again
docker compose up --build
```

## Inspecting MongoDB

Use [MongoDB Compass](https://www.mongodb.com/products/tools/compass) to connect directly to `mongodb://localhost:27017` with the credentials from your `.env` (`MONGO_DB_USERNAME` / `MONGO_DB_PASSWORD`).

## Dev Containers (VS Code)

The `micasaestuya-web` and `micasaestuya-api` repos each include a `.devcontainer/devcontainer.json` that references this repo's `docker-compose.yml`, giving you the full stack automatically when opening either project as a Dev Container.

**Requirement:** All repos must be cloned as siblings (see folder structure above).
