# ADR-009 — Hosted multi-organization platform (multi-tenant)

- **Status:** Approved by product owner
- **Date:** 2026-11-09
- **Owner:** Product owner
- **Related:** ADR-008 (deployment), `../PRODUCT-REQUIREMENTS.md` (FR-ID-01…03), `../database/DATA-MODEL.md` (tenant isolation rules)

## 1. Context

Ariba EPP will be hosted in one central location and used by multiple companies. The owner's examples are Akij Group, Boni IT and Nazim Group: each is a separate customer organization that must operate on the same Ariba EPP platform without seeing another organization's data.

This decision replaces the earlier open question "single-tenant on-prem vs hosted multi-tenant". It does **not** approve production deployment, availability targets, compliance status or data-residency commitments.

## 2. Decision

1. Ariba EPP is operated as a **centrally hosted platform** reachable over HTTPS. Customers do not run their own EPP control plane in the MVP.
2. The platform is **multi-organization**: multiple customer organizations share the same platform and infrastructure.
3. Each customer is represented by exactly one `organization` tenant. An organization may contain multiple `sites` (for example locations or departments).
4. Every tenant-owned record carries a non-null `organization_id` (directly or through a verified parent). Tenant scope is enforced **server-side** on every API, worker, search, cache, queue, object-storage and export path. Submitted tenant IDs are never trusted.
5. Cross-organization reads, writes, deletes and search results are **denied by default** and must have negative tests.
6. Roles are split by boundary:
   - **Platform Administrator** (Ariba): create/suspend organizations, manage platform settings and integrations. Global actions are audited.
   - **Organization Administrator**: manage only their own organization, its sites, users and endpoints.
   - Analyst, Auditor and Viewer roles are scoped to their permitted organization/site.
7. **Default isolation is logical isolation on a shared platform** (shared schema with organization scoping and, as defense in depth, PostgreSQL Row-Level Security if approved). A dedicated deployment for a customer that requires stronger isolation is a possible future option, not an MVP requirement.
8. Onboarding a customer creates: the organization, its first Organization Administrator, default site(s) where needed, and a tenant-scoped audit entry. No sample customer data is copied between organizations.
9. Platform-level support access (for example break-glass or support impersonation) is **not** enabled by default and requires an explicit, audited policy with customer consent.

## 3. Consequences

### Positive

- One platform to operate, monitor, patch and back up.
- Customers onboard quickly as new organizations without a new deployment.
- Shared detection engineering and integrations benefit every organization.
- Clear tenant boundary in the data model from day one.

### Risks and mitigations

| Risk | Mitigation |
|---|---|
| Cross-tenant data leak through a missing filter | Tenant scope required in the service layer, RLS defense in depth, cross-tenant tests per endpoint/worker, deny-by-default |
| Noisy tenant consumes shared capacity | Per-organization quotas, rate limits and fair-use monitoring |
| Tenant data deletion/export disputes | Documented per-tenant retention, export and deletion process before pilot |
| Platform admin overreach | Least privilege, break-glass audit, customer-visible access log, no silent impersonation |
| Region/residency requirements differ per customer | Resolve hosting region and residency policy before onboarding the first external customer |

## 4. Still To confirm (not approved by this ADR)

- Hosting provider, region and data-residency policy.
- Concrete isolation mechanism: shared schema + `organization_id` + RLS vs schema-per-tenant vs database-per-tenant.
- Per-tenant quotas, retention, export, deletion and legal-hold behavior.
- Billing/plan model, if any.
- Custom domain/branding per organization.
- Per-tenant backup/restore and disaster-recovery targets.
- Whether an organization can have multiple workspaces/environments.
- Support-access, break-glass and impersonation policy.

## 5. Required evidence before this decision is called implemented

- Organization provisioning and suspension demonstrated end-to-end.
- Cross-organization negative tests pass for API, search, queue, cache and object storage.
- Tenant-scoped audit records verified.
- Backup/restore preserves tenant relationships and audit integrity.
- Isolation mechanism recorded in a follow-up ADR and reviewed against the threat model.
