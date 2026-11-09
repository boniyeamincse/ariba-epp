# Ariba EPP — Threat Model

**Status:** Initial threat-model draft; requires security-owner review before implementation of privileged features or pilot deployment.  
**Scope:** Management console/API, tenant data, broker/workers, endpoint agents, policies/updates, response path and external integrations.

## 1. Security objectives

- Protect endpoint and tenant identity, telemetry, policy, alerts, quarantine and evidence.
- Prevent unauthorized policy changes, telemetry injection, cross-tenant access and response execution.
- Preserve integrity and provenance of agent binaries, updates, policies, rules and audit records.
- Keep existing OS security active if EPP services fail.
- Make privileged actions attributable, bounded, recoverable and observable.
- Minimize collection and exposure of personal/customer data.

## 2. Assets

1. User identities, sessions, MFA state and role grants.
2. Enrollment tokens, endpoint certificates/private keys and service credentials.
3. Endpoint identity, software inventory, process/file/network telemetry.
4. Security policies, signing keys, detection rules and threat-intel source data.
5. Alerts, incidents, audit records and response commands/results.
6. Quarantined files, forensic artifacts, backups and encryption keys.
7. Tenant/site mapping, integrations and deployment configuration.

## 3. Trust boundaries

- Browser/user to public or internal web/API gateway.
- Agent running with OS privileges to management server.
- API gateway to broker and worker services.
- Worker to PostgreSQL, OpenSearch, Redis and object storage.
- Administrator/policy signer to agent policy channel.
- Ariba SOC/Wazuh/MISP and scanner adapters to EPP services.
- Build/release pipeline to signed agent/update distribution.
- Tenant A records to tenant B access paths, including search/cache/export.

## 4. Threat scenarios and required controls

| Threat | Impact | Required controls / verification |
|---|---|---|
| Stolen admin session or overprivileged user | Tenant-wide policy/response misuse | MFA, least privilege, scoped roles, session controls, server-side authorization, audit and role abuse tests |
| Reused/stolen enrollment token | Rogue endpoint enrollment | Short expiry, one-time use, scoped token, rate limits, secure storage/hash, revocation tests |
| Agent certificate theft/replay | Telemetry spoofing or command access | Unique cert, protected key, rotation/revocation, channel binding, replay/idempotency checks |
| Cross-tenant IDOR/query bug | Data disclosure or cross-company actions | Tenant scope derived from identity, RLS consideration, negative API/worker/search/export tests |
| Forged telemetry or event flood | False alert, resource exhaustion, rule confusion | Authenticated agent, schema validation, size/rate/batch limits, dedupe, backpressure and anomaly monitoring |
| Malicious or rollback policy | Disable protection or misconfigure many endpoints | Signed versioned policy, staged rollout, anti-rollback, scope confirmation, approvals and recovery |
| Tampered agent/update | Fleet compromise | Signed builds, trusted update keys, secure pipeline, provenance/SBOM, staged rollout and rollback |
| Unauthorized response request | Process/file/network disruption | Separate integration scope, deny-by-default, approval, target validation, expiry, action allowlist and audit |
| Endpoint isolation cuts management path | Endpoint unrecoverable | Preserve restricted management channel, tested release/break-glass and pre-rollout validation |
| Quarantine/evidence leak | Sensitive file/data disclosure | Restricted object access, encryption, hash/integrity, retention/deletion, access audit |
| Broker/search/storage outage | Lost visibility or queued data growth | Bounded queues, backpressure, health alerts, retry/idempotency, capacity and restore tests |
| Supply-chain/dependency compromise | Platform/agent compromise | Dependency pin/review, scanning, SBOM, signed artifacts, build access controls and release review |
| Telemetry overcollection | Privacy/legal risk | Data inventory, allowlists, minimization, retention, access control, notice/approval and deletion test |
| Audit tampering/deletion | Loss of accountability | Append-oriented/tamper-evident storage, restricted DB roles, export/monitoring and integrity review |
| AI hallucination or unsafe suggestion | Analyst misled or action improperly taken | Evidence references, uncertainty label, human review; no autonomous destructive action |

## 5. Security requirements before pilot

- Approved threat model and security owner.
- User MFA/role matrix and organization/site authorization tests.
- Agent enrollment, certificate rotation/revocation and signed package process.
- Signed policy verification and rollback recovery.
- Data inventory, collection minimization, retention, deletion and residency decisions.
- Response permission/approval matrix; response remains disabled until action-specific review.
- Dependency licenses, SBOM and vulnerability review.
- Encrypted backups, access controls and successful restore exercise.
- Security logs/alerts for privileged changes, enrollment anomalies and failed verification.
- Incident response, vulnerability reporting and disclosure contact finalized before external use.

## 6. Residual risks and open questions

- A privileged endpoint agent can create system risk if buggy, compromised or updated unsafely; independent review and staged rollout are needed.
- Telemetry coverage and prevention effectiveness vary by OS version and configuration; no universal coverage claim is permitted.
- Tenant architecture, hosted deployment and data residency are unresolved.
- Isolation's exact network-control mechanism and recovery path are unresolved.
- Signing key custody and emergency rotation mechanism are unresolved.
- Regulatory/compliance scope is not specified; do not claim compliance.

## 7. Security review gate

Before enabling any action that changes endpoint state, review its threat scenarios, least-privilege mechanism, target freshness, user approval, failure mode, audit fields, rollback/recovery, lab tests and operational ownership. “Works on my machine” is not release evidence.