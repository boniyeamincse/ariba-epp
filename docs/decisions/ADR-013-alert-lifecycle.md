# ADR-013 — Alert lifecycle and triage transitions

- **Status:** To confirm; mutation workflows are blocked
- **Owner:** Product owner / SOC workflow owner (to assign)
- **Related:** `../PRODUCT-REQUIREMENTS.md` (FR-DET-03), `../api/API-CONTRACT.md`, `../database/DATA-MODEL.md`, `../dashboard/README.md`, ADR-017 (separate incident cases), ADR-018 (role grants)

## 1. Context

Release 1 includes alert management and investigation. Dashboard and API drafts mention assignment, triage notes, false-positive disposition and resolution, but exact alert states, transitions, permissions and metrics have not been approved. A separate Incident Case may or may not be included; ADR-017 decides that independently.

## 2. Decision required

Approve an alert lifecycle and transition table. Candidate states for review (not approved):

`new → acknowledged/investigating → resolved`

A false-positive/benign disposition may be a terminal status or a disposition field. Decide which, and whether reopen/reassign/escalate transitions exist. Do not equate alert resolution with a separate incident-case closure.

## 3. Decisions to record

- Canonical state names, permitted transitions and terminal states.
- Whether assignment is a separate field or state; who may claim/assign.
- Whether false-positive, benign, duplicate and informational outcomes are states or dispositions.
- Required actor, timestamp, reason/comment and audit fields for each transition.
- Severity taxonomy, priority mapping and any SLA/MTTA calculation definition.
- Role/scope grants from ADR-018, including Company SOC Analyst and Group SOC Lead behavior.
- API conflict/invalid-transition response and concurrent-update handling.
- Notification/Ariba SOC event behavior and idempotency.

## 4. Safe interim behavior

Until this ADR is approved, alert data may be shown read-only within approved scope. Hide/disable alert assignment, state change, false-positive marking, resolution and SLA metrics. Do not implement alert PATCH behavior beyond non-mutating reads.

## 5. Acceptance after approval

- State transition table is documented in Product Requirements/API/DB contract and matches dashboard controls.
- Positive and negative permission tests cover every transition and hierarchy scope.
- Invalid, repeated and concurrent transitions fail deterministically and are audited where appropriate.
- Metrics use the approved state timestamps, timezone, window and denominator.
