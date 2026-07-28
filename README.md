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
  nginx-ingress.yaml       # Child Application
charts/
  cert-manager/            # Helm chart
  demo-app/                # Helm chart (Node.js + PostgreSQL)
  nginx-ingress/           # Helm chart
```

## Quick start

1. **Create the Secret** (must exist before the first sync):

   ```bash
   kubectl create namespace demo --dry-run=client -o yaml | kubectl apply -f -
   kubectl -n demo create secret generic demo-stack-db \
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
   argocd app get demo-app
   ```

4. **Access the demo app:**

   ```bash
   kubectl -n demo port-forward service/demo-app 8081:80
   ```

   Open <http://localhost:8081>.

## Environment overrides

The `demo-app` child Application uses two value files layered in order:

- `charts/demo-app/values.yaml` — sensible defaults (local dev)
- `charts/demo-app/values-demo.yaml` — k3d environment overrides (storage class, ingress class)

To add a new environment, create a new `values-<env>.yaml` file and reference it
in a new Application manifest under `apps/`.
