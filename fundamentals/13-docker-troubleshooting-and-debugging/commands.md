# D13 — Docker Troubleshooting Commands

## Container Status

```bash
docker ps
docker ps -a
```

## Logs

```bash
docker logs <container>
docker logs -f <container>
docker logs --tail 100 <container>
docker logs -t <container>
```

## Inspect

```bash
docker inspect <container>
docker inspect --format '{{.State.Status}}' <container>
docker inspect --format '{{.State.ExitCode}}' <container>
```

## Shell

```bash
docker exec -it <container> sh
docker exec -it <container> bash
```

## Process / Environment

```bash
docker exec <container> id
docker exec <container> env
docker exec <container> ps
```

## Ports

```bash
docker port <container>
```

## Networks

```bash
docker network ls
docker network inspect <network>
docker network create d13-network
```

## Volumes

```bash
docker volume ls
docker volume inspect <volume>
```

## Resource Usage

```bash
docker stats
```

## Disk Usage

```bash
docker system df
```

## Build Debugging

```bash
docker build -t myapp .
docker build --no-cache -t myapp .
```

## Compose

```bash
docker compose ps
docker compose logs
docker compose logs backend
docker compose logs -f backend
docker compose config
```

## Test a Container Port

```bash
curl http://localhost:8080
```

## Cleanup

```bash
docker rm -f <container>
docker image prune
docker system prune
```

Use cleanup commands carefully on shared CI hosts.

## Network Lab

```bash
docker network create d13-network

docker run -d   --name d13-web   --network d13-network   nginx:alpine

docker run --rm   --network d13-network   curlimages/curl   http://d13-web
```
