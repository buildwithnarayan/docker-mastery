# D12 — Docker CI/CD Integration

## Objective

Learn how Docker fits into a Jenkins/CI/CD pipeline.

## CI/CD Flow

```text
Git
 ↓
Jenkins
 ↓
Checkout
 ↓
Test
 ↓
Docker Build
 ↓
Container Test
 ↓
Security Scan
 ↓
Registry Push
 ↓
Deployment
```

## Basic Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
    }
}
```

## Git Commit as Image Tag

```bash
GIT_SHA=$(git rev-parse --short HEAD)

docker build   -t myapp:${GIT_SHA} .
```

Example:

```text
Git commit: a8c91f2
Docker image: myapp:a8c91f2
```

This makes the image traceable to source code.

## Multiple Tags

The same image can have multiple references:

```bash
docker build -t myapp:${GIT_SHA} .

docker tag   myapp:${GIT_SHA}   myapp:build-${BUILD_NUMBER}
```

Useful tags include:

```text
1.4.2
git-a8c91f2
build-125
```

Avoid relying on mutable `latest` for controlled production deployments.

## Registry Authentication

Never hardcode passwords or tokens in Jenkinsfiles.

Use Jenkins Credentials, cloud IAM roles, OIDC/workload identity, or the registry's secure authentication mechanism.

Example pattern:

```bash
echo "$REGISTRY_PASSWORD" | docker login   --username "$REGISTRY_USER"   --password-stdin
```

## Build Once, Promote Many

A strong CI/CD model is:

```text
Build image
   ↓
Scan
   ↓
Push immutable image
   ↓
Development
   ↓
Testing
   ↓
Staging
   ↓
Production
```

The same image should be promoted instead of rebuilding a different artifact for each environment.

Environment-specific configuration should normally be injected at deployment time.

## Docker Layer Caching

Place stable dependency instructions before frequently changing application files.

Example:

```dockerfile
COPY package*.json .
RUN npm install

COPY . .
```

This allows Docker to reuse dependency layers when only application source changes.

## Health Testing

For a web application:

```bash
docker run -d   --name test-container   -p 8080:8080   myapp:${VERSION}

curl --fail http://localhost:8080/health
```

Clean up:

```bash
docker rm -f test-container
```

## AWS ECR Flow

```text
Git
 ↓
Jenkins
 ↓
Docker Build
 ↓
Docker Test
 ↓
AWS ECR
 ↓
EKS / ECS / EC2
```

Typical authentication pattern:

```bash
aws ecr get-login-password --region "$AWS_REGION" |
docker login   --username AWS   --password-stdin "$ECR_REGISTRY"
```

Then:

```bash
docker build -t "$ECR_REGISTRY/myapp:$VERSION" .
docker push "$ECR_REGISTRY/myapp:$VERSION"
```

Use secure AWS identity/credentials. Never hardcode AWS access keys.

## Failure Handling

```text
Docker build fails
    ↓
Pipeline stops
    ↓
No push

Tests fail
    ↓
Pipeline stops
    ↓
No push

Security scan fails
    ↓
Pipeline stops

Registry push fails
    ↓
Pipeline fails
```

## Jenkins Cleanup

Inspect Docker disk usage:

```bash
docker system df
```

Cleanup carefully:

```bash
docker image prune
```

More aggressive cleanup:

```bash
docker system prune
```

Be careful on shared Jenkins agents.

## Labs

### Lab 1 — Build and Test

```bash
docker build -t d12-demo:1.0.0 .
docker run -d --name d12-test -p 8080:80 d12-demo:1.0.0
curl --fail http://localhost:8080
docker rm -f d12-test
```

### Lab 2 — Git Commit Tag

```bash
GIT_SHA=$(git rev-parse --short HEAD)
docker build -t d12-demo:${GIT_SHA} .
```

### Lab 3 — Simulate Jenkins Build Number

```bash
BUILD_NUMBER=101
docker build -t d12-demo:build-${BUILD_NUMBER} .
```

### Lab 4 — Multiple Tags

```bash
GIT_SHA=$(git rev-parse --short HEAD)
BUILD_NUMBER=101

docker build -t d12-demo:${GIT_SHA} .
docker tag d12-demo:${GIT_SHA} d12-demo:build-${BUILD_NUMBER}
```

## Kubernetes Connection

The eventual pipeline becomes:

```text
Git
 ↓
Jenkins
 ↓
Docker Build
 ↓
Registry
 ↓
Kubernetes
 ↓
Deployment
 ↓
Pods
```

Kubernetes pulls the image from the registry when creating Pods.

## Knowledge Check

1. Why is Docker useful in CI/CD?
2. Why test an image before pushing it?
3. Why tag images with Git commit IDs?
4. What is BUILD_NUMBER useful for?
5. Why shouldn't registry credentials be hardcoded?
6. What does build once, promote many mean?
7. Why separate configuration from the image?
8. What is Docker layer caching?
9. Why put dependency installation before frequently changing source code?
10. Why scan images?
11. What happens when the Docker build fails?
12. What happens when registry push fails?
13. Why do Jenkins agents need Docker cleanup?
14. How does Jenkins connect Docker to Kubernetes?
