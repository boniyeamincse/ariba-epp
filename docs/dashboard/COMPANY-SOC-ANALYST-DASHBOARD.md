# Ariba EPP — Company SOC Analyst Dashboard

**Role:** Company SOC Analyst
**Scope:** One assigned Company (`company_id`) and authorized sites
**Status:** Proposed; granular Company SOC permissions and hierarchy mapping remain blocked on RBAC reconciliation.

## Purpose and boundary

Review and investigate security alerts for one Company. The user must not see another Company’s events, raw file contents, unrelated user data or endpoint controls. Alert state changes follow ADR-013; disruptive response follows ADR-014 and is not enabled by this draft.

## Navigation proposal

```text
Company SOC Dashboard
├── Detection Events (Release 1; validated event summaries)
├── Alerts (Release 1; read-only until ADR-013 + permission mapping)
├── Alert Investigation (Release 1; authorized process/file evidence only)
├── Incident Cases (separate case lifecycle To confirm in ADR-017)
├── Threat Intelligence (approved source only; Release 2 integration candidate)
└── Response Status (Release 2; read-only only if approved and implemented)
```

## KPI widget contract

| Widget ID / status | Widget | Source / scope | Definition, freshness and guardrail |
|---|---|---|---|
| CAS-01 · Release 1 | Recent Detection Events | Validated event/alert service; Company/site scope | Event and ingestion timestamps; no claim of real-time streaming unless measured |
| CAS-02 · Release 1 + ADR-013 | Alert Queue | Alert records; Company/site scope | Count by approved status/severity; state machine and severity taxonomy required |
| CAS-03 · Release 1 | Endpoint Context | Endpoint inventory/health; Company/site scope | Minimum necessary metadata only; unknown/stale states explicit |
| CAS-04 · To confirm (ADR-017) | Incident Cases | Case service; Company scope | Hidden until a separate incident-case scope/lifecycle is approved |
| CAS-05 · Release 2 / ADR-014 | Response Outcomes | Action audit/result records; Company scope | Read-only and hidden until actions are approved/implemented |
| CAS-06 · Release 2 / P1 | IOC Enrichment | Approved threat-intel source; Company scope | Show provenance, confidence and expiry; hide when integration is absent |

## Workflows and guardrails

- Review rule ID/version, evidence references, event time, ingestion time and telemetry coverage.
- Claim, assign, add notes or mark false-positive only after ADR-013 and ADR-018 approval; create an incident only if ADR-017 approves a separate case lifecycle.
- A scan request may be available only after Company SOC receives an explicit `response:scan` mapping, and only for supported scan targets; audit request/result.
- No kill/quarantine/isolate/release control until ADR-014 and an action-specific approval, recovery path and lab evidence are approved.
- Do not show command-line or file evidence unless the exact fields and role are explicitly approved; always scope data at the server.
