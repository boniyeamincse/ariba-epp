# Ariba EPP — Company Administrator Dashboard

**Role:** Company Administrator
**Scope:** One subsidiary Company (`company_id`) and its sites/endpoints
**Status:** Proposed; exact Company-role permission codes require reconciliation in the canonical RBAC matrix.

## Purpose and boundary

Manage a single Company’s sites, authorized users, endpoint inventory, enrollment and operational health. Company Admin cannot read another Company’s endpoint data or administer the parent Group/Platform unless separately assigned an explicitly approved role.

## Navigation proposal

```text
Company Dashboard
├── Endpoints (company-scoped inventory)
├── Agent Health
├── Sites & Units (own Company)
├── Enrollment (scoped one-time tokens; permission-gated)
├── Users & Roles (hidden until Company permission is defined)
├── Policy Status (read/assignment controls only as authorized)
└── Audit (Company-scoped)
```

Vulnerability Management, application control, firewall and patch posture are Release 2 / P1 and remain hidden until implemented. Group-wide navigation is not shown.

## KPI widget contract

| Widget ID / status | Widget | Source / scope | Definition, freshness and guardrail |
|---|---|---|---|
| CAD-01 · Foundation v0.1 | Endpoint Coverage | Endpoint lifecycle + heartbeat; Company | Counts use approved lifecycle definitions and show `as_of`; no hard-coded demo metrics |
| CAD-02 · Foundation v0.1 when reported | Site Coverage | Endpoint-to-site assignment; Company | Show unassigned separately; no user location inference |
| CAD-03 · Foundation v0.1 when reported | Agent Health | Agent heartbeat; Company/site | Missing signal is `Unknown`; thresholds and stale interval To confirm |
| CAD-04 · Foundation v0.1, permission-gated | Enrollment Token Status | Token metadata; permitted Company scope | Never show token value after creation; show status, expiry, scope and audit reference |
| CAD-05 · Release 1 | Policy Acknowledgement | Policy assignment/ack records | Only when policy distribution exists; show version, denominator and last report time |
| CAD-06 · Release 2 / P1 | Vulnerability Summary | Inventory/CVE service; Company | Hidden until implemented; source, coverage and scan time required |

## Actions and safeguards

- Site/user administration requires `site:manage`, `user:manage` and `user:assign_role` in Company scope; these tenancy permissions appear in the canonical matrix.
- Enrollment token generation/revocation and endpoint lifecycle operations remain hidden until the legacy endpoint permission rows are explicitly mapped to Company Admin. Tokens must be short-lived, scoped and single-use per FR-ID-04.
- Endpoint retirement, certificate revocation and exports show target/scope, confirmation, reason and audit result only when permissioned.
- No security-policy authoring/signing or disruptive response unless a distinct approved permission is granted.
- Company ID is derived from authenticated context; the UI cannot switch to an unauthorized Company by editing a URL or filter.
