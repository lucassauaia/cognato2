# AI Workforce Orchestration — Task Tracker & Lessons

> Per CLAUDE.md Workflow Orchestration: all work tracked here with checkable items.

---

## Phase 0: Planning & Research
- [x] Read and internalize `CLAUDE.md` core principles
- [x] Research Paperclip repo: architecture, Dockerfile, docker-compose, .env.example
- [x] Research OpenClaw: gateway architecture, Docker deployment, ports, volumes
- [x] Research Hermes Agent: memory architecture, Docker deployment, persistence
- [x] Understand inter-service integration (adapter system, hermes-paperclip-adapter)
- [x] Draft implementation plan with architecture diagram
- [x] Write `tasks/lesson.md` (this file)
- [x] Present plan summary and request user approval

---

## Phase 1: Project Scaffold
- [ ] Create `.gitignore` (ignore `.env`, `node_modules/`, Docker artifacts)
- [ ] Create `.env.example` with all documented variables
- [ ] Create `services/` directory structure

---

## Phase 2: Service Setup — Paperclip (Management Layer)
- [ ] Clone `paperclipai/paperclip` into `services/paperclip/`
- [ ] Verify the existing `Dockerfile` builds cleanly
- [ ] Configure `DATABASE_URL` to point to the compose `db` service
- [ ] Configure `BETTER_AUTH_SECRET` and `PAPERCLIP_PUBLIC_URL`

---

## Phase 3: Service Setup — OpenClaw (Execution Layer)
- [ ] Determine official Docker image or build strategy
- [ ] Create Dockerfile if no official image exists
- [ ] Configure gateway port (`18789`) and data volume
- [ ] Test WebSocket connectivity

---

## Phase 4: Service Setup — Hermes (Memory Layer)
- [ ] Determine official Docker image or build strategy
- [ ] Create Dockerfile if no official image exists
- [ ] Map persistent volume for `~/.hermes` (MEMORY.md, USER.md, SQLite FTS5)
- [ ] Configure API keys pass-through

---

## Phase 5: Docker Compose Assembly
- [ ] Write `docker-compose.yml` with all 4 services
- [ ] Define `ai-workforce` bridge network
- [ ] Define named volumes: `pgdata`, `paperclip_home`, `openclaw_data`, `hermes_data`
- [ ] Add health checks to all services
- [ ] Set `restart: unless-stopped` on all services
- [ ] Set `depends_on` with health conditions

---

## Phase 6: Verification
- [ ] `docker compose config` — validates without errors
- [ ] `docker compose up -d` — all services reach healthy state
- [ ] Health check probes pass on all endpoints
- [ ] Inter-service DNS resolution works (`ping` between containers)
- [ ] Volume persistence verified across `down`/`up` cycles
- [ ] Fault isolation test: stop one service, others continue

---

## Phase 7: Documentation
- [ ] Write `README.md` with quickstart instructions
- [ ] Document architecture diagram
- [ ] Document `.env` configuration guide
- [ ] Add troubleshooting section

---

## Review Section
_To be filled after implementation is complete._

---

## Lessons Learned
_Updated after any corrections from the user._

| Date | Lesson | Rule |
|---|---|---|
| — | — | — |
