# Session 10 – Docker Compose and Service Contracts

- **Date:** Monday, Jan 5, 2026
- **Theme:** Run multiple services together with Docker Compose and document how they interact.

## Learning Objectives
- Describe why multi-service architectures use reverse proxies and shared caches.
- Write a Docker Compose file that starts the API, a Redis cache, and an nginx front door.
- Document service contracts (endpoints, ports, health checks, cache responsibilities) clearly.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Holiday recap | 10 min | Conversation | Quick check-in and EX3 excitement |
| Microservice rationale | 20 min | Talk + drawing | When to split services, pros/cons |
| Service contracts | 15 min | Talk | HTTP contracts, cache responsibilities, health checks |
| AWS module check | 5 min | Announcement | Confirm AWS Academy certificates were submitted by Dec 16 and note any make-up steps |
| Lab 1 | 45 min | Guided coding | Compose file + Redis cache + nginx proxy |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Structured logging/metrics + documentation |
| EX3 assignment briefing | 10 min | Talk | Requirements, milestones, grading |

## Teaching Script – Why Compose?
1. Illustrate monolith vs. split services on the board. Emphasize: “Compose lets us run all services with one command.”
2. Explain the role of a reverse proxy: “nginx sits at the edge, handles TLS (later), and routes traffic to FastAPI.”
3. Introduce the concept of a service contract: endpoint list, auth requirements, success/error schemas.
4. Confirm AWS Academy **Databases** completion: “All AWS modules were due **Tuesday, Dec 16, 2025**. If you still owe a submission, upload it tonight and ping me on Discord.”

## Part B – Hands-on Lab 1 (45 Minutes)
### Directory Prep
```bash
mkdir -p ops
```

### Install Redis Client
```bash
uv add redis
```

### Update `app/main.py`
Add Redis caching for recommendations:
```python
import json
import os
import redis

REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379/0")
redis_client = redis.Redis.from_url(REDIS_URL, decode_responses=True)


@app.get("/recommendations/{user_id}")
def get_recommendations(user_id: int, limit: int = 5) -> list[int]:
    cached = redis_client.get("recommendations")
    if cached:
        model = json.loads(cached)
        return model.get(str(user_id), [])[:limit]
    ratings = repository.list_ratings()
    results = recommender.recommend_for_user(ratings, user_id, k=limit)
    return results
```

Inside the background task from Session 09, persist the rebuilt model:
```python
model = recommender.build_model(ratings)
redis_client.set("recommendations", json.dumps(model))
```

### nginx Configuration (`ops/nginx.conf`)
```nginx
server {
    listen 80;

    location / {
        proxy_pass http://api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Docker Compose (`docker-compose.yml`)
```yaml
version: "3.9"

services:
  api:
    build: .
    command: ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
    environment:
      REDIS_URL: redis://redis:6379/0
    ports:
      - "8000:8000"
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 10s
      timeout: 2s
      retries: 5
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 2s
      retries: 10
  nginx:
    image: nginx:alpine
    volumes:
      - ./ops/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy
```

### Run Compose
```bash
docker compose up --build
```
Test through the proxy and cache:
```bash
curl http://localhost:8080/health
curl http://localhost:8080/recommendations/7?limit=3
```

### Clean Up
`Ctrl+C` to stop, then `docker compose down`
## Part C – Hands-on Lab 2 (45 Minutes)
### Add Structured Logs
Update `app/main.py` logging middleware to output JSON-style lines:
```python
import json


@app.middleware("http")
async def log_requests(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration_ms = (time.perf_counter() - start) * 1000
    log_message = json.dumps(
        {
            "method": request.method,
            "path": request.url.path,
            "status": response.status_code,
            "duration_ms": round(duration_ms, 2),
        }
    )
    logger.info(log_message)
    return response
```

### Document Service Contract
Create `docs/service-contract.md` and list:
- External URL: `http://localhost:8080`
- Public endpoints: `/health`, `/movies`, `/movies/top`, `/recommendations/{user_id}`, `/recommender/rebuild` (async).
- Expected request/response shapes (copy from FastAPI models)
- Authentication: none yet (Session 11 adds JWT for protected writes). Include Redis as the cache provider for recommendation data.

### Optional Metrics Endpoint
Add a simple `/metrics` route returning how many requests were processed (store a counter in `app.state`).

## Exercise 3 Briefing
- **Assigned:** Today (Mon Jan 5).
- **Milestone demo:** Tue Jan 20 – show stack running with Compose and advanced feature in progress.
- **Final due:** Tue Feb 10 – full stack, documentation, tests.
- **Advanced feature options:**
  1. Async background job with status dashboard (session 09).
  2. JWT-based auth (session 11 preview).
  3. Observability: metrics + log aggregation.
- Rubric highlights: architecture (25), advanced feature (30), reliability (15), tests (15), documentation (15).

## Troubleshooting
- If Compose cannot find the Dockerfile, verify `docker-compose.yml` is in the project root.
- For `nginx` permission denials on Windows, ensure file sharing is enabled for the drive.
- Health check failing? Make sure `/health` returns 200 and the container exposes port 8000.

## Student Success Criteria
- `docker compose up` starts `api`, `redis`, and `nginx` services.
- API reachable via `http://localhost:8080` with proxied traffic logged and cached recommendations served via Redis.
- Service contract document captured endpoints, expected JSON payloads, and cache responsibilities (what lives in Redis vs. SQLite).
- Students have confirmed their AWS Databases certificate was submitted by **Tue Dec 16, 2025** (or have a catch-up plan).

## Optional: Quick CI with GitHub Actions
Create `.github/workflows/ci.yml` in your repo:
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install uv
        run: pip install uv
      - name: Sync deps
        run: uv sync --dev
      - name: Run tests
        run: uv run pytest -q
```

## AI Prompt Kit (Copy/Paste)
- “Write a `docker-compose.yml` with services `api`, `redis`, and `nginx` where the API consumes `REDIS_URL` and waits for Redis to be healthy.”
- “Draft an `nginx.conf` that proxies `/` to `http://api:8000` with standard headers and passes through `/health` and `/recommendations`.”
- “Create a minimal GitHub Actions workflow that installs uv, syncs dependencies, and runs pytest on every push.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `docker compose fastapi redis nginx example`
- **ChatGPT prompt:** “List three talking points to explain why we pair FastAPI + Redis + nginx in a Compose demo.”
