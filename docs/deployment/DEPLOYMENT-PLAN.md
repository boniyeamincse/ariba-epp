# Ariba EPP — Deployment and Operations Plan

**Status:** Draft deployment direction; no production topology or SLO is approved.  
**Initial use:** local development and isolated agent lab.

## 1. Environment boundaries

| Environment | Purpose | Data/actions |
|---|---|---|
| Developer | Fast local feature development | Synthetic data; no real credentials/endpoints |
| Lab | Agent/API/security integration validation | Owned isolated endpoints; observe-only by default |
| Pilot | Limited authorized user/endpoints | Staged, approved, monitored; response actions separately gated |
| Production | Customer/enterprise operation | Only after owner/security/operations release approval |

Docker Compose is a development/local-lab convenience, not proof of production hardening, high availability or isolation.

## 2. Development topology

Candidate components: web console, management API, PostgreSQL, Redis, RabbitMQ, OpenSearch, Nginx and optional scanner/integration services. Add only the dependencies required by the approved milestone. Containers should use non-default credentials supplied outside version control, least privilege, health checks, bounded resources and development-only network exposure.

Do not place real secrets in Compose files, sample `.env`, documentation or Git. Provide `.env.example` with placeholders only after implementation is authorized. Do not expose databases, brokers, search engines or admin APIs on public interfaces by default.

## 3. Pilot and production topology decisions

Before pilot, decide:

- Hosting provider, region and data residency for the centrally hosted multi-organization platform (deployment model approved in ADR-009); concrete tenant-isolation mechanism still to confirm.
- Service network segmentation, ingress/egress, DNS, certificate authority and key rotation.
- PostgreSQL, broker, search and object-store redundancy and backup/restore.
- Availability, RPO/RTO, endpoint/EPS limits and data retention.
- Monitoring, paging/on-call, security incident escalation and log ownership.
- Agent package distribution, staged deployment rings, rollback and maintenance windows.
- Customer export/delete, legal retention and deletion propagation across stores and backups.

Do not treat rough VM sizes in the architecture blueprint as a purchase or capacity specification; load-test with representative event rates and retention.

## 4. Production hardening gate

- TLS configuration and service identity; rotate/revoke credentials and certificates.
- Network segmentation and least-privilege service accounts.
- Secret manager/key custody and signing-key protection.
- Hardened container/host images, pinned/reviewed dependencies and vulnerability process.
- Protected audit logging, monitoring and alerting for privileged events and failures.
- Encrypted backups, restore rehearsal, disaster recovery and deletion propagation.
- Signed agent/policy artifacts, staged rollout, health verification and rollback.
- Resource quotas, queue/backpressure, disk alerts, log rotation and retention enforcement.
- Access review, MFA, break-glass account controls and incident response plan.
- Independent security review and documented pilot approval.

## 5. Observability requirements

Measure API availability/latency/errors, enrollment failures, agent last-seen/health, queue lag, event ingest/drop/duplicate counts, detection worker lag, alert latency, search health, database capacity, storage retention, update verification failures and privileged response outcomes. Avoid logs containing secrets or unnecessary process/user content. Monitoring data must include tenant-safe scoping.

## 6. Backup, recovery and upgrades

- Define RPO/RTO and retention before production.
- Back up PostgreSQL, search configuration/data as required, signing metadata and object references; protect backup keys separately.
- Test restore and tenant/audit integrity before production and at an approved cadence.
- Have rollback plans for API schema, DB migrations, agent package and policy version.
- Agent updates are staged by cohort, verified by signature and health, paused on elevated failures, and reversible.
- If an endpoint is isolated, maintain a restricted management/recovery path and validate it in lab first.
- Record operator, reason, change reference, version, completion and rollback result.

## 7. Operations readiness checklist

- Named service/component owner and escalation contact.
- Deployment/configuration guide with no embedded secrets.
- Supported OS and dependency matrix.
- Health dashboard and actionable alerts.
- Backup/restore, upgrade/rollback and incident runbooks.
- Data collection/retention/deletion and access review.
- Release provenance, SBOM and security issue reporting process.
- Pilot approval and endpoint owner authorization.