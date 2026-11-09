# Infrastructure

Deployment, proxy, monitoring and recovery assets.

| Directory | Purpose |
|---|---|
| `docker/` | Docker Compose for local development with safe local-only defaults |
| `kubernetes/` | Only if measured requirements justify Kubernetes |
| `nginx/` | TLS reverse proxy configuration |
| `monitoring/` | Health, metrics and alerting configuration |
| `backup/` | Backup, restore and verification procedures |

Docker Compose is a development and small-pilot option, not a production
security or high-availability guarantee. Production needs segmented services,
least-privilege identities, encrypted secrets, tested backups and a hardening
review. Do not commit certificates or keys.
