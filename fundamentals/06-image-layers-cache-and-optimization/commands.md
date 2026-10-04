# D06 — Image Layers, Cache & Optimization Commands

## Inspect Image Layers

```bash
docker history nginx:alpine
docker history my-app:v1
```

## Inspect Image Metadata

```bash
docker image inspect my-app:v1
```

## Check Image Sizes

```bash
docker images
docker image ls
```

## Build an Image

```bash
docker build -t my-app:v1 .
```

## Rebuild to Observe Cache

```bash
docker build -t my-app:v2 .
```

## Build Without Cache

Use only when you intentionally want a clean rebuild:

```bash
docker build --no-cache -t my-app:v1 .
```

## Demonstrate Container Writable Layer

```bash
docker run -d --name layer-test nginx:alpine
docker exec -it layer-test /bin/sh
```

Inside:

```sh
echo "hello" > /tmp/example.txt
exit
```

Then:

```bash
docker diff layer-test
```

Clean up:

```bash
docker rm -f layer-test
```

## Multi-Stage Build

Example:

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

## Useful Investigation Commands

```bash
docker system df
docker history my-app:v1
docker image inspect my-app:v1
docker images
```

## Cleanup

```bash
docker image prune
docker builder prune
```

Use cleanup commands carefully on shared CI/CD machines.
