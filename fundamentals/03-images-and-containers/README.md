# D03 — Docker Images & Containers

## 🎯 Objective

Understand Docker images, image layers, containers, container lifecycle, logs, inspection, command execution, and port mapping.

By the end of this module, you should understand:

- Docker images
- Image repositories and tags
- Image IDs
- Image layers
- Pulling images
- Inspecting images
- Image history
- Creating and running containers
- Detached mode
- Container names
- Container lifecycle
- Logs
- Container inspection
- `docker exec`
- Port mapping

---

## 1. Docker Image

A Docker image is a packaged, read-only template used to create containers.

Conceptually:

```text
nginx:latest
│
├── Base filesystem
├── Nginx
├── Configuration
├── Libraries
└── Image metadata
```

When a container is created from an image, Docker adds a writable container layer.

```text
Docker Image
┌─────────────────────┐
│     Read-only       │
│       layers        │
└─────────────────────┘
          +
┌─────────────────────┐
│ Container writable  │
│       layer         │
└─────────────────────┘
          ↓
      Container
```

---

## 2. Repository and Tag

Docker image references commonly follow:

```text
repository:tag
```

Example:

```text
nginx:1.29
```

Where:

```text
Repository → nginx
Tag        → 1.29
```

If no tag is specified:

```bash
docker pull nginx
```

Docker normally uses:

```text
nginx:latest
```

> `latest` is a tag name. It does not guarantee that it is permanently the newest version.

---

## 3. Image IDs

List local images:

```bash
docker images
```

Example:

```text
REPOSITORY   TAG       IMAGE ID
nginx        latest    abc123def456
```

You can also use:

```bash
docker image ls
```

The image ID identifies the local image.

---

## 4. Pull an Image

Pull Nginx from a registry:

```bash
docker pull nginx
```

Simplified flow:

```text
Docker CLI
    ↓
Docker Daemon
    ↓
Container Registry
    ↓
Download Image
    ↓
Store Locally
```

Then verify:

```bash
docker images
```

---

## 5. Image Layers

Docker images are built from layers.

Conceptually:

```text
┌─────────────────────────┐
│ Application layer       │
├─────────────────────────┤
│ Configuration layer     │
├─────────────────────────┤
│ Dependencies layer      │
├─────────────────────────┤
│ Base filesystem         │
└─────────────────────────┘
```

Each layer represents filesystem changes.

Docker can reuse unchanged layers, which helps improve:

- Build speed
- Storage efficiency
- Image transfer efficiency
- CI/CD performance

We will explore layers more deeply when we learn Dockerfiles.

---

## 6. Inspect an Image

Use:

```bash
docker image inspect nginx
```

This displays detailed JSON metadata including information such as:

- Image ID
- Architecture
- Operating system
- Environment
- Entrypoint
- Layers
- Configuration

You do not need to memorize every field.

---

## 7. Image History

Use:

```bash
docker history nginx
```

This displays image history and layer-related information.

It becomes useful when investigating how an image was constructed and why an image may be large.

---

## 8. Create a Container

Run Nginx:

```bash
docker run nginx
```

Docker roughly performs:

```text
Check Image
     ↓
Create Container
     ↓
Start Container
     ↓
Run Nginx
```

Because Nginx runs in the foreground, the terminal can remain attached to the process.

---

## 9. Detached Mode

Use:

```bash
docker run -d nginx
```

`-d` means **detached mode**.

The container runs in the background.

Check it:

```bash
docker ps
```

Docker returns a container ID when the container starts.

---

## 10. Container Names

If no name is provided, Docker generates a random human-readable name.

You can specify your own:

```bash
docker run -d --name my-nginx nginx
```

Now the container can be referenced using:

```text
my-nginx
```

This is much easier for administration and troubleshooting.

---

## 11. Container Lifecycle

A simplified lifecycle:

```text
             docker run
                  │
                  ▼
              CREATED
                  │
                  ▼
               RUNNING
              /   │                /    │                ▼     ▼     ▼
         STOP   RESTART  KILL
            │
            ▼
          EXITED
            │
       ┌────┴─────┐
       ▼          ▼
     START        RM
       │
       ▼
    RUNNING
```

Important commands:

```bash
docker start
docker stop
docker restart
docker kill
docker rm
```

---

