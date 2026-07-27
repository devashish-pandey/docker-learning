# Java Quotes App — Dockerized

Containerized [java-quotes-app](https://github.com/LondheShubham153/java-quotes-app) — a simple Java console/quotes application — using Docker.

## 📦 What This Demonstrates
- Using `eclipse-temurin` — an official OpenJDK-based image — as the base
- Compiling Java source code **inside the container** at build time (`RUN javac`)
- Running a compiled `.class` file as the container's entrypoint

## Steps I Followed

```bash
git clone https://github.com/LondheShubham153/java-quotes-app.git
cd java-quotes-app
vim Dockerfile
```

### Dockerfile
```dockerfile
FROM eclipse-temurin:17-jdk
WORKDIR /app
COPY . .
RUN javac Main.java
EXPOSE 8000
CMD ["java", "Main"]
```

### Build & Run
```bash
docker build -t java-app:latest .
docker run -p 8000:8000 java-app:latest
```

## 📝 Notes / Learnings
- `eclipse-temurin:17-jdk` is a widely-used, community-trusted OpenJDK distribution image (successor to the old AdoptOpenJDK images) — good to know it exists as an alternative to `openjdk`.
- Here, compilation (`javac`) happens **inside the Docker build step** rather than compiling locally first — this means the container image includes the full JDK (larger image), which is fine for learning but not ideal for production (a multi-stage build would compile with the JDK, then run with just a JRE for a smaller final image).
- `EXPOSE 8000` documents the port; `-p 8000:8000` in `docker run` is what actually maps it to the host.
<img width="846" height="273" alt="image" src="https://github.com/user-attachments/assets/cc8be06b-d093-4faa-bf3b-5252c304b38b" />
