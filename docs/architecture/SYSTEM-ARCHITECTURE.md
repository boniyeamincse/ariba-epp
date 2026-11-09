# Ariba EPP — System Architecture

**Status:** Proposed architecture for owner review.  
**Decision direction:** Modular monolith for control plane, independently scalable event/detection workers, Windows/Linux agents, hybrid security engines.

## 1. Context and constraints

- EPP owns endpoint identity/lifecycle, policy, detection orchestration and endpoint response.
- Ariba SOC/Wazuh is an integration and cross-source SOC layer, not the EPP frontend/control-plane substitute.
- Initial endpoint platforms are Windows and Linux; versions are unresolved.
- Full antivirus/kernel prevention is not established by this architecture. Use OS protections and mature integrations until a specific enforcement mechanism is designed, tested and approved.
- Enterprise target requires tenant-aware authorization, audit, signing, update/recovery, privacy and measured scalability.

## 2. Logical components

| Component/module | Boundary | Initial deployment direction |
|---|---|---|
| Web console | User workflows; never sole authorization boundary | Separate web app or existing UI only after repo review |
| Management API | User/agent auth, validation, authorization and orchestration | Modular monolith |
| Identity/enrollment | User roles, tenant scope, endpoint identity, renewal/revocation | Module in control-plane app |
| Endpoint management | Inventory, health, lifecycle, group/site assignment | Module + PostgreSQL |
| Policy | Authoring, versions, signing, assignments, acknowledgements | Module + PostgreSQL; signed agent-facing bundle |
| Telemetry ingestion | Schema validation, rate/size limits, dedupe, normalization | Independently scalable worker/API boundary |
| Detection/alert | Rule evaluation, enrichment, correlation, alert lifecycle | Worker/module; OpenSearch event refs + PostgreSQL control state |
| Response | Authorization, approval, command expiry, result and recovery | Isolated privileged module; disabled by default initially |
| Integrations | Wazuh, MISP, OpenSearch and scanner adapters | Versioned adapters; failures isolated from core state |
| Storage | Transaction data vs event search vs evidence | PostgreSQL, OpenSearch and restricted object storage roles |

Do not deploy each logical module as a microservice in the first version. Extract only after measured scaling, fault-isolation or ownership needs justify it.

## 3. Trust boundaries and data flow

1. **User/browser → API:** authenticated session/token, CSRF/origin policy as applicable, server-side RBAC/tenant scope.
2. **Agent → gateway:** unique endpoint certificate identity, authenticated encrypted transport, bounded and validated requests.
3. **Gateway → broker/workers:** authenticated service identity, schema version, bounded payload and retry/idempotency metadata.
4. **Workers → stores:** least-privilege service access; raw telemetry has explicit retention and tenant keys.
5. **Admin policy → agent:** validate, sign, stage, scope, acknowledge and support controlled rollback.
6. **SOC → EPP response API:** separate integration identity and scoped permission; every request is treated as untrusted until authorized.
7. **EPP → endpoint response:** expiring signed command, endpoint/target validation, allowed-action policy and recorded result.

### Event path

Agent collector → local bounded queue → authenticated ingestion → validation/deduplication/normalization → event store → detection/rule evaluation → alert/control records → EPP console and optional SOC integration.

### Policy path

Authorized administrator → policy validation/version → approval/signing → scope assignment → agent fetch → signature and version check → apply/deny → acknowledgement and audit.

### Response path (later, guarded)

Analyst request → API identity/scope checks → action-specific approval and freshness checks → expiring command → agent signature/target/expiry check → execution result → audit, console status and recovery path.

## 4. Failure behavior

| Failure | Required behavior |
|---|---|
| Management API unavailable | Agent retains bounded permitted queue; no unbounded disk growth; existing OS security remains active |
| Broker unavailable | Backpressure and retry; reject/limit new ingestion when safe bounds are reached; expose degraded health |
| Search store unavailable | Do not silently discard; queue/retry within bounds, record degradation and preserve control-plane state |
| Integration unavailable | Retry with limits/idempotency; EPP core identity/policy state remains authoritative |
| Agent offline | Mark status stale/offline by approved threshold; queued command expires and is not reported executed |
| Invalid/unsigned/old policy | Reject, report reason and retain last-known-good state where safe |
| Update verification fails | Abort update; keep current agent operational and emit health/audit signal |
| Isolation recovery path unavailable | Do not enable isolation policy; require tested break-glass/release mechanism before rollout |
| Database restore | Verify tenant consistency, audit chain/integrity and agent credential lifecycle before reopening writes |

## 5. Availability and scale

No availability, throughput, latency, endpoint maximum or retention SLO is approved. Determine these through owner decision and representative load testing. Scale ingestion/search independently when measurements justify it. Candidate sizing in the enterprise blueprint is indicative only and must not be used for procurement.

## 6. Architecture decisions to record

Create an ADR in `docs/decisions/` for each owner-approved decision. Initial proposed decisions:

| ADR | Topic | Current direction | Status |
|---|---|---|---|
| ADR-001 | Application topology | Modular monolith + scalable workers before microservices | Proposed |
| ADR-002 | Control-plane data | PostgreSQL | Proposed |
| ADR-003 | Event search | OpenSearch candidate; validate operations/retention | Proposed |
| ADR-004 | Agent language | Go candidate | Proposed |
| ADR-005 | Backend/UI | FastAPI and Next.js candidates; inspect repo before selection | Proposed |
| ADR-006 | Broker | RabbitMQ candidate; compare requirements before selection | Proposed |
| ADR-007 | Endpoint prevention | Integrate OS/native mature engines first; don't disable existing protection | Proposed |
| ADR-008 | Deployment | Docker Compose for development only; production topology separately approved | Proposed |

## 7. Architecture acceptance checklist

- Every component has a clear source-of-truth and privilege boundary.
- Tenant and identity context propagates end-to-end and is tested at each API/worker/storage boundary.
- Raw event, control-plane, quarantine and audit storage purposes are distinct.
- All asynchronous paths define idempotency, retry, backpressure, failure visibility and retention.
- Agent update, policy signing, command expiry and offline recovery are specified.
- SOC integrations cannot bypass EPP authorization for endpoint action.
- No production architecture claim exists without deployment, recovery and load-test evidence.