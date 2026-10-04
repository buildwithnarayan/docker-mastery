# D07 — Docker Networking

## 🎯 Objective

Understand how Docker containers communicate with each other and with the host.

Topics:

- Docker networking
- Default bridge network
- Custom bridge networks
- Container-to-container communication
- Docker DNS
- Port publishing
- `EXPOSE` vs `-p`
- Network inspection
- Connecting/disconnecting containers
- Host networking
- `none` networking
- Basic network isolation
- Multi-container networking

---

## 1. List Docker Networks

```bash
docker network ls
```

Common built-in networks:

```text
bridge
host
none
```

---

## 2. Create a Custom Bridge Network

```bash
docker network create app-network
```

Inspect:

```bash
docker network inspect app-network
```

---

## 3. Run Containers on the Network

```bash
docker run -d \
  --name web \
  --network app-network \
  nginx:alpine
```

```bash
docker run -d \
  --name client \
  --network app-network \
  alpine:3.22 \
  sleep 3600
```

---

## 4. Container-to-Container Communication

Enter the client:

```bash
docker exec -it client /bin/sh
```

Install curl:

```sh
apk add --no-cache curl
```

Then:

```sh
curl http://web
```

On a custom Docker network, containers can normally discover each other by container name.

---

## 5. Docker DNS

Prefer:

```text
http://backend:8080
```

over:

```text
http://172.18.0.5:8080
```

Container IP addresses can change.

Container/service names provide stable discovery within the network.

---

## 6. Inspect a Network

```bash
docker network inspect app-network
```

Useful information includes:

- Network ID
- Driver
- Subnet
- Gateway
- Connected containers
- Container IP addresses

---

## 7. Connect a Running Container

```bash
docker network connect app-network test
```

A container can be connected to multiple networks.

---

## 8. Disconnect a Container

```bash
docker network disconnect app-network test
```

---

## 9. Remove a Network

```bash
docker network rm app-network
```

Containers must first be disconnected or removed.

---

## 10. Port Publishing

Example:

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  nginx
```

Meaning:

```text
Host :8080
     ↓
Container :80
```

Access:

```text
http://localhost:8080
```

---

## 11. `EXPOSE` vs `-p`

Dockerfile:

```dockerfile
EXPOSE 80
```

documents the container port.

It does not publish the port.

Publishing requires:

```bash
docker run -p 8080:80 nginx
```

---

## 12. Internal vs External Communication

External:

```text
Browser
   ↓
Host :8080
   ↓
Container :80
```

Internal:

```text
Frontend
   ↓
Docker network
   ↓
Backend
```

Container-to-container communication on the same Docker network does not require publishing the backend port to the host.

---

## 13. Host Network

```bash
docker run --network host nginx
```

Host networking gives the container more direct access to the host network namespace.

Use it only when there is a specific reason.

---

## 14. None Network

```bash
docker run --network none alpine
```

This provides strong network isolation for the container.

---

## 15. Network Drivers

Common drivers include:

```text
bridge
host
none
overlay
macvlan
ipvlan
```

For this module, focus on `bridge`, `host`, and `none`.

---

# 🧪 D07 Labs

## Lab 1 — Create a Network

```bash
docker network create docker-lab
docker network ls
docker network inspect docker-lab
```

## Lab 2 — Nginx + Client

```bash
docker run -d \
  --name web \
  --network docker-lab \
  nginx:alpine
```

```bash
docker run -d \
  --name client \
  --network docker-lab \
  alpine:3.22 \
  sleep 3600
```

Test:

```bash
docker exec -it client /bin/sh
```

Inside:

```sh
apk add --no-cache curl
curl http://web
```

## Lab 3 — Port Publishing

```bash
docker rm -f web
```

```bash
docker run -d \
  --name web \
  --network docker-lab \
  -p 8080:80 \
  nginx:alpine
```

Open:

```text
http://localhost:8080
```

## Lab 4 — Multi-Container Network

Create:

```bash
docker network create application-network
```

Backend:

```bash
docker run -d \
  --name backend \
  --network application-network \
  nginx:alpine
```

Frontend/client:

```bash
docker run -d \
  --name frontend \
  --network application-network \
  alpine:3.22 \
  sleep 3600
```

Test:

```bash
docker exec -it frontend /bin/sh
```

Inside:

```sh
apk add --no-cache curl
curl http://backend
```

## Cleanup

```bash
docker rm -f web client backend frontend
docker network rm docker-lab application-network
```

---

# 🎯 Mental Model

External:

```text
Browser
   ↓
Host Port
   ↓
Container Port
```

Internal:

```text
Frontend
   ↓
Docker Network
   ↓
Backend
   ↓
Docker Network
   ↓
Database
```

The key principle:

> Use container/service names for internal communication rather than hardcoded container IP addresses.

---

# 🔥 Kubernetes Connection

Docker:

```text
Container
   ↓
Docker Network
   ↓
Container DNS/name
```

Kubernetes:

```text
Pod
   ↓
Kubernetes Network
   ↓
Service
   ↓
DNS
   ↓
Pod
```

Understanding Docker networking makes Kubernetes networking much easier.

---

# ❓ Knowledge Check

1. What is Docker networking?
2. What is the default bridge network?
3. Why create a custom bridge network?
4. What does `docker network create` do?
5. What does `docker network inspect` show?
6. How do containers communicate on the same custom network?
7. Why avoid hardcoded container IP addresses?
8. What is Docker DNS?
9. What does `-p 8080:80` mean?
10. Does `EXPOSE 80` publish port 80?
11. Does internal container communication require `-p`?
12. What is host networking?
13. What is the `none` network?
14. Why is network isolation useful?
