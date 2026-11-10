# Ariba EPP — Security Administrator Dashboard

**Role:** Security Administrator  
**Security Boundary:** Organization Security Configuration & Policy Enforcement  
**Specification File:** `docs/dashboard/SECURITY-ADMIN-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Access is limited to explicitly assigned organization/sites. The owner-confirmed feature matrix in `PRODUCT-REQUIREMENTS.md` controls release availability.

---

## 1. Role Purpose & Authority
The Security Administrator reviews security posture and, when available, drafts or submits policies/rules for the approved signing and rollout workflow. This role does not bypass approval, directly alter the endpoint, or execute response actions. Release 2 firewall, ransomware, application control and MISP workflows remain planned.

---

## 2. Sidebar Navigation Visibility

```text
ARIBA EPP
├── Dashboard (Security Posture & Enforcement Summary)
├── Protection
│   ├── Malware Scanning (Release 1: implemented scanner integrations; request scan only with `response:scan`)
│   ├── Ransomware Protection (Release 2 / planned)
│   ├── Host Firewall (Release 2 / planned)
│   └── Application Control (Release 2 / planned)
├── Threat Intelligence
│   ├── IOC Management (Release 2 / P1; import/curate only after source and licensing approval)
│   ├── YARA Rules (Release 1 author/compile/test via `yara:manage`; deployment not granted by this permission)
│   └── Threat Feeds (MISP; Release 2 / planned)
├── Policy Management (Release 1; draft, review, signed rollout status)
└── Reports (Release 3 / P2 candidate; hide until report definitions and permissions are approved; no compliance claim)
```

---

## 3. Dashboard KPI Widgets

| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| SAD-01 · Release 1 | Policy Distribution | Assignment/acknowledgement records; assigned organization/sites | Applied vs pending/rejected by policy version; denominator and server `as_of` visible |
| SAD-02 · Per-integration release | Protection Integration Status | Health reports from enabled integrations only | `Not enabled`, `Unknown`, `Degraded` and `Healthy` defined per integration; do not imply unimplemented engines |
| SAD-03 · Release 1 | Scanner / Rule Status | Release 1 scanner/rule metadata and test status | Show rule version, validation result and last update; detection efficacy/“hit rate” only with validated denominator |
| SAD-04 · Release 2 / P1 | Threat Intelligence Sync | MISP integration metadata; assigned organization | Hide until approved integration exists; show provenance and last successful sync, not only IOC count |
| SAD-05 · Release 1 | Policy Rejection / Drift | Agent policy acknowledgements | Count by version/status with assigned-scope filter and last report time |
| SAD-06 · Release 2 / P1 | Firewall / App Events | Release 2 event source | Hidden until feature exists; never imply enforcement from monitoring-only signals |

---

## 4. Key Workflows & Actions

1. **Policy Authoring & Distribution:**
   - Create a draft containing only fields approved for the active release (Release 1: supported scanning/monitoring settings). Release 2 ransomware/firewall fields must remain unavailable until their capabilities and policy schemas are approved and implemented.
   - Use `policy:sign` only through the approved signing service and policy workflow; the UI must never expose private signing keys. Key custody/rotation and whether an author may sign their own draft remain To confirm, so keep the sign control gated until those rules are approved.
   - Stage rollout only after the policy signing, approval and target-group contracts are approved; show acknowledgement before any promotion.
   - Rollback is a new, authorized signed policy action; no automatic or silent rollback.
2. **YARA & Threat Intelligence Curation:**
   - Author/compile and syntax-test rules only with `yara:manage`; rule deployment/publishing controls remain hidden until an explicit permission and release workflow are approved.
   - Request an on-demand scan only through the supported Release 1 workflow with `response:scan`; confirm authorized scope and audit the request/result.
   - MISP feed management is Release 2 / P1 and remains hidden until its source, licensing, confidence model and integration are approved.
3. **Firewall & Device Access Rules:**
   - Host firewall rule authoring and USB/removable-media control are not available in Release 1; do not show active controls in this dashboard.

## 5. Security and audit constraints

- Every policy/rule change records actor, tenant/site scope, reason, version, reviewer/signature reference and rollout outcome.
- Security Administrator cannot assign themselves broader scope, approve their own changes unless a future owner-approved policy explicitly permits it, or execute endpoint response.
- Any destructive/endpoint-changing operation remains denied by default and is governed by ADR-014; this dashboard provides no response button.
- Rule and feed changes require provenance, validation/test evidence, staged rollout and rollback procedure before publication.
