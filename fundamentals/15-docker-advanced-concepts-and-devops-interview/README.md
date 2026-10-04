# D15 — Docker Advanced Concepts & DevOps Interview Preparation

## Objective

Consolidate Docker knowledge into advanced production, CI/CD, troubleshooting, and DevOps interview skills.

## Topics

- Docker architecture
- Images, containers, and layers
- Layer caching
- Multi-stage builds
- Build context and `.dockerignore`
- Networking and container DNS
- Volumes and bind mounts
- Docker Compose
- Registries and image tagging
- Security
- Secrets
- Image scanning
- Jenkins CI/CD
- Image traceability
- Troubleshooting scenarios
- Docker vs VM
- Docker vs Kubernetes
- Production checklist
- DevOps interview questions

## 1. Docker Architecture

```text
Docker CLI
    |
    v
Docker Engine
    |
    +-------------+-------------+
    |             |             |
  Images      Containers     Networks
    |             |             |
    |          Volumes         |
    +-------------+-------------+
                  |
            Container Runtime
```

## 2. Image vs Container

An image is a read-only template used to create containers.

```text
Image
  |
  +-- Container 1
  +-- Container 2
  +-- Container 3
```

A container is a running instance created from an image.

## 3. Image Layers

Docker images are built from layers.

```dockerfile
FROM ubuntu
RUN apt update
RUN apt install nginx
COPY app /app
```

Conceptually:

```text
COPY app
---------
RUN install
---------
RUN apt update
---------
Ubuntu
```

Unchanged layers can be reused.

## 4. Layer Caching

Prefer ordering Dockerfiles so rarely changing dependencies are installed before frequently changing application source.

Better example:

```dockerfile
COPY package.json .
RUN npm install
COPY . .
```

This can reuse the dependency layer when source files change but dependencies do not.

## 5. Multi-Stage Builds

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /app/target/app.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Benefits:

- Smaller final image
- Faster deployments
- Smaller attack surface
- Build tools excluded from runtime image

## 6. Build Context

When running:

```bash
docker build -t myapp .
```

`.` is the build context.

Use `.dockerignore` to keep unnecessary files out:

```text
.git
node_modules
target
*.log
.env
README.md
```

This improves build performance and reduces accidental data exposure.

## 7. Docker Networking

Typical model:

```text
Container A
     |
Docker Network
     |
Container B
```

Common network types include:

```text
bridge
host
none
overlay
```

On a user-defined network, containers can use Docker DNS and service/container names instead of hardcoded IP addresses.

Example:

```text
http://backend:8080
```

Avoid:

```text
http://172.18.0.4:8080
```

because container IP addresses can change.

## 8. Volumes

Containers are ephemeral.

Persistent data should use storage outside the container's writable layer.

```text
Container
    |
    v
Volume
    |
    v
Persistent Data
```

Commands:

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
```

## 9. Bind Mount vs Named Volume

Bind mount:

```bash
docker run \
  -v /host/path:/container/path \
  myapp
```

Named volume:

```bash
docker run \
  -v app-data:/data \
  myapp
```

Bind mounts are useful when a specific host path matters. Named volumes are managed by Docker.

## 10. Docker Compose

Compose defines multi-container applications.

Example:

```yaml
services:
  app:
    image: myapp:1.0
    ports:
      - "8080:8080"
    depends_on:
      - db

  db:
    image: postgres:16
```

Commands:

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
docker compose build
```

## 11. Docker Registry

Typical workflow:

```text
Dockerfile
   |
docker build
   |
Image
   |
docker tag
   |
Registry
   |
docker pull
   |
Deployment
```

Examples of registries:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Azure Container Registry
- Google Artifact Registry

## 12. Image Tagging

Avoid relying only on:

```text
latest
```

Prefer traceable tags:

```text
myapp:1.4.2
myapp:git-a8c91f2
myapp:build-125
```

Example:

```bash
docker tag myapp:1.0 \
  registry.example.com/myapp:1.0.0
```

## 13. Docker Security

Production checklist:

```text
Non-root user
Minimal image
No secrets in image
Read-only filesystem where possible
Drop unnecessary capabilities
Avoid unnecessary privileged mode
Image scanning
Trusted base images
```

## 14. Secrets

Never store secrets directly in:

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

