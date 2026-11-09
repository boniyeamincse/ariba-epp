# Services

Planned domain modules. Per the architecture decision, these are **not** separate
deployables initially. Build them as modules inside the modular monolith, and
extract a service only when measured load or availability requirements justify it.

| Directory | Responsibility | Priority |
|---|---|---|
| `enrollment-service/` | One-time enrollment tokens, agent identity, renewal and revocation | P0 |
| `endpoint-service/` | Endpoint inventory, lifecycle, health and organization/site scope | P0 |
| `telemetry-ingestion/` | Schema validation, rate limits, deduplication, normalization | P0 |
| `detection-engine/` | Rule evaluation, correlation, severity and ATT&CK context | P0 |
| `policy-engine/` | Policy authoring, versioning, signing, assignment and acknowledgement | P0 |
| `alert-service/` | Alert lifecycle, assignment and notifications | P0 |
| `response-service/` | Guarded, approved, audited endpoint actions | P1 (guarded) |
| `inventory-service/` | Software inventory and vulnerability context | P1 |
| `notification-service/` | Delivery of alerts and workflow notifications | P1 |
