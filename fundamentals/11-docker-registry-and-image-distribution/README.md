# D11 — Docker Registry & Image Distribution

## Objective

Learn how Docker images are tagged, pushed to registries, pulled by other environments, and used in CI/CD.

Topics:
- Container registries
- Docker Hub
- Public vs private registries
- Image naming
- Tags
- `docker login`
- `docker push`
- `docker pull`
- `docker tag`
- Image digests
- Versioning
- Amazon ECR concepts
- CI/CD image workflow
- Immutable image references
- Registry retention

## Image Naming

General format:

```text
registry/namespace/image:tag
```

Example:

```text
docker.io/buildwithnarayan/backend:1.0.0
```

## Build an Image

```bash
docker build -t d11-demo:1.0.0 .
```

## Tag an Image

```bash
docker tag d11-demo:1.0.0 buildwithnarayan/d11-demo:1.0.0
```

## Login

```bash
docker login
```

Use an access token for Docker Hub rather than putting account passwords into automation.

## Push

```bash
docker push buildwithnarayan/d11-demo:1.0.0
```

## Pull

```bash
docker pull buildwithnarayan/d11-demo:1.0.0
```

Run it:

```bash
docker run -d   --name d11-test   -p 8080:80   buildwithnarayan/d11-demo:1.0.0
```

## Tags

Examples:

```text
backend:1.0.0
backend:1.0.1
backend:2.0.0
```

Avoid relying on mutable `latest` for controlled production deployments.

## Image Digests

A digest identifies a specific immutable image:

```text
sha256:...
```

Inspect:

```bash
docker image inspect buildwithnarayan/d11-demo:1.0.0
```

Look for `RepoDigests`.

## CI/CD Flow

```text
Git Commit
    ↓
Jenkins
    ↓
Docker Build
    ↓
Image Tag
    ↓
Security Scan
    ↓
Registry Push
    ↓
Deployment
```

A strong practice is to build once and promote the same immutable image through environments.

## Amazon ECR

Typical flow:

```text
Jenkins
   ↓
AWS authentication
   ↓
Amazon ECR
   ↓
ECS / EKS / EC2
```

Example authentication pattern:

```bash
aws ecr get-login-password --region <region>   | docker login     --username AWS     --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
```

Then:

```bash
docker push <account>.dkr.ecr.<region>.amazonaws.com/backend:1.0.0
```

Use IAM roles or secure CI/CD credentials. Never hardcode AWS credentials.

## Image Scanning

Before production:

```text
Build
 ↓
Scan
 ↓
Pass?
 ├─ No → Stop
 └─ Yes
      ↓
    Push
```

Scan for known vulnerabilities in OS packages and application dependencies.

## Registry Retention

Registries accumulate old images. Use retention policies to remove unused artifacts while retaining:

- Current production images
- Release versions
- Required rollback images
- Compliance-required artifacts

## Labs

### Lab 1 — Tagging

```bash
docker build -t d11-demo:1.0.0 .
docker tag d11-demo:1.0.0 buildwithnarayan/d11-demo:1.0.0
docker images
```

### Lab 2 — Push/Pull

```bash
docker login
docker push buildwithnarayan/d11-demo:1.0.0
docker pull buildwithnarayan/d11-demo:1.0.0
```

### Lab 3 — Multiple Tags

```bash
docker tag d11-demo:1.0.0 buildwithnarayan/d11-demo:stable
docker push buildwithnarayan/d11-demo:stable
```

### Lab 4 — Inspect

```bash
docker image inspect buildwithnarayan/d11-demo:1.0.0
```

### Lab 5 — CI/CD Simulation

```bash
VERSION=1.0.1

docker build -t buildwithnarayan/d11-demo:${VERSION} .
docker push buildwithnarayan/d11-demo:${VERSION}
```

## Kubernetes Connection

Kubernetes pulls images from registries when starting Pods:

```text
Kubernetes
    ↓
Container Registry
    ↓
Container Image
    ↓
Pod
```

Example:

```yaml
containers:
  - name: backend
    image: registry.example.com/backend:1.4.2
```

## Knowledge Check

1. What is a container registry?
2. Why do we need one?
3. What does `docker tag` do?
4. What does `docker push` do?
5. What does `docker pull` do?
6. Why can `latest` be dangerous?
7. What is an image digest?
8. Why should images be traceable to Git commits?
9. What is Amazon ECR?
10. Why use a private registry?
11. Why scan images?
12. What does build-once/promote mean?
13. Why are registry retention policies useful?
14. How does Kubernetes use a registry?
