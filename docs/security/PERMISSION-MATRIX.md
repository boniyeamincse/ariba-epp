# Ariba EPP — RBAC and Hierarchical Permission Matrix

**Status:** Proposed authorization contract under ADR-018; owner/security approval required before implementation.
**Approved direction:** Platform → Parent Group (`organization_id`) → Company (`company_id`) → Site (`site_id`) → Endpoint.
**Enforcement:** Server-side, deny by default. UI visibility is not authorization.

> Until this matrix is explicitly approved, `Proposed` and `To confirm` grants are **not effective**: deny the operation. This document resolves the prior mismatch between the current corporate roles and the old Org Admin / SOC L1 / SOC L2 permission columns; it does not claim the new grants are owner-approved.

## 1. Access-control rules

1. Derive role and scope from the authenticated identity/session. Never trust a submitted organization, company, site or endpoint ID to widen access.
2. Enforce scope on API, worker, search, cache, queue, object storage, export, notification and audit paths.
3. Cross-scope resource access returns `404` where revealing existence is sensitive; authenticated but insufficient permission on a known in-scope operation returns `403`. Unauthenticated/invalid credentials return `401`.
4. Separate aggregate access from record-level access. Group rollup does not automatically grant raw company telemetry; endpoint metadata does not automatically grant process/file evidence.
5. Every mutation, privilege grant/revocation, denied privileged attempt, export and response request is auditable. Never log secrets or unnecessary personal data.
6. High-risk endpoint actions are denied by default. This includes process termination, quarantine/restore, endpoint isolation/release, firewall changes and forensic collection.
7. On-demand scanning is a distinct, non-disruptive operation (`response:scan`); it still needs explicit permission, supported scope, confirmation/limits, result reporting and audit.
8. The platform model does not itself settle exact role grants. The matrix below is a proposed baseline; obtain owner/security approval per the register in §7.

## 2. Current roles and scope boundaries

| Role ID | Role | Maximum proposed scope | Boundary |
|---|---|---|---|
| `platform_super_admin` | Platform Super Admin | Global platform administration | No routine raw customer telemetry/files; exceptional support access requires a separately approved, customer-consented, time-bounded break-glass path (ADR-009) |
| `group_admin` | Group Administrator | One parent Group and its Companies/Sites | No other Group; company detail only for the Group's own Companies and only with the relevant permission |
| `group_soc_lead` | Group SOC Lead | One parent Group | Group-level alert aggregates; company evidence/drill-down requires an explicit grant |
| `company_admin` | Company Administrator | One Company and its Sites | No sibling Company, other Group or Platform administration |
| `company_soc_analyst` | Company SOC Analyst | Assigned Company and optionally assigned Sites | Triage scope only; no disruptive response by default |
| `site_admin` | Unit / Site Administrator | One assigned Site | Local operations only; endpoint/token/health permissions remain To confirm until explicitly mapped |
| `security_admin` | Security Administrator | Explicitly assigned Group and/or Company scope | Policy/rule workflow only; no automatic role elevation or response authority |
| `auditor` | Security Auditor | Explicitly assigned Group or Company | Read-only audit/evidence scope; export is separate and audited |
| `viewer` | Executive Viewer | Explicitly assigned Group or Company | Aggregate posture only; no raw events, user identifiers, command lines or forensic content |

A user may have multiple roles only if the identity/role-assignment policy explicitly permits it; effective access is the intersection of role grants with every assigned scope, not the union of scopes across unrelated tenants. Conflicts/role activation policy are To confirm.

## 3. Permission status vocabulary

- `Proposed`: product/security recommendation awaiting explicit approval; not effective.
- `To confirm`: important decision unresolved; operation denied until resolved.
- `Approved`: product owner/security approver recorded the decision/date and updated this matrix.
- `—`: explicitly denied for this role in the current proposed baseline.

In the tables, `P(scope)` means a proposed grant bounded to the named scope. `TBC (deny)` means no grant until decision. **No cells in the proposed tables are effective before approval.**

