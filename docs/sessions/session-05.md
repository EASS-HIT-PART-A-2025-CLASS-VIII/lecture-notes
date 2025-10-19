# Session 05 – Movie Service Persistence with SQLite

- **Date:** Monday, Dec 1, 2025
- **Theme:** Turn the in-memory demo into a small Movie Recommendation backend powered by SQLite so it is ready for a frontend and recommendation logic in later sessions.

## Learning Objectives
- Model movies and user ratings with SQLModel and SQLite.
- Provide REST endpoints for browsing movies and recording ratings.
- Seed the database with starter data that later sessions (Streamlit UI, Redis caching, ALS) will build on.
- Adjust automated tests to validate movie and rating behaviour.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm start & EX1 mini demos | 15 min | Student lightning talks | Show CRUD progress and share blockers |
| Persistence primer | 20 min | Talk + sketching | Why movies + ratings need relational tables |
| Architecture mini-lesson | 10 min | Talk | API layer vs. data layer vs. upcoming recommendation logic |
| AWS module check-in | 5 min | Discussion | Confirm Compute module progress; remind Storage soft deadline (Dec 9) |
| Lab 1 | 45 min | Guided coding | Build `Movie` + `Rating` tables, seed sample catalog |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Implement endpoints, rating aggregation, and tests |
| EX2 announcement | 10 min | Talk | Preview the Movie UI build in next session |

## Teaching Script – Why Persistence Matters for Movies
1. Start with the problem statement: “We want a movie recommendation API that remembers titles, genres, and student ratings even after a restart.”
2. Ask: “If we reboot the server, what happens to our in-memory movies list?” Let students observe the data loss.
3. Introduce SQLite as “a lightweight database that keeps our catalog on disk—ideal for laptops and class projects.”
4. Explain SQLModel: “It gives us typed models for movies/ratings and creates tables automatically.”
5. Draw the flow: Request → FastAPI route → Repository → SQLite tables → (later) Redis cache/ALS.
6. Set expectations: “By the end of today, we’ll have `/movies`, `/movies/{id}`, `/ratings`, and `/movies/top` backed by SQLite with starter data.”
7. Remind students about AWS pacing: finish the **Storage** module by **Tuesday, Dec 9, 2025** so the **Tue Dec 16** hard deadline is easy.

## Part B – Hands-on Lab 1 (45 Minutes)
### Install Packages
```bash
cd hello-uv
uv add sqlmodel sqlalchemy
mkdir -p app
```

### Database Module (`app/db.py`)
Create `app/db.py`:
```python
from contextlib import contextmanager
from typing import Callable

from sqlmodel import SQLModel, create_engine, Session

DATABASE_URL = "sqlite:///./movies.db"
engine = create_engine(
    DATABASE_URL,
    echo=False,
    connect_args={"check_same_thread": False},
)


def init_db(seed_fn: Callable[[Session], None] | None = None) -> None:
    """Create tables and optionally seed starter movies."""
    SQLModel.metadata.create_all(engine)
    if seed_fn:
        with Session(engine) as session:
            seed_fn(session)


@contextmanager
def get_session() -> Session:
    """Provide a session that commits on success and rolls back on error."""
    session = Session(engine)
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()
```

### Models (`app/models.py`)
Create `app/models.py`:
```python
from datetime import datetime
from sqlmodel import Field, SQLModel


class MovieCreate(SQLModel):
    title: str
    year: int
    genre: str


class MovieRead(MovieCreate):
    id: int
    average_rating: float | None = None


class RatingCreate(SQLModel):
    movie_id: int
    user_id: int
    score: int = Field(ge=1, le=5)


class MovieRow(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    title: str
    year: int
    genre: str


class RatingRow(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    movie_id: int = Field(foreign_key="movierow.id")
    user_id: int
    score: int
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

### Repository (`app/repository.py`)
Create `app/repository.py`:
```python
from collections.abc import Sequence
from statistics import mean
from typing import Optional

from sqlmodel import select

