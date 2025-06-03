# ArgoCD ApplicationSet

This repository uses an **Argo CD ApplicationSet** to manage multiple `wallet` microservice deployments across environments (e.g., `dev`, `sit`) using a single declarative config.

---

## 🔧 ApplicationSet Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: wallet
spec:
  generators:
    - list:
        elements:
          - cluster: dev
            url: 'https://kubernetes.default.svc'
          - cluster: sit
            url: 'https://kubernetes.default.svc'
  template:
    metadata:
      name: '{{cluster}}-wallet'
    spec:
      project: default
      source:
        repoURL: 'https://github.com/jojo-saritrat/nonprod-k8s-manifests.git'
        targetRevision: HEAD
        path: 'nonprod/{{cluster}}/wallet'
      destination:
        server: '{{url}}'
        namespace: wallet
```

---

## 📖 What is ApplicationSet?

Argo CD `ApplicationSet` allows you to generate multiple `Application` resources dynamically based on a generator (e.g., list, git, matrix). This is great for deploying the same app across multiple clusters or environments.

### 🏢 Benefits
- DRY approach to managing deployments across environments
- Git-driven dynamic generation
- Scales better than writing separate Application manifests manually

---

## 📚 Sync Behavior and Options

If you want each generated Application to auto-sync and manage its namespace:

```yaml
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Add this block inside the `spec` of the `template` section in your ApplicationSet.

---

## 🔨 How to Apply ApplicationSet

1. Save the above YAML into a file, e.g. `argocd-applicationset.yaml`

2. Apply using kubectl:

```bash
kubectl apply -f argocd-applicationset.yaml
```