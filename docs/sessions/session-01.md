# Session 01 – Kickoff and Environment Setup

- **Date:** Monday, Nov 3, 2025
- **Theme:** Welcome everyone, set expectations, and make sure every student can create and run a Python project using uv, Git, and VS Code.

## Learning Objectives
- Understand course structure, grading, and support channels.
- Install or verify core tools: uv, Git, Python 3.11+, VS Code, Docker Desktop.
- Create a minimal Python project with uv and run tests from the terminal.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm welcome | 5 min | Discussion | Names, icebreaker question (“What do you hope to build this year?”) |
| Course overview | 15 min | Talk | Syllabus, grading (3 exercises), expectations for collaboration and AI usage |
| Tool belt briefing | 25 min | Talk + quick demos | Why uv, how Git will be used, VS Code essentials |
| AWS Academy onboarding | 15 min | Guided setup | Enroll in AWS Academy Cloud Foundations and preview required modules |
| Lab 1 | 45 min | Guided coding | Scaffold the starter project, run first test |
| Break | 10 min | — | Encourage movement |
| Lab 2 | 45 min | Guided coding | Explore VS Code, Git basics, README writing |
| Wrap-up | 5 min | Discussion | Recap, homework reminder |

## Talking Script – Course Overview (First 45 Minutes)
1. **Introduce yourself and the course tone.** “This class is about learning by doing. You will see me live-code and you will copy/paste freely. Questions are always welcome.”
2. **Set expectations for participation.** “We use Discord for day-to-day help. Post what you tried before asking. Pair up during labs so nobody gets stuck. Join using https://discord.gg/EYjQrSmF7f so you have access to the `#helpdesk` channel.”
3. **Explain grading.** “We have three large exercises. Each one is assigned on a Monday and due on a Tuesday three weeks later. There are no surprise quizzes. Show up, code along, and you will earn the grade.”
4. **Clarify AI policy.** “You may use AI tools like ChatGPT or Cursor. You must understand the code you submit and document any AI-generated sections in your README.”
5. **Outline the tool belt.** “Today we confirm Python 3.11+, uv for environments, Git for version control, VS Code for editing, and Docker Desktop for later sessions.”
6. **Preview AWS Academy requirement.** “You also complete three AWS Academy Cloud Foundations modules—Compute, Storage, and Databases. All certificates must be submitted by **Tuesday, Dec 16, 2025** (mid-December Tuesday). I’ll share pacing tips so nothing piles up.”
7. **Transition to Lab 1.** “Let’s make sure every laptop can create a project. Follow along exactly; copy/paste saves time.”

