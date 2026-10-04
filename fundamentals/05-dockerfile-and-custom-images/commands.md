# D05 — Dockerfile & Custom Image Commands

## Build

```bash
docker build -t docker-mastery-web:v1 .
```

## List Images

```bash
docker images
docker image ls
```

## Inspect Image

```bash
docker image inspect docker-mastery-web:v1
```

## View Image History

```bash
docker history docker-mastery-web:v1
```

## Tag Image

```bash
docker tag docker-mastery-web:v1 docker-mastery-web:latest
```

## Run Custom Image

```bash
docker run -d \
  --name docker-mastery-web \
  -p 8080:80 \
  docker-mastery-web:v1
```

## Check Container

```bash
docker ps
```

## View Logs

```bash
docker logs docker-mastery-web
```

## Stop and Remove

```bash
docker stop docker-mastery-web
docker rm docker-mastery-web
```

## Dockerfile Instruction Quick Reference

| Instruction | Purpose |
|---|---|
| `FROM` | Select base image |
| `RUN` | Execute build-time command |
| `COPY` | Copy files into image |
| `ADD` | Copy files with additional behavior |
| `WORKDIR` | Set working directory |
| `ENV` | Define environment variable |
| `EXPOSE` | Document container port |
| `CMD` | Define default command/arguments |
| `ENTRYPOINT` | Define primary executable |

## Example Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

## Example `.dockerignore`

```text
.git
.gitignore
node_modules
logs
*.log
.env
README.md
```
