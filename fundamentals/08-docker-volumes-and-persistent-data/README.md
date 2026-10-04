# D08 — Docker Volumes & Persistent Data

## 🎯 Objective

Understand how Docker handles data persistence.

The central problem:

> What happens to application data when a container is deleted?

By the end of this module, you should understand:

- Container writable layer
- Ephemeral container data
- Docker volumes
- Named volumes
- Bind mounts
- tmpfs mounts
- Volume lifecycle
- `docker volume` commands
- Database persistence
- Backup/restore basics
- Permission considerations
- When to use volumes vs bind mounts

---

# 1. The Container Data Problem

A container has a writable layer.

```text
Docker Image
     ↓
Container Writable Layer
     ↓
Application Data
```

If the container is removed:

```bash
docker rm container
```

data stored only in that writable layer is removed with the container.

Therefore:

> Important application data should not depend on the container writable layer.

---

# 2. Docker Volumes

A Docker volume is managed by Docker and stored outside the container's writable layer.

Conceptually:

```text
Container
    │
    │ mount
    ▼
Docker Volume
    │
    ▼
Persistent Data
```

---

# 3. List Volumes

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect my-volume
```

Create:

```bash
docker volume create my-volume
```

Remove:

```bash
docker volume rm my-volume
```

---

# 4. Named Volume

Create:

```bash
docker volume create app-data
```

Run a container:

```bash
docker run -d \
  --name volume-test \
  -v app-data:/data \
  alpine:3.22 \
  sleep 3600
```

The mapping is:

```text
app-data
   ↓
/data inside container
```

---

# 5. Write Data to the Volume

Enter:

```bash
docker exec -it volume-test /bin/sh
```

Inside:

```sh
echo "Persistent Docker data" > /data/message.txt
cat /data/message.txt
```

Exit:

```sh
exit
```

Remove the container:

```bash
docker rm -f volume-test
```

The volume still exists:

```bash
docker volume ls
```

---

# 6. Reuse the Volume

Create a new container using the same volume:

```bash
docker run --rm \
  -v app-data:/data \
  alpine:3.22 \
  cat /data/message.txt
```

You should still see:

```text
Persistent Docker data
```

This demonstrates persistence beyond the lifecycle of one container.

---

# 7. Volume Mount Syntax

Traditional short syntax:

```bash
-v app-data:/data
```

Meaning:

```text
volume
   ↓
app-data
   ↓
container path
   ↓
/data
```

Modern long syntax:

```bash
docker run --rm \
  --mount type=volume,source=app-data,target=/data \
  alpine:3.22 \
  ls -la /data
```

The `--mount` syntax is more explicit and can be easier to read for complex configurations.

---

# 8. Bind Mounts

A bind mount maps a host filesystem path into the container.

Example:

```bash
mkdir -p ~/docker-data
```

Then:

```bash
docker run --rm \
  -v ~/docker-data:/data \
  alpine:3.22 \
  sh -c 'echo "Hello from container" > /data/hello.txt'
```

Check on the host:

```bash
cat ~/docker-data/hello.txt
```

The file exists directly on your host filesystem.

---

# 9. Volume vs Bind Mount

### Named Volume

```text
Docker
  │
  ▼
Managed volume
```

Good for:

- Databases
- Application data
- Docker-managed persistence

### Bind Mount

```text
Host directory
     │
     ▼
Container directory
```

Good for:

- Local development
- Source code
- Configuration files
- Development workflows

---

# 10. tmpfs

A tmpfs mount stores data in memory rather than persistent disk storage.

Example:

```bash
docker run --rm \
  --tmpfs /tmp \
  alpine:3.22 \
  sh -c 'echo hello > /tmp/test.txt && cat /tmp/test.txt'
```

tmpfs data is temporary.

When the container stops, that data is not persisted.

---

# 11. Read-Only Mounts

A volume can be mounted read-only:

```bash
docker run --rm \
  -v app-data:/data:ro \
  alpine:3.22 \
  ls -la /data
```

With long syntax:

```bash
docker run --rm \
  --mount type=volume,source=app-data,target=/data,readonly \
  alpine:3.22 \
  ls -la /data
```

This is useful when a container only needs to read data.

---

# 12. Database Example — MySQL

A database is one of the best examples of why persistent storage matters.

Create a volume:

```bash
docker volume create mysql-data
```

Run MySQL:

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=example-password \
  -e MYSQL_DATABASE=appdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

The important mapping is:

```text
mysql-data
     ↓
/var/lib/mysql
```

The database files are stored in the volume rather than only in the container layer.

For learning only, the example password above is fine. Never use hardcoded credentials like this in production.

---

# 13. Verify the Database Container

```bash
docker ps
```

Logs:

```bash
docker logs mysql
```

Inspect volume:

```bash
docker volume inspect mysql-data
```

---

# 14. Database Persistence Test

After the database has initialized, connect to MySQL:

```bash
docker exec -it mysql mysql -uroot -pexample-password
```

Create a database/table or insert test data.

Then exit.

Remove the container:

```bash
docker rm -f mysql
```

Recreate MySQL using the same volume:

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=example-password \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

The persistent data remains because the volume survived the container deletion.

---

# 15. Volume Lifecycle

Think:

```text
docker volume create
        │
        ▼
     Volume
        │
        ├── Container A
        │
        └── Container B
