# infra — Agent Guide

> **`CLAUDE.md` can't be centralized — it stays in this repo for technical
> reasons** (coding tools read it automatically when working here). But
> **the project's source of truth is `micasaestuya-docs`**: architecture,
> decisions, product vision and anything business-related are written there,
> never in a `CLAUDE.md`. This file is only this repo's code style guide.

## What's here

Docker Compose dev environment for micasaestuya.com.

```
infra/
  docker-compose.yml   # All services: nuxt, express, mongo, redis
  .env.example         # Environment variables template
  seed/
    mongo-init.js      # MongoDB init script (runs on first container start)
    regions.json       # Region seed data
    regions_cu.json    # Cuba regions
    regions_do.json    # Dominican Republic regions
```

## Sibling repos

Four independent git repos, all on GitHub under `marlonbdez`. A change touching
two of them needs two commits, and **`api` goes first** whenever `web` depends
on one of its endpoints.

```
micasaestuya/
  micasaestuya-api/    ← github.com/marlonbdez/micasaestuya-api
  micasaestuya-web/    ← github.com/marlonbdez/micasaestuya-web
  micasaestuya-infra/  ← this repo
  micasaestuya-docs/   ← architecture, ADRs, product vision, status — start here
```

**Everything cross-cutting lives in `micasaestuya-docs/`** — start with
`product-vision.md`, then `status.md`. `web`-specific implementation detail is
in `micasaestuya-web/docs/`; `api`-specific in `micasaestuya-api/docs/`.

## Local ports

| Port  | Service     |
|-------|-------------|
| 3000  | Nuxt 3      |
| 3001  | Express API |
| 27017 | MongoDB     |
| 6379  | Redis       |

## Common commands

```bash
docker compose up --build       # start all services
docker compose down             # stop
docker compose down -v          # stop + delete volumes
docker compose exec express npm run lint  # api lint — only works in here
docker compose up -d --force-recreate nuxt  # fixes EADDRINUSE on worker.sock

docker exec express npm run redis:seed  # seed Redis — WIPES IT FIRST, see below
```

## ❌ Do not

- Commit `.env` (contains secrets)
- Modify `mongo-init.js` without testing locally first (runs only on first DB init)
- Run `redis:seed` to "check" anything. It starts with `flushdb`, wiping the
  whole database, and repopulating is 600k+ writes. To check Redis is alive:
  `curl 'http://localhost:3001/api/regions/suggest?term=hab&country_code=CU'`

## Note on node_modules

Dependencies live inside the container volumes, so `api/node_modules` is **empty
on the host**. Anything needing them — lint, tests — runs via
`docker compose exec`.
