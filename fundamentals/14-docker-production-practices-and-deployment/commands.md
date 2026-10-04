# D14 — Docker Production Commands

## Resource Limits

```bash
docker run -d \
  --name d14-nginx \
  --memory=256m \
  --cpus=0.5 \
  nginx:alpine
```

## Monitor Resources

```bash
docker stats
docker stats d14-nginx
```

## Restart Policy

```bash
docker run -d \
  --name d14-restart \
  --restart unless-stopped \
  nginx:alpine
```

Inspect:

```bash
docker inspect \
  --format '{{.HostConfig.RestartPolicy.Name}}' \
  d14-restart
```

## Health Check

```bash
docker run -d \
  --name d14-health \
  --health-cmd="wget --spider -q http://localhost:80 || exit 1" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=3 \
  nginx:alpine
```

Inspect:

```bash
docker inspect d14-health
```

## Read-Only Root Filesystem

```bash
docker run --rm \
  --read-only \
  nginx:alpine
```

With temporary storage:

```bash
docker run --rm \
  --read-only \
  --tmpfs /tmp \
  nginx:alpine
```

## Drop Linux Capabilities

```bash
docker run \
  --cap-drop ALL \
  myapp
```

Only add required capabilities.

## Environment Configuration

```bash
docker run \
  -e DB_HOST=database \
  -e DB_PORT=5432 \
  myapp
```

## Production-Style Container

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --memory=512m \
  --cpus=1 \
  myapp:1.0.0
```

## Image Inspection

```bash
docker image inspect myapp:1.0.0
```

## Image History

```bash
docker history myapp:1.0.0
```

## Disk Usage

```bash
docker system df
```

## Cleanup

```bash
docker rm -f <container>
docker image prune
docker system prune
```

Use cleanup commands carefully on shared CI hosts.

## Non-Root Verification

```bash
docker exec <container> id
```

## Container Health Status

```bash
docker ps
docker inspect <container>
```
