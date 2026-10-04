# D13 — Docker Troubleshooting & Debugging

## Objective

Learn a systematic approach to troubleshooting Docker containers, images, networking, volumes, Compose, builds, logs, and resource problems.

## Troubleshooting Method

```text
Problem
   ↓
Is the container running?
   ↓
Check status
   ↓
Check logs
   ↓
Inspect container
   ↓
Check processes
   ↓
Check networking
   ↓
Check volumes
   ↓
Check resources
   ↓
Find root cause
```

## Essential Commands

```bash
docker ps
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
docker network ls
docker network inspect <network>
docker volume ls
docker volume inspect <volume>
docker system df
```

For Compose:

```bash
docker compose ps
docker compose logs
docker compose config
```

## Container Status

```bash
docker ps
docker ps -a
```

`docker ps` shows running containers. `docker ps -a` also shows stopped containers.

Example:

```text
Exited (1)
```

Check the logs:

```bash
docker logs <container>
```

## Container Logs

Follow logs:

```bash
docker logs -f <container>
```

Last 100 lines:

```bash
docker logs --tail 100 <container>
```

With timestamps:

```bash
docker logs -t <container>
```

## Exit Codes

A common convention is:

```text
0 = successful exit
non-zero = failure or abnormal termination
```

Always combine the exit code with logs and application behavior.

Inspect the exit code:

```bash
docker inspect --format '{{.State.ExitCode}}' <container>
```

## Why Containers Exit

A container normally remains alive while its main process is running.

For example:

```dockerfile
CMD ["echo", "Hello"]
```

prints the message and exits, so the container stops.

A long-running service must have an appropriate foreground process.

## Docker Inspect

```bash
docker inspect <container>
```

Useful information includes:

- Image
- Command
- Environment
- Mounts
- Networks
- IP address
- Port configuration
- Restart policy
- State

Specific values can be queried:

```bash
docker inspect --format '{{.State.Status}}' <container>
docker inspect --format '{{.State.ExitCode}}' <container>
```

## Enter a Running Container

```bash
docker exec -it <container> sh
```

If Bash is available:

```bash
docker exec -it <container> bash
```

Minimal images such as Alpine commonly provide `sh` rather than Bash.

## exec vs attach

`docker exec` starts a new process inside the container:

```bash
docker exec -it <container> sh
```

`docker attach` connects to the existing main process:

```bash
docker attach <container>
```

For debugging, `exec` is usually safer.

## Port Troubleshooting

Port syntax:

```text
-p HOST_PORT:CONTAINER_PORT
```

Example:

```bash
docker run -d -p 8080:80 nginx:alpine
```

Host port:

```text
8080
```

Container port:

```text
80
```

Check mappings:

```bash
docker port <container>
```

If an application listens on port 8080 inside the container, map it correctly:

```bash
docker run -d -p 8080:8080 <image>
```

## Application Binding

An application listening only on:

```text
127.0.0.1:8080
```

may not be reachable from outside the container.

Containerized applications commonly need to listen on:

```text
0.0.0.0:8080
```

Test from inside the container:

```bash
curl http://localhost:8080
```

Then test from the host:

```bash
curl http://localhost:8080
```

If it works inside but not outside, investigate port mapping and networking.

## Docker Networks

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect <network>
```

Create a custom network:

```bash
docker network create d13-network
```

Run a container on it:

```bash
docker run -d   --name web   --network d13-network   nginx:alpine
```

Containers on a user-defined network can normally reach one another using container/service names.

Prefer:

```text
http://backend:8080
```

instead of hardcoding a container IP.

## Environment Variables

Inspect environment variables:

```bash
docker exec <container> env
```

Or inspect the container:

```bash
docker inspect <container>
```

Example:

```bash
docker run   -e DB_HOST=db   -e DB_PORT=5432   myapp
```

A missing environment variable can cause application startup or connection failures.

## Volumes

List volumes:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect <volume>
```

Check container mounts:

```bash
docker inspect <container>
```

Common volume problems include:

- Wrong host path
- Wrong container path
- Read-only mount
- Permission problems
- Missing expected data

## Permission Problems

Check the container user:

```bash
docker exec <container> id
```

Check filesystem permissions:

```bash
docker exec <container> ls -la
```

