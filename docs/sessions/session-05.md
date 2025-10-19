# Session 05 – Adding Persistence with SQLite

- **Date:** Monday, Dec 1, 2025
- **Theme:** Replace the in-memory dictionary with a lightweight SQLite database and prepare for user interfaces.

## Learning Objectives
- Describe why persistent storage is required in most applications.
- Integrate SQLModel with FastAPI to store data in SQLite.
- Adjust automated tests to work with a database-backed repository.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm start & EX1 mini demos | 15 min | Student lightning talks | Share working CRUD endpoints and lessons learned |
| Persistence primer | 20 min | Talk + sketching | Tables, rows, primary keys, when to migrate |
| Architecture mini-lesson | 10 min | Talk | Separating API layer and data layer |
| AWS module check-in | 5 min | Discussion | Confirm Compute module submissions and announce Storage module timeline |
| Lab 1 | 45 min | Guided coding | Add SQLModel and wire FastAPI to SQLite |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Update tests and cover new CRUD paths |
| EX2 announcement | 10 min | Talk | Explain the upcoming frontend assignment |

## Teaching Script – Why Persistence Matters
1. Ask: “What happens to our `ITEMS` dictionary if we restart the server?” Let students respond.
2. Summarize: “In-memory data disappears. Users expect their work to stay. We need storage on disk.”
3. Introduce SQLite as “a single file database that ships with Python—perfect for class projects.”
4. Explain SQLModel: “It combines SQLAlchemy and pydantic to give us typed models and automatic table creation.”
5. Draw a mini diagram on the board:
   - Request → FastAPI route → Repository → Database.
6. Set expectations: “By the end of today, your API will persist items across restarts and your tests will run against a temporary database.”
7. Announce the AWS Academy **Storage** module plan: “Start the Storage module this week so it’s finished by **Tuesday, Dec 9, 2025**. The hard cutoff for all AWS modules is **Tuesday, Dec 16, 2025**—upload your completion screenshots to Canvas just like you did for Compute.”

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
from sqlmodel import SQLModel, create_engine, Session

DATABASE_URL = "sqlite:///./items.db"
engine = create_engine(
    DATABASE_URL,
    echo=False,
    connect_args={"check_same_thread": False},
)


def init_db() -> None:
    """Create tables if they do not already exist."""
    SQLModel.metadata.create_all(engine)


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
from sqlmodel import Field, SQLModel


class ItemCreate(SQLModel):
    name: str
    quantity: int


class ItemUpdate(SQLModel):
    name: str | None = None
    quantity: int | None = None


class ItemRead(ItemCreate):
    id: int


