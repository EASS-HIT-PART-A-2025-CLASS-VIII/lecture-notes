# Session 06 – Building a Simple Frontend

- **Date:** Monday, Dec 8, 2025
- **Theme:** Compare two approachable frontend options—Streamlit and minimal React—and connect them to the FastAPI backend.

## Learning Objectives
- Understand how a browser client communicates with the API using HTTP.
- Build a Streamlit interface that lists movies, shows average ratings, and lets students submit new ratings.
- Preview a minimal React app for teams who want more customization while staying aligned with the movie backend.
- Finalize plans for Exercise 2 (frontend project).

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & homework check | 10 min | Discussion | Ask teams if their API persists data successfully |
| Frontend overview | 20 min | Talk + whiteboard | Explain client/server, CORS, form submission |
| Tool comparison | 15 min | Talk | Streamlit vs. React vs. static HTML |
| Lab 1 | 45 min | Guided coding | Build Streamlit movie UI |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | React movie dashboard + CORS middleware |
| EX2 planning circle | 10 min | Group discussion | Teams choose tech stack and next steps |

## Teaching Script – Frontend Basics
1. Draw the architecture: Browser → HTTP → FastAPI → SQLite.
2. Explain: “The browser cannot talk to localhost:8000 by default when we run another UI. We must allow cross-origin requests with CORS middleware.”
3. Define CORS in plain language: “The browser checks if the server allows JavaScript from `http://localhost:3000` (React) or `http://localhost:8501` (Streamlit). We will configure the API to say yes.”
4. Describe Streamlit: “Python, minimal code, great for quick dashboards.”
5. Describe React: “JavaScript, component-based, more setup but more control.”
6. Remind them of Exercise 2 requirements: show list, create, update, delete; due Tuesday, Dec 23, 2025.

## Part B – Hands-on Lab 1 (45 Minutes) – Streamlit Movie Dashboard
### Confirm the API Is Running
Open a terminal in the backend project and keep the API alive:
```bash
uv run uvicorn app.main:app --reload
```

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

st.set_page_config(page_title="Movie Ratings", layout="wide")
st.title("Movie Recommendation Sandbox")

@st.cache_data(ttl=30)
def fetch_movies() -> list[dict]:
    response = httpx.get(f"{API_URL}/movies", timeout=5.0)
    response.raise_for_status()
    return response.json()

movies = []
try:
    movies = fetch_movies()
except httpx.HTTPError as exc:
    st.error(f"Failed to load movies: {exc}")

if movies:
    st.subheader("Catalogue")
    st.dataframe(
        [
            {
                "Title": movie["title"],
                "Year": movie["year"],
                "Genre": movie["genre"],
                "Average Rating": movie.get("average_rating") or "—",
            }
            for movie in movies
        ],
        use_container_width=True,
    )
else:
    st.info("No movies yet. Add one below.")

st.divider()

st.subheader("Add a Movie")
with st.form("add-movie", clear_on_submit=True):
    title = st.text_input("Title")
    year = st.number_input("Year", min_value=1900, max_value=2100, value=2020)
    genre = st.text_input("Genre")
    submit_movie = st.form_submit_button("Create movie")
    if submit_movie:
        try:
            response = httpx.post(
                f"{API_URL}/movies",
                json={"title": title, "year": int(year), "genre": genre},
                timeout=5.0,
            )
            response.raise_for_status()
            st.success("Movie added! Refresh the catalogue using the button below.")
        except httpx.HTTPError as exc:
            st.error(f"Failed to add movie: {exc}")

st.subheader("Rate a Movie")
with st.form("rate-movie", clear_on_submit=True):
    if not movies:
        st.info("Load or create a movie first.")
    else:
        movie_options = {m["title"]: m["id"] for m in movies}
        selected_title = st.selectbox("Movie", list(movie_options.keys()))
        score = st.slider("Score", min_value=1, max_value=5, value=4)
        user_id = st.number_input("Your user id", min_value=1, value=101)
        submit_rating = st.form_submit_button("Submit rating")
        if submit_rating:
            try:
                response = httpx.post(
                    f"{API_URL}/ratings",
                    json={
                        "movie_id": movie_options[selected_title],
                        "user_id": int(user_id),
                        "score": int(score),
                    },
                    timeout=5.0,
                )
                response.raise_for_status()
                st.success("Thanks for rating!")
            except httpx.HTTPError as exc:
                st.error(f"Failed to submit rating: {exc}")

if st.button("Refresh catalogue"):
    fetch_movies.clear()
    st.experimental_rerun()
```

### Run Streamlit
In a second terminal:
```bash
uv run streamlit run ui/app.py
```
Visit `http://localhost:8501` and confirm the catalogue appears. Add a rating and refresh the table.

## Part C – Hands-on Lab 2 (45 Minutes) – React Movie Dashboard
### Create a React Starter (Optional Teams)
```bash
npm create vite@latest movie-ui -- --template react
cd movie-ui
npm install
```