## AWS Academy Onboarding (15 Minutes)
### Instructor Steps
1. Share the enrollment link: `https://awsacademy.instructure.com/users/573468/external_tools/4718578`.
2. Ask students to browse to [https://www.awsacademy.com](https://www.awsacademy.com), choose **Student**, and sign in with their school email (or create an account if needed).
3. Walk them through joining the **AWS Academy Cloud Foundations** course and verifying that the **Compute**, **Storage**, and **Databases** modules appear.
4. Explain the completion schedule: all three modules are due **Tuesday, Dec 16, 2025**. Recommended pacing:
   - Finish **Compute** by **Tuesday, Nov 25, 2025**.
   - Finish **Storage** by **Tuesday, Dec 9, 2025**.
   - Finish **Databases** no later than **Tuesday, Dec 16, 2025**.
5. Show how to capture the completion certificate screenshot and upload it to the Canvas assignment titled with the module name.
6. Remind students that AWS modules are independent work but they can ask questions in the `#aws-support` channel on Discord.

### Student Action Items
- Enroll in AWS Academy Cloud Foundations immediately.
- Confirm access to the three required modules.
- Note the deadlines in your planner and set reminders.
- Complete the Compute module before **Tuesday, Nov 25, 2025** and keep proof of completion ready to upload so the **Tue Dec 16, 2025** deadline is relaxed.
- Join the course Discord via https://discord.gg/EYjQrSmF7f and post a short hello in `#introductions`.

## Windows WSL Setup Guide (Self‑Contained)
- In Windows PowerShell (Administrator):
  ```powershell
  wsl --install
  # Reboot if prompted, then set Ubuntu as default distro
  wsl --set-default-version 2
  ```
- Launch “Ubuntu” from Start Menu, then in the Ubuntu shell:
  ```bash
  sudo apt update && sudo apt -y upgrade
  sudo apt -y install build-essential curl git python3 python3-venv
  ```
- Install VS Code “Remote – WSL” extension and open the folder in WSL:
  - In Windows: open VS Code → Extensions → install “Remote – WSL”.
  - Press F1 → “WSL: New WSL Window” → File → Open Folder → your Linux path (e.g., `/home/<you>`).
- Verify cross‑filesystem access:
  ```bash
  ls /mnt/c/Users  # Windows files visible under /mnt/c
  ```

## Docker Sanity Check (5 Minutes)
- Verify Docker Desktop is installed and running (Windows: enable WSL Integration in Docker Settings).
- Run a quick container:
  ```bash
  docker run --rm hello-world
  ```
  Expect a success message confirming Docker works.

## Part B – Hands-on Lab 1 (45 Minutes)
### Instructor Demo Steps
1. Ask everyone to open a terminal in their home directory.
2. Run the following commands together. Say each command before you execute it:
   ```bash
   mkdir hello-uv
   cd hello-uv
   uv venv
   source .venv/bin/activate
   uv init
   uv add pytest
   git init
   git status
   ```
3. Explain what each command did: `uv venv` created the environment, `uv init` generated `pyproject.toml`, and `uv add pytest` recorded dependencies.
4. Create `tests/test_math.py` live:
   ```python
   def test_addition():
       assert 2 + 2 == 4
   ```
5. Run tests:
   ```bash
   uv run pytest -q
   ```
6. Stage and commit:
  ```bash
  git add .
  git commit -m "chore: bootstrap project with uv and pytest"
  ```

### Optional: Connect to GitHub (SSH or HTTPS)
1. Generate an SSH key (recommended):
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   cat ~/.ssh/id_ed25519.pub  # copy this to GitHub → Settings → SSH keys
   ```
   Or plan to use HTTPS.
2. Create an empty GitHub repo (e.g., `hello-uv-<name>`), then add a remote and push:
   ```bash
   git remote add origin git@github.com:<user>/hello-uv-<name>.git
   git branch -M main
   git push -u origin main
   ```
   For HTTPS, use: `git remote add origin https://github.com/<user>/hello-uv-<name>.git`.

### Student Task Card
- Repeat every command on your machine.
- Verify: `uv run pytest -q` shows 1 passed.
- If something fails, raise your hand immediately; we will debug together.

### Expected Output
```
1 passed in 0.01s
```

## Part C – Hands-on Lab 2 (45 Minutes)
### Lab Goals
- Get comfortable inside VS Code.
- Practice basic Git commands (`status`, `add`, `commit`).
- Document how to run tests in a README.

### Step-by-Step Instructions
1. **Open the folder.** In VS Code: `File -> Open Folder -> hello-uv`.
2. **Install recommended extensions.** Python, Pylance, and GitLens. Show students how to check the extensions sidebar.
3. **Run the test from the VS Code test explorer.** Demonstrate the green play button next to the test.
4. **Terminal Tour.** In the VS Code terminal run:
   ```bash
   ls -l
   pwd
   echo "Hello class" > notes.txt
   cat notes.txt
   rm notes.txt
   ```
5. **Create README.** In VS Code create `README.md` with:
   ```markdown
   # Hello uv Project

   ## How to Run the Tests
   ```bash
   uv run pytest -q
   ```
   ```
6. **Commit changes.**
   ```bash
   git status
   git add README.md
   git commit -m "docs: add instructions to run tests"
   ```

### Instructor Tips
- Walk the room to confirm everyone sees the same output.
- Pair students: one reads instructions aloud, the other types.
- Remind students that copy/paste is absolutely fine—accuracy beats speed.

## Wrap-Up Script (5 Minutes)
1. “Today you built a working Python project and ran your first automated test.”
2. “Homework: push this project to GitHub Classroom if you already have the invite. If not, relax—we will share the link this week.”
3. “Start the AWS Academy **Compute** module tonight—aim to finish by **Tue Nov 25**, even though the hard deadline for all modules is **Tue Dec 16, 2025**.”
4. “Next session we learn how browsers and servers talk using HTTP. Bring your laptops charged and ready.”

## Materials Checklist
- Python 3.11 or newer installed.
- uv installed (`pip install uv` or the official installer).
- Git installed and configured (`git config user.name` and `user.email`).
- VS Code with Python extension.
- Docker Desktop installed (verification only; no usage yet).

## Troubleshooting Notes
- If `uv venv` fails, fallback to `python3 -m venv .venv` and revisit uv installation after class.
- If `git commit` complains about identity, run:
  ```bash
  git config --global user.name "First Last"
  git config --global user.email "you@example.com"
  ```
- Remind Windows users to use PowerShell or Git Bash; avoid spaces in directory names.

## Student Success Criteria
- Project folder contains `.venv`, `pyproject.toml`, `README.md`, and `tests/test_math.py`.
- `uv run pytest -q` completes without error.
- Students can explain what uv does and how to add dependencies.

## AI Prompt Kit (Copy/Paste)
- “Write a numbered checklist for students to verify uv, pytest, git, VS Code, and Docker Desktop are installed correctly on Windows + WSL.”
- “Draft a short Discord welcome message reminding students to post what they tried before asking for help.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `install WSL 2 Windows 11 step by step`
- **ChatGPT prompt:** “Create a five-point checklist to confirm uv, pytest, git, and Docker Desktop are installed correctly on a student laptop.”
