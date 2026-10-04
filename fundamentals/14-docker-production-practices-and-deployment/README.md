# D14 — Docker Production Practices & Deployment Patterns

## Objective

Learn how to design, secure, operate, and deploy Docker containers in production.

## Topics

- Production-ready containers
- Stateless vs stateful containers
- Immutable containers
- Graceful shutdown
- Health checks
- Restart policies
- CPU and memory limits
- Logging
- Configuration and secrets
- Non-root containers
- Read-only filesystems
- Linux capabilities
- Image optimization
- Rolling deployments
- Blue/Green deployments
- Canary deployments
- Rollbacks
- Production checklist

## 1. Development vs Production

A development container may be as simple as:

```bash
docker run -it ubuntu bash
```

Production requires additional concerns:

```text
Application
    ↓
Docker Image
    ↓
Security
    ↓
Health Check
    ↓
Resource Limits
    ↓
Logging
    ↓
Monitoring
    ↓
Graceful Shutdown
    ↓
Deployment
    ↓
Rollback
```

## 2. Stateless Containers

Production application containers should ideally be stateless.

```text
Container
    │
    ▼
Persistent Database / Storage
```

If a container is deleted, the application should be able to start a replacement without losing important state.

Do not store important application data only inside the writable container filesystem.

## 3. Immutable Containers

Treat production containers as immutable.

Avoid manually modifying a running production container:

```bash
docker exec myapp sh
apt update
apt install ...
```

Instead:

```text
Dockerfile
   ↓
Build
   ↓
New Image
   ↓
Deploy
```

This makes deployments reproducible.

## 4. Container Lifecycle

Conceptually:

```text
Created
   ↓
Running
   ↓
Stopped
   ↓
Removed
```

Commands:

```bash
docker create
docker start
docker stop
docker rm
```

Usually:

```bash
docker run
```

creates and starts a container.

## 5. Graceful Shutdown

When Docker stops a container:

```bash
docker stop myapp
```

Docker requests graceful termination.

A well-behaved application should:

1. Stop accepting new work.
2. Finish existing work where possible.
3. Close connections.
4. Flush important data.
5. Exit cleanly.

This becomes especially important in Kubernetes.

## 6. SIGTERM and SIGKILL

Conceptually:

```text
docker stop
     ↓
SIGTERM
     ↓
Application shuts down gracefully
     ↓
Application exits
```

If the process does not terminate within the grace period, it can eventually be forcefully terminated.

Applications should handle termination signals correctly.

## 7. PID 1

The main process inside a container has an important role in signal handling and child-process management.

Prefer exec-form commands:

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

rather than unnecessarily wrapping the application in shell commands.

## 8. Health Checks

A running container does not necessarily mean the application is healthy.

Example:

```dockerfile
HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --retries=3 \
    CMD wget --spider -q http://localhost:8080/health || exit 1
```

Health states can include:

```text
starting
healthy
unhealthy
```

Inspect:

```bash
docker inspect <container>
```

## 9. Restart Policies

Example:

```bash
docker run -d \
  --restart unless-stopped \
  nginx
```

Common policies:

```text
no
on-failure
always
unless-stopped
```

Restart policies should not be used to hide application failures.

A restart loop:

```text
Start
 ↓
Crash
 ↓
Restart
 ↓
Crash
 ↓
Restart
```

requires root-cause analysis.

## 10. Resource Limits

Limit CPU and memory:

```bash
docker run -d \
  --memory=512m \
  --cpus=1.0 \
  myapp
```

Monitor:

```bash
docker stats
```

Resource limits prevent one workload from unnecessarily consuming the entire host.

## 11. Logging

A production application should write useful logs.

A common Docker pattern is:

```text
Application
    ↓
stdout / stderr
    ↓
Docker logging driver
    ↓
Central logging platform
```

Examples of logging levels:

```text
DEBUG
INFO
WARN
ERROR
```

Avoid relying only on files inside the container for production logs.

## 12. Log Rotation

Logs can consume disk space.

Check Docker usage:

```bash
docker system df
```

Production systems should have appropriate log retention and rotation.

## 13. Configuration

Do not bake environment-specific configuration into the image.

Avoid:

```dockerfile
ENV DB_HOST=production-db
```

Prefer runtime configuration:

```bash
docker run \
  -e DB_HOST=database \
  -e DB_PORT=5432 \
  myapp
```

The same image can then be promoted through environments.

## 14. Secrets

Never put secrets in:

```text
Dockerfile
Git repository
Docker image
README
```

Bad:

```dockerfile
ENV DB_PASSWORD=SuperSecret123
```

Use secure secret-management mechanisms such as:

- Jenkins Credentials
- AWS Secrets Manager
- Kubernetes Secrets
- HashiCorp Vault
- Cloud-native identity mechanisms

## 15. Non-Root Containers

Prefer running applications as a dedicated non-root user.

Example:

```dockerfile
FROM alpine:3.20

RUN adduser -D appuser

USER appuser

CMD ["sleep", "3600"]
```

This reduces the impact of some container compromises.

## 16. Read-Only Filesystem

If the application does not need to write to its root filesystem:

```bash
docker run \
  --read-only \
  myapp
```

Temporary writable storage can be provided where required:

```bash
docker run \
  --read-only \
  --tmpfs /tmp \
  myapp
```

## 17. Linux Capabilities

Avoid unnecessary privileges.

For example:

