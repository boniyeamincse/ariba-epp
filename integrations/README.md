# Integrations

Adapters to existing security tools and the Ariba SOC ecosystem. These are
integrations, not substitutes for the EPP control plane.

| Directory | Purpose |
|---|---|
| `wazuh/` | Forward normalized EPP events to Ariba SOC/Wazuh and receive response requests |
| `opensearch/` | Event search, analytics and investigation timeline |
| `misp/` | Import/export indicators with lifecycle tracking |
| `clamav/` | Malware scanning adapter |
| `microsoft-defender/` | Defender status/scan integration without disabling it |

Each integration needs a versioned contract, failure handling, retry behavior and
an owner. Confirm interface versions and licensing before coupling.
