# Ariba EPP — Product Requirements

**Status:** Draft for owner review; no product behavior is approved by this draft alone.  
**Scope:** Enterprise Windows/Linux EPP, centralized control plane and endpoint agents.  
**Readers:** Product owner, engineering, security, QA and future coding agents.

## 1. Problem and product outcome

Organizations need a central view of endpoint security posture and a controlled way to enroll endpoints, apply security policies, investigate endpoint alerts and request authorized remediation. Ariba EPP is intended to provide its own endpoint identity, lifecycle, policy and detection control plane, integrating mature operating-system/security engines and publishing events to Ariba SOC/Wazuh.

**Outcome:** an authorized Windows and Linux test endpoint can enroll, report health/inventory, receive an approved policy, submit a test event, create an explainable alert, and show a complete audit trail. This outcome does not prove complete malware prevention.

## 2. Audience and roles

| Role | Primary need | Initial access boundary |
|---|---|---|
| Platform administrator | Configure platform, organizations and integrations | Global, privileged and audited |
| Organization administrator | Manage own organization, sites and endpoints | Own organization only |
| Security administrator | Author and assign security policies | Authorized organization/site scope |
| SOC analyst L1 | Triage alerts and assign investigations | Read/triage; no disruptive actions by default |
| SOC analyst L2 / incident responder | Investigate and request approved response | Specific action permissions and approval policy |
| Auditor | Review audit and reports | Read-only, scoped |
| Viewer | See permitted dashboard data | Read-only, scoped |

MFA provider and exact role permissions are **To confirm**. Enforcement must be server-side, not only hidden in UI.

### Deployment and tenancy model (owner-approved 2026-11-09)

Ariba EPP is a **centrally hosted multi-organization platform**: multiple customer organizations (for example Akij Group, Boni IT, Nazim Group) share one platform, and each customer is an `organization` tenant with its own sites, users, endpoints, policies, alerts and audit history. Platform Administrators manage organizations; Organization Administrators manage only their own organization. Cross-organization access is denied by default.

See [ADR-009](decisions/ADR-009-hosted-multi-tenant.md). Hosting region, data residency, isolation mechanism, per-tenant quotas/retention and billing remain **To confirm**.

## 3. Goals and non-goals

### Goals

- Centrally enroll/manage Windows and Linux endpoints through unique agent identities.
- Report agent/endpoint health, last-seen status and approved software/system inventory.
- Define, sign, version, assign and track endpoint policies.
- Ingest a versioned telemetry schema and produce traceable, explainable detection alerts.
- Support alert/incident triage and audited response requests.
- Provide organization/site scope, RBAC and audit history.
- Integrate with selected security engines and Ariba SOC/Wazuh/MISP through versioned contracts.
- Begin with observe-only/lab-safe operation and scale only after measured load tests.

### Non-goals for Foundation v0.1

- Replacing Microsoft Defender or any existing antivirus.
- A proprietary kernel-level prevention driver or complete antivirus engine.
- Guaranteed detection/prevention of every malware or ransomware family.
- Automated destructive response or AI-directed remediation.
- macOS/mobile agents, automated patch deployment, full application control, or complete XDR.
- Production deployment, availability guarantees or compliance certification before those are separately specified and tested.

## 4. Product and delivery scope

