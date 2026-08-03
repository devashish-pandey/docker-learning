# 11. Docker Compose
Tool for defining  & running multi-container Docker application using YAML config file.
## The Problem Compose Solves
Running a multi-container app manually means a lot of repetitive, error-prone commands:
```bash
docker network create app-network
docker volume create mysql-data
docker run -d --name mysql-db --network app-network -v mysql-data:/var/lib/mysql -e MYSQL_DATABASE=devops_db mysql:8.0
docker run -d --name flask-app --network app-network -e MYSQL_HOST=mysql-db -p 5000:5000 flask-app:latest
```
Docker Compose lets you define **all of this in a single YAML file** and bring the whole stack up/down with one command.

## Basic Structure of `docker-compose.yml`
```yaml
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    environment:
      MYSQL_PASSWORD: dev
      MYSQL_USER: dev
      MYSQL_ROOT_PASSWORD: dev
      MYSQL_DATABASE: devops_db
    volumes:
      - ./mysql_data_new:/var/lib/mysql
    networks:
      - twotier
    ports:
      - "3307:3306"
    healthcheck:
      test: ["CMD","mysqladmin","ping","-h","localhost","-udev","-pdev"]
      interval: 10s
      retries: 5
      start_period: 60s
      timeout: 5s

  flask-app:
    build:
      context: .
    container_name: flask-app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: dev
      MYSQL_PASSWORD: dev
      MYSQL_ROOT_PASSWORD: dev
      MYSQL_DB: devops_db

    networks:
      - twotier
    ports:
      - "5000:5000"
    depends_on:
      mysql:
        condition: service_healthy

volumes:
  mysql_data_new:

networks:
  twotier:

```

## Key Sections Explained
| Section | Purpose |
|---|---|
| `services` | Each service = one container (app, database, cache, etc.) |
| `build` | Build an image from a local Dockerfile (instead of `image:`) |
| `image` | Use a pre-built image (e.g., pulled from Docker Hub) |
| `ports` | Host:container port mapping (same as `-p` in `docker run`) |
| `environment` | Environment variables passed into the container |
| `depends_on` | Controls startup order (does NOT wait for the service to be "ready", just "started") |
| `volumes` (top-level) | Named volumes for persistent data |
| `networks` (top-level) | Custom networks so services can resolve each other by name |

## Compose Automatically Handles Networking
Docker Compose creates a network for your project by default, and every service in the file can reach every other service **by its service name** — this is why in the example above, Flask can connect using `MYSQL_HOST=mysql-db` without any manual `docker network create` step.

## Common Compose Commands
```bash
docker compose up              # create and start all services
docker compose up -d           # detached mode (background)
docker compose up --build      # rebuild images before starting
docker compose down            # stop and remove containers + network
docker compose down -v         # also remove volumes (⚠️ deletes persisted data)
docker compose ps              # list running services
docker compose logs -f         # follow logs of all services
docker compose logs -f flask-app   # follow logs of one specific service
docker compose exec flask-app sh   # open a shell inside a running service
docker compose build           # build/rebuild images without starting
```

## `depends_on`
`depends_on` only waits for the container to **start**, not for the actual service inside (e.g., MySQL) to be **ready to accept connections**. This is a very common source of "connection refused" errors on first `docker compose up` — the app container starts before MySQL has finished initializing.

**Common fixes:**
- Add retry/wait logic in your app's connection code
- Use a wait-for-it / healthcheck script
- Define a proper `healthcheck` in Compose:
```yaml
mysql-db:
  image: mysql:8.0
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
    interval: 5s
    retries: 5

flask-app:
  depends_on:
    mysql-db:
      condition: service_healthy
```
