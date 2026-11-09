# Ariba EPP — Project Task List, Milestones and Language Plan

**Status:** Planning checklist. Nothing below is implemented yet. All technology entries are **Proposed** until owner approval/ADR.
**Purpose:** Full-project build order in small, trackable tasks.
**Tracking model:** GitHub Milestone = `M0`…`M10`; GitHub Issue = one task ID (e.g. `M2-T04`); GitHub Project board = Backlog → Ready → In Progress → Review → Done.

> **Gate:** this roadmap does not authorize coding. Follow [`IMPLEMENTATION-PLAN.md`](IMPLEMENTATION-PLAN.md) for the documentation gate and source-of-truth order until the product owner approves a milestone. If this file and a normative document disagree, the normative document wins.
>
> Estimates are illustrative for a focused team, not commitments. Do not mark a task done without implementation evidence and a test or demo note.

---

## 1. Language and technology plan

### 1.1 Why multiple languages

One language for everything sounds simple, but each layer has a different constraint:

- **Backend/control plane** needs fast iteration, strong validation and a big security ecosystem → **Python**.
- **Web console** needs typed UI, component reuse and fast rendering → **TypeScript**.
- **Endpoint agent** needs a small, fast, cross-compilable single binary with low memory/CPU → **Go**.
- **Data, policy and rules** are configuration and query languages → **SQL, YAML, JSON**.
- **Packaging/ops** are shell-native → **PowerShell, Bash**.

Using the right language per layer is normal for security products. The rule is: **one language per deployable, stable contracts between them.**

### 1.2 Proposed stack (candidate until ADR approval)

| Layer | Language / Tool | Version target | Why this is a good fit | Lives in |
|---|---|---|---|---|
| Management API | **Python + FastAPI** | Python 3.12 | Fast delivery, Pydantic validation, OpenAPI for free, mature security libs | `apps/api-server/`, `services/` |
| Background workers | **Python** | 3.12 | Same codebase as API; detection and ingestion logic reusable | `services/detection-engine/`, `services/telemetry-ingestion/` |
| Database access | **SQL + SQLAlchemy + Alembic** | PostgreSQL 16 | Typed models, versioned migrations, transaction safety | `database/`, API |
| Web console | **TypeScript + Next.js + React** | Node 20 LTS | Typed UI, server components, good dashboard ergonomics | `apps/web-console/` |
| Endpoint agent | **Go** | Go 1.22+ | Single static binary, cross-compile Windows/Linux, low footprint, easy concurrency | `agent/` |
| Agent IPC/service | **Go + native OS APIs** | — | ETW/Event Log on Windows, auditd/osquery on Linux | `agent/windows/`, `agent/linux/` |
| Detection rules | **YAML (Ariba) + Sigma + YARA** | — | Human-reviewable, versionable, portable rule sources | `detection/` |
| Policy / config | **YAML** | — | Readable, diff-friendly, easy signing | `detection/`, policies |
| Tests — backend | **pytest** | — | Fixtures, parametrization, async support | `tests/` |
| Tests — frontend | **Vitest + Playwright** | — | Unit + real browser E2E | `tests/e2e/` |
| Tests — agent | **Go `testing`** | — | Built-in, no extra dependency | `agent/` |
| Windows packaging | **PowerShell + WiX/MSI** | — | Native service install and MSI signing | `agent/packaging/` |
| Linux packaging | **Bash + fpm/nfpm** | — | DEB/RPM build scripts | `agent/packaging/` |
| Dev deployment | **Docker Compose** | — | One-command local stack | `infrastructure/docker/` |
| Reverse proxy | **Nginx** | — | TLS termination and routing | `infrastructure/nginx/` |
| Event search | **OpenSearch** | — | High-volume telemetry search | `integrations/opensearch/` |
| Message broker | **RabbitMQ** | — | Reliable async ingest with retries | `services/telemetry-ingestion/` |
| Cache/coordination | **Redis** | — | Short-lived state only, not durable evidence | backend |
| CI/CD | **GitHub Actions** | — | Lint, test, scan, build, sign | `.github/workflows/` |

