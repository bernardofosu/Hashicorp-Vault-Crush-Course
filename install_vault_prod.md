
# Deploying HashiCorp Vault in Production Mode

This guide provides step-by-step instructions for deploying HashiCorp Vault in a production environment, transitioning from a development setup to production-ready deployment.

# Why Use `/etc/vault.d/vault.hcl` Instead of a Custom One? 🤔

## You Can Use This Command Instead of the Custom One: 🚀
```bash
sudo vault server -config=/etc/vault.d/vault.hcl
```

## You Asked: **Why?** 🤷‍♂️

### Explanation 🧑‍💻

Using `sudo vault server -config=/etc/vault.d/vault.hcl` instead of a custom configuration is recommended because:

### ✅ **1. Existing Configuration Management**
- `/etc/vault.d/vault.hcl` is typically the default location for Vault's configuration on Linux systems.
- It is managed by system administrators and often aligned with systemd services.

### 🛡 **2. Systemd Integration**
- Vault service configurations (`vault.service`) often point to this file for consistency and reliability.
- Using this ensures the service can be managed using `systemctl`.

### 🔒 **3. Permissions and Path Management**
- The `/etc/vault.d/` directory is designed for configuration files with appropriate permissions and ownership.
- Running from a custom path like `./config.hcl` may introduce permission or security issues.

### 📊 **4. Service Monitoring and Logging**
- Logs and error reports are integrated with `journalctl` and `systemctl` when using `/etc/vault.d/vault.hcl`.
- Custom paths may complicate log management.

### 🔐 **5. TLS and Security Management**
- If a TLS setup exists in `/etc/vault.d/vault.hcl`, using it ensures secure communication.
- Your current example disables TLS (`tls_disable = "true"`), which is fine for local testing but not recommended in production.

[Start Vault Systemd](start_vault_systemd.md)

## 1. Stop Vault Development Mode

First, stop any Vault instance running in development mode. Open the terminal where Vault is running and press `Ctrl + C` to shut down the development server. Verify it has stopped by running:
```bash
vault status
```
You should see a "connection refused" message, indicating that the development server is no longer active.

## 2. Reset the Vault Token

Reset the Vault token to clear any lingering development credentials:
```bash
unset VAULT_TOKEN
```

## 3. Create the Production Configuration File (custom)

Create a configuration file named `config.hcl` to specify Vault’s production settings. Use the following configuration:
```sh
touch config.hcl
```
```h
storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = "true"
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true
```
### Explanation of Configuration Parameters:
- **Storage**: Defines Raft storage for persistent data storage at `./vault/data`.
- **Listener**: Sets up TCP listener at localhost. TLS is disabled for this example.
- **API and Cluster Addresses**: Specifies API and cluster endpoints.
- **UI**: Enables the Vault web interface.

## 4. Create the Storage Directory

Create the storage directory for the 'Raft' backend with the following command:

```bash
mkdir -p ./vault/data
```

## 5. Start Vault in Production Mode

Start Vault using the config file:

```bash
vault server -config=config.hcl
```
This initiates Vault with the specified production settings. Vault is now running in production mode.

```sh
sudo cat /etc/vault.d/vault.hcl
``` 
## 6. Export Vault Address

To ensure the client can communicate with the Vault server, export the Vault address:

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
```
```sh
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ vault status        
Key                Value
---                -----
Seal Type          shamir
Initialized        false
Sealed             true
```

## 7. Initialize Vault

Initialize Vault to generate unseal keys and a root token:

```bash
vault operator init
```

This command provides the unseal keys and root token. Store these securely as they are critical for accessing and managing Vault.

## 8. Unseal Vault

Vault starts in a sealed state. Use the following command multiple times with unseal keys to unseal Vault:

```bash
vault operator unseal
```

Alternatively, use the Vault UI to unseal Vault for a more interactive experience.

## 9. Access the Vault UI (AWS Example)

If you’re running Vault on an AWS EC2 instance, access it via the public IP. Open a browser and enter the IP address followed by the port:

```
http://<public_ip>:8200/ui
```

Use the unseal keys and root token in the UI to access Vault.

## 10. Explore the REST API

Vault provides a REST API for interacting from other applications or the command line. Here’s an example command to check if Vault is initialized:

```bash
curl -X GET $VAULT_ADDR/v1/sys/init
```

The REST API is useful for development and troubleshooting.


# 🚀 Running HashiCorp Vault in the Background

When you start Vault using `vault server -config=config.hcl`, it runs in the foreground and will stop once you close the terminal. To keep it running in the background, consider these options:

## ✅ **Option 1: Run in the Background Using `nohup`**

You can run Vault in the background using `nohup` (No Hang Up) to ensure it continues running after the terminal is closed:

```bash
nohup vault server -config=config.hcl > vault.log 2>&1 &
```
- `vault.log` will store the logs.
- `&` runs the process in the background.

### 📌 **Check if it’s running:**
```bash
ps aux | grep vault
```

### 📌 **View logs:**
```bash
cat vault.log
```

### 📌 **Stop Vault:**
```bash
pkill vault
```

---

## ✅ **Option 2: Use `tmux` or `screen`**

You can use terminal multiplexer tools like `tmux` or `screen` to keep Vault running in an active session.

### 📌 **Start a new session:**
insatll tmux
```sh
sudo apt update && sudo apt install tmux
```

```bash
tmux new -s vault
tmux kill-session -t vault
```
✅ How to Use Tmux Keybindings Correctly
- Press Ctrl + b (hold Ctrl, then press b).
- Release both keys.
- Then press % for a vertical split or " for a horizontal split.

### 📌 **Run Vault:**
```bash
vault server -config=config.hcl
```

### 📌 **Detach from the session:**
- Press `Ctrl + b`, then `d` to detach.

### 📌 **Reattach to the session later:**
```bash
tmux attach -t vault
```

### 📌 **Stop Vault:**
- Reattach using the above command and press `Ctrl + C`.