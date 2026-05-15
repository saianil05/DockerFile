# Docker Compose `develop.watch` Detailed Explanation with Dockerfile

---

# Dockerfile

```dockerfile
FROM python:3.12-alpine

WORKDIR /code

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["flask", "run", "--host=0.0.0.0"]
```

---

# Dockerfile Step-by-Step

---

## `FROM python:3.12-alpine`

```dockerfile
FROM python:3.12-alpine
```

Uses:

* Python 3.12
* Alpine Linux (lightweight Linux distribution)

Purpose:

* smaller Docker image
* faster pull/build time

---

## `WORKDIR /code`

```dockerfile
WORKDIR /code
```

Sets working directory inside container.

Equivalent to:

```bash
cd /code
```

All next commands run from:

```text
/code
```

---

## `COPY requirements.txt .`

```dockerfile
COPY requirements.txt .
```

Copies:

```text
requirements.txt
```

from local machine into container.

Destination:

```text
/code/requirements.txt
```

---

## `RUN pip install --no-cache-dir -r requirements.txt`

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

Installs Python dependencies.

`--no-cache-dir`:

* avoids storing pip cache
* reduces image size

---

## `COPY . .`

```dockerfile
COPY . .
```

Copies entire project into container.

Example:

```text
Local Machine
├── app.py
├── templates/
├── static/
```

becomes:

```text
Container
/code/app.py
/code/templates/
/code/static/
```

---

## `CMD ["flask", "run", "--host=0.0.0.0"]`

Starts Flask server.

`0.0.0.0` means:

```text
Accept requests from outside container
```

Without it:

* app only listens internally
* browser cannot access container

---

# Docker Compose File

```yaml
services:
  web:
    build: .

    ports:
      - "${APP_PORT:-8000}:5000"

    environment:
      - REDIS_HOST=${REDIS_HOST:-redis}
      - REDIS_PORT=${REDIS_PORT:-6379}

    depends_on:
      redis:
        condition: service_healthy

    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code
          ignore:
            - requirements.txt

        - action: rebuild
          path: requirements.txt

  redis:
    image: redis:alpine

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
```

---

# High-Level Architecture

```text
Browser
   ↓
localhost:8000
   ↓
Web Container (Flask)
   ↓
Redis Container
```

---

# `develop.watch` Detailed Explanation

---

# What is `develop.watch`?

`develop.watch` is a Docker Compose development feature used for:

* automatic file watching
* automatic sync
* automatic restart
* automatic rebuild

during local development.

---

# Purpose

Without `develop.watch`:

Every code change requires:

```bash
docker compose build
docker compose up
```

again.

Very slow.

---

# With `develop.watch`

Docker automatically:

* detects file changes
* syncs files
* restarts container
* rebuilds image if needed

---

# First Watch Rule

```yaml
- action: sync+restart
  path: .
  target: /code
  ignore:
    - requirements.txt
```

---

# `action: sync+restart`

Meaning:

```text
1. Sync changed files into container
2. Restart container automatically
```

---

# Example

Suppose you edit:

```text
app.py
```

Docker automatically:

* detects change
* copies file into `/code`
* restarts Flask container

No manual rebuild needed.

---

# `path: .`

Watch:

```text
current project directory
```

Docker monitors:

* Python files
* templates
* configs
* static files

---

# `target: /code`

Sync changed files into:

```text
/code
```

inside container.

Remember Dockerfile:

```dockerfile
WORKDIR /code
```

So application runs there.

---

# Visual Understanding

```text
Local Machine
app.py
templates/
       │
       ▼
Docker Watch detects changes
       │
       ▼
Container
/code/app.py
/code/templates/
```

---

# `ignore:`

```yaml
ignore:
  - requirements.txt
```

Means:

```text
Do NOT use sync+restart for requirements.txt
```

Why?

Because dependency changes require:

* image rebuild
* pip install rerun

Simple restart is insufficient.

---

# Second Watch Rule

```yaml
- action: rebuild
  path: requirements.txt
```

---

# `action: rebuild`

Meaning:

```text
Completely rebuild Docker image
```

Equivalent to:

```bash
docker compose build
docker compose up
```

automatically.

---

# Example

Suppose you add:

```text
pandas
```

to:

```text
requirements.txt
```

Docker automatically:

* rebuilds image
* reruns pip install
* recreates container

---

# Why Two Separate Actions?

| File Change  | Action         |
| ------------ | -------------- |
| Source code  | sync + restart |
| Dependencies | rebuild        |

Optimized development workflow.

---

# Redis Healthcheck

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
```

Docker checks:

```bash
redis-cli ping
```

Expected response:

```text
PONG
```

---

# `depends_on`

```yaml
depends_on:
  redis:
    condition: service_healthy
```

Means:

```text
Start web container ONLY after Redis becomes healthy
```

Prevents startup failures.

---

# Development Workflow

## Change Python File

```text
Edit app.py
      ↓
Docker detects change
      ↓
File synced to container
      ↓
Container restarted
```

---

## Change requirements.txt

```text
Edit requirements.txt
      ↓
Docker detects change
      ↓
Image rebuild
      ↓
Dependencies reinstalled
      ↓
Container recreated
```

---

# Why This Is Powerful

Benefits:

* faster development
* automatic syncing
* reduced rebuilds
* better developer experience
* near hot-reload workflow

---

# Important Production Note

`develop.watch` is mainly for:

```text
Local development
```

Production environments usually:

* rebuild images via CI/CD
* use immutable containers
* do not live sync source code

---

# Final Summary

Your setup creates:

```text
Code Change
    ↓
Automatic Sync
    ↓
Automatic Restart

Dependency Change
    ↓
Automatic Rebuild
    ↓
Container Recreation
```

This is a modern Docker development workflow optimized for developer productivity.
