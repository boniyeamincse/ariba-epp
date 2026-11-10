# Ariba EPP — Core Dashboard & Navigation Specification

**Status:** Draft UI contract; owner/security review required  
**Scope:** Navigation, role dashboards, widget data contract and visibility rules  
**Audience:** Product owner, UX, frontend/backend engineers, security and QA  

> This folder specifies intended UI behavior only. It does not authorize implementation of a feature or grant API permissions. Server-side authorization is authoritative. Follow `docs/PRODUCT-REQUIREMENTS.md`, approved ADRs and `docs/IMPLEMENTATION-PLAN.md` if anything conflicts.

## 0. Release and status rules

Use the owner-confirmed feature matrix in `docs/PRODUCT-REQUIREMENTS.md` as the source for release scope. Foundation v0.1 is only the first proof slice; it is not the whole Release 1.

| UI status | Meaning | Required behavior |
|---|---|---|
| `Proposed Foundation v0.1` | Candidate endpoint enrollment, heartbeat, basic inventory/status and console shell; exact login/MFA/admin scope requires owner freeze | Show only data actually implemented; empty/loading/error states required; do not treat the proof scope as approved yet |
| `Release 1 / P0` | Process/file monitoring, malware scanning, detection, alerts and policy management | Show only after the relevant capability and acceptance tests are verified |
| `Release 2 / P1` | Quarantine/guarded response, ransomware, firewall/isolation, vulnerability and application control | Hide or clearly mark planned until approved and implemented; response also requires ADR-014 |
| `Release 3 / P2` | AI assistant and advanced threat hunting | Planned; no autonomous response |
| `To confirm` | A product/security decision is unresolved | Do not imply enabled behavior or invent a default |

Do not label a route “live”, “protected”, “compliant”, “isolated” or “healthy” unless a defined, verified data source supports that state. Roadmap-only routes should be hidden from production navigation; prototype-only routes must be visibly labeled and access-restricted.

---

## 1. Core Navigation Sidebar Structure

The navigation below is a **long-term menu map**, not an enabled-feature list. Each route is shown only in the release where its capability is approved, implemented and verified; otherwise hide it or label it planned in non-production prototypes.

```text
ARIBA EPP
├── Dashboard (role-scoped; actual authorized data only)
├── Endpoint Management
│   ├── All Endpoints (tenant/site scoped)
│   ├── Windows Endpoints
│   ├── Linux Endpoints
│   ├── Endpoint Groups (planned; show only when implemented)
│   └── Agent Health
├── Protection
│   ├── Malware Scanning (Release 1; verified YARA/ClamAV integration scope)
│   ├── Ransomware Protection (Release 2 / planned)
│   ├── Host Firewall (Release 2 / planned)
│   └── Application Control (Release 2 / planned)
├── Detection & Response
│   ├── Threat Detection and Alerts (Release 1; validated event/rule data)
│   ├── Alert Investigation (Release 1; assigned scope)
│   ├── Incident Cases (standalone case lifecycle To confirm; show only if approved)
│   └── Response Actions (Release 2; disabled until ADR-014/action approval)
├── Vulnerability Management (Release 2 / planned)
│   ├── Software Inventory
│   └── Vulnerability Findings (no patch execution in current scope)
├── Threat Intelligence
│   ├── IOC Management (Release 2 integration candidate)
│   ├── YARA Rule Status (Release 1 scan scope only)
│   └── Threat Feeds (Release 2 / planned)
├── Policy Management (Release 1; signed/versioned contract)
├── Reports (planned; no compliance claim before evidence)
├── Integrations (only configured, authorized integrations)
└── Administration
    ├── Users & Roles (own authorized scope)
    ├── Organizations (Platform Administrator only)
    ├── Sites & Locations (own organization)
    ├── Enrollment Tokens (one-time, scoped, short-lived)
    ├── Audit Logs (assigned tenant/platform audit scope)
    └── Platform Settings (privileged and audited)
```

---

## 2. Role-Based Dashboard Files Index

The current product model uses Platform → Parent Group → Company → Site → Endpoint scope. The role list below follows the owner-updated product requirements. The proposed hierarchy-aware `PERMISSION-MATRIX.md` is the one place for grants, but remains unapproved under ADR-018; hide or disable role-specific mutation controls until owner/security review approves the relevant grants.

