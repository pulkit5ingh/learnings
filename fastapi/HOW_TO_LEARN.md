# How to Learn from This Curriculum

## The Learning Loop

For every assignment and project, follow this four-step loop:

1. **Read** the entire file before writing a single line of code. Understand the goal and what the deliverables actually mean.
2. **Build** working code. Do not copy-paste snippets blindly — type them, understand each line.
3. **Break it** intentionally. Change a parameter, remove a dependency, break a query. See what the error says.
4. **Check** each deliverable. Only mark a checkbox when you can demo it working.

---

## Environment Setup

### Required Tools
```bash
# Python 3.11+ (use pyenv to manage versions)
pyenv install 3.11.9
pyenv global 3.11.9

# Verify
python --version  # Python 3.11.x

# uv (fast package manager, use instead of pip)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Docker Desktop — install from docker.com

# PostgreSQL client (for debugging)
brew install postgresql  # macOS

# Redis CLI
brew install redis       # macOS
```

### Project Template (use for every assignment)
```bash
mkdir my-project && cd my-project

# Create virtual env with uv
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows

# Create pyproject.toml
uv init --no-workspace

# Install common deps
uv add fastapi uvicorn[standard] pydantic

# Run dev server
uvicorn main:app --reload
```

### Recommended `pyproject.toml` Base
```toml
[project]
name = "fastapi-project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.110.0",
    "uvicorn[standard]>=0.29.0",
    "pydantic>=2.6.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "httpx>=0.27.0",
    "ruff>=0.4.0",
    "mypy>=1.9.0",
]

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]

[tool.mypy]
python_version = "3.11"
strict = true
```

---

## How Assignments Work

Each assignment (`A{phase}.{num}`) is a focused deep-dive into one topic. Assignments are not projects — they are learning exercises. A typical assignment takes 2–4 hours.

**What to do:**
- Read the Background section fully.
- Work through the Tasks in order. Each task is a small, runnable program.
- You should be able to run every code snippet and see the expected output.
- Complete the Deliverables checklist.

**Red flags that you need to slow down:**
- You are copy-pasting without understanding.
- You cannot explain a line of code without re-reading it.
- A snippet works but you don't know why.

---

## How Projects Work

Each project (`P{phase}.{num}`) is an application that uses the skills from the phase. Projects take 4–10 hours depending on scope.

**What to do:**
- Study the Architecture section — understand the file structure before coding.
- Implement using the code stubs as starting points, not as final code.
- Add tests (even if the assignment does not say to — it is good practice).
- Run the project end-to-end: start the server, hit endpoints with curl or a REST client.

---

## Tooling Habits

### Always use ruff for linting
```bash
ruff check .
ruff format .
```

### Always use mypy for type checking
```bash
mypy .
```

### Run tests before considering anything done
```bash
pytest -v --tb=short
```

### Use httpx or curl to test endpoints manually
```bash
# With httpx in Python
import httpx
r = httpx.get("http://localhost:8000/health")
print(r.json())

# With curl
curl -X POST http://localhost:8000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "test", "price": 9.99}'
```

---

## Common Mistakes to Avoid

### 1. Mixing sync and async
```python
# BAD — blocking call inside async function
async def get_user(user_id: int):
    time.sleep(1)  # blocks the entire event loop
    ...

# GOOD
async def get_user(user_id: int):
    await asyncio.sleep(1)  # yields to event loop
    ...
```

### 2. Not using response_model
```python
# BAD — leaks internal fields (like hashed_password)
@app.get("/users/{user_id}")
async def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.get(User, user_id)

# GOOD
@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int, db: Session = Depends(get_db)):
    return db.get(User, user_id)
```

### 3. Creating a new DB session per query instead of per request
```python
# BAD
async def get_item(item_id: int):
    async with AsyncSession(engine) as session:  # new session every call
        ...

# GOOD — use Depends() to get one session per request
async def get_db():
    async with AsyncSession(engine) as session:
        yield session

@app.get("/items/{item_id}")
async def get_item(item_id: int, db: AsyncSession = Depends(get_db)):
    ...
```

### 4. Ignoring validation errors
Never return a 500 for a validation error. FastAPI returns 422 automatically, but you should understand what it means and test for it.

### 5. Hardcoding secrets
```python
# BAD
DATABASE_URL = "postgresql://user:password@localhost/mydb"

# GOOD
from app.config import settings
DATABASE_URL = settings.database_url
```

---

## Phase Progression

Do not skip phases. Each phase assumes the previous one. If you find Phase 2 hard because of async Python, go back and redo A1.1.

### Minimum Viable Completion per Phase
Before moving to the next phase, you must be able to:

| Phase | Gate Check |
|-------|-----------|
| 1 | Explain `async def` vs `def`, write a FastAPI route with Pydantic validation, use `Depends()` |
| 2 | Write an async SQLAlchemy query, run an Alembic migration, use repository pattern |
| 3 | Implement JWT login endpoint, protect a route, add a role check |
| 4 | Write a pytest fixture, use `override_dependency`, get 80%+ coverage on a small API |
| 5 | Send a task to Celery, cache a response in Redis, add rate limiting |
| 6 | Version an API, write a custom error handler, configure CORS correctly |
| 7 | Publish a RabbitMQ message, consume it in another service |
| 8 | Build a Docker image, write a GitHub Actions workflow that runs tests |
| 9 | Implement a WebSocket endpoint, add Prometheus metrics to a route |

---

## Resources

- FastAPI docs: https://fastapi.tiangolo.com
- Pydantic v2 docs: https://docs.pydantic.dev/latest/
- SQLAlchemy 2.0 docs: https://docs.sqlalchemy.org/en/20/
- Alembic docs: https://alembic.sqlalchemy.org/en/latest/
- pytest docs: https://docs.pytest.org/en/stable/
- Celery docs: https://docs.celeryq.dev/en/stable/
- aio-pika docs: https://aio-pika.readthedocs.io/en/latest/
- Strawberry GraphQL: https://strawberry.rocks/docs
- OpenTelemetry Python: https://opentelemetry.io/docs/languages/python/

---

## A Note on AI Assistance

Use AI tools (Claude, Copilot, etc.) to:
- Explain concepts you do not understand.
- Review your code for issues.
- Suggest better approaches.

Do NOT use AI tools to:
- Generate your assignment code for you without understanding it.
- Skip the debugging step.

The goal is that after this curriculum, you can build a production FastAPI service without AI assistance — and use AI to make you 10x faster, not as a crutch.
