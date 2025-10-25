# EASS – Engineering of Advanced Software Solutions (Course Materials)

This repository contains the fully scripted 12-session plan for the **EASS 8 – Engineering of Advanced Software Solutions** course. Each class blends 45 minutes of theory with two 45-minute hands-on blocks, and the entire arc follows a single project: building a movie recommendation platform with FastAPI, SQLModel, Streamlit/React, Redis, and Docker.

## 🚀 Quick Start for Instructors

```bash
git clone https://github.com/EASS-HIT-PART-A-2025-CLASS-VIII/lecture-notes.git
cd lecture-notes
```

Open the `docs/` folder (or load the repo in VS Code) to follow any session directly—no static site build is required.

Key documents:

- `docs/index.md` – entry point with links to every session and deadline summary.
- `docs/exercises.md` – specifications and rubrics for EX1–EX3 plus AWS Academy requirements.
- `docs/sessions/session-XX.md` – detailed talk tracks, copy/paste code, AI prompt kits, troubleshooting, and verification commands for each class.
- `docs/troubleshooting.md` – quick fixes for common environment issues (uv, imports, Redis, etc.).
- `examples.http` – ready-to-run VS Code REST Client requests for the movie API.

## 🧠 Course Highlights

- Sessions 01–04: developer environment, HTTP/REST, FastAPI fundamentals, Docker + nginx.
- Session 05 onward: movie service persistence, Streamlit & React frontends, testing/logging, AI-assisted coding, async recommendation rebuilds, Docker Compose with Redis, JWT security, and tool-friendly APIs.
- Exercises pace with the storyline:
  - **EX1** (due Tue 2 Dec 2025, 23:59 Israel time): ship the FastAPI movie backend with tests and Docker.
  - **EX2** (due Tue 23 Dec 2025, 23:59 Israel time): deliver a Streamlit or React movie dashboard.
  - **EX3** (assigned Mon 5 Jan 2026, final due Tue 10 Feb 2026, 23:59 Israel time): compose the full stack (API + Redis + nginx) with an advanced feature (async rebuild, auth, or observability).
- AWS Academy Cloud Foundations modules (Compute, Storage, Databases) must all be submitted by **Tue 16 Dec 2025, 23:59 Israel time** (soft pacing: Compute by Nov 25, Storage by Dec 9).

## 🗂️ Legacy Materials

Historical slides and Natalie’s notes live under `lectures/`:

- `lectures/archive/` – previous slide decks and Makefile.
- `lectures/notes/` – Natalie’s comprehensive PDF reference.

These are preserved for reference but the new scripted sessions in `docs/` are the canonical teaching materials.

## 🤝 Contributing / Updating

1. Edit the relevant `docs/sessions/session-XX.md` file (each is standalone and self-contained).
2. Run through the verification commands provided in that session (most require `uv run pytest -q` or `curl` checks).
3. Commit changes and push to `main` (the repository is intentionally kept current for instructors).

If you spot an issue or want to suggest an improvement, open a GitHub issue or pull request with the session number in the title (e.g., `Session 05 – clarify rating fixture`).

Have a great semester!
