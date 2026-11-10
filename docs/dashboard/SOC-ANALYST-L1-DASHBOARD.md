# Ariba EPP — SOC Analyst L1 (Triage) Dashboard

> **Legacy role taxonomy:** the current role model uses Company SOC Analyst and Group SOC Lead. Use [Company SOC Analyst Dashboard](COMPANY-SOC-ANALYST-DASHBOARD.md) and [Group SOC Lead Dashboard](GROUP-SOC-LEAD-DASHBOARD.md) for current scope. This file is retained as historical workflow input only; do not treat L1 permissions as current grants.

**Role:** SOC Analyst L1 (Triage & Monitoring)  
**Security Boundary:** Scoped Operational Triage (Observe & Triage)  
**Specification File:** `docs/dashboard/SOC-ANALYST-L1-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Foundation v0.1 has no alert workflow. Release 1 alert triage appears only after the alert contract and authorization are approved.

---

## 1. Role Purpose & Authority
The Tier 1 SOC Analyst reviews assigned-scope detections and records triage context. Alert assignment/false-positive marking wait for ADR-013 and the canonical RBAC permission; incident creation also depends on approval of a separate incident-case scope/lifecycle.

> [!NOTE]
> SOC Analyst L1 has read-only access to endpoint context and evidence explicitly authorized for the assigned tenant/site. They cannot browse endpoint files or execute any disruptive response action. Claim/assign/comment/disposition are state-changing operations and remain disabled until ADR-013 defines transitions and the role permission matrix is approved.

---

## 2. Sidebar Navigation Visibility

```text
ARIBA EPP
├── Dashboard (Real-time Threat Triage & Monitoring)
├── Detection & Response
│   ├── Threat Detection (Release 1; only validated, authorized event summaries)
│   ├── Alerts (Release 1; read-only until ADR-013 and permissions are approved)
│   └── Incident Cases (standalone case lifecycle To confirm; use alert investigation until approved)
├── Endpoint Management (Read-Only)
│   ├── All Endpoints (Basic inventory & connectivity context)
│   └── Agent Health (Inspect reporting delays or offline states)
└── Threat Intelligence (planned integration; show only if an approved source is enabled)
```

---

## 3. Dashboard KPI Widgets

| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| L1D-01 · Release 1 | Recent Detection Events | Validated detection/alert API; assigned organization/sites | Paginated event summaries with event and ingestion times; no unsupported “live stream” guarantee |
| L1D-02 · Release 1 + ADR-013 | Alert Counts | Alert records; assigned scope | Count by approved severity/status; severity taxonomy is To confirm; `as_of` and time window visible |
| L1D-03 · Release 2 / P1 if mapping is enabled | ATT&CK Coverage | Validated rule mappings; assigned scope | Do not show empty or inferred technique mapping as coverage; include mapping provenance |
| L1D-04 · To confirm (ADR-013) | Triage Time | Audit/status transition timestamps; assigned scope | Formula, included states, timezone and reporting window require ADR-013; hide until defined |
| L1D-05 · Release 1 + ADR-013 | Unassigned Alerts | Alert assignment data; assigned scope | Read-only until assignment permission/state transition is approved |
| L1D-06 · Release 1 | Top Endpoints | Alert aggregate; assigned scope | Prefer endpoint IDs/hostnames only as role permits; no user ranking or cross-tenant aggregation |

---

## 4. Key Workflows & Actions

1. **Alert Assessment:**
   - Review unassigned alerts read-only until alert assignment is enabled by ADR-013 and `alert:triage` authorization.
   - Inspect triggered rule ID, detection timestamp, host context, and process metadata only within assigned tenant/site scope and approved telemetry fields.
2. **Context Enrichment & Verification:**
   - Review a hash/evidence reference only if collected and approved; external reputation lookup is unavailable unless an approved integration is configured, and its source/confidence must be shown.
   - Check endpoint history for repeated suspicious activity.
   - An on-demand scan request may be offered only when Release 1 scanner support exists, with `response:scan`, authorized scope, user confirmation and audit; it is not a disruptive response action.
3. **Disposition & Escalation:**
   - Proposed dispositions (false positive, informational, escalate) must follow the ADR-013 state machine and permission matrix. Until approved, show details read-only and do not render mutation buttons.
   - Incident creation/assignment is unavailable until a separate incident-case scope/lifecycle is approved; then it requires an audit reason and tenant-scoped `incident:create` permission. Escalation never implies a response action.

## 5. Safety and accessibility

- Severity is conveyed with text/icon as well as color; no audio/flash-only critical notification. User-configurable motion/notification preferences are required.
- Evidence views show data source and event/ingestion timestamps; stale or missing telemetry is explicit.
- Hide command lines, file contents and unrelated personal data; detail access is permission checked and audited.
- All list filters, claims and future dispositions remain restricted to the authenticated tenant/site scope.