```

Removing a container does not automatically remove the named volume.

This separation is intentional.

---

# 16. Anonymous Volumes

Docker can also create anonymous volumes.

They usually have generated names.

For production workflows, named volumes are generally easier to identify and manage.

---

# 17. `VOLUME` in Dockerfile

A Dockerfile can declare a volume:

```dockerfile
VOLUME ["/data"]
```

This indicates that `/data` is intended for persistent or externally managed data.

However, applications should still have a clear runtime storage strategy.

---

# 18. Volume Backup Concept

A common approach is to mount the volume into a temporary container and create an archive.

Conceptually:

```text
Docker Volume
     │
     ▼
Temporary Backup Container
     │
     ▼
tar archive
```

Example:

```bash
docker run --rm \
  -v app-data:/data \
  -v "$PWD":/backup \
  alpine:3.22 \
  tar czf /backup/app-data.tar.gz -C /data .
```

This creates:

```text
app-data.tar.gz
```

in the current host directory.

---

# 19. Volume Restore Concept

Create a volume:

```bash
docker volume create restored-data
```

Restore:

```bash
docker run --rm \
  -v restored-data:/data \
  -v "$PWD":/backup \
  alpine:3.22 \
  sh -c 'tar xzf /backup/app-data.tar.gz -C /data'
```

Verify:

```bash
docker run --rm \
  -v restored-data:/data \
  alpine:3.22 \
  ls -la /data
```

---

# 20. Permissions

One common issue with mounted data is permissions.

Example:

```text
Host file
   ↓
UID/GID
   ↓
Container process
```

A process inside the container may not have permission to write to a bind-mounted directory.

Troubleshooting commands:

```bash
ls -la
```

Inside container:

```bash
id
```

Check ownership and permissions before changing them.

Avoid blindly using:

```bash
chmod -R 777
```

especially in production.

---

# 21. Volumes and CI/CD

For Jenkins and CI/CD, understand the distinction:

```text
Build artifacts
    ↓
Usually managed by CI/CD or artifact storage

Application persistent data
    ↓
Volume / external storage

Database data
    ↓
Persistent storage
```

Don't treat container writable storage as durable application storage.

---

# 22. Docker Compose Preview

Later, Docker Compose will make multi-container applications easier.

Conceptually:

```yaml
services:
  backend:
    image: backend:v1

  mysql:
    image: mysql:8.4
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

Compose creates and manages the application topology.

We'll cover Compose in a dedicated module.

---

# 🧪 D08 Lab 1 — Named Volume

Create:

```bash
docker volume create docker-lab-data
```

Run:

```bash
docker run -d \
  --name volume-test \
  -v docker-lab-data:/data \
  alpine:3.22 \
  sleep 3600
```

Write:

```bash
docker exec volume-test sh -c 'echo "Docker-Mastery persistence" > /data/test.txt'
```

Read:

```bash
docker exec volume-test cat /data/test.txt
```

Remove container:

```bash
docker rm -f volume-test
```

Create another container:

```bash
docker run --rm \
  -v docker-lab-data:/data \
  alpine:3.22 \
  cat /data/test.txt
```

The data should still exist.

---

# 🧪 D08 Lab 2 — Bind Mount

Create:

```bash
mkdir -p ~/docker-bind-test
```

Run:

```bash
docker run --rm \
  -v ~/docker-bind-test:/data \
  alpine:3.22 \
  sh -c 'echo "Bind mount test" > /data/test.txt'
```

On host:

```bash
cat ~/docker-bind-test/test.txt
```

---

# 🧪 D08 Lab 3 — tmpfs

```bash
docker run --rm \
  --tmpfs /tmp \
  alpine:3.22 \
  sh -c 'echo "temporary" > /tmp/test.txt && cat /tmp/test.txt'
```

The data is temporary.

---

# 🧪 D08 Lab 4 — MySQL Persistence

Create:

```bash
docker volume create mysql-data
```

Run:

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=example-password \
  -e MYSQL_DATABASE=appdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

Check:

```bash
docker ps
docker logs mysql
```

Inspect:

```bash
docker volume inspect mysql-data
```

Remove:

```bash
docker rm -f mysql
```

Recreate using the same volume and verify the data remains.

---

# 🧹 Cleanup

```bash
docker rm -f volume-test mysql
```

Remove lab volumes only when you no longer need the data:

```bash
docker volume rm docker-lab-data mysql-data
```

Check:

```bash
docker volume ls
```

---

# 🎯 D08 Mental Model

Without persistent storage:

```text
Container
   │
   ▼
Writable Layer
   │
   ▼
Container deleted
   │
   ▼
Data lost
```

With a named volume:

```text
Container
   │
   ▼
Named Volume
   │
   ▼
Container deleted
   │
   ▼
Volume remains
   │
   ▼
New container
   │
   ▼
Data available
```

---

# 🔥 Kubernetes Connection

Docker:

```text
Container
   ↓
Volume
```

Kubernetes:

```text
Pod
   ↓
Volume
   ↓
PersistentVolume
   ↓
PersistentVolumeClaim
   ↓
Storage
```

The concepts become more powerful in Kubernetes, but the fundamental problem is the same:

> Separate application data from the lifecycle of the container.

---

# ❓ Knowledge Check

1. Why is data in the container writable layer considered ephemeral?
2. What is a Docker volume?
3. What is a named volume?
4. What is a bind mount?
5. What is tmpfs?
6. What is the difference between a volume and a bind mount?
7. What happens to a named volume when its container is deleted?
8. Why are volumes useful for databases?
9. What does `docker volume inspect` show?
10. How can you back up a Docker volume?
11. Why can bind mounts create permission problems?
12. Why should `chmod -R 777` generally be avoided?
13. Why shouldn't important application data be stored only in the container writable layer?
14. How does Docker volume persistence relate to Kubernetes persistent storage?