### 1.3 Languages to avoid for now

| Technology | Decision | Reason |
|---|---|---|
| Rust in the agent | **Later, not v1** | Only if Go profiling shows a real CPU/memory bottleneck. Rewriting early adds risk without evidence. |
| Kubernetes | **Later** | Docker Compose is enough until measured availability/scale needs it. |
| Kafka | **Later** | RabbitMQ covers MVP throughput; Kafka adds operational weight. |
| C/C++ kernel driver | **Out of scope** | Kernel-level prevention is long-term research, not MVP. Do not claim it. |
| Microservices per folder | **No** | Build a modular monolith; extract only when measured. |

### 1.4 Repository ownership map

| Directory | Primary language | Owner team |
|---|---|---|
| `apps/api-server/` | Python | Backend |
| `apps/web-console/` | TypeScript | Frontend |
| `services/` | Python | Backend + Detection |
| `agent/` | Go (+ PowerShell/Bash packaging) | Endpoint |
| `detection/` | YAML / Sigma / YARA | Detection |
| `integrations/` | Python (+ YAML contracts) | Backend |
| `database/` | SQL | Backend |
| `infrastructure/` | YAML / HCL (later) / shell | Platform |
| `tests/` | Python / TypeScript / Go | All |

---

## 2. Milestone overview

| ID | Milestone | Goal | Exit evidence | Window |
|---|---|---|---|---|
| M0 | Foundation & decisions | Repo, ADRs, threat model, dev environment, quality gates | Approved ADRs, `docker compose up` works, CI green | Weeks 1–2 |
| M1 | Identity, RBAC & tenancy | Users, roles, org/site scope, audit | Role-matrix and tenant-isolation tests pass | Weeks 2–4 |
| M2 | Endpoint identity & enrollment | Agent enrolls, heartbeats, appears in console | 1 Windows + 1 Linux endpoint online in console | Weeks 3–5 |
| M3 | Telemetry pipeline | Events flow from agent to searchable store | Synthetic event is queryable end-to-end | Weeks 6–9 |
| M4 | Detection & alerts | Versioned rules produce explainable alerts | Safe test event → traceable alert with ATT&CK context | Weeks 10–13 |
| M5 | Policy engine | Signed, versioned policies with acknowledgement | Agent applies only signed policy; rollback tested | Weeks 14–16 |
| M6 | Guarded response | Approved, audited, expiring endpoint actions | Lab kill/quarantine/isolate with full audit + rollback | Weeks 15–17 |
| M7 | Integrations | Wazuh, MISP, OpenSearch, notifications | Versioned contract tests and failure handling | Weeks 18–21 |
| M8 | Inventory & vulnerability | Software inventory + CVE context | Inventory and findings visible per endpoint | Weeks 18–21 |
| M9 | Hardening & security validation | Threat-model-driven tests and independent review | Security checklist + load/backup/migration evidence | Weeks 22–24 |
| M10 | Packaging & pilot | Signed installers, pilot deployment, runbooks | Pilot rollback plan + known limitations approved | Weeks 22–26 |

**P0 (must for MVP):** M0–M6 core, M9. **P1:** M7–M8, M10. **P2 (later):** application control, firewall, AI assistant, macOS/mobile.

---

## 3. Detailed task breakdown

Each task is sized for roughly 0.5–2 days. Every milestone ends with a review gate.

### M0 — Foundation & decisions (Weeks 1–2)

**Definition of Done:** repo usable, decisions recorded, local stack boots, CI enforces lint + tests.

