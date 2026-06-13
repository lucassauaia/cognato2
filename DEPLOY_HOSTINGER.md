# Deploy Cognato2 via Hostinger Docker Manager

Hostinger Docker Manager only supports `image:` references — it cannot build from source.  
This guide publishes the Paperclip image to GHCR, then deploys through the Docker Manager compose URL screen.

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

### 1.3 Tag and push the image

```bash
docker tag ai-workforce-paperclip:latest ghcr.io/lucassauaia/cognato/paperclip:latest
docker push ghcr.io/lucassauaia/cognato/paperclip:latest
```

### 1.4 Make the package public

1. Go to https://github.com/lucassauaia?tab=packages
2. Click **cognato/paperclip**
3. Package Settings → Change visibility → **Public**

> Public = VPS pulls without credentials.

### 1.5 Switch compose from build to image

In [docker-compose.yml](docker-compose.yml), replace the `paperclip` build block:

```yaml
# Remove this:
    build:
      context: ./services/paperclip
      dockerfile: Dockerfile

# Replace with:
    image: ghcr.io/lucassauaia/cognato/paperclip:latest
```

Commit and push:

```bash
git add docker-compose.yml
git commit -m "fix: use pre-built GHCR image for Hostinger Docker Manager deploy"
git push origin master
```

---

## Part 2 — Deploy via Hostinger Docker Manager

### 2.1 Open the Compose URL screen

Go to: **hPanel → VPS → Gerenciador Docker → Compose**  
Direct URL: `https://hpanel.hostinger.com/vps/1687333/docker-manager/compose-url`

### 2.2 Fill in the form

| Field | Value |
|---|---|
| **URL** | `https://github.com/lucassauaia/cognato/blob/master/docker-compose.yml` |
| **Nome do projeto** | `cognato` |

Click **Implantar**.

### 2.3 Set environment variables

After clicking Implantar, Docker Manager prompts for env vars. Add:

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

Generate secrets:
```bash
openssl rand -hex 32   # run twice — one per secret
```

### 2.4 Confirm deploy

Docker Manager pulls all images and starts the containers.

### 2.5 Verify

```bash
ssh root@2.24.112.86
docker compose -p cognato ps
curl -s http://localhost:3100/api/health
```

Expected: `{"status":"ok",...}`

### 2.6 Bootstrap first admin

```bash
docker exec cognato-paperclip-1 node /app/cli/dist/index.js auth bootstrap-ceo
```

Copy the invite URL from output and open in browser.

---

## Part 3 — Add HTTPS with Caddy (optional, requires domain)

### 3.1 Point DNS to VPS

In your domain registrar → add A record:  
`your-domain.com → 2.24.112.86`

### 3.2 Update env vars in Docker Manager

```
CADDY_DOMAIN=your-domain.com
PAPERCLIP_PUBLIC_URL=https://your-domain.com
```

### 3.3 Remove Caddy profile restriction

Docker Manager doesn't support `--profile`. Remove `profiles: [production]` from the Caddy service in docker-compose.yml before redeploying.

### 3.4 Redeploy

Click **Reimplantar** in Docker Manager.  
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
| URL field rejects link | Use the `/blob/master/` GitHub URL, not raw.githubusercontent |
