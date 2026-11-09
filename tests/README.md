# Tests

| Directory | Scope |
|---|---|
| `unit/` | Isolated logic and validation tests |
| `integration/` | Service, database and integration-contract tests |
| `security/` | Authorization, tenant-isolation, enrollment and response-safety tests |
| `performance/` | Event throughput, latency, resource budgets and retention tests |
| `e2e/` | Enrollment, heartbeat, telemetry and alert-flow tests |

Keep tests deterministic and safe: no live malware, no real credentials and no
calls to production endpoints. Every security claim in a report or release note
must point to reproducible test evidence.
