# ArgoCD Sync

## 🔧 Argo CD Application Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart
  namespace: argocd
spec:
  project: default
  source:
    repoURL: git@github.com:jojo-saritrat/nonprod-k8s-manifests.git
    targetRevision: develop
    path: dev/cart
  destination:
    server: https://kubernetes.default.svc
    namespace: cart
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 🔄 How to Use `argocd.argoproj.io/sync-wave`

In Argo CD, you can control the order in which Kubernetes resources are applied using the `argocd.argoproj.io/sync-wave` annotation.

### ✅ What is it?

This annotation tells Argo CD to apply resources in waves during a sync. Lower wave numbers are applied first.

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: '1'
```

### 🧠 Common Use Cases

| Resource        | Sync Wave | Reason                                |
|----------------|-----------|----------------------------------------|
| Namespace       | `-1`      | Must exist before other resources      |
| ConfigMap/Secret| `0`       | Needed by Deployments                  |
| Deployment      | `1`       | Depends on other resources             |
| Job/Post-Deploy | `2+`      | Run after app is deployed              |

---

### 📚 Where to Add `sync-wave`

You should add the `argocd.argoproj.io/sync-wave` annotation inside the `metadata.annotations` block of each Kubernetes manifest in your Git repo.

For example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: cart
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  replicas: 1
  ...
```

This must be done **in the actual manifest files** (e.g., YAML files under `dev/cart/`) — not in the Argo CD `Application` itself.

---

### 🔨 How to Apply the Changes

1. **Commit and Push** the updated YAML files to your Git repository (with `sync-wave` annotations added).

2. **Argo CD will detect the changes** automatically and sync them according to wave order if Auto-Sync is enabled.

3. Or you can trigger manual sync:

```bash
argocd app sync cart
```

4. Check sync status:

```bash
argocd app get cart
```

---
## 🔄 How to Use `argocd.argoproj.io/automated-sync`


### 📚 Where to Add `automated sync`

You enable `automated sync` directly in your Argo CD `Application` manifest inside the `syncPolicy` block:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

- `prune: true` ensures that Argo CD deletes resources removed from Git.
- `selfHeal: true` lets Argo CD revert out-of-sync resources automatically.

> Note: If you set `enabled: false`, automated sync won't trigger even if changes are detected.

---

### 🔨 How to Apply the Changes (Automated Sync)

1. Edit the `Application` YAML to include or update the `syncPolicy.automated` section.

2. Apply the updated YAML to the cluster:

```bash
kubectl apply -f application.yaml
```

3. Argo CD will automatically sync any changes in your Git repo to the cluster.

Or via the Argo CD UI, under the **App Details → Sync Status** section.

---
## 🔄 How to Use `argocd.argoproj.io/sync-options`

### 📚 Where to Add `syncOptions`

You define `syncOptions` under the `syncPolicy` field of your Argo CD `Application` manifest. These options allow you to customize sync behavior.

Example:

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
```

- `CreateNamespace=true`: Argo CD will automatically create the destination namespace if it does not exist.

> You can include multiple options in the list, depending on your use case. Refer to the [Argo CD docs](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/) for more.

---

### 🔨 How to Apply the Changes (Sync Options)

1. Edit your Argo CD `Application` YAML to include or update the `syncOptions` field.

2. Apply the changes to the cluster:

```bash
kubectl apply -f application.yaml
```

3. Argo CD will use the sync options on the next sync operation.

4. Check the sync status:

```bash
argocd app get cart
```

Or verify behavior from the Argo CD UI under your Application's Sync Details.