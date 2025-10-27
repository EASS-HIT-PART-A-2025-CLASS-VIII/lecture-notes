# Session 04 – Docker Basics and Reverse Proxy Demo

- **Date:** Monday, Nov 24, 2025
- **Theme:** Package the FastAPI app into a production-friendly Docker image (multi-stage, non-root) and route traffic through `nginx` like production setups.

## Learning Objectives
- Explain containers vs. images and when to choose multi-stage builds.
- Write a Dockerfile that installs dependencies with `uv`, runs as a non-root user, and exposes a healthcheck.
- Create a `.dockerignore` to keep images lean and wire `nginx` as a reverse proxy.
- Use Docker networks and logs to verify both services share trace IDs from Sessions 02–03.

## Before Class – Container Preflight (JiTT)
- Run `docker --version` and `docker compose version`; reinstall or start Docker Desktop if commands fail.
- In `hello-uv`, create `.dockerignore` (if missing):
  ```ignore
  __pycache__/
  .pytest_cache/
  .venv/
  .uv/
  .env
  *.pyc
  docs/contracts/
  ```
- Pull the latest `nginx:alpine` image: `docker pull nginx:alpine`. Saves time in class.
- Skim GitHub’s article “Best practices for containers” (shared in LMS) and note one question about multi-stage builds or non-root users.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & intent | 7 min | Discussion | Share FastAPI progress + any Docker install blockers. |
| Docker mental model | 15 min | Talk + sketches | Images, layers, copy-on-write, registries, `docker history`. |
| Micro demo: image size diff | 3 min | Live demo (≤120 s) | Compare single-stage vs multi-stage sizes with `docker image ls`. |
| Reverse proxy concepts | 20 min | Talk + diagram | Why we use `nginx`, mapped ports, healthcheck endpoints, observing trace IDs. |
| **Part B – Lab 1** | **45 min** | **Guided build** | **Write multi-stage Dockerfile, build image, run healthcheck.** |
| Break | 10 min | — | Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes). |
| **Part C – Lab 2** | **45 min** | **Guided integration** | **Add `nginx` reverse proxy, inspect logs, prep for Compose.** |
| EX1 help clinic | 10 min | Q&A | Checklist review, backlog (healthcheck, coverage, CI cache). |

## Part A – Theory Highlights
1. **Whiteboard layers:** Base image → system deps (curl) → `uv` install → project files → runtime command. Stress deterministic builds (`uv sync --frozen`).
2. **Multi-stage rationale:** Build stage with compilers, runtime stage slim; smaller attack surface, faster pulls.
3. **Non-root containers:** Use `useradd` to create `app` user and run `uvicorn` without root. Mention security tie-in for Session 11.
4. **Healthcheck endpoint:** Map `/health` (built in Session 03) to Docker `HEALTHCHECK`. Explain CI/CD gating on it.
5. **Reverse proxy preview:** `nginx` handles TLS, compression, static caching; our app stays simple.

## Part B – Hands-on Lab 1 (45 Minutes)
### 1. Update FastAPI app for readiness probe (optional but recommended)
In `app/main.py`, ensure `/health` returns `{"status": "ok"}` quickly—no DB calls yet. This keeps Docker healthchecks fast.

### 2. Create `Dockerfile`
```dockerfile
# syntax=docker/dockerfile:1.7

FROM python:3.12-slim AS builder

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl build-essential \
    && curl -LsSf https://astral.sh/uv/install.sh | sh \
    && rm -rf /var/lib/apt/lists/*

ENV PATH="/root/.local/bin:${PATH}"
WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-install-project

COPY . .
RUN uv sync --no-dev --frozen

FROM python:3.12-slim AS runtime

ENV PYTHONUNBUFFERED=1 \
    UV_LINK_MODE=copy \
    PATH="/home/app/.local/bin:${PATH}"

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && curl -LsSf https://astral.sh/uv/install.sh | sh \
    && useradd --create-home app \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=builder /app /app
RUN chown -R app:app /app
USER app

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -fsS http://localhost:8000/health || exit 1

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```
Explain each block: install uv once per stage, copy only necessary layers, run as `app` user, healthcheck verifying `/health`.

### 3. Build and run
```bash
docker build -t movies-api:multi-stage .
docker run --rm -p 8000:8000 --name movies movies-api:multi-stage
```
Test in a second terminal:
```bash
curl -H "X-Trace-Id: docker-demo" http://localhost:8000/movies
```
Show the log line with the trace ID (thanks to Session 03 middleware). Stop the container with `Ctrl+C`.

### 4. Inspect layers & size
```bash
docker image ls movies-api:multi-stage
```
Compare against a quick single-stage build (for teaching only) and note the size delta.

## Part C – Hands-on Lab 2 (45 Minutes)
### 1. Prepare `ops/nginx.conf`
```nginx
server {
    listen 80;

    location / {
        proxy_pass http://movies-api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Trace-Id $http_x_trace_id;
        proxy_redirect off;
    }
}
```
Point out we forward `X-Trace-Id` so downstream logs retain correlation.

### 2. Create a Docker network and launch services
```bash
docker network create movies-net
docker run -d --name movies-api --network movies-net movies-api:multi-stage
docker run -d --name movies-proxy --network movies-net \
  -p 8080:80 \
  -v "$(pwd)/ops/nginx.conf:/etc/nginx/conf.d/default.conf:ro" \
  nginx:alpine
```
Verify the proxy:
```bash
curl -H "X-Trace-Id: proxy-demo" http://localhost:8080/health
```
Use `docker logs movies-api` to show the trace ID from the proxy, then `docker logs movies-proxy` to highlight request forwarding.

### 3. Cleanup
```bash
docker rm -f movies-api movies-proxy
docker network rm movies-net
```
Discuss how Docker Compose (Session 10) will codify the same setup.

### 4. Optional stretch – healthcheck inside Compose
Preview Compose file snippet to revisit later:
```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:8000/health"]
  proxy:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy
```

## EX1 Help Clinic & Backlog
- **Due Tue Dec 2, 23:59.**
- Minimum checklist: CRUD endpoints, tests, Docker image (`movies-api:multi-stage`), README instructions, AI usage notes.
- **Stretch/backlog ideas:** add `HEALTHCHECK` to Dockerfile (done), produce `docker compose` override, prebuild GitHub Actions job using cache (`uv sync --frozen`), and document `X-Trace-Id` behavior in README.

## Troubleshooting
- **Permission denied binding ports** → ensure Docker Desktop is running and port 8000/8080 free (`lsof -i :8000`).
- **Healthcheck failing** → confirm `/health` endpoint is reachable and returns 200 quickly.
- **Large image** → check `.dockerignore`, ensure `uv sync` runs with `--no-dev`, prune with `docker image prune` if disk is low.

## AI Prompt Seeds
- “Write a multi-stage Dockerfile that installs dependencies with uv and runs FastAPI as a non-root user.”
- “Draft an nginx config that forwards `X-Trace-Id` headers to an upstream FastAPI service.”
- “Explain how Docker healthchecks integrate with GitHub Actions deploy pipelines.”
