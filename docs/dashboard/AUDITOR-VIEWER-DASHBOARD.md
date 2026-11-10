# Ariba EPP — Auditor & Viewer Dashboard

**Roles:** Security Auditor / Executive Viewer  
**Security Boundary:** Scoped Read-Only Audit & Posture Visibility  
**Specification File:** `docs/dashboard/AUDITOR-VIEWER-DASHBOARD.md`  

**Status:** Proposed; owner/security review required. Auditor and Viewer are distinct roles with different scopes. This document does not establish ISO/SOC 2 compliance, audit immutability, or any certification.

---

## 1. Role Purpose & Authority

### Auditor
The Auditor needs read-only, scoped access to approved audit records for review. The scope is either a specific customer organization or a separately approved platform-audit scope; it is not automatically global.
- **Access:** Read-only to authorized audit records, policy history and case/action metadata if those records are in approved scope. Sensitive evidence is minimized/masked, and data export requires separate permission and audit.
- **Claim boundary:** “Tamper-evident”, “non-repudiable”, hash-chain integrity, and compliance packs are not claims until the audit design and evidence are independently defined and verified.

### Viewer (Executive / Manager)
The Viewer needs high-level executive visibility into the organization's security posture and, when defined and supported, risk/vulnerability trends without operational technical clutter. Incident-resolution metrics require an approved case lifecycle and formula.
- **Access:** High-level charts, summary reports, and read-only fleet posture.

---

## 2. Sidebar Navigation Visibility

### Auditor View:
```text
ARIBA EPP (Auditor Mode)
├── Dashboard (Audit and Authorized Posture Overview; no certification claim)
├── Policy Management (Release 1, read-only version history where permitted)
├── Detection & Response (Read-only alert investigations; separate incident cases To confirm in ADR-017)
├── Reports (Planned; export/compliance evidence only when specified and permissioned)
└── Administration
   └── Audit Logs (Scoped audit events; integrity status only if verified)
```

### Viewer View:
```text
ARIBA EPP (Executive View)
├── Dashboard (Executive Cyber Posture Overview)
├── Endpoint Management (High-level Fleet Status & OS Distribution)
├── Vulnerability Management (Release 2 / P1; planned)
└── Reports (Planned; export requires scope and audit controls)
```

---

## 3. Dashboard KPI Widgets

### For Executive Viewers:
| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| AVW-01 · Proposed | Fleet Protection Posture | Verified agent/policy status; assigned organization | Do not show a composite score until formula, version, coverage denominator and unknown-state handling are approved |
| AVW-02 · To confirm (ADR-017) | Active Incident Cases | Incident records; assigned Group/Company scope | Show only if a separate case lifecycle is approved; no response control |
| AVW-03 · Release 2 / P1 | Vulnerability Posture | Inventory service; assigned organization | Hidden until implemented; no 30-day SLA unless owner-approved policy exists |
| AVW-04 · To confirm (ADR-017; ADR-013 if based on alert states) | Resolution Trend | Approved case/alert audit timestamps; assigned Group/Company | Hidden until the measured lifecycle, formula, denominator, timezone and reporting window are approved |

### For Auditors:
| Widget ID / status | Widget | Proposed source / scope | Definition / freshness |
|---|---|---|---|
| AUD-01 · Proposed | Administrative Changes | Audit events; auditor's assigned scope | Actor/action/object/time/outcome; sensitive values masked; source timestamp and pagination |
| AUD-02 · Release 2 / ADR-014 | Response Action Records | Approved response audit records; auditor's assigned scope | Show actual action state and requester/approver/result; never imply an action ran from request status |
| AUD-03 · Proposed | Authentication / Access Events | Security audit source; assigned scope | Show defined security events, not a “brute-force” conclusion without validated detection logic |
| AUD-04 · Future | Audit Integrity | Approved audit-integrity verifier | Show `Not configured`/`Unknown` until verification mechanism and evidence are implemented; never hard-code green |

---

## 4. Key Workflows & Actions

1. **Audit Review & Export:**
   - Filter audit events only within assigned tenant/platform scope, with redaction and access checks.
   - Export is disabled until format, signing/integrity model, retention, permission and audit behavior are specified; every export request is itself audited.
2. **Evidence Reports:**
   - Report only controls with an authoritative source, known coverage and an approved definition. Do not label output ISO 27001, SOC 2 or regulatory-compliant without approved evidence and review.
3. **Executive Summary:**
   - PDF/export is a future feature; if enabled, apply tenant scope, data minimization, watermark/provenance, access audit and expiry controls.

## 5. Scope and privacy safeguards

- Organization Auditor sees only that organization's approved records; platform auditing is a separate role/permission.
- Viewer receives aggregates only; no raw events, user identifiers, command lines, file paths or forensic content.
- Audit/API error states must distinguish unavailable data from verified integrity.
- “Read-only” does not exempt viewing or export from access logging.
