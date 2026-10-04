# D06 — Docker Image Layers, Build Cache & Optimization

## 🎯 Objective

Understand how Docker images are constructed internally and learn how to make Docker builds faster, smaller, and more reliable.

By the end of this module, you should understand:

- Docker image layers
- Read-only image layers
- Container writable layer
- `docker history`
- Build cache
- Cache invalidation
- Dockerfile instruction ordering
- Image size
- Layer optimization
- `.dockerignore`
- Multi-stage builds
- BuildKit basics
- Practical image optimization

---

# 1. Docker Images Are Layered

A Docker image is not one large block of data.

It is composed of layers.

Conceptually:

```text
┌──────────────────────────────┐
│ Application layer            │
├──────────────────────────────┤
│ Dependencies layer           │
├──────────────────────────────┤
│ Runtime/configuration layer  │
├──────────────────────────────┤
│ Base image layers            │
└──────────────────────────────┘
```

Each Dockerfile instruction can create a new filesystem layer.

For example:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install -y curl

COPY app.sh /app/app.sh
```

Conceptually:

```text
Layer 4 → COPY app.sh
Layer 3 → install curl
Layer 2 → apt-get update
Layer 1 → ubuntu base image
```

---

# 2. Why Layers Matter

Layers provide:

- Reusability
- Build caching
- Faster image builds
- Efficient image distribution
- Better storage utilization

Suppose you build two images:

```text
Image A
  ├── Ubuntu
  ├── Java
  └── App A

Image B
  ├── Ubuntu
  ├── Java
  └── App B
```

Docker can reuse common layers:

```text
             ┌── App A
             │
Ubuntu ─ Java ┤
             │
             └── App B
```

This is one reason container registries can avoid repeatedly transferring identical layers.

---

# 3. Image Layers vs Container Writable Layer

When a container starts from an image:

```text
Docker Image
┌─────────────────────┐
│ Read-only layers    │
├─────────────────────┤
│ Read-only layers    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Container writable  │
│ layer                │
└─────────────────────┘
```

The image layers are read-only.

The running container gets a writable layer on top.

This explains an important behavior:

> Changes made inside a normal container do not automatically modify the original image.

---

# 4. Demonstrating the Writable Layer

Run:

```bash
docker run -d --name layer-test nginx:alpine
```

Enter:

```bash
docker exec -it layer-test /bin/sh
```

Inside:

```sh
echo "hello" > /tmp/example.txt
exit
```

Now:

```bash
docker diff layer-test
```

You should see the changed path.

The original image is still unchanged.

Remove the container:

```bash
docker rm -f layer-test
```

The temporary container-layer change disappears with the container.

---

# 5. `docker history`

Use:

```bash
docker history nginx:alpine
```

This helps show how the image was assembled.

For your own image:

```bash
docker history docker-mastery-web:v1
```

You may see entries corresponding to Dockerfile instructions.

---

# 6. Build Cache

Docker can reuse previously built layers.

Consider:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

If the base image has not changed and `index.html` has not changed, Docker can reuse the existing build result.

Conceptually:

```text
Build #1

FROM nginx:alpine        → BUILD
COPY index.html          → BUILD


Build #2

FROM nginx:alpine        → CACHE
COPY index.html          → CACHE
```

This can make repeated builds dramatically faster.

---

# 7. Cache Invalidation

Now modify:

```text
index.html
```

and build again:

```bash
docker build -t docker-mastery-web:v2 .
```

The `COPY` instruction changes.

Therefore the affected layer needs to be rebuilt.

A simplified model:

```text
Layer 1 → CACHE
Layer 2 → REBUILD
Layer 3 → REBUILD
Layer 4 → REBUILD
```

Once Docker reaches an instruction whose cache can no longer be reused, later dependent layers generally need to be rebuilt.

---

# 8. Dockerfile Instruction Ordering

This is extremely important for CI/CD.

Consider:

```dockerfile
FROM node:22-alpine

COPY . /app

WORKDIR /app

RUN npm install
```

If any application file changes, the `COPY . /app` layer changes.

That can invalidate the following dependency installation layer.

A better pattern is often:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .
```

Now:

