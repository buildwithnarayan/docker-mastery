# D10 — Docker Security & Best Practices Commands

## Build

```bash
docker build -t my-image:1.0 .
```

## Run as Non-Root

```bash
docker run --rm my-image:1.0 id
```

## Read-Only Filesystem

```bash
docker run --rm --read-only nginx:alpine
```

## Read-Only + tmpfs

```bash
docker run --rm   --read-only   --tmpfs /tmp   nginx:alpine
```

## Drop Capabilities

```bash
docker run --rm   --cap-drop=ALL   nginx:alpine
```

## Resource Limits

```bash
docker run --rm   --memory=256m   --cpus=0.5   nginx:alpine
```

## Inspect Container

```bash
docker inspect <container>
```

## Runtime Statistics

```bash
docker stats
```

## Logs

```bash
docker logs <container>
```

## Dockerfile Checks

```bash
docker build -t security-demo .
docker run --rm security-demo id
```

## Cleanup

```bash
docker container prune
docker image prune
```

Use prune commands carefully.
