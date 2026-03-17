# Docker: Containers for Consistent Development and Deployment

So far you've run your backend, frontend, and PostgreSQL on your own machine. That works, but setup can differ between developers ("it works on my machine"), and deploying to a server requires installing Python, Node, PostgreSQL, and all dependencies again. **Docker** helps solve this by packaging your application and its dependencies into **containers** — lightweight, reproducible environments that run the same way on any computer or server.

This topic introduces Docker: what containers are, how images and Dockerfiles work, and how to use Docker for your fullstack project (e.g. running PostgreSQL or your whole Task Tracker stack). We focus on concepts and basic usage for development.

**Prerequisites:** Familiarity with running a backend (FastAPI), frontend (Vite/React), and PostgreSQL. Command-line basics (running scripts, environment variables).

---

## What Is Docker?

**Docker** is a platform for building, shipping, and running **containers**. A **container** is a runnable instance of an **image** — a packaged snapshot that includes an application and everything it needs (runtime, libraries, dependencies). Containers are isolated from the host system and from each other, but they share the host's kernel, which makes them lighter than full virtual machines.

**Why use Docker?**

- **Consistency** — "It works on my machine" becomes "it works in the container." Same image runs on your laptop, a colleague's PC, and a cloud server.
- **Isolation** — Each container has its own filesystem and network. You can run PostgreSQL in a container without installing it directly on your machine.
- **Reproducibility** — A `Dockerfile` or `docker-compose.yml` describes exactly how to build and run your app. Anyone with Docker can reproduce the same environment.
- **Easy cleanup** — Remove a container and its contents are gone. No leftover PostgreSQL data or Python packages cluttering your system.

---

## Key Concepts

| Concept | Meaning |
|---------|---------|
| **Image** | A read-only template. Built from a Dockerfile or pulled from a registry (e.g. Docker Hub). Contains the app and dependencies. |
| **Container** | A running instance of an image. You can create many containers from one image. |
| **Dockerfile** | A text file with instructions to build an image (e.g. base OS, install Python, copy code, run the app). |
| **Docker Compose** | A tool to define and run multi-container applications. One YAML file can start PostgreSQL, your backend, and optionally your frontend. |
| **Registry** | A store for images. Docker Hub is the default public registry (e.g. `postgres`, `python`). |

---

## Installing Docker

