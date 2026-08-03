# 09. Docker Volumes

## What is a Volume?
A **volume** is Docker-managed storage that lives **outside** the container's writable layer, on the host machine, but is managed by Docker itself (unlike a bind mount, which points to a specific host folder you choose).

## Creating and Using a Volume
```bash
docker volume create mysql-data
docker volume ls
docker volume inspect mysql-data
```

### Running MySQL with a Volume
```bash
docker run -d -e MYSQL_USER = root MYSQL_PASSWORD = root MYSQL_ROOT_PASSWORD = dex -v mysql_new:/var/lib/mysql -p 3306:3306 mysql:8.0 .
docker exec -it <container id> bash
```

- `-v mysql-data:/var/lib/mysql` → mounts the named volume `mysql-data` to `/var/lib/mysql` inside the container, which is where MySQL stores its actual database files.
- Even if this container is removed (`docker rm mysql-db`), the data inside the `mysql-data` volume **persists**.

### Proving Persistence
```bash
docker rm -f mysql-db
docker run -d --name mysql-db -v mysql-data:/var/lib/mysql -p 3306:3306 mysql:8.0
```
The database and its data are still there — because the volume, not the container, is what's holding the actual data.

## Volumes vs Bind Mounts
| | Volume | Bind Mount |
|---|---|---|
| Managed by | Docker | You (points to a specific host path) |
| Location | Docker's internal storage area | Anywhere you specify on the host |
| Portability | More portable across environments | Tied to the host's exact file structure |
| Typical use | Databases, persistent app data | Local development (live code reload) |

## Useful Volume Commands
```bash
docker volume ls                 # List all volumes
docker volume inspect <name>     # See where it's stored + metadata
docker volume rm <name>          # Delete a volume (only if unused)
docker volume prune              # Remove all unused volumes
```
