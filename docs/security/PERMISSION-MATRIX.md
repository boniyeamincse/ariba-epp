# Ariba EPP — RBAC & Tenant Permission Matrix

**Status:** Canonical Security Contract (Approved Direction)  
**Security Model:** Role-Based Access Control (RBAC) + Multi-Tenant Scoping (`organization_id` & `site_id`)  
**Enforcement:** Server-Side Mandatory Enforcement on all API Routes; Deny by Default  
**Audience:** Security Engineers, Backend Developers, Frontend Developers, QA  

---

## 1. Principles of Access Control

1. **Deny by Default:** Any action not explicitly granted in the matrix below is denied with HTTP `403 Forbidden`.
2. **Tenant Boundary Is Absolute:** Even a user with `endpoint:read` cannot view or query endpoints outside their assigned `organization_id`. Cross-organization requests must return `404 Not Found` (to avoid leaking record existence).
3. **Privileged Actions Require Verification:** Disruptive actions (`response:*`) cannot be executed by single-click or without an explicit target re-validation, documented operational reason, idempotency key, and automatic expiration TTL.
4. **Separation of Duties (SoD):** The persona authoring and signing policies (Security Administrator) is decoupled from the persona triaging live alerts (SOC Analyst L1) and the persona managing overall corporate tenancy (Organization Administrator).
5. **Audit Everything:** Every grant, revocation, privileged execution, and authentication failure is immutably recorded in `audit_logs`.

---

## 2. Standard Platform Roles (Corporate Hierarchy)

| Role Identifier | Role Display Name | Primary Scope | Operational Summary |
|---|---|---|---|
| `platform_admin` | **Platform Super Admin** | Global (Cross-Tenant) | System-level maintenance, tenant onboarding/quotas, platform health. No routine access to customer files. |
| `group_admin` | **Group Administrator** | Parent Group (`org_id`) | Overall security visibility across all subsidiary companies; onboard new companies, manage group users. |
| `group_soc` | **Group SOC Lead** | Parent Group (`org_id`) | Group-wide threat monitoring, cross-company alert visibility, and threat hunting. |
| `company_admin` | **Company Administrator** | Single Company (`company_id`) | Specific business entity management (e.g. Akij Plastic); manages factory/office sites, company users, and fleet. |
| `company_soc` | **Company SOC Analyst** | Single Company (`company_id`) | Triage, investigations, and alert handling strictly within their assigned company. |
| `site_admin` | **Unit / Site Administrator** | Single Site (`site_id`) | Local endpoint enrollment and agent health for a specific factory, branch, or depot. |
| `security_admin` | **Security Administrator** | Group / Company | Authoring, signing, and assigning policies; configuring protection engines; managing YARA & CTI feeds. |
| `auditor` | **Security Auditor** | Scoped (Group or Company)| Read-only compliance reviews, tamper-evident audit trail inspection, regulatory reporting. |
| `viewer` | **Executive Viewer** | Scoped (Group or Company)| Executive dashboards, posture summaries, high-level SLA metrics. No operational tooling. |

---

## 3. Comprehensive Granular Permission Matrix

### Legend:
- **`GLOBAL`**: Granted across all organizations in the platform (Platform Admin only).
- **`GROUP`**: Granted across all subsidiary companies in the user's parent organization (`organization_id`).
- **`COMPANY`**: Granted strictly within the user's assigned subsidiary company (`company_id`).
- **`SITE`**: Granted strictly within the user's assigned operating unit/site (`site_id`).
- **`NONE`**: Explicitly denied.
- **`[A]`**: Requires dual-authorization / approval workflow if enabled by policy.

---

### 3.1 Tenancy & Corporate Administration

