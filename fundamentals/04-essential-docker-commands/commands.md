# D04 — Essential Docker Commands

## Container Lifecycle

```bash
docker run nginx
docker run -d --name my-nginx -p 8080:80 nginx
docker ps
docker ps -a
docker start my-nginx
docker stop my-nginx
docker restart my-nginx
docker kill my-nginx
docker rm my-nginx
docker rm -f my-nginx
```

## Image Management

```bash
docker pull nginx
docker images
docker image ls
docker image inspect nginx
docker history nginx
docker rmi nginx
docker image rm nginx
```

## Troubleshooting

```bash
docker logs my-nginx
docker logs -f my-nginx
docker logs --tail 100 my-nginx
docker logs -t my-nginx
docker inspect my-nginx
docker exec -it my-nginx /bin/bash
docker exec -it my-nginx /bin/sh
docker stats my-nginx
docker top my-nginx
```

## File and Port Operations

```bash
docker cp test.txt my-nginx:/tmp/test.txt
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf
docker port my-nginx
docker diff my-nginx
docker rename old-name new-name
```

## Pause and Resume

```bash
docker pause my-nginx
docker unpause my-nginx
```

## Disk Usage and Cleanup

```bash
docker system df
docker container prune
docker image prune
docker network prune
docker builder prune
docker system prune
```

Use cleanup commands carefully, especially on shared CI/CD machines.

## Command Reference

| Command | Purpose |
|---|---|
| `docker run` | Create and start a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker start` | Start a stopped container |
| `docker stop` | Gracefully stop a container |
| `docker restart` | Restart a container |
| `docker kill` | Immediately terminate a container |
| `docker rm` | Remove a container |
| `docker pull` | Download an image |
| `docker images` | List images |
| `docker rmi` | Remove an image |
| `docker logs` | View container logs |
| `docker exec` | Execute a command inside a running container |
| `docker inspect` | Inspect an object |
| `docker stats` | Monitor resource usage |
| `docker top` | View processes |
| `docker cp` | Copy files between host and container |
| `docker port` | Show port mappings |
| `docker diff` | Show filesystem changes |
| `docker system df` | Show Docker disk usage |
| `docker prune` | Remove unused resources |
