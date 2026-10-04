# Security Policy

## Supported Versions

Security-related fixes are primarily maintained for the latest release.

| Version | Supported |
|---|---|
| 1.x | ✅ |
| Older versions | ❌ |

## Reporting a Security Issue

Please do not publish sensitive security information in a public issue.

For a security concern, contact the repository owner privately through GitHub.

When reporting an issue, include:

- Description of the issue
- Affected file/module
- Steps to reproduce
- Potential impact
- Suggested mitigation, if known

## Secrets

Never commit:

- Passwords
- API keys
- Access tokens
- Private certificates
- Cloud credentials
- SSH private keys
- Production configuration

If a secret is accidentally committed, rotate/revoke it immediately. Removing it from the latest commit does not guarantee that it has disappeared from Git history.
