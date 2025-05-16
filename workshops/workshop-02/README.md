# 🚀 Welcome to Argo CD

This guide provides step-by-step instructions to install **Argo CD**, a declarative GitOps continuous delivery tool for Kubernetes.


## 🛠️ Prerequisites

- Kubernetes Cluster
- `kubectl` CLI
- `argocd` CLI *(optional)*


## 📦 Installation Steps

### 1. Create the Argo CD Namespace

```bash
kubectl create namespace argocd
```

### 2. Install Argo CD Using the Official Manifests

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Verify the Argo CD Pods

```
kubectl get pods -n argocd
```

### 4. Expose the Argo CD API Server (for Local Access)

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 5. Get the Initial Admin Password
- **kubectl**
     ```
     kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
     ```

- **argocd** CLI:
     ```
     argocd admin initial-password -n argocd
     ```

### 6. Add SSH Credentials for Private Git Repository Access

To allow Argo CD to access your **private Git repository via SSH**, follow these steps:

   - **Generate an SSH Key:**
     Run the following command to generate a new SSH key pair:

     ```bash
     ssh-keygen -t ed25519 -C "<objective>" -f argocd_ssh_key
     ```

     This will create:
        - `argocd_ssh_key` — the **private key**
        - `argocd_ssh_key.pub` — the **public key**

          > **Note:**  
          >`-f` option in the `ssh-keygen` command specifies the **filename** for the generated key files.         
          >`-C` option allows you to add a **comment** to the key, which is useful for identifying the key later (e.g., `"<objective>"`).

   - **Add the SSH Public Key to Your Git Repository:**
     Add `argocd_ssh_key.pub` as a **Deploy Key** to your Git repository:

     
- **Add the SSH Public Key to Your Git Repository:**
     Download the following file in your repo:
     [argocd-repo-creds.yaml](./argocd-repo-creds.yaml)
     Paste the content of `argocd_ssh_key` (private key) into the `sshPrivateKey:` field:
     
    ```yaml
    sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...your key here...
    -----END OPENSSH PRIVATE KEY-----
    ```
    
    - **Apply the Secret to the Cluster:**
    ```bash
    kubectl apply -f argocd-repo-creds.yaml
    ```

# ✅ Step-by-Step: Expose Argo CD via Ingress

### 1. Install an Ingress Controller (e.g., NGINX Ingress)
If you're running a **bare-metal or self-hosted cluster**, install the NGINX Ingress Controller using:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/baremetal/deploy.yaml
```
### 2. Change Service Type to LoadBalancer
By default, the **ingress-nginx-controller** service is of type **ClusterIP.**
Update it to ***LoadBalancer** so traffic can reach it from outside:

```
kubectl edit svc ingress-nginx-controller -n ingress-nginx
```

change this line:

```yaml
spec:
  type: LoadBalancer
```

### 3. Create Ingress Resource for Argo CD
```yaml
sshPrivateKey: |
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd
  namespace: argocd
spec:
  ingressClassName: nginx
  rules:
  - host: jojo.saritrat   #  <-------  Replace with your domain --------
    http:
      paths:
      - backend:
          service:
            name: argocd-server
            port:
              number: 80
        path: /
        pathType: Prefix
```

Apply the resource:

```
kubectl apply -f argocd-ingress.yaml
```

### 4. Map Domain to Ingress Node IP
Get the IP of your Kubernetes node:
```
kubectl get nodes -o wide
```
### Local (edit hosts file)
- macOS / Linux

     1. Open Terminal.
     2. Run the command to edit `/etc/hosts` with sudo privileges:

     ```bash
     sudo vi /etc/hosts
     ```
- Windows

     1. Open file as **Administrator** (Right-click → Run as administrator).
     ```
     C:\Windows\System32\drivers\etc\hosts
     ```

Add a new line with your Node IP and domain:
```
<YOUR_NODE_IP> argocd.example.com
```

### DNS Records on Cloud


### 1. Map LoadBalancer IP to DNS

- If your LoadBalancer provides a **static IPv4 address**, create an **A record** pointing your domain/subdomain to this IPv4 address.
  
- If your LoadBalancer provides a **static IPv6 address**, create an **AAAA record** pointing your domain/subdomain to this IPv6 address.

---

### 2. Map LoadBalancer Hostname (DDNS) to DNS

- Some cloud providers (like AWS ELB, Google Cloud Load Balancer) provide a **dynamic hostname** (e.g., `abcd1234.elb.amazonaws.com`).

- In this case, you **cannot create A/AAAA records** directly with IPs because the IP may change.

- Instead, create a **CNAME record** for your domain/subdomain that points to the LoadBalancer's hostname.

---

### Example DNS Records

| Record Type | Hostname (Domain/Subdomain) | Points to                     | Use case                      |
|-------------|-----------------------------|------------------------------|------------------------------|
| A           | argocd.example.com           | 192.0.2.123                  | Static IPv4 LoadBalancer IP   |
| AAAA        | argocd.example.com           | 2001:0db8:85a3::8a2e:0370:7334 | Static IPv6 LoadBalancer IP   |
| CNAME       | argocd.example.com           | abcd1234.elb.amazonaws.com   | Dynamic hostname LoadBalancer |

---

### 5. Configure Argo CD to Run Without TLS (Optional for Local Ingress)

To allow Argo CD to be accessed via Ingress over HTTP (without HTTPS/TLS), you can disable TLS on the `argocd-server` by setting:

Go to the file in your repo: [argocd-cmd-params-cm.yaml](./02_argocd-cmd-params-cm.yaml) Save the above YAML content into a file named **argocd-cmd-params-cm.yaml.**


- Apply the ConfigMap to your Kubernetes cluster:
    ```
    kubectl apply -f argocd-cmd-params-cm.yaml
    ```
- Restart the Argo CD server deployment to apply the changes:
    ```
    kubectl rollout restart deployment argocd-server -n argocd
    ```

### 6. Access Argo CD

You can now visit:
```
https://jojo.saritrat
```

### 🎉 Welcome to Argo CD

![Welcome to Argo CD](./assets/argocd-portal.jpeg)