from app.db import get_session
from app.models import (
    MovieCreate,
    MovieRead,
    MovieRow,
    RatingCreate,
    RatingRow,
)


def seed_movies(session) -> None:
    if session.exec(select(MovieRow)).first():
        return
    starter_movies = [
        MovieRow(title="Inception", year=2010, genre="Sci-Fi"),
        MovieRow(title="The Matrix", year=1999, genre="Sci-Fi"),
        MovieRow(title="The Godfather", year=1972, genre="Crime"),
        MovieRow(title="Spirited Away", year=2001, genre="Animation"),
        MovieRow(title="Black Panther", year=2018, genre="Action"),
    ]
    session.add_all(starter_movies)
    session.commit()


def list_movies() -> list[MovieRead]:
    with get_session() as session:
        rows = session.exec(select(MovieRow)).all()
        return [MovieRead.model_validate(row) for row in rows]


def get_movie(movie_id: int) -> Optional[MovieRead]:
    with get_session() as session:
        row = session.get(MovieRow, movie_id)
        return MovieRead.model_validate(row) if row else None


def create_movie(payload: MovieCreate) -> MovieRead:
    with get_session() as session:
        row = MovieRow.model_validate(payload)
        session.add(row)
        session.flush()
        return MovieRead.model_validate(row)


def add_rating(payload: RatingCreate) -> None:
    with get_session() as session:
        movie = session.get(MovieRow, payload.movie_id)
        if movie is None:
            raise ValueError("movie not found")
        session.add(RatingRow.model_validate(payload))


def top_movies(limit: int = 5) -> list[MovieRead]:
    with get_session() as session:
        movies = session.exec(select(MovieRow)).all()
        enriched: list[MovieRead] = []
        for movie in movies:
            ratings = session.exec(
                select(RatingRow).where(RatingRow.movie_id == movie.id)
            ).all()
            avg = mean([r.score for r in ratings]) if ratings else None
            enriched.append(
                MovieRead(
                    id=movie.id,
                    title=movie.title,
                    year=movie.year,
                    genre=movie.genre,
                    average_rating=avg,
                )
            )
        enriched.sort(key=lambda m: (m.average_rating or 0), reverse=True)
        return enriched[:limit]


def list_ratings() -> list[dict]:
    with get_session() as session:
        ratings = session.exec(select(RatingRow)).all()
        return [rating.model_dump() for rating in ratings]
```

### Update FastAPI (`app/main.py`)
Replace the in-memory logic with a movie-focused API:
```python
from fastapi import FastAPI, HTTPException

from app.db import init_db
from app import repository
from app.models import MovieCreate, MovieRead, RatingCreate

app = FastAPI(title="Movie Service", version="0.3.0")


@app.on_event("startup")
def startup() -> None:
    init_db(seed_fn=repository.seed_movies)


@app.get("/health")
def health_check() -> dict[str, str]:
    return {"status": "ok"}


@app.get("/movies", response_model=list[MovieRead])
def get_movies() -> list[MovieRead]:
    return repository.list_movies()


@app.get("/movies/{movie_id}", response_model=MovieRead)
def get_movie(movie_id: int) -> MovieRead:
    movie = repository.get_movie(movie_id)
    if movie is None:
        raise HTTPException(status_code=404, detail="Movie not found")
    return movie


@app.post("/movies", response_model=MovieRead, status_code=201)
def create_movie(payload: MovieCreate) -> MovieRead:
    return repository.create_movie(payload)


@app.post("/ratings", status_code=204)
def create_rating(payload: RatingCreate) -> None:
    try:
        repository.add_rating(payload)
    except ValueError as exc:
        raise HTTPException(status_code=404, detail=str(exc)) from exc


@app.get("/movies/top", response_model=list[MovieRead])
def get_top_movies(limit: int = 5) -> list[MovieRead]:
    return repository.top_movies(limit=limit)
