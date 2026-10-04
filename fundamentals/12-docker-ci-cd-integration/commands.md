# D12 — Docker CI/CD Commands

## Build

```bash
docker build -t myapp:1.0.0 .
```

## Run for Testing

```bash
docker run --rm myapp:1.0.0
```

## Run a Web Container

```bash
docker run -d   --name test-container   -p 8080:8080   myapp:1.0.0
```

## Health Test

```bash
curl --fail http://localhost:8080/health
```

## Cleanup

```bash
docker rm -f test-container
```

## Git Commit Tag

```bash
GIT_SHA=$(git rev-parse --short HEAD)

docker build   -t myapp:${GIT_SHA} .
```

## Build Number Tag

```bash
BUILD_NUMBER=101

docker build   -t myapp:build-${BUILD_NUMBER} .
```

## Multiple Tags

```bash
docker tag   myapp:${GIT_SHA}   myapp:build-${BUILD_NUMBER}
```

## Registry Login

```bash
echo "$REGISTRY_PASSWORD" | docker login   --username "$REGISTRY_USER"   --password-stdin
```

## Push

```bash
docker push myapp:${GIT_SHA}
```

## Docker Disk Usage

```bash
docker system df
```

## Cleanup

```bash
docker image prune
docker system prune
```

Use cleanup commands carefully on shared CI agents.

## AWS ECR Login

```bash
aws ecr get-login-password --region "$AWS_REGION" |
docker login   --username AWS   --password-stdin "$ECR_REGISTRY"
```

## ECR Build and Push

```bash
docker build   -t "$ECR_REGISTRY/myapp:$VERSION" .

docker push   "$ECR_REGISTRY/myapp:$VERSION"
```