| Permission Code | Permission Description | Platform Admin | Group Admin | Company Admin | Site Admin | Security Admin | Group/Co SOC | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `org:create` | Onboard new Parent Group | **GLOBAL** | NONE | NONE | NONE | NONE | NONE | NONE | NONE |
| `org:update` | Edit Group settings & quotas | **GLOBAL** | **GROUP** | NONE | NONE | NONE | NONE | NONE | NONE |
| `org:view` | View Group profile | **GLOBAL** | **GROUP** | **COMPANY** | **SITE** | **GROUP** | **GROUP** | **GROUP** | **GROUP** |
| `company:create` | Onboard subsidiary Company | **GLOBAL** | **GROUP** | NONE | NONE | NONE | NONE | NONE | NONE |
| `company:manage` | Edit Company settings & units | **GLOBAL** | **GROUP** | **COMPANY** | NONE | NONE | NONE | NONE | NONE |
| `company:view` | View Company details | **GLOBAL** | **GROUP** | **COMPANY** | **SITE** | **GROUP** | **GROUP/CO** | **GROUP** | **GROUP/CO** |
| `site:manage` | Create, edit, delete sites | **GLOBAL** | **GROUP** | **COMPANY** | NONE | NONE | NONE | NONE | NONE |
| `site:view` | View sites and locations | **GLOBAL** | **GROUP** | **COMPANY** | **SITE** | **GROUP** | **GROUP/CO** | **GROUP** | **GROUP/CO** |
| `user:manage` | Invite, update, deactivate users | **GLOBAL** | **GROUP** | **COMPANY** | NONE | NONE | NONE | NONE | NONE |
| `user:assign_role`| Assign RBAC roles to users | **GLOBAL** | **GROUP** | **COMPANY** | NONE | NONE | NONE | NONE | NONE |
| `user:view` | View user list & assigned roles | **GLOBAL** | **GROUP** | **COMPANY** | **SITE** | **GROUP** | NONE | **GROUP** | NONE |

---

### 3.2 Endpoint & Agent Lifecycle

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `token:generate` | Generate agent enrollment tokens | NONE | **ORG** | **ORG** | NONE | NONE | NONE | NONE |
| `token:revoke` | Revoke active enrollment tokens | NONE | **ORG** | **ORG** | NONE | NONE | NONE | NONE |
| `endpoint:read` | View endpoint inventory & metadata | **GLOBAL** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** |
| `endpoint:update` | Edit endpoint tags, group, location | NONE | **ORG** | **ORG** | NONE | NONE | NONE | NONE |
| `endpoint:retire` | Retire/decommission an endpoint | NONE | **ORG** | NONE | NONE | NONE | NONE | NONE |
| `endpoint:revoke_cert`| Revoke agent identity certificate | **GLOBAL** | **ORG** | **ORG** | NONE | NONE | NONE | NONE |
| `health:view` | View agent CPU, RAM, queue health | **GLOBAL** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | NONE |

---

### 3.3 Security Policies & Configuration

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `policy:read` | View security policies & history | **GLOBAL** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | NONE |
| `policy:create` | Author draft security policy | NONE | NONE | **ORG** | NONE | NONE | NONE | NONE |
| `policy:sign` | Cryptographically sign policy | NONE | NONE | **ORG** | NONE | NONE | NONE | NONE |
| `policy:assign` | Assign policy to Org/Site/Group | NONE | **ORG** | **ORG** | NONE | NONE | NONE | NONE |
| `policy:rollback` | Rollback policy to prior version | NONE | NONE | **ORG** | NONE | NONE | NONE | NONE |
| `policy:compliance`| View policy drift & compliance | **GLOBAL** | **ORG** | **ORG** | NONE | NONE | **ORG** | **ORG** |

---

### 3.4 Detection, Alerts & Incidents

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `alert:read` | View alerts & triggered evidence | NONE | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | NONE |
| `alert:triage` | Assign, prioritize, add triage notes | NONE | NONE | NONE | **ORG** | **ORG** | NONE | NONE |
| `alert:false_positive`| Mark alert as false positive | NONE | NONE | **ORG** | **ORG** | **ORG** | NONE | NONE |
| `incident:create` | Create incident case from alerts | NONE | NONE | NONE | **ORG** | **ORG** | NONE | NONE |
| `incident:read` | View incident timeline & details | NONE | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** |
| `incident:close` | Document resolution & close case | NONE | NONE | NONE | NONE | **ORG** | NONE | NONE |
| `rule:manage` | Enable, disable, edit Sigma/Ariba rules| NONE | NONE | **ORG** | NONE | NONE | NONE | NONE |

---

### 3.5 Guarded Incident Response (Privileged Actions)

