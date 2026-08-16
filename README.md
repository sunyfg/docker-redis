# Redis Docker Compose (Local Dev)

A minimal, production-conscious Redis 7 environment for local development on macOS
(Intel and Apple Silicon). Password auth, AOF persistence, healthcheck, and a
Docker named volume — all configured via `.env`.

## Environment Requirements

- Docker Desktop for Mac (Docker Engine + Compose v2)
- `redis-cli` (optional, for host-side connections; `docker compose exec` works without it)

## Quick Start

```bash
cp .env.example .env   # first time only, then edit REDIS_PASSWORD
docker compose up -d
```

## Start Redis

```bash
docker compose up -d
```

## Check Status

```bash
docker compose ps
```

Redis should report `healthy` (healthcheck runs `redis-cli -a "$REDIS_PASSWORD" ping`).

## View Logs

```bash
docker compose logs -f redis
```

## Connect to Redis

Read the actual password from your local `.env`, then connect inside the container:

```bash
docker compose exec redis sh
```

```bash
redis-cli -a "$REDIS_PASSWORD"
```

From the host (requires `redis-cli` installed locally; never paste the real
password into the README — check your `.env`):

```bash
redis-cli -h 127.0.0.1 -p 6379 -a '<REDIS_PASSWORD>'
```

## Test Connection

```redis
PING
```

Expected:

```text
PONG
```

## Basic Read / Write

```redis
SET demo:message "hello redis"
GET demo:message
```

## Redis Info

```redis
INFO
```

## Persistence Configuration

```redis
CONFIG GET appendonly
CONFIG GET appendfsync
```

Expected: `appendonly yes`, `appendfsync everysec` (AOF enabled).

## Stop Redis

```bash
docker compose down
```

## Stop and Delete Redis Data

> **WARNING:** the following command deletes ALL local Redis data (removes the `redis_data` volume):

```bash
docker compose down -v
```

Start again:

```bash
docker compose up -d
```

You get a brand-new Redis instance.

## Application Connection Parameters

| Parameter  | Value                    |
|------------|--------------------------|
| host       | `127.0.0.1`              |
| port       | `6379` (see `.env`)      |
| password   | see local `.env`         |
| database   | `0`                      |

Standard Redis URL (fill in the password from `.env`):

```text
redis://:<PASSWORD>@127.0.0.1:6379/0
```

## Project Layout

```text
.
├── compose.yaml     # Redis service: auth, AOF, healthcheck, named volume
├── .env             # local secrets (git-ignored)
├── .env.example     # template for .env
├── .gitignore
└── README.md
```

## Notes

- Persistence: AOF (`appendonly yes`, `appendfsync everysec`) written to the
  `redis_data` named volume mounted at `/data`. `docker compose down` keeps the
  volume; `docker compose down -v` deletes it.
- Password is injected via `REDIS_PASSWORD` env var into the container and
  referenced inside the command with `$$REDIS_PASSWORD` — never hardcoded in
  `compose.yaml`.
- No `platform` pin: the official `redis:latest` image runs natively on both
  Intel and Apple Silicon Macs.