# Applications

Deployable user-facing applications. Both are planned; neither is implemented yet.

| Directory | Candidate technology | Purpose |
|---|---|---|
| `web-console/` | Next.js + TypeScript | Endpoint posture, alerts, investigation, policy and administration UI |
| `api-server/` | Python FastAPI | Management API; start as a modular monolith |

Before starting either app, confirm whether an existing Ariba SOC frontend or
backend can be reused, and record the decision in `docs/architecture/`.
