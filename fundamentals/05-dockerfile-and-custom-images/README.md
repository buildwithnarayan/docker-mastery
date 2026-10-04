# D05 — Dockerfile & Building Custom Images

## 🎯 Objective

Learn how to create custom Docker images using a Dockerfile.

By the end of this module, you should understand:

- Dockerfile
- `FROM`
- `RUN`
- `COPY`
- `ADD`
- `WORKDIR`
- `ENV`
- `EXPOSE`
- `CMD`
- `ENTRYPOINT`
- Docker build context
- `.dockerignore`
- Image layers
- Build cache
- Basic Dockerfile best practices
- Building and running a custom Docker image

---

## 1. What is a Dockerfile?

A Dockerfile is a text file containing instructions for building a Docker image.

Example:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build it:

```bash
docker build -t my-nginx .
```

---

## 2. Docker Build Context

The `.` in:

```bash
docker build -t my-nginx .
```

means the current directory is used as the build context.

Example:

```text
my-app/
├── Dockerfile
├── index.html
└── app/
```

The build context is supplied to the Docker builder.

---

## 3. `FROM`

Specifies the base image:

```dockerfile
FROM nginx:alpine
```

Examples:

```dockerfile
FROM ubuntu:24.04
FROM node:22-alpine
FROM eclipse-temurin:21-jre
```

Think:

```text
Base Image
     ↓
Your changes
     ↓
Your Image
```

---

## 4. `WORKDIR`

Sets the working directory:

```dockerfile
WORKDIR /app
```

Subsequent instructions operate from this directory.

---

## 5. `COPY`

Copies files from the build context into the image:

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

Or:

```dockerfile
COPY . /app
```

---

## 6. `ADD`

`ADD` can perform additional operations beyond ordinary file copying.

For straightforward copying, prefer:

```dockerfile
COPY
```

Use `ADD` only when its additional behavior is actually needed.

---

## 7. `RUN`

Executes commands while the image is being built:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

Related operations can often be combined:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

---

## 8. `ENV`

Defines environment variables:

```dockerfile
ENV APP_ENV=production
```

A runtime value can override it:

```bash
docker run -e APP_ENV=development my-app
```

Never bake passwords, API keys, or other secrets into a Dockerfile.

---

## 9. `EXPOSE`

Documents the port the application listens on:

```dockerfile
EXPOSE 80
```

Important:

> `EXPOSE` does not publish the port to the host.

Publishing requires:

```bash
docker run -p 8080:80 my-nginx
```

---

## 10. `CMD`

Defines the default command:

```dockerfile
CMD ["python", "app.py"]
```

If no overriding command is supplied when the container starts, Docker uses the `CMD`.

---

## 11. `ENTRYPOINT`

Defines the primary executable:

```dockerfile
ENTRYPOINT ["python"]
```

Combined with:

```dockerfile
CMD ["app.py"]
```

the effective command becomes:

```text
python app.py
```

---

## 12. CMD vs ENTRYPOINT

Think of:

```text
CMD
→ default command or arguments

ENTRYPOINT
→ primary executable
```

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Result:

```text
python app.py
```

---

## 13. Exec Form vs Shell Form

Shell form:

```dockerfile
CMD python app.py
```

Exec form:

```dockerfile
CMD ["python", "app.py"]
```

Exec form is generally preferred for application containers because process and signal behavior is clearer.

This becomes particularly important when Kubernetes manages container termination.

---

## 14. First Custom Image — Nginx

Create:

```text
docker-d05/
├── Dockerfile
└── index.html
```

### `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker-Mastery</title>
</head>
<body>
    <h1>Hello from Docker-Mastery!</h1>
    <p>This page is running from my custom Docker image.</p>
</body>
</html>
```

### `Dockerfile`

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

## 15. Build the Image

From the directory containing the Dockerfile:

```bash
docker build -t docker-mastery-web:v1 .
```

Verify:

```bash
docker images
```

The image should appear as:

```text
docker-mastery-web:v1
```

---

## 16. Run the Custom Image

```bash
docker run -d \
  --name docker-mastery-web \
  -p 8080:80 \
  docker-mastery-web:v1
