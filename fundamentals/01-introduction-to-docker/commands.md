# D01 — Docker Commands

This file contains the commands introduced in Module D01.

## Check Docker Version

```bash
docker --version
```

Shows the installed Docker CLI version.

---

## Display Docker Information

```bash
docker info
```

Displays information about the Docker installation, daemon, containers, images, storage, and other configuration details.

---

## Run the Hello World Container

```bash
docker run hello-world
```

Downloads the `hello-world` image if it is not available locally, creates a container, starts it, and runs the image's default program.

The container normally exits after printing its message.

---

## List Running Containers

```bash
docker ps
```

Shows currently running containers.

---

## List All Containers

```bash
docker ps -a
```

Shows running and stopped/exited containers.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `docker --version` | Show Docker version |
| `docker info` | Show Docker environment information |
| `docker run hello-world` | Run a test container |
| `docker ps` | Show running containers |
| `docker ps -a` | Show all containers |

> More commands will be added as they are introduced in later modules.
