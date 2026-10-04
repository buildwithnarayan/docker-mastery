# D08 — Docker Volumes & Persistent Data Commands

## Volume Management

```bash
docker volume ls
docker volume create app-data
docker volume inspect app-data
docker volume rm app-data
```

## Named Volume

```bash
docker run -d \
  --name volume-test \
  -v app-data:/data \
  alpine:3.22 \
  sleep 3600
```

## Write Data

```bash
docker exec volume-test sh -c 'echo "Persistent data" > /data/test.txt'
docker exec volume-test cat /data/test.txt
```

## Remove Container

```bash
docker rm -f volume-test
```

## Reuse Volume

```bash
docker run --rm \
  -v app-data:/data \
  alpine:3.22 \
  cat /data/test.txt
```

## `--mount` Syntax

```bash
docker run --rm \
  --mount type=volume,source=app-data,target=/data \
  alpine:3.22 \
  ls -la /data
```

## Bind Mount

```bash
mkdir -p ~/docker-data

docker run --rm \
  -v ~/docker-data:/data \
  alpine:3.22 \
  sh -c 'echo "Hello" > /data/hello.txt'
```

## tmpfs

```bash
docker run --rm \
  --tmpfs /tmp \
  alpine:3.22 \
  sh -c 'echo hello > /tmp/test.txt && cat /tmp/test.txt'
```

## Read-only Volume

```bash
docker run --rm \
  -v app-data:/data:ro \
  alpine:3.22 \
  ls -la /data
```

## MySQL Example

```bash
docker volume create mysql-data

docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=example-password \
  -e MYSQL_DATABASE=appdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8.4
```

## Volume Backup

```bash
docker run --rm \
  -v app-data:/data \
  -v "$PWD":/backup \
  alpine:3.22 \
  tar czf /backup/app-data.tar.gz -C /data .
```

## Volume Restore

```bash
docker volume create restored-data

docker run --rm \
  -v restored-data:/data \
  -v "$PWD":/backup \
  alpine:3.22 \
  sh -c 'tar xzf /backup/app-data.tar.gz -C /data'
```

## Cleanup

```bash
docker volume rm app-data
docker volume prune
```

Use `docker volume prune` carefully because it removes unused volumes.
