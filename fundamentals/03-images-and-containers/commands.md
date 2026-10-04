# D03 — Docker Images & Containers Commands

## Images

### Pull an Image

```bash
docker pull nginx
```

Downloads an image from a registry.

### List Images

```bash
docker images
```

or:

```bash
docker image ls
```

### Inspect an Image

```bash
docker image inspect nginx
```

### View Image History

```bash
docker history nginx
```

---

## Containers

### Run a Container

```bash
docker run nginx
```

### Run in Detached Mode

```bash
docker run -d nginx
```

### Run with a Name

```bash
docker run -d --name my-nginx nginx
```

### Run with Port Mapping

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

### List Running Containers

```bash
docker ps
```

### List All Containers

```bash
docker ps -a
```

### Stop a Container

```bash
docker stop my-nginx
```

### Start a Stopped Container

```bash
docker start my-nginx
```

### Restart a Container

```bash
docker restart my-nginx
```

### Remove a Container

```bash
docker rm my-nginx
```

---

## Troubleshooting

### View Logs

```bash
docker logs my-nginx
```

### Follow Logs

```bash
docker logs -f my-nginx
```

### Inspect a Container

```bash
docker inspect my-nginx
```

### Execute a Shell

```bash
docker exec -it my-nginx /bin/bash
```

Alternative:

```bash
docker exec -it my-nginx /bin/sh
```

---

## Port Mapping

Syntax:

```bash
-p HOST_PORT:CONTAINER_PORT
```

Example:

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

This maps:

```text
Host port 8080 → Container port 80
```

---

## Command Reference

| Command | Purpose |
|---|---|
| `docker pull` | Download an image |
| `docker images` | List images |
| `docker image inspect` | Inspect image metadata |
| `docker history` | View image history |
| `docker run` | Create and start a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop` | Stop a container |
| `docker start` | Start a stopped container |
| `docker restart` | Restart a container |
| `docker rm` | Remove a container |
| `docker logs` | View container logs |
| `docker exec` | Execute a command inside a running container |
| `docker inspect` | Inspect container details |
