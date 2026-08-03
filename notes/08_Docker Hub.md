# 08. Docker Hub — Push, Pull & Tag

## What is Docker Hub?
Docker Hub is the default **public registry** for Docker images — a place to store, share, and distribute images, similar to how GitHub hosts code repositories.

## Pulling an Image
Downloads an image from Docker Hub to your local machine.
```bash
docker pull nginx
docker pull nginx:1.25       # specific version
docker pull mysql:8.0
```
If no tag is specified, Docker defaults to `:latest`.

## Tagging an Image
Before you can push an image to your own Docker Hub account, it must be tagged in the format:
```
<dockerhub-username>/<repository-name>:<tag>
```

```bash
docker tag python-mini-app:latest <username>/python-mini-app:latest
```
- This doesn't create a new image — it just adds another name/label pointing to the same image ID.
- You can check with `docker images` — both tags will show the same **Image ID**.

## Pushing an Image
Uploads your locally built (and tagged) image to Docker Hub.

```bash
docker login                                  # authenticate first
docker push <username>/python-mini-app:latest
```

### Full Push Workflow
```bash
docker build -t python-mini-app.
docker tag  python-mini-app <username>/ python-mini-app:latest.
docker login
docker push <username>/python-mini-app:latest
```

## Verifying
After pushing, the image is visible on your Docker Hub profile at:
```
https://hub.docker.com/r/<username>/<repository-name>
```
Anyone can now pull it:
```bash
docker pull <username>/python-mini-app:latest

## Key Concepts Recap
| Command | Purpose |
|---|---|
| `docker pull` | Download an image from a registry |
| `docker tag` | Give an image a new name/label (needed before pushing to your own repo) |
| `docker push` | Upload a tagged image to a registry |
| `docker login` | Authenticate with Docker Hub (or another registry) before pushing |