```

Open:

```text
http://localhost:8080
```

You should see:

```text
Hello from Docker-Mastery!
```

---

## 17. Inspect the Image

```bash
docker image inspect docker-mastery-web:v1
```

View image history:

```bash
docker history docker-mastery-web:v1
```

This helps you understand the image's layers.

---

## 18. Image Tagging

Create another tag:

```bash
docker tag docker-mastery-web:v1 docker-mastery-web:latest
```

Then:

```bash
docker images
```

Different tags can point to the same image ID.

---

## 19. `.dockerignore`

A `.dockerignore` file prevents unnecessary files from being included in the build context.

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
- Faster builds
- Better security
- Cleaner builds

Never rely on `.dockerignore` alone for secret protection; sensitive files should not be present in the build context when avoidable.

---

## 20. Docker Build Cache

Docker can reuse previously built layers.

Conceptually:

```text
Previous Build
      │
      ├── Layer 1 → CACHE
      ├── Layer 2 → CACHE
      └── Layer 3 → REBUILD
```

Build cache can significantly improve CI/CD build times.

---

## 21. Dockerfile Layer Strategy

A poor pattern:

```dockerfile
COPY . /app

RUN apt-get update
RUN apt-get install -y ...
```

A better pattern for dependency-based applications is often:

```dockerfile
COPY package*.json ./
RUN npm install

COPY . .
```

This can preserve dependency-layer caching when only application source code changes.

The exact best structure depends on the application.

---

## 22. Dockerfile Best Practices

### Use an appropriate base image

```dockerfile
FROM nginx:alpine
```

when appropriate.

### Use `.dockerignore`

Avoid unnecessary build context.

### Do not bake secrets into images

Avoid:

```dockerfile
ENV DB_PASSWORD=MySecretPassword
```

### Prefer controlled image versions

Instead of:

```dockerfile
FROM node:latest
```

consider:

```dockerfile
FROM node:22-alpine
```

when that version is appropriate for your application.

### Keep images focused

A container should generally have one primary responsibility.

### Use exec-form commands

Prefer:

```dockerfile
CMD ["python", "app.py"]
```

---

# 🧪 D05 Hands-on Lab

Create:

```text
docker-d05/
├── Dockerfile
├── index.html
└── .dockerignore
```

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker-Mastery D05</title>
</head>
<body>
    <h1>Docker-Mastery D05</h1>
    <p>My first custom Docker image.</p>
</body>
</html>
```

### `.dockerignore`

```text
.git
.gitignore
*.log
.env
README.md
```

Build:

```bash
docker build -t docker-mastery-web:v1 .
```

Verify:

```bash
docker images
```

Run:

```bash
docker run -d \
  --name docker-mastery-web \
  -p 8080:80 \
  docker-mastery-web:v1
```

Test:

```text
http://localhost:8080
```

Inspect:

```bash
docker image inspect docker-mastery-web:v1
```

History:

```bash
docker history docker-mastery-web:v1
```

Check the container:

```bash
docker ps
```

Logs:

```bash
docker logs docker-mastery-web
```

Clean up:

```bash
docker stop docker-mastery-web
docker rm docker-mastery-web
```

---

# 🎯 D05 Mental Model

```text
                 Application
                     │
                     ▼
                Dockerfile
                     │
                 docker build
                     ▼
              ┌──────────────┐
              │ Docker Image  │
              │              │
              │ Base layers  │
              │ Your layers  │
              └──────┬───────┘
                     │
                 docker run
                     │
                     ▼
               ┌───────────┐
               │ Container │
               └─────┬─────┘
                     │
                     ▼
                 Application
```

DevOps flow:

```text
Git
 │
 ▼
Jenkins
 │
 ▼
Dockerfile
 │
 ▼
docker build
 │
 ▼
Docker Image
 │
 ▼
Container Registry
 │
 ▼
Kubernetes
```

This is the bridge between your Git/Jenkins knowledge and Kubernetes.

---

# ❓ D05 Knowledge Check

1. What is a Dockerfile?
2. What does `FROM` do?
3. What does `RUN` do?
4. What does `COPY` do?
5. What is the difference between `COPY` and `ADD`?
6. What does `WORKDIR` do?
7. What does `ENV` do?
8. Does `EXPOSE` publish a port?
9. What does `CMD` do?
10. What does `ENTRYPOINT` do?
11. What is the difference between `CMD` and `ENTRYPOINT`?
12. What is Docker build context?
13. Why do we need `.dockerignore`?
14. What is Docker build cache?
15. Why does Dockerfile instruction ordering matter?
16. Why should secrets not be placed inside Dockerfiles?
17. Why might you choose `node:22-alpine` instead of `node:latest`?

---

## 🔜 Next Module

### D06 — Docker Images: Layers, Cache & Optimization

We will go deeper into:

```text
Dockerfile
    ↓
Instructions
    ↓
Layers
    ↓
Build Cache
    ↓
Cache Invalidation
    ↓
Image Size
    ↓
Multi-stage Builds
```

We'll also build a real application image rather than only a static Nginx page.
