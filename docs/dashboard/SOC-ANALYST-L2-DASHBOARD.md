# Ariba EPP — SOC Analyst L2 & Incident Responder Dashboard

**Role:** SOC Analyst L2 / Incident Responder  
**Security Boundary:** Deep Investigation & Guarded Response  
**Specification File:** `docs/dashboard/SOC-ANALYST-L2-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Foundation v0.1 has no investigation/response workflow. Release 1 supports only evidence actually collected and authorized. Release 2 response controls remain unavailable until ADR-014 and action-specific approval are complete.

---

## 1. Role Purpose & Authority
The Tier 2 Analyst investigates assigned-scope incidents using approved telemetry. A future responder may request an owner-approved action through a separate approval workflow. This role does not directly execute commands; kill, quarantine and isolation are not enabled by this draft.

> [!IMPORTANT]
> Any future response action requires, at minimum, and remains blocked until ADR-014 defines the authority/approval model:
> 1. Target re-validation (verifying PID, process path, and file hash).
> 2. Documented justification reason and idempotency key.
> 3. Automatic command expiration (TTL).
> 4. Non-repudiable audit recording.
> 5. Management channel preservation during host isolation.

---

## 2. Sidebar Navigation Visibility

```text
ARIBA EPP
├── Dashboard (Investigation Overview)
├── Detection & Response
│   ├── Alerts (Full investigative detail & evidence links)
│   ├── Incident Cases (standalone case lifecycle To confirm; priority/SLA definitions also To confirm)
│   ├── Investigation (Interactive process tree, timeline & forensic query)
│   └── Response Actions (Release 2; read-only status view only if approved; no command control)
├── Protection
│   └── Malware Scanning (Release 1; scan request only if supported, authorized and audited)
├── Endpoint Management (Contextual Host Details)
│   └── All Endpoints
└── Threat Intelligence (Release 2 / P1 integration; expose only approved, scoped IOC data)
```

---

## 3. Dashboard KPI Widgets

| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| L2D-01 · To confirm | Active Incident Cases | Incident records; assigned organization/sites | Show only if a separate case lifecycle is approved; priority labels and SLA timers are To confirm |
| L2D-02 · Release 2 / ADR-014 | Pending Approvals | Response workflow; assigned scope | Hidden until ADR-014 defines approval states; read-only status, never approve via a summary shortcut |
| L2D-03 · Release 2 / ADR-014 | Isolated Endpoints | Response outcome; assigned scope | Hidden until isolation is approved/implemented; no quick “Release” control |
| L2D-04 · Release 1 | Investigation Timeline | Process/file event evidence authorized for assigned scope | Preserve event and ingestion times, source and missing-data gaps; no fabricated sequence |
| L2D-05 · Release 2 / ADR-014 | Response Outcome History | Audited action records; assigned scope | Show only when action is implemented; actor/approver/result and timestamps, no raw command secrets |
| L2D-06 · To confirm (ADR-013) | MTTR | Incident state/audit timestamps | Hide until ADR-013 defines states and formula; any future value shows time window and denominator |

---

## 4. Key Workflows & Actions

1. **Root-Cause Investigation:**
   - Inspect process lineage only from validated, collected process events; sensitive command-line fields remain hidden unless separately approved.
   - Query only available, authorized event types and show coverage gaps; do not imply network telemetry until specified and collected.
2. **Guarded Remote Response Execution:**
   - Kill, quarantine and network-isolation buttons are absent/disabled until ADR-014 and one action-specific specification, approval rule, test plan and recovery path are owner/security approved.
   - A future action request must show target, scope, reason, approver requirement, expiry and expected impact before submission; the endpoint result is authoritative.
   - A scan request may be available only if supported by Release 1 implementation and separately permissioned/audited; do not expose arbitrary path input.
3. **Evidence Collection:**
   - No memory dump or arbitrary forensic collection in this dashboard scope. Any future evidence collection needs separate privacy, storage, authorization, retention and threat-model approval.
4. **Remediation & Incident Closure:**
   - Verify threat containment.
   - Endpoint release is a separate privileged action; no quick-release control. It requires the same authorization, target validation, audit and recovery controls as isolation.
   - Document investigation findings. Closing an incident case is available only if a separate case lifecycle is approved and the user has `incident:close`.

## 5. Response UI release gate

Before enabling any action control, require ADR-014 approval, action-specific requirements, role/API permission mapping, approver separation decision, target freshness rules, expiry/idempotency behavior, failure states, recovery procedure, isolated-lab test evidence and owner sign-off. Until then this page is investigation-only.
