# infra

Docker Compose dev environment for [micasaestuya.com](https://micasaestuya.com).

## Structure

```
infra/
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
  api/       ← backend repo
  web/       ← frontend repo
  infra/     ← this repo
```

## Setup

### 1. Clone all repos as siblings

```bash
mkdir micasaestuya && cd micasaestuya
git clone git@gitlab.com:micasaestuya/api.git
git clone git@gitlab.com:micasaestuya/web.git
git clone git@gitlab.com:micasaestuya/infra.git
```

### 2. Configure environment variables (optional)

```bash
cp infra/.env.example infra/.env
# Edit .env if you need to change credentials or ports
```

Default values in `docker-compose.yml` work for local development without any changes.

### 3. Start all services

```bash
cd infra
docker compose up --build
```

### 4. Seed Redis (first time only)

Location autocomplete requires Redis to be populated:

```bash
docker exec express npm run redis:seed
```

### Stop services

```bash
docker compose down
```

Remove persistent volumes (clears MongoDB and Redis data):

```bash
docker compose down -v
```

## Inspecting MongoDB

Use [MongoDB Compass](https://www.mongodb.com/products/tools/compass) to connect directly to `mongodb://localhost:27017` with the credentials from your `.env` (`MONGO_DB_USERNAME` / `MONGO_DB_PASSWORD`).

## Dev Containers (VS Code)

The `web` repo includes a `.devcontainer/devcontainer.json` that references this repo's `docker-compose.yml`, giving you the full stack automatically when opening the project as a Dev Container.

**Requirement:** All repos must be cloned as siblings (see folder structure above).
