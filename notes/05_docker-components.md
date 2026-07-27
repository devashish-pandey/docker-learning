# 05. Docker Components

Docker is made up of several core components that work together.

## 1. Docker Engine
The **core software** that runs and manages containers. It's a client-server application consisting of:
- The **Docker Daemon** (background service)
- The **Docker CLI** (command-line tool used to interact with the daemon)
- A **REST API** that programs use to talk to the daemon

## 2. Docker Daemon (`dockerd`)
The **background process** that does the actual work — building images, running containers, managing networks and volumes. It listens for API requests and manages Docker objects.

## 3. Docker CLI (Client)
The **command-line tool** (`docker`) you use to type commands like `docker build`, `docker run`, `docker ps`. The CLI sends these commands to the Docker Daemon via the REST API.

## 4. Docker Images
Read-only templates used to create containers (covered in detail in the terminology notes).

## 5. Docker Containers
Running instances of images — the actual isolated processes executing your application.

## 6. Docker Hub / Registry
The storage and distribution service for images, allowing you to pull existing images or push your own.

## 7. Docker Compose
A separate tool (bundled with Docker Desktop) for defining and managing multi-container applications via a YAML file.

## How They Work Together
```
You (Developer)
   │  type command
   ▼
Docker CLI  ──────────────►  Docker Daemon
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                Images       Containers      Networks/Volumes
                    │
                    ▼
              Docker Hub (pull/push images)
```

**Flow example:**
1. You run `docker run nginx` in the CLI.
2. CLI sends this request to the Docker Daemon.
3. Daemon checks if the `nginx` image exists locally.
4. If not, it pulls the image from **Docker Hub**.
5. Daemon creates and starts a **container** from that image.
