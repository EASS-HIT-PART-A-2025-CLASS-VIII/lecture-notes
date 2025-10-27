# Session 10 – Docker Compose, Redis, and Service Contracts

- **Date:** Monday, Jan 12, 2026
- **Theme:** Run the movie API with Redis and worker services via Docker Compose, enforce service contracts, and introduce rate limiting/background jobs.

## Learning Objectives
- Model multi-service environments with Docker Compose (API, Redis, worker, proxy).
- Use Redis for caching and rate limiting (`slowapi`) and discuss background jobs (Celery or Arq) for async tasks.
- Harden API contracts with Schemathesis (from Session 02 stretch) and document expectations in `docs/service-contract.md`.
- Wire coverage + contract tests into GitHub Actions for CI confidence.

## Before Class – Compose Preflight (JiTT)
- Install Redis locally or ensure Docker Desktop can pull `redis:7-alpine`:
  ```bash
  docker pull redis:7-alpine
  ```
- Review the provided Compose primer (LMS) and list one question about networking or environment variables.
- Run `uv run schemathesis run docs/contracts/openapi.json --checks status_code_conformance --dry-run` to confirm tooling works.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & async wins | 10 min | Discussion | Share Session 09 takeaways, identify bottlenecks. |
| Compose architecture | 18 min | Talk + diagram | api + redis + worker + proxy, networks, volumes. |
| Micro demo: redis cache hit | 5 min | Live demo | Use `redis-cli` to show cache set/get vs API latency. |
| Contract testing refresher | 12 min | Talk | Schemathesis, JSON schema examples, service obligations. |
| **Part B – Lab 1** | **45 min** | **Guided build** | **Docker Compose stack with FastAPI, Redis, background worker.** |
| Break | 10 min | — | Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes). |
| **Part C – Lab 2** | **45 min** | **Guided validation** | **Rate limiting, contract tests, GitHub Actions CI pipeline.** |
| Wrap-up | 10 min | Q&A | EX3 milestone reminder, deployment prep.

## Part A – Theory Highlights
1. **Compose structure:** `services` block, named volumes, networks, environment variables, healthchecks (`depends_on` with `service_healthy`).
2. **Redis roles:** caching (movie list), rate limiting tokens, job queue backend.
3. **Background jobs:** choose one lightweight runner (Arq or Celery). For class we’ll show Arq because it rides on asyncio.
4. **Contracts:** `docs/service-contract.md` documents request/response shapes; Schemathesis enforces them alongside integration tests.
5. **CI pipeline:** Compose for local dev, GitHub Actions for CI (use services job with `services.redis`, run `uv sync --frozen`, `pytest --cov`, `schemathesis`).

## Part B – Hands-on Lab 1 (45 Minutes)
### 1. Compose file (`compose.yaml`)
```yaml
services:
  api:
    build: .
    command: uv run uvicorn app.main:app --host 0.0.0.0 --port 8000
    ports:
      - "8000:8000"
    environment:
      - MOVIE_REDIS_URL=redis://redis:6379/0
      - MOVIE_RATE_LIMIT_PER_MINUTE=20
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 5s
      retries: 3

  worker:
    build: .
    command: uv run arq app.worker.WorkerSettings
    environment:
      - MOVIE_REDIS_URL=redis://redis:6379/0
    depends_on:
      redis:
        condition: service_healthy

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
```
Explain networks (default) and volumes (add later for persistence if needed).

### 2. Redis client helper (`app/cache.py`)
```python
from functools import lru_cache

import redis.asyncio as redis

from app.config import Settings


@lru_cache(maxsize=1)
def get_redis(settings: Settings) -> redis.Redis:
    return redis.from_url(settings.redis_url, decode_responses=True)
```
Extend `Settings` with `redis_url: str` and `rate_limit_per_minute: int`.

