# Session 08 – Working with AI Coding Assistants

- **Date:** Monday, Dec 22, 2025
- **Theme:** Use AI responsibly to speed up coding while staying in control of the output.

## Learning Objectives
- Practice safe prompting patterns for code generation and refactoring.
- Review generated code critically and back it with tests.
- Call a local LLM endpoint programmatically.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Check-in & EX2 demos | 20 min | Student demos | Each team shows their UI talking to the API |
| AI usage policy | 15 min | Talk | Expectations, attribution, academic integrity |
| Prompt patterns | 20 min | Talk + live examples | Spec-first, tests-first, refactor requests |
| Lab 1 | 45 min | Guided coding | Use AI to extend the API, then review and test |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Call LM Studio (local LLM) from Python |
| Retrospective | 10 min | Discussion | Share wins, blockers, and next steps |

## Teaching Script – AI Ground Rules
1. “AI is a teammate, not an autopilot. You remain responsible for the code.”
2. Outline the class policy: record prompts or summarize AI assistance in the README; never submit code you do not understand; always run tests after accepting AI suggestions.
3. Present three safe prompt structures:
   - **Spec-first:** “Given these requirements, draft the function.”
   - **Tests-first:** “Write pytest cases for these behaviors.”
   - **Refactor:** “Here is existing code; make it cleaner without changing behavior.”
4. Warn against copying random snippets from the internet without validating them.

## Part B – Hands-on Lab 1 (45 Minutes)
### Prompt 1 – Update Movie Metadata via AI
Use Cursor, Copilot Chat, or ChatGPT. Suggested prompt:
```
You are assisting with a FastAPI movie recommendation service that already exposes
GET /movies and POST /movies.
Add PUT /movies/{id} to update title/year/genre, DELETE /movies/{id} to remove a movie,
and update the pytest suite to cover these behaviours. Keep responses small and explain each change.
```

### Review AI Output for Prompt 1
- Check for validation logic, HTTP status codes, and consistent models.
- Reject anything that removes existing tests without replacing them.

### Verify with Tests (Prompt 1)
```bash
uv run pytest -q
```
If tests fail, debug manually. Emphasize that AI suggestions are starting points, not truth.

### Commit Message (Prompt 1)
Encourage students to use AI to draft a commit message, but edit it for clarity before committing.

### Prompt 2 – Scaffold a Recommendation Stub
After merging the CRUD improvements, have AI draft a simple collaborative filtering helper. Suggested prompt:
```
We store movie ratings with fields (movie_id, user_id, score).
Write a Python module `app/recommender.py` that exposes `build_user_matrix(ratings)` and
`recommend_for_user(ratings, user_id, k=5)` using a lightweight ALS-style approach (NumPy only).
Return movie_id recommendations sorted by predicted score. Include docstrings and type hints.
```

### Integrate the Stub Manually
1. Create `app/recommender.py` from the AI response.
2. In `app/main.py`, add a placeholder route:
   ```python
   from app import recommender

   @app.get("/recommendations/{user_id}")
   def recommend(user_id: int, limit: int = 5) -> list[int]:
       ratings = repository.list_ratings()
       return recommender.recommend_for_user(ratings, user_id, k=limit)
   ```
   (This will return empty lists until Session 09 builds the background job.)
3. Ask AI to generate pytest cases that feed a small rating sample into `recommend_for_user` and assert on the ordering.

### Verify with Tests (Prompt 2)
```bash
uv add numpy
uv run pytest -q
```
Address any numerical edge cases before moving on.


## Part C – Hands-on Lab 2 (45 Minutes) – Local LLM Call
### Start LM Studio
- Launch LM Studio and load a model that supports chat completions.
- Note the local API URL (commonly `http://localhost:1234/v1`).

### Create `scripts/query_llm.py` (reuse from Session 08)
Install dependency if needed:
```bash
uv add httpx
```
```python
import httpx

MODEL = "local-llm"
ENDPOINT = "http://localhost:1234/v1/chat/completions"

PROMPT = """
Write a FastAPI route that returns a JSON response with status: ok when /health is requested.
Use async def.
"""

payload = {
    "model": MODEL,
    "messages": [
        {"role": "system", "content": "You help write concise FastAPI endpoints."},
        {"role": "user", "content": PROMPT.strip()},
    ],
    "temperature": 0.2,
}

response = httpx.post(ENDPOINT, json=payload, timeout=30)
response.raise_for_status()
message = response.json()["choices"][0]["message"]["content"]
print(message)
```

### Run the Script
```bash
uv run python scripts/query_llm.py
```
Copy the output into a scratch file, review it, and discuss what needs to change before using it in production (e.g., missing validation, untested code).

### Group Discussion
- When would you reject AI output completely?
- How do you record AI assistance in your project documentation?

## Retrospective
- Celebrate EX2 completion; highlight thoughtful UI touches across teams.
- Brainstorm improvements for the next assignment (EX3).

## Troubleshooting
- If the AI tool suggests invalid Python, ask it explicitly for syntax-conforming code.
- For LM Studio connection errors, confirm the server is running and the endpoint URL is correct.
- Remind students to disable auto-commit features if their IDEs offer them; commits must remain intentional.

## Student Success Criteria
- PUT and DELETE routes for `/movies/{id}` exist, are tested, and pass pytest locally.
- Students can articulate how they validated AI-generated code and the new recommendation stub.
- `scripts/query_llm.py` successfully receives a response from a local LLM and prints it.

## AI Prompt Kit (Copy/Paste)
- “Given the FastAPI movie service, add PUT `/movies/{id}` and DELETE `/movies/{id}`, update the models, and write pytest coverage for the new behaviour.”
- “Summarize the new `app/recommender.py` module into a README section with install commands, how to trigger `/recommendations/{user_id}`, and test instructions.”
- “Draft a conventional commit message for adding movie update/delete endpoints plus a recommender stub.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI use ChatGPT to write tests safely`
- **ChatGPT prompt:** “Give me a 4-step checklist for reviewing AI-generated code before committing it.”
