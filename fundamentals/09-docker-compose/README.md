# D09 — Docker Compose

## Objective

Learn how to define and run multi-container applications with Docker Compose.

Topics:
- compose.yaml
- Services
- Service-to-service DNS
- Ports
- Environment variables
- Volumes
- Networks
- depends_on
- Health checks
- Build with Compose
- Logs and lifecycle
- Compose and Kubernetes concepts

## Basic Compose

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Run:

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

Open:

```text
http://localhost:8080
```

## Multiple Services

```yaml
services:
  backend:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - database

  database:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: example-password
      MYSQL_DATABASE: appdb
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

The service name `database` becomes the internal DNS name.

For example:

```text
DB_HOST=database
DB_PORT=3306
```

Do not hardcode container IP addresses.

## Common Commands

```bash
docker compose up
docker compose up -d
docker compose up --build -d
docker compose ps
docker compose logs
docker compose logs -f
docker compose logs backend
docker compose exec backend sh
docker compose build
docker compose down
docker compose down -v
```

## Build from a Dockerfile

```yaml
services:
  backend:
    build:
      context: ./backend
    ports:
      - "8080:80"
```

## Networks

```yaml
services:
  backend:
    image: backend:v1
    networks:
      - app-network

  database:
    image: mysql:8.4
    networks:
      - app-network

networks:
  app-network:
```

## Environment Files

`.env`:

```text
MYSQL_ROOT_PASSWORD=example-password
MYSQL_DATABASE=appdb
```

Compose:

```yaml
services:
  database:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
```

Never commit real production secrets to Git.

## Health Checks

`depends_on` controls startup ordering but does not guarantee that an application is ready.

Use health checks and application retry logic for reliable systems.

## Lab

Create a directory:

```bash
mkdir d09-app
cd d09-app
```

Create `compose.yaml` with a backend and MySQL database, then run:

```bash
docker compose up -d
docker compose ps
docker compose logs
```

Test persistence:

```bash
docker compose down
docker compose up -d
```

Then compare with:

```bash
docker compose down -v
```

## Docker Compose → Kubernetes

Conceptually:

```text
Compose service
    ↓
Kubernetes Deployment

Compose DNS/network
    ↓
Kubernetes Service/DNS

Compose volume
    ↓
PersistentVolume / PersistentVolumeClaim

Compose environment
    ↓
ConfigMap / Secret
```

## Knowledge Check

1. What problem does Docker Compose solve?
2. What is `compose.yaml`?
3. What is a Compose service?
4. How does one service reach another?
5. What DNS name does a service named `database` provide?
6. What does `docker compose up -d` do?
7. What does `docker compose down` remove?
8. What is the difference between `down` and `down -v`?
9. What does `depends_on` do?
10. Does `depends_on` guarantee application readiness?
11. Why use health checks?
12. Why use named volumes for databases?
13. What does `build:` do?
14. What is `.env` used for?
15. Why should production secrets not be committed?
16. How is Compose different from multiple `docker run` commands?
17. How does Compose prepare you for Kubernetes?
