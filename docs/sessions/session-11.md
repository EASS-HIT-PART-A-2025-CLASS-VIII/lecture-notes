# Session 11 – Security Foundations

- **Date:** Monday, Jan 12, 2026
- **Theme:** Introduce authentication, authorization, and safe secret handling for the FastAPI service.

## Learning Objectives
- Explain the difference between authentication (who) and authorization (what).
- Implement a simple JWT-based login flow in FastAPI.
- Protect routes with role-based access control and add security-focused tests.

## Agenda
| Segment | Duration | Format | Focus |
| --- | --- | --- | --- |
| Warm-up question | 10 min | Discussion | “What security mishap have you heard about recently?” |
| Security overview | 20 min | Talk | Credentials, secrets, HTTPS, principle of least privilege |
| JWT primer | 15 min | Talk + whiteboard | Token structure (header, payload, signature), expiration |
| Lab 1 | 45 min | Guided coding | Implement login and token creation |
| Break | 10 min | — | |
| Lab 2 | 45 min | Guided coding | Protect endpoints, write tests, load secrets from `.env` |
| EX3 progress check | 10 min | Discussion | Ensure teams are on track for Jan 20 milestone |
| AWS reminder | 5 min | Announcement | Confirm AWS certificates were submitted by Dec 16 (or note make-up steps) |

## Teaching Script – Security Basics
1. “Security is about keeping the right people in and the wrong people out.”
2. Clarify definitions:
   - Authentication: verify identity (username/password, OAuth, etc.).
   - Authorization: decide what an authenticated user can do.
3. Describe JWT structure using a simple example: `header.payload.signature`, base64-encoded.
4. Emphasize safe secret handling: `.env` files, environment variables, never committing secrets to Git.
5. Remind students to double-check that all AWS certificates were uploaded by **Tuesday, Dec 16, 2025**. Anyone missing proof should update Canvas and notify the instructor immediately.

## Part B – Hands-on Lab 1 (45 Minutes)
### Install Dependencies
```bash
uv add "python-jose[cryptography]" "passlib[bcrypt]" python-dotenv
```

### Environment Setup
Create `.env` (not committed to Git):
```
SECRET_KEY=change-me-to-a-long-random-string
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
```

### Security Module (`app/security.py`)
Create `app/security.py` with all security logic in one place:
```python
import os
from datetime import datetime, timedelta
from typing import Dict

from dotenv import load_dotenv
from jose import jwt
from passlib.context import CryptContext

from app.models import User


load_dotenv()
SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret")
ALGORITHM = os.getenv("ALGORITHM", "HS256")
ACCESS_TOKEN_EXPIRE_MINUTES = int(os.getenv("ACCESS_TOKEN_EXPIRE_MINUTES", "30"))

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


fake_users_db: Dict[str, dict[str, str]] = {
    "student": {
        "username": "student",
        "full_name": "Student User",
        "hashed_password": pwd_context.hash("secret"),
        "role": "viewer",
        "disabled": False,
    },
    "teacher": {
        "username": "teacher",
        "full_name": "Teacher User",
        "hashed_password": pwd_context.hash("classroom"),
        "role": "editor",
        "disabled": False,
    },
}


def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)


def get_user(username: str) -> User | None:
    user_dict = fake_users_db.get(username)
    return User(**user_dict) if user_dict else None


def authenticate_user(username: str, password: str) -> User | None:
    user = get_user(username)
    if not user:
        return None
    if not verify_password(password, fake_users_db[username]["hashed_password"]):
        return None
    return user


def create_access_token(data: dict[str, str], expires_delta: timedelta | None = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```

### User Model and Store
Add to `app/models.py`:
```python
from pydantic import BaseModel


class Token(BaseModel):
    access_token: str
    token_type: str


class TokenData(BaseModel):
    username: str | None = None


class User(BaseModel):
    username: str
    full_name: str
    disabled: bool = False
    role: str = "viewer"
```

The `app/security.py` file provides user storage, hashing, verification, and token creation.

### Login Endpoint
In `app/main.py`:
```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from fastapi import Depends, HTTPException
from app.security import authenticate_user, create_access_token, SECRET_KEY, ALGORITHM, get_user
from app.models import Token, User

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()) -> Token:
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=401, detail="Incorrect username or password")
    access_token = create_access_token({"sub": user.username, "role": user.role})
    return Token(access_token=access_token, token_type="bearer")
```

