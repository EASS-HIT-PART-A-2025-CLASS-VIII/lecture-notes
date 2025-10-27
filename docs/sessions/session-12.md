# Session 12 – Tool-Friendly APIs and Final Prep

- **Date:** Monday, Jan 26, 2026
- **Theme:** Polish the API for external tools (MCP-friendly), lock in documentation/testing pipelines, and prep EX3 final presentations.

## Learning Objectives
- Expose tool-friendly endpoints with deterministic request/response models and OpenAPI examples (happy + sad paths).
- Add finishing touches: ETag/`If-None-Match`, pagination, feature flags, CSV export endpoint.
- Automate docs + quality gates (MkDocs/pdocs, Ruff, mypy, pre-commit, changelog).
- Plan release checklist for EX3, including deployment commands and verification steps.

## Before Class – Final Prep (JiTT)
- Ensure EX3 repos are up to date, Docker Compose stack runs cleanly, and tests pass with coverage + Schemathesis.
- Install doc tooling if not already:
  ```bash
  uv add "mkdocs-material==9.*" "pdocs==1.*" "ruff==0.*" "mypy==1.*" "pre-commit==3.*"
  ```
- Draft a release checklist outline (who, what, when) for your EX3 team.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| EX3 dry run | 20 min | Student demos | Show the Compose stack + tool endpoint + observability dashboards. |
| Tool-friendly design | 15 min | Talk | Deterministic schemas, pagination strategy, ETags, versioning. |
| Micro demo: ETag handshake | 5 min | Live demo | `curl` with `If-None-Match` to show 304 responses. |
| Release hygiene | 15 min | Talk | Pre-commit, Ruff, mypy, docs generation, changelog management. |
| **Part B – Lab 1** | **45 min** | **Guided polish** | **Add pagination, ETags, CSV export, OpenAPI examples.** |
| Break | 10 min | — | Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes). |
| **Part C – Lab 2** | **45 min** | **Guided automation** | **Docs build, pre-commit hooks, release checklist rehearsal.** |
| Closing circle | 10 min | Discussion | Reflect on growth, commit to next steps, celebrate wins.

## Part A – Theory Highlights
1. **Tool readiness:** consistent schema, explicit examples, stable ids (ULIDs vs ints). For now keep ints but document transition plan.
2. **Pagination + filtering conventions:** `?page=1&page_size=20`, `X-Total-Count` header, link relations.
3. **ETag caching:** Return `ETag` for list endpoints, support conditional GET to reduce load.
4. **Docs automation:** MkDocs or pdocs to publish API docs; pre-commit ensures formatting/lint before commits; changelog via Conventional Commits.
5. **Release checklist:** version bump, `uv sync --frozen`, `docker compose build`, smoke tests, tag release, update docs.

## Part B – Lab 1 (45 Minutes)
### 1. Pagination helpers (`app/pagination.py`)
```python
from math import ceil
from typing import Sequence

from fastapi import Query


def paginate(items: Sequence, page: int = Query(1, ge=1), page_size: int = Query(20, ge=1, le=100)):
    total = len(items)
    pages = ceil(total / page_size) if page_size else 1
    start = (page - 1) * page_size
    end = start + page_size
    return items[start:end], {"page": page, "page_size": page_size, "total": total, "pages": pages}
```
Update `/movies` route:
```python
from fastapi import Response
from app.pagination import paginate


@app.get("/movies", response_model=list[Movie], openapi_extra={
    "responses": {
        200: {
            "description": "List movies",
            "content": {
                "application/json": {
                    "examples": {
                        "default": {
                            "summary": "First page",
                            "value": [{"id": 1, "title": "Arrival", "year": 2016, "genre": "Sci-Fi"}],
                        }
                    }
                }
            },
        }
    }
})
async def list_movies(response: Response, repository: RepositoryDep, settings: SettingsDep, page: int = 1, page_size: int = 20) -> list[Movie]:
    movies = [movie for movie in repository.list()]
    page_items, meta = paginate(movies, page, page_size)
    response.headers["X-Total-Count"] = str(meta["total"])
    response.headers["X-Total-Pages"] = str(meta["pages"])
    return page_items
```

