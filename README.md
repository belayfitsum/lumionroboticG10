# LumionRobotic G10 - Autonomous Lawn Mower

A multi-disciplinary project combining Full Stack, AI/Data, Robotics, and Cybersecurity to build an autonomous lawn mower system.

## Teams

| Team | Responsibility | Owns |
|---|---|---|
| Full Stack | Mobile app (React Native) + Backend (FastAPI) | /backend, /mobile |
| AI/Data | Route planning, decision making | /ai |
| Robotics | Robot firmware and control | /robotics |
| Cybersecurity | Security pipeline, auth, infrastructure | /.github, /auth |

## System Overview

```
Mobile App (React Native + MapLibre)
    ↕ WebSocket
FastAPI Backend
    ↕
PostgreSQL Database
    ↕
Robot (subscribes to commands, publishes telemetry)
```

## Security

See [SECURITY.md](./SECURITY.md) for security policy and controls.

## Branch Protection

- Direct commits to `main` are not allowed
- All pull requests require review from the relevant CODEOWNER
- Security pipeline must pass before merge

## Getting Started

Each team works in their respective folder. See the folder-level README for setup instructions.