```

### Verify Persistence
- Run `uv run uvicorn app.main:app --reload`.
- Visit `http://localhost:8000/movies` to confirm the seeded catalogue.
- Add a rating:
  ```bash
  curl -X POST http://localhost:8000/ratings     -H "Content-Type: application/json"     -d '{"movie_id": 1, "user_id": 42, "score": 5}'
  ```
- Retrieve top movies:
  ```bash
  curl "http://localhost:8000/movies/top?limit=3"
  ```
- Stop the server, restart, and check the data still exists.

## Part C – Hands-on Lab 2 (45 Minutes)
### Test Fixtures (`tests/conftest.py`)
```python
import tempfile
from collections.abc import Iterator

import pytest
from sqlmodel import SQLModel, create_engine, Session

from app import db, repository


@pytest.fixture(autouse=True)
def temporary_db(monkeypatch: pytest.MonkeyPatch) -> Iterator[None]:
    with tempfile.NamedTemporaryFile(suffix=".db") as tmp:
        engine = create_engine(
            f"sqlite:///{tmp.name}",
            connect_args={"check_same_thread": False},
        )
        monkeypatch.setattr(db, "engine", engine)
        SQLModel.metadata.create_all(engine)
        with Session(engine) as session:
            repository.seed_movies(session)
        yield
```

### Expand Tests (`tests/test_movies.py`)
Add assertions for list, recommendations, and error paths. Example snippet:
```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_get_movies_returns_seeded_catalogue():
    response = client.get("/movies")
    assert response.status_code == 200
    body = response.json()
    assert any(movie["title"] == "Inception" for movie in body)


def test_delete_movie_removes_entry():
    created = client.post(
        "/movies",
        json={"title": "Temp", "year": 2021, "genre": "Drama"},
    ).json()
    delete_response = client.delete(f"/movies/{created['id']}")
    assert delete_response.status_code == 204
    assert client.get(f"/movies/{created['id']}").status_code == 404
```

### Run the Suite
```bash
uv run pytest -q
```

### Reflection Prompt
- Ask: “How did database fixtures keep tests isolated?”
- Highlight: The movie repository hides persistence details, so new features slot in without touching route code.
## Exercise 2 Announcement
- **Goal:** Build a user interface (Streamlit or small React app) that talks to this API.
- **Assigned:** Today.
- **Due:** Tuesday, Dec 23, 2025 at 23:59.
- **Deliverables:** UI supporting list/create/update/delete actions, README with run instructions, AI usage notes.
- Encourage students to choose teams and decide between Streamlit or React tonight.
- **AWS reminder:** Storage module completion screenshot should be uploaded by **Tuesday, Dec 9, 2025** to stay ahead of the **Tue Dec 16, 2025** hard deadline.

## Troubleshooting
- If you see `sqlite3.ProgrammingError: SQLite objects created in a thread can only be used in that same thread`, confirm `check_same_thread` is set to `False`.
- Delete `movies.db` when schema changes cause issues (this is safe in development).
- On Windows, make sure the project path does not contain spaces to prevent SQLite file locking problems.

## Student Success Criteria
- Movies and ratings persist across server restarts.
- Tests cover create/read/update/delete plus rating and top-movie logic without leaking state.
- Students can explain why separating models/repository makes the code easier to maintain.
- Students have recorded the **Dec 9 (soft)** and **Dec 16 (hard)** dates for the Storage module and know how to submit proof of completion.

## Quick Reference (External Search / ChatGPT)
- **Google:** `SQLModel FastAPI movie rating example`
- **ChatGPT prompt:** “Outline three talking points to explain why the movie service moved from in-memory storage to SQLModel + SQLite.”

## AI Prompt Kit (Copy/Paste)
- “Refactor the in-memory FastAPI movie service to use SQLModel + SQLite. Create `MovieRow`, `RatingRow`, repository helpers, and seed starter movies on startup.”
- “Write pytest fixtures that replace the SQLite engine with a temp file, seed movies, and test `/movies`, `/ratings`, and `/movies/top`.”
- “Explain when to use `model_validate` vs. `model_dump` with SQLModel/Pydantic v2 using the movie repository as the example.”
