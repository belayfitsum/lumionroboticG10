# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| main branch | ✅ |
| feature branches | ❌ |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please do **not** open a public GitHub issue.

Instead, report it privately by emailing the cybersecurity team. We will acknowledge receipt within 48 hours and aim to resolve critical issues within 7 days.

## Security Controls in Place

- **Secret Scanning**: Gitleaks runs on every pull request. Merges are blocked if secrets are detected.
- **SCA**: GitHub Dependabot and Safety check all dependencies for known CVEs.
- **Static Analysis**: Bandit scans Python backend code for security anti-patterns.
- **Branch Protection**: Direct commits to main are not allowed. All changes require a pull request and CODEOWNERS review.
- **Environment Separation**: Development, staging, and production environments are isolated with separate credentials.
- **Least Privilege**: Each component has access only to what it needs.
