# Cybersecurity Section: Autonomous Lawn Mower System
## LumionRobotic G10

---

## Chapter 1: Purpose and Scope

Lumion Robotic lawn mower system poses a unique cybersecurity challenge. It combines a mobile application, a cloud backend, database and a physical robot operating in an outdoor environment. A vulnerability at any of these mentioned layers can compromise the whole mower system - such as unauthorized mowing commands to physical safety risks.

This section outlines the secure development practices aimed to ensure the system is built, tested and deployed with security embedded in each phase. The purpose of this section is to define the security goals of the system and the measures taken to achieve them. Even though we do not claim ISO/IEC 27001 certification, we thrive to implement our own ISMS as per the standards. The primary security goals are:

- **Confidentiality**: User data, mowing zones, and schedules are accessible only to authorized users.
- **Integrity**: Commands sent to the robot have not been tampered with in transit.
- **Availability**: The system remains operational and resilient against denial of service attempts.
- **Safety**: A security failure must not result in physical harm - an unauthorized or malformed command must never cause the robot to operate dangerously.

The scope covers the mobile application, the FastAPI backend, the PostgreSQL database, the CI/CD pipeline, and the robot's communication interface.

---

## Chapter 2: Governance and Compliance

### Information Security Management

The project adopts principles from ISO/IEC 27001, the international standard for information security management. While full certification is outside the scope of a student project, the standard provides a practical framework for identifying risks and implementing controls proportionate to those risks.

### Risk Table

| Risk | Likelihood | Impact | Control |
|---|---|---|---|
| Hardcoded API key committed to git | High | High | Gitleaks secret scanning in CI pipeline |
| Unauthorized mowing command issued | Medium | Critical | Authentication + RBAC on all API endpoints |
| Robot firmware tampered via OTA | Low | Critical | Signed firmware packages + version verification |
| GPS spoofing redirects robot outside zone | Medium | High | Boundary validation on backend, not just robot |
| Dependency vulnerability in backend | High | Medium | Dependabot + Safety SCA checks |
| Stolen FCM token used for fake alerts | Low | Medium | Token rotation + server-side validation |
| Database credentials exposed | Low | Critical | Environment separation + secrets management |

### Security Policy Statement

All team members are responsible for the security of the components they develop. No credentials, API keys, or tokens are to be committed to version control under any circumstances. All changes to security-sensitive code require review and approval from the cybersecurity team before merging. Security controls are not optional and cannot be bypassed without documented justification.

---

## Chapter 3: Secure Development Processes

### 3.1 Secure Software Development Lifecycle (SSDLC)

Security is considered in each phase of the software development process.

During the **design phase**, the team identified the system attack surfaces: the WebSocket connection between the app and backend, the command interface between the backend and robot, the OTA firmware update channel, and the database storing user and telemetry data. Threat modelling at the beginning of this process shaped the security posture and requirements even before a single line of code is written.

In the **development phase**, secure coding practices are implemented through peer review and static analysis. The FastAPI backend validates all incoming data including mowing zone coordinates, schedule parameters and other commands before executing them.

During the **testing phase**, automated security checks run on every pull request. No code is allowed to merge to main without passing secret scanning, dependency checks, and static analysis. Any failure in these checks blocks the merge automatically, ensuring security issues are caught before they reach the codebase.

During the **deployment phase**, a strict environment separation i.e staging/production ensures that any development process never affects production data or the live robot.

### 3.2 DevSecOps

DevSecOps principles shifts our security strongly by integrating into the development workflow rather than a separate audit at the end. We strive to implement the following controls in the CI/CD pipelines.

**Branch Protection and CODEOWNERS**: Direct commits to the main branch are not permitted. All changes must go through a pull request. A CODEOWNERS file assigns ownership of specific files for specific teams. For example security related code changes must get approval from cybersecurity team members as API related changes go to the developers. This enforces the four eyes principle whereby no single developer or team member can introduce a change unilaterally before all team members review the code and give their approvals.

