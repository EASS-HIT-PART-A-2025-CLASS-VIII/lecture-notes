# Session 01 – Kickoff and Environment Setup

- **Date:** Monday, Nov 3, 2025
- **Theme:** Welcome everyone, set expectations, and make sure every student can create and run a Python project using uv, Git, and VS Code.

## Learning Objectives
- Understand course structure, grading, and support channels.
- Install or verify core tools: uv, Git, Python 3.11+, VS Code, Docker Desktop.
- Create a minimal Python project with uv and run tests from the terminal.

## Before Class – Equipment Check (JiTT)
Ask students to complete this 5-minute checklist the night before:
- Run `uv --version`, `git --version`, and `vim --version` (add `code --version` only if you already use VS Code) and paste the outputs in the Discord `#helpdesk` channel using the Problem → Action → Result → Desired format.
- Note any failures or missing commands so we can triage on arrival.
- This just-in-time teaching checkpoint lets us spend class time on the fun parts instead of long installs.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| **PART A – Theory & Motivation** | **45 min** | **Talk + Discussion** | **Course overview, tools landscape, mindset** |
| Warm welcome | 5 min | Discussion | Names, icebreaker question ("What do you hope to build this year?") |
| Course overview | 15 min | Talk | Syllabus, grading (3 exercises), expectations for collaboration and AI usage |
| Tool belt briefing | 25 min | Talk + quick demos | Why uv, how Git will be used, VS Code essentials, FastAPI for backends, Docker for deployment, LLMs as coding assistants and in projects |
| **Break** | **10 min** | **—** | **Encourage movement – launch [10-minute timer](https://e.ggtimer.com/10minutes)** |
| **PART B – Hands-on Lab 1** | **45 min** | **Guided coding** | **Scaffold the starter project, run first test** |
| Environment setup | 10 min | Guided | Install verification, create project directory |
| uv project init | 15 min | Live code | Create venv, init project, add pytest |
| First test | 15 min | Live code | Write and run test_math.py |
| Git foundation | 5 min | Live code | Initialize repo, first commit |
| **Break** | **10 min** | **—** | **Encourage movement – launch [10-minute timer](https://e.ggtimer.com/10minutes)** |
| **PART C – Hands-on Lab 2** | **45 min** | **Guided coding** | **Explore VS Code, Git basics, README writing** |
| VS Code tour | 15 min | Guided | Open project, install extensions, test explorer |
| Terminal comfort | 15 min | Live code | Basic shell commands, file operations |
| Documentation | 10 min | Live code | Create README with run instructions |
| Git workflow | 5 min | Live code | Stage, commit, push (optional GitHub setup) |
| **Wrap-up** | **5 min** | **Discussion** | **Recap, homework reminder** |

### Session Flow at a Glance
1. **Part A – Theory & Motivation:** Kickoff, expectations, tooling overview, and legacy resources.
2. **Part B – Hands-on Lab 1:** Scaffold `hello-uv`, run tests, commit your work.
3. **Break – 10 Minutes:** Launch the shared [10-minute timer](https://e.ggtimer.com/10minutes) and reset.
4. **Part C – Hands-on Lab 2:** VS Code workflow, terminal practice, documentation, and Git basics.
5. **Wrap-up:** Recap key wins and outline homework.

## Lab Quick Reference (Printable)

Use these numbered steps when you call out B# (Part B) or C# (Part C). Every command assumes a Bash-compatible shell (macOS, Linux, or WSL). Windows PowerShell users can substitute `ni` for `touch`.

| Step | Command(s) | Purpose |
| --- | --- | --- |
| **B1** | `mkdir hello-uv && cd hello-uv` | Create and enter the project folder. |
| **B2** | `uv venv --python 3.11`<br>`uv init`<br>`uv add pytest` | Provision an isolated Python, scaffold `pyproject.toml`, and lock `pytest`. |
| **B3** | `mkdir -p tests`<br>`vim tests/test_math.py` | Create the tests directory and open the starter test file in `vim`. |
| **B4** | _In vim_ paste:<br>`def test_addition(): ...` (three functions)<br>Then `Esc :wq` | Add the arithmetic test trio; save and exit. |
| **B5** | `uv run pytest -q` | Run the suite; expect three passing dots. |
| **B6** | `cat > .gitignore <<'EOF' ... EOF`<br>`git init`<br>`git add .`<br>`git commit -m "chore: bootstrap hello-uv project"` | Ignore virtualenv artefacts, snapshot the initial project. |
| **C1** | `code .` (or VS Code → **Open Folder**) | Launch VS Code with the project. Install Python, Pylance, GitLens, REST Client, Docker extensions. |
| **C2** | `ls -la` · `echo "Scratch pad" > notes.txt` · `git status` · `rm notes.txt` | Practice shell navigation, create/delete a file, observe Git status changes. |
| **C3** | `touch README.md` · `vim README.md` | Create and edit the README with setup and test guidance. |
| **C4** | `git add README.md`<br>`git commit -m "docs: add project README"` | Stage and commit documentation. |
| **C5 (Optional)** | `git branch -M main`<br>`git remote add origin git@github.com:EASS-HIT-PART-A-2025-CLASS-VIII/hello-uv-<handle>.git`<br>`git push -u origin main` | Publish to the course GitHub organisation. |

> Keep this table handy during class. The detailed teaching notes below reference the same B#/C# labels instead of repeating commands inline.

## Talking Script – Course Overview (First 45 Minutes)
1. **Introduce yourself and the course tone.** "This class is about learning by doing. You will see me live-code and you will copy/paste freely. Questions are always welcome. Trust the process—even if things feel unfamiliar at first, by the end you'll be building real web applications with modern tools."
2. **Set expectations for participation.** "We use Discord for day-to-day help. Post what you tried before asking. Pair up during labs so nobody gets stuck. Join using https://discord.gg/EYjQrSmF7f and use the `#helpdesk` channel for all technical questions."
3. **Explain grading.** “We have three large exercises. Each one is assigned on a Monday and due on a Tuesday three weeks later. There are no surprise quizzes. Show up, code along, and you will earn the grade.”
4. **Clarify AI policy.** "You may use AI tools like ChatGPT, Claude, Gemini, Cursor, Copilot, LM Studio, or Ollama. You must understand every line you submit, keep a lightweight spec (spec.md or a tessl.io export) in your repo, and when you ask for help share Problem → Action → Result → Desired."
5. **Outline the tool belt.** "Today we confirm Python 3.11+, uv for environments, Git for version control, VS Code for editing, and Docker Desktop for later sessions. Soon we'll add FastAPI for building web APIs and learn to use LLMs both as coding assistants (to help you write code faster) and as components inside your applications (like calling a local LLM endpoint)."
6. **Preview AWS Academy requirement.** "You also complete three AWS Academy Cloud Foundations modules—Compute, Storage, and Databases. All three modules are due **Tuesday, Dec 16, 2025**. This gives you plenty of time, but don't wait until the last minute. Start early and pace yourself."
7. **Transition to Part B.** "Let's make sure every laptop can create a project. Follow along exactly; copy/paste saves time."

---

## PART A – Theory & Motivation (45 Minutes)

### Warm Welcome (5 Minutes)
**Instructor Actions:**
1. Introduce yourself: name, background, what you're excited about this semester.
2. Ice breaker question: "What do you hope to build this year?" (Go around the room or ask for volunteers.)
3. Explain the "trust the process" philosophy: "Even if Git or Docker feel foreign now, by mid-semester you'll be deploying real applications."

**Student Actions:**
- Share your name and one thing you hope to build (web app, API, automation tool, etc.).
- Note: No technical prerequisites needed today—we build from scratch together.

### Course Overview (15 Minutes)
**Instructor Talking Points:**

1. **Course structure:**
   - 12 meetings (Mondays), each with theory + two hands-on labs.
   - Three major exercises (EX1, EX2, EX3) with Tuesday deadlines.
   - AWS Academy Cloud Foundations: three modules (Compute, Storage, Databases) due **Tue Dec 16, 2025**.

2. **Grading breakdown:**
   - EX1 (Backend API): 25%
   - EX2 (Frontend UI): 25%
   - EX3 (Multi-service with Docker Compose): 25%
   - AWS Academy (3 modules): 25%
   - All components must be completed to pass the course.

3. **Collaboration & support:**
   - Discord server: https://discord.gg/EYjQrSmF7f
   - Use the `#helpdesk` channel for all technical questions, AWS Academy help, and general support.
   - When asking for help, follow the **Problem → Action → Result → Desired** pattern: “I tried X, saw result R, and I’m aiming for Y.” Screenshots alone are not enough—share the exact command and output.
   - Pair programming during labs is encouraged—nobody should be stuck alone.
   - Once you join Discord, share your GitHub username and email so I can send you the GitHub Classroom invitations.
   - Legacy study aids: download the Hebrew alumni notes at `lectures/notes/EASS-Complete-Natalie.pdf` and the archived slide deck at `lectures/archive/all_slides.pdf` for extra context between sessions.

4. **AI policy (crucial):**
   - **You MAY use AI tools** (ChatGPT, Claude, Gemini, Cursor, GitHub Copilot, LM Studio, Ollama, etc.).
   - **You MUST understand** every line of code you submit and be ready to explain it during reviews.
   - Keep a lightweight specification in your repo (even a `spec.md` or a [tessl.io](https://tessl.io/) export) so both humans and AI helpers know the goal before you start coding. Update it as requirements evolve.

5. **Learning philosophy:**
   - "This is a hands-on class. You'll see me type commands, and you'll type the same commands."
   - "Copy/paste is not cheating—it's efficient. But you must run the code and verify it works."
   - "Questions are always welcome. If you're confused, others probably are too."

**Student Actions:**
- Join Discord now if you have a device handy.
- Note the three exercise deadlines in your calendar.
- Ask any questions about grading or expectations.

### Tool Belt Briefing (25 Minutes)
**Instructor Talking Points:**

**The Five Pillars of Our Stack:**

1. **Python 3.11+** – Our main programming language.
   - Modern syntax (type hints, `match` statements).
   - Rich ecosystem for web development.

2. **uv** – Fast Python environment and dependency manager.
   - Replaces `pip` + `virtualenv` + `pip-tools` with one tool.
   - Creates `pyproject.toml` (project manifest) and `uv.lock` (exact versions).
   - Can specify Python version: `uv venv --python 3.11` or `uv venv --python 3.12`.
   - Why not just pip? uv is faster (written in Rust), handles lockfiles automatically, manages Python versions, and will be the industry standard soon.
   - Think of it as: cargo (Rust) or npm (Node.js) but for Python.
   - Quick demo: 
     ```bash
     uv venv --python 3.11    # creates .venv with Python 3.11
     uv add pytest            # installs and locks dependency
     uv run pytest            # runs command in the environment
     uv run python --version  # check Python version in venv
     ```

3. **Git & GitHub** – Version control and collaboration.
   - Every project starts with `git init`.
   - Git tracks changes to your code over time—like "undo" on steroids.
   - You'll commit often: "Save early, save often."
   - We'll use GitHub Classroom for exercise submissions.
   - Basic workflow you'll learn today:
     ```bash
     git status         # what changed?
     git add .          # stage changes
     git commit -m "…"  # save snapshot with a message
     git push           # send to GitHub
     ```
   - Example: if you break something, `git log` shows history and you can go back to any previous version.

4. **curl** – Command-line tool for HTTP requests.
   - Imagine a browser without the graphics: you type the request, it prints the response.
   - Use it before writing Python so you can see the raw data flowing over HTTP.
   - Quick demo:
     ```bash
     curl https://api.github.com/users/github        # GET request
     curl -s https://api.github.com/users/github | head -5
     ```
     - `curl` fetches data from the URL.
     - `-s` hides progress noise; piping to `head -5` keeps the output short.
   - We'll rely on `curl` to debug APIs all semester.

5. **VS Code** – Our code editor.
   - Extensions we'll install today: Python, Pylance, GitLens, Docker, REST Client.
   - Built-in terminal (no need to switch windows).
   - Test explorer (run tests with one click).
   - IntelliSense (autocomplete and hints as you type).
   - Why VS Code? Free, powerful, works the same on Mac/Windows/Linux, and has extensions for everything.

6. **FastAPI** – Modern web framework (coming in Meeting 3).
   - Build REST APIs with automatic validation and documentation.
   - Type-safe: catches bugs before they reach production.
   - Quick preview:
     ```python
     from fastapi import FastAPI
     app = FastAPI()
     
     @app.get("/")
     def read_root():
         return {"message": "Hello World"}
     ```
   - Why FastAPI? Fast (high performance), modern (uses Python type hints), has built-in OpenAPI docs, and is used by Netflix, Uber, and Microsoft.
   - We'll also use **httpx** (not requests) for making HTTP calls—it's modern, supports async, and is the recommended client for FastAPI.

7. **Docker** – Package your app and its environment (Meeting 4).
   - "Works on my machine" → "Works everywhere."
   - One `Dockerfile` = reproducible deployment.
   - Docker Compose = run multiple services together (API + database + reverse proxy).
   - Think of it as: a lightweight virtual machine that packages your app with all its dependencies.
   - Why Docker? Consistent environments, easy deployment, industry standard, and makes collaboration easier.
   - Preview command: `docker run hello-world` (we'll verify this works today).

8. **LLMs (Large Language Models)** – AI as tool and component.
   - **As coding assistant:** Use ChatGPT/Cursor to generate boilerplate, write tests, debug errors.
     - Example prompt: "Write a FastAPI endpoint that returns all items from a list."
     - You review, edit, and verify the code.
   - **Inside your projects:** Call LLMs via API (OpenAI) or locally (LM Studio).
     - Example: build a chatbot API that uses a local LLM for responses.
     - We'll cover this in Meetings 8 and 12.
   - **Key principle:** AI accelerates your work, but you stay in the driver's seat. You must understand every line of code.

9. **pytest** – Testing framework (we'll use today).
   - Automatically discovers and runs test functions (functions starting with `test_`).
   - Simple assertions: `assert 2 + 2 == 4`
   - Why test? Catch bugs early, document expected behavior, enable refactoring with confidence.
   - Example:
     ```python
     def test_addition():
         result = 2 + 2
         assert result == 4  # test passes if True, fails if False
     ```

**Quick Demos (5 Minutes Each):**

1. **curl demo:**
   ```bash
   curl https://api.github.com/users/aws
   curl -s https://api.github.com/users/aws | head -5
   ```
   - Confirms your network works and shows the raw JSON the API returns.
   - The second command trims the output so it is readable in class.

2. **uv demo:**
   ```bash
   uv venv --python 3.11
   source .venv/bin/activate  # Mac/Linux
   # .venv\Scripts\activate   # Windows
   uv add httpx
   uv run python --version
   uv run python -c "import httpx; print(httpx.get('https://api.github.com/users/aws').json()['followers'])"
   ```
   - `uv venv` creates an isolated Python.
   - `uv add` installs a package and locks the version.
   - `uv run` executes commands inside that environment.

3. **Git demo:**
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "you@school.edu"
   git init
   echo "# Test Project" > README.md
   git add README.md
   git commit -m "Initial commit"
   git log --oneline
   ```
   - Configure your identity once, then initialize a repo, stage, and commit.
   - `git log --oneline` proves the commit is stored.

4. **VS Code demo:**
   - Open Command Palette (Cmd+Shift+P / Ctrl+Shift+P).
   - Install Python extension.
   - Show integrated terminal (Ctrl+`).
   - Show test explorer (beaker icon on left sidebar).

**What-If Troubleshooting Chat (5 Minutes):**
- Ask the room: "What if `uv --version` fails? What if `python3 --version` shows 3.9? What if `git status` is empty?" Collect answers and reinforce that the Troubleshooting Guide at the end lists the fixes.
- Goal: Normalize encountering errors and remind students to capture Problem → Action → Result → Desired when they ask for help.

**Mini Quiz (3 Minutes):**
- Rapid-fire questions: "Which command activates the virtualenv?", "What does `git add .` do?", "Why keep a spec file in the repo?" Let students answer verbally or via a quick poll to reinforce retention.

**Student Actions:**
- Verify you have a terminal open and can run basic commands.
- If you haven't already, install uv (we'll do this together in Part B if needed).
- Open VS Code and locate the terminal and extensions panels.

**Transition to Part B:**
"Now we're going to build a real project together. Open your terminal and follow along. If you get an error, raise your hand—we'll fix it together."

---

## BREAK (10 Minutes)
Encourage movement, grab water, and launch the shared [10-minute timer](https://e.ggtimer.com/10minutes) before diving into Part B.

---

## PART B – Hands-on Lab 1: Build Your First Python Project (45 Minutes)

**Goal:** By the end of this lab, every student will have a working Python project with a passing test, managed by uv and tracked with Git.

### Environment Setup (10 Minutes)

**Instructor Actions:**
1. Invite students to run **Quick Reference B1** (create project directory) and confirm they are inside it.
2. Walk through the uv installation check from **B2** step-by-step: verify `uv --version`, demonstrate the installer command if anyone is missing it, then show how restarting the shell surfaces the new binary.
3. Remind everyone to validate Git identity (`git config --global ...`) before moving on, modelling the Problem → Action → Result → Desired pattern if someone’s identity is unset.

**Student Checklist:**
- [ ] Terminal is open
- [ ] uv is installed (run `uv --version` to verify)
- [ ] Git is configured with your name and email

**Buddy Check (5 Minutes):**
- Pair up with the person next to you and confirm each checkbox together.
- If one of you is stuck, post in `#helpdesk` using the Problem → Action → Result → Desired format so an instructor or classmate can jump in.

### uv Project Initialization (15 Minutes)

**Instructor Live Coding (students follow along exactly):**
1. Execute **Quick Reference B2** together. Narrate what each command does: `uv venv` isolates Python, `uv init` scaffolds metadata, `uv add` pins dependencies.
2. After activation, pause so students can notice the `(.venv)` prompt. Emphasise why running tools through `uv run ...` keeps them inside the environment.
3. Close with a group `uv run pytest --version` to prove the dependency landed.

**Student Checkpoint:**
- Raise your hand if you see "pytest 8.x.x" or similar output.
- If you see an error, stop and ask for help.

### First Test (15 Minutes)

**Instructor Live Coding:**
1. Guide students through **Quick Reference B3-B5**. Narrate the vim keystrokes (insert, paste, `Esc :wq`) and explain pytest output symbols (`.` vs `F`).
2. After the intentional failure demo, debrief why red tracebacks are learning signals, not disasters. Encourage them to share PAR↴D-formatted questions when they get stuck.

**Student Task Card:**
- Run `uv run pytest -q` and verify you see `3 passed`.
- If you see failures, check your code matches exactly.
- Raise your hand if tests don't pass.

### Git Foundation (5 Minutes)

**Instructor Live Coding:**
1. Execute **Quick Reference B6** and pause after each sub-command to emphasise the why: `.gitignore` keeps noise out of history, `git add .` stages, the commit message documents intent.
2. Show `git log --oneline` and explain how it becomes the time machine for the project.

**Student Checkpoint:**
- Everyone should see one commit in `git log --oneline`.
- If you see an error about user.name or user.email, run the config commands from earlier.

### Part B Wrap-Up (2 Minutes)

**Instructor Summary:**
"You just created a Python project from scratch using modern tools. You have:
- A virtual environment (`.venv`)
- A project manifest (`pyproject.toml`)
- A lockfile (`uv.lock`)
- Automated tests (`tests/test_math.py`)
- Version control (`.git`)

This is the foundation for every project you'll build in this course."

**Student Success Criteria:**
- [ ] `uv run pytest -q` shows `3 passed`
- [ ] `git log --oneline` shows at least one commit
- [ ] You understand what each command does (if not, ask now!)

**Stretch Goal (Optional):**
- Create a new branch `feature/division-test`, add a division test that guards against zero division, commit it, and keep it ready for your first pull request next week.

---

## BREAK (10 Minutes)
Stand up, stretch, grab water. When we return, we'll explore VS Code and improve our project.
Launch a shared [10-minute timer](https://e.ggtimer.com/10minutes) so everyone returns on time.

---

## PART C – Hands-on Lab 2: VS Code, Git Workflow, and Documentation (45 Minutes)

**Goal:** Get comfortable with VS Code as your development environment, practice Git basics, and document your project professionally.

### VS Code Tour (15 Minutes)

**Instructor Live Demo (students follow along):**
1. Trigger **Quick Reference C1** to open VS Code (either `code .` or File → Open Folder). If the shell command is missing, demonstrate installing it via the Command Palette.
2. **Explorer sidebar (left side):**
   - Show the file tree
   - Expand/collapse folders
   - Create new files/folders by right-clicking
3. **Install essential extensions:**
   - Click Extensions icon (or Ctrl+Shift+X / Cmd+Shift+X)
   - Install these one by one:
     1. **Python** (Microsoft) – syntax highlighting, debugging
     2. **Pylance** (Microsoft) – type checking, IntelliSense
     3. **GitLens** – Git superpowers (blame, history)
     4. **REST Client** – test HTTP APIs (we'll use this in Meeting 2)
     5. **Docker** (Microsoft) – manage containers (we'll use this in Meeting 4)
4. **Open integrated terminal:**
   - Press Ctrl+` (backtick) or View → Terminal
   - You can have multiple terminal tabs
   - The terminal starts in your project directory automatically
5. **Test Explorer:**
   - Click the beaker icon on the left sidebar (Testing)
   - You should see `tests/test_math.py` with three tests
   - Click the green play button next to a test to run it
   - Click the play button at the top to run all tests
6. **Run a test from the explorer:**
   - Click the play button next to `test_addition`
   - See the checkmark when it passes
   - Click on the test to see the code

**Instructor Tips:**
- Walk around the room to verify everyone has VS Code open
- Help students who get stuck on extension installation
- Show how to split the editor (drag a file tab to the side)

**Student Checklist:**
- [ ] VS Code is open with your `hello-uv` project
- [ ] Python, Pylance, and GitLens extensions are installed
- [ ] Test Explorer shows your three tests
- [ ] You can run a test from the Test Explorer

### Terminal Comfort (15 Minutes)

**Instructor Live Coding (students type along):**
1. Follow **Quick Reference C2** as a guided practice: run `pwd`, `ls -la`, create `notes.txt`, inspect it, then clean it up. Explain what each command reveals about the project.
2. Demonstrate `git log --oneline --graph` and optional aliases once the workspace is back to a clean state.

**Additional Terminal Commands to Demonstrate:**

```bash
# Count lines in a file:
wc -l tests/test_math.py

# Search for text in files:
grep "def test" tests/test_math.py

# Show disk usage:
du -sh .venv

# Show Python version:
python3 --version
uv run python --version

# Show environment info:
uv run python -c "import sys; print(sys.executable)"
```

Share a printed or PDF **terminal cheat sheet** (Mac shortcuts on the left, Windows/WSL on the right) so students can reference commands after class.

**Student Task:**
- Type each command and observe the output.
- Ask questions if anything is confusing.
- Experiment: create a file, read it, delete it.

### Documentation (10 Minutes)

**Instructor Live Coding:**
1. Execute **Quick Reference C3** and model the `vim` workflow again: insert mode, paste template, `Esc :wq`.
2. **Write documentation** (students copy this structure):

   ```markdown
   # Hello uv Project
   
   A minimal Python project demonstrating modern tooling with `uv`, `pytest`, and `Git`.
   
   ## Setup
   
   1. Clone this repository (or create a new project):
      ```bash
      git clone <your-repo-url>
      cd hello-uv
      ```
   
   2. Create and activate a virtual environment:
      ```bash
      uv venv
      source .venv/bin/activate  # Mac/Linux
      # .venv\Scripts\Activate.ps1  # Windows
      ```
   
   3. Install dependencies:
      ```bash
      uv sync
      ```
   
   ## How to Run Tests
   
   Run all tests:
   ```bash
   uv run pytest
   ```
   
   Run with verbose output:
   ```bash
   uv run pytest -v
   ```
   
   Run in quiet mode:
   ```bash
   uv run pytest -q
   ```
   
   ## Project Structure
   
   ```
   hello-uv/
   ├── .venv/              # Virtual environment (not committed)
   ├── tests/              # Test files
   │   └── test_math.py
   ├── pyproject.toml      # Project manifest
   ├── uv.lock             # Locked dependencies
   └── README.md           # This file
   ```
   
   ## Tools Used
   
   - **uv**: Fast Python package manager and environment manager
   - **pytest**: Testing framework
   - **Git**: Version control
   
   ## Next Steps
   
   - Add more complex tests
   - Create application code in `app/` directory
   - Add FastAPI endpoints (Meeting 3)
   - Containerize with Docker (Meeting 4)
   
   ## Author
   
   Your Name - EASS Course, Fall 2025
   ```

3. **Save the file** (Cmd+S / Ctrl+S).

**Instructor Explanation:**
"A good README answers three questions:
1. What is this project?
2. How do I set it up?
3. How do I use it?

Every project you submit must have a README. This is your first impression."

**Student Task:**
- Create your own `README.md` with at least:
  - Project title
  - Setup instructions
  - How to run tests
  - Your name

### Git Workflow (5 Minutes)

**Instructor Live Coding:**
1. Run **Quick Reference C4** to stage and commit `README.md`. Pause to highlight the difference between working tree, staging area, and commit history.
2. For teams ready to publish, follow **Quick Reference C5**. Show the live GitHub repo so students see the end-to-end flow (create repo under the organisation, add remote, push, verify).

**Collaboration Preview (2 Minutes):**
- Explain that future exercises will use feature branches and pull requests (branch → commit → open PR → code review → merge).
- Show one slide or live GitHub screenshot so students see where comments and reviews will appear.

**Instructor Note:**
If students don't have GitHub accounts or SSH keys set up, make this optional homework. The important part is they understand local Git workflow.

**Student Checkpoint:**
- [ ] `README.md` exists and has content
- [ ] `git log --oneline` shows two commits
- [ ] (Optional) Project is pushed to GitHub

### Part C Wrap-Up (2 Minutes)

**Instructor Summary:**
"You now have a complete development workflow:
1. Write code in VS Code
2. Run tests to verify it works
3. Commit changes with Git
4. Document your work in README

This is the workflow you'll use for all three exercises."

**Student Success Criteria:**
- [ ] Comfortable opening projects in VS Code
- [ ] Can run tests from Test Explorer
- [ ] Can use basic terminal commands
- [ ] Have created a professional README
- [ ] Understand `git add`, `git commit`, `git status`

---

## Wrap-Up Script (5 Minutes)

**Instructor Final Thoughts:**

1. **Celebrate progress:**
   "Today you built a working Python project from scratch and learned the tools you'll use all semester. That's a huge accomplishment!"

2. **Review what you accomplished:**
   - ✅ Created a Python project with `uv`
   - ✅ Wrote and ran automated tests with `pytest`
   - ✅ Tracked your work with `Git`
   - ✅ Documented your project with `README.md`
   - ✅ Set up VS Code as your development environment

3. **Homework and next steps:**
   - **Tonight:** 
     - Join Discord (https://discord.gg/EYjQrSmF7f)
     - Share your GitHub username and email in `#helpdesk` so I can send you invitations
   - **This week:** 
     - Wait for the AWS Academy invitation link (I'll send it after you share your email)
     - Once you get it, enroll in AWS Academy Cloud Foundations immediately
     - Start the Compute module early—all three modules are due **Tuesday, Dec 16, 2025**
     - (Optional) Push your `hello-uv` project to GitHub once you get the GitHub Classroom link
   - **Before next class:**
     - Make sure Docker Desktop is installed and running
     - Run `docker run --rm hello-world` to verify Docker works
     - Review HTTP basics if you're rusty (we'll cover it in depth next week)

4. **Preview of Meeting 2:**
   "Next Monday we dive into HTTP and REST APIs. You'll learn how browsers and servers talk, and you'll build your first HTTP client using httpx. Bring your laptops charged and your `hello-uv` project ready."

5. **Q&A:**
   "Any questions about today's material, the exercises, grading, or AWS Academy?"

6. **Final reminder:**
   "Remember: trust the process. If something doesn't make sense today, it will make sense by Meeting 4. Keep coding, keep asking questions in `#helpdesk`, and help each other out on Discord."

7. **Resource wall:**
   "Snapshot these links—Discord, GitHub Classroom FAQ, Troubleshooting Guide, tessl.io spec template, and the terminal cheat sheet PDF—so you can reference them after class."

**Student Exit Ticket (Optional):**
Post in Discord `#helpdesk`:
- One thing you learned today
- One thing you're confused about
- One thing you're excited to build

---
## Materials Checklist (For Instructor)

### Before Class:
- [ ] Python 3.11+ installed on all lab machines
- [ ] uv installed (`pip install uv` or official installer)
- [ ] Git installed and verified
- [ ] VS Code installed with Python extension
- [ ] Docker Desktop installed (for verification only today)
- [ ] Discord invite link ready (https://discord.gg/EYjQrSmF7f)
- [ ] Slides or talking points prepared for Part A
- [ ] Example `hello-uv` project on your machine for demo

### During Class:
- [ ] Share Discord link in chat/on board
- [ ] Remind students to share GitHub username and email in `#helpdesk`
- [ ] Walk around during hands-on labs to help students
- [ ] Take note of common errors for troubleshooting guide

### After Class:
- [ ] Post session recording (if recorded) to Canvas
- [ ] Share troubleshooting notes in Discord
- [ ] Send AWS Academy invitation links to students who shared their emails
- [ ] Send GitHub Classroom invitation links to students who shared their GitHub usernames
- [ ] Prepare starter code for Meeting 2 (HTTP client examples with httpx)

---

## Support Blocks (Run as Needed)

### AWS Academy Onboarding (15 Minutes)
*Assign during class only if many students still need access; otherwise set as homework.*

**Instructor prompts:**
1. Highlight the three required Cloud Foundations modules: **Compute**, **Storage**, **Databases**.
2. Point students to [https://www.awsacademy.com](https://www.awsacademy.com) → **Student** sign-in and remind them to share their GitHub username + email in `#helpdesk` so you can issue invites.
3. Restate the single deadline: **Tuesday, Dec 16, 2025** for all modules. Encourage pacing (e.g., one module per two weeks).
4. Demonstrate how to capture the completion certificate for Canvas submission and where to ask questions (always `#helpdesk`, using PAR↴D format).

**Student follow-up:** enroll, confirm module access, schedule time to finish Compute (target: Nov 25) and the other two by Dec 16.

### Windows WSL Setup Guide (Self-Contained)
*Share with Windows participants who are still on PowerShell-only environments.*

1. In elevated PowerShell:
   ```powershell
   wsl --install
   wsl --set-default-version 2
   ```
2. Inside the new Ubuntu shell:
   ```bash
   sudo apt update && sudo apt -y upgrade
   sudo apt -y install build-essential curl git python3 python3-venv
   ```
3. Install the VS Code "Remote – WSL" extension, open the project inside WSL, and verify `/mnt/c` visibility so students know where Windows files live.

### Docker Sanity Check (5 Minutes)
Assign as homework once uv/Git are stable.

```bash
docker run --rm hello-world
```
Expect the success message; on Windows remind them to enable WSL Integration in Docker Desktop first.

---

## Troubleshooting Guide

### Issue: `uv: command not found`

**Solution:**
```bash
# Try installing with pip:
pip install uv

# Or use the official installer:
# Mac/Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows:
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# After installing, restart your terminal or run:
source ~/.bashrc  # or ~/.zshrc on Mac
```

### Issue: `git commit` fails with "Please tell me who you are"

**Solution:**
```bash
git config --global user.name "Your Name"
git config --global user.email "you@school.edu"
```

### Issue: `uv venv` fails on Windows

**Solution:**
```bash
# Make sure you're in PowerShell (not Command Prompt)
# Enable script execution:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Then try again:
uv venv
.venv\Scripts\Activate.ps1
```
If the installer still fails mid-class, fall back temporarily:
```bash
python3 -m venv .venv
.venv\Scripts\Activate.ps1
```
Then revisit the uv installation steps after the session.

### Issue: Python version is too old (< 3.11)

**Solution:**
- **Mac:** `brew install python@3.12` (requires Homebrew)
- **Windows:** Download from python.org or use Windows Store
- **Linux:** `sudo apt install python3.12` (Ubuntu) or equivalent

### Issue: Tests don't run in VS Code Test Explorer

**Solution:**
1. Make sure Python extension is installed
2. Open Command Palette (Cmd+Shift+P / Ctrl+Shift+P)
3. Type "Python: Configure Tests"
4. Select "pytest"
5. Select "tests" as the test directory

### Issue: Docker doesn't work on Windows

**Solution:**
1. Verify WSL 2 is installed: `wsl --version`
2. Open Docker Desktop → Settings → General → "Use WSL 2 based engine" is checked
3. Go to Settings → Resources → WSL Integration → enable integration with your distro
4. Restart Docker Desktop

### Issue: `ModuleNotFoundError: No module named 'httpx'`

**Solution:**
```bash
uv add httpx
uv run python -c "import httpx; client = httpx.Client(); print(client.get('https://api.github.com/users/zozo123').status_code)"
```
- Make sure you run the command inside the project directory so `uv run` picks up the virtual environment.
- If you already installed dependencies, run `uv sync` before retrying your `python -c` script.
- For multi-line scripts, pipe a here-doc into Python instead of packing everything into one line:
  ```bash
  uv run python - <<'PY'
  import httpx

  client = httpx.Client()
  n = p = 0
  while True:
      response = client.get(
          "https://api.github.com/users/zozo123/repos",
          params={"per_page": 100, "page": p + 1},
      ).json()
      if not response:
          break
      n += len(response)
      p += 1
  print(n)
  PY
  ```

### Issue: Can't push to GitHub (permission denied)

**Solution for SSH:**
```bash
# Generate SSH key:
ssh-keygen -t ed25519 -C "you@school.edu"

# Start ssh-agent:
eval "$(ssh-agent -s)"

# Add key:
ssh-add ~/.ssh/id_ed25519

# Copy public key:
cat ~/.ssh/id_ed25519.pub
# Paste this into GitHub → Settings → SSH and GPG keys → New SSH key
```

**Solution for HTTPS:**
- Use a Personal Access Token instead of password
- GitHub → Settings → Developer settings → Personal access tokens → Generate new token
- Use the token as your password when pushing

### Issue: VS Code `code` command doesn't work

**Solution:**
- **Mac:** Open VS Code → Command Palette (Cmd+Shift+P) → "Shell Command: Install 'code' command in PATH"
- **Windows:** Should work by default, make sure VS Code is in your PATH
- **Linux:** Usually works after install, or add to `.bashrc`: `export PATH="$PATH:/usr/share/code/bin"`

---

## Extension: Advanced Git Setup (Optional, for Fast Students)

If students finish early, guide them through:

### 1. Create a `.gitignore` file:

```bash
# Create .gitignore in project root:
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# Virtual environments
.venv/
venv/
ENV/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Testing
.pytest_cache/
.coverage
htmlcov/

# OS
.DS_Store
Thumbs.db

# uv
.uv/
EOF

git add .gitignore
git commit -m "chore: add .gitignore for Python project"
```

### 2. Create a feature branch:

```bash
# Create and switch to a new branch:
git checkout -b feature/add-division-test

# Add a new test:
# (edit tests/test_math.py to add test_division)

git add tests/test_math.py
git commit -m "test: add division test"

# Switch back to main:
git checkout main

# Merge the feature:
git merge feature/add-division-test
```

### 3. View commit history with more detail:

```bash
git log --oneline --graph --decorate --all
git log --stat
git show HEAD
```

---

## Student Success Criteria (Final Checklist)

By the end of Session 01, every student should be able to:

- [ ] **Environment:** Create a Python virtual environment with `uv venv`
- [ ] **Dependencies:** Add packages with `uv add <package>`
- [ ] **Testing:** Write and run tests with `pytest`
- [ ] **Version Control:** Initialize a Git repo, stage files, and commit changes
- [ ] **IDE:** Open a project in VS Code and use the Test Explorer
- [ ] **Terminal:** Navigate directories, create/delete files, run commands
- [ ] **Documentation:** Create a README with setup and usage instructions
- [ ] **Troubleshooting:** Know where to ask for help (Discord `#helpdesk`)
- [ ] **Artifacts:** Project folder includes `.venv`, `pyproject.toml`, `README.md`, and `tests/test_math.py`

**If a student cannot do any of the above, schedule office hours before Meeting 2.**

---

## AI Prompt Kit (For Instructors and Students)

Use these prompts with ChatGPT/Cursor to generate additional content:

### For Instructors:

1. **Generate troubleshooting scenarios:**
   > "Create five common error scenarios students might encounter when setting up a Python project with uv and pytest on Windows, with detailed solutions."

2. **Create quiz questions:**
   > "Write ten multiple-choice questions testing understanding of Git basics (init, add, commit, status) appropriate for first-time users."

3. **Generate coding exercises:**
   > "Create three progressive coding exercises for students learning pytest: 1) basic assertions, 2) parameterized tests, 3) fixtures."
4. **Verify installations:**
   > "Write a numbered checklist for students to confirm uv, pytest, git, VS Code, and Docker Desktop are installed correctly on Windows with WSL."
5. **Welcome message template:**
   > "Draft a short Discord welcome message reminding students to post what they tried before asking for help."

### For Students:

1. **Understand an error message:**
   > "I got this error when running `uv add pytest`: [paste error]. What does it mean and how do I fix it?"

2. **Learn a command:**
   > "Explain what `git commit -m 'message'` does, and why the `-m` flag is important. Give me three example commit messages."

3. **Debug code:**
   > "My pytest test is failing with this error: [paste error]. Here's my code: [paste code]. What's wrong?"

4. **Generate boilerplate:**
   > "Write a pytest test function that checks if a string is a palindrome. Include docstring and edge cases."

---

## Quick Reference (External Resources)

**For quick lookups during class:**

### Google searches:
- `uv python package manager tutorial`
- `pytest getting started guide`
- `git basics cheat sheet`
- `VS Code Python tutorial`
- `install WSL 2 Windows 11 step by step`

### Official documentation:
- uv: https://github.com/astral-sh/uv
- pytest: https://docs.pytest.org/
- Git: https://git-scm.com/doc
- VS Code Python: https://code.visualstudio.com/docs/python/python-tutorial

### Video resources:
- "Git explained in 100 seconds" (Fireship on YouTube)
- "Python virtual environments explained" (Corey Schafer)
- "VS Code setup for Python" (Microsoft)

### ChatGPT prompt:
- “Create a five-point checklist to confirm uv, pytest, git, and Docker Desktop are installed correctly on a student laptop.”

---

**End of Session 01 Guide**
