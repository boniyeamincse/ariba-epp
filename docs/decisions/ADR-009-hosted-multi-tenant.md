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
3. Each customer enterprise/conglomerate is represented by a Parent `organization` tenant (e.g. Akij Group). An organization contains subsidiary `companies` (e.g. Akij Plastic, Akij Piles, Akij Jute), and each company operates `sites` (e.g. Head Office, Factory 01, Depot). Endpoint records are intended to retain `organization_id`, `company_id`, and `site_id` to support hierarchical scope and rollups. This is a data-model direction, not a measured zero-join/performance guarantee; referential consistency and indexing require validation.
4. Every tenant-owned record carries a non-null `organization_id` and, where applicable, `company_id` and `site_id`. Tenant scope is enforced **server-side** on every API, worker, search, cache, queue, object-storage and export path. Submitted tenant IDs are never trusted.
5. Cross-organization and cross-company reads, writes, deletes and search results are **denied by default** and must have negative tests.
6. The product role families follow [PRODUCT-REQUIREMENTS.md](../PRODUCT-REQUIREMENTS.md):
   - **Platform Super Admin** (Ariba): create/suspend organizations, manage global platform settings and integrations. Global actions are audited.
   - **Group Administrator & Group SOC Lead**: manage parent group, roll up security visibility across all subsidiary companies.
   - **Company Administrator & Company SOC Analyst**: manage and monitor only their own company (e.g. Akij Plastic), its sites, users and endpoints. Cross-company access is denied.
   - **Unit / Site Administrator**: local factory/office endpoint enrollment and status.
   - Analyst, Auditor and Viewer roles are intended to be scoped to their assigned group/company/site. The detailed grants in [PERMISSION-MATRIX.md](../security/PERMISSION-MATRIX.md) are a proposal and require owner/security approval before implementation; this ADR does not approve each permission.
7. **The proposed default direction is logical isolation on a shared platform** (shared schema with organization and company scoping and, as defense in depth, PostgreSQL Row-Level Security if approved). The concrete mechanism remains ADR-010. A dedicated deployment for a customer that requires stronger isolation is a possible future option, not an MVP requirement.
8. Onboarding a customer creates: the parent organization, its first Group Administrator, default companies/sites where needed, and a tenant-scoped audit entry. No sample customer data is copied between organizations.
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