Use secure mechanisms such as:

- Jenkins Credentials
- AWS Secrets Manager
- Kubernetes Secrets
- HashiCorp Vault
- Cloud IAM mechanisms

## 15. Image Scanning

A mature pipeline can use:

```text
Build
  |
Scan
  |
Test
  |
Push
  |
Deploy
```

Possible tools include:

```text
Trivy
Docker Scout
Grype
```

Look for critical and high-severity vulnerabilities, vulnerable dependencies, outdated packages, and risky base images.

## 16. Docker with Jenkins

A typical CI/CD workflow:

```text
Developer
   |
Git
   |
Jenkins
   |
Checkout
   |
Unit Tests
   |
Docker Build
   |
Image Scan
   |
Container Test
   |
Registry Push
   |
Deployment
```

Image tags should allow the deployed artifact to be traced back to source code.

Example:

```text
myapp:2.1234.a8c91f2
```

or:

```text
myapp:build-125
```

## 17. Troubleshooting Framework

When something fails:

```text
1. Is the container running?
2. Check docker ps -a
3. Check logs
4. Inspect container
5. Check environment variables
6. Check network
7. Check volumes
8. Check resources
9. Check application behavior
```

Useful commands:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
docker network inspect <network>
docker volume inspect <volume>
```

## 18. Scenario — Container Immediately Exits

Start with:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Check:

- Main process
- Command
- Entrypoint
- Environment variables
- Application errors

Important principle:

> A container normally lives as long as its main process lives.

## 19. Scenario — Application Is Not Reachable

Check:

```bash
docker port <container>
```

Verify that the application is listening on the appropriate interface, commonly `0.0.0.0` inside the container.

Then check:

- Port mapping
- Host firewall
- Network configuration
- Application status

Example:

```bash
docker run -p 8080:8080 myapp
```

## 20. Scenario — Container Cannot Reach Database

Check:

```bash
docker network ls
docker network inspect <network>
docker exec <app> env
```

Verify:

- DB hostname
- DB port
- Credentials
- Network membership
- Database readiness

Prefer the service/container name over a hardcoded IP.

## 21. Scenario — Docker Host Disk Full

Start with:

```bash
docker system df
```

Investigate:

```text
Images
Containers
Volumes
Build cache
Logs
```

Cleanup examples:

```bash
docker image prune
docker system prune
```

Use aggressive cleanup carefully on shared CI hosts.

## 22. Scenario — Image Too Large

Inspect:

```bash
docker history myapp
```

Consider:

```text
Multi-stage builds
Smaller base image
.dockerignore
Fewer packages
Remove build tools
Remove unnecessary files
```

## 23. Scenario — Build Is Slow

Check:

```text
Dockerfile ordering
Build context size
Layer caching
.dockerignore
Dependency installation
```

Bad:

```dockerfile
COPY . .
RUN npm install
```

Better:

```dockerfile
COPY package*.json .
RUN npm install
COPY . .
```

## 24. Scenario — Container Uses Too Much Memory

Start with:

```bash
docker stats
```

Then investigate:

```text
Application memory
Container limit
Traffic/load
Possible memory leak
```

Possible actions:

- Fix memory leaks
- Tune application
- Set appropriate memory limits
- Scale horizontally
- Profile the application
- Analyze workload

Do not blindly increase memory.

## 25. Docker vs Virtual Machine

VM:

```text
Hardware
   |
Hypervisor
   |
Guest OS
   |
Application
```

Docker:

```text
Hardware
   |
Host OS
   |
Docker Engine
   |
Containers
   |
Applications
```

Containers generally share the host kernel, while VMs provide a separate guest operating system.

## 26. Docker vs Kubernetes

Docker helps with:

```text
Build
Package
Run
Distribute
```

Kubernetes focuses on:

```text
Deploy
Scale
Schedule
Heal
Expose
Roll out
Roll back
```

They solve related but different problems.

## 27. Is Kubernetes a Replacement for Docker?

Kubernetes is not simply "Docker at scale."

Modern Kubernetes commonly uses container runtimes such as:

```text
containerd
CRI-O
```

Docker remains highly useful for:

- Development
- Image building
- Local workflows
- CI/CD
- Container fundamentals

## 28. Docker Compose vs Kubernetes

Compose is well suited to local and smaller multi-container workflows.

Kubernetes provides production orchestration capabilities such as:

- Scheduling
- Scaling
- Self-healing
- Service discovery
- Rolling deployments
- Rollbacks

Conceptually:

```text
Compose service
       |
       v
