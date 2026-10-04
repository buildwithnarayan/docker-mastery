# D09 — Docker Compose Commands

## Start

```bash
docker compose up
docker compose up -d
docker compose up --build -d
```

## Status

```bash
docker compose ps
```

## Logs

```bash
docker compose logs
docker compose logs -f
docker compose logs backend
```

## Execute

```bash
docker compose exec backend sh
```

## Build

```bash
docker compose build
```

## Stop / Remove

```bash
docker compose down
docker compose down -v
```

## Validate Compose File

```bash
docker compose config
```

## Common Workflow

```bash
docker compose up --build -d
docker compose ps
docker compose logs -f
docker compose down
```
