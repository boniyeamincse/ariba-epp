# Architecture Decision Records

This directory holds the decision register and ADR records. The register may list
Proposed/To confirm items; only an ADR explicitly marked **Approved** records an
owner-approved decision. A proposal is not approval.

## Conventions

- One file per decision: `ADR-NNN-short-title.md`.
- Status values: `Proposed`, `Approved`, `Superseded`, `Rejected`.
- `Approved` means the product owner accepted it; it is not proof that the feature
  is implemented, tested, certified or deployed.
- Record decision owner, date, context, decision, consequences and what remains
  to confirm.
- If a decision changes, write a new ADR that supersedes the old one; do not
  rewrite history silently.

## Approved

| ADR | Decision | Date | Status |
|---|---|---|---|
| [ADR-009](ADR-009-hosted-multi-tenant.md) | Centrally hosted multi-organization (multi-tenant) platform; customers are organizations | 2026-11-09 | Approved |

## Proposed / pending

| ADR | Topic | Current direction | Status |
|---|---|---|---|
| ADR-001 | Application topology | Modular monolith plus independently scalable workers before microservices | Proposed |
| ADR-002 | Control-plane database | PostgreSQL for transactional/control-plane records | Proposed |
| ADR-003 | Event search/storage | OpenSearch candidate; validate volume, retention and operations | Proposed |
| ADR-004 | Agent language/collectors | Go core with OS-native collectors; validate support and overhead | Proposed |
| ADR-005 | Backend/web stack | FastAPI and Next.js/TypeScript candidates; inspect repo/team constraints | Proposed |
| ADR-006 | Message broker | RabbitMQ candidate for initial async delivery | Proposed |
| ADR-007 | Prevention/scanning strategy | Integrate mature OS/security engines first; never disable existing protection | Proposed |
| ADR-008 | Deployment environments | Docker Compose for development/lab; production runtime hardening/region pending | Proposed |
| ADR-010 | Concrete tenant-isolation mechanism | Decide shared schema + org scope/RLS vs stronger physical isolation | To confirm |
| ADR-011 | Agent identity/enrollment | Unique identity and one-time enrollment; certificate/mTLS candidate | To confirm |
| ADR-012 | Event schema and retention | Version, field allowlist, compatibility and retention | To confirm |
| ADR-013 | Alert lifecycle | Statuses, transitions, assignment and false-positive disposition | To confirm |
| ADR-014 | Response action authority | Approval, requester/approver separation, action scope, expiry and recovery | To confirm |
| ADR-015 | User identity and MFA | Identity provider, session strategy and privileged MFA | To confirm |
| ADR-016 | Hosting region/residency | Provider/region, data residency and operational targets | To confirm |

This register is canonical for IDs and status. Architecture/task docs may reference an ADR, but must not allocate a different topic to the same ID. Approved ADR-009 is not superseded by these pending infrastructure/isolation choices.
