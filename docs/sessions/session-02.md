# Session 02 – Introduction to HTTP and REST

- **Date:** Monday, Nov 10, 2025
- **Theme:** Demystify HTTP request/response flows and capture reusable probes that will shape Exercise 1.

## Learning Objectives
- Break down every component of an HTTP request and response (method, path, query, headers, cookies, body, status code).
- Contrast “resource-first” REST patterns with ad-hoc RPC calls so students can spot good API design.
- Call public APIs with `httpx`, `curl`, and the VS Code REST Client, piping outputs through `python -m json.tool` for fast inspection.
- Automate exploratory requests behind a Typer CLI so EX1 debugging is repeatable.
- Document baseline contracts (`.http` files + error format checklist) that evolve into EX1 tests.

## Before Class – REST Warm-Up (JiTT)
- In your `hello-uv` workspace run: 
  ```bash
  uv add "httpx==0.*" "pydantic==2.*" "typer==0.*"
  ```
  Post the command output in Discord `#helpdesk` using **Problem → Action → Result → Desired** if anything fails.
- Install the VS Code **REST Client** extension and run:
  ```bash
  curl https://httpbin.org/get?ping=preflight | python -m json.tool
  ```
  Share the pretty-printed JSON screenshot in your lab thread so everyone confirms the tooling.
- Complete the **AWS Academy Cloud Foundations – Compute** module by **Tue Nov 25**; flag blockers early.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & AWS checkpoint | 5 min | Guided discussion | Round-robin: what automation/aliases did you add after Session 01? Confirm Compute module progress. |
| HTTP dissection & tooling | 18 min | Talk + board + devtools | Methods, status codes, headers, caching, auth, and where CORS/rate limiting appear. |
| Micro demo: `curl → json.tool` | 2 min | Live demo (≤120 s) | Show CLI piping raw JSON into `python -m json.tool` and highlight why we log every request. |
| REST design patterns | 20 min | Talk + whiteboard | Resource naming, idempotency, error normalization, OpenAPI examples, and upcoming `slowapi` rate limiting. |
| **Part B – Lab 1** | **45 min** | **Guided coding** | **Build a reusable HTTP probe with `httpx` + Typer CLI.** |
| Break | 10 min | — | Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes) and stretch. |
| **Part C – Lab 2** | **45 min** | **Guided practice** | **Capture `.http` recipes, validate JSON, prep contract-test inputs.** |
| EX1 briefing & backlog | 10 min | Talk + Q&A | Scope reminder, backlog items (pagination, rate limiting, Schemathesis). |

## Part A – Theory & Micro Demo (45 Minutes)
1. **Board sketch:** Browser → FastAPI (Session 03) → SQLite (Session 05) → Redis (Session 10). Label each hop with verbs (`GET`, `POST`) and metadata (headers, trace IDs, content-type).
2. **Status code ladder:** 2xx success, 3xx redirects, 4xx client errors, 5xx server errors. Stress that EX1 must never leak stack traces—only structured JSON errors.
3. **Header callouts:** `Accept`, `Content-Type`, `Authorization`, `X-Trace-Id`, `Retry-After`. Explain how we will inject a trace ID even before full observability tooling.
4. **Micro demo (≤120 s):**
   ```bash
   curl -s https://httpbin.org/get?city=Haifa \
     -H "Accept: application/json" \
   | uv run python -m json.tool
   ```
   Ask: “What are the headers? Where would you stash correlation IDs?”
5. **REST heuristics:** Use nouns (`/movies`, `/movies/{movie_id}/ratings`), keep verbs in query/body, make `PUT` idempotent, and document error shapes.
6. **Preview EX1 contract requirements:** health endpoint, CRUD, predictable error payloads, coverage reports (Session 07), Docker packaging (Session 04).

## Part B – Hands-on Lab 1 (45 Minutes)
### 1. Setup commands
```bash
cd hello-uv
uv add "httpx==0.*" "pydantic==2.*" "typer==0.*"
mkdir -p app
touch app/__init__.py app/http_client.py app/cli.py
```

### 2. Implement `app/http_client.py`
```python
from __future__ import annotations

import logging
import uuid
from typing import Any

import httpx
from pydantic import BaseModel

logger = logging.getLogger(__name__)


class PingResponse(BaseModel):
    args: dict[str, Any]
    headers: dict[str, Any]
    url: str
    origin: str


def ping(city: str = "Haifa") -> PingResponse:
    """Call httpbin with a trace header and validate the JSON response."""
    trace_id = uuid.uuid4().hex[:8]
    with httpx.Client(timeout=httpx.Timeout(10.0)) as client:
        response = client.get(
            "https://httpbin.org/get",
            params={"city": city},
            headers={
                "Accept": "application/json",
                "X-Trace-Id": trace_id,
            },
        )
        response.raise_for_status()
        payload = PingResponse.model_validate(response.json())
        logger.info(
            "httpbin echo",
            extra={"trace_id": trace_id, "status_code": response.status_code},
        )
        return payload
```