- [ ] `M0-T01` Confirm product name, audience (internal SOC vs external customers) and deployment model.
- [ ] `M0-T02` Confirm supported Windows builds and Linux distros/kernels for the pilot.
- [ ] `M0-T03` Decide console strategy: separate Next.js app vs reuse Ariba SOC frontend.
- [ ] `M0-T04` Confirm whether FastAPI/Go/PostgreSQL/OpenSearch/RabbitMQ match team skills and constraints.
- [ ] `M0-T05` Write ADR-001: modular monolith + independently scalable workers.
- [ ] `M0-T06` Write ADR-002: agent language = Go, and the telemetry schema strategy.
- [ ] `M0-T07` Write ADR-003: tenant-isolation model (service-layer filtering vs PostgreSQL RLS).
- [ ] `M0-T08` Write ADR-004: agent identity = certificate/mTLS + one-time enrollment token.
- [ ] `M0-T09` Write ADR-005: event schema versioning and retention.
- [ ] `M0-T10` Produce initial threat model (trust boundaries, assets, attacker goals).
- [ ] `M0-T11` Create `.env.example` values and secret-handling rules (no real secrets).
- [ ] `M0-T12` Build `docker-compose.dev.yml`: PostgreSQL, Redis, RabbitMQ, OpenSearch, API, console.
- [ ] `M0-T13` Add backend tooling: Ruff, mypy, pytest config, pre-commit hooks.
- [ ] `M0-T14` Add frontend tooling: ESLint, Prettier, TypeScript strict, Vitest.
- [ ] `M0-T15` Add agent tooling: `golangci-lint`, `go vet`, `go test` targets.
- [ ] `M0-T16` Create CI workflow: lint + unit tests + secret scan + dependency scan.
- [ ] `M0-T17` Define task tracking: milestones, IDs, labels, PR checklist.
- [ ] `M0-T18` Record open decisions in `docs/architecture/open-decisions.md`.
- [ ] `M0-T19` Milestone review: sign off M0 evidence.

### M1 — Identity, RBAC & tenancy (Weeks 2–4)

**Definition of Done:** login works, roles enforced server-side, tenant data isolated, audit written.

**Backend (Python)**
- [ ] `M1-T01` Scaffold `apps/api-server` as modular monolith (routers per domain).
- [ ] `M1-T02` Configure async SQLAlchemy + Alembic.
- [ ] `M1-T03` Migration: `organizations`, `sites`, `users`, `roles`, `user_roles`, `audit_logs`.
- [ ] `M1-T04` Add UUID PKs, timestamps, org scoping columns and indexes.
- [ ] `M1-T05` Implement password hashing with Argon2id.
- [ ] `M1-T06` Implement login/logout and refresh or session strategy (ADR-backed).
- [ ] `M1-T07` Implement permission dependency (`require_permission`) used by every route.
- [ ] `M1-T08` Implement organization/site scoping helpers; reject cross-org access.
- [ ] `M1-T09` Implement audit-log writer (actor, action, target, before/after, time).
- [ ] `M1-T10` Add rate limiting on auth endpoints.
- [ ] `M1-T11` Seed roles: Super Admin, Org Admin, Security Admin, Analyst L1/L2, Responder, Auditor, Viewer.
- [ ] `M1-T12` Implementation decision: MFA provider and enrollment flow (record; may be MVP+1).

**Frontend (TypeScript)**
- [ ] `M1-T13` Scaffold Next.js app with TypeScript strict + layout shell.
- [ ] `M1-T14` Build login page with error/loading states.
- [ ] `M1-T15` Build authenticated layout and sidebar shell.
- [ ] `M1-T16` Build Organization/Site management screens (list, create, edit).
- [ ] `M1-T17` Build Users & Roles screen (read-only first if needed).
- [ ] `M1-T18` Add permission-aware UI guards (server remains source of truth).

**Tests**
- [ ] `M1-T19` Tests: unauthorized role cannot access restricted route.
- [ ] `M1-T20` Tests: user in org A cannot read org B data (IDOR + tenant).
- [ ] `M1-T21` Tests: audit record created for create/update actions.
- [ ] `M1-T22` Milestone review.

### M2 — Endpoint identity & enrollment (Weeks 3–5)

**Definition of Done:** one Windows and one Linux test agent enroll with unique identity and show healthy/online in console; revocation and token expiry tested.

