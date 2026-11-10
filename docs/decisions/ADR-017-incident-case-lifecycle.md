# ADR-017 — Incident-case lifecycle

- **Status:** To confirm; no incident-case write workflow is approved
- **Owner:** Product owner / SOC owner (to assign)
- **Related:** `../PRODUCT-REQUIREMENTS.md` (alert and incident scope), `../api/API-CONTRACT.md`, `../database/DATA-MODEL.md`, `../dashboard/README.md`, ADR-013 (alert state machine)

## 1. Context

Release 1 includes alert management and investigation. Some documents also mention a separate `Incident` entity grouping multiple alerts, with assignment, priority, timeline, closure and SLA metrics. The feature release matrix does not explicitly define a separate incident-case lifecycle. ADR-013 governs alert states only and must not be assumed to approve an incident entity.

## 2. Decision required

Choose one before implementing incident APIs or mutation controls:

- **Option A — Alert investigation only in Release 1:** investigate and disposition alerts; defer standalone incident cases to a later release.
- **Option B — Separate incident cases in Release 1:** approve the entity, alert-to-case linking, states/transitions, ownership, permissions, audit requirements and acceptance tests in a follow-up decision.

Until the owner records the choice, show no incident-case creation/closure controls. Read-only alert investigation may proceed only within approved permissions and telemetry.

## 3. Questions to resolve

- Are multiple alerts grouped into a case, and can an alert belong to more than one case?
- What are the case states, transitions, priority/severity and required closure fields?
- Which Company/Group SOC roles can create, assign, update, close or reopen cases?
- Is Group SOC cross-Company case visibility aggregate-only or detailed? How is Company consent/scope represented?
- Are SLA timers/MTTA/MTTR required? Define formula, timezone, pause states and denominator before showing metrics.
- What evidence, notes, exports and retention rules apply?

## 4. Consequences

- APIs, schemas, menus, KPI widgets and tasks must label case lifecycle `To confirm` until approval.
- ADR-013 remains authoritative only for alert states; it does not settle case states.
- Implementation must update API, database, RBAC, test and dashboard contracts together after approval.