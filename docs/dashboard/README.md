# Ariba EPP — Core Dashboard & Navigation Specification

**Status:** Draft UI contract; owner/security review required  
**Scope:** Navigation, role dashboards, widget data contract and visibility rules  
**Audience:** Product owner, UX, frontend/backend engineers, security and QA  

> This folder specifies intended UI behavior only. It does not authorize implementation of a feature or grant API permissions. Server-side authorization is authoritative. Follow `docs/PRODUCT-REQUIREMENTS.md`, approved ADRs and `docs/IMPLEMENTATION-PLAN.md` if anything conflicts.

## 0. Release and status rules

Use the owner-confirmed feature matrix in `docs/PRODUCT-REQUIREMENTS.md` as the source for release scope. Foundation v0.1 is only the first proof slice; it is not the whole Release 1.

| UI status | Meaning | Required behavior |
|---|---|---|
| `Foundation v0.1` | Endpoint enrollment, heartbeat, basic inventory/status and console shell | Show only data actually implemented; empty/loading/error states required |
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

To ensure strict least privilege and intuitive user experience, each role receives a customized dashboard and filtered sidebar visibility:

| Role | Specification File | Primary Scope & Responsibilities |
|---|---|---|
| **Platform Administrator** | [`PLATFORM-ADMIN-DASHBOARD.md`](PLATFORM-ADMIN-DASHBOARD.md) | Platform health, aggregate organization/fleet metadata, tenant lifecycle; no raw customer endpoint browsing. |
| **Organization Administrator** | [`ORG-ADMIN-DASHBOARD.md`](ORG-ADMIN-DASHBOARD.md) | Own-tenant endpoint/site/user administration and enrollment. |
| **Security Administrator** | [`SECURITY-ADMIN-DASHBOARD.md`](SECURITY-ADMIN-DASHBOARD.md) | Assigned-scope policy/rule review; signing and enforcement follow approved workflows. |
| **SOC Analyst L1 (Triage)** | [`SOC-ANALYST-L1-DASHBOARD.md`](SOC-ANALYST-L1-DASHBOARD.md) | Assigned-scope alert review; state-changing triage depends on ADR-013. |
| **SOC Analyst L2 / Responder** | [`SOC-ANALYST-L2-DASHBOARD.md`](SOC-ANALYST-L2-DASHBOARD.md) | Assigned-scope investigation; response controls unavailable pending ADR-014. |
| **Auditor & Viewer** | [`AUDITOR-VIEWER-DASHBOARD.md`](AUDITOR-VIEWER-DASHBOARD.md) | Separate scoped read-only audit and aggregate viewer experiences; no certification claim. |

---

## 3. UI Design & Security Principles

1. **Server-Side Enforcement:** The sidebar navigation hides unauthorized modules, but all API routes enforce server-side RBAC and tenant boundaries. Client-side menu hiding is purely for ergonomics.
2. **State Transparency:** Every widget distinguishes Live, Stale, Offline, No Data and degraded states using owner-approved freshness thresholds; this draft sets no stale threshold.
3. **Illustrative Data in Staging:** Sample counts and graphs in documentation/mockups are strictly labeled as illustrative; synthetic data only until real lab telemetry is ingested.
4. **Multilingual Support:** Bangla (`bn-BD`) and English (`en-US`) are proposed; exact release and localization acceptance criteria remain To confirm.

## 4. Role-based access matrix (UI mapping; server-side enforcement required)

The role pages define UI affordances, not a second permission system. Map every control to the canonical [`PERMISSION-MATRIX.md`](../security/PERMISSION-MATRIX.md); that contract defines the granular codes and server-side behavior. The summary below is only a navigation guide. Any mutation requires its explicit server-side permission and an audit event. Tenant scope comes from authenticated identity, never from a client-selected organization ID.

| Role | Scope / key permissions | UI behavior and guardrail |
|---|---|---|
| Platform Administrator | Platform scope; `org:create/update/suspend`, `endpoint:read` (GLOBAL), `health:view`, `audit:read` | Platform and aggregate views; endpoint metadata only when authorized/audited; no raw customer telemetry/files by default |
| Organization Administrator | Own organization; `site:manage`, `user:manage`, `user:assign_role`, `token:generate/revoke`, `endpoint:read/update/retire`, `health:view` | Tenant-only administration; never grant platform role or cross-organization access |
| Security Administrator | Assigned organization/sites; `policy:read/create/sign/assign/rollback`, `rule:manage`, scoped `alert:read/false_positive`, `ioc:manage`, `feed:sync`, `response:scan` | Policy/rule controls follow approved workflow; Release 1 scans require permission, scope confirmation and audit; no disruptive response |
| SOC Analyst L1 | Assigned organization/sites; `endpoint:read`, `health:view`, `alert:read/triage/false_positive`, `incident:create/read`, `ioc:read`, `response:scan` | Read-only endpoint evidence; triage waits for ADR-013; supported Release 1 scans are separately permissioned/audited; no disruptive response |
| SOC Analyst L2 / Responder | Assigned organization/sites; investigation permissions plus `incident:close`, `response:view`, `response:scan`; disruptive `response:* [A]` | Scans are separate from disruptive actions; disruptive requests require Release 2 and ADR-014; no direct execution/bypass |
| Auditor | Assigned organization; `audit:read/export`, scoped read permissions in canonical matrix | Read-only, tenant-scoped; export is separately permissioned and itself audited; no compliance claim by UI alone |
| Viewer | Assigned organization; `incident:read`, `vuln:read`, `policy:compliance`, `report:executive` as allowed | Aggregated read-only views; no raw event, user, file, command-line or response details |

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
- [ ] Role permission matrix is approved and mapped to API permissions.
- [ ] Alert state transitions are resolved in ADR-013 before triage mutation is built.
- [ ] Decide whether incident cases are a separate Release 1 entity/workflow or remain part of alert investigation.
- [ ] No disruptive response control is enabled until ADR-014 and one action's recovery/test plan are approved.
- [ ] On-demand scans appear only when Release 1 scan support exists and the user has `response:scan`; scope and audit behavior are verified.
- [ ] Every widget has source, scope, formula/meaning and freshness/empty/error behavior.
- [ ] Tenant boundaries, PII masking, export logging and permission-denied states are specified.
- [ ] Every menu item is tagged to the release matrix; unimplemented modules are hidden/disabled.
- [ ] UI tests cover role, tenant, loading, empty, stale, partial failure and keyboard behavior.