**Backend (Python)**
- [ ] `M2-T01` Migration: `endpoints`, `agent_credentials`, `agent_heartbeats`, `enrollment_tokens`.
- [ ] `M2-T02` Generate short-lived, single-use enrollment tokens (hash-only storage).
- [ ] `M2-T03` `POST /api/v1/agents/enroll` with token validation, replay rejection, audit.
- [ ] `M2-T04` Issue unique agent identity/certificate (dev CA first).
- [ ] `M2-T05` `POST /api/v1/agents/{id}/heartbeat` with auth + health payload.
- [ ] `M2-T06` `POST /api/v1/agents/renew` for certificate renewal.
- [ ] `M2-T07` Endpoint list/detail API with org/site scoping and last-seen state.
- [ ] `M2-T08` Revocation API + credential invalidation.
- [ ] `M2-T09` Offline detection: mark stale endpoints via scheduled job.

**Agent (Go)**
- [ ] `M2-T10` Agent skeleton: `cmd/ariba-agent`, config loader, structured logging.
- [ ] `M2-T11` Enrollment client: read one-time token, send host facts, persist identity securely.
- [ ] `M2-T12` Heartbeat loop with exponential backoff and jitter.
- [ ] `M2-T13` Bounded local queue (SQLite) with encryption-at-rest decision.
- [ ] `M2-T14` Windows service wrapper + install/uninstall PowerShell.
- [ ] `M2-T15` Linux systemd unit + DEB/RPM build script skeleton.
- [ ] `M2-T16` Certificate storage with restrictive file permissions.
- [ ] `M2-T17` Agent health/metrics endpoint or log evidence.

**Frontend**
- [ ] `M2-T18` Endpoints list with online/offline/stale states and filters.
- [ ] `M2-T19` Endpoint detail page (identity, OS, agent version, last seen).
- [ ] `M2-T20` Enrollment token UI (create/expire/revoke).

**Tests**
- [ ] `M2-T21` Test: reused enrollment token is rejected.
- [ ] `M2-T22` Test: expired token is rejected.
- [ ] `M2-T23` Test: revoked agent can no longer heartbeat.
- [ ] `M2-T24` Lab test: Windows + Linux agents enroll and appear online.
- [ ] `M2-T25` Milestone review (Milestone 01 demo).

### M3 — Telemetry pipeline (Weeks 6–9)

**Definition of Done:** a synthetic process event travels agent → API → queue → normalizer → OpenSearch → console timeline, with dedup and limits.

**Contracts**
- [ ] `M3-T01` Define Ariba event schema v1 (ECS-compatible subset) in `detection/` + docs.
- [ ] `M3-T02` Define event versioning and backward-compatibility rules.
- [ ] `M3-T03` Define sensitive-field policy (no file content/command line unless approved).

**Ingestion (Python)**
- [ ] `M3-T04` `POST /api/v1/telemetry/events` with schema validation and tenant/agent checks.
- [ ] `M3-T05` Payload size limits, rate limits and batch acceptance.
- [ ] `M3-T06` Idempotency/dedup key strategy.
- [ ] `M3-T07` Publish to RabbitMQ with retry and dead-letter queue.
- [ ] `M3-T08` Ingestion worker: consume, normalize, enrich endpoint context.
- [ ] `M3-T09` Write to OpenSearch with index template, mapping and retention policy.
- [ ] `M3-T10` Backpressure and bounded-batch handling.
- [ ] `M3-T11` Observability: event counters, lag, failure metrics.

**Agent collectors (Go)**
- [ ] `M3-T12` Collector interface: start/stop, bounded buffer, backpressure.
- [ ] `M3-T13` Linux process collector (auditd or osquery adapter).
- [ ] `M3-T14` Linux file collector (inotify).
- [ ] `M3-T15` Linux network collector (connection snapshot/events).
- [ ] `M3-T16` Windows process collector (ETW + Event Log).
- [ ] `M3-T17` Windows file collector (ReadDirectoryChangesW / USN).
- [ ] `M3-T18` Windows network collector (ETW/network events).
- [ ] `M3-T19` Per-OS resource budget measurement (CPU/RAM/disk queue).
- [ ] `M3-T20` Offline behavior: queue, resume, no data loss beyond bounded limit.

