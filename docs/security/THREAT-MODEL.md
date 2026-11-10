# Ariba EPP — Threat Model

**Status:** Initial threat-model draft; security-owner review required before privileged features or pilot deployment.
**Scope:** Management console/API, Platform/Group/Company/Site hierarchy, broker/workers, endpoint agents, policies/updates, response path and external integrations.

## 1. Security objectives

- Protect endpoint and customer identity, telemetry, policy, alerts, quarantine and evidence.
- Prevent unauthorized policy changes, telemetry injection, cross-Group/Company/Site access and response execution.
- Preserve integrity and provenance of agent binaries, updates, policies, rules and audit records.
- Keep existing OS security active if EPP services fail.
- Make privileged actions attributable, bounded, recoverable and observable.
- Minimize collection and exposure of personal/customer data.

## 2. Assets

1. User identities, sessions, MFA state and role grants.
2. Enrollment tokens, endpoint certificates/private keys and service credentials.
3. Parent Group/Company/Site membership, endpoint identity and software inventory.
4. Process/file/network telemetry and source/collection metadata.
5. Security policies, signing keys, detection rules and threat-intel source data.
6. Alerts, any approved incident cases, audit records and response commands/results.
7. Quarantined files, forensic artifacts, backups and encryption keys.
8. Integrations, tenant quotas, deployment configuration and operational logs.

## 3. Trust boundaries

- Browser/user to web/API gateway.
- Authenticated Platform/Group/Company/Site principal to APIs and administrative workflows.
- Privileged agent to management server.
- API gateway to broker and worker services.
- Workers to PostgreSQL, OpenSearch, Redis and object storage.
- Administrator/policy signer to agent policy channel.
- Ariba SOC/Wazuh/MISP and scanner adapters to EPP services.
- Build/release pipeline to signed agent/update distribution.
- Parent Group A to Group B, sibling Company and out-of-scope Site access paths, including search/cache/queue/object-storage/export.

## 4. Threat scenarios and required controls

| Threat | Impact | Required controls / verification |
|---|---|---|
| Stolen admin session or overprivileged user | Tenant-wide policy/response misuse | MFA, least privilege, scope-limited roles, session controls, server-side authorization, audit and role-abuse tests |
| Reused/stolen enrollment token | Rogue endpoint enrollment | Short expiry, one-time use, Group/Company/Site scope, rate limits, secure hash storage and revocation tests |
| Agent certificate theft/replay | Telemetry spoofing or command access | Unique certificate, protected key, rotation/revocation, channel binding and replay/idempotency tests |
| Cross-Group/Company/Site IDOR or query bug | Data disclosure or unauthorized cross-company action | Scope derived from identity/resource relationships; deny-by-default; negative API/worker/search/cache/queue/object/export tests at each hierarchy level |
| Forged telemetry or event flood | False alert, resource exhaustion or rule confusion | Authenticated agent, schema validation, size/rate/batch limits, dedupe, backpressure and monitoring |
| Malicious or rollback policy | Disable protection or misconfigure endpoints | Signed versioned policy, staged rollout, anti-rollback, scope confirmation, approvals and recovery |
| Tampered agent/update | Fleet compromise | Signed builds, protected signing, secure pipeline, provenance/SBOM, staged rollout and rollback |
| Unauthorized response request | Process/file/network disruption | Separate integration scope, deny-by-default, approval, target validation, expiry, action allowlist and audit |
| Endpoint isolation cuts management path | Endpoint unrecoverable | Preserve restricted management channel, tested release/recovery and pre-rollout validation |
| Quarantine/evidence leak | Sensitive file/data disclosure | Restricted object access, encryption, hash/integrity, retention/deletion and access audit |
| Broker/search/storage outage | Lost visibility or unbounded queue growth | Bounded queues, backpressure, health alerts, retry/idempotency, capacity and restore tests |
| Supply-chain/dependency compromise | Platform/agent compromise | Dependency review/scanning, SBOM, signed artifacts, build access controls and release review |
| Telemetry overcollection | Privacy/legal risk | Data inventory, allowlists, minimization, retention, access control, notice/approval and deletion test |
| Audit tampering/deletion | Loss of accountability | Append-oriented/tamper-evident design, restricted DB roles, export monitoring and integrity review; do not claim immutability without evidence |
| AI hallucination or unsafe suggestion | Analyst misled or action improperly taken | Evidence references, uncertainty labels, human review; no autonomous destructive action |

## 5. Security requirements before pilot

- Security owner approves threat model and residual-risk handling.
- Owner-approved Platform/Group/Company/Site permission matrix; negative hierarchy-isolation tests pass.
- User MFA/identity choice and privileged account recovery are defined.
- Agent enrollment, certificate rotation/revocation and signed package process are tested.
- Signed policy verification and rollback recovery are tested.
- Data inventory, collection minimization, retention, deletion and residency decisions are approved.
- Response permission/approval matrix is approved; disruptive response remains disabled until action-specific review.
- Dependency licenses, SBOM and vulnerability review are complete.
- Encrypted backups, access controls and successful restore exercise are evidenced.
- Security monitoring covers privileged changes, enrollment anomalies and failed verification.
- Incident response, vulnerability reporting and disclosure contact are finalized before external use.

## 6. Residual risks and open questions

- A privileged endpoint agent can create system risk if buggy, compromised or updated unsafely; independent review and staged rollout are required.
- Telemetry coverage and prevention effectiveness vary by OS version/configuration; no universal coverage claim is permitted.
- Hosted multi-organization hierarchy is approved in ADR-009; provider/region, data residency and concrete tenant-isolation mechanism remain unresolved.
- Exact Group SOC access to subsidiary-company evidence and Site Admin endpoint/token/health permissions remain blocked on RBAC approval.
- Standalone incident-case lifecycle and its data retention are not yet approved.
- Isolation's exact network-control mechanism and recovery path are unresolved.
- Signing-key custody, author/signer separation and emergency key rotation are unresolved.
- Regulatory/compliance scope is not specified; do not claim compliance.

## 7. Security review gate

Before enabling any action that changes endpoint state, review its threat scenarios, least-privilege mechanism, target freshness, user approval, failure mode, audit fields, rollback/recovery, isolated-lab tests and operational ownership. “Works on my machine” is not release evidence.
