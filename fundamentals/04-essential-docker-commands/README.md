# D04 — Essential Docker Commands

## 🎯 Objective

Build a practical Docker command toolbox for development, CI/CD, and troubleshooting.

By the end of this module, you should be comfortable with:

- Running containers
- Listing containers
- Starting, stopping, restarting, and removing containers
- Managing images
- Reading logs
- Executing commands inside containers
- Inspecting containers and images
- Monitoring container resources
- Viewing processes
- Copying files
- Checking port mappings
- Checking Docker disk usage
- Cleaning unused Docker resources

---

## 1. Docker Command Categories

Think of Docker commands by purpose:

```text
Docker Commands
│
├── Images
│   ├── pull
│   ├── images
│   ├── inspect
│   ├── history
│   └── rmi
│
├── Containers
│   ├── run
│   ├── ps
│   ├── start
│   ├── stop
│   ├── restart
│   ├── kill
│   └── rm
│
├── Troubleshooting
│   ├── logs
│   ├── inspect
│   ├── exec
│   └── stats
│
├── Files
│   └── cp
│
└── Cleanup
    ├── prune
    └── system df
```

---

## 2. `docker run`

Create and start a container:

```bash
docker run nginx
```

A more realistic example:

```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  nginx
```

Breakdown:

```text
docker run
   │
   ├── -d
   │    └── detached mode
   │
   ├── --name my-nginx
   │    └── container name
   │
   ├── -p 8080:80
   │    └── port mapping
   │
   └── nginx
        └── image
```

---

## 3. `docker ps`

List running containers:

```bash
docker ps
```

List all containers, including stopped ones:

```bash
docker ps -a
```

A common troubleshooting command is:

```bash
docker ps -a
```

because an application may have already exited.

---

## 4. `docker start`

Start an existing stopped container:

```bash
docker start my-nginx
```

It starts an existing container; it does not create a new one.

---

## 5. `docker stop`

Gracefully stop a running container:

```bash
docker stop my-nginx
```

Conceptually:

```text
RUNNING
   │
   │ docker stop
   ▼
STOPPED
```

---

## 6. `docker restart`

Restart a container:

```bash
docker restart my-nginx
```

Conceptually:

```text
stop
 ↓
start
```

---

## 7. `docker kill`

Immediately terminate a container:

```bash
docker kill my-nginx
```

Difference:

```text
docker stop
    ↓
Graceful shutdown

docker kill
    ↓
Immediate termination
```

Use `kill` when immediate termination is required or the container is not responding.

---

## 8. `docker rm`

Remove a container:

```bash
docker rm my-nginx
```

A running container normally needs to be stopped first.

Force removal:

```bash
docker rm -f my-nginx
```

Use force removal carefully.

---

## 9. `docker rmi`

Remove an image:

```bash
docker rmi nginx
```

Equivalent:

```bash
docker image rm nginx
```

Remember:

```text
docker rm
    ↓
Container

docker rmi
    ↓
Image
```

---

## 10. `docker logs`

View container logs:

```bash
docker logs my-nginx
```

Follow logs:

```bash
docker logs -f my-nginx
```

Show the last 100 lines:

```bash
docker logs --tail 100 my-nginx
```

Show timestamps:

```bash
docker logs -t my-nginx
```

Combine options:

```bash
docker logs -f --tail 100 my-nginx
```

`docker logs` is one of the most important container troubleshooting commands.

---

## 11. `docker exec`

Execute a command inside a running container:

```bash
docker exec my-nginx ls
```

Open an interactive shell:

```bash
docker exec -it my-nginx /bin/bash
```

If Bash is unavailable:

```bash
docker exec -it my-nginx /bin/sh
```

Options:

```text
-i → interactive
-t → allocate a terminal
```

---

## 12. `docker inspect`

Inspect a container:

```bash
docker inspect my-nginx
```

Inspect an image:

```bash
docker image inspect nginx
```

Useful information includes:

- Container/image ID
- State
- Network configuration
- IP address
- Ports
- Mounts
- Environment variables
- Runtime configuration

---

## 13. `docker stats`

Monitor resource usage:

```bash
docker stats
```

Specific container:

```bash
docker stats my-nginx
```

Useful for identifying CPU and memory consumption.

---

## 14. `docker top`

View processes running inside a container:

```bash
docker top my-nginx
```

Useful when investigating what processes are actually running.

---

## 15. `docker cp`

Copy a file from the host into a container:

```bash
docker cp test.txt my-nginx:/tmp/test.txt
```

Copy a file from a container to the host:

```bash
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf
```

Use this primarily for troubleshooting and controlled file transfer, not as the normal application deployment mechanism.

---

## 16. `docker rename`

Rename a container:

```bash
docker rename old-name new-name
```

Example:

```bash
docker rename my-nginx nginx-web
```

---

## 17. `docker pause` and `docker unpause`

Pause processes inside a container:

```bash
docker pause my-nginx
```

Resume:

