# 06. Dockerfile Anatomy & Build vs Pull

## What a Dockerfile Usually Contains

A Dockerfile is a step-by-step recipe for building an image. Here are the common instructions, in the order they're typically used:

### 1. `FROM` — Base Image (OS)
Specifies the starting point — usually a minimal OS or a pre-built runtime image.
```dockerfile
FROM ubuntu:22.04
# or
FROM node:18-alpine
```
Every Dockerfile must start with a `FROM` instruction (except in advanced multi-stage builds).

### 2. `WORKDIR` — Working Directory
Sets the working directory inside the container for any following instructions (`RUN`, `COPY`, `CMD`, etc.). Creates the directory if it doesn't exist.
```dockerfile
WORKDIR /app
```

### 3. `COPY` — Copy Files
Copies files/folders from your local machine (build context) into the container's filesystem.
```dockerfile
COPY package.json .
COPY . .
```

### 4. `RUN` — Execute Commands at Build Time
Runs a command **during the image build process** — typically used to install dependencies. Each `RUN` creates a new layer.
```dockerfile
RUN npm install
RUN apt-get update && apt-get install -y curl
```

### 5. `EXPOSE` — Document the Port
Informs Docker (and other developers) which port the application listens on inside the container. **Note:** this doesn't actually publish the port — it's documentation. You still need `-p` when running the container to map it to the host.
```dockerfile
EXPOSE 3000
```

### 6. `CMD` — Default Command
Specifies the **default command to run when the container starts**. Unlike `RUN`, this executes at **container runtime**, not build time. There can only be one `CMD` per Dockerfile (the last one wins if multiple are specified).
```dockerfile
CMD ["node", "app.js"]
```

## Putting It All Together (Example)
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

## `RUN` vs `CMD` — Common Confusion
| | RUN | CMD |
|---|---|---|
| Executes at | Build time | Container start time |
| Purpose | Install packages, set up environment | Define the default startup command |
| How many | Multiple allowed | Only one takes effect |

---

## Build vs Pull — Two Ways to Get an Image

There are two ways to get a Docker image to run as a container:

### Option 1: Build it yourself
Write your own **Dockerfile** and build a custom image tailored to your application.
```bash
docker build -t my-custom-app .
```
Use this when you're containerizing **your own application**.

### Option 2: Pull a pre-built image from Docker Hub
Many common tools/software already have official images published on **Docker Hub**. Instead of writing a Dockerfile from scratch, you just pull and run them.
```bash
docker pull nginx
docker run nginx
```
Use this when you need standard, well-maintained software like `nginx`, `mysql`, `redis`, `postgres`, etc.

### When to Use Which
| Scenario | Approach |
|---|---|
| Running your own app/code | **Build** using a Dockerfile |
| Using standard software (nginx, databases, etc.) | **Pull** from Docker Hub |
| Customizing an existing image (e.g., nginx + your config) | **Build**, using that image as your `FROM` base |

**Example combining both:** In my `nginx-app` project, I used `FROM nginx` as the base image (pulled automatically during build) and then `COPY`-ed my custom `index.html` / `nginx.conf` into it — this shows how build and pull often work together rather than being an either/or choice.