## 12. Stop a Container

```bash
docker stop my-nginx
```

A stopped container still exists.

Verify:

```bash
docker ps
```

It will not appear because it is not running.

But:

```bash
docker ps -a
```

will show it.

---

## 13. Start a Stopped Container

```bash
docker start my-nginx
```

Then:

```bash
docker ps
```

The container should be running again.

> Stopping a container does not delete it.

---

## 14. Restart a Container

```bash
docker restart my-nginx
```

This stops and starts the container.

---

## 15. Remove a Container

After stopping:

```bash
docker rm my-nginx
```

The container is removed.

The image remains:

```bash
docker images
```

This demonstrates an important distinction:

> Removing a container does not automatically remove its image.

---

## 16. Container Logs

Logs are essential for troubleshooting.

```bash
docker logs my-nginx
```

Follow logs continuously:

```bash
docker logs -f my-nginx
```

`-f` means follow.

As a DevOps engineer, `docker logs` is one of the commands you will use frequently.

---

## 17. Inspect a Container

Use:

```bash
docker inspect my-nginx
```

This returns detailed JSON information about the container.

It can show:

- Container ID
- Image
- State
- Network configuration
- IP address
- Mounts
- Environment variables
- Ports
- Runtime configuration

---

## 18. Execute Commands Inside a Container

Run:

```bash
docker exec -it my-nginx /bin/bash
```

If Bash is unavailable:

```bash
docker exec -it my-nginx /bin/sh
```

The options mean:

```text
-i → interactive
-t → allocate a terminal
```

Inside the container:

```bash
ls
```

```bash
whoami
```

Exit:

```bash
exit
```

---

## 19. Port Mapping

A container may run Nginx on port 80 internally.

To expose it through port 8080 on the host:

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```text
-p 8080:80
```

means:

```text
Mac:8080
    ↓
Container:80
```

Open:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

---

# 🧪 Hands-on Lab

## Step 1 — Pull Nginx

```bash
docker pull nginx
```

## Step 2 — Check the image

```bash
docker images
```

## Step 3 — Inspect the image

```bash
docker image inspect nginx
```

## Step 4 — Check image history

```bash
docker history nginx
```

## Step 5 — Run Nginx

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

## Step 6 — Check the container

```bash
docker ps
```

## Step 7 — Open the application

Open:

```text
http://localhost:8080
```

## Step 8 — Check logs

```bash
docker logs my-nginx
```

## Step 9 — Inspect the container

```bash
docker inspect my-nginx
```

## Step 10 — Enter the container

```bash
docker exec -it my-nginx /bin/bash
```

If required:

```bash
docker exec -it my-nginx /bin/sh
```

Inside the container:

```bash
ls
```

Then:

```bash
exit
```

## Step 11 — Stop it

```bash
docker stop my-nginx
```

## Step 12 — Start it again

```bash
docker start my-nginx
```

## Step 13 — Remove it

```bash
docker stop my-nginx
docker rm my-nginx
```

---

# 🎯 D03 Mental Model

```text
              Container Registry
                    │
                docker pull
                    ↓
              ┌───────────┐
              │   IMAGE   │
              │ nginx     │
              └─────┬─────┘
                    │
                docker run
                    ↓
              ┌───────────┐
              │ CONTAINER │
              │ my-nginx  │
              └─────┬─────┘
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        logs      exec      inspect
          │
          ↓
      Troubleshooting
```

---

# ❓ Knowledge Check

1. What is a Docker image?
2. What is an image layer?
3. What is an image tag?
4. What does `docker pull` do?
5. What does `docker run` do?
6. What does `-d` mean?
7. What does `--name` do?
8. What is the difference between `docker stop` and `docker rm`?
9. What is the difference between an image and a container?
10. What does `docker logs` do?
11. What does `docker inspect` do?
12. What does `docker exec` do?
13. What does `-p 8080:80` mean?
14. Why can multiple containers be created from one image?

---

## 🔜 Next Module

### D04 — Essential Docker Commands

We will build your practical Docker command toolbox:

```text
docker run
docker ps
docker stop
docker start
docker restart
docker rm
docker images
docker pull
docker rmi
docker logs
docker exec
docker inspect
docker cp
docker stats
docker system
```

We will finish D04 with a container troubleshooting exercise.
