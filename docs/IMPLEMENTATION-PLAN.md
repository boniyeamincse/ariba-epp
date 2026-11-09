# Ariba EPP — Documentation Gate and Implementation Plan

**Status:** Documentation pack in progress; implementation is blocked pending owner review.  
**Purpose:** Give a future developer/AI a deterministic reading order, task sequence and completion contract.

## 1. Docs-first workflow

The owner wants documentation completed before coding. Follow these gates:

1. **Capture:** keep the original owner source in `ARIBA-EPP-PRODUCT-IDEA.md`; normalize requirements into the canonical docs below.
2. **Review:** owner reviews requirements, architecture, API/data contracts, agent/security behavior, test plan and deployment boundaries.
3. **Resolve blockers:** decide the high-impact items listed in §4. Use `Approved`, `Proposed` or `To confirm`; no implicit approval.
4. **Freeze Foundation v0.1:** confirm exact deliverables and acceptance criteria.
5. **Implement one phase at a time:** inspect the repo, implement only the approved task, keep tests/docs synchronized and report evidence.
6. **Verify:** run the relevant test and security checks; update status only from evidence.

**Current gate:** Gate 1/Review — the documents are drafts and owner review is required. Do not start code from this plan until the user explicitly selects/approves the milestone.

## 2. Document authority order

When two documents appear to disagree, use this order and stop on unresolved conflicts:

1. Owner-approved decision recorded in `docs/decisions/` (decision log/ADR).
2. `docs/PRODUCT-REQUIREMENTS.md` for product behavior, scope and acceptance.
3. Component contract: API, data model, agent specification, threat model, test strategy or deployment plan.
4. `docs/architecture/SYSTEM-ARCHITECTURE.md` for component boundaries and flows.
5. `docs/ARIBA-EPP-BLUEPRINT.md` and `docs/ARIBA-EPP-ENTERPRISE-ARCHITECTURE.md` for product direction/detail.
6. `docs/ARIBA-EPP-PRODUCT-IDEA.md` as source input only; examples/ideas there do not override approved requirements.

No owner-approved ADR directory exists yet. Create a decision record when the owner resolves a decision; do not treat empty directory structure as an approval.

## 3. Dependency-ordered backlog

Status for every item starts **PLANNED**. An AI must not mark tasks complete without repository change plus verification evidence.

| ID | Task | Dependencies | Exit evidence |
|---|---|---|---|
| DOC-01 | Owner approves product scope, deployment model and Foundation v0.1 acceptance | None | Dated decisions in docs |
| DOC-02 | Approve OS support matrix, privacy/telemetry allowlist and retention | DOC-01 | Versioned matrix and data policy |
| DOC-03 | Approve identity/RBAC/tenant model and response authorization | DOC-01 | Role/action matrix and threat-model review |
| DOC-04 | Approve technology/dependency/licensing choices | DOC-01, repo audit | ADRs with alternatives/rationale |
| DOC-05 | Review API, event, policy and DB contracts for consistency | DOC-02–04 | Contract review checklist passed |
| FND-01 | Establish development environment/repository conventions | DOC-01–05 | Repeatable clean setup |
| FND-02 | Implement authentication shell and server-side authorization | FND-01 | Positive + negative auth/RBAC tests |
| FND-03 | Implement organizations/sites and tenant-scoped schema | FND-01, FND-02 | Migration, isolation and restore tests |
| FND-04 | Implement one-time enrollment and unique agent identity | FND-02, FND-03, security decisions | Reuse/expiry/revocation tests |
| FND-05 | Implement Windows/Linux heartbeat prototypes | FND-04, agent spec | Lab endpoint reconnect tests |
| FND-06 | Display endpoint inventory and health in console | FND-03, FND-05 | UI/API E2E evidence |
| FND-07 | Establish safe logging, monitoring, backup and local deployment | FND-01–06 | Setup, restore and secret scans |
| TEL-01 | Finalize event schema and ingestion contract | DOC-02, DOC-05 | Schema review and fixture tests |
| TEL-02 | Implement bounded ingestion and storage adapter | TEL-01, FND-07 | Load/idempotency/retry tests |
| DET-01 | Add safe sample rule and versioned rule test harness | TEL-02 | Expected match/non-match tests |
| ALT-01 | Implement alert lifecycle and explainable alert detail | DET-01 | State/permission tests |
| POL-01 | Implement signed/versioned policy distribution | FND-04, DOC-03–05 | Signature/replay/rollback tests |
| INT-01 | Integrate selected scanner and SOC/Wazuh adapter | DOC-04, TEL-01 | Contract, timeout and retry tests |
| RSP-01 | Define one limited response action and recovery path | DOC-03, threat-model review | Owner approval + lab-only security tests |
| REL-01 | Pilot hardening and release evidence | Prior scope | Performance, backup, update, security review |

## 4. Blocking open decisions (owner required)

Do not silently choose values for these:

1. Supported Windows builds, Linux distributions and kernels.
2. On-prem/private cloud/hosted SaaS; single-tenant or shared multi-tenant control plane.
3. Identity provider, MFA and exact role/site permission matrix.
4. Agent telemetry allowlist, personal data treatment, retention, deletion and residency.
5. Heartbeat cadence/stale thresholds, event volume/EPS, target retention, availability and recovery objectives.
6. Which exact features ship in Foundation v0.1 versus later P0/P1/P2 phases.
7. Whether any response action is included in first pilot; observe-only is default until approved.
8. Selected frameworks, libraries, broker, event store, scanner engines and commercial licenses.
9. API/event/policy schema compatibility and versioning rules.
10. Supported languages/UI design/accessibility expectations.

## 5. AI implementation contract

When the user later asks to implement:

1. Read `AGENTS.md`, this plan and the relevant specifications.
2. Confirm user selected a milestone/task; if not, propose the next planned task instead of changing files.
3. Inspect current source, build/test commands, dependency and Git state. Do not assume a planned directory is empty or replace existing work.
4. If a high-impact `To confirm` affects the requested task, ask only the blocking question(s) or implement a non-committing scaffold that does not decide the issue.
5. Make the smallest change that satisfies named requirement IDs and acceptance criteria.
6. Update specification/API/schema/security/deployment docs whenever implementation changes their contract.
7. Add tests, run relevant checks and report exact pass/fail/unverified results.
8. Review diff and repository state; never commit/push unless user authorization and repository policy allow it.

## 6. Definition of Done

- Scope maps to requirement/task IDs and stays within approved milestone.
- Relevant API/data/agent/security docs match behavior; examples are not mistaken for actual data.
- Positive, negative, failure, authorization and tenant-boundary tests are included as appropriate.
- Safe lab-only test executed; no production action or live malware.
- Tests/checks and limitations are reported honestly.
- No secrets, real endpoint data, unrelated changes or unsupported product claims.
- Rollback/recovery/operational notes updated for agent, policy, data or response changes.

## 7. Status vocabulary

| Status | Meaning |
|---|---|
| `PLANNED` | Not implemented; no evidence claimed |
| `IN PROGRESS` | Work started; acceptance not yet met |
| `BLOCKED` | A decision/dependency prevents progress; name owner and question |
| `VERIFIED` | Acceptance evidence exists; cite tests/review/build |
| `DEFERRED` | Owner explicitly moved scope to a later milestone |

Do not use “done”, “secure”, “production-ready” or “commercial-grade” as status without defined evidence and owner approval.