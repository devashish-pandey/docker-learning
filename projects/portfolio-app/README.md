# Community Portfolio App — Dockerized

Containerized [community_portfolio](https://github.com/iemafzalhassan/community_portfolio) — a Node.js-based portfolio site — using Docker.

## 📦 What This Demonstrates
- Using an official `node` image as base
- Installing dependencies and building a Node.js app inside a container
- Running a dev server via `CMD`

## Steps I Followed

```bash
git clone https://github.com/iemafzalhassan/community_portfolio.git
cd community_portfolio
vim Dockerfile
```

### Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

### Build & Run
```bash
docker build -t portfolio-app .
docker run -p 3000:3000 portfolio-app
```

## 📝 Notes / Learnings
- Running `npm run build` then `npm run dev` in the same container is a bit redundant (build creates a production bundle, dev starts a development server that usually doesn't need the build output) — this is something I want to look into: likely I only need one or the other depending on whether I want a dev environment or a production-ready build served via something like `serve` or `nginx`.
- This project helped me practice **debugging containers** — checking `docker logs <container_id>` was useful to see why the app wasn't reachable when the port mapping was wrong.

