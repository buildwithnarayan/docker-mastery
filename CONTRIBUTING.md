# Contributing to Docker Mastery

Thank you for your interest in contributing to Docker Mastery!

The goal of this repository is to provide clear, practical, DevOps-focused Docker learning material.

## How to Contribute

1. Fork the repository.
2. Create a feature branch.
3. Make your changes.
4. Test commands and examples where possible.
5. Keep documentation clear and beginner-friendly.
6. Commit your changes using a meaningful commit message.
7. Push your branch.
8. Open a Pull Request.

Example:

```bash
git checkout -b feature/improve-docker-networking
```

## Documentation Guidelines

When adding or modifying learning content:

- Explain the concept before the command.
- Prefer practical examples.
- Include expected behavior where useful.
- Avoid unnecessary complexity.
- Keep examples safe to run.
- Do not include real credentials, secrets, tokens, or private infrastructure details.

## Dockerfile Guidelines

Prefer:

- Small, appropriate base images.
- Multi-stage builds where useful.
- Non-root users for production examples.
- Explicit image versions where reproducibility matters.
- `.dockerignore`.
- Secure configuration practices.

Avoid:

- Hardcoded passwords.
- API keys.
- Private credentials.
- Unnecessary `--privileged` usage.
- Unsafe commands without explanation.

## Commit Convention

Use clear commit messages.

Examples:

```text
docs: improve docker networking explanation
docs: add volume lab
fix: correct compose example
feat: add advanced troubleshooting lab
```

## Pull Requests

A Pull Request should explain:

- What changed?
- Why was it changed?
- Which module is affected?
- How was it tested?

Keep Pull Requests focused.

## Code of Conduct

Please follow the project's Code of Conduct and maintain a respectful learning environment.