- **Windows / Mac:** Install [Docker Desktop](https://www.docker.com/products/docker-desktop). It includes the Docker engine, CLI, and Docker Compose.
- **Linux:** Install the Docker Engine and Docker Compose via your package manager.

After installation, run `docker --version` and `docker compose version` to confirm.

---

## Running a Pre-built Image: PostgreSQL

The simplest way to use Docker is to run an existing image. For example, to run PostgreSQL:

```bash
# Run PostgreSQL in a container; create a volume for data persistence
docker run -d \
  --name task-tracker-db \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=task_tracker \
  -p 5432:5432 \
  postgres:16
```

**What each part does:**

| Part | Purpose |
|------|---------|
| `docker run` | Create and start a container from an image |
| `-d` | Run in the background (detached) |
| `--name task-tracker-db` | Give the container a memorable name |
| `-e POSTGRES_PASSWORD=postgres` | Set environment variable (PostgreSQL password) |
| `-e POSTGRES_DB=task_tracker` | Create a database named `task_tracker` on first run |
| `-p 5432:5432` | Map host port 5432 to container port 5432 (so your backend can connect to `localhost:5432`) |
| `postgres:16` | The image name and tag (from Docker Hub) |

Your backend can now connect with `DATABASE_URL=postgresql://postgres:postgres@localhost:5432/task_tracker`.

**Useful commands:**

```bash
docker ps              # List running containers
docker stop task-tracker-db   # Stop the container
docker start task-tracker-db  # Start it again (data persists if you used a volume)
docker rm task-tracker-db     # Remove the container (add -f if still running)
```

**Data persistence:** By default, data inside the container is lost when the container is removed. To persist PostgreSQL data, add a volume:

```bash
docker run -d \
  --name task-tracker-db \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=task_tracker \
  -p 5432:5432 \
  -v task_tracker_data:/var/lib/postgresql/data \
  postgres:16
```

`-v task_tracker_data:/var/lib/postgresql/data` creates a named volume so data survives container removal.

---

## Docker Compose: Multi-Container Setup

When your app has several services (e.g. PostgreSQL + backend), **Docker Compose** lets you define and run them together in one file.

**Example: `docker-compose.yml` for Task Tracker**

```yaml
# docker-compose.yml — run PostgreSQL and optionally your backend
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: task_tracker
    ports:
      - "5432:5432"
    volumes:
      - task_tracker_data:/var/lib/postgresql/data

volumes:
  task_tracker_data:
```

**Run it:**

```bash
docker compose up -d    # Start all services in the background
docker compose down    # Stop and remove containers (volumes are kept by default)
docker compose logs -f db   # Follow database logs
```

Your backend (running on the host with `uvicorn main:app --reload`) connects to `localhost:5432` as before. The database runs in a container; the backend does not need to be in Docker for basic development.

---

## Dockerizing Your Backend (Optional)

You can package your FastAPI backend into a Docker image so it runs inside a container too. This is useful for deployment and for a fully containerized dev setup.

**Example: `Dockerfile` for the backend**

```dockerfile
# Dockerfile — build an image for the FastAPI backend
FROM python:3.12-slim

WORKDIR /app

# Install dependencies first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port for the API
EXPOSE 8000

# Run the app
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**What each instruction does:**

| Instruction | Purpose |
|-------------|---------|
| `FROM python:3.12-slim` | Start from a base image with Python 3.12 |
| `WORKDIR /app` | Set the working directory inside the container |
| `COPY requirements.txt .` | Copy dependency file into the image |
| `RUN pip install ...` | Install dependencies (cached until requirements.txt changes) |
| `COPY . .` | Copy the rest of your code |
| `EXPOSE 8000` | Document that the app uses port 8000 |
| `CMD [...]` | Command to run when the container starts |

**Build and run:**

```bash
docker build -t task-tracker-api .
docker run -d -p 8000:8000 --env-file .env task-tracker-api
```

`-t task-tracker-api` tags the image. `--env-file .env` passes environment variables (e.g. `DATABASE_URL`) into the container. Ensure `.env` has the correct database host: if the backend runs in a container and the database is in another container, use the service name (e.g. `db`) as the host, not `localhost`.

---

## Docker Compose with Backend and Database

To run both the database and backend in containers:

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: task_tracker
    ports:
      - "5432:5432"
    volumes:
      - task_tracker_data:/var/lib/postgresql/data

  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/task_tracker
    depends_on:
      - db

volumes:
  task_tracker_data:
```

Note: `DATABASE_URL` uses `db` as the host — Docker Compose creates a network where services reach each other by service name. The backend connects to the database over that network.

```bash
docker compose up -d
```

The API is available at `http://localhost:8000`. The frontend (running on the host with `npm run dev`) can call it there.

---

## Summary: Key Concepts

| Concept | Meaning |
|---------|---------|
| **Image** | Read-only template; built from a Dockerfile or pulled from a registry |
| **Container** | Running instance of an image |
| **Dockerfile** | Instructions to build an image |
| **Volume** | Persistent storage for container data |
| **Docker Compose** | Define and run multi-container apps with one file |
| **Service** | A container defined in a `docker-compose.yml` |

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Explain what Docker containers and images are and why they help with consistency.
- Run a pre-built image (e.g. PostgreSQL) with `docker run`.
- Use Docker Compose to run PostgreSQL (and optionally your backend) with one command.
- Understand the basics of a Dockerfile for a Python/FastAPI app.
- Know when to use `localhost` vs a service name for database connections (host vs containers).
