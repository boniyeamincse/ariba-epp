# Ariba EPP Documentation

এই folder-এ Ariba EPP-এর source idea, requirements, architecture ও code-এর আগে review করার specifications/contracts রাখা হয়েছে। **নিয়ম: product owner docs review/approve না করা পর্যন্ত product code শুরু হবে না।** Technical choices/acceptance-এ `Proposed`, `To confirm` বা `Approved` status ব্যবহার হবে; planning text-কে implemented behavior বলে ধরা যাবে না।

## Documents

- [Ariba EPP Product Blueprint](ARIBA-EPP-BLUEPRINT.md) — vision, feature modules, MVP boundary, architecture direction, security principles, roadmap এবং open decisions।
- [Ariba EPP Enterprise Architecture](ARIBA-EPP-ENTERPRISE-ARCHITECTURE.md) — সংযুক্ত enterprise blueprint-এর বিস্তারিত architecture, modules, endpoint agent, data/API, policy/detection, security, delivery roadmap এবং প্রথম milestone।
- [Product Requirements](PRODUCT-REQUIREMENTS.md) — personas, goals/non-goals, testable functional requirements, quality attributes ও acceptance criteria।
- [Domain Glossary](DOMAIN-GLOSSARY.md) — canonical Platform/Group/Company/Site terms, roles, release/status language and alert/incident definitions.
- [Implementation Plan](IMPLEMENTATION-PLAN.md) — docs approval gate, task-source order, milestone DoD ও AI handoff protocol।
- [ADR Register](decisions/README.md) — unique decision IDs, status and canonical ADR location.
- [System Architecture](architecture/SYSTEM-ARCHITECTURE.md) — architecture boundaries, runtime/data flows, failure behavior ও decisions to confirm।
- [API Contract](api/API-CONTRACT.md) — proposed management/agent endpoints, authorization, payloads, errors and lifecycle rules।
- [Data Model](database/DATA-MODEL.md) — entities, tenant ownership, storage boundaries, retention and integrity constraints।
- [Endpoint Agent Specification](agent/AGENT-SPECIFICATION.md) — agent lifecycle, OS collectors, offline behavior, policy and safe command rules।
- [Threat Model](security/THREAT-MODEL.md) — assets, trust boundaries, threats, required controls and residual risks।
- [RBAC & Hierarchical Permission Matrix](security/PERMISSION-MATRIX.md) — Platform/Group/Company/Site permission proposal; all unapproved grants deny-by-default.
- [Test Strategy](testing/TEST-STRATEGY.md) — lab-only validation, test layers, release gates and acceptance evidence।
- [Deployment Plan](deployment/DEPLOYMENT-PLAN.md) — dev/lab/pilot/production boundaries, hardening, observability and recovery।
- [Ariba EPP Product Idea](ARIBA-EPP-PRODUCT-IDEA.md) — owner-এর দেওয়া মূল product concept (source material)।
- [Project Task List & Roadmap](PROJECT-TASKS.md) — M0–M10 milestone, step-by-step small tasks, language plan এবং tracking workflow।
- [Core Dashboard & Navigation](dashboard/README.md) — current Platform/Group/Company/Site role views, release gates, widget contracts and legacy-page notes.

## Subdirectories

| Directory | Purpose |
|---|---|
| `architecture/` | System design and architecture decisions |
| `api/` | Agent and management API contracts; implementation OpenAPI must conform |
| `agent/` | Agent lifecycle, support matrix and OS-specific requirements |
| `dashboard/` | Core dashboard navigation and individual role-based dashboard specifications |
| `database/` | Data model, retention and tenant isolation |
| `decisions/` | ADR register and owner-approved decision records |
| `deployment/` | Topologies, hardening and recovery |
| `security/` | Threat model, privacy and response safeguards |
| `testing/` | Test strategy, lab setup and acceptance evidence |
| `user-guide/` | Operator/user documentation (write after UI workflows are approved) |

## Documentation rules

- **Confirmed** বলতে user-এর দেওয়া requirement বা repository-তে যাচাই করা তথ্য বোঝায়।
- **Proposed** হলো আলোচনার ভিত্তিতে সাজানো recommendation; অনুমোদন না পাওয়া পর্যন্ত final decision নয়।
- **To confirm** অংশে product owner-এর সিদ্ধান্ত প্রয়োজন এমন বিষয় রাখা হবে।
- Current role hierarchy: Platform → Group → Company → Site → Endpoint. Unapproved permission proposals are not active grants.
- Dashboard-এর sample metric ও UI data সবসময় illustrative হবে; real telemetry নয়।
- বাস্তব endpoint-এ disruptive response action চালুর আগে explicit authorization, audit trail, safeguards এবং lab validation থাকতে হবে।
- Documentation-এ কোনো capability-কে implemented, production-ready, certified বা antivirus-equivalent বলা যাবে না যতক্ষণ implementation evidence না থাকে।
- Conflict হলে AI/developer প্রথমে [Implementation Plan](IMPLEMENTATION-PLAN.md)-এর source-of-truth order অনুসরণ করবে এবং implementation শুরুর আগে owner approval চাইবে।
