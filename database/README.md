# Database

PostgreSQL is the proposed control-plane and transactional store.

| Directory | Contents |
|---|---|
| `migrations/` | Version-controlled schema migrations |
| `seeds/` | Safe development seed data (no real endpoint or personal data) |
| `schemas/` | Reference schema definitions and diagrams |

The SQL in the architecture document is a sketch, not a production migration.
Before use, add tenant scoping, constraints, indexes, retention rules,
authorization and migration-rollback tests. Never store plaintext passwords or
reusable agent secrets. Raw high-volume telemetry belongs in OpenSearch, not in
unbounded PostgreSQL JSON columns.
