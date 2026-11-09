# Ariba EPP — Endpoint Agent Specification

**Status:** Draft; Windows/Linux supported versions and telemetry policy require owner approval.  
**Candidate core:** Go with OS-specific collectors. No agent code is specified as implemented.

## 1. Purpose and operating modes

The agent enrolls a company-authorized endpoint, reports bounded health/inventory/security events, fetches authenticated signed policy, and later may process a narrow allowlisted response command. It must not be a remote shell or general-purpose arbitrary command runner.

Modes:

1. **Unenrolled:** no telemetry submission or response; only safe local enrollment flow.
2. **Enrolled observe-only (initial):** heartbeat, approved inventory and selected telemetry; response commands disabled.
3. **Policy-managed:** applies only verified compatible signed policy; reports acknowledgement.
4. **Guarded response (future):** only approved action types, after separate security/owner gate.
5. **Degraded/offline:** existing OS protection remains active; bounded queue and explicit health status.

## 2. Support matrix (must be approved before release)

| Platform | Candidate signal APIs | Packaging candidate | Status |
|---|---|---|---|
| Windows 10/11 and Windows Server versions | ETW, Windows Event Log, supported OS APIs | Signed MSI/service | Exact build matrix To confirm |
| Linux distributions/kernels | auditd/osquery and OS-native notifications | Signed DEB/RPM/system service | Distro/kernel matrix To confirm |

macOS/mobile are out of initial platform scope. Verify API privilege needs, distribution support, upgrade behavior and resource impact per version.

## 3. Agent lifecycle

1. Install signed package through an authorized administrator.
2. Enroll using short-lived, single-use authorization scoped to intended organization/site.
3. Obtain unique endpoint identity/certificate; protect private key and support revocation/renewal.
4. Start service in observe-only mode; report heartbeat and approved inventory.
5. Fetch signed, versioned policy; verify signature, target scope, schema and anti-rollback rules.
6. Collect only enabled/approved signals and send bounded batches with idempotent event IDs.
7. On transient disconnect, queue within configured disk/age limits, back off and reconnect.
8. On upgrade, verify signed package, stage/health-check and rollback safely if failed.
9. On uninstall/revoke, stop authenticated communication, revoke identity and follow defined local data cleanup policy.

## 4. Candidate agent components

| Module | Responsibility |
|---|---|
| Config | Parse/validate protected local configuration |
| Enrollment/identity | One-time enrollment, certificate identity, rotation/revocation |
| Heartbeat | Health, version, policy state and last contact |
| Collectors | OS-specific process/file/network/security metadata |
| Normalizer | Map collector output to versioned Ariba event schema |
| Queue | Bounded local queue, dedupe, retry, backpressure and expiry |
| Transport | Authenticated encrypted communication and response validation |
| Policy | Signature/version/target verification and acknowledgement |
| Response | Disabled initially; allowlisted, signed, expiring, audited commands later |
| Updater | Signed update check, staged activation and rollback |
| Logging | Minimal structured logs; no credentials or unnecessary personal content |

Candidate tools: Go core; Windows ETW/Event Log; Linux auditd/osquery; native file notification APIs; HTTPS initially; Protobuf candidate; SQLite queue candidate; MSI/DEB/RPM packages. Validate compatibility before selection.

## 5. Telemetry contract

Each event carries schema version, unique event ID, event type, event timestamp, producer/agent version, platform and typed payload. Endpoint and tenant identity derive from authenticated agent context server-side. The data-field allowlist, collection toggles, user visibility and retention are **To confirm**.

Potential classes: system health, process lifecycle, selected file metadata, selected network connection metadata and policy/update events. Do not collect file contents, keystrokes, browser content or unrelated user activity. Command-line, user identity and paths are sensitive and need explicit purpose/privacy approval.

## 6. Offline, retry and resource behavior

- Queue has maximum bytes/events/age; behavior when full is defined and observable (drop/deny policy must be approved per event class).
- Retry uses bounded exponential backoff with jitter and server backpressure response.
- Event IDs support deduplication; timestamps distinguish endpoint occurrence from server receipt.
- Commands expire; never execute stale queued response commands after reconnect.
- Local storage permissions are restrictive; encrypt sensitive queue contents if present.
- CPU, memory, disk and scan limits are configurable and measured on representative hardware. The 1–3% normal CPU figure is only an initial target, not a guarantee.
- Agent crash/restart does not lose identity or repeatedly enroll duplicate endpoints.
- Server unavailable does not disable OS-native antivirus/firewall protections.

## 7. Policy and update verification

Agent validates signer trust, signature, policy/update version, endpoint scope, expiry and supported schema before application. Reject tampered, unsigned, incompatible, wrong-scope or unauthorized rollback artifacts. Keep last-known-good state when safe, report failure reason and support authorized recovery. Signing-key rotation, trust bootstrap and emergency revocation require a documented plan before production.

## 8. Future response command guardrails

Response is disabled for Foundation v0.1. Later commands must:

- Be explicit allowlisted action types, never arbitrary shell/script execution.
- Carry unique command ID, endpoint binding, target identity, actor/approval reference, reason, issue time and expiry.
- Be authenticated/signed, replay-resistant and idempotent where possible.
- Revalidate process/file/network target immediately before action.
- Return accepted/rejected/running/succeeded/failed/expired state and safe result details.
- Preserve secure recovery communication during endpoint isolation.
- Emit audit evidence and have a tested undo/release path where technically possible.

## 9. Agent tests and release gates

- Unit: config validation, normalization, queue limits, event IDs, state machine.
- Integration: certificate/enrollment, server errors, policy signatures, retries, revocation and schema compatibility.
- Platform: supported OS version collectors, permissions, service lifecycle, upgrade/uninstall, resource budget.
- Security: tampered/expired/replayed command rejection, certificate revocation, untrusted policy/update rejection and least privilege.
- Reliability: offline, queue-full, clock skew, restart, server outage, partial update and recovery.
- Package: signing verification, clean install/upgrade/uninstall and rollback in an isolated lab.

No production endpoint rollout before support matrix, signed packaging, rollback, resource tests, privacy approval and incident/recovery procedures pass review.