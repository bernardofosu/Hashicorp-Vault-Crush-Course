# 🔐 What is HashiCorp Vault?

HashiCorp Vault is a tool designed to **secure, store, and tightly control access** to tokens, passwords, certificates, API keys, and other sensitive data in modern computing. It provides a **unified interface** to any secret while ensuring **tight access control** and **detailed audit logging**.

Vault was built to address the complex task of managing secrets and protecting sensitive data in a **dynamic, distributed, and multi-cloud environment**. It supports multiple types of backends for storing secrets, including **in-memory, file system, and cloud storage services**.

## ⚡ Vault's Main Features

- 🔑 **Secret Management** - A centralized location for storing and accessing secrets securely.
- 🏗 **Dynamic Secrets** - Generates on-demand secrets for AWS, SQL databases, and more with defined TTL (Time-to-Live).
- 🔒 **Data Encryption** - Encrypts and decrypts data without storing it, allowing security teams to define encryption parameters.
- ⏳ **Leasing & Renewal** - Secrets can have a specific lifetime and automatically revoked when expired.
- ❌ **Revocation** - Revoke individual secrets or entire trees of secrets based on users or types.
- 📜 **Auditing** - Built-in auditing system to track interactions with Vault for security and compliance.

---

# 🚀 HashiCorp Vault Demo (Kubernetes)

## 1️⃣ Create the Vault Namespace
```bash
kubectl create namespace vault
```

## 2️⃣ Add the HashiCorp Helm Repository
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

## 3️⃣ Install HashiCorp Vault using Helm
```bash
helm install vault hashicorp/vault \
  --set="server.dev.enabled=true" \
  --set="ui.enabled=true" \
  --set="ui.serviceType=NodePort" \
  --namespace vault
```

## 4️⃣ Enter the Vault Pod to Configure Vault with Kubernetes
```bash
kubectl exec -it vault-0 -n vault -- /bin/sh
```

## 5️⃣ Create a Policy for Reading Secrets (read-policy.hcl)
```bash
cat <<EOF > /home/vault/read-policy.hcl
path "secret*" {
  capabilities = ["read"]
}
EOF
```

## 6️⃣ Write the Policy to Vault
```bash
vault policy write read-policy /home/vault/read-policy.hcl
```

## 7️⃣ Enable Kubernetes Authentication in Vault
```bash
vault auth enable kubernetes
```

## 8️⃣ Configure Vault to Communicate with the Kubernetes API Server
```bash
vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://${KUBERNETES_PORT_443_TCP_ADDR}:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

## 9️⃣ Create a Role (vault-role) to Bind the Policy to a Kubernetes Service Account
```bash
vault write auth/kubernetes/role/vault-role \
   bound_service_account_names=vault-serviceaccount \
   bound_service_account_namespaces=vault \
   policies=read-policy \
   ttl=1h
```

## 🔑 Creating & Managing Secrets

### 🔹 Create a Secret
```bash
vault kv put secret/clisecret token=secretcreatedbycli
```

### 🔍 Verify if the Secret was Created
```bash
vault kv list secret
```

### 📁 Check if the Secret is Injected into a Pod
```bash
kubectl exec -it <pod name> -n vault -- ls /vault/secrets/
kubectl exec -it <pod name> -n vault -- cat /vault/secrets/clisecret
kubectl exec -it <pod name> -n vault -- cat /vault/secrets/uisecret
```

