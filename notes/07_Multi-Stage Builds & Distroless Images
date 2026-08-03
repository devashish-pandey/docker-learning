# 07. Multi-Stage Builds & Distroless Images

## Multi-Stage Build
A **multi-stage build** uses multiple `FROM` instructions in one Dockerfile. Each `FROM` starts a new "stage." You build/compile in an early stage, then copy only the final artifact into a clean, minimal final stage — leaving all the build tools behind.

## Benefits of Multi-Stage Builds
- **Smaller final images** — no leftover build tools, source maps, or dev dependencies
- **Better security** — smaller attack surface, fewer packages that could have vulnerabilities
- **Cleaner separation** — build environment vs runtime environment are clearly split


## Distroless Images
**Distroless images** (from Google's `gcr.io/distroless` project) go a step further than something like `alpine` — they contain **only your application and its runtime dependencies**, with no shell, no package manager, no OS utilities at all.

### Regular base image vs Distroless
| | Regular base (e.g. `debian`, `ubuntu`) | Distroless |
|---|---|---|
| Shell (`bash`/`sh`) | ✅ Yes | ❌ No |
| Package manager (`apt`, `apk`) | ✅ Yes | ❌ No |
| Size | Larger | Much smaller |
| Attack surface | Larger (more binaries = more potential vulnerabilities) | Minimal |
| Debuggability | Easy (`docker exec -it <container> bash`) | Harder — no shell to exec into |


### Example: Java app with multi-stage build and distroless image
```dockerfile
#Stage 1
FROM python:3.9-slim AS builder
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt --target=/app/deps
                                                                                                5,1           Top 
#Stage 2
FROM gcr.io/distroless/python3-debian12
WORKDIR /work
COPY --from=builder /app/deps /work/depen
COPY --from=builder /app .
ENV PYTHONPATH="/work/depen"
EXPOSE 80
CMD["run.py"]
```


## When to Use Distroless
- **Production environments** where minimizing attack surface matters
- When you're confident in your app and don't need to shell into the container for debugging
- Security-conscious deployments (fewer CVEs to patch since there's barely anything installed)

## Trade-off to Be Aware Of
Since there's no shell, you **can't** run `docker exec -it <container> bash` to debug interactively. Debugging distroless containers usually relies on:
- Application logs (`docker logs`)
- Building a separate "debug" variant of the image temporarily
- Sidecar debugging tools in more advanced setups

## Key Takeaway
Multi-stage builds separate **build-time** concerns from **run-time** concerns.
Distroless images take minimalism to the extreme by removing everything except what your app strictly needs to run — smaller, faster, and more secure images, at the cost of easy interactive debugging.