**Frontend**
- [ ] `M3-T21` Endpoint timeline view (process/file/network events).
- [ ] `M3-T22` Loading/empty/stale/error state handling.

**Tests**
- [ ] `M3-T23` Integration test: synthetic event reaches OpenSearch.
- [ ] `M3-T24` Test: oversized/malformed event rejected safely.
- [ ] `M3-T25` Load test at target EPS; record measured numbers.
- [ ] `M3-T26` Milestone review.

### M4 — Detection engine & alerts (Weeks 10–13)

**Definition of Done:** versioned rules evaluate events, produce explainable alerts with ATT&CK context, and alert lifecycle works.

**Detection (Python + YAML)**
- [ ] `M4-T01` Rule schema: ID, severity, platform, telemetry, logic, ATT&CK, FP notes, tests.
- [ ] `M4-T02` Rule loader + validator (reject malformed/unsigned production rules).
- [ ] `M4-T03` Rule evaluator for v1 conditions (process/file/network).
- [ ] `M4-T04` Correlation window support (e.g. repeated events).
- [ ] `M4-T05` Transparent risk scoring (`Severity × Confidence × Impact`).
- [ ] `M4-T06` Rule provenance and version recorded on each alert.
- [ ] `M4-T07` Ship 5–10 starter rules with safe test cases and FP notes.
- [ ] `M4-T08` Rule unit-test harness with synthetic safe fixtures.

**Alerts**
- [ ] `M4-T09` Migration: `detection_rules`, `security_alerts`, `incidents`.
- [ ] `M4-T10` Alert creation from detection result; link event reference.
- [ ] `M4-T11` Alert lifecycle API: new → investigating → resolved (+ assignment).
- [ ] `M4-T12` Incident creation and timeline API.

**Malware integration**
- [ ] `M4-T13` Scan adapter interface; YARA + ClamAV integration.
- [ ] `M4-T14` Microsoft Defender status/scan adapter (read/trigger, never disable).
- [ ] `M4-T15` Quarantine record metadata (no destructive action yet).
- [ ] `M4-T16` Validate with EICAR in an isolated lab only.

**Frontend**
- [ ] `M4-T17` Alerts list with severity/status filters.
- [ ] `M4-T18` Alert detail: evidence, endpoint context, timeline, ATT&CK.
- [ ] `M4-T19` Alert status/assignment actions with permission checks.
- [ ] `M4-T20` Investigation view (event evidence + notes).

**Tests**
- [ ] `M4-T21` Test: each starter rule fires on its fixture and not on clean data.
- [ ] `M4-T22` Test: alert contains rule version, evidence and ATT&CK mapping.
- [ ] `M4-T23` Milestone review (detect → alert demo).

### M5 — Policy engine (Weeks 14–16)

**Definition of Done:** policies are versioned and signed; agents reject unsigned/tampered policy; ack and rollback are auditable.

- [ ] `M5-T01` Policy schema (scanning, monitoring, updates, retention knobs).
- [ ] `M5-T02` Migration: `security_policies`, `policy_assignments`, `policy_versions`.
- [ ] `M5-T03` Policy authoring API + validation.
- [ ] `M5-T04` Versioning and approval state.
- [ ] `M5-T05` Signing pipeline (Ed25519/minisign) and key management plan.
- [ ] `M5-T06` Assignment by org/site/group with staged rollout.
- [ ] `M5-T07` Acknowledgement tracking and status.
- [ ] `M5-T08` Rollback to previous signed version.
- [ ] `M5-T09` Agent: `GET /agents/{id}/policy`, signature verification, apply.
- [ ] `M5-T10` Agent: reject unsigned policy and unauthorized rollback; log reason.
- [ ] `M5-T11` Policy UI: list, edit, version history, assignment, ack status.
- [ ] `M5-T12` Tests: tampered/unsigned policy rejected; rollback audited.
- [ ] `M5-T13` Milestone review.

