# Deploy Cognato2 via Hostinger Docker Manager

Hostinger Docker Manager only supports `image:` references — it cannot build from source.  
This guide publishes the Paperclip image to GitHub Container Registry (GHCR), then deploys the full stack through the Docker Manager UI.

---

## Part 1 — Publish Paperclip Image to GHCR (run once, on your Mac)

### 1.1 Add write:packages scope to gh token

```bash
gh auth refresh -s write:packages
```

Follow the browser prompt to authorize.

### 1.2 Login to GHCR

```bash
gh auth token | docker login ghcr.io -u lucassauaia --password-stdin
```

### 1.3 Tag and push the already-built image

```bash
docker tag ai-workforce-paperclip:latest ghcr.io/lucassauaia/cognato2/paperclip:latest
docker push ghcr.io/lucassauaia/cognato2/paperclip:latest
```

### 1.4 Make the package public

1. Go to https://github.com/lucassauaia?tab=packages
2. Click **cognato2/paperclip**
3. Package Settings → Change visibility → **Public**

> Public visibility means the VPS can pull without credentials.

### 1.5 Update docker-compose.yml to use the image

Replace the `build:` block with the published image:

```bash
cd /Users/promo.warriors/Documents/cognato2

# Edit docker-compose.yml — change paperclip service from:
#   build:
#     context: ./services/paperclip
#     dockerfile: Dockerfile
# to:
#   image: ghcr.io/lucassauaia/cognato2/paperclip:latest
```

Then commit and push:

```bash
git add docker-compose.yml
git commit -m "fix: use pre-built GHCR image for Hostinger Docker Manager deploy"
git push origin master
```

### 1.6 Get the raw compose URL

```
https://raw.githubusercontent.com/lucassauaia/cognato2/master/docker-compose.yml
```

---

## Part 2 — Deploy via Hostinger Docker Manager

### 2.1 Open Docker Manager

1. Log in to **hPanel** → https://hpanel.hostinger.com
2. Select your VPS
3. Left sidebar → **Docker Manager**

### 2.2 Create new project

1. Click **+ New Project**
2. Enter project name: `cognato2`
3. Paste the raw compose URL:
   ```
   https://raw.githubusercontent.com/lucassauaia/cognato2/master/docker-compose.yml
   ```
   — or paste the YAML content directly

### 2.3 Set environment variables

Add each variable in the Docker Manager env vars section:

| Variable | Value |
|---|---|
| `BETTER_AUTH_SECRET` | output of `openssl rand -hex 32` |
| `OPENCLAW_GATEWAY_TOKEN` | output of `openssl rand -hex 32` |
| `ANTHROPIC_API_KEY` | `sk-ant-...` |
| `OPENAI_API_KEY` | (optional) |
| `PAPERCLIP_PUBLIC_URL` | `http://2.24.112.86:3100` |
| `POSTGRES_USER` | `paperclip` |
| `POSTGRES_PASSWORD` | strong password |
| `POSTGRES_DB` | `paperclip` |

Generate secrets locally:
```bash
openssl rand -hex 32   # run twice — one for each secret
```

### 2.4 Deploy

Click **Deploy** (or **Start**).  
Docker Manager pulls all images and starts the 4 containers.

### 2.5 Verify

```bash
# SSH into VPS:
ssh root@2.24.112.86

docker compose -p cognato2 ps
curl -s http://localhost:3100/api/health
```

Expected: `{"status":"ok",...}`

### 2.6 Bootstrap first admin

```bash
docker exec cognato2-paperclip-1 node /app/cli/dist/index.js auth bootstrap-ceo
```

Copy the invite URL from the output and open it in your browser.

---

## Part 3 — Add HTTPS with Caddy (optional, requires domain)

### 3.1 Point DNS to VPS

In your domain registrar → add A record:  
`your-domain.com → 2.24.112.86`

### 3.2 Set domain in env vars

Update in Docker Manager env vars:
```
CADDY_DOMAIN=your-domain.com
PAPERCLIP_PUBLIC_URL=https://your-domain.com
```

### 3.3 Enable Caddy profile

Docker Manager may not support `--profile`. If so, remove the `profiles: [production]` line from the Caddy service in the compose before deploying.

### 3.4 Redeploy

Click **Redeploy** in Docker Manager.  
Caddy auto-provisions TLS via Let's Encrypt within ~30 seconds.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `manifest unknown` | Image not pushed to GHCR or package is private |
| `BETTER_AUTH_SECRET must be set` | Env var missing in Docker Manager |
| Paperclip keeps restarting | Check logs: Docker Manager → logs tab |
| Port 3100 unreachable | Open firewall: `ufw allow 3100` on VPS |
| `build: context` error | Docker Manager doesn't support build — must use `image:` |
