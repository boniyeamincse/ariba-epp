# Endpoint Agent

Planned cross-platform agent (Go candidate) for Windows and Linux.

| Directory | Responsibility |
|---|---|
| `common/` | Shared configuration, logging, versioning and normalization helpers |
| `windows/` | Windows-specific collectors, service packaging and OS integration |
| `linux/` | Linux-specific collectors, service packaging and OS integration |
| `collectors/` | Process, file, network and security event collectors |
| `transport/` | Authenticated heartbeat, telemetry upload and bounded offline queue |
| `policy/` | Policy retrieval, signature verification and acknowledgement |
| `response/` | Authorized command handling; denied by default until validated |

Non-negotiable requirements: unique certificate-based identity, short-lived
one-time enrollment tokens, signed binaries and policies, bounded local queue,
tested upgrade rollback, and predictable offline behavior. Do not enable
disruptive response actions in the first milestone.
