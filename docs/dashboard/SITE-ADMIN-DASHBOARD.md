# Ariba EPP — Unit / Site Administrator Dashboard

**Role:** Unit / Site Administrator
**Scope:** One assigned operating Site (`site_id`) inside an authorized Company
**Status:** Proposed; Site-role endpoint/enrollment permissions are To confirm in the canonical RBAC matrix. The current matrix grants Site Admin site-level view metadata but has no explicit Site Admin endpoint/token/health permission columns.

## Purpose and boundary

Support local endpoint onboarding and agent-health monitoring for an authorized factory, office, branch or depot. A Site Admin cannot browse other sites/companies, manage company-wide users/policies, or perform security response unless a separate approved permission explicitly grants it.

## Navigation proposal

```text
Site Dashboard
├── Local Endpoints (assigned site only)
├── Agent Health (assigned site only)
├── Enrollment Status (token creation/use only if site-scoped permission is approved)
├── Site Inventory (minimum approved endpoint metadata)
└── Site Audit (read-only scoped activity where permitted)
```

No vulnerability, policy-authoring, threat-intelligence or response controls are included by default. Routes outside the site scope are hidden, and the server independently rejects unauthorized IDs.

## KPI widget contract

| Widget ID / status | Widget | Source / scope | Definition, freshness and guardrail |
|---|---|---|---|
| SADM-01 · Foundation v0.1, permission-gated | Site Endpoint Count | Endpoint inventory; assigned site | Hidden until `endpoint:read` is granted to Site Admin at SITE scope; source `as_of` shown |
| SADM-02 · Foundation v0.1, permission-gated | Agent Health | Heartbeat/health; assigned site | Hidden until `health:view` is granted at SITE scope; thresholds require approval |
| SADM-03 · To confirm | Enrollment Progress | Enrollment and endpoint records; assigned site | Hidden until a site-scoped token/endpoint permission is approved; never reveal token value |
| SADM-04 · Proposed | Site Distribution/Unassigned Assets | Endpoint-to-site relation | Highlight unassigned only within authorized Company scope; no location inferred from IP |

## Operations and privacy safeguards

- Enrollment workflows require a one-time, short-lived, site-scoped token only if that scope and `token:generate` are explicitly granted to Site Admin; otherwise controls remain unavailable. No multi-use default.
- Installer download is available only for signed/released packages supporting that OS.
- Site Admin cannot elevate user roles, assign policy, manage global integrations or retire endpoints unless the granular matrix explicitly grants it. Endpoint listing/health also requires explicit `endpoint:read`/`health:view` SITE scope.
- Every enrollment/revocation/update action is confirmed, reasoned and audited.
- Keep user identity, command-line, file paths and security-event evidence out of site summary views unless approved and necessary.