Kubernetes workload / Pod
```

and:

```text
Compose networking
       |
       v
Kubernetes networking / Services
```

## 29. Production Docker Checklist

```text
[ ] Minimal image
[ ] Trusted base image
[ ] Multi-stage build where appropriate
[ ] Non-root user
[ ] No secrets in image
[ ] Health check
[ ] Resource limits
[ ] Graceful shutdown
[ ] Proper logging
[ ] Log rotation
[ ] Immutable version tag
[ ] Security scan
[ ] Dependency scan
[ ] .dockerignore
[ ] Tested image
[ ] Rollback strategy
```

# Final Labs

## Lab 1 — Multi-Stage Build

Create a simple Java application and compare a single-stage image with a multi-stage production image.

Check:

```bash
docker images
docker history <image>
```

## Lab 2 — Production Container

Create a container with:

- Non-root user
- Health check
- Memory limit
- CPU limit
- Restart policy

Example:

```bash
docker run -d \
  --name d15-app \
  --restart unless-stopped \
  --memory=512m \
  --cpus=1 \
  myapp:1.0.0
```

## Lab 3 — Registry Workflow

```bash
docker build -t myapp:1.0.0 .
docker tag myapp:1.0.0 <registry>/myapp:1.0.0
docker push <registry>/myapp:1.0.0
docker pull <registry>/myapp:1.0.0
```

## Lab 4 — Troubleshooting

Intentionally break:

```text
Port
Environment variable
Database hostname
Container command
File permission
```

Then troubleshoot using:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec <container> sh
docker stats
docker network inspect <network>
```

# DevOps Interview Questions

## Fundamentals

1. What is Docker?
2. Docker vs VM?
3. Image vs container?
4. What is Docker Engine?
5. What is a Docker layer?
6. What is a Docker Registry?
7. What is Docker Hub?

## Dockerfile

8. COPY vs ADD?
9. CMD vs ENTRYPOINT?
10. RUN vs CMD?
11. ARG vs ENV?
12. What is `.dockerignore`?
13. What is a multi-stage build?
14. How do you reduce image size?

## Networking

15. What is Docker bridge networking?
16. What is host networking?
17. How do containers communicate?
18. How does Docker DNS work?
19. Why should container IPs not be hardcoded?

## Storage

20. Volume vs bind mount?
21. Why are containers ephemeral?
22. How do you persist database data?

## Security

23. Why should containers not run as root?
24. Why should secrets not be stored in Dockerfiles?
25. What is `--privileged`?
26. What are Linux capabilities?
27. How do you secure Docker images?

## CI/CD

28. How do you integrate Docker with Jenkins?
29. How do you tag images?
30. How do you implement rollback?
31. Why scan Docker images?
32. Why use immutable tags?

## Troubleshooting

33. Container exits immediately — what do you check?
34. Container is running but application is unreachable — what do you check?
35. Container cannot connect to another container — what do you check?
36. Image is huge — how do you reduce it?
37. Docker build is slow — what do you check?
38. Docker host is out of disk — how do you troubleshoot?
39. Container consumes too much memory — what do you investigate?

## Production

40. What is a health check?
41. What is graceful shutdown?
42. What is a restart policy?
43. What is rolling deployment?
44. What is Blue/Green deployment?
45. What is Canary deployment?
46. What is immutable infrastructure?

## Kubernetes

47. Why Kubernetes?
48. Docker vs Kubernetes?
49. Is Kubernetes a replacement for Docker?
50. How does Docker knowledge help with Kubernetes?

# Docker Mastery Completion

After D15:

```text
Git Mastery
     |
     v
Docker Mastery
     |
     +-- Fundamentals
     +-- Images
     +-- Containers
     +-- Networking
     +-- Volumes
     +-- Compose
     +-- Security
     +-- Registry
     +-- CI/CD
     +-- Troubleshooting
     +-- Production
     +-- Advanced Concepts
     |
     v
Kubernetes Mastery
```

D15 completes the major Docker learning track and prepares for Kubernetes.
