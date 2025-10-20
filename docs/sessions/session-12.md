# Session 12 – Tool-Friendly APIs and Final Prep

- **Date:** Monday, Jan 19, 2026
- **Theme:** Shape APIs so they are easy for automation tools and LLMs to consume, and rehearse for the Exercise 3 milestone.

## Learning Objectives
- Design deterministic endpoints with clear success and error schemas.
- Document API usage for both humans and automated tools.
- Validate Exercise 3 projects against a readiness checklist.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm-up | 10 min | Discussion | “What finishing touches did you add to EX3 over the weekend?” |
| Tool-friendly API talk | 25 min | Talk + examples | Deterministic responses, idempotency, explicit error codes |
| Prompt-to-tool demo | 20 min | Live coding | Call an API endpoint from LM Studio or vLLM (Docker) |
| Lab 1 | 45 min | Guided coding | Create `/tool/recommend-movie` with clear schema |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided practice | Run readiness checklist, update docs, rehearse demos |
| Closing circle | 10 min | Discussion | Share biggest lessons from the course |

## Teaching Script – Tool-Friendly Design
1. “Bots are impatient students—they need predictable responses.”
2. Define a good tool endpoint:
   - Input schema fully specified.
   - Output contains `status`, `data`, and `error` fields.
   - Errors use machine-readable codes (for example, `NO_DATA`, `UNAUTHORISED`).
3. Introduce idempotency: sending the same request twice should not create duplicate records.
4. Stress documentation: include example request/response bodies right in the README.

## Part B – Hands-on Lab 1 (45 Minutes)
### Endpoint Implementation
Add to `app/main.py`:
```python
from fastapi import Body
from pydantic import BaseModel


class ToolRecommendation(BaseModel):
    user_id: int
    limit: int = 5


@app.post("/tool/recommend-movie")
async def tool_recommend_movie(
    payload: ToolRecommendation = Body(..., embed=True),
    user: User = Depends(require_role("editor")),
) -> dict[str, object]:
    ratings = repository.list_ratings()
    recommendations = recommender.recommend_for_user(
        ratings,
        user_id=payload.user_id,
        k=payload.limit,
    )
    return {
        "status": "ok",
        "data": {
            "user_id": payload.user_id,
            "recommendations": recommendations,
        },
        "error": None,
    }
```

### Start the API (if not already running)
```bash
uv run uvicorn app.main:app --reload
```
Leave this terminal running while you test the tool endpoint.

### Obtain an Editor Token (Session 11 auth)
In a second terminal, request a JWT for the `teacher` role and export it for reuse:
```bash
curl -X POST http://localhost:8000/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=teacher&password=classroom"

export TOKEN="<paste-access-token-here>"
```

### Documentation Block
Encourage instructors to paste the following example into `docs/service-contract.md`:
```markdown
### POST /tool/recommend-movie
- Request body:
  ```json
  {
    "payload": {
      "user_id": 42,
      "limit": 5
    }
  }
  ```
- Success response:
  ```json
  {
    "status": "ok",
    "data": {
      "user_id": 42,
      "recommendations": [3, 7, 1, 9, 5]
    },
    "error": null
  }
  ```
- Error response (no data yet):
  ```json
  {
    "status": "ok",
    "data": {
      "user_id": 42,
      "recommendations": []
    },
    "error": null
  }
  ```
```

### Tool Call Demo
Use LM Studio or plain `curl`:
```bash
curl -X POST http://localhost:8000/tool/recommend-movie \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"payload": {"user_id": 42, "limit": 5}}'
```
Show how deterministic responses make it easy to parse results.

Optional LM Studio script (save as `scripts/ask_tool.py`):
```python
import httpx

payload = {
    "model": "local-llm",
    "messages": [
        {
            "role": "user",
            "content": "Should I call POST /tool/recommend-movie with name=Notebook?"
        }
    ],
}

response = httpx.post("http://localhost:1234/v1/chat/completions", json=payload, timeout=30)
print(response.json()["choices"][0]["message"]["content"])
```
Run it with:
```bash
uv run python scripts/ask_tool.py
```
Adjust the LM Studio URL/model as needed.

