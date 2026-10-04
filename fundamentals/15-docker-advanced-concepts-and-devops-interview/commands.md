# D15 — Docker Advanced Commands

## Image Inspection

```bash
docker images
docker image inspect <image>
docker history <image>
```

## Build

```bash
docker build -t myapp:1.0.0 .
```

## Build Without Cache

```bash
docker build --no-cache -t myapp:1.0.0 .
```

## Tag

```bash
docker tag myapp:1.0.0 registry.example.com/myapp:1.0.0
```

## Push

```bash
docker push registry.example.com/myapp:1.0.0
```

## Pull

```bash
docker pull registry.example.com/myapp:1.0.0
```

## Containers

```bash
docker ps
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
```

## Port Mapping

```bash
docker run -d \
  --name myapp \
  -p 8080:8080 \
  myapp:1.0.0
```

## Environment Variables

```bash
docker run -d \
  -e DB_HOST=database \
  -e DB_PORT=5432 \
  myapp:1.0.0
```

## Resource Limits

```bash
docker run -d \
  --memory=512m \
  --cpus=1 \
  myapp:1.0.0
```

## Restart Policy

```bash
docker run -d \
  --restart unless-stopped \
  myapp:1.0.0
```

## Health Check

```bash
docker inspect <container>
```

## Networks

```bash
docker network ls
docker network inspect <network>
docker network create app-network
```

Connect:

```bash
docker network connect app-network <container>
```

Disconnect:

```bash
docker network disconnect app-network <container>
```

## Volumes

```bash
docker volume ls
docker volume create app-data
docker volume inspect app-data
```

## Compose

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
docker compose build
```

## Disk Usage

```bash
docker system df
```

## Cleanup

```bash
docker image prune
docker container prune
docker volume prune
docker network prune
docker system prune
```

Use cleanup commands carefully on shared environments.

## Troubleshooting Flow

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
docker network inspect <network>
docker volume inspect <volume>
```

## Example Production Run

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --memory=512m \
  --cpus=1 \
  -p 8080:8080 \
  myapp:1.0.0
```
