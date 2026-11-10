# Ariba EPP — Group SOC Lead Dashboard

**Role:** Group SOC Lead
**Scope:** Parent Group rollups; company-level evidence only with an explicit permission
**Status:** Proposed; Group SOC granular permissions are not yet represented in the detailed RBAC matrix.

## Purpose and boundary

Give an authorized Group SOC Lead visibility into security posture and alert trends across subsidiary companies. Group-level aggregate visibility is not blanket permission to read raw company telemetry. Per-company alert/evidence drill-down is disabled until the canonical permission matrix defines and tests Group SOC access.

## Navigation proposal

```text
Group SOC Overview (aggregate)
├── Alerts (group rollup; details permission-gated)
├── Company Security Posture (aggregates; scoped drill-down)
├── Investigation (only approved evidence fields and authorized companies)
├── Threat Intelligence (enabled sources only; Release 2 integration candidate)
├── Threat Hunting (Release 3 / P2; planned)
└── Audit (hidden until an explicit Group-scoped `audit:read` permission is approved)
```

## KPI widget contract

| Widget ID / status | Widget | Source / scope | Definition, freshness and guardrail |
|---|---|---|---|
| GSD-01 · Release 1 when implemented | Alert Volume | Validated alert records; Group aggregate | Count by approved severity/status; no company names/details unless authorized |
| GSD-02 · Release 1 | Company Alert Rollup | Alert records grouped by Company | Show `as_of`, denominator and coverage gaps; link to details only with company permission |
| GSD-03 · To confirm (ADR-013) | Triage Trend | Alert state/audit timestamps; Group aggregate | Hide until alert states, transitions, formula and time window are approved |
| GSD-04 · Release 1 | Agent Reporting Posture | Health aggregates | Distinguish stale/offline/unknown; thresholds remain To confirm |
| GSD-05 · Release 2 / P1 | IOC Enrichment | Approved MISP/IOC source and provenance | Hide until integration and licensing are approved; show source/confidence/expiry |
| GSD-06 · Release 3 / P2 | Threat Hunting | Historical event store | Planned; no arbitrary cross-company query before Group SOC permissions, privacy rules and audit are approved |
## Workflow and safety

- Review Group-level alert aggregates and escalate only using approved tenant/company routing.
- Company alert assignment, notes and evidence access require explicit Group SOC permissions; current detailed RBAC has no corresponding columns.
- No destructive response control is available. Scan requests follow `response:scan` only after a new Group SOC permission mapping and release approval.
- High-severity indicators are conveyed by text/icon as well as color; no audio/flash-only alerting.
- Search, exports and saved views must preserve parent Group scope and avoid revealing other groups.