### 3. Wire a Typer CLI (`app/cli.py`)
```python
import json
import logging
from typing import Optional

import typer

from .http_client import ping

logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")

app = typer.Typer(help="HTTP probes for EX1 preflight")


@app.command()
def echo(city: str = typer.Argument("Haifa"), pretty: bool = True) -> None:
    """Call httpbin with optional pretty JSON output."""
    response = ping(city=city)
    if pretty:
        typer.echo(json.dumps(response.model_dump(), indent=2))
    else:
        typer.echo(response.model_dump_json())


@app.command()
def headers(city: str = "Haifa") -> None:
    """Print the normalized headers we receive back."""
    response = ping(city=city)
    for key, value in response.headers.items():
        typer.echo(f"{key}: {value}")


def main(argv: Optional[list[str]] = None) -> None:
    app(standalone_mode=True)


if __name__ == "__main__":
    main()
```

### 4. Run the probe
```bash
uv run python -m app.cli echo --city Tel-Aviv
uv run python -m app.cli headers --city London
```
Ask students to highlight the `X-Trace-Id` header in the output and explain why we will propagate it into FastAPI logs next week.

### 5. Optional extension – quick pytest
```bash
mkdir -p tests
cat <<'PY' > tests/test_ping.py
from app.http_client import ping


def test_ping_default_city():
    response = ping()
    assert response.args["city"] == "Haifa"
    assert response.url.startswith("https://httpbin.org")
PY

uv run pytest -q
```
Encourage students to keep the test even if httpbin occasionally flakes—later we will replace it with a mocked contract.

## Part C – Hands-on Lab 2 (45 Minutes)
### 1. Create reusable `.http` scripts
`requests.http`:
```
### GET echo
GET https://httpbin.org/get?city=Haifa
Accept: application/json
X-Trace-Id: demo-1234

### POST form example
POST https://httpbin.org/post
Content-Type: application/x-www-form-urlencoded
X-Trace-Id: demo-5678

name=EASS
cohort=2025

### HEAD request for metadata
HEAD https://httpbin.org/get
Accept: application/json
```
Demonstrate sending each block from VS Code (Cmd/Ctrl+Alt+R) and show the response panel. Save useful responses for EX1 debugging.

### 2. Normalize error payloads
Draft `docs/contracts/http-errors.md` with a minimal template:
```markdown
## Standard Error Envelope
- `status`: HTTP status code (int)
- `error`: machine-readable string (e.g., `resource_not_found`)
- `detail`: human explanation
- `trace_id`: echo the inbound `X-Trace-Id`
```
Explain how this document becomes a checklist when students implement `/movies`.

### 3. Stretch: Schemathesis smoke test
```bash
uv add "schemathesis==3.*"
urql="https://httpbin.org/spec.json"  # replace with EX1 OpenAPI once available
uv run schemathesis run "$urql" --checks status_code_conformance
```
Reinforce that contract tests are optional now but required for excellence submissions.

## EX1 Briefing & Backlog Hit List
- Deliverable: FastAPI CRUD for `/movies`, deterministic JSON errors, 80% branch coverage, Docker image (Session 04) by **Tue Dec 2**.
- **Backlog ideas:** feature flags for beta endpoints, pagination & filtering conventions, rate limiting (`slowapi`) with a 429 test, OpenAPI examples for happy/sad paths, ETag/`If-None-Match` demo for caching.
- Encourage journaling any stretch goals so we can fold them into Sessions 07–10.

## Troubleshooting Notes
- `urllib3` SSL errors: run `export SSL_CERT_FILE=$(python -m certifi)` on macOS if needed.
- `httpx.ConnectTimeout`: demonstrate adding `timeout=httpx.Timeout(30.0)` and retrying later with `async` client in Session 09.
- VS Code REST Client not installed? Use `uv run python -m http.client` as a backup but fix the extension before EX1.

## AI Prompt Seeds
- “Explain like I’m a TA how to surface `X-Trace-Id` headers from `httpx` responses in logs.”
- “Generate Typer commands that wrap a reusable FastAPI probe with pretty JSON output.”
- “Draft a JSON error envelope spec that keeps parity between 400-level and 500-level responses.”
