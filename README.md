# Argo CD app-of-apps

This repository uses the Argo CD app-of-apps pattern. The root Application
discovers the child Applications in `apps/`, and each child deploys a chart
from `charts/`.

## Demo application

`demo-app` deploys a small nginx web page into the `demo` namespace.

```bash
kubectl apply -f app-of-apps.yaml
kubectl -n demo port-forward service/demo-app 8081:80
```

Open <http://localhost:8081> after the root and child Applications are synced.