### M6 — Guarded response (Weeks 15–17, lab only)

**Definition of Done:** an approved action (kill/quarantine/isolate) executes on a lab endpoint with authorization, expiry, audit and rollback; unsupported actions default-deny.

- [ ] `M6-T01` Migration: `response_actions` with approval and execution state.
- [ ] `M6-T02` Action catalog + explicit deny list for unsupported/risky actions.
- [ ] `M6-T03` Approval workflow (requester ≠ approver where possible), reason required.
- [ ] `M6-T04` Target revalidation against recent endpoint context.
- [ ] `M6-T05` Command expiry and single-use action tokens.
- [ ] `M6-T06` Command delivery via agent polling (outbound-only) or secure channel.
- [ ] `M6-T07` Agent handlers: kill process, quarantine file, isolate network.
- [ ] `M6-T08` Isolation preserves a narrow management/recovery channel.
- [ ] `M6-T09` Result reporting and completion/failure states.
- [ ] `M6-T10` Release/restore procedures and rollback.
- [ ] `M6-T11` Audit every step (who, when, why, target, result).
- [ ] `M6-T12` UI: request, approve, track, and view audit.
- [ ] `M6-T13` Lab tests: unauthorized user blocked; expiry respected; rollback works.
- [ ] `M6-T14` Milestone review (guarded-response demo, lab only).

### M7 — Integrations (Weeks 18–21)

- [ ] `M7-T01` Write versioned integration contracts in `docs/architecture/`.
- [ ] `M7-T02` Wazuh/Ariba SOC event forwarder.
- [ ] `M7-T03` Wazuh/Ariba SOC response-request handoff with auth.
- [ ] `M7-T04` MISP IOC import with dedup and provenance.
- [ ] `M7-T05` MISP IOC export/feedback.
- [ ] `M7-T06` OpenSearch index lifecycle and dashboards.
- [ ] `M7-T07` Notification service (email/webhook) with retry and rate limits.
- [ ] `M7-T08` Integration failure-mode tests (timeout, duplicate, bad payload).
- [ ] `M7-T09` Milestone review.

### M8 — Inventory & vulnerability context (Weeks 18–21, P1)

- [ ] `M8-T01` Software inventory collector (Windows registry/ARP, Linux package DB).
- [ ] `M8-T02` Migration + API for `software_inventory`.
- [ ] `M8-T03` Inventory UI per endpoint and fleet.
- [ ] `M8-T04` CVE lookup adapter (NVD/OSV) with caching.
- [ ] `M8-T05` Migration + API for `vulnerability_findings`.
- [ ] `M8-T06` Findings UI with severity and patch status.
- [ ] `M8-T07` Milestone review.

### M9 — Hardening & security validation (Weeks 22–24)

- [ ] `M9-T01` Authorization matrix test across every role and route.
- [ ] `M9-T02` Tenant-isolation test suite (read + write + delete).
- [ ] `M9-T03` Enrollment replay/expiry/revocation adversarial tests.
- [ ] `M9-T04` Signed-policy tamper and rollback-attack tests.
- [ ] `M9-T05` Rate-limit and abuse tests on auth/telemetry/response.
- [ ] `M9-T06` SAST + dependency scan + secret scan + SBOM generation in CI.
- [ ] `M9-T07` Load/soak test and documented capacity numbers.
- [ ] `M9-T08` Backup/restore and migration-rollback drill.
- [ ] `M9-T09` Update/rollback drill for agent and server.
- [ ] `M9-T10` Privacy review: telemetry fields, retention, deletion.
- [ ] `M9-T11` Independent security assessment (if available).
- [ ] `M9-T12` Update threat model with findings; close or accept risks.
- [ ] `M9-T13` Milestone review.

### M10 — Packaging & pilot (Weeks 22–26, P1)

