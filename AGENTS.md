# Ariba EPP — AI contributor instructions

## Project status and first rule

This repository is in **documentation-first planning**. Do not start implementation, generate product code, or silently resolve a product/security decision until the owner approves the documentation gate and selects a milestone. The current source tree is a scaffold; product behavior is not yet implemented.

## Read these documents before acting

1. `docs/README.md` — documentation index and status vocabulary.
2. `docs/IMPLEMENTATION-PLAN.md` — source-of-truth order, open blockers and permitted next step.
3. `docs/PRODUCT-REQUIREMENTS.md` — goals, non-goals and acceptance criteria.
4. `docs/architecture/SYSTEM-ARCHITECTURE.md` — component boundaries and flows.
5. Relevant component contracts: `docs/api/API-CONTRACT.md`, `docs/database/DATA-MODEL.md`, `docs/agent/AGENT-SPECIFICATION.md`, `docs/security/THREAT-MODEL.md`, `docs/testing/TEST-STRATEGY.md`, and `docs/deployment/DEPLOYMENT-PLAN.md`.
6. `docs/ARIBA-EPP-PRODUCT-IDEA.md` is source input; use it only through the normalized decisions and requirements in the documents above.

## Source-of-truth and conflict rules

- Approved decisions/requirements in `docs/` outrank recommendations in source notes and examples.
- `Approved` is a product-owner decision. `Proposed` is not approval; `To confirm` is unresolved.
- If two normative docs conflict, stop and report the exact conflict. Do not choose whichever is easiest to code.
- Never infer production OS support, event fields, retention, tenant rules, response permissions, or compliance status.
- Do not copy malformed HTML/CSS Mermaid render output from source material; use valid Markdown and Mermaid source.

## Implementation rules after approval

- Implement only the explicitly selected milestone and its acceptance criteria; do not broaden scope.
- Before coding, inspect the existing repository, toolchain, dependencies, and conventions. Preserve existing user work.
- Update the relevant contract/spec before changing behavior. Keep API, database, agent, security, tests and user docs synchronized.
- Add negative authorization and tenant-isolation tests for security-sensitive endpoints.
- Prefer observe-only behavior first. No disruptive action is enabled by default.
- Test only on owned/authorized endpoints in an isolated lab. Never disable existing antivirus or host protections.
- Never use/commit live malware, real telemetry, credentials, private keys or certificates. Use benign approved test artifacts such as EICAR in isolated tests.
- Do not claim a feature is production-ready, certified, effective against all threats, or a full antivirus replacement without evidence and owner approval.
- Do not add a technology or dependency merely because it appears as a candidate in a document; first verify the approved ADR and repository constraints.
- Run relevant tests and report exact evidence, failures and limitations. Do not claim tests ran if they did not.

## Response-action safety

Process termination, quarantine, deletion, firewall changes and endpoint isolation are privileged. Deny by default; require server-side authorization, scope/target validation, explicit approval where required, expiry, idempotency, audit, result reporting and a tested recovery path. Never let an AI-generated score alone execute a destructive action.