```bash
docker run \
  --cap-drop ALL \
  myapp
```

Add only the capabilities genuinely required by the application.

Test carefully before applying restrictive settings to production workloads.

## 18. Avoid Unnecessary Privileged Containers

Avoid:

```bash
docker run --privileged myapp
```

unless there is a specific, understood requirement.

`--privileged` grants extensive privileges and should not be used as a generic fix for permission errors.

## 19. Production Dockerfile Example

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

RUN useradd --system --create-home appuser

COPY --chown=appuser:appuser app.jar /app/app.jar

USER appuser

EXPOSE 8080

HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --retries=3 \
    CMD wget --spider -q http://localhost:8080/health || exit 1

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

This demonstrates:

- Runtime-oriented image
- Dedicated user
- Correct ownership
- Health check
- Explicit port
- Exec-form entrypoint

## 20. Image Optimization

Smaller images generally provide:

- Faster pulls
- Faster deployments
- Less storage usage
- Smaller attack surface

Use:

```text
Multi-stage builds
+
Minimal runtime image
+
.dockerignore
+
Only required files
```

## 21. Rolling Deployment

A rolling deployment gradually replaces old instances.

```text
Version 1
 ├── Container 1
 ├── Container 2
 └── Container 3

        ↓

Version 2
 ├── Container 1
 ├── Container 2
 └── Container 3
```

Traffic can continue while instances are replaced.

Kubernetes Deployments provide this pattern natively.

## 22. Blue/Green Deployment

Two environments exist:

```text
BLUE  → Version 1
GREEN → Version 2
```

Initially:

```text
Users
  ↓
BLUE
```

After validation:

```text
Users
  ↓
GREEN
```

If a problem occurs:

```text
Users
  ↓
BLUE
```

Rollback can therefore be fast.

## 23. Canary Deployment

Only a small percentage of traffic initially reaches the new version.

```text
Users
 ├── 95% → Version 1
 └── 5%  → Version 2
```

Then gradually increase:

```text
5%
 ↓
10%
 ↓
25%
 ↓
50%
 ↓
100%
```

Monitor:

- Error rate
- Latency
- CPU
- Memory
- Business metrics

## 24. Rollback

Every production deployment should have a rollback strategy.

Example:

```text
v1.4.2
   ↓
Deploy v1.5.0
   ↓
Problem discovered
   ↓
Rollback to v1.4.2
```

Immutable image tags make this safer and more traceable.

Avoid depending entirely on:

```text
latest
```

for controlled production rollback.

## 25. Production Image Lifecycle

A mature workflow:

```text
Developer
   ↓
Git
   ↓
CI
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Registry
   ↓
Deploy
   ↓
Monitor
   ↓
Rollback if needed
```

## Labs

### Lab 1 — Resource Limits

```bash
docker run -d \
  --name d14-nginx \
  --memory=256m \
  --cpus=0.5 \
  nginx:alpine

docker stats d14-nginx
```

### Lab 2 — Restart Policy

```bash
docker run -d \
  --name d14-restart \
  --restart unless-stopped \
  nginx:alpine

docker inspect \
  --format '{{.HostConfig.RestartPolicy.Name}}' \
  d14-restart
```

### Lab 3 — Health Check

```bash
docker run -d \
  --name d14-health \
  --health-cmd="wget --spider -q http://localhost:80 || exit 1" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=3 \
  nginx:alpine

docker ps
docker inspect d14-health
```

### Lab 4 — Read-Only Container

```bash
docker run --rm \
  --read-only \
  nginx:alpine
```

Observe that applications may require specific writable paths.

### Lab 5 — Non-Root Container

Create:

```dockerfile
FROM alpine:3.20

RUN adduser -D appuser

USER appuser

CMD ["sleep", "3600"]
```

Build:

```bash
docker build -t d14-nonroot .
```

Run:

```bash
docker run -d --name d14-nonroot d14-nonroot
```

Verify:

```bash
docker exec d14-nonroot id
```

## Cleanup

```bash
docker rm -f \
  d14-nginx \
  d14-restart \
  d14-health \
  d14-nonroot
```

## Knowledge Check

1. What makes a Docker container production-ready?
2. What is a stateless container?
3. Why should important data not be stored only inside a container?
4. What does immutable infrastructure mean?
5. What happens when `docker stop` is executed?
6. What are SIGTERM and SIGKILL?
7. Why are health checks important?
8. What is the difference between a running container and a healthy application?
9. What are Docker restart policies?
10. Why don't restart policies solve application bugs?
11. Why should containers have CPU and memory limits?
12. Why should applications generally run as non-root?
13. Why should secrets never be baked into images?
14. What is a read-only root filesystem?
15. Why should `--privileged` be avoided unless required?
16. What is a rolling deployment?
17. What is Blue/Green deployment?
18. What is Canary deployment?
19. Why are immutable image tags important for rollback?
20. How do these Docker concepts map to Kubernetes?

## Kubernetes Connection

| Docker | Kubernetes |
|---|---|
| Health check | Liveness / Readiness probes |
| Restart policy | Pod restart behavior |
| CPU / memory limits | Resource requests and limits |
| Environment variables | ConfigMaps / Secrets |
| Volumes | PersistentVolumes |
| Container networking | Services / CNI |
| Rolling deployment | Deployment strategy |
| Container image | Pod image |
| Registry | Image source |
| Graceful shutdown | Pod termination lifecycle |
