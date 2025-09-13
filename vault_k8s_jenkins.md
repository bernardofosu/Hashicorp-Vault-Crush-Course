# 🚀 HashiCorp Vault Integration with Kubernetes & Jenkins

## 📌 Prerequisites
- Kubernetes Cluster (minikube, k3s, or cloud-based like EKS, AKS, GKE)
- Helm installed (`brew install helm` or `curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash`)
- Jenkins installed in Kubernetes
- `kubectl` CLI configured
- `vault` CLI installed (`brew install vault` or `sudo apt install vault`)

---

## 🔹 Step 1: Install HashiCorp Vault in Kubernetes

### 🏗️ Deploy Vault using Helm
```sh
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault --set "server.dev.enabled=true"
```

### 🔍 Verify Vault Deployment
```sh
kubectl get pods
kubectl get svc | grep vault
```

---

## 🔹 Step 2: Initialize and Unseal Vault

### 🔑 Initialize Vault
```sh
kubectl exec -it vault-0 -- vault operator init
```
*Copy the generated unseal keys and root token somewhere safe.*

### 🔓 Unseal Vault
```sh
kubectl exec -it vault-0 -- vault operator unseal <UNSEAL_KEY_1>
kubectl exec -it vault-0 -- vault operator unseal <UNSEAL_KEY_2>
kubectl exec -it vault-0 -- vault operator unseal <UNSEAL_KEY_3>
```

### 🔐 Login to Vault
```sh
kubectl exec -it vault-0 -- vault login <ROOT_TOKEN>
```

---

## 🔹 Step 3: Enable Kubernetes Authentication in Vault

### 🏗️ Enable Kubernetes Auth
```sh
kubectl exec -it vault-0 -- vault auth enable kubernetes
```

### 🔍 Get Kubernetes Service Account Token
```sh
SA_NAME=$(kubectl get sa vault -o jsonpath='{.secrets[0].name}')
TOKEN=$(kubectl get secret $SA_NAME -o jsonpath='{.data.token}' | base64 --decode)
```

### 🔐 Configure Vault Authentication
```sh
kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
    token_reviewer_jwt="$TOKEN" \
    kubernetes_host=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.server}') \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

---

## 🔹 Step 4: Store Jenkins Secrets in Vault

### 🔑 Enable KV Secrets Engine
```sh
kubectl exec -it vault-0 -- vault secrets enable -path=jenkins kv
```

### 📌 Store a Secret
```sh
kubectl exec -it vault-0 -- vault kv put jenkins/api-token value=super-secret-token
```

### 🔍 Retrieve Secret (for testing)
```sh
kubectl exec -it vault-0 -- vault kv get jenkins/api-token
```

---

## 🔹 Step 5: Integrate Vault with Jenkins

### 🔧 Install Vault Plugin in Jenkins
1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Manage Plugins**
2. Search for **Vault Plugin** and install it

### 🛠️ Configure Vault in Jenkins
1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Configure System**
2. Find **HashiCorp Vault** section
3. Set **Vault URL**: `http://vault.default.svc.cluster.local:8200`
4. Choose **Vault Authentication Method**: Kubernetes
5. Set **Kubernetes Role**: `jenkins`
6. Click **Test Connection** → Save

---

## 🔹 Step 6: Use Vault in Jenkins Pipeline

### 📜 Sample Jenkinsfile
```groovy
pipeline {
    agent any
    environment {
        VAULT_ADDR = 'http://vault.default.svc.cluster.local:8200'
        VAULT_SECRET = vault path: 'jenkins/api-token', key: 'value'
    }
    stages {
        stage('Show Secret') {
            steps {
                script {
                    echo "Vault Secret: ${VAULT_SECRET}"
                }
            }
        }
    }
}
```

---

## ✅ Conclusion
🎉 You have successfully **installed HashiCorp Vault**, **configured it in Kubernetes**, and **integrated it with Jenkins**! Now, your secrets are securely managed and injected into Jenkins Pipelines. 🔒🚀


# 🚀🔐 HashiCorp Vault vs. Ansible Vault

Great question! **HashiCorp Vault and Ansible Vault** both deal with **secrets management**, but they serve different purposes and have key differences. 🚀🔐

## 🔍 **HashiCorp Vault vs. Ansible Vault**

| **Feature**          | **HashiCorp Vault 🏦** | **Ansible Vault 🔐** |
|---------------------|----------------------|----------------------|
| **Purpose**        | Centralized secrets management system for dynamic & static secrets | Encrypts sensitive data in Ansible playbooks |
| **Scope**          | General-purpose, integrates with multiple apps & platforms | Mainly for Ansible automation |
| **Secret Types**   | API keys, database credentials, dynamic cloud creds | Ansible YAML files with encrypted vars |
| **Access Control** | Fine-grained access with policies (ACLs) | Basic access with passwords |
| **Dynamic Secrets** | Generates temporary credentials for AWS, DBs, etc. | No dynamic secrets, only encrypted storage |
| **API & CLI**      | Provides REST API & CLI for programmatic access | CLI-based, no REST API |
| **Storage Backend** | Multiple options (Consul, MySQL, AWS S3, etc.) | Local file encryption only |
| **Encryption**      | Uses transit encryption & AES-GCM | Uses AES-256 encryption |
| **Integration**     | Works with Kubernetes, Jenkins, Terraform, CI/CD | Only works within Ansible |

---

## 🛠️ **When to Use Each?**

✅ **Use HashiCorp Vault** when you need a **central, secure, and scalable** secrets management system across **multiple applications, platforms, and services**.  
✅ **Use Ansible Vault** when you **only need to encrypt sensitive data** (like passwords, SSH keys) inside **Ansible playbooks**.  

---

## 🔹 **Example: HashiCorp Vault with Ansible**

If you're using **Ansible with Vault**, you can dynamically fetch secrets from Vault inside a playbook:

```yaml
- name: Fetch secret from HashiCorp Vault
  hosts: localhost
  tasks:
    - name: Retrieve secret
      ansible.builtin.uri:
        url: "http://127.0.0.1:8200/v1/secret/data/myapp"
        method: GET
        headers:
          X-Vault-Token: "{{ lookup('env', 'VAULT_TOKEN') }}"
      register: vault_response

    - name: Show Secret
      debug:
        msg: "{{ vault_response.json.data.data.password }}"
```

---

## ✅ **Final Verdict**

👉 **HashiCorp Vault** is a **general-purpose secrets management system** that can integrate with multiple tools (**Kubernetes, CI/CD, cloud providers**).  
👉 **Ansible Vault** is **specialized for Ansible**, mainly for **encrypting sensitive data in playbooks**.  

Let me know if you need help setting up either! 🚀🔐

