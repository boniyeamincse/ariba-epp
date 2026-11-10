# ADR-018 — Hierarchical RBAC grants

- **Status:** To confirm; proposed grants are not effective
- **Owner:** Product owner / security owner (to assign)
- **Related:** `../PRODUCT-REQUIREMENTS.md` (role families), `../security/PERMISSION-MATRIX.md`, `../api/API-CONTRACT.md`, `../database/DATA-MODEL.md`, `../dashboard/README.md`, ADR-009

## 1. Context

The owner-confirmed hierarchy is Platform → Parent Group (`organization_id`) → Company (`company_id`) → Site (`site_id`) → Endpoint. Current role families are Platform Super Admin, Group Administrator, Group SOC Lead, Company Administrator, Company SOC Analyst, Unit/Site Administrator, Security Administrator, Security Auditor and Executive Viewer.

The permission matrix has been rewritten for these roles, but its grants are proposals. Group SOC evidence access, Site Admin endpoint/token/health operations, security-policy author/sign separation, role stacking and response permissions affect customer data and endpoint safety.

## 2. Decision required

Review and approve or revise `docs/security/PERMISSION-MATRIX.md`. Specify, at minimum:

- Permission grants, action denials and maximum scope for each role.
- Whether Group-level roles see only aggregates or detailed Company records.
- Site-level grants and how users receive one/multiple Company/Site assignments.
- Role stacking, temporary access, role delegation, self-elevation denial and periodic access review.
- Policy author/reviewer/signer separation and signing-key custody.
- Whether to add a distinct Company Incident Responder role or grant action-specific elevation to Company SOC Analysts.
- On-demand scan permissions versus disruptive response permissions.
- Auditor and Viewer data fields, exports and audit visibility.

Until approved, all `Proposed`/`To confirm` grants are denied. UI visibility does not grant API access.

## 3. Security acceptance evidence

- Positive and negative permission tests for every role/scope/action.
- Cross-Group, cross-Company and out-of-scope Site read/write/search/export denial tests.
- Tests for role assignment, revocation, session refresh and attempted self-elevation.
- Audit evidence for grants, denials, role changes, exports and privileged actions.
- Dashboard/API permission mappings reviewed together; no stale Org Admin/SOC L1/SOC L2 mapping remains.

## 4. Consequences

The RBAC matrix is review-ready documentation, not an implementation authorization. Identity/API/UI/database implementation tasks that depend on exact grants remain blocked until this ADR is approved.