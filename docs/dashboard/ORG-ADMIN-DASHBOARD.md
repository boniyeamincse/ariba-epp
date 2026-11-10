# Ariba EPP — Organization Administrator Dashboard

> **Legacy role taxonomy:** current product roles are Platform Super Admin, Group Admin, Company Admin, Company SOC Analyst and Unit/Site Admin. Use [Company Administrator Dashboard](COMPANY-ADMIN-DASHBOARD.md) for the current Company role. This file is retained as historical source only; do not implement its role grants.

**Role:** Organization Administrator  
**Security Boundary:** Single Tenant (`organization_id` strictly enforced)  
**Specification File:** `docs/dashboard/ORG-ADMIN-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Foundation v0.1 is limited to verified organization/site, endpoint enrollment, heartbeat and basic inventory views. No sample values below are live data.

---

## 1. Role Purpose & Authority
The Organization Administrator manages their company's specific deployment: enrolling endpoints, organizing sites and departments, configuring groups, managing internal users and access roles, and tracking overall deployment coverage.

---

## 2. Sidebar Navigation Visibility

```text
ARIBA EPP (Org Mode: {Organization Name})
├── Dashboard (Organization Posture & Fleet Overview)
├── Endpoint Management
│   ├── All Endpoints
│   ├── Windows Endpoints
│   ├── Linux Endpoints
│   ├── Endpoint Groups (only after group feature is implemented)
│   └── Agent Health
├── Software Inventory (Release 2 / P1; planned)
├── Vulnerability Management (Release 2 / P1; planned)
│   └── Patch Management / patch compliance (planned; no patch execution in current scope)
├── Policy Management (Assigned Policies & Deployment Status)
├── Reports (Executive, Asset & Compliance Reports)
└── Administration
    ├── Users & Roles (Org Users & Role Assignments)
    ├── Sites & Locations (Branch Offices, Datacenters)
    ├── Enrollment Tokens (Generate & Revoke Agent Tokens)
    └── Audit Logs (Tenant-scoped Activity History)
```

---

## 3. Dashboard KPI Widgets

| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| OAD-01 · Foundation v0.1 | Endpoint Coverage | Endpoint lifecycle + heartbeat; current organization only | Enrolled/active/offline counts use owner-approved state definitions; show server `as_of` time; no fabricated sample values |
| OAD-02 · Foundation v0.1 when reported | Agent Health | Agent heartbeat/health report; current organization/site scope | Health dimensions and thresholds To confirm; missing telemetry is `Unknown`, not healthy |
| OAD-03 · Foundation v0.1 | Site Distribution | Endpoint-to-site mapping; current organization | Count by authorized site; unassigned site shown separately; no external customer names |
| OAD-04 · Foundation v0.1, permission-gated | Enrollment Token Status | Token metadata; permitted administrator scope | Show token ID suffix, status, scope and expiry only; never reveal token value after creation |
| OAD-05 · Release 1 | Policy Acknowledgement | Policy assignment/acknowledgement records | Show applied/rejected/pending only when policy feature is deployed; denominator and freshness visible |
| OAD-06 · Release 2 / P1 | Vulnerability Summary | Inventory/CVE service; current organization | Hidden until implemented; CVE source, scan time and coverage denominator required |

---

## 4. Key Workflows & Actions

1. **Agent Enrollment & Deployment:**
   - Generate a scoped, short-lived, single-use enrollment token per FR-ID-04. Exact expiry is To confirm; do not offer multi-use tokens unless the approved requirement changes.
   - Installer download is shown only when signed packages for the target OS have been released and verified; otherwise show setup status, not a download control.
   - Revoke compromised enrollment tokens or decommission retired endpoints.
2. **Fleet Organization:**
   - Create and organize static or dynamic tag-based Endpoint Groups (e.g., `Servers-Linux`, `Finance-Workstations`).
   - Assign endpoints to Sites and physical locations.
3. **Internal User Management:**
   - Provision users and assign only owner-approved tenant-scoped roles; never grant Platform Administrator from this screen.
   - Privileged MFA is required before external pilot (FR-ID-03); provider/configuration remains To confirm.

## 5. Tenant and privacy safeguards

- Organization scope is derived server-side; organization/site filters cannot expand the signed-in user's access.
- Site-scoped roles see only assigned sites; exact role-to-site matrix is To confirm.
- Do not show command lines, file contents, raw forensic evidence or user data unless separately approved and permissioned.
- Token create/revoke, user/role changes and endpoint retirement are audited; exports are permission checked and audited.
- Planned Release 2 routes are hidden/disabled until the corresponding capability is verified.
