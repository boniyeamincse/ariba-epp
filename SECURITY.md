# Security Policy

## Scope

This policy covers the Ariba EPP repository and, once deployed, the Ariba EPP
management platform and endpoint agents. It does not authorize testing of any
third-party service or of endpoints you do not own.

## Reporting a vulnerability

Do not open a public issue for a suspected vulnerability.

- Preferred: use the private security-reporting channel designated by the Ariba
  security team (to be confirmed before the repository is shared externally).
- Include: affected component and version/commit, impact, reproduction steps,
  and any proof-of-concept needed to validate the report.
- Do not include real credentials, private keys, customer data or live malware.
  Redact secrets and personal data from screenshots and logs.

Until a private channel is published, contact the repository owner directly.
This contact detail is a placeholder and must be replaced before external use.

## Safe testing rules

- Test only on owned or explicitly authorized endpoints and an isolated lab.
- Begin in observe-only mode. Do not run process termination, file quarantine,
  endpoint isolation or deletion against production endpoints.
- Use approved benign test artifacts (for example EICAR). Never commit or ship
  live malware.
- Do not attempt to disable, bypass or degrade existing antivirus or host
  protection as part of testing.

## Handling of sensitive material

- Agent credentials, enrollment tokens and certificates are secrets. Store them
  in an approved secret store; never commit them.
- Quarantine objects and forensic evidence require restricted access, integrity
  metadata and a defined retention period.
- Telemetry must follow the approved data-collection, retention and privacy
  policy, including tenant and data-residency requirements.

## Response and disclosure

The Ariba security team will acknowledge a report, assess severity, coordinate a
fix and agree a disclosure timeline with the reporter. Please allow reasonable
time for remediation before any public disclosure.

## Out of scope

- Reports based only on missing best-practice headers with no demonstrated impact.
- Automated scanner output without a validated, reproducible finding.
- Issues in dependencies that have no exploitable path in this project
  (report them upstream as well).
