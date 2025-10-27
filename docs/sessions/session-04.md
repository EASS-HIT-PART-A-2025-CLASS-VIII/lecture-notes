# Session 04 – Docker Basics and Reverse Proxy Demo

- **Date:** Monday, Nov 24, 2025
- **Theme:** Package the FastAPI service into a portable Docker image and understand how reverse proxies route traffic.

## Learning Objectives
- Explain the difference between Docker images and containers.
- Write a Dockerfile that uses uv to install dependencies.
- Run FastAPI inside a container and reach it through an `nginx` reverse proxy.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Recap & motivation | 10 min | Talk | Why “works on my machine” is not enough |
| Docker concepts | 20 min | Talk + sketches | Layers, images, containers, ports |
| Reverse proxy concept | 15 min | Talk | How `nginx` forwards requests |
| Lab 1 | 45 min | Guided coding | Build and run the Docker image |
| Break | 10 min | — | Launch [10-minute timer](https://e.ggtimer.com/10minutes) and reset |
| Lab 2 | 45 min | Guided coding | Wire `nginx` to the app |
| EX1 help clinic | 10 min | Q&A | Answer remaining assignment questions |

## Teaching Script – Docker Primer
1. Draw a stack: Hardware → Host OS → Docker Engine → Containers. Explain: “A container is a lightweight box with everything the app needs.”
2. Introduce key terms:
   - **Dockerfile:** recipe describing how to build the image.
   - **Image:** snapshot of filesystem + config.
   - **Container:** running instance of an image.
3. Emphasize ports: “FastAPI listens on port 8000. We map it to port 8000 on our laptop so the browser can reach it.”
4. Preview reverse proxy: “nginx will sit in front, listening on port 8080, and forward traffic to FastAPI. This is common in production.”

## Part B – Hands-on Lab 1 (45 Minutes)
### Create the Dockerfile
In the project root (`hello-uv`), create `Dockerfile` with the content below and explain each instruction:
```dockerfile
FROM python:3.12-slim

RUN apt-get update \
    && apt-get install -y curl \
    && curl -LsSf https://astral.sh/uv/install.sh | sh \
    && rm -rf /var/lib/apt/lists/*

ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-install-project

COPY . .
RUN uv sync --no-dev

EXPOSE 8000

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Build and Run
```bash
docker build -t movies-api .
docker run --rm -p 8000:8000 movies-api
```
Test with:
```bash
curl http://localhost:8000/health
```
Stop the container with `Ctrl+C` when finished.

### Discussion Points
- Show `docker image ls` to confirm the image exists.
- Explain the benefit: any teammate can run `docker run movies-api` and get the same behavior.

---

## Break (10 Minutes)
Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes), stretch, and prepare for Part C.

---

## Part C – Hands-on Lab 2 (45 Minutes)
### Start nginx
1. Create a network so containers can talk:
   ```bash
   docker network create movies-net
   ```
2. Launch the API on that network:
   ```bash
   docker run -d --name movies-api --network movies-net movies-api
   ```
3. Create `ops/nginx.conf` with:
   ```nginx
   server {
       listen 80;

       location / {
           proxy_pass http://movies-api:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```
4. Run nginx on the same network:
   ```bash
   docker run -d --name movies-proxy --network movies-net \
     -p 8080:80 \
     -v "$(pwd)/ops/nginx.conf:/etc/nginx/conf.d/default.conf:ro" \
     nginx:alpine
   ```
5. Test:
   ```bash
   curl http://localhost:8080/health
   ```
   Expect `{"status":"ok"}`.

### Cleanup
```bash
docker rm -f movies-api movies-proxy
docker network rm movies-net
```

## EX1 Help Clinic
- Remind students EX1 is due Tuesday, Dec 2 at 23:59.
- Checklist for completion:
  - CRUD endpoints implemented.
  - Tests covering happy path and at least one error path.
  - Dockerfile similar to today’s example.
  - README with setup instructions and AI usage notes.

## Troubleshooting Notes
- If `curl` cannot reach the container, check that Docker Desktop is running.
- Permission denied when mounting volume on Windows? Use PowerShell and ensure the repo path is shared with Docker Desktop.
- If port 8000 or 8080 already in use, pick alternatives (e.g., `-p 8001:8000`).

## Student Success Criteria
- Docker image builds without error.
- `/health` endpoint accessible from both port 8000 (direct container) and 8080 (`nginx` proxy).
- Students can explain the role of the reverse proxy and why it is useful.

## AI Prompt Kit (Copy/Paste)
- “Write a minimal, production-friendly Dockerfile for a FastAPI app using uv. It should install uv, copy `pyproject.toml` and lockfile first for caching, then copy sources, expose 8000, and run uvicorn. Avoid unnecessary layers.”
- “Provide an `nginx.conf` that reverse proxies all paths `/` to http://api:8000 with correct `proxy_set_header` values and no buffering. Explain where to mount it in a container.”
- “Explain `docker run -p host:container` vs. `EXPOSE` in plain English for a beginner and show two example commands for mapping 8000 and 8080.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI Dockerfile uv example`
- **ChatGPT prompt:** “List three reasons we put nginx in front of FastAPI in a classroom Docker demo.”
