# Argo CD app-of-apps

This repository uses the Argo CD **app-of-apps** pattern. The root Application
discovers the child Applications in `apps/`, and each child deploys a chart
from `charts/`.

## Structure

```text
app-of-apps.yaml          # Root ArgoCD Application (apply once)
apps/
  cert-manager.yaml        # Child Application
  demo-app.yaml            # Child Application
  external-secrets.yaml    # Child Application (ESO Operator)
  nginx-ingress.yaml       # Child Application
charts/
  cert-manager/            # Helm chart
  demo-app/                # Helm chart (Node.js + PostgreSQL + Vault integration)
  nginx-ingress/           # Helm chart
```

## Quick start

1. **Create the Secret** — choose **one** of:

   **Option A – Vault + External Secrets Operator (recommended):**

   a. Create prerequisite Kubernetes Secrets (in `demo-app` namespace):

   ```bash
   kubectl create namespace demo-app --dry-run=client -o yaml | kubectl apply -f -

   # CA Cert Secret (for self-signed TLS)
   kubectl -n demo-app create secret generic vault-ca-cert \
     --from-file=ca.crt=/path/to/vault-ca.pem

   # AppRole Credentials Secret
   kubectl -n demo-app create secret generic vault-approle \
     --from-literal=role-id='<YOUR_VAULT_ROLE_ID>' \
     --from-literal=secret-id='<YOUR_VAULT_SECRET_ID>'
   ```

   b. Store your DB secret in Vault (KV v2 path: `secret/data/demo-app/db`):

   ```bash
   vault kv put secret/demo-app/db \
     POSTGRES_DB=demo \
     POSTGRES_USER=demo \
     POSTGRES_PASSWORD='<random-password>' \
     DATABASE_URL='postgresql://demo:<url-encoded-password>@demo-app-postgres:5432/demo'
   ```

   c. Set `vault.enabled=true` and your Vault server `address` in `charts/demo-app/values-demo.yaml`.

   **Option B – Manual K8s Secret (fallback):**

   ```bash
   kubectl create namespace demo-app --dry-run=client -o yaml | kubectl apply -f -
   kubectl -n demo-app create secret generic demo-app-db \
     --from-literal=POSTGRES_DB=demo \
     --from-literal=POSTGRES_USER=demo \
     --from-literal=POSTGRES_PASSWORD='<random-password>' \
     --from-literal=DATABASE_URL='postgresql://demo:<url-encoded-password>@demo-app-postgres:5432/demo'
   ```

2. **Apply the root Application:**

   ```bash
   kubectl apply -f app-of-apps.yaml
   ```

3. **Verify sync:**

   ```bash
   argocd app get app-of-apps
   argocd app get external-secrets
   argocd app get demo-app
   ```

4. **Access the demo app:**

   ```bash
   kubectl -n demo-app port-forward service/demo-app 8081:80
   ```

   Open <http://localhost:8081>.

## Environment overrides

The `demo-app` child Application uses two value files layered in order:

- `charts/demo-app/values.yaml` — sensible defaults (local dev)
- `charts/demo-app/values-demo.yaml` — k3d environment overrides (storage class, ingress class, vault integration)

To add a new environment, create a new `values-<env>.yaml` file and reference it
in a new Application manifest under `apps/`.
