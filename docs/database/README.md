# Database Documentation

PostgreSQL control-plane model and data-lifecycle documentation.

Planned contents:

- Entity-relationship diagrams and table catalog.
- Tenant-isolation model and Row-Level Security review.
- Retention and deletion rules for events, alerts, inventory and evidence.
- Backup, restore and migration-rollback evidence.

The schema sketch in the architecture document is illustrative only. Do not put
high-volume raw telemetry in PostgreSQL; use the event store.
