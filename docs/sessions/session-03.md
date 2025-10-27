# Session 03 – FastAPI Fundamentals (Movie Edition)

- **Date:** Monday, Nov 17, 2025
- **Theme:** Build the first version of the movie recommendation backend with FastAPI and confidence-building tests.

## Learning Objectives
- Map HTTP routes to Python functions with FastAPI.
- Use Pydantic models for movie payload validation.
- Write unit and integration tests with `TestClient`.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & motivation | 10 min | Talk | Why we need our own movie service rather than httpbin |
| FastAPI walkthrough | 20 min | Talk + live coding | Anatomy of a FastAPI app |
| Testing mindset | 15 min | Talk + discussion | Benefits of tests, how they guide development |
| Lab 1 | 45 min | Guided coding | Build in-memory `/movies` API |
| Break | 10 min | — | Launch [10-minute timer](https://e.ggtimer.com/10minutes) and stretch |
| Lab 2 | 45 min | Guided coding | Create tests and run them |
| Q&A on EX1 | 10 min | Discussion | Clarify assignment questions |

## Teaching Script – Why FastAPI?
1. “We want to own the movie catalogue. Today we become the API provider.”
2. FastAPI maps functions to routes and validates payloads using Pydantic.
3. Tests give us confidence so later refactors (SQLite, Redis, ALS) don’t break things.
4. Write on the board the EX1 checklist: `/movies` CRUD, validation, tests, Docker.

## Part B – Hands-on Lab 1 (45 Minutes)
### Setup Commands
```bash
cd hello-uv
uv add fastapi uvicorn pydantic httpx pytest
mkdir -p app
touch app/__init__.py app/main.py
```

### Live Coding Script (`app/main.py`)
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Movie Service", version="0.1.0")


class Movie(BaseModel):
    id: int
    title: str
    year: int = Field(ge=1900, le=2100)
    genre: str


class MovieCreate(BaseModel):
    title: str
    year: int = Field(ge=1900, le=2100)
    genre: str


MOVIES: dict[int, Movie] = {}
NEXT_ID = 1


@app.get("/health")
def health_check() -> dict[str, str]:
    return {"status": "ok"}


@app.post("/movies", response_model=Movie, status_code=201)
def create_movie(payload: MovieCreate) -> Movie:
    global NEXT_ID
    movie = Movie(id=NEXT_ID, **payload.model_dump())
    MOVIES[movie.id] = movie
    NEXT_ID += 1
    return movie


@app.get("/movies/{movie_id}", response_model=Movie)
def read_movie(movie_id: int) -> Movie:
    try:
        return MOVIES[movie_id]
    except KeyError as exc:
        raise HTTPException(status_code=404, detail="Movie not found") from exc
```

### Run the Server
```bash
uv run uvicorn app.main:app --reload
```
Visit `http://localhost:8000/docs` to interact with the auto-generated UI.

### Optional Practice Routes
Add other helpers to `app/main.py`:
```python
@app.get("/movies", response_model=list[Movie])
def list_movies() -> list[Movie]:
    return list(MOVIES.values())


@app.delete("/movies/{movie_id}", status_code=204)
def delete_movie(movie_id: int) -> None:
    if movie_id not in MOVIES:
        raise HTTPException(status_code=404, detail="Movie not found")
    MOVIES.pop(movie_id)
```

Try:
```bash
curl -X POST http://localhost:8000/movies   -H "Content-Type: application/json"   -d '{"title": "Inception", "year": 2010, "genre": "Sci-Fi"}'

curl http://localhost:8000/movies
```

---

## Break (10 Minutes)
Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes), hydrate, and be ready for Part C.

---

## Part C – Hands-on Lab 2 (45 Minutes)
### Create the Test Suite
1. Create `tests/test_movies.py`:
   ```python
   from fastapi.testclient import TestClient
   from app.main import app

   client = TestClient(app)


   def test_health_check():
       response = client.get("/health")
       assert response.status_code == 200
       assert response.json() == {"status": "ok"}


   def test_create_and_read_movie():
       create_response = client.post(
           "/movies",
           json={"title": "The Matrix", "year": 1999, "genre": "Sci-Fi"},
       )
       assert create_response.status_code == 201
       movie = create_response.json()

       read_response = client.get(f"/movies/{movie['id']}")
       assert read_response.status_code == 200
       assert read_response.json()["title"] == "The Matrix"


   def test_missing_movie_returns_404():
       response = client.get("/movies/999")
       assert response.status_code == 404
   ```
2. Run:
   ```bash
   uv run pytest -q
   ```
3. Encourage students to add a validation test:
   ```python
   def test_create_rejects_bad_year():
       response = client.post(
           "/movies",
           json={"title": "Future", "year": 3020, "genre": "Sci-Fi"},
       )
       assert response.status_code == 422
   ```

### Reinforce Testing Mindset
- Green tests mean the behavior matches expectations.
- Show how a failing test pinpoints the regression.

## EX1 Reminder
- Today’s code is the starting point.
- Homework: add `/movies` listing, `/movies/{id}` update/delete, and tests for each.
- Suggested next steps: introduce ratings, write tests for 404 and validation paths.

## Troubleshooting Notes
- If `uvicorn` cannot import `app.main`, ensure `app/__init__.py` exists.
- If the server port is busy, stop previous runs or change to `--port 8001`.
- Validation errors show detailed JSON; read them aloud when debugging.

## Student Success Criteria
- Server returns 200 from `/health`.
- Students can create and read movies via the API.
- Tests run green and include at least one failure case (404 or validation).

## AI Prompt Kit (Copy/Paste)
- “Write a FastAPI endpoint that accepts movie metadata (title/year/genre), validates fields, and returns a JSON response.”
- “Generate pytest tests for POST `/movies` and GET `/movies/{id}` covering happy path and 404 path.”
- “Explain how to structure FastAPI apps so core logic (movie storage) can later be swapped with SQLite.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI pydantic BaseModel tutorial`
- **ChatGPT prompt:** “Explain, in three bullet points, how to organize FastAPI tests that mix pure function tests with TestClient integration checks.”