Long-term product capabilities use EPP-01 through EPP-14 priorities in the [Enterprise Architecture](ARIBA-EPP-ENTERPRISE-ARCHITECTURE.md#3-functional-modules-and-priorities). Foundation v0.1 must prove the management/agent path first:

1. User login shell and organization/site administration.
2. Tenant-scoped API and audit model.
3. Secure enrollment and unique endpoint identity.
4. Windows/Linux agent prototypes with heartbeat and basic inventory.
5. Endpoint status in console.
6. Development-only deployment and repeatable test environment.

Next release gates add events, detections and policies; then a single action-specific, lab-only response prototype may be considered after explicit owner/security approval; broader SOC/scanner integrations follow the approved dependency order. Do not implement kill, quarantine or isolation as a bundle just because the roadmap lists them. The exact feature set, sequence and supported OS builds are owner decisions.

## 5. Functional requirements

Each requirement needs a test or review artifact. `P0` is foundation/first production-oriented MVP planning priority, not a claim that the complete long-term capability ships in v0.1.

### Identity and tenancy

- **FR-ID-01 (P0):** The platform shall authenticate users and enforce permissions on every protected API request.
- **FR-ID-02 (P0):** Every organization-owned record shall be scoped to an organization; cross-organization reads/writes shall be denied and tested.
- **FR-ID-03 (P0):** Privileged user MFA is required before external pilot; provider and rollout are To confirm.
- **FR-ID-04 (P0):** Enrollment shall use an authorized, short-lived, single-use token and assign a unique endpoint identity.
- **FR-ID-05 (P0):** Agent identity can be revoked and renewed without sharing a reusable fleet-wide secret.

### Endpoint and agent management

- **FR-EP-01 (P0):** Authorized users can list and inspect endpoints within their permitted organization/site scope.
- **FR-EP-02 (P0):** Endpoint records include stable ID, hostname, platform/OS, agent version, organization/site, status and last-seen data; optional personal fields require privacy approval.
- **FR-EP-03 (P0):** Agent heartbeat reports health and supported version; stale/offline thresholds are configurable and documented.
- **FR-EP-04 (P0):** Agent reconnects after temporary network loss and uploads queued permitted events within configured bounds.
- **FR-EP-05 (P0):** Agent update supports signature verification, staged rollout and tested rollback before pilot.

### Policy management

- **FR-POL-01 (P0):** Policies are versioned, validated and scoped to organization/site/endpoint groups.
- **FR-POL-02 (P0):** Agent verifies policy authenticity and reports applied/rejected version and reason.
- **FR-POL-03 (P0):** Unsigned or unauthorized rollback policy is rejected.
- **FR-POL-04 (P0):** Policy changes record actor, reason, previous/new version, scope and outcome.

### Telemetry and detection

- **FR-EVT-01 (P0):** Ingestion validates schema version, endpoint identity, tenant scope, size and timestamps.
- **FR-EVT-02 (P0):** Events have deduplication/idempotency and ingestion status; sensitive fields follow the approved allowlist.
- **FR-DET-01 (P0):** Detection rules are versioned, enabled/disabled explicitly and linked to test evidence and required telemetry.
- **FR-DET-02 (P0):** A matched test event creates an alert with rule ID/version, endpoint, evidence reference, severity, explanation and timestamps.
- **FR-DET-03 (P0):** Alert lifecycle supports an owner-approved state machine. Candidate states are new, investigating, resolved and false-positive/accepted; exact states/transitions are To confirm in ADR-013.
- **FR-DET-04 (P1):** ATT&CK mapping and IOC enrichment are added only when mappings/source confidence are recorded.

### Incident response

- **FR-RSP-01 (P1, guarded):** Response actions are denied by default and require action-specific authorization and target validation.
- **FR-RSP-02 (P1, guarded):** The owner-approved ADR-014 approval policy, confirmation, reason and action expiry are enforced before a disruptive command is issued. Until approved, no response action is implemented/enabled.
- **FR-RSP-03 (P1, guarded):** Endpoint reports accepted/rejected/running/succeeded/failed/expired status with safe error detail.
- **FR-RSP-04 (P1, guarded):** Endpoint isolation preserves a restricted management/recovery channel and a tested release procedure.
- **FR-RSP-05 (P1, guarded):** Every action is auditable and has an operator recovery/rollback path.
- **FR-RSP-06 (P2):** AI may summarize evidence or suggest next steps but cannot execute a disruptive action autonomously.

### Audit and integrations

- **FR-AUD-01 (P0):** Authentication, enrollment, policy, permission and response changes generate protected audit records.
- **FR-INT-01 (P1):** Wazuh/Ariba SOC events use a versioned integration contract with retry and duplicate handling.
- **FR-INT-02 (P1):** MISP IOC import/export records source, timestamp, confidence and expiry.
- **FR-INT-03 (P0 discovery):** Integration failure must not corrupt EPP's source-of-truth endpoint/policy state.

## 6. Quality requirements

- **Security:** threat-model-based controls, least privilege, encrypted transport, signed agent/update/policy, tenant isolation and secret handling.
- **Privacy:** data minimization, collection inventory, retention/deletion, access logging and data-residency decision before pilot.
- **Reliability:** bounded retries/queues, idempotent operations, explicit failure states and recovery tests.
- **Performance:** publish event throughput, end-to-end latency and agent resource targets only after representative lab measurements.
- **Scalability:** architecture can scale ingestion/search independently; supported maximum endpoint count remains To confirm pending load tests.
- **Usability:** alert reason and response risk understandable; Bangla/English requirement is a product decision to confirm.
- **Maintainability:** versioned API/event/policy schemas, migrations, tests, deployment instructions and known limitations.
- **Accessibility:** keyboard-accessible core console flows and clear status/permission messaging (detailed UI criteria later).

## 7. User workflows

### Enroll and verify endpoint

Admin creates scoped one-time enrollment authorization → agent proves authorization → server issues unique identity → agent sends authenticated heartbeat → console shows pending/healthy state → admin verifies endpoint/organization mapping and audit record.

### Investigate a test alert

Agent submits approved synthetic test event → platform validates tenant/agent/schema → detection rule evaluates → alert stores explanation/evidence reference → analyst triages and records disposition → EPP sends normalized alert to SOC integration if enabled.

### Request a response action (later phase)

Authorized analyst selects a supported action and exact target → platform revalidates target and scope → approval/confirmation as required → expiring command sent → agent checks signature/expiry/target → result is returned → audit and recovery guidance are visible. Failure to contact agent leaves action pending/expired; it must not be reported as succeeded.

## 8. Acceptance criteria for Foundation v0.1

1. One owned Windows lab endpoint and one owned Linux lab endpoint enroll through the approved flow.
2. Each has a unique identity and is bound to the correct test organization/site.
3. Invalid, expired, reused and revoked enrollment tokens are rejected.
4. Authorized user sees only permitted tenant/site records; cross-scope attempts fail in API tests.
5. Heartbeat/last-seen reflects simulated loss and recovery according to documented threshold.
6. Console displays endpoint identity, OS/agent version and status without fabricated telemetry.
7. Agent disconnect/reconnect does not lose identity or duplicate endpoint records.
8. Audit captures actor/agent, operation, scope, timestamp and outcome without logging secrets.
9. Local dev deployment can be started from documented instructions without production secrets.
10. Tests are repeatable and isolated; no live malware or production endpoint action is used.

## 9. Decisions still required from owner

- Exact Windows releases and Linux distributions/kernel support matrix.
- Hosting provider/region and data residency for the centrally hosted multi-organization platform (deployment model approved in ADR-009).
- User identity provider/MFA and initial organization/site permission matrix.
- Fields considered personal/sensitive; assigned-user collection and retention.
- Heartbeat interval, stale/offline threshold, telemetry allowlist and retention.
- Initial event schema and which process/file/network fields are acceptable.
- Which component owns authority for integration-originated response requests and whether a requester may ever approve their own action (ADR-014).
- Which P0 features are required at Foundation v0.1 versus later.
- UI languages, design system and accessibility target.
- Expected endpoint/EPS volume, availability and recovery objectives.
- Dependency licensing/commercial redistribution review.

## 10. Change control

Requirement IDs are stable references for tasks, tests and PRs. Change scope or acceptance by editing this document and relevant contracts first, recording decision owner/date/status, then updating the implementation plan. Do not silently weaken a security/acceptance requirement to make a test pass.