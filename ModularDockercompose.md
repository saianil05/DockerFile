# Modular Docker Compose Architecture

---

# infra.yaml

```yaml
services:
  redis:
    image: redis:alpine

    volumes:
      - redis-data:/data

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s

volumes:
  redis-data:
```

---

# infra.yaml Explanation

## Purpose

`infra.yaml` contains infrastructure-related services such as:

* Redis
* Volumes
* Healthchecks

This keeps infrastructure separate from application services.

---

# `services:`

Defines containers/services.

---

# `redis:`

Creates Redis container.

---

# `image: redis:alpine`

Uses:

* Redis official image
* Alpine Linux lightweight version

Benefits:

* smaller image
* faster startup
* reduced attack surface

---

# `volumes:`

```yaml
volumes:
  - redis-data:/data
```

Mounts Docker volume:

```text
redis-data
```

to:

```text
/data
```

inside Redis container.

Redis stores persistence files there.

This allows:

* data persistence
* counter survives container recreation

---

# Persistence Flow

```text
Docker Volume
    ↓
redis-data
    ↓
Mounted To
    ↓
/data inside Redis
    ↓
Redis saves dump.rdb
```

---

# `healthcheck:`

Docker periodically checks whether Redis is healthy.

---

# `test`

```yaml
test: ["CMD", "redis-cli", "ping"]
```

Runs:

```bash
redis-cli ping
```

Expected output:

```text
PONG
```

---

# `interval: 5s`

Check Redis every:

```text
5 seconds
```

---

# `timeout: 3s`

If Redis does not respond within:

```text
3 seconds
```

healthcheck fails.

---

# `retries: 5`

Docker retries:

```text
5 times
```

before marking container unhealthy.

---

# `start_period: 10s`

Grace period:

```text
10 seconds
```

Allows Redis startup time before healthchecks begin.

---

# `volumes:`

```yaml
volumes:
  redis-data:
```

Creates named Docker volume:

```text
redis-data
```

Persistent even if containers deleted.

---

# compose.yaml

```yaml
include:
  - path: ./infra.yaml

services:
  web:
    build: .

    ports:
      - "${APP_PORT}:5000"

    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}

    depends_on:
      redis:
        condition: service_healthy

    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code

        - action: rebuild
          path: requirements.txt
```

---

# compose.yaml Explanation

## Purpose

`compose.yaml` contains:

* application service
* development workflow
* application-specific configuration

It imports infrastructure from:

```text
infra.yaml
```

---

# `include:`

```yaml
include:
  - path: ./infra.yaml
```

Imports:

* Redis service
* Redis volume
* Redis healthcheck

from separate Compose file.

---

# Why Separate Files?

Production teams separate:

* infrastructure
* applications
* monitoring
* overrides

Benefits:

* modularity
* maintainability
* cleaner architecture

---

# `web:`

Defines Flask/Python application container.

---

# `build: .`

Builds Docker image from local Dockerfile.

Equivalent:

```bash
docker build .
```

---

# `ports:`

```yaml
ports:
  - "${APP_PORT}:5000"
```

Maps:

```text
Host Port → Container Port
```

Example:

```text
8000:5000
```

Browser accesses:

```text
localhost:8000
```

---

# `environment:`

Passes environment variables into container.

---

# `REDIS_HOST`

```yaml
- REDIS_HOST=${REDIS_HOST}
```

Python app reads:

```python
os.environ.get("REDIS_HOST")
```

Usually resolves to:

```text
redis
```

---

# `REDIS_PORT`

```yaml
- REDIS_PORT=${REDIS_PORT}
```

Usually:

```text
6379
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

# `develop.watch`

Enables automatic development workflow.

Docker monitors filesystem changes.

---

# `sync+restart`

```yaml
- action: sync+restart
```

When source code changes:

1. sync files into container
2. restart container automatically

---

# `path: .`

Watch current project directory.

---

# `target: /code`

Sync files into:

```text
/code
```

inside container.

Matches Dockerfile:

```dockerfile
WORKDIR /code
```

---

# `rebuild`

```yaml
- action: rebuild
  path: requirements.txt
```

When dependencies change:

* rebuild image
* reinstall dependencies
* recreate container

Equivalent to:

```bash
docker compose build
docker compose up
```

automatically.

---

# High-Level Architecture

```text
Browser
   ↓
Web Container (Flask)
   ↓
Redis Container
   ↓
Docker Volume
```

---

# Full Request Flow

```text
User Request
     ↓
Flask App
     ↓
Redis Counter Increment
     ↓
Redis Stores Data
     ↓
Docker Volume Persists Data
```

---

# Final Understanding

This setup provides:

* modular Compose architecture
* Redis persistence
* automatic healthchecks
* service dependency management
* live development workflow
* automatic sync/rebuild
* persistent storage using Docker volumes

This is close to real production-style Docker Compose architecture.
