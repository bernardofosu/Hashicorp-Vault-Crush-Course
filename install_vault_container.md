# 🐳 Using HashiCorp Vault with Docker

HashiCorp Vault is a powerful tool for managing secrets and protecting sensitive data. Docker provides a convenient way to run Vault in a containerized environment. Here's how to set it up!

---

## 🛠️ **Prerequisites**

- Docker installed on your system ([Install Docker](https://docs.docker.com/get-docker/))
- Basic knowledge of Docker commands

---

## 🚀 **Running Vault with Docker**

You can quickly start a Vault container using the following command:

```bash
docker run --cap-add=IPC_LOCK -e 'VAULT_DEV_ROOT_TOKEN_ID=root' -e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200' -p 8200:8200 hashicorp/vault
```

### 🔎 **Explanation:**
- `--cap-add=IPC_LOCK`: Allows Vault to use memory locking for increased security.
- `-e VAULT_DEV_ROOT_TOKEN_ID=root`: Sets the root token (for development only).
- `-e VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200`: Ensures Vault listens on all network interfaces.
- `-p 8200:8200`: Maps Vault’s default port to your local machine.
- `hashicorp/vault`: The official Vault Docker image.

> ⚠️ **Note:** This setup is for development only. Never use `VAULT_DEV_ROOT_TOKEN_ID` in production.

---

## 🔑 **Accessing the Vault UI**

Once the container is running, you can access the Vault UI at:

```
http://localhost:8200
```

- Use the root token (`root`) to log in.

---

## 📥 **Running Vault in Server Mode**

For a more production-like setup, run Vault in server mode using a configuration file. Create a `config.hcl` file with the following content:

```hcl
storage "file" {
  path = "/vault/data"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}

ui = true
```

Then run the Vault server using Docker:

```bash
docker run --cap-add=IPC_LOCK -v $(pwd)/config.hcl:/vault/config/config.hcl -v $(pwd)/data:/vault/data -p 8200:8200 hashicorp/vault server -config=/vault/config/config.hcl
```

### 🛡️ **Explanation:**
- `-v $(pwd)/config.hcl:/vault/config/config.hcl`: Mounts your config file into the container.
- `-v $(pwd)/data:/vault/data`: Stores Vault data locally.
- `server -config=/vault/config/config.hcl`: Starts Vault in server mode using your config.

---

## 🧑‍💻 **Using Vault CLI**

You can interact with Vault using the CLI within the container:

```bash
docker exec -it <container_name> vault status
```

### Examples:
- Initialize Vault:
  ```bash
  docker exec -it <container_name> vault operator init
  ```
- Unseal Vault (use 3 keys from the init step):
  ```bash
  docker exec -it <container_name> vault operator unseal
  ```
- Authenticate using the root token:
  ```bash
  docker exec -it <container_name> vault login root
  ```

---

## 🛡️ **Best Practices**

- **Persistent Storage:** Ensure Vault data is persisted using Docker volumes.
- **TLS Security:** Enable TLS in production environments.
- **Token Management:** Avoid using root tokens in production.
- **Sealing and Unsealing:** Manage unsealing using Shamir’s secret sharing.

---

## 🎉 **Conclusion**

You now know how to run and manage HashiCorp Vault using Docker. This setup is great for development, but for production use, consider securing Vault further with TLS and proper access control policies.

Happy vaulting! 🔐🚀

