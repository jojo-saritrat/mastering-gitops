# 🎧 Argo CD Notifications - Discord & Email

This configuration enables **Argo CD Notifications** to send updates to **Discord** and **Email** when specific events such as sync success occur.

---

## 📢 Supported Channels

### 🔊 Discord
- Uses a Webhook URL
- Sends message when application sync succeeds

### 📧 Email (optional)
- Can be added using SMTP configuration (not included in this example, see below for instructions)

---

## 📂 Example ConfigMap: `argocd-notifications-cm`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd

data:
  service.webhook.discord: |
    url: https://discord.com/api/webhooks/<your_channel_webhook>
    headers:
      - name: Content-Type
        value: application/json

  template.app-sync-succeeded: |
    webhook:
      discord:
        method: POST
        body: |
          {
            "username": "{{.app.metadata.name}}",
            "content": "Application {{.app.metadata.name}} is now running new version of deployments manifests.\nURL: {{.context.argocdUrl}}/applications/{{.app.metadata.name}}\nContext: continuous-delivery/{{.app.metadata.name}}"
          }

  trigger.on-sync-succeeded: |
    - description: Application syncing has succeeded
      send:
        - app-sync-succeeded
      when: app.status.operationState.phase in ['Succeeded']
```

---

## ✅ How to Apply

1. Save the configuration to a file: `argocd-notifications-cm.yaml`

2. Apply it with `kubectl`:

```bash
kubectl apply -f argocd-notifications-cm.yaml
```

3. Restart the notifications controller (if needed):

```bash
kubectl rollout restart deployment argocd-notifications-controller -n argocd
```

---

## 🚀 Add Email Notification (Optional)

To use SMTP for email alerts:

1. Add this to a **Secret**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: argocd-notifications-secret
  namespace: argocd
stringData:
  email.username: your@email.com
  email.password: yourpassword
```

2. Add this to the ConfigMap:

```yaml
data:
  service.email.default: |
    username: $email.username
    password: $email.password
    host: smtp.yourmail.com
    port: 587
    from: no-reply@yourmail.com
    insecure: true
```

3. Update triggers and templates to include `email.default` as a receiver.

---

## 📉 Verify

Check logs of the notifications controller:

```bash
kubectl logs deployment/argocd-notifications-controller -n argocd
```

Or monitor messages in Discord / Email inbox.