### 2. ETag support
```python
import hashlib
import json
from fastapi import Request, Response, status


def compute_etag(payload: str) -> str:
    return hashlib.sha256(payload.encode("utf-8")).hexdigest()


@app.get("/movies", response_model=list[Movie], ...)
async def list_movies(
    request: Request,
    response: Response,
    repository: RepositoryDep,
    settings: SettingsDep,
    page: int = 1,
    page_size: int = 20,
) -> list[Movie]:
    movies = [movie for movie in repository.list()]
    page_items, meta = paginate(movies, page, page_size)
    response.headers["X-Total-Count"] = str(meta["total"])
    response.headers["X-Total-Pages"] = str(meta["pages"])

    payload = json.dumps([movie.model_dump() for movie in page_items], sort_keys=True)
    etag = compute_etag(payload)
    if request.headers.get("If-None-Match") == etag:
        return Response(status_code=status.HTTP_304_NOT_MODIFIED)

    response.headers["ETag"] = etag
    response.headers["Cache-Control"] = "public, max-age=60"
    return page_items
```
Ensure tests cover 200 + 304 paths.

### 3. CSV export endpoint
```python
from fastapi.responses import StreamingResponse


@app.get("/movies/export.csv")
async def export_movies_csv(repository: RepositoryDep) -> StreamingResponse:
    def generate():
        yield "id,title,year,genre\n"
        for movie in repository.list():
            yield f"{movie.id},{movie.title},{movie.year},{movie.genre}\n"

    return StreamingResponse(generate(), media_type="text/csv")
```
Add tests verifying content type, header, sample rows.

### 4. Feature flags (config driven)
Use `Settings.feature_preview` to gate experimental endpoints (toggle via `.env`). Document in README.

## Part C – Lab 2 (45 Minutes)
### 1. Documentation build
- Generate API docs: `uv run pdocs serve app` or `uv run mkdocs serve` (choose one per team).
- Publish `docs/service-contract.md` updates with OpenAPI examples, rate limiting info, and agent endpoints.

### 2. Pre-commit + lint/type checks
```bash
cat <<'CFG' > .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
CFG

pre-commit install
pre-commit run --all-files
```
Add `uv run ruff check .` and `uv run mypy app` to CI.

### 3. Changelog & release checklist
- Adopt Conventional Commits (`feat:`, `fix:`, `docs:`) or equivalent; generate changelog with `git cliff` or manual notes.
- Final release checklist example:
  1. Run `uv run pytest --cov` + `schemathesis` + `ruff` + `mypy`.
  2. Build Docker images (`docker compose build`).
  3. Run smoke tests (`docker compose run api uv run pytest tests/smoketests`).
  4. Tag release (`git tag -a v0.3.0 -m "EASS EX3"`).
  5. Publish docs (`uv run mkdocs build && netlify deploy` or similar).

### 4. MCP teaser
Preview how today’s deterministic responses feed directly into the optional MCP workshop (tool endpoints for agents). Encourage teams to read `sessions/optional/mcp.md` before elective session.

## Closing Circle
- Share one capability you can now ship with confidence (e.g., async pipelines, secure auth, containers).
- Commit to final EX3 deliverable timeline.
- Celebrate wins—this wraps the 12-week reboot!

## Troubleshooting
- **ETag mismatches** → ensure `ETag` computed on canonical JSON (sorted keys). Consider `json.dumps(..., sort_keys=True)`.
- **Pre-commit slow** → use `--hook-stage manual` for heavy hooks, or run `uv run ruff --watch` during dev.
- **CSV export encoding issues** → enforce UTF-8 and escape commas if titles contain them (use `csv` module if needed).

## AI Prompt Seeds
- “Add pagination, total count headers, and OpenAPI examples to a FastAPI list endpoint.”
- “Implement conditional GET with ETag/If-None-Match for a JSON response.”
- “Generate a release checklist covering tests, Docker build, documentation, and tagging.”
