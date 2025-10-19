# Session 03 – FastAPI Fundamentals

- **Date:** Monday, Nov 17, 2025
- **Theme:** Build the first REST API with FastAPI and validate it using automated tests.

## Learning Objectives
- Create typed request and response models with `pydantic`.
- Map HTTP routes to Python functions in FastAPI.
- Test endpoints using `fastapi.testclient.TestClient`.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & motivation | 10 min | Talk | Why we need our own server instead of relying on httpbin |
| FastAPI walkthrough | 20 min | Talk + live coding | Anatomy of a FastAPI app |
| Testing mindset | 15 min | Talk + discussion | Benefits of tests, how they guide development |
| Lab 1 | 45 min | Guided coding | Build the API |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Create tests and run them |
| Q&A on EX1 | 10 min | Discussion | Clarify remaining questions about the assignment |

## Teaching Script – Why FastAPI?
1. “Last time we acted as API clients. Today the class becomes the API provider.”
2. “FastAPI gives us a quick way to describe endpoints with Python functions and uses `pydantic` so bad inputs are rejected automatically.”
3. “We practice Test-Driven Development lite: write an endpoint, then confirm it with a test.”
4. Write on the board what EX1 requires: `POST /items`, `GET /items/{id}`, `GET /items`, `PUT`, `DELETE`, validation, tests, Dockerfile.

## Part B – Hands-on Lab 1 (45 Minutes)
### Setup Commands
```bash
cd hello-uv
uv add fastapi uvicorn
mkdir -p app
touch app/__init__.py app/main.py
```

### Live Coding Script
Explain each block while typing:
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI(title="Items Service", version="0.1.0")


class Item(BaseModel):
    id: int | None = None
    name: str = Field(..., min_length=1, max_length=30)
    quantity: int = Field(..., ge=0, le=100)


ITEMS: dict[int, Item] = {}
NEXT_ID = 1


@app.get("/health")
def health_check() -> dict[str, str]:
    return {"status": "ok"}


@app.post("/items", response_model=Item)
def create_item(item: Item) -> Item:
    global NEXT_ID
    item.id = NEXT_ID
    NEXT_ID += 1
    ITEMS[item.id] = item
    return item


@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int) -> Item:
    if item_id not in ITEMS:
        raise HTTPException(status_code=404, detail="Item not found")
    return ITEMS[item_id]
```

### Run the Server
```bash
uv run uvicorn app.main:app --reload
```
Visit `http://localhost:8000/docs` in a browser and show the automatic interactive documentation. Demonstrate a POST request and inspect the response.

### Student Task Card
- Start the server locally.
- Use `curl` or the API docs to create an item:
  ```bash
  curl -X POST http://localhost:8000/items -H "Content-Type: application/json" -d '{"name": "Pen", "quantity": 3}'
  ```
- Fetch the item by ID and confirm the JSON matches.

### Optional Practice Routes (Copy/Paste)
Add two simple endpoints to practice path and query parameters:
```python
@app.get("/echo/{text}")
def echo(text: str) -> dict[str, str]:
    return {"echo": text}


@app.get("/add")
def add(a: int, b: int) -> dict[str, int]:
    return {"result": a + b}
```
Try:
```bash
curl "http://localhost:8000/echo/hello"
curl "http://localhost:8000/add?a=5&b=7"
```

## Part C – Hands-on Lab 2 (45 Minutes)
### Create the Test Suite
1. Install testing helpers if not already present:
   ```bash
   uv add pytest
   ```
2. Create `tests/test_items.py`:
   ```python
   from fastapi.testclient import TestClient
   from app.main import app

   client = TestClient(app)


   def test_health_check():
       response = client.get("/health")
       assert response.status_code == 200
       assert response.json() == {"status": "ok"}


   def test_create_and_fetch_item():
       create_response = client.post(
           "/items",
           json={"name": "Marker", "quantity": 2},
       )
       assert create_response.status_code == 200
       payload = create_response.json()
       assert payload["name"] == "Marker"
       assert payload["quantity"] == 2

       item_id = payload["id"]
       fetch_response = client.get(f"/items/{item_id}")
       assert fetch_response.status_code == 200
       assert fetch_response.json()["id"] == item_id


   def test_missing_item_returns_404():
       response = client.get("/items/999")
       assert response.status_code == 404
       assert response.json()["detail"] == "Item not found"
   ```
3. Run:
   ```bash
   uv run pytest -q
   ```
4. Encourage students to add a validation test:
   ```python
   def test_create_rejects_empty_name():
       response = client.post("/items", json={"name": "", "quantity": 1})
       assert response.status_code == 422
   ```

## AI Prompt Kit (Copy/Paste)
- “Write a FastAPI endpoint that accepts path and query parameters and returns validated JSON. Include two example curl commands.”
- “Generate pytest tests for FastAPI endpoints /echo/{text} and /add?a&b, covering happy and error paths.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI TestClient example`
- **ChatGPT prompt:** “Explain, in three bullet points, how to organize FastAPI tests that mix pure function tests with TestClient integration checks.”

### Reinforce Testing Mindset
- Green tests mean the behavior matches expectations.
- Show how a failing test pinpoints the regression.

## EX1 Reminder
- Today’s code is the starting point.
- Homework: add `GET /items` listing endpoint.
- Suggested next steps: implement `PUT` and `DELETE`, write tests for each.

## Troubleshooting Notes
- If `uvicorn` cannot import `app.main`, ensure `app/__init__.py` exists.
- If the server port is in use, stop previous runs or change to `--port 8001`.
- For validation errors, inspect the 422 response to see details about the failing field.

## Student Success Criteria
- Server returns 200 from `/health`.
- Students can create and read back an item.
- Tests run green and include at least one failure case (404 or validation).