class ItemRow(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str
    quantity: int
```

### Repository (`app/repository.py`)
Create `app/repository.py`:
```python
from collections.abc import Sequence
from sqlmodel import select

from app.db import get_session
from app.models import ItemCreate, ItemRead, ItemRow, ItemUpdate


def create_item(payload: ItemCreate) -> ItemRead:
    with get_session() as session:
        row = ItemRow.model_validate(payload)
        session.add(row)
        session.flush()  # populates row.id
        return ItemRead.model_validate(row)


def list_items() -> Sequence[ItemRead]:
    with get_session() as session:
        rows = session.exec(select(ItemRow)).all()
        return [ItemRead.model_validate(row) for row in rows]


def get_item(item_id: int) -> ItemRead | None:
    with get_session() as session:
        row = session.get(ItemRow, item_id)
        return ItemRead.model_validate(row) if row else None


def update_item(item_id: int, payload: ItemUpdate) -> ItemRead | None:
    with get_session() as session:
        row = session.get(ItemRow, item_id)
        if not row:
            return None
        if payload.name is not None:
            row.name = payload.name
        if payload.quantity is not None:
            row.quantity = payload.quantity
        session.add(row)
        session.flush()
        return ItemRead.model_validate(row)


def delete_item(item_id: int) -> bool:
    with get_session() as session:
        row = session.get(ItemRow, item_id)
        if not row:
            return False
        session.delete(row)
        session.flush()
        return True
```

### Update FastAPI (`app/main.py`)
Replace the existing in-memory logic with:
```python
from fastapi import FastAPI, HTTPException

from app.db import init_db
from app import repository
from app.models import ItemCreate, ItemRead, ItemUpdate

app = FastAPI(title="Items Service", version="0.2.0")


@app.on_event("startup")
def startup() -> None:
    init_db()


@app.get("/health")
def health_check() -> dict[str, str]:
    return {"status": "ok"}


@app.post("/items", response_model=ItemRead, status_code=201)
def create_item(payload: ItemCreate) -> ItemRead:
    return repository.create_item(payload)


@app.get("/items", response_model=list[ItemRead])
def list_items() -> list[ItemRead]:
    return list(repository.list_items())


@app.get("/items/{item_id}", response_model=ItemRead)
def read_item(item_id: int) -> ItemRead:
    item = repository.get_item(item_id)
    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return item


@app.put("/items/{item_id}", response_model=ItemRead)
def replace_item(item_id: int, payload: ItemCreate) -> ItemRead:
    updated = repository.update_item(item_id, ItemUpdate(**payload.model_dump()))
    if updated is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return updated


@app.patch("/items/{item_id}", response_model=ItemRead)
def patch_item(item_id: int, payload: ItemUpdate) -> ItemRead:
    updated = repository.update_item(item_id, payload)
    if updated is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return updated


@app.delete("/items/{item_id}", status_code=204)
def remove_item(item_id: int) -> None:
    deleted = repository.delete_item(item_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="Item not found")
```

### Verify Persistence
- Run `uv run uvicorn app.main:app --reload`.
- Create an item, stop the server, start again, and fetch the item. It should still exist.

## Part C – Hands-on Lab 2 (45 Minutes)
### Test Fixtures (`tests/conftest.py`)
Add:
```python
import tempfile
from collections.abc import Iterator

import pytest
from sqlmodel import SQLModel, create_engine, Session

from app import db


@pytest.fixture(autouse=True)
def temporary_db(monkeypatch: pytest.MonkeyPatch) -> Iterator[None]:
    with tempfile.NamedTemporaryFile(suffix=".db") as tmp:
        engine = create_engine(
            f"sqlite:///{tmp.name}",
            connect_args={"check_same_thread": False},
        )
        db.engine = engine
        SQLModel.metadata.create_all(engine)
        yield
```

### Expand Tests (`tests/test_items.py`)
Add assertions for list, update, delete, and error paths. Example snippet:
```python
def test_update_item():
    create_response = client.post("/items", json={"name": "Pencil", "quantity": 1})
    item_id = create_response.json()["id"]

    update_response = client.put(
        f"/items/{item_id}",
        json={"name": "Pencil", "quantity": 5},
    )
    assert update_response.status_code == 200
    assert update_response.json()["quantity"] == 5
```

### Run the Suite
```bash
uv run pytest -q
```

### Reflection Prompt
- Ask: “What code had to change when we introduced the database?”
- Highlight: Routes stayed simple because the repository hides the database details.

## Exercise 2 Announcement
- **Goal:** Build a user interface (Streamlit or small React app) that talks to this API.
- **Assigned:** Today.
- **Due:** Tuesday, Dec 23, 2025 at 23:59.
- **Deliverables:** UI supporting list/create/update/delete actions, README with run instructions, AI usage notes.
- Encourage students to choose teams and decide between Streamlit or React tonight.
- **AWS reminder:** Storage module completion screenshot should be uploaded by **Tuesday, Dec 9, 2025** to stay ahead of the **Tue Dec 16, 2025** hard deadline.

## Troubleshooting
- If you see `sqlite3.ProgrammingError: SQLite objects created in a thread can only be used in that same thread`, confirm `check_same_thread` is set to `False`.
- Delete `items.db` when schema changes cause issues (this is safe in development).
- On Windows, make sure the project path does not contain spaces to prevent SQLite file locking problems.

## Student Success Criteria
- Items persist across server restarts.
- Tests cover create, read, update, delete, and failure paths.
- Students can explain why separating models/repository makes the code easier to maintain.
- Students have recorded the **Dec 9 (soft)** and **Dec 16 (hard)** dates for the Storage module and know how to submit proof of completion.

## Quick Reference (External Search / ChatGPT)
- **Google:** `SQLModel FastAPI example sqlite`
- **ChatGPT prompt:** “Outline three talking points to explain why we moved from in-memory storage to SQLModel + SQLite in FastAPI.”

## AI Prompt Kit (Copy/Paste)
- “Refactor an in-memory FastAPI items service to use SQLModel + SQLite. Create `ItemRow`, `ItemCreate`, `ItemRead`, `ItemUpdate`, a repository with create/get/list/update/delete, and wire routes. Include a startup hook to create tables.”
- “Write pytest fixtures that replace the production SQLite engine with a temporary file DB and auto-create tables. Provide example tests for PUT, PATCH, DELETE.”
- “Explain when to use `model_validate` vs. `model_dump` with SQLModel and Pydantic v2 in one concise paragraph and give one example.”
