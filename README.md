# Cognato2 — AI Workforce Stack

Local AI workforce: three agent services + PostgreSQL on a shared Docker bridge network.

| Service | Role | Port |
|---|---|---|
| **Paperclip** | Management / Control Plane | 3100 |
| **OpenClaw** | Execution / Gateway | 18789 |
| **Hermes** | Persistent Memory / Learning | 8642 |
| **PostgreSQL** | Paperclip database | 5432 |

## Architecture

```
ai-workforce network (bridge)
┌────────────┐   DATABASE_URL   ┌──────────────┐
│  paperclip │ ──────────────── │     db       │
│  :3100     │                  │  postgres:17  │
└────────────┘                  └──────────────┘
      │ adapter API  ╮
      │              ├── ┌──────────────┐
      │              │   │   openclaw   │
      │              │   │   :18789     │
      │              │   └──────────────┘
      │              │
      ╰──────────────┴── ┌──────────────┐
                         │    hermes    │
                         │   :8642      │
                         └──────────────┘
```

## Prerequisites

- Docker + Docker Compose v2
- `git`

## Quickstart

```bash
# 1. Clone repo
git clone https://github.com/lucassauaia/cognato.git
cd cognato

# 2. Clone Paperclip source
git clone https://github.com/paperclipai/paperclip.git services/paperclip

# 3. Configure environment
cp .env.example .env
# Edit .env — fill in ANTHROPIC_API_KEY, OPENAI_API_KEY, and BETTER_AUTH_SECRET
# Generate secret: openssl rand -hex 32

# 4. Start stack
docker compose up -d

# 5. Open Paperclip
open http://localhost:3100
```

## Environment Variables

Copy `.env.example` to `.env` and set:

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes | Anthropic API key |
| `OPENAI_API_KEY` | Optional | OpenAI API key |
| `BETTER_AUTH_SECRET` | Yes | Run `openssl rand -hex 32` |
| `PAPERCLIP_PUBLIC_URL` | Yes | Public URL for Paperclip (default: `http://localhost:3100`) |
| `DATABASE_URL` | Auto | Set automatically via defaults |

## Verification

```bash
# Validate compose file
docker compose config

# Check all services healthy
docker compose ps

# Health probes
curl -f http://localhost:3100/api/health   # Paperclip
curl -f http://localhost:18789/healthz     # OpenClaw
curl -f http://localhost:8642/health       # Hermes

# Inter-service DNS
docker compose exec paperclip ping -c 1 openclaw
docker compose exec paperclip ping -c 1 hermes
docker compose exec paperclip ping -c 1 db

# Fault isolation: stop one, others keep running
docker compose stop hermes
curl -f http://localhost:3100/api/health   # still responds
docker compose start hermes
```

## Volumes

All data persists in named Docker volumes — survives `docker compose down`.
Destroyed only with `docker compose down -v`.

| Volume | Service | Contents |
|---|---|---|
| `pgdata` | db | PostgreSQL data |
| `paperclip_home` | paperclip | App data |
| `openclaw_data` | openclaw | Gateway state |
| `hermes_data` | hermes | MEMORY.md, USER.md, SQLite FTS5 |

## Troubleshooting

**Paperclip won't start** — ensure `db` is healthy first: `docker compose logs db`

**OpenClaw permission error** — Docker socket mount requires Docker daemon running on host.

**Hermes not starting** — check `docker compose logs hermes`. Needs at least one API key set.

**Port conflicts** — override in `.env`: `PAPERCLIP_PORT=3101`, `OPENCLAW_PORT=18790`, etc.