```text
package.json unchanged
        ↓
npm install layer can stay cached

source code changed
        ↓
only later layers rebuild
```

This can make Jenkins builds significantly faster.

---

# 9. `.dockerignore`

The build context can contain unnecessary files.

Example:

```text
project/
├── .git/
├── node_modules/
├── logs/
├── .env
├── Dockerfile
├── package.json
└── src/
```

Create:

```text
.dockerignore
```

Example:

```text
.git
.gitignore
node_modules
logs
*.log
.env
README.md
```

Benefits:

- Smaller build context
- Faster build startup
- Less unnecessary data
- Reduced chance of accidentally including sensitive files

---

# 10. Image Size

Check image sizes:

```bash
docker images
```

Or:

```bash
docker image ls
```

Example:

```text
REPOSITORY       TAG       SIZE
my-app           v1        450MB
my-app           v2        180MB
```

Smaller images generally improve:

- Registry storage
- Push time
- Pull time
- Deployment time
- Startup workflows
- Network usage

But don't optimize size blindly.

A slightly larger image may be preferable if it provides required compatibility or operational tooling.

---

# 11. Avoid Unnecessary Files

Don't copy everything into the image:

```dockerfile
COPY . /app
```

unless you actually need everything.

Use `.dockerignore` and targeted `COPY` operations.

For example:

```dockerfile
COPY package*.json ./
COPY src ./src
```

This gives you more control.

---

# 12. Clean Package Manager Metadata

For Debian/Ubuntu-based images, this pattern is common:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

The cleanup removes package metadata that is no longer needed at runtime.

The exact cleanup strategy depends on the package manager and base image.

---

# 13. Alpine Images

You may see:

```dockerfile
FROM node:22-alpine
```

or:

```dockerfile
FROM nginx:alpine
```

Alpine images are often significantly smaller than their full distribution counterparts.

However:

> Alpine is not automatically the correct choice for every application.

Some applications and native dependencies behave differently with Alpine's libc environment.

Choose based on compatibility, security, size, and operational requirements.

---

# 14. Multi-Stage Builds

One of the most important Docker optimization techniques is the multi-stage build.

Imagine compiling a Java application.

You need:

```text
JDK
Maven
Source code
Build tools
```

to build the application.

But the running application may only need:

```text
JRE
Application JAR
```

A single-stage image could contain unnecessary build tools.

Multi-stage builds solve this.

---

# 15. Multi-Stage Example

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /build

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn package -DskipTests
```

Then:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder /build/target/app.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

The result:

```text
Builder Image
├── Maven
├── JDK
├── Source
└── Build tools
        │
        │ build
        ▼
     app.jar
        │
        ▼
Runtime Image
├── JRE
└── app.jar
```

The final image does not need Maven or the source code.

---

# 16. Why Multi-Stage Builds Matter for You

This is particularly relevant to your DevOps/Jenkins workflow.

Your current build flow can eventually become:

```text
Git
 │
 ▼
Jenkins
 │
 ▼
Maven Build
 │
 ▼
Docker Build
 │
 ▼
Runtime Image
 │
 ▼
Container Registry
 │
 ▼
Kubernetes
```

A multi-stage Dockerfile can move the build process into Docker itself:

```text
Jenkins
  │
  ▼
docker build
  │
  ├── Builder stage
  │      ├── Maven
  │      ├── JDK
  │      └── source
  │
  └── Runtime stage
         ├── JRE
         └── JAR
```

We'll use this heavily when we build Java application images.

---

# 17. BuildKit

Modern Docker uses BuildKit as its build engine.

It provides improved:

- Build performance
- Caching
- Parallelism
- Build output
- Build features

A normal command remains:

```bash
docker build -t my-app:v1 .
```

You don't need to memorize BuildKit internals right now.

The important concept is:

```text
Dockerfile
    ↓
BuildKit
    ↓
