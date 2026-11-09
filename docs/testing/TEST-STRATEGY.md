# Ariba EPP — Test Strategy

**Status:** Draft; test environments and supported OS matrix are To confirm.  
**Safety:** owned/authorized isolated lab only; no live malware or production response actions.

## 1. Test principles

- Requirements use stable IDs from `PRODUCT-REQUIREMENTS.md`; each ID maps to test evidence.
- Security, privacy and tenant isolation are test concerns at every boundary, not a final-only phase.
- Use deterministic synthetic events and benign artifacts (for example EICAR) in isolated scanner tests.
- Do not disable existing antivirus/host controls.
- No secrets, customer telemetry or personal endpoint data in fixtures.
- Test both success and failure/negative paths; denial must be the default for unauthorized action.
- Distinguish `PASS`, `FAIL`, `BLOCKED` and `NOT RUN`; never report unrun checks as passed.

## 2. Test layers

| Layer | Scope |
|---|---|
| Unit | Validation, state machines, tenant filters, normalization, rule evaluation, signing checks |
| Contract | API schemas, event/policy version compatibility, Wazuh/MISP/scanner adapter contracts |
| Integration | API + DB, broker/workers, search store, enrollment and agent identity |
| Security | Authentication, authorization, tenant isolation, replay, signature failures, secret leakage and response safeguards |
| Agent platform | Per-supported Windows/Linux build: install, service, collector permissions, resource, update/uninstall |
| Reliability | Offline/reconnect, retry, bounded queue, dependency outage, duplicate event, clock skew and restore |
| Performance | EPS, event size, alert latency, search query, retention, CPU/memory/disk budget |
| E2E | Enrollment → heartbeat → inventory → synthetic event → alert → audit; later policy/response flow |
| UI | Role visibility, loading/empty/stale/error states, keyboard/accessibility and tenant-safe navigation |

## 3. Required test cases

### Foundation v0.1

- Valid authorized enrollment succeeds and produces unique endpoint identity.
- Expired/reused/revoked token, wrong scope and malformed request are rejected.
- Heartbeat updates server receipt/health; stale/offline and reconnect transitions are correct.
- Duplicate retry does not create duplicate endpoint or corrupt status.
- User from another tenant cannot list, read, mutate or infer endpoint existence.
- Missing/insufficient permission is rejected server-side even if UI control is bypassed.
- API/log/error output contains no tokens, certificates, private keys or unnecessary personal data.
- Agent offline queue stays within configured bounds and reports degraded state.
- Clean local setup uses safe development-only configuration and no production secret.

### Telemetry and detection (later gate)

- Valid event accepted; malformed, oversize, wrong schema, unknown endpoint and unauthorized tenant rejected.
- Duplicate event retry has deterministic idempotent behavior.
- Synthetic matching event triggers expected rule version; near-miss event does not.
- Alert includes rule version, explanation, timestamps and evidence reference.
- Rule disabled/unsupported platform behavior is explicit.
- Storage outage causes visible degradation and bounded retry, not silent success.

### Policy and response (later gate)

- Valid signed policy applies and is acknowledged.
- Unsigned, tampered, wrong-tenant, expired, incompatible and unauthorized rollback policy is rejected.
- Response disabled by default; unauthorized, unapproved, expired, replayed, wrong-target and unsupported actions fail closed.
- Successful lab action returns exact outcome and audit evidence; failed/offline action is never shown as success.
- Isolation test retains management recovery path; release action works before pilot approval.

## 4. Performance/load test plan

Do not accept sizing numbers from planning tables alone. First define endpoint count, event rate per endpoint, payload distribution, burst factor, retention, query mix and availability target. Then measure:

- Sustained and burst events per second; ingestion acceptance and end-to-end alert latency.
- Broker lag, worker recovery, duplicate rate and bounded queue behavior.
- OpenSearch indexing/search performance and retention rollover.
- PostgreSQL transactions, tenant-scoped query latency and backup restore time.
- Agent CPU, memory and disk under idle, normal telemetry, burst and scan workloads.
- Failure/recovery under component restart, network loss and storage pressure.

Record environment, versions, dataset, method, result and confidence. Do not extrapolate without stating assumptions.

## 5. Test environments and data

- Developer environment: local synthetic data; no real endpoints.
- Agent lab: isolated owned Windows and Linux VMs/devices with snapshots and documented reset.
- Integration environment: isolated dependencies, mock/staging identities and synthetic tenant data.
- Pilot: explicitly authorized endpoints, staged rollout, rollback and support owner; only after security gate.
- Production: separate approval, monitoring, backup/restore, incident response and release sign-off.

Never connect an automated test to arbitrary public IPs, customer endpoints or production services.

## 6. Release evidence checklist

- Traceable requirement/task IDs and current known limitations.
- Unit/contract/integration/E2E results for changed areas.
- Security and tenant-isolation negative-test results.
- Agent package signature, update/rollback and supported OS matrix evidence.
- Performance evidence against approved targets or explicit `NOT VERIFIED` status.
- Backup/restore and operational runbook review.
- Dependency/license/SBOM and secret scan results.
- Owner approval for pilot scope and any response action.