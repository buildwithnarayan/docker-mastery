# D01 — Introduction to Docker

## 🎯 Objective

Understand why Docker exists, the problem it solves, and the fundamental concepts required before working with Docker commands.

By the end of this module, you should be able to explain:

- Why Docker was created
- The "works on my machine" problem
- Virtual machines vs containers
- What Docker is
- What a Docker image is
- What a Docker container is
- Image vs container
- What a container registry is
- What `docker run` does at a high level
- Why Docker is important for DevOps and CI/CD

---

## 1. The Problem Docker Solves

An application depends on more than its source code.

For example:

```text
Application
    ↓
Runtime
    ↓
Libraries
    ↓
System dependencies
    ↓
Configuration
    ↓
Operating System
```

An application may work correctly on a developer's machine but fail on another server because the environments are different.

This leads to the familiar:

> **"It works on my machine!"**

problem.

Docker helps package an application together with its required dependencies and provides a consistent way to run it.

---

## 2. Traditional Deployment

A traditional deployment may look like:

```text
Server
│
├── Operating System
│
├── Application A
│
├── Application B
│
└── Application C
```

Different applications may require different runtime versions or system dependencies.

For example:

```text
Application A → Java 8
Application B → Java 17
```

Managing these requirements on the same server can become difficult.

---

## 3. Virtual Machines

Virtual machines provide stronger isolation by running separate guest operating systems.

```text
Physical Server
      │
  Hypervisor
      │
 ┌────┴─────┐
 ↓          ↓
VM 1       VM 2
│          │
Guest OS   Guest OS
│          │
App A      App B
```

Each VM typically includes its own guest operating system and therefore consumes additional resources.

---

## 4. Containers

Containers use operating-system-level isolation while sharing the host kernel.

Conceptually:

```text
Host Server
    │
Docker Engine
    │
 ┌──┼──────────────┐
 ↓  ↓              ↓
C1  C2             C3
│   │              │
App A App B        App C
```

Containers package applications and their dependencies while using the host operating system's kernel.

---

## 5. VM vs Container

### Virtual Machine

```text
Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

### Container

```text
Hardware
   ↓
Host OS
   ↓
Container Engine
   ↓
Container
   ↓
Application
```

A useful mental model:

> **VMs virtualize machines. Containers isolate application processes.**

Containers can generally start faster and use fewer resources than full virtual machines because they do not require a separate guest OS for every application.

---

## 6. What Is Docker?

Docker is a platform and ecosystem for building, packaging, distributing, and running applications as containers.

A simplified lifecycle is:

```text
BUILD
  ↓
Docker Image
  ↓
DISTRIBUTE
  ↓
Container Registry
  ↓
PULL
  ↓
RUN
  ↓
Container
```

---

## 7. Docker Image

A Docker image is an immutable package/template used to create containers.

Examples include images for:

```text
Ubuntu
Nginx
Node.js
Java
MySQL
Redis
```

Think of an image as a **blueprint**.

---

## 8. Docker Container

A container is a running or stopped instance created from an image.

```text
             Docker Image
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
   Container 1         Container 2
```

One image can be used to create multiple containers.

### Remember

> **Image = blueprint/package**

> **Container = instance created from the image**

---

## 9. Container Registry

A container registry stores and distributes container images.

Example:

```text
Developer
    │
    │ docker build
    ↓
Docker Image
    │
    │ docker push
    ↓
Container Registry
    │
    │ docker pull
    ↓
Server
    │
    ↓
Container
```

Docker Hub is one popular public container registry.

---

## 10. First Docker Command

Check your Docker installation:

```bash
docker --version
```

Get Docker environment information:

```bash
docker info
```

Run the first test container:

```bash
docker run hello-world
```

List running containers:

```bash
docker ps
```

List all containers, including stopped containers:

```bash
docker ps -a
```

---

## 11. What Happens During `docker run hello-world`?

At a high level:

```text
docker run hello-world
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
      Start container
           │
           ▼
      Execute program
           │
           ▼
      Container exits
```

The `hello-world` container prints a message and exits. This is expected.

A container does not have to remain running forever.

---

## 12. Docker in DevOps

A traditional deployment pipeline might look like:

```text
Developer
   ↓
Git
   ↓
Jenkins
   ↓
Maven Build
   ↓
JAR
   ↓
Server
   ↓
Configure Runtime
   ↓
Start Application
```

With Docker:

```text
Developer
   ↓
Git
   ↓
Jenkins
   ↓
Build
   ↓
Docker Image
   ↓
Container Registry
   ↓
Deployment
   ↓
Container
```

Later, Kubernetes will orchestrate these containers:

```text
Git
 ↓
Jenkins
 ↓
Docker Image
 ↓
Registry
 ↓
Kubernetes
 ↓
Pods
 ↓
Containers
```

---

## 🧪 Hands-on Lab

Run the following commands in order:

```bash
docker --version
```

```bash
docker info
```

```bash
docker run hello-world
```

```bash
docker ps
```

```bash
docker ps -a
```

### Observation

Compare:

```bash
docker ps
```

with:

```bash
docker ps -a
```

The first shows currently running containers.

The second shows all containers, including containers that have exited.

---

## ❓ Knowledge Check

Before moving to D02, make sure you can answer:

1. Why does Docker exist?
2. What problem does Docker solve?
3. What is the difference between a VM and a container?
4. What is a Docker image?
5. What is a Docker container?
6. Can one image create multiple containers?
7. What is a container registry?
8. What does `docker run` do at a high level?
9. Why might a container exit immediately?
10. How does Docker fit into a CI/CD pipeline?

---

## 🔜 Next Module

### D02 — Docker Architecture

We will explore:

```text
Docker CLI
    ↓
Docker API
    ↓
Docker Daemon
    ↓
Container Runtime
    ↓
Images
    ↓
Containers
    ↓
Registry
```

We will also understand what actually happens inside Docker when you execute a command.