Optimized build process
```

---

# 18. Inspect Image Size and Layers

Useful commands:

```bash
docker images
```

```bash
docker history my-app:v1
```

```bash
docker image inspect my-app:v1
```

For deeper investigation, tools such as Docker Scout and image-analysis tools can be introduced later.

---

# 19. Practical Optimization Rules

Keep these rules:

### Rule 1 — Use a suitable base image

```dockerfile
FROM node:22-alpine
```

when compatible.

### Rule 2 — Use `.dockerignore`

Exclude:

```text
.git
node_modules
logs
.env
```

### Rule 3 — Order Dockerfile instructions carefully

Put stable instructions before frequently changing application code.

### Rule 4 — Separate dependency files from source code

Example:

```dockerfile
COPY package*.json ./
RUN npm install
COPY . .
```

### Rule 5 — Use multi-stage builds

Keep build dependencies out of the final image.

### Rule 6 — Remove unnecessary package metadata

Clean package caches when appropriate.

### Rule 7 — Don't copy unnecessary files

Only include what the runtime requires.

### Rule 8 — Don't optimize blindly

Correctness and maintainability matter too.

---

# 🧪 D06 Hands-on Lab 1 — Observe Layers

Create:

```text
d06-layers/
└── Dockerfile
```

Dockerfile:

```dockerfile
FROM alpine:3.22

RUN echo "Layer 1"

RUN echo "Layer 2"

RUN echo "Layer 3"

CMD ["sh"]
```

Build:

```bash
docker build -t docker-layer-demo:v1 .
```

Inspect:

```bash
docker history docker-layer-demo:v1
```

Observe the layers.

---

# 🧪 D06 Hands-on Lab 2 — Cache

Create:

```text
d06-cache/
├── Dockerfile
└── index.html
```

Dockerfile:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

Build:

```bash
docker build -t docker-cache-demo:v1 .
```

Build again:

```bash
docker build -t docker-cache-demo:v2 .
```

Observe the output.

You should see cached build steps where applicable.

Now change `index.html`.

Build again:

```bash
docker build -t docker-cache-demo:v3 .
```

Observe which step is rebuilt.

---

# 🧪 D06 Hands-on Lab 3 — Multi-Stage Java Image

Create:

```text
d06-java/
├── Dockerfile
├── pom.xml
└── src/
```

Example Dockerfile:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /build

COPY pom.xml .

RUN mvn dependency:go-offline

COPY src ./src

RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=builder /build/target/*.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

Build:

```bash
docker build -t java-app:v1 .
```

Inspect:

```bash
docker images
```

Notice that the final runtime image does not need the Maven builder environment.

---

# 🎯 D06 Mental Model

The most important concept:

```text
             Dockerfile
                  │
                  ▼
          ┌───────────────┐
          │ Build Process │
          └───────┬───────┘
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
     CACHE               REBUILD
        │                   │
        └─────────┬─────────┘
                  ▼
              Image Layers
                  │
                  ▼
             Docker Image
                  │
                  ▼
              Container
```

And for production builds:

```text
              Source Code
                   │
                   ▼
             Builder Stage
          ┌─────────────────┐
          │ JDK / Maven     │
          │ Source          │
          │ Build tools     │
          └────────┬────────┘
                   │
                   │ artifact
                   ▼
             Runtime Stage
          ┌─────────────────┐
          │ JRE             │
          │ Application JAR │
          └────────┬────────┘
                   │
                   ▼
             Small Runtime
                Image
```

---

# ❓ D06 Knowledge Check

1. What is a Docker image layer?
2. Why are Docker images layered?
3. What is the container writable layer?
4. What happens to changes made inside a container when the container is deleted?
5. What does `docker history` show?
6. What is Docker build cache?
7. What causes cache invalidation?
8. Why does Dockerfile instruction ordering matter?
9. Why is `.dockerignore` important?
10. Why are smaller images generally useful?
11. Why isn't the smallest possible image always the best choice?
12. What is a multi-stage Docker build?
13. Why would a Java application benefit from a multi-stage build?
14. What is the difference between a builder image and a runtime image?
15. How can Docker build optimization improve Jenkins pipeline performance?

---

## 🔜 Next Module

### D07 — Docker Networking

We'll move into:

```text
Container
   │
   ├── bridge network
   ├── host network
   ├── container-to-container communication
   ├── DNS
   ├── ports
   └── custom networks
```

Then we'll build a small multi-container application where:

```text
Frontend Container
       │
       ▼
Backend Container
       │
       ▼
Database Container
```

That will prepare you for the networking model you'll later see in Kubernetes.