**Secret Scanning**: Gitleaks scans every pull request to detect any accidental secret committed. If a Foreca API, database password, API key or any sensitive information is detected in the diff, the pipeline fails and merge is blocked automatically.

**Software Composition Analysis (SCA)**: The Python backend depends on third-party libraries. GitHub Dependabot and the Safety tool scan these dependencies against known CVE databases. A vulnerable dependency is flagged before it reaches production.

**Static Analysis**: Bandit analyses the Python backend for security anti-patterns such as SQL injection risks, use of weak cryptographic functions, and improper exception handling.

**Least Privilege and Role Based Access (RBAC)**: To make sure access is granted to users for their need only, roles are defined with permissions scoped to the needs. Any user, admin or robot can only access data based on the role and group they are assigned to.

### 3.3 Zero Trust

The system adopts a zero trust posture - no component is trusted by default, regardless of whether it sits inside or outside the network. Every request between the mobile app, backend and robot must be authenticated independently. The robot authenticates with the backend using a device-specific credential tied to its record in the database. The backend also verifies the robot's firmware version on every connection and rejects outdated devices. An unauthenticated WebSocket connection from the mobile app is rejected immediately.

### 3.4 OTA Update Security

Over-the-air firmware updates are one of the highest risk operations in any IoT system. A successful attack on the OTA channel gives an attacker direct control over the physical robot. To mitigate this, firmware packages are cryptographically signed before distribution. The robot verifies the signature before applying any update and rejects packages that fail verification.

A rollback mechanism ensures the robot can revert to the previous verified version if an update causes unexpected behaviour. The firmware_version field in the Mowers table allows the backend to track each robot's version and flag devices that have not received critical updates.

---

## Chapter 4: Tools for Secure Development

| Tool | Category | Type | Purpose |
|---|---|---|---|
| Gitleaks | Secret Scanning | Open Source | Scans git history and pull requests for hardcoded secrets and API keys |
| GitHub Dependabot | SCA | Free/GitHub Native | Monitors Python and JavaScript dependencies for known CVEs and raises automated fix PRs |
| Bandit | Static Analysis | Open Source | Analyzes Python source code for common security vulnerabilities and anti-patterns |
| Safety | SCA | Open Source | Checks Python dependencies against a database of known vulnerabilities at CI time |
| GitHub Branch Protection + CODEOWNERS | Access Control | GitHub Native | Enforces pull request reviews, blocks direct commits, and requires cybersecurity approval on sensitive files |
| HTTPS / TLS | Transport Security | Protocol | Encrypts all communication between the mobile app, backend, and external APIs |
| Environment Secrets Management | Secrets Management | GitHub Native | Stores API keys and credentials as encrypted GitHub Actions secrets, never in source code |

---

## Chapter 5: Minimum Security Controls

The following controls are considered the minimum acceptable security baseline for this system. Removing any one of them would leave a critical gap in the security posture.

- **Secret scanning must run on every pull request.** The system integrates with multiple external APIs and services. A single accidentally committed credential could expose the entire system. Gitleaks enforces this automatically and blocks merges when secrets are detected.

- **All API endpoints must authenticate and authorize every request.** Given that the backend issues commands to a physical robot, an unauthenticated endpoint is not just a data risk but a physical safety risk. No endpoint is exempt from this requirement.

- **Firmware updates must be cryptographically signed and verified.** An unsigned OTA update channel is an open door for an attacker to take full control of the robot. Signature verification on the device is non-negotiable.

- **Environment separation must be maintained between development and production.** Developers must never have direct access to the production database or the live robot. All production deployments go through the CI/CD pipeline with appropriate approval gates.

- **Dependencies must be scanned for known vulnerabilities before every deployment.** Third-party libraries are a common attack vector. Automated SCA checks ensure the team is not unknowingly shipping software with publicly disclosed exploits.