> [!CAUTION]
> All response actions are denied by default and require strict server-side parameter re-validation (PID, Path, Hash, Expiry).

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `response:view` | View response action queue & logs | NONE | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | NONE |
| `response:kill_process`| Terminate running process on host | NONE | NONE | NONE | NONE | **ORG [A]** | NONE | NONE |
| `response:quarantine` | Move file to encrypted quarantine | NONE | NONE | NONE | NONE | **ORG [A]** | NONE | NONE |
| `response:restore` | Restore file from quarantine | NONE | NONE | **ORG** | NONE | **ORG [A]** | NONE | NONE |
| `response:isolate` | Network-isolate host (EPP open) | NONE | NONE | NONE | NONE | **ORG [A]** | NONE | NONE |
| `response:release` | Release host from network isolation | NONE | NONE | NONE | NONE | **ORG [A]** | NONE | NONE |
| `response:scan` | Trigger on-demand endpoint AV scan | NONE | NONE | **ORG** | **ORG** | **ORG** | NONE | NONE |
| `response:forensics`| Request forensic triage package | NONE | NONE | NONE | NONE | **ORG [A]** | NONE | NONE |

---

### 3.6 Threat Intelligence & Vulnerabilities

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `ioc:read` | View IOC indicators & reputation | **GLOBAL** | **ORG** | **ORG** | **ORG** | **ORG** | NONE | NONE |
| `ioc:manage` | Add, edit, expire IOC hashes/IPs | **GLOBAL** | NONE | **ORG** | NONE | **ORG** | NONE | NONE |
| `yara:manage` | Author & compile custom YARA rules | NONE | NONE | **ORG** | NONE | NONE | NONE | NONE |
| `feed:sync` | Trigger MISP / threat feed sync | **GLOBAL** | NONE | **ORG** | NONE | NONE | NONE | NONE |
| `vuln:read` | View software inventory & CVEs | NONE | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** | **ORG** |

---

### 3.7 Audit & Reporting

| Permission Code | Permission Description | Platform Admin | Org Admin | Security Admin | SOC L1 | SOC L2 | Auditor | Viewer |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `audit:read` | View tamper-evident audit logs | **GLOBAL** | **ORG** | **ORG** | NONE | NONE | **ORG** | NONE |
| `audit:export` | Export signed CSV/JSON audit trails | **GLOBAL** | **ORG** | NONE | NONE | NONE | **ORG** | NONE |
| `report:executive` | View/download executive PDF summary| **GLOBAL** | **ORG** | **ORG** | NONE | NONE | **ORG** | **ORG** |
| `report:compliance`| Generate compliance packs (ISO/SOC2)| **GLOBAL** | **ORG** | **ORG** | NONE | NONE | **ORG** | NONE |

---

## 4. Server-Side Enforcement Architecture (FastAPI Blueprint)

Enforcement happens via dependency injection in FastAPI. Client-supplied IDs are never trusted.

```python
# Conceptual FastAPI enforcement pattern
from fastapi import Depends, HTTPException, status
from typing import Annotated

async def require_permission(permission_code: str):
    async def permission_checker(current_user: Annotated[User, Depends(get_current_user)]):
        if not current_user.has_permission(permission_code):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Permission denied: '{permission_code}' required"
            )
        return current_user
    return permission_checker

# Example protected route:
@router.post("/api/v1/response/actions")
async def execute_response_action(
    action_request: ResponseActionRequest,
    current_user: User = Depends(require_permission("response:isolate")),
    db: AsyncSession = Depends(get_db)
):
    # Tenant verification
    endpoint = await db.get(Endpoint, action_request.endpoint_id)
    if not endpoint or endpoint.organization_id != current_user.organization_id:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Endpoint not found")
    
    # Process guarded action...
```

---

## 5. Security Acceptance Tests for RBAC

1. **Cross-Tenant IDOR Test:**
   - User in Org A attempts to request `/api/v1/endpoints/{org_b_endpoint_id}`.
   - Result: Must return `404 Not Found` (never `200` and never leak existence).
2. **Privilege Escalation Test:**
   - SOC Analyst L1 attempts to send `POST /api/v1/response/actions` (`action_type: kill_process`).
   - Result: Server must reject with `403 Forbidden` and log an unauthorized privileged attempt in `audit_logs`.
3. **Unauthenticated Request Test:**
   - Any API request without valid Bearer JWT or certificate credentials must return `401 Unauthorized`.
4. **Platform Admin Data Isolation Test:**
   - Platform Admin cannot read customer raw telemetry or execute customer response actions without triggering an explicit Break-Glass Audit Event.
