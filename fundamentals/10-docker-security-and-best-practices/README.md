# D10 — Docker Security & Best Practices

## Objective

Learn the security practices needed to build safer Docker images and run containers more securely.

Topics:
- Non-root users
- Minimal images
- Image scanning
- Secrets
- Capabilities
- Read-only filesystems
- Resource limits
- Health checks
- `.dockerignore`
- Dependency pinning
- Image provenance
- Runtime isolation
- Production checklist

## 1. Run as Non-Root

Avoid running application processes as root when possible.

Dockerfile example:

```dockerfile
FROM alpine:3.22

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser

CMD ["sh"]
```

Verify:

```bash
docker run --rm my-secure-image id
```

The application should run as the non-root user.

## 2. Use Minimal Base Images

Prefer images that contain only what the application needs.

Examples include:

```text
alpine
distroless
slim variants
```

Smaller images generally reduce attack surface and transfer size, but compatibility and debugging must still be considered.

## 3. Pin Important Versions

Avoid relying blindly on:

```dockerfile
FROM node:latest
```

Prefer an intentional version:

```dockerfile
FROM node:22-alpine
```

For highly controlled production builds, pinning image digests can provide stronger reproducibility.

## 4. Use `.dockerignore`

Example:

```text
.git
.gitignore
node_modules
target
*.log
.env
.DS_Store
```

This prevents unnecessary or sensitive files from entering the build context.

## 5. Never Put Secrets in Dockerfiles

Avoid:

```dockerfile
ENV DB_PASSWORD=my-password
```

Avoid committing real passwords, tokens, or private keys.

Use a secret-management mechanism appropriate for the environment.

For local development, environment variables or an ignored `.env` file can be useful, but they should not be treated as a complete production secret-management solution.

## 6. Read-Only Filesystem

Where possible, run containers with:

```bash
docker run --read-only nginx:alpine
```

Some applications need writable temporary directories. In that case, use a tmpfs mount:

```bash
docker run   --read-only   --tmpfs /tmp   nginx:alpine
```

## 7. Linux Capabilities

Containers do not need every Linux capability.

You can drop capabilities:

```bash
docker run   --cap-drop=ALL   nginx:alpine
```

If a specific application needs a capability, add only the required one.

Example:

```bash
--cap-add=NET_BIND_SERVICE
```

Do not add privileged access without a clear reason.

## 8. Avoid Privileged Containers

Avoid:

```bash
docker run --privileged ...
```

unless there is a specific, well-understood requirement.

Privileged containers significantly increase access to the host.

## 9. Resource Limits

Containers can consume excessive CPU or memory.

Example:

```bash
docker run   --memory=512m   --cpus=1   nginx:alpine
```

This helps protect the host from one container consuming excessive resources.

## 10. Health Checks

A health check lets Docker report whether an application is healthy.

Example:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3   CMD wget -qO- http://localhost/ || exit 1
```

Health checks are useful for operational visibility, but they do not replace application-level monitoring.

## 11. Scan Images

Use an appropriate image vulnerability scanner in CI/CD.

The goal is to identify:

- OS vulnerabilities
- Vulnerable libraries
- Outdated dependencies
- Known CVEs

Scanning should be part of the image build/release workflow rather than an afterthought.

## 12. Trusted Base Images

Use reputable, maintained base images.

Before adopting an image, consider:

- Publisher
- Maintenance activity
- Release cadence
- Vulnerability history
- Documentation
- Image provenance

## 13. Keep Containers Small and Focused

Prefer:

```text
one main application responsibility per container
```

rather than putting an entire stack into one container.

Example:

```text
Frontend container
Backend container
Database container
```

rather than:

```text
One giant container containing everything
```

## 14. Don't Store Persistent Data in the Image

Images should be immutable application artifacts.

Persistent data should use external storage such as Docker volumes or cloud storage.

## 15. Log to stdout/stderr

Containers should normally write application logs to:

```text
stdout
stderr
```

Then the container runtime or centralized logging platform can collect them.

Avoid depending on files inside the container as the only source of application logs.

## 16. Avoid Shell Secrets

Be careful with commands such as:

```bash
docker run -e PASSWORD=my-secret ...
```

Environment variables can sometimes be visible through process/runtime inspection and logs depending on the workflow.

For production, use a dedicated secret-management system.

## 17. Security Layers

Think of container security as multiple layers:

```text
Image security
      ↓
Dependency security
      ↓
Build security
      ↓
Runtime security
      ↓
Network security
      ↓
Host security
      ↓
Monitoring
```

No single Docker setting makes a container secure.

## 18. Production Checklist

Before production, review:

```text
[ ] Trusted base image
[ ] Version/digest strategy
[ ] Vulnerability scanning
[ ] Non-root user
[ ] Minimal packages
[ ] .dockerignore
[ ] No secrets in image
[ ] Read-only filesystem where practical
[ ] Drop unnecessary capabilities
[ ] Avoid privileged mode
[ ] CPU/memory limits
[ ] Health checks
[ ] Centralized logging
[ ] Persistent storage strategy
[ ] Network isolation
[ ] Image provenance/signing strategy
```

## Lab 1 — Non-Root Container

Create a Dockerfile:

```dockerfile
FROM alpine:3.22

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

USER appuser

CMD ["sh"]
```

Build:

```bash
docker build -t d10-nonroot .
```

Run:

```bash
docker run --rm d10-nonroot id
```

Verify that the process is not running as root.

## Lab 2 — Read-Only Container

```bash
docker run --rm --read-only nginx:alpine
```

Observe whether the application requires writable paths.

Then experiment with:

```bash
docker run --rm   --read-only   --tmpfs /tmp   nginx:alpine
```

## Lab 3 — Resource Limits

```bash
docker run --rm   --memory=256m   --cpus=0.5   nginx:alpine
```

Inspect:

```bash
docker stats
```

## Lab 4 — Capability Reduction

```bash
docker run --rm   --cap-drop=ALL   nginx:alpine
```

Only add capabilities when the application actually requires them.

## Lab 5 — Dockerfile Best Practices

Create an image that:

- Uses a specific base image
- Creates a non-root user
- Uses `.dockerignore`
- Does not contain credentials
- Contains only required dependencies
- Uses a health check where appropriate

## Knowledge Check

1. Why should containers avoid running as root?
2. What is the purpose of `.dockerignore`?
3. Why avoid `latest` for controlled production builds?
4. What is an image digest?
5. Why should secrets not be stored in Dockerfiles?
6. What does `--read-only` do?
7. What is tmpfs useful for?
8. What are Linux capabilities?
9. Why avoid `--privileged`?
10. Why set CPU and memory limits?
11. What is a Docker health check?
12. Why scan images?
13. Why use minimal base images?
14. Why should containers normally log to stdout/stderr?
15. Why should persistent data be external to the image?
16. What is the Docker security defense-in-depth model?
17. What should you verify before deploying an image to production?

## Docker Security → Kubernetes

Docker security concepts carry directly into Kubernetes:

```text
Docker non-root user
    ↓
Kubernetes securityContext

Docker read-only filesystem
    ↓
Kubernetes readOnlyRootFilesystem

Docker capabilities
    ↓
Kubernetes capabilities

Docker resource limits
    ↓
Kubernetes resources.requests / resources.limits

Docker volumes
    ↓
Kubernetes volumes / PVCs

Docker networks
    ↓
Kubernetes networking / NetworkPolicy
```

This is why D10 is an important bridge into Kubernetes.
