# 🚀 Connect Multiple Vault Nodes in a Raft Cluster

To connect multiple Vault nodes in a Raft cluster and enable leader election, follow these steps:

## ✅ Step 1: Install and Configure Vault on Each Node

Ensure that Vault is installed on all nodes. You can verify it using:

```bash
vault version
```

---

## 📦 Step 2: Configure the Vault Configuration File

On each node, create or edit the configuration file `/etc/vault.d/vault.hcl` and customize it for that node. Here's how:

### 🟢 **Node 1 Configuration (Leader Candidate)**
```hcl
ui = true

storage "raft" {
  path    = "/opt/vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  cluster_address = "https://192.168.1.10:8201"
  tls_disable = "true"
}

api_addr = "http://192.168.1.10:8200"
cluster_addr = "https://192.168.1.10:8201"
```

### 🟢 **Node 2 Configuration**
```hcl
ui = true

storage "raft" {
  path    = "/opt/vault/data"
  node_id = "node2"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  cluster_address = "https://192.168.1.11:8201"
  tls_disable = "true"
}

api_addr = "http://192.168.1.11:8200"
cluster_addr = "https://192.168.1.11:8201"
```

### 🟢 **Node 3 Configuration**
```hcl
ui = true

storage "raft" {
  path    = "/opt/vault/data"
  node_id = "node3"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  cluster_address = "https://192.168.1.12:8201"
  tls_disable = "true"
}

api_addr = "http://192.168.1.12:8200"
cluster_addr = "https://192.168.1.12:8201"
```

> Ensure `node_id` is unique for each node.

---

## 🚀 Step 3: Initialize and Join the Cluster

### 🔎 **Initialize the First Node (Node 1)**
Run the following on Node 1:

```bash
vault operator init
```
- Save the unseal keys and root token.
- You'll need these to unseal Vault and manage it.

### 🔎 **Unseal the First Node**

```bash
vault operator unseal
```
Repeat this command 3 times using different unseal keys.

### 🔎 **Join Node 2 and Node 3 to the Cluster**
On each follower node (Node 2 and Node 3):

```bash
vault operator raft join http://192.168.1.10:8200
```
- Replace `192.168.1.10` with the leader node’s API address.

- Check the join status:

```bash
vault operator raft list-peers
```

### 🔎 **Unseal the Other Nodes**
Use the same unseal keys from Node 1:

```bash
vault operator unseal
```

---

## 🛡️ Step 4: Verify the Cluster

On any node, check cluster status:

```bash
vault status
```
- **Leader:** One node will show `HA Mode: active`.
- **Followers:** Other nodes will show `HA Mode: standby`.

You can also check all nodes using:

```bash
vault operator raft list-peers
```

---

## 💡 Next Steps

- Enable TLS for secure communication.
- Configure replication if needed.
- Set up policies and authentication.

Good luck with your Vault cluster! 🚀