### 3. Rate limiting middleware (`app/rate_limit.py`)
```python
import time

from fastapi import HTTPException, Request

from app.cache import get_redis
from app.config import Settings


async def rate_limit(request: Request, call_next):
    settings = Settings()
    redis_client = get_redis(settings)
    key = f"rate:{request.client.host}:{request.url.path}"
    window = 60
    max_requests = settings.rate_limit_per_minute

    current = await redis_client.incr(key)
    if current == 1:
        await redis_client.expire(key, window)

    if current > max_requests:
        raise HTTPException(status_code=429, detail="Too many requests")

    response = await call_next(request)
    response.headers["X-RateLimit-Limit"] = str(max_requests)
    response.headers["X-RateLimit-Remaining"] = str(max_requests - current)
    return response
```
Mount middleware in `app/main.py` before routes.

### 4. Background worker (`app/worker.py`)
```python
from arq import cron
from arq.connections import RedisSettings

from app.config import Settings


class WorkerSettings:
    redis_settings = RedisSettings.from_url(Settings().redis_url)

    async def startup(self, ctx):
        ctx["settings"] = Settings()

    async def shutdown(self, ctx):
        ctx.pop("settings", None)

    async def refresh_recommendations(self, ctx, user_id: int) -> None:
        settings: Settings = ctx["settings"]
        # call async refresher from Session 09

    functions = [refresh_recommendations]
    cron_jobs = [cron(refresh_recommendations, cron_string="0 * * * *", kwargs={"user_id": 42})]
```
Explain Arq’s simplicity and how it leverages async functions. Mention Celery as alternative if teams want more features.

### 5. Caching `GET /movies`
Wrap the list route in `app/main.py`:
```python
import json

from fastapi import Depends

from app.cache import get_redis


@app.get("/movies", response_model=list[Movie])
async def list_movies(repository: RepositoryDep, settings: SettingsDep) -> list[Movie]:
    redis_client = get_redis(settings)
    cache_key = "movies:list"
    cached = await redis_client.get(cache_key)
    if cached:
        return json.loads(cached)

    result = [movie for movie in repository.list()]
    await redis_client.setex(cache_key, 60, json.dumps([movie.model_dump() for movie in result]))
    return result
```
Remember to invalidate cache in POST/PUT/DELETE handlers.

## Part C – Lab 2 (45 Minutes)
### 1. Contract tests with Schemathesis
```bash
uv run schemathesis run http://localhost:8000/openapi.json --checks status_code_conformance --workers 2
```
Address failures immediately; update docs if behavior changes. Store command in `Makefile` or `scripts/check_contract.sh`.

### 2. GitHub Actions pipeline
`.github/workflows/ci.yaml` (excerpt):
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v1
      - run: uv sync --frozen
      - run: uv run pytest --cov=app --cov-report=term-missing
      - run: uv run schemathesis run http://localhost:8000/openapi.json --checks status_code_conformance --dry-run
```
Explain how to swap the dry run for real execution once you can boot the API inside CI (Compose + `uvicorn` background run).

### 3. Rate limiting test (429)
```python
def test_rate_limit(client):
    for _ in range(Settings().rate_limit_per_minute):
        assert client.get("/movies").status_code == 200

    response = client.get("/movies")
    assert response.status_code == 429
    assert response.json()["detail"] == "Too many requests"
```
Use `freezegun` or monkeypatch time if the window needs reset.

### 4. Contract documentation
Update `docs/service-contract.md` with:
- Rate limit headers and expected values.
- Cache invalidation rules.
- Background job SLA (refresh runs every hour; immediate manual trigger via CLI).

## Wrap-up & Next Steps
- ✅ Compose stack, Redis cache + rate limiting, background worker, contract testing, CI pipeline outline.
- Prep for Session 11 (Security Foundations): audit secrets, rotate tokens, create `.env.example` entries for Redis auth if enabled.

## Troubleshooting
- **Redis connection refused** → confirm Compose network is up or local Redis running; check `redis_url` env.
- **Slow Schemathesis runs** → narrow to critical endpoints with `--endpoint` filter or run in CI nightly.
- **Arq import error** → install `uv add "arq==0.*"`; ensure worker service uses same image/tag as API.

## AI Prompt Seeds
- “Draft a docker-compose.yml with FastAPI, Redis, and an Arq worker including healthchecks.”
- “Write FastAPI middleware that rate limits requests using Redis tokens.”
- “Generate a GitHub Actions workflow that runs pytest with coverage and Schemathesis contract tests.”
