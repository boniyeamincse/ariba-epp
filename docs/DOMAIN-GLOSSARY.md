# Ariba EPP — Domain Glossary and Canonical Vocabulary

**Status:** Proposed terminology baseline for owner review.  
**Purpose:** Keep product, API, database, dashboard and implementation-task language consistent. This glossary defines terms; it does not approve unresolved permissions or feature behavior.

## 1. Product and hierarchy terms

| Term | Canonical meaning | Identifier / scope |
|---|---|---|
| Platform | Centrally hosted Ariba EPP control plane shared by customer Groups | Platform scope; global access is reserved for explicitly approved Platform Super Admin permissions |
| Group / Parent Organization | Customer enterprise/conglomerate and top-level customer boundary in the hosted platform | `organization_id`; one Group may contain multiple Companies |
| Company / Subsidiary | Business entity within a Group | `company_id`; belongs to one `organization_id` and may contain multiple Sites |
| Site / Unit / Location | Factory, office, depot, branch or other operating location within a Company | `site_id`; belongs to one `company_id` |
| Endpoint / Asset | Managed Windows or Linux device with an Ariba Agent identity | `endpoint_id`; carries/resolves Group, Company and Site scope |
| Hierarchy path | Resource ownership path from platform to endpoint | Platform → Group → Company → Site → Endpoint |
| Scope | The maximum hierarchy/data subset a principal may access for a specific permission | `GLOBAL`, `GROUP`, `COMPANY`, `SITE` or `NONE`; specific grants are defined only by the approved RBAC matrix |
| Aggregate | Counts/statistics that do not reveal underlying records or evidence | Still requires an explicit aggregate permission and minimum privacy/tenant safeguards |
| Evidence/detail | Endpoint-level events, process/file metadata, user-linked data or forensic artifact | Separate, more sensitive permission; never implied by aggregate visibility |

**Important:** Group-level aggregate visibility is not the same as unrestricted access to every Company record. Company-scoped roles cannot access sibling Companies. Site scope is enforced independently. Examples of customer names in source material are illustrative context; use synthetic identifiers in development/tests.

## 2. Canonical role families

These role families are owner-confirmed product concepts. The exact permission mapping remains **Proposed/To confirm** in [RBAC & Hierarchical Permission Matrix](security/PERMISSION-MATRIX.md) under ADR-018; do not infer effective grants from a role title.

| Role ID | Display name | Intended scope |
|---|---|---|
| `platform_super_admin` | Platform Super Admin | Platform operations and Parent Group lifecycle; no routine raw customer evidence access |
| `group_admin` | Group Administrator | One Parent Group; group/company administration as approved |
| `group_soc_lead` | Group SOC Lead | One Parent Group; group alert rollups and investigation only as approved |
| `company_admin` | Company Administrator | One Company and its Sites |
| `company_soc_analyst` | Company SOC Analyst | One Company and explicitly assigned Sites |
| `site_admin` | Unit / Site Administrator | One assigned Site |
| `security_admin` | Security Administrator | Explicitly assigned Group/Company scope for policy/rule workflows |
| `auditor` | Security Auditor | Explicitly assigned Group/Company audit scope, read-only |
| `viewer` | Executive Viewer | Explicitly assigned Group/Company aggregate posture, read-only |

No role implicitly receives another role's permissions. Role stacking, delegation, self-approval and temporary access need owner/security decisions.

## 3. Product and release terms

| Term | Meaning |
|---|---|
| Foundation v0.1 | Narrow M0–M2 proof: local setup, hierarchy/identity basics, one Windows and one Linux lab endpoint enrolled, heartbeat/inventory status visible. It does not prove malware detection or deliver all Release 1 features. |
| Release 1 / P0 | Owner-confirmed initial product feature set: Endpoint/Agent Management, process/file telemetry, malware scanning, detection/alerts and policy management. Detailed OS support, field allowlist, alert states and signing workflow remain subject to their contracts/decisions. |
| Release 2 / P1 | Owner-confirmed capability phase including quarantine/guarded response, ransomware protection, host firewall, vulnerability management and application control. Disruptive actions still require ADR-014/action-specific approval. |
| Release 3 / P2 | AI Security Assistant and advanced threat hunting; no autonomous destructive action. |
| `Proposed` | Recommendation awaiting owner/security approval; not an effective requirement/grant. |
| `To confirm` | A decision is unresolved; dependent behavior remains blocked or unavailable. |
| `Approved` | Product owner/security approver recorded the decision/date; approval does not mean implemented or tested. |
| `PLANNED` | Task not implemented; no evidence claimed. |
| `VERIFIED` | Acceptance evidence exists and is linked; not a claim of certification or general effectiveness. |

Feature authority: the release matrix in [Product Requirements](PRODUCT-REQUIREMENTS.md). Execution authority: [Project Tasks](PROJECT-TASKS.md), after the [Documentation Gate](IMPLEMENTATION-PLAN.md).

## 4. Security workflow terms

| Term | Canonical meaning |
|---|---|
| Telemetry event | An approved, schema-versioned observation with event time and server ingestion time; hierarchy IDs derive from authenticated endpoint context, not the payload |
| Detection rule | Versioned logic evaluated against approved telemetry; must carry provenance, required fields and safe tests |
| Alert | A detection result with rule/version, evidence references, severity and investigation status. Alert state transitions are To confirm in ADR-013. |
| Incident case | A separate case entity that may group alerts and define ownership/closure/SLA. Whether this exists in Release 1 is To confirm in ADR-017. Do not equate an alert with an incident case. |
| Policy | Validated, versioned configuration assigned to approved Group/Company/Site/endpoint scope; agent verifies authenticity and reports acknowledgement |
| Response request | An authorized request for a specific action; it is not proof the endpoint executed it |
| Response execution/result | Endpoint-confirmed outcome for a signed, scoped and unexpired command; disruptive actions remain blocked pending ADR-014 |
| Audit event | A protected record of actor/agent, scope, action, target, time and outcome; do not call it immutable/tamper-proof until implementation evidence supports that claim |
| Freshness / stale | Comparison of event/heartbeat source time with server time under an approved threshold; no numeric threshold is set until decided |

## 5. Document source-of-truth links

- Product/release scope: [PRODUCT-REQUIREMENTS.md](PRODUCT-REQUIREMENTS.md)
- Role permission grants: [PERMISSION-MATRIX.md](security/PERMISSION-MATRIX.md) and pending ADR-018
- Alert states: pending ADR-013
- Incident-case scope: pending ADR-017
- Disruptive response authority: pending ADR-014
- Data and hierarchy: [DATA-MODEL.md](database/DATA-MODEL.md) and ADR-009
- Dashboard navigation/data behavior: [dashboard/README.md](dashboard/README.md)
- Docs gate and AI instructions: [IMPLEMENTATION-PLAN.md](IMPLEMENTATION-PLAN.md) and root `AGENTS.md`
