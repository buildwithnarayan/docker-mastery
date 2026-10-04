# D07 — Docker Networking Commands

## Networks

```bash
docker network ls
docker network create app-network
docker network inspect app-network
docker network rm app-network
```

## Run on a Network

```bash
docker run -d --name web --network app-network nginx:alpine
docker run -d --name client --network app-network alpine:3.22 sleep 3600
```

## Test Container Communication

```bash
docker exec -it client /bin/sh
```

Inside:

```sh
apk add --no-cache curl
curl http://web
```

## Connect / Disconnect

```bash
docker network connect app-network test
docker network disconnect app-network test
```

## Port Publishing

```bash
docker run -d --name web -p 8080:80 nginx
```

## Host / None Networking

```bash
docker run --network host nginx
docker run --network none alpine
```

## Cleanup

```bash
docker rm -f web client
docker network rm app-network
```
