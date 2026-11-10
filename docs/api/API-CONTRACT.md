# Ariba EPP — API Contract (Draft)

**Status:** Proposed contract; not an implemented API or final OpenAPI definition.  
**Candidate base path:** `/api/v1`  
**Candidate backend:** FastAPI; confirm after repository/toolchain review.

## 1. Contract rules

- JSON over authenticated TLS; agent identity uses unique certificate credentials, mutual TLS preferred.
- User APIs require user authentication, action permission and organization/site scope. Agent APIs require endpoint identity and endpoint-scoped permissions. Integration APIs have separate credentials/scopes.
- Authorization occurs server-side on every request. IDs in URL/body do not grant access.
- Use UTC RFC 3339 timestamps; identify schema/API versions explicitly.
- Use request IDs and idempotency keys for retryable writes. Agent event IDs are unique per endpoint/producer scope.
- Enforce body-size, field-length, rate and batch limits. Validate all enums and unknown-field behavior.
- Never return reusable secrets. Secret material is write-only where necessary and redacted from errors/logs.
- Breaking changes require a new version or documented compatibility window.

## 2. Common response and error model

Success response example:

```json
{
  "data": {},
  "request_id": "opaque-request-id"
}
```

Error response example:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request could not be validated",
    "details": [{"field": "platform", "reason": "unsupported_value"}]
  },
  "request_id": "opaque-request-id"
}
```

Do not disclose cross-tenant record existence in error messages. Common HTTP behavior: `400` malformed request, `401` unauthenticated, `403` unauthorized, `404` absent or intentionally hidden, `409` state/idempotency conflict, `413` payload too large, `422` schema/semantic validation, `429` rate limit, `503` dependency unavailable. Final mapping remains to confirm.

## 3. Proposed endpoint inventory

| Method | Route | Principal | Purpose |
|---|---|---|---|
| POST | `/agents/enroll` | One-time enrollment authorization | Enroll agent and issue unique identity |
| POST | `/agents/renew` | Existing agent identity | Renew credentials |
| GET | `/agents` | User with endpoint-read scope | List scoped endpoints |
| GET | `/agents/{endpoint_id}` | User with endpoint-read scope | Endpoint detail |
| POST | `/agents/{endpoint_id}/heartbeat` | That endpoint agent | Health/last-seen update |
| POST | `/telemetry/events` | Enrolled agent | Submit bounded event batch |
| GET | `/agents/{endpoint_id}/policy` | That endpoint agent | Get signed assigned policy |
| POST | `/policies` | Policy-admin user | Create draft policy |
| PUT | `/policies/{policy_id}` | Policy-admin user | Create/update version; final semantics TBD |
| POST | `/policies/{policy_id}/assignments` | Policy-admin user | Assign policy to authorized scope |
| GET | `/alerts` | User with alert-read scope | List scoped alerts |
| GET | `/alerts/{alert_id}` | User with alert-read scope | Alert detail/evidence references |
| PATCH | `/alerts/{alert_id}` | User with alert-triage scope | Change permitted alert state/assignment |
| POST | `/incidents` | User with incident-write scope | Create investigation |
| GET | `/incidents/{incident_id}` | User with incident-read scope | Incident and event timeline |
| POST | `/response/actions` | Authorized responder/integration identity | Request guarded action |
| GET | `/response/actions/{action_id}` | Scoped user/endpoint | Action status/result |
| POST | `/malware/scans` | Authorized user or agent | Request scan; exact flow to confirm |
| GET | `/inventory/software` | Scoped inventory reader | Software inventory |
| GET | `/vulnerabilities` | Scoped security reader | Vulnerability findings |
| POST | `/iocs` | Threat-intel administrator/integration | Import IOC with provenance |
The authoritative alert state machine and allowed transitions are **To confirm** in ADR-013. Candidate states are listed in `PRODUCT-REQUIREMENTS.md`; do not implement alert state mutation until ADR-013 and matching requirements/tasks are approved.

## 4. Enrollment contract sketch

Request fields: short-lived single-use `enrollment_token`, `hostname`, `platform`, `agent_version`, optional requested organization/site claim that server must independently authorize. No client-supplied organization ID is trusted without token scope.

```json
{
  "enrollment_token": "ONE_TIME_TOKEN",
  "hostname": "ORG-WS-001",
  "platform": "windows",
  "agent_version": "0.1.0"
}
```

Response sketch:

```json
{
  "data": {
    "endpoint_id": "opaque-id",
    "status": "enrolled",
    "identity_enrollment": "required",
    "policy_version": 1
  },
  "request_id": "opaque-request-id"
}
```

Actual key/certificate issuance protocol, token storage hash, expiry and renewal/revocation are **To confirm** and security-reviewed before coding. Never return a fleet-wide reusable secret.

## 5. Heartbeat contract sketch

Agent submits `agent_version`, `platform`, `health`, `policy_version`, `reported_at`, `boot/session identifier` as approved, and an idempotency/event ID. Server sets its own `received_at` and derives online state from configured thresholds.

```json
{
  "event_id": "unique-agent-event-id",
  "reported_at": "2026-11-09T10:00:00Z",
  "agent_version": "0.1.0",
  "health": "healthy",
  "policy_version": 1
}
```

## 6. Telemetry contract sketch

An event envelope contains `schema_version`, `event_id`, `event_type`, endpoint identity from authenticated context, `event_time`, producer/agent version, platform and an allowlisted typed payload. Organization scope is derived server-side from the authenticated agent, not trusted from payload.

```json
{
  "schema_version": "1.0",
  "event_id": "endpoint-local-unique-id",
  "event_type": "process_created",
  "event_time": "2026-11-09T10:00:00Z",
  "payload": {
    "process_name": "example.exe",
    "process_id": 4321,
    "parent_name": "example-parent.exe"
  }
}
```

Command lines, file paths, user identifiers and network details can be sensitive; collect only fields approved in the data-classification/telemetry decision. Define size, batch, enum, ordering and duplicate semantics before implementing ingestion.

## 7. Response request contract (later, guarded)

Request must include `endpoint_id`, supported `action_type`, exact target reference, request reason, idempotency key and approval reference required by ADR-014. Approval requirements, requester/approver separation and any exception are **To confirm**; until ADR-014 is approved, response endpoints are specification-only and implementation is blocked. Server records authenticated actor and derives tenant scope. Never accept arbitrary shell/script text as a response action.

```json
{
  "endpoint_id": "opaque-id",
  "action_type": "quarantine_file",
  "target": {"file_identity": "opaque-agent-reference", "sha256": "expected-hash"},
  "reason": "Analyst-approved test response",
  "approval_reference": "approved-workflow-id",
  "idempotency_key": "unique-request-key"
}
```

Response states: `pending_approval`, `queued`, `accepted`, `running`, `succeeded`, `failed`, `rejected`, `expired`, `cancelled`. State transitions, cancellation and approval rules must be frozen before response implementation. Endpoint verifies command signature, expiry, action allowlist and target identity; failure is audited and never reported as success.

## 8. Authorization matrix (minimum)

| Operation | Platform admin | Org admin | Security admin | Analyst L1 | Analyst L2 / responder | Auditor/viewer |
|---|---:|---:|---:|---:|---:|---:|
| Read endpoint in scope | Yes | Own org | Assigned scope | Assigned scope | Assigned scope | Read-only scope |
| Enroll/revoke endpoint | Yes | Own org, policy TBD | Explicit permission | No | No by default | No |
| Create/assign policy | Yes | Policy TBD | Assigned scope | No | No by default | No |
| Read/triage alert | Yes | Own org | Assigned scope | Yes | Yes | Read-only |
| Request disruptive action | No implicit grant | No implicit grant | No implicit grant | No | Explicit permission + approval | No |
| Manage roles/integrations | Yes | Own org policy TBD | No by default | No | No | No |

This matrix is a safe baseline, not a final product-owner role design. Deny by default until approved.

## 9. API acceptance tests

- Unauthenticated, expired and revoked identities fail.
- Cross-tenant IDs fail without revealing object existence.
- Reused enrollment token fails; duplicate endpoint enrollment is handled idempotently or rejected according to approved policy.
- Oversized/invalid event batches fail without partial silent corruption.
- Event retries with the same idempotency identity do not create duplicate alerts unintentionally.
- Unauthorized response requests are denied even if the UI sends them.
- Expired, replayed, unsigned and wrong-endpoint commands are rejected.
- Error payloads contain no secrets, stack traces or sensitive cross-tenant detail.