Avoid running applications as root unless there is a specific requirement.

## Resource Troubleshooting

Live resource usage:

```bash
docker stats
```

Monitor:

- CPU
- Memory
- Network I/O
- Block I/O
- PIDs

High CPU or memory usage may indicate application bugs, leaks, excessive workload, or configuration issues.

## Disk Troubleshooting

```bash
docker system df
```

Docker hosts can consume disk space through:

- Old images
- Stopped containers
- Build cache
- Unused volumes
- Large logs

Cleanup carefully:

```bash
docker image prune
docker system prune
```

Do not use aggressive cleanup blindly on shared CI agents.

## Docker Build Failures

Run:

```bash
docker build -t myapp .
```

Read the first meaningful error.

For cache-related diagnosis:

```bash
docker build --no-cache -t myapp .
```

`--no-cache` is a diagnostic option, not a default fix.

## Build Context

In:

```bash
docker build -t myapp .
```

the final `.` is the build context.

Dockerfile `COPY` instructions can only access files available within the build context and not excluded by `.dockerignore`.

## .dockerignore

Example:

```text
.git
.gitignore
node_modules
target
*.log
.env
```

Benefits:

- Smaller build context
- Faster builds
- Avoid accidental inclusion of secrets
- Better caching behavior

## Image Filesystem Debugging

Start a shell:

```bash
docker run --rm -it <image> sh
```

Then inspect:

```bash
ls -la
find /app
```

Compare the actual filesystem with Dockerfile `COPY` and `WORKDIR` instructions.

## Docker Compose Troubleshooting

Check services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Specific service:

```bash
docker compose logs backend
```

Follow:

```bash
docker compose logs -f backend
```

Validate/resolve configuration:

```bash
docker compose config
```

Remember:

```text
Container started != application ready
```

A dependent service may start before its dependency is actually ready.

## Restart Loops

If a container constantly restarts:

```text
Start
 ↓
Crash
 ↓
Restart
 ↓
Crash
```

Check:

```bash
docker ps
docker logs <container>
docker inspect <container>
```

Do not use restart policies to hide application failures.

## Labs

### Lab 1 — Broken Container

```bash
docker run -d   --name d13-broken   alpine   sh -c "echo 'Application starting'; exit 1"

docker ps -a
docker logs d13-broken
```

### Lab 2 — Port Troubleshooting

```bash
docker run -d   --name d13-nginx   -p 8080:80   nginx:alpine

curl http://localhost:8080
docker port d13-nginx
docker inspect d13-nginx
```

### Lab 3 — Container Shell

```bash
docker exec -it d13-nginx sh
```

Inside:

```bash
ls
cat /etc/os-release
```

### Lab 4 — Resource Monitoring

```bash
docker stats
```

### Lab 5 — Network Debugging

```bash
docker network create d13-network

docker run -d   --name d13-web   --network d13-network   nginx:alpine

docker run --rm   --network d13-network   curlimages/curl   http://d13-web
```

## Cleanup

```bash
docker rm -f d13-broken d13-nginx d13-web
docker network rm d13-network
```

## Troubleshooting Cheat Sheet

| Problem | First check |
|---|---|
| Container stopped | `docker ps -a` |
| Application crash | `docker logs` |
| Restart loop | `docker logs` + `docker inspect` |
| Cannot reach application | Port mapping |
| Container-to-container failure | Network inspection |
| Missing configuration | Environment variables |
| Missing data | Volumes/mounts |
| Permission denied | User + filesystem permissions |
| High CPU | `docker stats` |
| High memory | `docker stats` |
| Disk full | `docker system df` |
| Build failure | Dockerfile + build output |
| Suspected cache issue | `docker build --no-cache` |
| Compose problem | `docker compose config` |
| Compose service failure | `docker compose logs` |

## Kubernetes Connection

Docker:

```bash
docker logs
docker inspect
docker exec
docker stats
```

Kubernetes has analogous troubleshooting commands:

```bash
kubectl logs
kubectl describe
kubectl exec
kubectl top
```

The troubleshooting philosophy remains the same:

```text
Application
   ↓
Container
   ↓
Network
   ↓
Storage
   ↓
Resources
```

Kubernetes adds additional layers such as Pods, Deployments, Services, and Nodes.