| Role | Specification File | Primary Scope & Responsibilities |
|---|---|---|
| **Platform Super Admin** | [`PLATFORM-ADMIN-DASHBOARD.md`](PLATFORM-ADMIN-DASHBOARD.md) | Platform operations and tenant onboarding; no routine customer telemetry/file access. |
| **Group Administrator** | [`GROUP-ADMIN-DASHBOARD.md`](GROUP-ADMIN-DASHBOARD.md) | Parent-group companies, users and aggregate security posture. |
| **Group SOC Lead** | [`GROUP-SOC-LEAD-DASHBOARD.md`](GROUP-SOC-LEAD-DASHBOARD.md) | Parent-group alert oversight and cross-company rollups; company evidence remains permission-scoped. |
| **Company Administrator** | [`COMPANY-ADMIN-DASHBOARD.md`](COMPANY-ADMIN-DASHBOARD.md) | Own company users, sites, endpoints and enrollment. |
| **Company SOC Analyst** | [`COMPANY-SOC-ANALYST-DASHBOARD.md`](COMPANY-SOC-ANALYST-DASHBOARD.md) | Own-company alert triage and investigation; actions remain separately gated. |
| **Unit / Site Administrator** | [`SITE-ADMIN-DASHBOARD.md`](SITE-ADMIN-DASHBOARD.md) | Local endpoint enrollment and health within one assigned site. |
| **Security Administrator** | [`SECURITY-ADMIN-DASHBOARD.md`](SECURITY-ADMIN-DASHBOARD.md) | Assigned Group/Company policy and rule workflows. |
| **Security Auditor** | [`AUDITOR-VIEWER-DASHBOARD.md`](AUDITOR-VIEWER-DASHBOARD.md) | Read-only audit within explicitly assigned Group/Company scope. |
| **Executive Viewer** | [`AUDITOR-VIEWER-DASHBOARD.md`](AUDITOR-VIEWER-DASHBOARD.md) | Aggregate posture within assigned Group/Company scope. |

### Legacy role pages

`ORG-ADMIN-DASHBOARD.md`, `SOC-ANALYST-L1-DASHBOARD.md` and `SOC-ANALYST-L2-DASHBOARD.md` use the previous role taxonomy. They are retained as source/history only, not as the active role/permission contract; use the Group/Company pages above.

---

## 3. UI Design & Security Principles

1. **Server-Side Enforcement:** The sidebar navigation hides unauthorized modules, but all API routes enforce server-side RBAC and tenant boundaries. Client-side menu hiding is purely for ergonomics.
2. **State Transparency:** Every widget distinguishes Live, Stale, Offline, No Data and degraded states using owner-approved freshness thresholds; this draft sets no stale threshold.
3. **Illustrative Data in Staging:** Sample counts and graphs in documentation/mockups are strictly labeled as illustrative; synthetic data only until real lab telemetry is ingested.
4. **Multilingual Support:** Bangla (`bn-BD`) and English (`en-US`) are proposed; exact release and localization acceptance criteria remain To confirm.

## 4. Role-based access map (UI mapping; server-side enforcement required)

The role pages define UI affordances, not a second permission system. Map every control to the proposed [RBAC & Hierarchical Permission Matrix](../security/PERMISSION-MATRIX.md). That draft covers current role names, but its `Proposed`/`To confirm` grants are not effective until ADR-018 is approved. The summary below is only a scope guide—not executable permission grants. Any mutation requires an approved server-side permission and audit event. Tenant scope comes from authenticated identity, never from a client-selected organization ID.

| Role | Scope / key permissions | UI behavior and guardrail |
|---|---|---|
| Platform Super Admin | Global platform scope | Platform operations and parent-organization lifecycle; no routine raw customer evidence |
| Group Administrator | One parent organization and its companies | Group-level rollups and company administration; exact writes await canonical granular RBAC mapping |
| Group SOC Lead | One parent organization; company rollup | Group-level alert overview; per-company evidence only where explicitly granted |
| Company Administrator | One subsidiary company and its sites | Company fleet/user/site administration; no other company access |
| Company SOC Analyst | One company and assigned sites | Company alert review/triage; state changes depend on ADR-013 and RBAC mapping |
| Unit / Site Administrator | One assigned site | Local enrollment/health views; all writes require explicit site-scoped permissions |
| Security Administrator | Assigned Group or Company scope | Policy/rule/scan tools only when release and granular permissions permit |
| Security Auditor | Assigned Group or Company scope | Read-only scoped audit; export separately permissioned and audited |
| Executive Viewer | Assigned Group or Company scope | Aggregate read-only posture; no raw event/user/file/command-line data |