## 4. Proposed permission catalog

### 4.1 Platform and hierarchy administration

| Permission | Meaning | Proposed scope and eligible role |
|---|---|---|
| `org:create` | Create parent Group/customer tenant | Platform Super Admin responsibility is owner-confirmed by ADR-009; exact permission/API workflow awaits ADR-018 |
| `org:update` | Update Group settings/quotas | `GLOBAL`: Platform Super Admin; `GROUP`: Group Administrator for permitted Group settings only (exact fields TBC) |
| `org:suspend` | Suspend/reactivate a Group | Platform Super Admin lifecycle responsibility is owner-confirmed by ADR-009; exact permission, notice/impact and recovery controls await ADR-018; keep action unavailable until approved |
| `company:create` | Create subsidiary Company | Group Administrator onboarding responsibility is owner-confirmed by ADR-009; exact permission/API workflow awaits ADR-018 |
| `company:manage` | Update Company profile/settings | `GROUP`: Group Administrator; `COMPANY`: Company Administrator for own Company (field-level restrictions TBC) |
| `company:view` | Read Company metadata | `GROUP`: Group roles for own Group; `COMPANY`: Company roles for own Company; `SITE`: Site Admin only minimum site context |
| `site:manage` | Create/update/deactivate Sites | `GROUP`: Group Administrator; `COMPANY`: Company Administrator for own Company |
| `site:view` | Read Site metadata | Parent Group / Company / assigned Site scope, subject to role grants |
| `user:manage` | Invite/update/deactivate users | Group Admin in own Group; Company Admin in own Company; site role TBC (deny) |
| `user:assign_role` | Grant/revoke roles | Group Admin in own Group and Company Admin in own Company are proposed; must not grant roles/scopes beyond their own administrative authority; self-elevation denied |
| `user:view` | Read scoped user/role metadata | Group/Company admins in their scope; Security Admin/Auditor only if needed and approved; Site/SOC/Viewer default deny |

### 4.2 Endpoint and agent lifecycle

| Permission | Meaning | Proposed scope and eligible role |
|---|---|---|
| `token:generate` | Create one-time enrollment authorization | Company Admin in own Company; Site Admin `SITE` scope is TBC and denied until approved |
| `token:revoke` | Revoke enrollment authorization | Company Admin in own Company; Site Admin scope TBC and denied until approved |
| `endpoint:aggregate:read` | Read fleet counts/rollups without endpoint drill-down | Platform Super Admin global aggregate; Group Admin/Group SOC Lead Group aggregate; Company roles own Company; Viewer assigned scope |
| `endpoint:read` | Read endpoint inventory/metadata | Platform Super Admin global metadata only; Group roles Group scope; Company roles Company scope; Site Admin SITE scope is TBC; Auditor/Viewer only minimum fields if separately approved |
| `endpoint:evidence:read` | Read approved telemetry evidence | Company SOC Analyst / Group SOC Lead only within assigned Company/Site and only approved fields; current Release 1 scope is process/file, network evidence requires separate release/privacy approval; platform, admin, auditor, viewer default deny; detailed grant TBC |
| `endpoint:update` | Update approved inventory tags/site assignment | Group Admin in Group scope; Company Admin in Company scope; Site Admin TBC (deny); no identity/scope fields editable through generic update |
| `endpoint:retire` | Decommission endpoint | Company Admin own Company is proposed; requires confirmation, reason and audit; Site Admin TBC (deny) |
| `endpoint:revoke_cert` | Revoke endpoint agent identity | Platform Super Admin global; Company Admin own Company; Group/Security/Site roles TBC (deny) |
| `health:view` | Read agent health/resources/last-seen | Platform Super Admin operational metadata; Group/Company roles within assigned hierarchy; Site Admin only after explicit `SITE` mapping |

`endpoint:read` does not imply access to raw telemetry. `endpoint:evidence:read` is a separate permission. Group aggregates must be clearly separated from Company/Site drill-down.