```bash
docker unpause my-nginx
```

These commands are useful to know even though they are less common in everyday Docker work.

---

## 18. `docker port`

Show port mappings:

```bash
docker port my-nginx
```

Example:

```text
80/tcp -> 0.0.0.0:8080
```

Meaning:

```text
Host 8080
    ↓
Container 80
```

---

## 19. `docker diff`

Show filesystem changes made inside a container:

```bash
docker diff my-nginx
```

This compares the container's filesystem state with the original image.

---

## 20. `docker system df`

Check Docker disk usage:

```bash
docker system df
```

It provides information about:

- Images
- Containers
- Local volumes
- Build cache

This is useful on developer machines and CI/CD build servers.

---

## 21. Docker Cleanup

Remove stopped containers:

```bash
docker container prune
```

Remove unused images:

```bash
docker image prune
```

Remove unused networks:

```bash
docker network prune
```

Remove unused build cache:

```bash
docker builder prune
```

General cleanup:

```bash
docker system prune
```

### ⚠️ Be careful

Do not blindly run:

```bash
docker system prune -a
```

on shared development or build machines.

Always understand what resources will be removed first.

---

# 🧪 Hands-on Lab

Create a controlled Docker environment.

## Step 1 — Run Nginx

```bash
docker run -d \
  --name docker-lab \
  -p 8080:80 \
  nginx
```

Check:

```bash
docker ps
```

Open:

```text
http://localhost:8080
```

---

## Step 2 — Check logs

```bash
docker logs docker-lab
```

Then:

```bash
docker logs --tail 20 docker-lab
```

---

## Step 3 — Inspect

```bash
docker inspect docker-lab
```

---

## Step 4 — Check resources

```bash
docker stats docker-lab
```

Press `CTRL + C` to leave the stats display.

---

## Step 5 — Check processes

```bash
docker top docker-lab
```

---

## Step 6 — Check ports

```bash
docker port docker-lab
```

---

## Step 7 — Enter the container

```bash
docker exec -it docker-lab /bin/bash
```

If required:

```bash
docker exec -it docker-lab /bin/sh
```

Inside the container:

```bash
hostname
```

Then:

```bash
ps
```

If `ps` is unavailable, do not worry; minimal container images may not contain every Linux utility.

Exit:

```bash
exit
```

---

## Step 8 — Stop it

```bash
docker stop docker-lab
```

Verify:

```bash
docker ps
```

Then:

```bash
docker ps -a
```

---

## Step 9 — Start it again

```bash
docker start docker-lab
```

Verify:

```bash
docker ps
```

---

## Step 10 — Restart

```bash
docker restart docker-lab
```

---

## Step 11 — Stop and remove

```bash
docker stop docker-lab
docker rm docker-lab
```

Verify:

```bash
docker ps -a
```

---

# 🧩 D04 Troubleshooting Exercise

Suppose Jenkins reports:

```text
Deployment failed
Container is not responding
```

You only know the container name:

```text
payment-service
```

A practical troubleshooting sequence is:

```bash
docker ps -a
```

Then:

```bash
docker logs payment-service
```

Then:

```bash
docker inspect payment-service
```

Then:

```bash
docker stats payment-service
```

Then:

```bash
docker top payment-service
```

If the container is running:

```bash
docker exec -it payment-service /bin/sh
```

This is the beginning of a real-world Docker troubleshooting workflow.

---

# 🎯 D04 Mental Model

Think about commands through the lifecycle rather than memorizing them randomly:

```text
                 IMAGE
                   │
              docker run
                   │
                   ▼
               CONTAINER
                   │
        ┌──────────┼───────────┐
        │          │           │
        ▼          ▼           ▼
      logs       inspect      stats
        │
        ▼
      exec
        │
        ▼
   troubleshooting
        │
        ▼
      stop
        │
        ▼
     start
        │
        ▼
    restart
        │
        ▼
       rm
```

Image management is separate:

```text
IMAGE
 │
 ├── pull
 ├── images
 ├── inspect
 ├── history
 └── rmi
```

---

# ❓ Knowledge Check

1. What is the difference between `docker stop` and `docker kill`?
2. What is the difference between `docker rm` and `docker rmi`?
3. What does `docker exec` do?
4. Why would you use `docker logs`?
5. What information can `docker inspect` provide?
6. What does `docker stats` show?
7. What does `docker top` show?
8. What does `docker cp` do?
9. What does `docker system df` show?
10. Why should `docker system prune -a` be used carefully?
11. How would you troubleshoot a container that starts and immediately exits?
12. How would you check whether a container is consuming excessive CPU or memory?

---

## 🔜 Next Module

### D05 — Dockerfile & Building Custom Images

We will move from:

```text
docker pull nginx
```

to:

```text
Dockerfile
     ↓
docker build
     ↓
Custom Docker Image
     ↓
docker run
     ↓
Container
```

Topics include:

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
- Build context
- Image layers
- Build cache
- `.dockerignore`
- Building a custom application image
