# Session 07 – Testing, Logging, and Profiling Basics

- **Date:** Monday, Dec 15, 2025
- **Theme:** Improve reliability by expanding tests, adding meaningful logs, and measuring performance.

## Learning Objectives
- Write tests that cover error paths and edge cases.
- Add structured logging to FastAPI requests.
- Measure response time with lightweight profiling tools.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm-up discussion | 10 min | Circle | Share one bug caught by tests over the weekend |
| Testing review | 20 min | Talk | Unit vs. integration tests, parametrization, fixtures |
| Logging primer | 15 min | Talk | Log levels, structured messages, why metadata matters |
| Lab 1 | 45 min | Guided coding | Expand pytest coverage using parametrization |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Add request logging middleware and timing |
| EX2 work sprint | 10 min | Lab | Dedicated help time; remind deadline |

## Teaching Script – Quality Mindset
1. “Shipping code without tests is like flying without instruments.” Encourage small, fast tests.
2. Introduce pytest parametrization to avoid duplicating code.
3. Explain log levels: DEBUG (noisy) → INFO (default) → WARNING → ERROR → CRITICAL.
4. Mention profiling: “You do not need fancy tools today. Timing functions with `time.perf_counter` is enough to spot slowdowns.”
5. Remind everyone to finish the AWS **Storage** module during the break so it’s uploaded by **Tuesday, Dec 9, 2025**, comfortably ahead of the **Tue Dec 16, 2025** hard deadline.

## Part B – Hands-on Lab 1 (45 Minutes) – Test Expansion for Movies
### Temporary Database Fixture (`tests/conftest.py`)
```python
import os
import tempfile
from collections.abc import Iterator

import pytest
from sqlmodel import SQLModel, create_engine

from app import db, repository


@pytest.fixture(autouse=True)
def temporary_db(monkeypatch: pytest.MonkeyPatch) -> Iterator[None]:
    with tempfile.NamedTemporaryFile(suffix=".db") as tmp:
        engine = create_engine(
            f"sqlite:///{tmp.name}", connect_args={"check_same_thread": False}
        )
        monkeypatch.setattr(db, "engine", engine)
        SQLModel.metadata.create_all(engine)
        repository.seed_movies(engine)
        yield
```

### Parametrized Tests (`tests/test_movies.py`)
```python
import pytest
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


@pytest.mark.parametrize(
    "payload",
    [
        {"title": "Interstellar", "year": 2014, "genre": "Sci-Fi"},
        {"title": "Hidden Figures", "year": 2016, "genre": "Drama"},
        {"title": "Inside Out", "year": 2015, "genre": "Animation"},
    ],
)
def test_create_movie_accepts_valid_payloads(payload):
    response = client.post("/movies", json=payload)
    assert response.status_code == 201
    body = response.json()
    assert body["title"] == payload["title"]
    assert body["genre"] == payload["genre"]


def test_rating_requires_existing_movie():
    payload = {"movie_id": 9999, "user_id": 7, "score": 4}
    response = client.post("/ratings", json=payload)
    assert response.status_code == 404


def test_rating_updates_top_movies():
    movie = client.post(
        "/movies",
        json={"title": "Test Feature", "year": 2023, "genre": "Thriller"},
    ).json()
    client.post(
        "/ratings",
        json={"movie_id": movie["id"], "user_id": 1, "score": 5},
    )
    top = client.get("/movies/top?limit=1").json()
    assert top
    assert top[0]["title"] == "Test Feature"
    assert top[0]["average_rating"] == 5
```

### Run the Suite
```bash
uv run pytest -q
```
Encourage students to watch for failing assertions and tie them back to missing edge cases.

## Part C – Hands-on Lab 2 (45 Minutes) – Logging and Profiling
### Configure Logging
Add to `app/main.py`:
```python
import logging
import time
from fastapi import Request

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger("movie-service")


@app.middleware("http")
async def log_requests(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration_ms = (time.perf_counter() - start) * 1000
    logger.info(
        "method=%s path=%s status=%s duration_ms=%.2f",
        request.method,
        request.url.path,
        response.status_code,
        duration_ms,
    )
    return response
```

### Mini Profiling Exercise
1. Create a slow endpoint for demonstration:
   ```python
   @app.get("/debug/slow")
   def slow_endpoint() -> dict[str, str]:
       time.sleep(0.2)
       return {"status": "slow"}
   ```
2. Call it with `curl http://localhost:8000/debug/slow` and observe the log line showing ~200 ms.
3. Remove or comment out the endpoint after the demo.

### Student Task
- Break a test intentionally, rerun pytest, and read the failure message.
- Trigger logging by calling `/movies`, `/movies/top`, and `/ratings`.
- Capture a log snippet highlighting method, path, status, and duration for their notes.

## EX2 Work Sprint (10 Minutes)
- Remind: Exercise 2 due Tuesday, Dec 23, 2025.
- Encourage students to use the logs to debug their UI/API integration.
- Offer office hours sign-up sheet for anyone behind schedule.

## Troubleshooting
- If logs appear twice, check for duplicate calls to `logging.basicConfig`.
- To silence uvicorn access logs during debugging, run `uv run uvicorn app.main:app --reload --log-level warning`.
- When parametrized tests appear to share state, ensure fixtures recreate the database for each test.

## Student Success Criteria
- Pytest suite covers movie creation, ratings, and the `/movies/top` endpoint without regressions.
- Request logs show method, path, status, and duration for each request.
- Students can explain how they would time a slow function or endpoint.
- Students know the Storage module soft deadline (**Dec 9**) and hard deadline (**Dec 16**) and have a plan to submit the completion proof on time.

## AI Prompt Kit (Copy/Paste)
- “Write parametrized pytest tests for a FastAPI movie service covering valid movie payloads, rating validation, and `/movies/top`. Use `TestClient`.”
- “Add an HTTP middleware to FastAPI that logs movie service requests (method/path/status/duration) using `logging` + `time.perf_counter`.”
- “Show a minimal example of timing a Python function with `time.perf_counter` and printing the elapsed milliseconds.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `pytest parametrized tests example`
- **ChatGPT prompt:** “Summarize the difference between unit, integration, and end-to-end tests in under 150 words for undergrads.”
