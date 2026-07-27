# 01. Introduction & Purpose of Docker

## What is Docker?
Docker is a platform that allows developers to package an application along with all its dependencies (libraries, runtime, system tools, code) into a single unit called a **container**. This container can then run consistently on any machine, regardless of the underlying environment.

## Why Docker? (The Problem It Solves)
Before Docker, developers faced the classic problem:
> "It works on my machine!"

This happened because:
- Different OS versions
- Different installed dependencies/versions
- Different configurations between dev, test, and production environments

Docker solves this by packaging the **application + its environment** together, so it behaves the same way everywhere — your laptop, your teammate's laptop, or a production server.

## Key Benefits
- **Consistency** — same environment everywhere (dev → test → prod)
- **Isolation** — each container runs independently without interfering with others
- **Portability** — build once, run anywhere Docker is installed
- **Lightweight** — containers share the host OS kernel, so they start fast and use fewer resources than VMs
- **Scalability** — easy to spin up multiple containers of the same app for scaling

## Real-World Analogy
Think of a container like a **shipping container** (this is literally where Docker got its name and logo inspiration):
- A shipping container can carry anything — furniture, electronics, food — and it can be loaded onto any ship, train, or truck without needing to repack the contents.
- Similarly, a Docker container can carry any application and run on any system that supports Docker, without needing to reconfigure the app for that system.
