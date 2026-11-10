# Ariba EPP — Group Administrator Dashboard

**Role:** Group Administrator
**Scope:** One parent Group (`organization_id`) and its subsidiary Companies (`company_id`)
**Status:** Proposed view; granular permission mapping is blocked until `docs/security/PERMISSION-MATRIX.md` is reconciled with the current role hierarchy.

## Purpose and boundary

Provide a parent-group view of registered companies, group-level configuration and aggregate endpoint posture. Group-level access does not imply routine access to raw endpoint telemetry, files, command lines or forensic evidence. Company drill-down must apply the user's explicit company/site scope; never trust a client-selected `organization_id`, `company_id` or `site_id`.

## Navigation proposal

```text
Group Dashboard
├── Group Overview (aggregate only)
├── Companies (list/onboarding only when explicit permission exists)
├── Sites (company-scoped; visibility follows permission)
├── Group Users & Roles (hidden until role-management permissions are mapped)
├── Endpoint Posture (aggregate; company drill-down permission-gated)
├── Policies (read status only unless separately granted)
└── Group Audit (scoped, read-only by default)
```

## KPI widget contract

| Widget ID / status | Widget | Source / scope | Definition, freshness and guardrail |
|---|---|---|---|
| GAD-01 · Proposed | Companies | Company registry; parent Group | Count by approved lifecycle state; show `as_of`; no other parent Group data |
| GAD-02 · Foundation/R1 when instrumented | Fleet Aggregate | Endpoint service aggregate; parent Group | Count only; drill-down requires company-scoped permission; unknown/stale devices separated |
| GAD-03 · Foundation v0.1 when available | Site Coverage | Company/site/endpoint relation | Show missing/unassigned site separately; never infer user location |
| GAD-04 · Proposed | Agent Health Rollup | Authorized heartbeat/health aggregates | Thresholds and freshness are To confirm; missing telemetry is `Unknown`, not healthy |
| GAD-05 · Release 1 | Policy Acknowledgement Rollup | Policy assignment records | Only after policy service is implemented; show denominator, version and last report time |
| GAD-06 · To confirm (`audit:read` Group scope) | Group Audit Highlights | Audit records within assigned Group scope | Show only if Group Admin receives `audit:read`; actor/action/time/outcome, sensitive values masked |

## Actions and permissions

- Company create/update, Group user/role management and site management require `company:create`, `company:manage`, `user:manage`, `user:assign_role` and `site:manage` in Group scope; these codes exist in the tenancy section. Enforce them server-side and test Group/Company isolation.
- Endpoint, policy, alert and telemetry permissions are not mapped to `group_admin` in the detailed matrix; keep those controls unavailable pending an owner-approved mapping.
- Group admin must not assign Platform Super Admin or elevate their own scope.
- Company suspension is not available until an explicit permission, confirmation, effect summary and recovery/appeal workflow are approved.
- Every change records actor, parent Group, affected Company, reason and outcome.

## Enterprise safeguards

- Aggregate dashboards do not expose raw cross-company events.
- Company-level drill-down rechecks company/site access at API and data layers.
- Export, saved filters and notifications preserve Group scope and are audited.
- Loading, stale, offline, no-data, not-enabled, partial/degraded and permission-denied states are distinct.
