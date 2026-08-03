# 10. Docker Networking 

## Why Networking Matters
By default, each container is isolated — it can't talk to another container unless Docker's networking explicitly allows it. For a real application (e.g., a Flask app that needs to talk to a MySQL database), the two containers need to be on the **same Docker network** so they can find and communicate with each other.

## Default Docker Networks
```bash
docker network ls
```
Docker ships with a few built-in networks:
- **bridge** (default) — containers can talk to each other only if explicitly connected/linked
- **host** — container shares the host's network directly (no isolation)
- **none** — no networking at all

For multi-container apps, the best practice is to create a **custom bridge network**.

## Creating a Custom Network
```bash
docker network create twotier
```

## Connecting Containers to the Same Network

### Run MySQL on the network
```bash
docker run -d --name mysql -v mysql_data_new:/var/lib/mysql --network=twotier \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=devops_db \
  -e MYSQL_USER = dev \
  -e MYSQL_PASSWORD = dev \
  - p 3306:3306 \
  mysql:8.0
```

### Run the Flask app on the same network
```bash
docker run -d \
  --name flask-app \
  --network=twotier \
  -e MYSQL_HOST=mysql-db \
  -e MYSQL_DB=devops_db \
  -p 5000:5000 \
  flask-app:latest
```

## 2-Tier Architecture (What This Looks Like)
```
        User Request
             │
             ▼
      ┌─────────────┐
      │ Flask App    │  (container: flask-app)
      │ container    │
      └──────┬───────┘
             │  app-network (custom bridge)
             ▼
      ┌─────────────┐
      │ MySQL DB     │  (container: mysql-db)
      │ container    │
      └─────────────┘
```

## Verifying Connectivity
```bash
docker exec -it flask-app sh
ping mysql-db          # should resolve and respond if on the same network
```

## Useful Networking Commands
```bash
docker network ls                     # list all networks
docker network create <name>          # create a custom network
docker network inspect <name>         # see connected containers + IPs
docker network connect <net> <cont>   # attach a running container to a network
docker network disconnect <net> <cont>
docker network rm <name>              # remove a network
```
