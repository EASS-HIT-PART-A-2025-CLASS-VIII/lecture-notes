# Exercises Overview

Every project is designed for undergraduates who are new to full-stack development. Each assignment is introduced during class and always due on a Tuesday to avoid weekend crunch.

**Single monorepo + EX3 free cloud deployment:** Students maintain one Git repository for **EX1**, **EX2**, and **EX3** (keep each exercise in clearly labeled folders or services). Only the **EX3** deliverable must go online: deploy the entire Docker Compose stack (backend, frontend, nginx/Redis, and any extra services) on a single Azure for Students compute instance using the \$100 credit (no credit card required). Monitor usage closely so you never spend beyond the grant, and budget time to record a ≤3 min demo movie showing the live deployment for graders.

## EX1 – Backend Foundations
- **Assigned:** Monday, Nov 10, 2025
- **Due:** Tuesday, Dec 2, 2025 at 23:59 (Israel time)
- **Goal:** Build a FastAPI movie service that supports create/read/update/delete operations and records ratings.
- **Sessions to revisit:** [Session 02 – Introduction to HTTP and REST](sessions/session-02.md), [Session 03 – FastAPI Fundamentals](sessions/session-03.md), [Session 04 – Docker Basics and Reverse Proxy Demo](sessions/session-04.md).
- **Required Features:**
  - Endpoints: `POST /movies`, `GET /movies`, `GET /movies/{id}`, `PUT /movies/{id}`, `DELETE /movies/{id}`, plus `POST /ratings` for new scores.
  - Validation via `pydantic` models (title/year/genre, rating score 1–5).
  - Automated tests using pytest and FastAPI’s `TestClient`.
  - Dockerfile using uv to install dependencies.
  - README documenting setup, run commands, and any AI assistance.
- **Rubric (100 pts):** correctness 35, validation/errors 15, automated tests 20, Dockerization 15, documentation/code style 15.

## EX2 – Frontend Integration
- **Assigned:** Monday, Dec 1, 2025
- **Due:** Tuesday, Dec 23, 2025 at 23:59 (Israel time)
- **Goal:** Create a movie dashboard UI that interacts with the EX1 API (list catalogue, submit ratings).
- **Sessions to revisit:** [Session 05 – Movie Service Persistence with SQLite](sessions/session-05.md), [Session 06 – Movie Dashboards with Streamlit & React](sessions/session-06.md), [Session 07 – Testing, Logging, and Profiling Basics](sessions/session-07.md).
- **Choices:**
  - Streamlit (fast path, Python-based).
  - Minimal React app (JavaScript-based) using Vite.
- **Required Features:**
  - List existing movies (title/year/genre/average rating) from the API.
  - Create new movies and submit ratings.
  - Update and delete movies (via controls) and show top movies.
  - Handle error states (failed fetch, validation errors, unauthorised requests).
  - README with setup steps, including how to run both the API and the UI.
- **Optional Stretch (+10 pts):** Integrate the UI into Docker Compose so one command starts both services.
- **Rubric (100 pts):** feature completeness 35, user experience 15, error handling 15, code quality/organization 15, documentation 10, optional Compose integration +10 bonus.

## EX3 – Advanced Backend + Compose
- **Assigned:** Monday, Jan 5, 2026
- **Milestone Demo:** Tuesday, Jan 20, 2026 (in-class show-and-tell)
- **Final Due:** Tuesday, Feb 10, 2026 at 23:59 (Israel time)
- **Goal:** Deliver a multi-service stack using Docker Compose and one advanced capability.
- **Sessions to revisit:** [Session 08 – Working with AI Coding Assistants](sessions/session-08.md), [Session 09 – Async Recommendation Refresh](sessions/session-09.md), [Session 10 – Docker Compose, Redis, and Service Contracts](sessions/session-10.md), [Session 11 – Security Foundations](sessions/session-11.md), [Session 12 – Tool-Friendly APIs and Final Prep](sessions/session-12.md).
- **Required Services:** at least the FastAPI API and an nginx reverse proxy. Optional additional services include databases, frontends, or monitoring tools.
- **Advanced Feature Choices (pick one):**
  1. Asynchronous recommendation rebuild job (Session 09).
  2. Authentication/authorization (JWT-based, roles, protected routes).
  3. Observability (metrics endpoint, structured logs, or tracing).
- **Deliverables:**
  - Working Compose stack (`docker compose up --build` runs without errors) with API, Redis cache, and nginx proxy.
  - Automated tests covering the advanced feature.
  - README including architecture diagram or description, environment variables, deployment URL/steps, Azure credit tracking notes, and AI usage documentation.
  - Cloud deployment of the Compose stack on the Azure for Students free tier. Run everything on a single compute instance, stay within the \$100 grant, and follow the [Azure Container Apps Deployment Playbook](../deployments/azure-container-apps.md); document the exact commands, resource names, public URLs, credit usage, and teardown steps you used.
  - Verification evidence: demonstrate the stack works locally **and** in Azure. Record a short unlisted YouTube demo movie (≤3 min) of the live cloud deployment in action, include the link in the README, and capture logs or screenshots from a successful local `docker compose up --build`.
  - Optional: expose a tool-friendly endpoint and demonstrate calling it from a script or local LLM (Session 08 covers LM Studio + vLLM setup).
- **Rubric (100 pts):** multi-service architecture 25, advanced feature depth 30, reliability/resilience 15, tests 15, documentation 15.

> 🔁 **Demo checklist:** run through the playbook end-to-end the week before submission so the grading team can access your site. Bring screenshots of the Azure Portal resource group, the `docker-compose.release.yml`, the Cost Management credit balance, a successful browser session pointed at your live URL, and have your unlisted YouTube demo link ready.


## AWS Academy Cloud Foundations Modules
- **Platform:** [AWS Academy](https://www.awsacademy.com/) – course: *AWS Academy Cloud Foundations*.
- **Enrollment Link:** https://awsacademy.instructure.com/users/573468/external_tools/4718578
- **Required Modules:** Compute, Storage, and Databases.
- **Hard deadline for all three modules:** **Tuesday, Dec 16, 2025 at 23:59 (Israel time).** (Mid-December Tuesday.)
- **Suggested pacing:**
  1. Finish **Compute** by Tue Nov 25, 2025 so you can focus on coding exercises.
  2. Finish **Storage** by Tue Dec 9, 2025.
  3. Finish **Databases** no later than Tue Dec 16, 2025 (hard cutoff for all modules).
- **How to submit:** Download or screenshot each module’s completion certificate and upload it to the matching Canvas assignment (`AWS Module – Compute`, `AWS Module – Storage`, `AWS Module – Databases`).
- **Support:** Direct all questions to the Discord `#helpdesk` channel (invite: https://discord.gg/EYjQrSmF7f) or bring them to office hours—this is the only Discord support channel for the course.

## Submission Guidelines
1. Push code to the designated GitHub Classroom repository (single monorepo for EX1–EX3).
2. Tag releases if requested by the instructor (e.g., `ex1-final`).
3. Include an “AI Assistance” section in each README describing tools used and how code was verified locally.
4. For EX3, provide deployment details (URL, credentials if needed, credit usage snapshot, teardown steps) for the Azure environment and confirm it runs entirely within the \$100 student credit.
5. Late policy: 48-hour grace period with 10% deduction; proactive communication encouraged.

## Support Channels
- Discord `#helpdesk` channel for quick questions (invite: https://discord.gg/EYjQrSmF7f).
- Weekly office hours (posted in class).
- Peer study sessions after select lectures (announced in Discord).
