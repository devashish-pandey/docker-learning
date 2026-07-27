# 03. What is a Build?

## Definition
A **build** in Docker is the process of creating a **Docker image** from a set of instructions written in a **Dockerfile**. The image is essentially a snapshot/template that contains everything needed to run your application — code, runtime, libraries, environment variables, and configuration.

## The Build Command
```bash
docker build -t my-app-name:tag .
```
- `-t` → tags the image with a name (and optionally a version)
- `.` → tells Docker to look for the Dockerfile in the current directory (this is called the **build context**)

## How the Build Process Works
1. Docker reads the **Dockerfile** line by line, top to bottom.
2. Each instruction (`FROM`, `RUN`, `COPY`, etc.) creates a new **layer**.
3. Layers are cached — if a layer hasn't changed since the last build, Docker reuses it instead of rebuilding, which makes builds faster.
4. Once all instructions are processed, Docker produces a final **image**.
5. This image can then be run as a **container**, or pushed to a registry like **Docker Hub**.

## Image Layers (Why Order Matters)
Each instruction in a Dockerfile adds a layer on top of the previous one. For example:
```
FROM node:18        → Layer 1 (base OS + Node.js)
WORKDIR /app         → Layer 2
COPY package.json .  → Layer 3
RUN npm install       → Layer 4
COPY . .             → Layer 5
```

**Best practice:** Place instructions that change less often (like installing dependencies) *before* instructions that change often (like copying source code). This way, Docker can reuse cached layers and avoid re-running expensive steps like `npm install` every time you change a single line of code.

## Build vs Run — Quick Distinction
- **Build** → creates the image (the blueprint)
- **Run** → creates a container from that image (the actual running instance)

Think of it like:
> Dockerfile (recipe) → `docker build` → Image (baked cake) → `docker run` → Container (a slice being served)