Optional vLLM note: If you started vLLM via Docker in Session 08, point the client to `http://localhost:8000/v1` and set `model` to the tiny model you launched (for example, `TinyLlama/TinyLlama-1.1B-Chat-v1.0`). The OpenAI Python client also works by supplying `base_url` and a dummy `api_key`.

## Part C – Hands-on Lab 2 (45 Minutes)
### Readiness Checklist
 Provide teams with the following list and check off items together:
- `docker compose up --build` succeeds on a clean machine.
- `.env.example` documents required secrets.
- README explains setup, tests, AI assistance, and tool endpoint usage.
- Automated tests pass (`uv run pytest -q`).
- Advanced feature works end-to-end (async job, auth, or observability).
- Optional: metrics endpoint or structured logs accessible through Compose.
- AWS Academy certificates (Compute/Storage/Databases) were uploaded by **Tue Dec 16, 2025** (or the make-up plan is in motion).
- Verify the tool endpoint directly:
  ```bash
  curl -X POST http://localhost:8000/tool/recommend-movie     -H "Content-Type: application/json"     -H "Authorization: Bearer $TOKEN"     -d '{"payload": {"user_id": 42, "limit": 3}}'
  ```
- If using Docker Compose, run a full-system check:
  ```bash
  docker compose up --build
  curl -X POST http://localhost:8080/tool/recommend-movie     -H "Content-Type: application/json"     -H "Authorization: Bearer $TOKEN"     -d '{"payload": {"user_id": 42, "limit": 3}}'
  docker compose down
  ```
  ```

### Demo Rehearsal
- Each team runs through a three-minute milestone presentation:
  1. Problem solved.
  2. Architecture (show Compose services).
  3. Advanced feature demo.
  4. Next steps before Feb 10 final submission.

### Support Planning
- Schedule office hours for the final stretch.
- Encourage teams to list remaining tasks in their project tracker.

## Closing Circle Prompts
- “What concept felt mysterious at the start but now makes sense?”
- “How will you keep practicing after this course?”
- “What feedback do you have for the next cohort?”

## Troubleshooting
- If the tool endpoint requires authentication, remind teams to create a dedicated API token user or to explain how to obtain a token in the README.
- When `curl` fails with 422, check whether the JSON used the `payload` wrapper (because `embed=True`).
- If Compose logs show stale data, remind teams to prune Docker volumes (`docker compose down -v`) and restart.
- If `$TOKEN` is empty, rerun the login `curl` command and export the value again.
- When LM Studio is not running, start it before executing `scripts/ask_tool.py` or skip the optional demo.
- For vLLM on macOS, keep models tiny and prompts short; see Session 08 for Docker startup commands.

## Student Success Criteria
- `/tool/recommend-movie` returns deterministic JSON for both success and error cases.
- Documentation includes copy-paste-ready examples of requests and responses.
- Teams feel ready for the Jan 20 milestone demo and know the next steps for the final submission.
- AWS module submissions confirmed (Dec 16 deadline met) or escalated for make-up.

## Note on Authorization for Tool Endpoint
If you did not implement Session 11 security, temporarily remove the dependency from the tool route:
```python
@app.post("/tool/recommend-movie")
async def tool_recommend_movie(payload: ToolRecommendation) -> dict[str, object]:
    # same logic as above, without a user dependency
    ...
```
Reintroduce the `require_role("editor")` dependency once JWT auth is in place.

## AI Prompt Kit (Copy/Paste)
- “Design a deterministic tool endpoint for the movie service that accepts `{user_id, limit}` and returns `{status,data,error}` with recommendations.”
- “Prepare a milestone readiness checklist for a two-service Docker Compose stack (api + nginx) including `.env.example`, health checks, tests, and documentation.”
- “Write a curl command that calls a protected POST endpoint with a Bearer token and JSON body, and then parse the JSON with `jq` to extract the `id`.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI tool endpoint deterministic response`
- **ChatGPT prompt:** “Provide a 5-point team readiness checklist before a software milestone demo.”