### 4.3 Policy and detection configuration

| Permission | Meaning | Proposed scope and eligible role |
|---|---|---|
| `policy:read` | Read policy and history | Group/Company/Site scope per assignment; Viewer/Auditor read access only if approved |
| `policy:create` | Create draft policy | Security Administrator in explicitly assigned scope |
| `policy:review` | Review/approve policy before signing | Separate authorized reviewer; eligible roles and separation requirements TBC |
| `policy:sign` | Request signing via protected signing service | Security Administrator is a candidate; signer key custody and author-self-sign policy TBC; UI disabled pending approval |
| `policy:assign` | Assign signed policy to Group/Company/Site/group target | Security Administrator; Group/Company admin eligibility TBC; scope cannot exceed actor's grant |
| `policy:rollback` | Request rollback to prior signed version | Security Administrator, after explicit confirmation and audit; automatic rollback prohibited |
| `policy:compliance` | Read policy acknowledgement/drift | Group/Company security roles; Viewer/Auditor aggregates only if approved |
| `rule:manage` | Author/validate/enable approved detection rule versions | Security Administrator in assigned scope; deployment remains controlled and audited |
| `yara:manage` | Author/compile/test YARA rules | Security Administrator in assigned scope; does **not** by itself grant deployment/publishing |

### 4.4 Alerts and incident cases

| Permission | Meaning | Proposed scope and eligible role |
|---|---|---|
| `alert:read` | Read alert summary/detail | Company SOC Analyst in Company scope; Group SOC Lead group rollup with company detail separately granted; Security Admin only if approved; Auditor read-only metadata if approved |
| `alert:triage` | Assign, prioritize, add triage notes | Company SOC Analyst; Group SOC Lead in Group scope is proposed; exact fields/transition must follow ADR-013 |
| `alert:false_positive` | Mark false positive with rationale | Company SOC Analyst and Group SOC Lead within scope are proposed; Security Admin may tune rules but not disposition unless separately approved |
| `incident:create` | Create standalone case from alerts | Company SOC Analyst / Group SOC Lead proposed; **scope and lifecycle To confirm**—deny until product owner approves whether separate incident cases are Release 1 |
| `incident:read` | Read case timeline/details | Only if incident-case scope is approved; derive Group/Company/Site boundary from assigned role |
| `incident:close` | Close case with resolution | Company SOC Analyst/Group SOC Lead eligibility TBC; no closure until case lifecycle/permissions approved |

ADR-013 covers alert states/transitions. ADR-017 governs whether a separate incident-case entity, SLA model or closure workflow exists.

### 4.5 Scan, response, threat intelligence and audit

| Permission | Meaning | Proposed scope and eligible role |
|---|---|---|
| `response:scan` | Request supported on-demand scan | Security Admin, Company SOC Analyst and Group SOC Lead are candidates within assigned scope; Site Admin TBC; permission does not exist until explicitly approved and scan implementation is verified |
| `response:view` | Read response request/result metadata | Assigned security roles only after response feature release; never implies raw evidence access |
| `response:kill_process` | Request process termination | Eligible role To confirm: Company SOC Analyst with explicit elevation vs a separately approved Responder role; Release 2/ADR-014; unavailable by default |
| `response:quarantine` | Request file quarantine | Eligible role To confirm under ADR-014/ADR-018; Release 2; unavailable by default |
| `response:restore` | Restore quarantined object | Eligible requester/approver roles To confirm under ADR-014; unavailable until recovery design approved |
| `response:isolate` | Request endpoint isolation | Eligible role To confirm under ADR-014/ADR-018; Release 2; management-channel recovery test required |
| `response:release` | Release isolated endpoint | Separate authorization/approval and target validation required; not a quick-release control |
| `response:forensics` | Request forensic collection | Deny by default; privacy, evidence scope, retention and ADR-014/action-specific approval required |
| `ioc:read` | Read approved indicators/provenance | Security Admin and scoped SOC roles when integration is released; Viewer/Auditor default deny |
| `ioc:manage` | Add/update/expire IOCs | Security Admin in assigned scope; provenance and licensing checks required |
| `feed:sync` | Trigger/configure approved feed sync | Security Admin in assigned scope; Platform Super Admin only for platform integration configuration; exact separation TBC |
| `vuln:read` | Read inventory/CVE findings | Release 2 / P1 only; Company/Group scope and field filtering required |
| `audit:read` | Read audit events | Platform Super Admin for platform-admin events; Group/Company Auditor only within assigned scope; SOC access TBC/deny |
| `audit:export` | Export scoped audit records | Auditor or admin only when separately approved; export is itself audited and redacted |
| `report:executive` | Read approved aggregate executive report | Viewer in assigned Group/Company scope after report definitions exist |
| `report:compliance` | Produce compliance evidence pack | TBC; no compliance claim or report-generation grant until control mappings and evidence are approved |

