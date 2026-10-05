# Helm Package Management & ArgoCD GitOps Continuous Delivery

Packaging Kubernetes applications and automating reconciliation using Git as the single source of truth.

---

## 1. Helm Chart Directory Structure

```text
my-app/
├── Chart.yaml          # Metadata: name, version, appVersion, dependencies
├── values.yaml         # Default configuration values
├── templates/          # Go template manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl    # Reusable template snippets
└── tests/              # Helm test verification scripts
```

### Essential Helm Commands
```bash
# Package and lint a chart
helm lint ./my-app
helm package ./my-app

# Dry-run template generation
helm template my-release ./my-app --values env-prod.yaml

# Upgrade or install atomically with automatic rollback on failure
helm upgrade --install my-release ./my-app \
  --namespace prod \
  --create-namespace \
  --values prod-values.yaml \
  --atomic \
  --timeout 5m
```

---

## 2. ArgoCD Application Manifest (GitOps)

ArgoCD continuously monitors the Git repository and synchronizes state to the Kubernetes cluster:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cloud-service-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: 'https://github.com/yahyaoncloud/devops-notes.git'
    targetRevision: HEAD
    path: k8s/manifests/production
    helm:
      valueFiles:
        - values-production.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true     # Automatically delete resources removed from Git
      selfHeal: true  # Revert manual changes made directly in cluster
    syncOptions:
      - CreateNamespace=true
```
