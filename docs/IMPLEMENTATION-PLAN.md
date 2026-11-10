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

1. Owner-approved decision recorded in [the ADR register](decisions/README.md) (decision log/ADR).
2. `docs/PRODUCT-REQUIREMENTS.md` for product behavior, scope and acceptance.
3. Component contract: API, data model, agent specification, threat model, test strategy or deployment plan.
4. `docs/architecture/SYSTEM-ARCHITECTURE.md` for component boundaries and flows.
5. `docs/ARIBA-EPP-BLUEPRINT.md` and `docs/ARIBA-EPP-ENTERPRISE-ARCHITECTURE.md` for product direction/detail.
6. `docs/PROJECT-TASKS.md` is the detailed task-ID/checklist ledger only; it cannot override requirements or security contracts.
7. `docs/ARIBA-EPP-PRODUCT-IDEA.md` is source input only; examples/ideas there do not override approved requirements.

The ADR register is a proposal index only. No ADR is approved until the owner records an explicit decision/status there.

## 3. Canonical task ledger and order

Use [PROJECT-TASKS.md](PROJECT-TASKS.md) as the **only detailed implementation task ledger** (IDs M0–M10). This plan is the documentation approval gate and AI handoff contract; do not create or maintain a competing checkbox list here.

The conceptual dependency order is:

1. **Documentation approval gate:** owner resolves blocking questions in §4 and approves the Foundation v0.1 scope, contracts and decisions.
2. **M0 — Foundation & decisions:** development environment, quality gates and approved ADRs. Product implementation/setup tasks begin only after gate 1.
3. **M1 — Identity/RBAC/tenancy.**
4. **M2 — Endpoint identity/enrollment/heartbeat.**
5. **M3 — Telemetry.**
6. **M4 — Detection and alerts.**
7. **M5 — Policy engine.**
8. **M6 — Guarded response, lab only.**
9. **M7–M8 — Integrations and vulnerability context as approved.**
10. **M9–M10 — Hardening, packaging and pilot after evidence/review.**

The task IDs, estimates and milestone ordering in `PROJECT-TASKS.md` are planning inputs until the owner approves them. Never mark a task complete without implementation plus verification evidence. P0 roadmap scope is not identical to the smaller Foundation v0.1 proof milestone; see the requirements document.

## 4. Blocking open decisions (owner required)

Do not silently choose values for these:

1. Supported Windows builds, Linux distributions and kernels.
2. Hosting provider/region, data residency and operational targets for the centrally hosted multi-organization platform approved in ADR-009.
3. Identity provider, MFA and exact role/site permission matrix.
4. Agent telemetry allowlist, personal data treatment, retention, deletion and residency.
5. Heartbeat cadence/stale thresholds, event volume/EPS, target retention, availability and recovery objectives.
6. Which exact features ship in Foundation v0.1 versus later P0/P1/P2 phases.
7. Whether any response action is included in first pilot and its action-specific approval rules; observe-only is default until ADR-014 and action scope are approved.
8. Selected frameworks, libraries, broker, event store, scanner engines and commercial licenses.
9. API/event/policy schema compatibility and versioning rules.
10. Supported languages/UI design/accessibility expectations.

## 5. AI implementation contract

When the user later asks to implement:

1. Read `AGENTS.md`, this plan and the relevant specifications.
2. Confirm the docs gate is approved and the user selected a milestone/task; if not, propose the next documentation review action instead of changing product files.
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