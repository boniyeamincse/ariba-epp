# ADR-008 — Deployment environments and runtime topology

- **Status:** Proposed; owner approval required
- **Owner:** Product owner / platform owner (to assign)
- **Related:** [ADR-009 — hosted multi-organization platform](ADR-009-hosted-multi-tenant.md), [deployment plan](../deployment/DEPLOYMENT-PLAN.md)

## 1. Context

Development and lab environments need a repeatable, low-friction deployment. The longer-term product is a centrally hosted multi-organization platform per the approved ADR-009, but that decision does not specify production provider, region, high availability, service topology, SLO or capacity.

## 2. Proposed decision

1. Use Docker Compose as a candidate for local development and isolated lab only.
2. Do not treat Compose as proof of production hardening, high availability or customer isolation.
3. Keep production runtime topology, provider/region, redundancy, RPO/RTO and capacity as separate owner/platform decisions before pilot.
4. Production services must use least privilege, restricted network paths, managed secrets/keys, encrypted backups, monitoring and tested recovery.

## 3. Consequences

- Local iteration can use a small repeatable stack while preserving the explicit hosted multi-organization product direction.
- Production design cannot be inferred from local container topology.
- Deployment sizing must follow representative event-rate, retention, search and availability tests.

## 4. To confirm

- Product owner approval of this development-vs-production boundary.
- Hosting provider, region, data residency and customer requirements (ADR-016).
- Production service redundancy, SLO/RPO/RTO and recovery ownership.
- Whether managed services or self-hosted services are used for each data component.