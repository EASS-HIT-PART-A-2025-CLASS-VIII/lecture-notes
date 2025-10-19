# Session 09 – Async Jobs and Recommendation Refresh

- **Date:** Monday, Dec 29, 2025
- **Theme:** Keep the movie API responsive by offloading recommendation rebuilds to background tasks and making clients resilient.

## Learning Objectives
- Distinguish between blocking and non-blocking operations in web services.
- Schedule background jobs using FastAPI’s `BackgroundTasks` to regenerate recommendations.
- Implement client-side retry logic with backoff and understand idempotency.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm-up chat | 10 min | Discussion | Share holiday wins and one reliability failure you have seen |
| Async fundamentals | 20 min | Talk + diagrams | Blocking vs. non-blocking, event loop basics |
| Reliability vocabulary | 15 min | Talk | Retries, backoff, idempotency, graceful shutdown |
| Lab 1 | 45 min | Guided coding | Background task to rebuild movie recommendations |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Add retries, Redis-ready hooks, and graceful shutdown |
| EX3 preview | 10 min | Talk | Describe advanced feature options |

## Teaching Script – Async Overview
1. Draw two timelines: blocking recomputation vs. async rebuild.
2. Explain why recommendation models (ALS) can run in the background after ratings arrive.
3. Define key terms: **idempotency**, **backoff**, **graceful shutdown**.
4. Set context for Exercise 3: one advanced feature pathway is “Async recommendation refresh”. Today’s lab seeds that work.

## Part B – Hands-on Lab 1 (45 Minutes) – Background Recommendation Task
### Dependencies
Install numerical helpers if you have not already (Prompt 2 from Session 08):
```bash
uv add numpy
```

### Update `app/recommender.py`
> **Note:** Session 05’s repository now includes `list_ratings()` to expose raw rating rows for the recommender.
Ensure the module exposes a builder that returns a mapping from user id to ranked movie ids:
```python
from collections import defaultdict
from typing import Dict, Iterable, List

import numpy as np

# In-memory cache built by Session 09 background job
MODEL_CACHE: Dict[int, List[int]] = {}


def build_model(ratings: Iterable[dict]) -> Dict[int, List[int]]:
    """Very small ALS-style approximation using NumPy outer products."""
    by_user: dict[int, dict[int, int]] = defaultdict(dict)
    for row in ratings:
        by_user[int(row["user_id"])][int(row["movie_id")] ] = int(row["score"])

    if not by_user:
        MODEL_CACHE.clear()
        return MODEL_CACHE

    # Build rating matrix with simple normalization
    user_ids = sorted(by_user.keys())
    movie_ids = sorted({movie_id for ratings_for_user in by_user.values() for movie_id in ratings_for_user})
    matrix = np.zeros((len(user_ids), len(movie_ids)))
    for ui, user_id in enumerate(user_ids):
        for mj, movie_id in enumerate(movie_ids):
            matrix[ui, mj] = by_user[user_id].get(movie_id, 0)

    # Compute user and item factors via naive SVD
    u, s, vt = np.linalg.svd(matrix, full_matrices=False)
    scores = np.matmul(np.matmul(u * s, vt), np.ones((len(movie_ids), 1))).reshape(len(user_ids), len(movie_ids))

    MODEL_CACHE.clear()
    for idx, user_id in enumerate(user_ids):
        ranked = [movie_ids[i] for i in np.argsort(scores[idx])[::-1]]
        MODEL_CACHE[user_id] = ranked
    return MODEL_CACHE


def recommend_for_user(ratings: Iterable[dict], user_id: int, k: int = 5) -> list[int]:
    if MODEL_CACHE:
        return MODEL_CACHE.get(user_id, [])[:k]
    # Fallback: build on the fly
    build_model(ratings)
    return MODEL_CACHE.get(user_id, [])[:k]
```

### Wire the Background Task (`app/main.py`)
```python
import time
import uuid
from fastapi import BackgroundTasks

from app import recommender, repository

TASK_STATUS: dict[str, str] = {}


def run_rebuild_task(task_id: str) -> None:
    TASK_STATUS[task_id] = "running"
    time.sleep(0.5)  # simulate heavier work / allow rating writes to finish
    ratings = repository.list_ratings()
    recommender.build_model(ratings)
    TASK_STATUS[task_id] = "complete"


@app.post("/recommender/rebuild", status_code=202)
async def rebuild_recommendations(background: BackgroundTasks) -> dict[str, str]:
    task_id = str(uuid.uuid4())
    TASK_STATUS[task_id] = "queued"
    background.add_task(run_rebuild_task, task_id)
    return {"task_id": task_id, "status": "queued"}


@app.get("/recommender/rebuild/{task_id}")
async def rebuild_status(task_id: str) -> dict[str, str | list[int]]:
    status = TASK_STATUS.get(task_id, "unknown")
    sample = recommender.MODEL_CACHE.get(1, [])[:3] if status == "complete" else []
    return {"task_id": task_id, "status": status, "sample": sample}
```

