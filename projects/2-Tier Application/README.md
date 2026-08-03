# Two-Tier Flask + MySQL App — Docker Compose

A 2-tier application (Flask + MySQL) orchestrated entirely with Docker Compose — including a proper **healthcheck** so the app waits for MySQL to actually be ready, not just started.

## 📦 docker-compose.yml
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
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-udev", "-pdev"]
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

## 🏗️ Architecture
```
        User
         │
         ▼
  ┌─────────────┐
  │ flask-app    │  container, port 5000
  └──────┬───────┘
         │  twotier network (resolves "mysql" by name)
         ▼
  ┌─────────────┐
  │ mysql        │  container, port 3307→3306, mysql:5.7
  └─────────────┘
```

## 🚀 How to Run
```bash
docker compose up -d
docker compose ps
```

Tear down:
```bash
docker compose down          # stop + remove containers/network
docker compose down -v       # also remove the named volume (if you switch to one — see note below)
```

---

## 🔑 Key Things This Compose File Gets Right

### 1. Matching database name on both sides
```yaml
mysql:
  environment:
    MYSQL_DATABASE: devops_db
flask-app:
  environment:
    MYSQL_DB: devops_db
```
Same value, correct variable name on each side — this was the exact bug from my earlier version of this project (Flask expected `MYSQL_DB`, MySQL expects `MYSQL_DATABASE`; if they don't both point to the same DB name, you get an access-denied error).

### 2. Container name as hostname
```yaml
flask-app:
  environment:
    MYSQL_HOST: mysql
```
Both services share the `twotier` network, so Flask can reach MySQL using the **service/container name** (`mysql`) instead of an IP address — Docker's built-in DNS resolves it.

### 3. Real health-based startup ordering
```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-udev", "-pdev"]
  interval: 10s
  retries: 5
  start_period: 60s
  timeout: 5s

flask-app:
  depends_on:
    mysql:
      condition: service_healthy
```
Plain `depends_on` only waits for a container to **start**, not for MySQL to actually be ready to accept connections — a very common cause of "connection refused" on the first `docker compose up`. Here, `condition: service_healthy` makes `flask-app` wait until `mysqladmin ping` actually succeeds inside the `mysql` container before starting. `start_period: 60s` gives MySQL a grace period to initialize on first run without counting early failures against the retry limit.

---

## ⚠️ A Subtlety Worth Knowing: Bind Mount vs Named Volume
This file declares a named volume:
```yaml
volumes:
  mysql_data_new:
```
But the `mysql` service actually mounts:
```yaml
volumes:
  - ./mysql_data_new:/var/lib/mysql
```
Because the source starts with `./`, Compose treats this as a **bind mount** to a folder in your project directory — **not** the named volume declared at the bottom. The named volume declaration is valid YAML but currently unused.

**Both approaches work fine for persistence**, but they behave differently:
| | Bind mount (`./mysql_data_new`) | Named volume (`mysql_data_new:`) |
|---|---|---|
| Data location | Visible in your project folder | Managed internally by Docker |
| Portability | Tied to this exact host path | Portable across environments |
| `docker compose down -v` | Does NOT delete it (not Docker-managed) | Deletes it |

If the intent was a named volume, the fix is:
```yaml
mysql:
  volumes:
    - mysql_data_new:/var/lib/mysql   # no "./" prefix
```
If the intent was a bind mount (e.g., to inspect MySQL's data files directly on the host), the current setup is correct as-is — just remove the unused top-level `volumes:` block, or keep it for future flexibility.

---

## 🧰 Useful Commands for This Setup
```bash
docker compose up -d
docker compose ps
docker compose logs -f mysql
docker compose logs -f flask-app
docker inspect --format='{{json .State.Health}}' mysql
docker exec -it mysql mysql -udev -pdev devops_db
docker compose down
docker compose down -v
```

<!-- ## 📌 Key Takeaway 
Getting a 2-tier app running isn't just "does it start" — it's making sure the database is actually *ready* before the app connects (`healthcheck` + `condition: service_healthy`), that both services agree on the same environment variable values, and understanding exactly where your persistent data lives (bind mount vs named volume) so you don't lose it — or think you did — by accident.
-->