### Minimal React Component (`src/App.jsx`)
```jsx
import { useEffect, useMemo, useState } from "react";

const API_URL = "http://localhost:8000";

async function fetchJSON(url, options) {
  const response = await fetch(url, options);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  return response.json();
}

function App() {
  const [movies, setMovies] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");
  const [newMovie, setNewMovie] = useState({ title: "", year: 2020, genre: "Sci-Fi" });
  const [rating, setRating] = useState({ movieId: null, userId: 101, score: 4 });

  async function loadMovies() {
    try {
      setLoading(true);
      const data = await fetchJSON(`${API_URL}/movies`);
      setMovies(data);
      if (data.length && !rating.movieId) {
        setRating((prev) => ({ ...prev, movieId: data[0].id }));
      }
    } catch (err) {
      setError(`Failed to load movies: ${err.message}`);
    } finally {
      setLoading(false);
    }
  }

  async function handleCreateMovie(event) {
    event.preventDefault();
    await fetchJSON(`${API_URL}/movies`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(newMovie),
    });
    setNewMovie({ title: "", year: 2020, genre: "Drama" });
    loadMovies();
  }

  async function handleSubmitRating(event) {
    event.preventDefault();
    await fetchJSON(`${API_URL}/ratings`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        movie_id: rating.movieId,
        user_id: Number(rating.userId),
        score: Number(rating.score),
      }),
    });
    loadMovies();
  }

  useEffect(() => {
    loadMovies();
  }, []);

  const movieOptions = useMemo(
    () => movies.map((movie) => ({ value: movie.id, label: movie.title })),
    [movies]
  );

  if (loading) {
    return <p>Loading movies...</p>;
  }

  return (
    <main style={{ margin: "2rem" }}>
      <h1>Movie Recommendation Sandbox</h1>
      {error && <p style={{ color: "crimson" }}>{error}</p>}

      <section>
        <h2>Catalogue</h2>
        <ul>
          {movies.map((movie) => (
            <li key={movie.id}>
              {movie.title} ({movie.year}) — {movie.genre}
              {" "}
              {movie.average_rating ? `★ ${movie.average_rating.toFixed(2)}` : "(no ratings yet)"}
            </li>
          ))}
        </ul>
      </section>

      <section>
        <h2>Add a Movie</h2>
        <form onSubmit={handleCreateMovie}>
          <label>
            Title
            <input
              value={newMovie.title}
              onChange={(event) => setNewMovie({ ...newMovie, title: event.target.value })}
            />
          </label>
          <label>
            Year
            <input
              type="number"
              min="1900"
              max="2100"
              value={newMovie.year}
              onChange={(event) => setNewMovie({ ...newMovie, year: Number(event.target.value) })}
            />
          </label>
          <label>
            Genre
            <input
              value={newMovie.genre}
              onChange={(event) => setNewMovie({ ...newMovie, genre: event.target.value })}
            />
          </label>
          <button type="submit">Create movie</button>
        </form>
      </section>

      <section>
        <h2>Rate a Movie</h2>
        <form onSubmit={handleSubmitRating}>
          <label>
            Movie
            <select
              value={rating.movieId ?? ""}
              onChange={(event) => setRating({ ...rating, movieId: Number(event.target.value) })}
            >
              {movieOptions.map((option) => (
                <option key={option.value} value={option.value}>
                  {option.label}
                </option>
              ))}
            </select>
          </label>
          <label>
            Score
            <input
              type="number"
              min="1"
              max="5"
              value={rating.score}
              onChange={(event) => setRating({ ...rating, score: Number(event.target.value) })}
            />
          </label>
          <label>
            User id
            <input
              type="number"
              min="1"
              value={rating.userId}
              onChange={(event) => setRating({ ...rating, userId: Number(event.target.value) })}
            />
          </label>
          <button type="submit">Submit rating</button>
        </form>
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
Visit the shown URL (usually `http://localhost:5173`). Confirm new movies and ratings appear immediately after submission.

### Discussion
- Streamlit is great for quick wins and uses the same language as the backend.
- React offers more control and mirrors industry tooling.
- In EX2, teams should expose movie catalogue, rating submission, and optional leaderboards (e.g., `/movies/top`).

## Exercise 2 Planning Circle (10 Minutes)
- Ask each team to state their chosen frontend stack, division of work, and timeline.
- Encourage teams to outline tasks in their README or project board before leaving.

## Troubleshooting
- If Streamlit shows `ModuleNotFoundError: No module named 'httpx'`, confirm the dependency is listed in `pyproject.toml` and run `uv sync`.
- If React cannot reach the API due to CORS, confirm the allowed origins include `http://localhost:5173` and restart the backend after CORS changes.
- For Streamlit reruns triggered repeatedly, remind students to guard expensive actions with forms/buttons and use `st.cache_data` for read-heavy calls.

## Student Success Criteria
- Streamlit app lists movies, allows adding new titles, and records ratings via `/ratings`.
- React dashboard fetches movies, displays average ratings, and sends POST requests to `/movies` / `/ratings`.
- Every team has a concrete EX2 plan for polishing the movie experience by Tuesday, Dec 23, 2025.

## AI Prompt Kit (Copy/Paste)
- “Generate a Streamlit page that lists movies from `GET /movies`, allows adding new titles, and posts ratings to `POST /ratings`.”
- “Create a React component that shows a movie catalogue with average ratings and includes forms for adding movies and ratings.”
- “Write a CORS middleware snippet for FastAPI that allows `http://localhost:5173` and `http://localhost:8501` so the React/Streamlit movie dashboards can call the API.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `Streamlit FastAPI movie rating tutorial`
- **ChatGPT prompt:** “Explain CORS for students building a React + FastAPI movie recommendation project.”
