# HashiCorp Vault Service Status Report 🛠️

## Command Execution 🚀
```bash
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ sudo vault server -config=/etc/vault.d/vault.hcl
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ sudo systemctl start vault
```

### Error Message ❌
```bash
Job for vault.service failed because the control process exited with error code.
See systemctl status vault.service and journalctl -xeu vault.service for details.
```

## Service Status Check 🔍
```bash
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ sudo systemctl status vault.service
```

### Status Output 📊
```bash
× vault.service - "HashiCorp Vault - A tool for managing secrets"
     Loaded: loaded (/usr/lib/systemd/system/vault.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Tue 2025-03-25 12:52:32 GMT; 7min ago
       Docs: https://developer.hashicorp.com/vault/docs
    Process: 1179 ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl (code=exited, status=1/FAILURE)
   Main PID: 1179 (code=exited, status=1/FAILURE)

Mar 25 12:52:32 Nana-Kwasi-Fosu systemd[1]: vault.service: Scheduled restart job, restart counter is at 3.
Mar 25 12:52:32 Nana-Kwasi-Fosu systemd[1]: vault.service: Start request repeated too quickly.
Mar 25 12:52:32 Nana-Kwasi-Fosu systemd[1]: vault.service: Failed with result 'exit-code'.
Mar 25 12:52:32 Nana-Kwasi-Fosu systemd[1]: Failed to start vault.service - "HashiCorp Vault - A tool for managing secrets".
```

## Journal Log Check 📜
```bash
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ sudo journalctl -xeu vault.service
```

### Error Details ⚠️
```bash
Error parsing listener configuration.
Error initializing listener of type tcp: listen tcp4 0.0.0.0:8200: bind: address already in use
2025-03-25T12:48:32.255Z [INFO] proxy environment: http_proxy="" https_proxy="" no_proxy=""
```

## Configuration Resolution 🔧
To resolve the conflict in your Vault configuration, you should comment out one of the listener blocks. Here’s how you can decide:

### Use HTTPS (Recommended for Production) 🔒
Keep the HTTPS listener and comment out the HTTP listener:

```h
# HTTP listener
# listener "tcp" {
#   address = "127.0.0.1:8200"
#   tls_disable = 1
# }

# HTTPS listener
listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/tls.crt"
  tls_key_file  = "/opt/vault/tls/tls.key"
}
```

### Use HTTP (For Testing Purposes Only) 🌐
Keep the HTTP listener and comment out the HTTPS listener:
```sh
sudo nano /etc/vault.d/vault.hcl
```
```h
# HTTP listener
listener "tcp" {
  address = "127.0.0.1:8200"
  tls_disable = 1
}

# HTTPS listener
# listener "tcp" {
#   address       = "0.0.0.0:8200"
#   tls_cert_file = "/opt/vault/tls/tls.crt"
#   tls_key_file  = "/opt/vault/tls/tls.key"
# }
```
```sh
workstation@Nana-Kwasi-Fosu:/mnt/d/Hashicorp Vault$ vault status        
Error checking seal status: Get "https://127.0.0.1:8200/v1/sys/seal-status": http: server gave HTTP response to HTTPS client
```
export root url using http instead of https
```sh
export VAULT_ADDR=http://127.0.0.1:8200
```