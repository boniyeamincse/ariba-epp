# Ariba EPP — Platform Administrator Dashboard

**Role:** Platform Administrator (Super Admin)  
**Security Boundary:** Global Multi-Tenant (System-wide)  
**Specification File:** `docs/dashboard/PLATFORM-ADMIN-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Foundation v0.1 shows only verified platform setup and aggregate endpoint health. All figures below are widget ideas, not live data or approved SLOs.

---

## 1. Role Purpose & Authority
The Platform Administrator manages the overall multi-organization platform, oversees server-side cluster health, monitors tenant quotas, manages global integration endpoints, and provisions customer organizations.

> [!CAUTION]
> The default dashboard shows platform operations and aggregate organization/fleet metadata. Endpoint inventory metadata is available only through the canonical `endpoint:read` permission and is audited. Platform Administrators do not browse raw customer telemetry, process/file evidence or host files. Any exceptional support access requires a separate approved, customer-consented, time-bounded and audited break-glass procedure; no such UI is in this draft.

---

## 2. Sidebar Navigation Visibility

```text
ARIBA EPP (Platform Mode)
├── Dashboard (Global Fleet & System Health)
├── Platform Health
│   ├── Organization / Fleet Aggregates (default view)
│   ├── Endpoint Metadata (only with endpoint:read; audit access)
│   └── Service Health (only deployed and authorized operational signals)
├── Integrations (configuration visibility; management actions separately permissioned)
└── Administration
    ├── Organizations (Tenant Onboarding, Quotas, Status)
    ├── Platform Settings (TLS, Rate Limits, Ingestion Buffers)
    └── Audit Logs (Platform-wide Administrative Audit Trail)
```

---

## 3. Dashboard KPI Widgets

| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| PAD-01 · Proposed | Active Organizations | Organization lifecycle records; platform aggregate | Active/suspended counts by approved status definition; source `as_of` displayed; no sample count as live data |
| PAD-02 · Foundation/R1 when instrumented | Enrolled Endpoint Aggregate | Endpoint service aggregate only; endpoint detail requires audited `endpoint:read` | Count by approved lifecycle state; freshness and stale threshold To confirm |
| PAD-03 · Proposed | Ingestion Health | Ingestion/broker/worker operational metrics, if deployed | EPS/lag/latency definitions and SLOs To confirm; label degraded/unknown distinctly |
| PAD-04 · Proposed | Service Health | Platform health checks; infrastructure scope | Show only components actually deployed; no pretend cluster replication/HA metrics |
| PAD-05 · Proposed | OS/Agent Version Distribution | Tenant-safe aggregate endpoint inventory | Aggregate buckets only; minimum cohort/privacy threshold To confirm |
| PAD-06 · Proposed | Administrative Audit Highlights | Platform-scoped audit events | Actor/action/time/outcome; sensitive values masked; source timestamp and pagination |

---

## 4. Key Workflows & Actions

1. **Organization Lifecycle:** Create/suspend an organization and provision its initial admin only with the approved platform permission; each mutation requires reason, confirmation and audit.
2. **Platform Health Monitoring:** Show queue/worker/storage metrics only when those components are deployed and instrumented; do not imply replication or redundancy that has not been built.
3. **Integration Administration:** Secrets are write-only/masked; edits are privileged, audited and environment-scoped. Specific integrations remain release-gated.
4. **Quota/License:** quota policy and commercial licensing are To confirm; do not display compliance or license-enforcement status before a defined contract.

## 5. Access and failure states

- Aggregate cards do not drill into endpoint records by default. Any endpoint-metadata drill-down requires `endpoint:read`, is audit logged and never exposes raw telemetry/files.
- Platform admin audit view is distinct from any future platform-auditor role.
- Service metric unavailable, stale or not configured is shown as `Unknown`/`Degraded`, not green.
- Cross-tenant exports and customer telemetry are not available in this dashboard.
