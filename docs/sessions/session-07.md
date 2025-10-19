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

## Part B – Hands-on Lab 1 (45 Minutes) – Test Expansion
### Parametrized Tests
Add to `tests/test_items.py`:
```python
import pytest


@pytest.mark.parametrize(
    "payload",
    [
        {"name": "Marker", "quantity": 3},
        {"name": "Notebook", "quantity": 0},
        {"name": "Eraser", "quantity": 100},
    ],
)
def test_create_accepts_valid_payloads(payload):
    response = client.post("/items", json=payload)
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == payload["name"]
    assert data["quantity"] == payload["quantity"]
```

### Error Case Tests
```python
def test_create_rejects_invalid_quantity():
    response = client.post("/items", json={"name": "Pen", "quantity": -1})
    assert response.status_code == 422
    assert "quantity" in response.text


def test_delete_missing_item_returns_404():
    response = client.delete("/items/999")
    assert response.status_code == 404
```

### Run the Suite
```bash
uv run pytest -q
```
Highlight how failures clearly indicate which case broke.

## Part C – Hands-on Lab 2 (45 Minutes) – Logging and Profiling
### Configure Logging
Add to `app/main.py`:
```python
import logging
import time
from fastapi import Request

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger("items-service")


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
   @app.get("/slow")
   def slow_endpoint() -> dict[str, str]:
       time.sleep(0.2)
       return {"status": "slow"}
   ```
2. Call it with `curl http://localhost:8000/slow` and observe log output showing ~200 ms.
3. Remove or comment out the slow endpoint after the demo, keeping the profiling technique.

### Student Task
- Intentionally fail a test, read the failure, and fix it.
- Trigger logging by hitting `/items` and `/items/1`.
- Capture a log snippet for their notes.

## EX2 Work Sprint (10 Minutes)
- Remind: Exercise 2 due Tuesday, Dec 23, 2025.
- Encourage students to use the logs to debug their UI/API integration.
- Offer office hours sign-up sheet for anyone behind schedule.

## Troubleshooting
- If logs appear twice, check for duplicate calls to `logging.basicConfig`.
- To silence uvicorn access logs during debugging, run `uv run uvicorn app.main:app --reload --log-level warning`.
- When parametrized tests appear to share state, ensure fixtures recreate the database for each test.

## Student Success Criteria
- Tests cover both success and failure scenarios and run green.
- Request logs show method, path, status, and duration for each request.
- Students can explain how they would time a slow function or endpoint.
- Students know the Storage module soft deadline (**Dec 9**) and hard deadline (**Dec 16**) and have a plan to submit the completion proof on time.

## AI Prompt Kit (Copy/Paste)
- “Write parametrized pytest tests for a FastAPI items service covering valid and invalid payloads, and a 404 path for GET /items/{id}. Use `TestClient`.”
- “Add an HTTP middleware to FastAPI that logs method, path, status, and duration in milliseconds using `logging` and `time.perf_counter`. Return the response unchanged.”
- “Show a minimal example of measuring function execution time with `time.perf_counter` and printing the result.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `pytest parametrized tests example`
- **ChatGPT prompt:** “Summarize the difference between unit, integration, and end-to-end tests in under 150 words for undergrads.”