If the permission contract changes, update it first and then align these pages; do not broaden access to match a menu mockup.

Do not combine tenant auditor and platform auditor privileges into one implicit global role. Break-glass/support access is separate, off by default, time-bounded, customer-consented where applicable and fully audited (ADR-009).

## 5. Widget data contract

Every KPI, chart, table and badge must document or expose:

| Contract field | Requirement |
|---|---|
| Widget ID / title | Stable ID; localized display title separate from metric semantics |
| Release/status | Foundation v0.1, Release 1/2/3, Proposed or To confirm |
| Source | Authoritative API/event/store and source timestamp; no hard-coded demo values in production |
| Scope | Platform aggregate, organization, site, endpoint group or endpoint; enforce server-side |
| Definition | Numerator, denominator, exclusions, severity/state mapping and time window |
| Freshness | Source `as_of` time, update cadence and stale threshold; threshold is To confirm until approved |
| Empty/partial/error | Distinguish no data, not enabled, stale, permission denied, dependency degraded and loading |
| Interaction | Drill-down inherits scope/filters; no broader access through chart links or exports |
| Sensitivity | Mask personal/sensitive fields; record and audit exports and privileged views |

Metric examples in role documents are illustrative only. Do not show a green “verified/compliant/protected” result without a defined testable rule, coverage denominator and source evidence. Scores must expose their formula/version and unknown coverage; do not imply certification.

## 6. Shared UI, security and accessibility principles

1. **Server-side enforcement:** sidebar visibility is convenience only; every API, search, export, cache, drill-down and background query independently enforces authorization and tenant scope.
2. **State transparency:** distinguish `Loading`, `Live`, `Stale`, `Offline`, `No data`, `Not enabled`, `Partial/degraded`, `Permission denied` and `Error`. Do not hard-code `Stale >10m`; heartbeat/staleness thresholds are To confirm.
3. **Illustrative data:** sample values are marked illustrative in designs and use synthetic data only. Production widgets use actual authorized telemetry, otherwise show no-data state.
4. **Privacy by default:** minimize endpoint/user details; mask sensitive fields; avoid command lines, file contents and raw forensic material in summary views.
5. **Response safety:** disruptive response routes are hidden/disabled until Release 2 and ADR-014/action-specific approval. Non-disruptive scans are separate: show only for roles with `response:scan`, after Release 1 support is implemented, with scope confirmation and audit. No quick-release or one-click disruptive action.
6. **Localization:** Bangla (`bn-BD`) and English (`en-US`) are proposed product languages; exact release requirement is To confirm. Localize dates, numbers, severity labels, empty/error states and keyboard/screen-reader labels.
7. **Accessibility:** keyboard navigation, focus order, non-color-only severity indicators, readable chart tables/labels and reduced-motion behavior are design requirements; detailed acceptance target is To confirm.
8. **Auditability:** state-changing UI shows actor, scope, reason/confirmation where needed and result; backend remains source of truth.

## 7. Review checklist before implementation

- [ ] Product owner approves navigation scope for the selected release.
- [ ] ADR-018 approves the role permission matrix and it is mapped to API permissions.
- [ ] Canonical permission matrix has granular permissions for Group Admin, Group SOC Lead, Company Admin, Company SOC Analyst and Site Admin; legacy role columns are removed or explicitly mapped.
- [ ] Alert state transitions are resolved in ADR-013 before triage mutation is built.
- [ ] ADR-017 decides whether incident cases are a separate Release 1 entity/workflow or remain part of alert investigation.
- [ ] No disruptive response control is enabled until ADR-014 and one action's recovery/test plan are approved.
- [ ] On-demand scans appear only when Release 1 scan support exists and the user has `response:scan`; scope and audit behavior are verified.
- [ ] Every widget has source, scope, formula/meaning and freshness/empty/error behavior.
- [ ] Tenant boundaries, PII masking, export logging and permission-denied states are specified.
- [ ] Every menu item is tagged to the release matrix; unimplemented modules are hidden/disabled.
- [ ] UI tests cover role, tenant, loading, empty, stale, partial failure and keyboard behavior.
