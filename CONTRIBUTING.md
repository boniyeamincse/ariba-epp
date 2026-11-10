# Contributing to Ariba EPP

## Before you start

- Read `AGENTS.md`, `docs/IMPLEMENTATION-PLAN.md`, `docs/DOMAIN-GLOSSARY.md`, and the relevant product/API/data/agent/security/test/deployment specifications.
- This repository is docs-first: do not begin product implementation until the owner approves the documentation gate and selects a milestone.
- Do not implement capabilities the documents mark as out of scope for the MVP.
- Do not describe planned or unverified work as complete, certified or
  production-ready.

## Working method

1. Complete/review the docs and get owner approval for scope, open decisions and a milestone.
2. Inspect repository/toolchain and implement only the selected task and requirement IDs.
3. Database: migration, tenant-isolation and restore tests.
4. Backend: authenticated APIs with negative authorization tests.
5. Frontend: operational views with explicit loading/empty/stale/error states.
6. Agent: OS-specific collection, resource budget and safe offline behavior.
7. Integrations: versioned contracts and failure handling.
8. Security testing: threat-model-derived tests and independent review where needed.
9. Release: signed packages, staged deployment, monitoring and rollback.

## Branches and pull requests

The proposed branch model is `main` (stable), `develop` (integration), and
`feature/<area>` branches. Confirm this against the repository's actual
convention before adopting it.

Every pull request should:

- Be small and scoped to one change.
- Include or update tests and documentation.
- State what is implemented, what is not, and how it was verified.
- Pass linting, type checks, unit tests, dependency scanning and secret scanning.
- Avoid unrelated reformatting.

## Code and security rules

- Never commit secrets, private keys, certificates, tokens, `.env` files or real
  endpoint/telemetry data.
- Never commit live malware; use approved benign test artifacts such as EICAR.
- Do not disable existing antivirus or host protection.
- Disruptive actions (process kill, quarantine, isolation, deletion) are denied
  by default and require authorization, approval, auditing, timeout and rollback.
- Validate all input at trust boundaries; enforce authorization server-side on
  every management action.
- Log no secrets or unnecessary personal data.

## Documentation rules

- **Confirmed** means a user-supplied requirement or information verified in the
  repository.
- **Proposed** means a recommendation that needs owner approval.
- **To confirm** marks an item that needs a product-owner decision.
- Sample metrics and UI data are illustrative and must never be shown as live
  telemetry.

## Commit messages

Use a clear, imperative summary line, for example:

```text
feat(agent): add heartbeats for linux collector
fix(api): reject expired enrollment tokens
docs(arch): record tenant-isolation decision
```