## Part C – Hands-on Lab 2 (45 Minutes)
### Protected Routes
Add helpers:
```python
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError:
        raise HTTPException(status_code=401, detail="Could not validate credentials")
    username: str | None = payload.get("sub")
    if username is None:
        raise HTTPException(status_code=401, detail="Invalid token payload")
    user = get_user(username)
    if user is None:
        raise HTTPException(status_code=401, detail="User not found")
    return user


def require_role(role: str):
    def role_checker(user: User = Depends(get_current_user)) -> User:
        if user.role != role:
            raise HTTPException(status_code=403, detail="Forbidden")
        return user
    return role_checker
```

Protect mutating endpoints:
```python
@app.post("/movies", response_model=MovieRead, status_code=201)
def create_movie(payload: MovieCreate, user: User = Depends(require_role("editor"))) -> MovieRead:
    return repository.create_movie(payload)


@app.post("/ratings", status_code=204)
def create_rating(payload: RatingCreate, user: User = Depends(require_role("editor"))) -> None:
    try:
        repository.add_rating(payload)
    except ValueError as exc:
        raise HTTPException(status_code=404, detail=str(exc)) from exc
```
(Apply similar dependencies to PUT/PATCH/DELETE routes.)

### Tests for Security
Add to `tests/test_security.py`:
```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)


def test_login_success():
    response = client.post("/token", data={"username": "teacher", "password": "classroom"})
    assert response.status_code == 200
    token = response.json()["access_token"]
    protected = client.post(
        "/movies",
        json={"title": "Secure", "year": 2024, "genre": "Drama"},
        headers={"Authorization": f"Bearer {token}"},
    )
    assert protected.status_code == 201


def test_login_failure():
    response = client.post("/token", data={"username": "teacher", "password": "wrong"})
    assert response.status_code == 401


def test_forbidden_for_viewer():
    token = client.post("/token", data={"username": "student", "password": "secret"}).json()["access_token"]
    response = client.post(
        "/movies",
        json={"title": "Forbidden", "year": 2024, "genre": "Drama"},
        headers={"Authorization": f"Bearer {token}"},
    )
    assert response.status_code == 403
```

### Secrets Handling Recap
- Remind students to add `.env` to `.gitignore`:
  ```
  .env
  ```
- Encourage generating strong secrets with `openssl rand -hex 32`.
- Stress rotating secrets if they ever leak.

## EX3 Progress Check
- Each team states current progress and blockers.
- Confirm milestone demo (Tue Jan 20) will include:
  - `docker compose up` working.
  - Advanced feature (async job, auth, or observability) at least half complete.
  - README updated with setup instructions and AI usage notes.

## Troubleshooting
- Bcrypt “work factor” errors: ensure `passlib` is installed and not conflicting with system libraries.
- If tokens never expire, check environment variable parsing and `ACCESS_TOKEN_EXPIRE_MINUTES` value.
- For invalid token errors, ensure the `Authorization` header uses `Bearer <token>` format.

## Student Success Criteria
- Successful login returns a JWT and allows authorized actions.
- Protected endpoints reject unauthenticated or low-privilege users with clear errors.
- Tests confirm authentication and authorization paths for success and failure.
- Students have verified their AWS submissions made the **Tue Dec 16, 2025** deadline (or completed the make-up plan).

## AI Prompt Kit (Copy/Paste)
- “Add JWT auth to a FastAPI app: create `/token` (OAuth2PasswordRequestForm), sign HS256 tokens with expiry, and add a dependency to protect write routes. Include tests for login success, login failure, and forbidden access for a viewer role.”
- “Write a `.env.example` for FastAPI security settings (SECRET_KEY, ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES) and a startup snippet that loads environment variables with python-dotenv.”
- “Explain risks of storing plaintext passwords and demonstrate hashing with `passlib[bcrypt]` (hash + verify) in a short code example.”

## Quick Reference (External Search / ChatGPT)
- **Google:** `FastAPI JWT tutorial python jose`
- **ChatGPT prompt:** “Explain authentication vs authorization in 100 words geared toward undergrad students.”
