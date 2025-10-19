# Session 09 – Async Jobs and Reliability Basics

- **Date:** Monday, Dec 29, 2025
- **Theme:** Keep the API responsive by offloading long work to background tasks and making clients resilient.

## Learning Objectives
- Distinguish between blocking and non-blocking operations in web services.
- Schedule background jobs using FastAPI’s `BackgroundTasks` utility.
- Implement client-side retry logic with backoff and understand idempotency.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm-up chat | 10 min | Discussion | Share holiday wins and one reliability failure you have seen |
| Async fundamentals | 20 min | Talk + diagrams | Blocking vs. non-blocking, event loop basics |
| Reliability vocabulary | 15 min | Talk | Retries, backoff, idempotency, graceful shutdown |
| Lab 1 | 45 min | Guided coding | Add background CSV import simulation |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Add retries and shutdown handling |
| EX3 preview | 10 min | Talk | Describe advanced feature options |

## Teaching Script – Async Overview
1. Draw two timelines:
   - Blocking: request waits 10 seconds for a CSV parse.
   - Non-blocking: request returns immediately with “processing” status.
2. Explain: “Background tasks let us finish long-running work after the HTTP response is sent.”
3. Introduce key terms:
   - **Idempotency:** making the same request twice yields the same effect.
   - **Backoff:** waiting longer between retries to avoid hammering the server.
   - **Graceful shutdown:** cleaning up tasks when the service stops.
4. Set context for Exercise 3: “At least one team member should pick an advanced feature around async jobs, auth, or observability. Today covers the async option.”

## Part B – Hands-on Lab 1 (45 Minutes)
### Dependencies
Ensure required packages are installed in your project:
```bash
uv add fastapi httpx pytest
```

### Add Background Task Support
Update `app/main.py`:
```python
import time
import uuid
from fastapi import BackgroundTasks

TASK_STATUS: dict[str, str] = {}


def run_import_task(task_id: str) -> None:
    """Pretend to parse a CSV file and store a summary."""
    TASK_STATUS[task_id] = "running"
    time.sleep(2)  # simulate slow work
    TASK_STATUS[task_id] = "complete"
```

### Create the Endpoints
```python
@app.post("/imports", status_code=202)
async def start_import(background: BackgroundTasks) -> dict[str, str]:
    task_id = str(uuid.uuid4())
    # Record an initial state so status is never "unknown" right after enqueueing
    TASK_STATUS[task_id] = "queued"
    background.add_task(run_import_task, task_id)
    return {"task_id": task_id, "status": "queued"}


@app.get("/imports/{task_id}")
async def get_import_status(task_id: str) -> dict[str, str]:
    status = TASK_STATUS.get(task_id, "unknown")
    return {"task_id": task_id, "status": status}
```
(Replace `api` with your FastAPI instance variable name; likely `app`.)

### Demo Flow
1. Start the server.
   ```bash
   uv run uvicorn app.main:app --reload
   ```
2. Start an import:
   ```bash
   curl -X POST http://localhost:8000/imports
   ```
3. Poll the status endpoint every second until it returns `"complete"`.
4. Show that the POST request returns immediately while work continues in the background.

### Discussion
- Talk about storing results: memory dictionary works for demo; in EX3 teams should persist to the database.
- Mention that background tasks run inside the same process. For heavy work they may need separate workers later.

## Part C – Hands-on Lab 2 (45 Minutes)
### Client-Side Retry Helper
Create `app/retry_client.py`:
```python
import random
import time
from typing import Any, Callable

import httpx


def with_backoff(
    func: Callable[[], httpx.Response],
    *,
    retries: int = 3,
    backoff_factor: float = 0.5,
    jitter: float = 0.1,
) -> httpx.Response:
    """Retry a function that performs HTTP requests."""
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

Use the helper to poll the import status and discuss why exponential backoff prevents overload.

### Graceful Shutdown
Add shutdown handling to `app/main.py`:
```python
@app.on_event("shutdown")
def on_shutdown() -> None:
    if any(status == "running" for status in TASK_STATUS.values()):
        print("Warning: background tasks still running; consider persisting state")
```
Explain real-world behavior: in production, we would cancel tasks or persist them for later resumption.

### Reflection Questions
- How would you make `run_import_task` idempotent? (Suggestion: operate on a specific file ID and store results before marking complete.)
- When should the client stop retrying? (Answer: after a reasonable number of attempts or if errors are not recoverable.)

## Exercise 3 Preview
- **Assigned:** Monday, Jan 5, 2026.
- **Milestone demo:** Tuesday, Jan 20, 2026.
- **Final due:** Tuesday, Feb 10, 2026.
- Advanced feature options: async background job (today’s topic), JWT auth, or observability metrics/logging.

## Troubleshooting
- If background tasks never run, verify you did not make the handler `async def run_import_task`. Background tasks expect a synchronous callable.
- For duplicate task IDs, ensure `uuid.uuid4()` is called inside the endpoint.
- If retry helper never stops, confirm `raise_for_status()` is only retried on safe status codes (4xx vs. 5xx).

## Student Success Criteria
- POST `/imports` returns quickly with a task ID.
- GET `/imports/{task_id}` shows `running` then `complete` without locking up the server.
- Students can explain why exponential backoff protects both client and server.

## Quick Test to Verify Background Tasks
Create `tests/test_imports.py`:
```python
import time
from fastapi.testclient import TestClient
from app.main import app


client = TestClient(app)


def test_import_lifecycle():
    # Start an import
    start = client.post("/imports")
    assert start.status_code == 202
    task_id = start.json()["task_id"]

    # Immediately check status (may be queued or running)
    status = client.get(f"/imports/{task_id}").json()["status"]
    assert status in {"queued", "running", "complete", "unknown"}

    # Poll until complete (max ~5 seconds)
    deadline = time.time() + 5
    while time.time() < deadline:
        state = client.get(f"/imports/{task_id}").json()["status"]
        if state == "complete":
            break
        time.sleep(0.2)
    assert client.get(f"/imports/{task_id}").json()["status"] == "complete"
```

Run tests:
```bash
uv run pytest -q
```

## AI Prompt Kit (Copy/Paste)
- “Add a background CSV import to a FastAPI app using `BackgroundTasks`. Implement POST `/imports` that returns a task id and GET `/imports/{id}` that returns `queued|running|complete`. Use an in-memory status map.”
- “Write a simple exponential backoff wrapper for Python httpx requests with jitter, and show an example of retrying a GET request.”
- “Explain idempotency for REST APIs in one paragraph and propose a header or parameter to make POST `/imports` idempotent.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI BackgroundTasks example`
- **ChatGPT prompt:** “Describe exponential backoff with jitter in two sentences for beginners.”
