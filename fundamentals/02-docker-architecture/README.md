# D02 — Docker Architecture

## 🎯 Objective

Understand the main components of Docker and how they work together when Docker commands are executed.

By the end of this module, you should understand:

- Docker CLI
- Docker API
- Docker Daemon
- Docker Engine
- Container runtime
- Images
- Containers
- Volumes
- Networks
- Container registries
- Docker Desktop vs Docker Engine
- What happens when `docker run` is executed

---

## 1. Docker Architecture Overview

Docker is not just the `docker` command.

```text
                    Docker
                       │
          ┌────────────┴────────────┐
          │                         │
     Docker CLI              Docker Desktop
          │
          ▼
      Docker API
          │
          ▼
     Docker Daemon
          │
     ┌────┼─────────────┐
     ▼    ▼             ▼
  Images Containers  Networks
          │
          ▼
       Volumes
          │
          ▼
    Container Runtime
```

This is a simplified mental model for learning Docker architecture.

---

## 2. Docker CLI

CLI means **Command Line Interface**.

The Docker CLI is the interface you use from a terminal:

```bash
docker ps
docker images
docker run
docker build
docker pull
docker push
```

The CLI sends requests to the Docker daemon.

> The Docker CLI is the client interface. It is not the component that performs every Docker operation itself.

---

## 3. Docker Daemon

The Docker daemon is the background service responsible for managing Docker objects and operations.

```text
Docker CLI
     │
     ▼
Docker Daemon
     │
 ┌───┼────────┬─────────┐
 ▼   ▼        ▼         ▼
Images Containers Networks Volumes
```

The daemon manages operations such as:

- Building images
- Pulling images
- Creating containers
- Starting containers
- Stopping containers
- Managing networks
- Managing volumes

---

## 4. Docker Client → Docker Daemon

When you execute:

```bash
docker ps
```

the simplified flow is:

```text
Terminal
   │
   ▼
Docker CLI
   │
   │ API request
   ▼
Docker Daemon
   │
   │ Query containers
   ▼
Docker CLI
   │
   ▼
Terminal output
```

This client/server relationship is important when troubleshooting.

For example, the CLI may be installed while the Docker daemon is unavailable.

---

## 5. Docker Engine

Docker Engine is the core technology used to build and run containers.

A simplified mental model is:

```text
Docker Engine
│
├── Docker CLI
├── Docker API
├── Docker Daemon
└── Container runtime
```

The underlying implementation contains additional components, which we will encounter later.

---

## 6. Container Runtime

The container runtime performs the low-level work required to create and run containers.

Simplified flow:

```text
Docker CLI
    ↓
Docker Daemon
    ↓
Container Runtime
    ↓
Container
```

Modern Docker environments use OCI-compatible runtime technologies.

For now, focus on the role rather than memorizing implementation details.

---

## 7. Docker Images

Images are packaged templates used to create containers.

Examples:

```text
nginx
ubuntu
node
java
mysql
redis
```

List local images:

```bash
docker images
```

Conceptually:

```text
       Docker Image
            │
      ┌─────┴─────┐
      ▼           ▼
 Container 1  Container 2
```

One image can create multiple containers.

---

## 8. Docker Containers

A container is an instance created from an image.

For example:

```bash
docker run nginx
```

Conceptually:

```text
nginx image
     │
     ▼
container created
     │
     ▼
container started
```

A container can be running, stopped, or exited.

---

## 9. Docker Volumes

Containers are commonly treated as ephemeral.

Data written only to a container's writable layer can disappear when the container is removed.

Docker volumes provide persistent storage:

```text
Container
    │
    │ mount
    ▼
Docker Volume
    │
    ▼
Persistent Data
```

Inspect volumes with:

```bash
docker volume ls
```

Volumes will be covered in detail in D08.

---

## 10. Docker Networks

Containers often need to communicate with other containers.

```text
┌─────────────┐
│ Web         │
│ Container   │
└──────┬──────┘
       │
    Network
       │
┌──────▼──────┐
│ Database    │
│ Container   │
└─────────────┘
```

List Docker networks:

```bash
docker network ls
```

Networking will be covered in detail in D07.

---

## 11. Container Registry

A container registry stores and distributes container images.

```text
Developer
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker push
    ▼
Container Registry
    │
    │ docker pull
    ▼
Server
    │
    ▼
Container
```

Docker Hub is one popular public container registry.

Registries become especially important in CI/CD.

---

## 12. Docker Desktop vs Docker Engine

On Linux, Docker can run directly using the Linux kernel.

On macOS and Windows, Docker Desktop provides the environment needed to run Linux containers.

Simplified Mac model:

```text
macOS
  │
Docker Desktop
  │
Linux environment / VM
  │
Docker Engine
  │
Containers
```

This is why Linux containers can run on macOS through Docker Desktop.

---

## 13. Docker Context

Docker contexts allow the Docker CLI to communicate with different Docker endpoints.

Inspect your contexts:

```bash
docker context ls
```

The active context is marked with `*`.

---

## 14. What Happens During `docker run`?

Suppose you execute:

```bash
docker run hello-world
```

A simplified flow is:

```text
docker run hello-world
          │
          ▼
      Docker CLI
          │
          ▼
      Docker API
          │
          ▼
    Docker Daemon
          │
          ▼
Does image exist locally?
       /            YES        NO
      │          │
      │          ▼
      │       Pull image
      │          │
      └────┬─────┘
           ▼
     Create container
           │
           ▼
     Container runtime
           │
           ▼
      Start container
           │
           ▼
      Execute program
           │
           ▼
      Container exits
```

This explains why `docker run` is more than simply "start a container."

---

## 15. DevOps Perspective

A practical CI/CD flow can look like:

```text
Developer
    ↓
Git
    ↓
Jenkins
    ↓
Build Application
    ↓
Build Docker Image
    ↓
Push Image to Registry
    ↓
Deployment System
    ↓
Container
```

Later, Kubernetes will become the orchestration layer:

```text
Git
 ↓
Jenkins
 ↓
Docker Image
 ↓
Container Registry
 ↓
Kubernetes
 ↓
Pods
 ↓
Containers
```

Understanding Docker architecture now will make Kubernetes architecture much easier later.

---

# 🧪 Hands-on Lab

Run:

```bash
docker version
docker info
docker context ls
docker images
docker ps
docker network ls
docker volume ls
```

These are inspection commands. Do not modify or remove anything during this lab.

## 🔎 Architecture Checklist

Identify:

- Which Docker context is active?
- Which Docker version are you using?
- Is the Docker daemon reachable?
- Are there any local images?
- Are any containers currently running?
- Which networks exist?
- Which volumes exist?

---

## ❓ Knowledge Check

1. What is the Docker CLI?
2. What is the Docker daemon?
3. How does the CLI communicate with the daemon?
4. What is Docker Engine?
5. What is a container runtime?
6. What is a Docker image?
7. What is a Docker container?
8. What is a Docker volume?
9. What is a Docker network?
10. What is a container registry?
11. What is Docker Context?
12. Why does Docker Desktop provide a Linux environment on macOS?
13. What happens at a high level when `docker run hello-world` executes?

---

## 🔜 Next Module

### D03 — Docker Images & Containers

We will go deeper into:

```text
Docker Image
    │
    ├── Layers
    ├── Tags
    ├── Image ID
    ├── Repository
    └── Registry
          │
          ▼
      Container
          │
          ├── Lifecycle
          ├── Start
          ├── Stop
          ├── Restart
          ├── Logs
          └── Inspect
```
