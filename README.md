# Ariba EPP

**Ariba Enterprise Endpoint Protection Platform** — centralized management plus distributed endpoint agents for Windows and Linux.

> **Status: planning and repository scaffold only.** No product code is implemented yet. The documents in `docs/` describe proposed requirements and architecture, not deployed behavior, verified detection coverage, certification, or antivirus equivalence. The first release must not be presented as a complete antivirus replacement.

## Documents

- [Product Blueprint](docs/ARIBA-EPP-BLUEPRINT.md) — vision, modules, MVP boundary, roadmap, open decisions.
- [Enterprise Architecture](docs/ARIBA-EPP-ENTERPRISE-ARCHITECTURE.md) — components, agent design, data/API model, security requirements, delivery plan.
- [Product Requirements](docs/PRODUCT-REQUIREMENTS.md) — requirement IDs, scope, workflows and testable acceptance criteria.
- [Implementation Plan](docs/IMPLEMENTATION-PLAN.md) — documentation gate, owner decisions and dependency-ordered backlog.
- [System Architecture](docs/architecture/SYSTEM-ARCHITECTURE.md)
- [API Contract](docs/api/API-CONTRACT.md)
- [Data Model](docs/database/DATA-MODEL.md)
- [Endpoint Agent Specification](docs/agent/AGENT-SPECIFICATION.md)
- [Threat Model](docs/security/THREAT-MODEL.md)
- [Test Strategy](docs/testing/TEST-STRATEGY.md)
- [Deployment Plan](docs/deployment/DEPLOYMENT-PLAN.md)
- [Product Idea (source)](docs/ARIBA-EPP-PRODUCT-IDEA.md) — original product concept supplied by the owner.
- [Project Task List](docs/PROJECT-TASKS.md) — milestone roadmap, step-by-step tasks, language plan and tracking rules.
- [Documentation index](docs/README.md).

## Repository layout

| Path | Purpose | Status |
|---|---|---|
| `apps/web-console/` | Web console (Next.js + TypeScript candidate) | Planned |
| `apps/api-server/` | Management API (FastAPI candidate, modular monolith) | Planned |
| `services/` | Domain modules; deployed together as a monolith first | Planned |
| `agent/` | Go endpoint agent for Windows and Linux | Planned |
| `detection/` | Versioned rules (Sigma/YARA), ATT&CK mapping, rule tests | Planned |
| `integrations/` | Wazuh, OpenSearch, MISP, ClamAV, Microsoft Defender adapters | Planned |
| `database/` | PostgreSQL migrations, seeds, schemas | Planned |
| `infrastructure/` | Docker Compose, Nginx, monitoring, backup | Planned |
| `tests/` | Unit, integration, security, performance, end-to-end | Planned |
| `docs/` | Product, architecture, API, operations and security documentation | Draft; owner review required |

Directories currently contain placeholders (`.gitkeep` or a README) only.

## Foundation v0.1 — first release proof milestone

**Do not begin implementation until the documentation gate in
[docs/IMPLEMENTATION-PLAN.md](docs/IMPLEMENTATION-PLAN.md) is approved by the
owner.**

Target deliverables: repository and architecture docs, login/admin console shell, organization and site management, PostgreSQL migrations, management API, development deployment, initial Windows/Linux agent prototypes, secure endpoint enrollment, and heartbeat.

**Definition of success:** one authorized Windows endpoint and one authorized Linux endpoint enroll with unique identities and their health/last-seen state is visible in the console. This proves the management/agent path only; it does not establish malware-prevention efficacy.

This release proof spans roadmap milestones `M0`–`M2`; it is not the same thing as `M2` alone. See [docs/PROJECT-TASKS.md](docs/PROJECT-TASKS.md) for the M0–M10 IDs.

## Ground rules

- Test only on owned or explicitly authorized endpoints in an isolated lab.
- Start in observe-only mode; disruptive response actions require authorization, approval, target validation, audit, timeout and rollback.
- Never disable existing antivirus protection to make this platform work.
- Do not use live malware in repository tests; use approved benign artifacts such as EICAR.
- Never commit secrets, private keys, certificates, or real endpoint/telemetry data.
- Mark planned work explicitly; do not describe proposals as implemented.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Security issues: see [SECURITY.md](SECURITY.md).

## License

Proprietary — all rights reserved. Licensing terms are not final; see [LICENSE](LICENSE).