## 5. Effective authorization algorithm

1. Authenticate principal and resolve current active role assignments.
2. Derive scope from server-side identity: platform, `organization_id`, `company_id`, `site_id` and endpoint ownership.
3. Check the exact permission code, status `Approved`, required scope, release flag and action-specific approval state.
4. For reads, filter at query time and re-check scope on each referenced object, including evidence/search/export/background jobs.
5. For writes, validate permitted fields and state transitions; never allow changing hierarchy IDs through generic update.
6. For protected or high-impact writes, record actor, effective role/scope, reason, target, request/idempotency ID, approvals, timestamps and outcome.
7. Unknown permission, mismatched scope, unapproved permission, stale/expired approval or unresolved `To confirm` item is denied.

Return `401` for missing/invalid authentication; `403` for authenticated principals lacking a permission; use `404` for inaccessible object IDs when needed to prevent existence disclosure. Do not mix the two responses arbitrarily.

## 6. Required security tests

1. Cross-Group, cross-Company and cross-Site read/write/search/export attempts fail and disclose no resource existence.
2. Group Admin sees its Group's Company list but cannot access another Group or edit endpoint evidence.
3. Company Admin cannot enumerate sibling Companies in the same Group.
4. Site Admin sees no endpoints/health/tokens until corresponding SITE-scoped permissions are approved and granted.
5. Group SOC Lead aggregate view does not expose Company detail absent explicit company permission.
6. Platform Super Admin default view excludes raw customer telemetry/files; break-glass is disabled until separately approved.
7. UI-hidden action called directly through API is still denied.
8. Unapproved/TBC permissions, Release-gated features and disabled response actions fail closed.
9. User role assignment cannot self-elevate or grant a scope wider than the grantor's authority.
10. Every privileged deny, grant, role change, export and response request creates a protected audit event without secrets.

## 7. Owner/security decisions required before permission implementation

- Approve the proposed scope/action matrix for each of the nine current roles (ADR-018).
- Decide if Group SOC has alert/evidence visibility across Companies and at what granularity.
- Decide alert-state and Company/Group SOC triage/false-positive grants (ADR-013, ADR-018).
- Decide if a separate incident-case lifecycle exists in Release 1 and which roles may create/read/close cases (ADR-017, ADR-018).
- Decide Company/Site Admin token generation, endpoint read/health, retirement and certificate-revocation grants.
- Decide Security Admin policy author/sign separation, approval/signing service and key custody.
- Decide which on-demand scan roles are allowed and whether each scan type needs confirmation/approval.
- Decide whether a separate Incident Responder role is needed or an existing Company SOC Analyst may receive action-specific elevation.
- Approve ADR-014 for every disruptive response action, approver separation, expiry, recovery and release procedure.
- Decide auditor/report scopes and which compliance evidence (if any) is supportable.
- Confirm role stacking, temporary grants, joiner/mover/leaver, MFA, break-glass and access-review rules.

Until these are decided and recorded, the matrix is review-ready documentation, not an implementation authorization.
