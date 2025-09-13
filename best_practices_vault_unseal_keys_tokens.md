# 🔒 1⃣ Best Practices for Securely Unsealing Vault

## ⚠️ What NOT to do:
- ❌ Hardcoding unseal keys in `.env` or source code
- ❌ Using plain environment variables (they can be accessed by any user/process)

## ✅ What TO DO:
- ✔️ Use **Shamir’s Secret Sharing** (Vault's built-in mechanism)
- ✔️ Store unseal keys in a **secure Hardware Security Module (HSM)**
- ✔️ Use **Auto-Unseal** with a **Cloud KMS** (AWS, GCP, or Azure)
- ✔️ Use an **external secure vault** (like AWS Secrets Manager or HashiCorp Vault Transit)

---

# 🔄 2⃣ Automate Vault Unsealing Securely

## 🔹 Option 1: Use Auto-Unseal (**Recommended**)
Instead of manually unsealing Vault, **enable auto-unseal** with a cloud-based **Key Management Service (KMS):**

- **AWS KMS**
- **Google Cloud KMS**
- **Azure Key Vault**

### Example (AWS KMS Auto-Unseal in Vault Config):
```hcl
seal "awskms" {
  region     = "us-east-1"
  kms_key_id = "your-aws-kms-key-id"
}
```
🔹 **This removes the need to store or use unseal keys manually.**

---

## 🔹 Option 2: Store Unseal Keys in a Secure Vault (**Not `.env`**)
1. 1️⃣ Store unseal keys **inside a separate Vault**
2. 2️⃣ Use a **privileged service account** to fetch and apply them
3. 3️⃣ Destroy the keys **after unsealing**

### Example (Fetching Unseal Keys Securely in Node.js):
```javascript
const vault = require('node-vault')({ endpoint: 'http://127.0.0.1:8200' });

async function fetchUnsealKeys() {
  console.log("🔐 Fetching unseal keys securely...");
  const result = await vault.read('secret/unseal-keys');
  return result.data.keys;  // These are stored inside Vault itself!
}

async function unsealVault() {
  try {
    const keys = await fetchUnsealKeys();

    console.log("🔓 Unsealing Vault...");
    for (let i = 0; i < 3; i++) {
      const response = await vault.unseal({ key: keys[i] });
      console.log(`🔑 Unseal Progress: ${response.progress}/3`);
      if (!response.sealed) {
        console.log("✅ Vault Unsealed Successfully!");
        break;
      }
    }
  } catch (err) {
    console.error("❌ Error unsealing Vault:", err.message);
  }
}

unsealVault();
```

🔹 **Why is this secure?**
- ✅ **Keys are never stored in plaintext**
- ✅ **Keys are retrieved from Vault itself** (not `.env`)
- ✅ **After unsealing, the keys can be revoked**

---

## 🔹 Option 3: Use Vault Agent Auto-Unseal
If you're using **Vault Enterprise**, you can deploy **Vault Agent** to automatically handle unsealing **without exposing secrets**.

### Steps:
1. 1️⃣ Configure Vault Agent with **auto-auth** and **template rendering**
2. 2️⃣ It will fetch the keys and **unseal automatically**
3. 3️⃣ No need to manually call `vault.unseal()` in Node.js

### Example Vault Agent Config (**Auto-Auth + Templating**):
```hcl
auto_auth {
  method "aws" {
    mount_path = "auth/aws"
    config = {
      role = "vault-auto-unseal"
    }
  }
}

vault {
  address = "https://vault.example.com"
}

template {
  destination = "/etc/secrets/unseal-keys.json"
  contents    = <<EOF
  {{ with secret "secret/unseal-keys" }}{{ .Data.keys_json }}{{ end }}
  EOF
}
```
🔹 **Vault Agent will retrieve the unseal keys securely and apply them.**

---

# 🏰 3⃣ Final Security Checklist
- ✔️ **Use Auto-Unseal (KMS or Vault Agent) instead of manually unsealing**
- ✔️ **Store unseal keys inside Vault itself**, not `.env`
- ✔️ **Rotate and destroy keys after unsealing**
- ✔️ **Use a Hardware Security Module (HSM) for extra protection**

---

# 🚀 Which Method Fits Your Setup?
- **Cloud-based?** ➡️ Use **Auto-Unseal with AWS/GCP/Azure KMS**
- **On-premises?** ➡️ Use **Vault Agent or Store Keys in Vault**
- **Tight security?** ➡️ **Rotate and revoke keys after unsealing**

