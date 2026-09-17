# Secret handling and incident recovery

Never commit plaintext Kubernetes Secrets. This repository intentionally only
contains non-applied `*.yaml.example` templates. Create runtime secrets through
an external secret manager, or configure Flux SOPS decryption and commit only
SOPS-encrypted manifests.

The workloads require these pre-existing Secrets:

- `api/postgres-secret`: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`,
  and `CONNECTION_STRING`.
- `monitoring/grafana-admin-credentials`: `admin-user` and `admin-password`.

For the January 2026 exposure, rotate both passwords before deploying these
manifests. Changing the PostgreSQL container environment does not change the
password of an already initialized database; use `ALTER ROLE app WITH PASSWORD
'new-value';`, update `postgres-secret`, and restart the API workload. Change
the Grafana admin password in Grafana (or with `grafana-cli`) and then update
its Secret.

After rotation, purge the old values from every Git ref with `git-filter-repo`,
force-push all rewritten branches and tags, expire any mirrors/caches, and have
all collaborators re-clone. Treat history rewriting as cleanup only: exposed
credentials remain compromised even after the commits disappear.
