## 🛡️ Using HashiCorp Vault with Python

To use **HashiCorp Vault** with Python, you can use the `hvac` (HashiCorp Vault API Client) library, which is a popular and well-maintained Python client for Vault. Here’s how you can get started:

---

## 🛠️ Installation

First, install the `hvac` library using `pip`:

```bash
pip install hvac
```

---

## 🚀 Connecting to Vault

Here’s a simple example to authenticate and read secrets using Python:

```python
import hvac
import os

# Connect to Vault
client = hvac.Client(
    url=os.getenv('VAULT_ADDR', 'http://127.0.0.1:8200')
)

# Authenticate using AppRole
role_id = os.getenv('VAULT_ROLE_ID')
secret_id = os.getenv('VAULT_SECRET_ID')

if not role_id or not secret_id:
    print("❌ VAULT_ROLE_ID or VAULT_SECRET_ID is missing!")
    exit(1)

response = client.auth.approle.login(role_id=role_id, secret_id=secret_id)

# Set the client token
client.token = response['auth']['client_token']
print("✅ Successfully authenticated with Vault!")
```

---

## 🔎 Reading Secrets

Once authenticated, you can read secrets from a specific path:

```python
# Path to secret (e.g., for KV version 2)
secret_path = 'secrets/data/prod'

try:
    secret_response = client.secrets.kv.v2.read_secret_version(path=secret_path)
    data = secret_response['data']['data']

    print("🔐 Retrieved Secrets:")
    for key, value in data.items():
        print(f"{key}: {value}")
except Exception as e:
    print(f"❌ Failed to read secret: {e}")
```

> **Note:** For KV v1, you would use `client.secrets.kv.v1.read_secret(path='secrets/prod')`.

---

## 🔄 Renewing Tokens

If your token is renewable, you can renew it using:

```python
try:
    renew_response = client.auth.token.renew_self()
    print("🔄 Token renewed successfully!")
    print(f"New Lease Duration: {renew_response['auth']['lease_duration']} seconds")
except Exception as e:
    print(f"⚠️ Failed to renew token: {e}")
```

---

## 🛡️ Best Practices

- **Environment Variables:**
    - Store sensitive data like `VAULT_ADDR`, `VAULT_ROLE_ID`, and `VAULT_SECRET_ID` using environment variables.
- **Error Handling:**
    - Implement error handling using `try-except` blocks to catch exceptions.
- **Token Management:**
    - Ensure tokens are renewed if needed, especially if they are short-lived.
- **Logging:**
    - Use proper logging instead of `print()` for better debugging in production.

---

## 🎉 Conclusion

You’ve now successfully set up and connected Python to HashiCorp Vault using the `hvac` library! You can authenticate using AppRole, read secrets, and manage tokens efficiently.

Let me know if you'd like further examples or explanations!

