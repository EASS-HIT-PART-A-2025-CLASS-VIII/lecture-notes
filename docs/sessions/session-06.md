# Session 06 – Building a Simple Frontend

- **Date:** Monday, Dec 8, 2025
- **Theme:** Compare two approachable frontend options—Streamlit and minimal React—and connect them to the FastAPI backend.

## Learning Objectives
- Understand how a browser client communicates with the API using HTTP.
- Build a Streamlit interface that lists and creates items.
- Preview a minimal React app for teams who want more customization.
- Finalize plans for Exercise 2 (frontend project).

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & homework check | 10 min | Discussion | Ask teams if their API persists data successfully |
| Frontend overview | 20 min | Talk + whiteboard | Explain client/server, CORS, form submission |
| Tool comparison | 15 min | Talk | Streamlit vs. React vs. static HTML |
| Lab 1 | 45 min | Guided coding | Build Streamlit UI |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | React teaser + CORS middleware |
| EX2 planning circle | 10 min | Group discussion | Teams choose tech stack and next steps |

## Teaching Script – Frontend Basics
1. Draw the architecture: Browser → HTTP → FastAPI → SQLite.
2. Explain: “The browser cannot talk to localhost:8000 by default when we run another UI. We must allow cross-origin requests with CORS middleware.”
3. Define CORS in plain language: “The browser checks if the server allows JavaScript from `http://localhost:3000` (React) or `http://localhost:8501` (Streamlit). We will configure the API to say yes.”
4. Describe Streamlit: “Python, minimal code, great for quick dashboards.”
5. Describe React: “JavaScript, component-based, more setup but more control.”
6. Remind them of Exercise 2 requirements: show list, create, update, delete; due Tuesday, Dec 23, 2025.

## Part B – Hands-on Lab 1 (45 Minutes) – Streamlit Path
### Configure CORS in FastAPI
In `app/main.py`, add before the routes:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost",
        "http://localhost:3000",
        "http://localhost:5173",  # Vite dev server
        "http://localhost:8501",  # Streamlit
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Install Streamlit and httpx
```bash
uv add streamlit httpx
mkdir -p ui
```

### Create Streamlit App (`ui/app.py`)
```python
import httpx
import streamlit as st

API_URL = "http://localhost:8000"

st.set_page_config(page_title="Items Dashboard", layout="centered")
st.title("Items Dashboard")

with st.form("create-item", clear_on_submit=True):
    name = st.text_input("Item name", help="Enter a short name")
    quantity = st.number_input("Quantity", min_value=0, max_value=100, value=1)
    submitted = st.form_submit_button("Create item")
    if submitted:
        response = httpx.post(
            f"{API_URL}/items",
            json={"name": name, "quantity": quantity},
            timeout=5.0,
        )
        if response.status_code == 201:
            st.success("Item created!")
        else:
            st.error(f"Failed to create item: {response.text}")

st.subheader("Current items")
items_response = httpx.get(f"{API_URL}/items", timeout=5.0)
if items_response.status_code == 200:
    items = items_response.json()
    if items:
        for item in items:
            st.write(f"• {item['name']} — {item['quantity']}")
    else:
        st.info("No items yet. Add one above!")
else:
    st.error("Could not load items from the API.")
```

### Run Streamlit
Open two terminals:
1. `uv run uvicorn app.main:app --reload`
2. `uv run streamlit run ui/app.py`

Visit `http://localhost:8501` and confirm items appear and persist.

## Part C – Hands-on Lab 2 (45 Minutes) – React Teaser
### Create a React Starter (Optional Teams)
```bash
npm create vite@latest items-ui -- --template react
cd items-ui
npm install
```

### Minimal React Component (`src/App.jsx`)
```jsx
import { useEffect, useState } from "react";

const API_URL = "http://localhost:8000";

function App() {
  const [items, setItems] = useState([]);
  const [name, setName] = useState("");
  const [quantity, setQuantity] = useState(1);

  async function loadItems() {
    const response = await fetch(`${API_URL}/items`);
    if (!response.ok) {
      console.error("Failed to load items");
      return;
    }
    const data = await response.json();
    setItems(data);
  }

  async function createItem(event) {
    event.preventDefault();
    const response = await fetch(`${API_URL}/items`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name, quantity: Number(quantity) }),
    });
    if (response.ok) {
      setName("");
      setQuantity(1);
      loadItems();
    }
  }

  useEffect(() => {
    loadItems();
  }, []);

  return (
    <main style={{ margin: "2rem" }}>
      <h1>Items Dashboard</h1>
      <form onSubmit={createItem}>
        <label>
          Name
          <input value={name} onChange={(event) => setName(event.target.value)} />
        </label>
        <label>
          Quantity
          <input
            type="number"
            min="0"
            max="100"
            value={quantity}
            onChange={(event) => setQuantity(event.target.value)}
          />
        </label>
        <button type="submit">Create item</button>
      </form>
      <section>
        <h2>Items</h2>
        <ul>
          {items.map((item) => (
            <li key={item.id}>
              {item.name} — {item.quantity}
            </li>
          ))}
        </ul>
      </section>
    </main>
  );
}

export default App;
```

### Run React Dev Server
```bash
npm run dev
```
Visit the shown URL (usually `http://localhost:5173`). Confirm CORS works because of the middleware we added earlier.

### Discussion
- Streamlit is great for quick start and requires only Python.
- React gives more control, styling options, and is closer to industry tooling.
- Regardless of choice, each team must support list/create/update/delete for Exercise 2.

## Exercise 2 Planning Circle (10 Minutes)
- Ask each team to state their chosen frontend stack, division of work, and timeline.
- Encourage teams to outline tasks in their README or project board before leaving.

## Troubleshooting
- If Streamlit shows `ModuleNotFoundError: No module named 'httpx'`, confirm the dependency is listed in `pyproject.toml` and run `uv sync`.
- If React cannot reach the API due to CORS, confirm the allowed origins include the dev server (`http://localhost:5173`).
- For Streamlit reruns triggered repeatedly, remind students to guard expensive actions with buttons or forms.

## Student Success Criteria
- Streamlit app displays items and can create new ones.
- Teams choosing React can fetch items and display them in the browser.
- Every team has committed to a plan for Exercise 2, due Tuesday, Dec 23, 2025.

## AI Prompt Kit (Copy/Paste)
- “Generate a Streamlit page that POSTs to `http://localhost:8000/items` with fields `name` (str) and `quantity` (int), shows success/error banners, and lists current items.”
- “Create a minimal React component that fetches `GET /items` from `http://localhost:8000` on mount and renders a list with keys. Add a controlled form to POST a new item.”
- “Write a CORS middleware snippet for FastAPI that allows origins `http://localhost:5173` and `http://localhost:8501` with all methods/headers allowed.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `Streamlit fastapi example items`
- **ChatGPT prompt:** “Explain in plain English how CORS works and why our FastAPI backend must allow localhost:5173 and localhost:8501.”