### Demo Flow
1. Start the server:
   ```bash
   uv run uvicorn app.main:app --reload
   ```
2. Record a few ratings (from Session 06 UI or curl). Example:
   ```bash
   curl -X POST http://localhost:8000/ratings      -H "Content-Type: application/json"      -d '{"movie_id": 1, "user_id": 7, "score": 5}'
   ```
3. Kick off a rebuild:
   ```bash
   curl -X POST http://localhost:8000/recommender/rebuild
   ```
4. Poll status until complete:
   ```bash
   curl http://localhost:8000/recommender/rebuild/<task_id>
   ```
5. Fetch recommendations:
   ```bash
   curl http://localhost:8000/recommendations/7?limit=3
   ```

## Part C – Hands-on Lab 2 (45 Minutes) – Reliability Patterns
### Client-Side Retry Helper (`app/retry_client.py`)
```python
import random
import time
from typing import Callable

import httpx


def with_backoff(
    func: Callable[[], httpx.Response],
    *,
    retries: int = 5,
    backoff_factor: float = 0.5,
    jitter: float = 0.2,
) -> httpx.Response:
    for attempt in range(retries):
        try:
            response = func()
            response.raise_for_status()
            return response
        except httpx.HTTPError as exc:
            sleep_for = backoff_factor * (2 ** attempt) + random.uniform(0, jitter)
            print(f"Attempt {attempt + 1} failed ({exc}). Sleeping {sleep_for:.2f}s")
            time.sleep(sleep_for)
    return func()
```

Use the helper to poll `/recommender/rebuild/{task_id}` until the job completes.

### Graceful Shutdown
```python
@app.on_event("shutdown")
def on_shutdown() -> None:
    if any(status == "running" for status in TASK_STATUS.values()):
        print("Warning: recommendation rebuild still running; consider persisting interim state")
```
Discuss how a production system might persist the task state (Redis/DB) before shutdown.

### Redis Preview (Optional)
If time allows, show how to cache `recommender.MODEL_CACHE` in Redis:
```bash
uv add redis
```
```python
import redis

redis_client = redis.Redis(host="localhost", port=6379, decode_responses=True)

# Inside run_rebuild_task
def run_rebuild_task(task_id: str) -> None:
    TASK_STATUS[task_id] = "running"
    ratings = repository.list_ratings()
    model = recommender.build_model(ratings)
    redis_client.set("recommendations", json.dumps(model))
    TASK_STATUS[task_id] = "complete"
```
Docker Compose wiring happens in Session 10.

## Exercise 3 Preview
- **Assigned:** Monday, Jan 5, 2026.
- **Milestone demo:** Tuesday, Jan 20, 2026.
- **Final due:** Tuesday, Feb 10, 2026.
- Advanced feature options: async recommendation job (today), JWT auth (Session 11), observability (Session 10).

## Troubleshooting
- If the task never runs, verify `run_rebuild_task` is synchronous and you are not `await`-ing it inside the endpoint.
- When status stays `queued`, ensure `background.add_task(run_rebuild_task, task_id)` is reached (no earlier exception).
- If retries hammer the endpoint, lower `retries` or increase `backoff_factor`.

## Student Success Criteria
- POST `/recommender/rebuild` returns immediately with a task id.
- Polling `/recommender/rebuild/{task_id}` moves from `queued` → `running` → `complete`.
- `/recommendations/{user_id}` returns movie ids influenced by the rebuild.

## Quick Test to Verify the Background Task
Create `tests/test_recommender_task.py`:
```python
import time
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_rebuild_flow():
    # seed a custom rating
    movie = client.post(
        "/movies",
        json={"title": "Async Demo", "year": 2024, "genre": "Sci-Fi"},
    ).json()
    client.post(
        "/ratings",
        json={"movie_id": movie["id"], "user_id": 88, "score": 5},
    )
    task_id = client.post("/recommender/rebuild").json()["task_id"]

    deadline = time.time() + 5
    while time.time() < deadline:
        status = client.get(f"/recommender/rebuild/{task_id}").json()["status"]
        if status == "complete":
            break
        time.sleep(0.2)

    assert client.get(f"/recommender/rebuild/{task_id}").json()["status"] == "complete"
    recs = client.get("/recommendations/88?limit=1").json()
    assert movie["id"] in recs
```
Run:
```bash
uv run pytest -q
```

## AI Prompt Kit (Copy/Paste)
- “Add a `/recommender/rebuild` endpoint that enqueues a background task, stores status, and updates an in-memory recommendation cache.”
- “Write a Python helper that polls an endpoint with exponential backoff and jitter, logging each attempt.”
- “Explain how to make `/recommender/rebuild` idempotent using a `X-Task-Id` header or similar token.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI BackgroundTasks recommendation system`
- **ChatGPT prompt:** “Describe exponential backoff with jitter for students building a movie recommendation API.”
