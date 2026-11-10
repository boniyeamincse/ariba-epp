# ADR-014 — Endpoint response authority and approval model

- **Status:** To confirm; disruptive response remains unavailable
- **Owner:** Product owner / security owner / incident-response owner (to assign)
- **Related:** `../PRODUCT-REQUIREMENTS.md` (FR-RSP-01…05), `../security/PERMISSION-MATRIX.md`, `../security/THREAT-MODEL.md`, `../agent/AGENT-SPECIFICATION.md`, `../api/API-CONTRACT.md`, `../dashboard/SOC-ANALYST-L2-DASHBOARD.md`

## 1. Context

The owner-confirmed Release 1 feature matrix includes malware scanning (on-demand and scheduled). That capability does not automatically approve a remote scan-request API or grant a role permission to invoke it. A scan request (`response:scan`) is a separate, non-disruptive operation with resource and target impact; its target types, supported OS, role/scope mapping, limits, confirmation, result reporting and audit behavior remain to be approved. Disruptive actions (process termination, quarantine/restore and isolation/release) are separate Release 2 candidates and remain unavailable.

## 2. Decision required

Approve each action separately; do not approve an implicit bundle:

| Action | Release candidate | Impact | Interim state |
|---|---|---|---|
| On-demand / scheduled scan | Release 1 feature category confirmed; invocation design gated | Resource usage; no intended endpoint state change | Scan capability may be implemented only to approved scope; remote request/scheduling controls stay hidden until targets, limits, `response:scan` mapping and audit are approved |
| Kill process | Release 2 candidate | Terminates endpoint process | Disabled |
| Quarantine file | Release 2 candidate | Moves/blocks access to endpoint file | Disabled |
| Restore file | Release 2 candidate | Restores quarantined content | Disabled |
| Isolate endpoint | Release 2 candidate | Changes network access and can disrupt operations | Disabled |
| Release isolated endpoint | Release 2 candidate | Restores network access | Disabled |
| Forensic collection | Future candidate | May collect sensitive data / large artifacts | Disabled |
| Firewall or application-control change | Release 2 candidate | Changes endpoint enforcement | Disabled |

For every enabled action, decide target freshness, supported OS, requester/approver roles, separation-of-duties rule (including self-approval), confirmation text, reason, idempotency, expiry, command signing, retry/cancel behavior, audit fields, failure states, recovery/rollback and lab/release tests.

The current role catalog has Company SOC Analyst but no distinct Company Incident Responder. Decide whether response is a separately elevated permission for selected SOC Analysts or requires a new role; do not invent a role in implementation.

## 3. Mandatory guardrails

- Deny by default; no AI score or recommendation executes a destructive action.
- Server validates principal, effective Group/Company/Site scope, exact target and current endpoint identity before issuing a command.
- Commands are action-allowlisted, signed/authenticated, endpoint-bound, replay-resistant, single-use where practical, and time-limited.
- Endpoint revalidates target immediately before action and returns truthful accepted/running/succeeded/failed/rejected/expired state.
- Preserve a restricted management/recovery channel during isolation and test the release path before enablement.
- Audit request, approval/denial, command issue, endpoint result and recovery, without logging secrets or excess personal data.
- Test only on owned/authorized isolated lab endpoints; never disable existing AV/host protection.

## 4. Interim behavior

Until this ADR and the selected action's design are approved, disruptive response endpoints/actions and dashboard controls are specification-only and unavailable. The Release 1 scanning feature category remains in scope, but remote scan requests/scheduling may be exposed only after target limitations, role mapping, resource limits, audit behavior and tests are explicitly approved.

## 5. Acceptance after approval

- Owner/security approval recorded for one named action, not the action catalog as a whole.
- Threat-model review and permission-matrix mapping complete.
- Expiry, idempotency, target validation, audit, failure reporting and recovery tests pass in an isolated lab.
- UI clearly identifies impact, target, scope, approver requirement, pending status and endpoint-confirmed outcome.
