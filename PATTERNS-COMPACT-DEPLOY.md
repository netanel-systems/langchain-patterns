# Deployment Patterns — Compact Reference
*8 patterns for deploying AI agents to production. When/where to use each one.*
*Full reference: cloud-deploy.md skill | Last updated: 2026-02-22*

---

## DP1 — Dockerfile

**What:** Text file that defines how to build a container image for your agent.

**When:** Every time you containerize a service — Dockerfile is the entry point for all Docker/Railway deploys.

**Where:** Project root (`./Dockerfile`).

**Key rule:** Copy `requirements.txt` before source code so Docker's layer cache isn't busted on every code edit. Use exec form for `CMD`, not shell form — required for graceful shutdown and LangGraph lifespan events.

```dockerfile
FROM python:3.12-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade -r requirements.txt
COPY ./app ./app
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser
USER appuser
ENV PORT=8000
CMD ["sh", "-c", "fastapi run app/main.py --host 0.0.0.0 --port $PORT"]
```

---

## DP2 — .dockerignore

**What:** File that tells Docker which files NOT to include in the build context.

**When:** Always. Missing this makes builds slow and can leak secrets into the image.

**Where:** Project root (`./.dockerignore`), same directory as `Dockerfile`.

**Key rule:** Always exclude `.env*`, `.git`, `__pycache__`, `tests/`, and editor dirs. Never let secrets reach the image.

```
.git
.env
.env.*
__pycache__/
*.pyc
.venv/
venv/
.pytest_cache/
tests/
docs/
.vscode/
.idea/
Dockerfile
.dockerignore
```

---

## DP3 — Local Docker Run

**What:** Build and run the container on your machine before pushing to Railway.

**When:** Before every deploy — validate the image works locally, env vars are wired up, and the app starts on the expected port.

**Where:** Local machine, after `docker build`.

**Key rule:** Mirror Railway's runtime: pass `-e PORT=8000` so you catch PORT-related bugs before they hit production.

```bash
# Build
docker build -t my-agent:latest .

# Run (mirrors Railway runtime)
docker run --rm \
  -p 8000:8000 \
  --env-file .env \
  -e PORT=8000 \
  my-agent:latest

# Verify health endpoint
curl http://localhost:8000/health
```

---

## DP4 — Railway Connect GitHub

**What:** Link a GitHub repository to Railway so every push triggers an automatic deploy.

**When:** For production services. Push-to-deploy removes the manual `railway up` step and creates a proper CI/CD pipeline.

**Where:** Railway dashboard → New Project → Deploy from GitHub repo.

**Key rule:** Railway detects `Dockerfile` automatically. Add `railway.toml` to control start command, health check, and restart policy as code.

```toml
# railway.toml — commit this to the repo
[deploy]
startCommand = "fastapi run app/main.py --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 300
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

Steps:
1. Dashboard → New Project → Deploy from GitHub repo
2. Search and select repository → Deploy Now
3. Commit `railway.toml` to control deploy config as code

---

## DP5 — Environment Variables on Railway

**What:** Secrets and config values injected into your container at runtime.

**When:** Any value that differs between environments (dev/staging/prod) or must never be committed to Git.

**Where:** Railway dashboard → Service → Variables tab. Or reference values from other services using `${{ }}` syntax.

**Key rule:** Never put secrets in `Dockerfile` or commit `.env` to Git. Railway injects vars at runtime — your app reads them with `os.environ` or Pydantic Settings.

```
# Dashboard → Variables → RAW Editor — paste your .env contents here
OPENAI_API_KEY=sk-...
LANGSMITH_API_KEY=ls-...
ENVIRONMENT=production

# Reference another service's variable (e.g., Postgres addon)
DATABASE_URL=${{ Postgres.DATABASE_URL }}

# Reference a shared project variable
API_KEY=${{ shared.API_KEY }}
```

Local development with Railway's vars loaded:
```bash
railway run python src/agent.py
```

---

## DP6 — Health Check Endpoint

**What:** An HTTP endpoint your app exposes that returns 200 when ready to serve traffic.

**When:** Any Railway service. Required for zero-downtime deploys — Railway won't route traffic to a new deployment until the health check passes.

**Where:** App code + `railway.toml` `healthcheckPath` field.

**Key rule:** App must listen on the `PORT` variable Railway injects — health checks hit that same port. Default timeout is 300 seconds.

```python
# app/main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/health")
async def health():
    return {"status": "ok"}
```

```toml
# railway.toml
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 300
```

Allow `healthcheck.railway.app` if you restrict incoming traffic by hostname.

---

## DP7 — Logs + Monitoring

**What:** Streaming build and runtime logs from Railway deployments.

**When:** After every deploy (verify it started clean), when debugging failures, and during incident response.

**Where:** CLI (`railway logs`) or Railway dashboard → Deployments → Logs panel.

**Key rule:** Set `PYTHONUNBUFFERED=1` in your Dockerfile — without it, Python buffers stdout and logs appear delayed or not at all in Railway.

```bash
# Stream live deployment logs
railway logs

# Show build logs (useful when deploy fails before starting)
railway logs --build

# JSON output for log pipelines / LangSmith ingestion
railway logs --json

# Check current project + deployment status
railway status
```

For LangGraph agents, add LangSmith tracing:
```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"  # set via Railway vars, not here
```

---

## DP8 — Zero-Downtime Deploy

**What:** A deployment strategy where the new version is validated before the old one is terminated.

**When:** Any production service where downtime is unacceptable. Requires a health check endpoint (DP6).

**Where:** Automatic when `healthcheckPath` is set in `railway.toml`. Railway handles the traffic switchover.

**Key rule:** The health check is the gate. New deployment only goes live after returning HTTP 200 on `healthcheckPath`. If it times out, the deploy fails and the old version stays live.

```toml
# railway.toml — zero-downtime config
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 300        # seconds before deploy is marked failed
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

Rollback if deploy fails:
```bash
# Redeploy the previous successful build from CLI
railway redeploy

# Or from dashboard: Deployments → select earlier build → Redeploy
```

Note: Services with attached volumes (persistent storage) experience brief downtime even with health checks configured — this is a Railway platform limitation.

---

## Quick Decision Table

| Situation | Use | Pattern |
|-----------|-----|---------|
| Containerizing a Python agent | Write Dockerfile | DP1 |
| Builds are slow, image is large | Add .dockerignore | DP2 |
| Verifying before pushing | Local docker run | DP3 |
| Setting up CI/CD pipeline | Connect GitHub repo | DP4 |
| Storing API keys securely | Railway Variables tab | DP5 |
| Need zero-downtime deploys | Add /health endpoint + railway.toml | DP6 |
| Debugging a failed deploy | railway logs --build | DP7 |
| Production service — no downtime | healthcheckPath in railway.toml | DP8 |
| Local dev with prod vars | railway run python agent.py | DP5 |