- [ ] `M10-T01` Windows MSI with service install, signed binary.
- [ ] `M10-T02` Linux DEB/RPM packages, signed, with systemd unit.
- [ ] `M10-T03` Agent upgrade and rollback path.
- [ ] `M10-T04` Pilot Docker Compose deployment with hardened defaults.
- [ ] `M10-T05` TLS, secrets, backup and monitoring setup.
- [ ] `M10-T06` Pilot onboarding checklist (authorized endpoints only).
- [ ] `M10-T07` Rollback and recovery runbook.
- [ ] `M10-T08` Known-limitations document (explicitly not full AV).
- [ ] `M10-T09` Support and incident-response runbook.
- [ ] `M10-T10` Pilot acceptance test and sign-off.
- [ ] `M10-T11` Milestone review / release retrospective.

---

## 4. Always-on cross-cutting backlog

These are not one milestone; add them to every milestone as needed.

- [ ] `X-T01` Documentation updated with every feature (status: confirmed/proposed).
- [ ] `X-T02` Audit event for every state-changing action.
- [ ] `X-T03` Authorization check server-side for every route.
- [ ] `X-T04` Loading / empty / stale / permission-denied / partial-connection UI states.
- [ ] `X-T05` Bangla + English strings and terminology (i18n from the start).
- [ ] `X-T06` Structured logging with no secrets or unnecessary personal data.
- [ ] `X-T07` Metrics, health checks and alerting for each service.
- [ ] `X-T08` Error handling and safe failure modes (fail safe, not fail open).
- [ ] `X-T09` Data retention and deletion jobs.
- [ ] `X-T10` Accessibility and keyboard navigation on console screens.
- [ ] `X-T11` Dependency, license and SBOM review before each release.
- [ ] `X-T12` Safe test data only (no real endpoint data, no live malware).

---

## 5. How to track this

1. Create GitHub milestone `M0`…`M10`.
2. Create one issue per checkbox above, title = task ID + short name.
3. Labels: `area:backend`, `area:frontend`, `area:agent`, `area:detection`, `area:infra`; `type:feature`, `type:test`, `type:docs`, `type:security`; `priority:P0/P1/P2`.
4. Board columns: Backlog → Ready → In Progress → Review → Done.
5. A task is **Done** only when: code merged, tests pass, docs updated, and evidence linked in the issue.
6. Weekly review: what moved, what is blocked, what is next. Update milestone % only from real Done issues.
7. Never move a task to Done because "it should work" — attach a test, screenshot or command output.

### Definition of Done template

```text
- [ ] Implementation merged via reviewed PR
- [ ] Unit/integration test added and passing
- [ ] Authorization and tenant-scope checked (if route/data)
- [ ] Audit event emitted (if state-changing)
- [ ] Docs updated (status + API/data contract)
- [ ] Evidence linked (test output, screenshot, or demo note)
- [ ] No secrets, keys, real endpoint data or live malware added
```

---

## 6. Suggested first two weeks (concrete start)

1. `M0-T01`–`M0-T05`: confirm scope, name, platforms and record ADR-001/002.
2. `M0-T10`: draft threat model.
3. `M0-T12`–`M0-T16`: local stack + lint/test CI.
4. `M1-T01`–`M1-T03`: API skeleton and first migrations.
5. `M1-T05`–`M1-T07`: login + permission dependency.
6. `M1-T13`–`M1-T15`: console shell + login page.
7. Demo: log in, create org/site, see audit record.

---

## 7. Language decision summary (one line each)

- **Python** — control plane, workers, detection, integrations.
- **TypeScript** — web console.
- **Go** — endpoint agents.
- **SQL** — PostgreSQL schema and migrations.
- **YAML / Sigma / YARA** — rules, policies, configuration.
- **PowerShell / Bash** — packaging and operations.
- **JSON / Protocol Buffers (later)** — event and agent wire formats.

This mix is appropriate for an enterprise endpoint security product: fast product development where it matters most, and a small, efficient agent where the endpoint cannot afford overhead.
