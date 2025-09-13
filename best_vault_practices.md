## 🔐 Best Approach for Managing Vault Secrets in a Node.js Application

The best method depends on your security model and how your application accesses secrets. Here are two common approaches:

### 1️⃣ Use Vault Roles (Recommended for Most Cases)

✅ Best for applications like Node.js, backend services, CI/CD pipelines, and automation.
Instead of creating individual users, use roles with policies to allow applications to dynamically authenticate and access only what they need.

#### How to Set Up a Vault Role for Your Node.js App

**1️⃣ Enable AppRole Authentication (if not enabled):**
```sh
vault auth list

vault auth enable approle
```

**2️⃣ Create a Policy (`node-app-policy`) to grant read access:**
```sh
echo 'path "secrets/data/prod" {
  capabilities = ["read"]
}' > node-app-policy.hcl
```
```hcl
path "secrets/data/prod" {
  capabilities = ["read"]
}
```
```sh
vault policy write node-app-policy node-app-policy.hcl
```

**3️⃣ Attach the Policy to an AppRole:**
```sh
vault write auth/approle/role/node-app \
  token_policies="node-app-policy" \
  token_ttl=1h \
  token_max_ttl=4h
```

**🔹 Command to Create a Renewable Token**
```sh
vault write auth/approle/role/node-app \
  token_policies="node-app-policy" \
  token_period=720h
```
**📌 What This Does**
token_period=720h → The token will automatically renew every 30 days until explicitly revoked.

Unlike token_ttl, it does not expire as long as it's renewed before Vault restarts.

**4️⃣ Retrieve the Role ID and Secret ID:**
```sh
vault read auth/approle/role/node-app/role-id
vault write -f auth/approle/role/node-app/secret-id
```
## 🔐 Retrieving Authentication Credentials for AppRole in Vault

These two commands are used to retrieve authentication credentials (Role ID and Secret ID) for an **AppRole** in Vault. Let's break them down:

---

### 🔹 1️⃣ Retrieve the Role ID

```sh
vault read auth/approle/role/node-app/role-id
```

✔ **What it does**:
- Reads the **Role ID** for the `node-app` AppRole.
- The **Role ID** is static and acts like a **username** for authentication.

✔ **Example output**:

```
Key        Value
---        -----
role_id    12345678-abcd-90ef-1234-56789abcdef0
```

- The `role_id` will be used by your Node.js application to authenticate with Vault.

---

### 🔹 2️⃣ Generate a Secret ID

```sh
vault write -f auth/approle/role/node-app/secret-id
```

✔ **What it does**:
- Generates a new **Secret ID** for the `node-app` AppRole.
- The **Secret ID** acts like a **password** and is **one-time use** (unless configured otherwise).

✔ **Example output**:

```
Key         Value
---         -----
secret_id   a1b2c3d4-e5f6-7890-abcd-1234ef567890
```

- Your application needs both `role_id` and `secret_id` to log in.

---

### 🔹 How Your Application Uses These

Your **Node.js app** will use both `role_id` and `secret_id` to request a Vault token:

```sh
vault write auth/approle/login role_id="your-role-id" secret_id="your-secret-id"
```

If authentication is successful, Vault returns a **client token** that your app will use to read secrets.


**5️⃣ In Your Node.js App (`config.js`), Authenticate and Fetch Secrets:**
```js
const vault = require("node-vault")({
    endpoint: "http://127.0.0.1:8200",
});

async function getSecrets() {
    const role_id = process.env.VAULT_ROLE_ID;
    const secret_id = process.env.VAULT_SECRET_ID;

    try {
        const auth = await vault.approleLogin({ role_id, secret_id });
        vault.token = auth.auth.client_token; // Set the new token

        const result = await vault.read("secrets/data/prod");
        console.log("✅ Secrets loaded successfully!");
        return result.data.data;
    } catch (err) {
        console.error("❌ Failed to fetch secrets from Vault:", err);
        process.exit(1);
    }
}

module.exports = getSecrets;
```

**6️⃣ Set Environment Variables (.env or Secure Secrets Management)**
```sh
export VAULT_ROLE_ID="your-role-id"
export VAULT_SECRET_ID="your-secret-id"
```

### 🔹 Why AppRole?
✅ More secure than hardcoding secrets.  
✅ Tokens are short-lived (can be revoked).  
✅ Can be rotated without updating the application code.  

---

### 2️⃣ Using User-Based Authentication (Userpass)

✅ **Best for human users (Admins, Devs, Operators)**  
Instead of using AppRole, you can create user accounts:
```sh
vault write auth/userpass/users/<username> password=<password> policies=<policy>
```
However, this is **not ideal for applications** because:
- ❌ Tokens don’t rotate automatically (you’d need to renew manually).
- ❌ It’s harder to manage programmatically.
- ❌ If compromised, it grants too much access.

**Example:**
```sh
vault auth enable userpass

vault write auth/userpass/users/node-user password="securepassword" policies="node-app-policy"
```

**Then, in your Node.js app:**
```js
const vault = require("node-vault")({
   endpoint: "http://127.0.0.1:8200"
});

async function getSecrets() {
   try {
       const auth = await vault.userpassLogin({ username: "node-user", password: "securepassword" });
       vault.token = auth.auth.client_token;

       const result = await vault.read("secrets/data/prod");
       return result.data.data;
   } catch (err) {
       console.error("❌ Failed to fetch secrets from Vault:", err);
   }
}
```

### 🔹 Why Not Userpass for Applications?
❌ Tokens do not auto-rotate.  
❌ Storing passwords is risky.  
❌ Not scalable for multiple applications.  

---

## 🎯 Best Practice: Use Vault Roles (AppRole) Instead of Users

- **For Applications:** Use AppRole authentication (**most secure, automated**).  
- **For Developers/Admins:** Use Userpass or Token authentication.  
- **For CI/CD Pipelines:** Use GitHub or JWT authentication.  


### ✅ Option 3: Automatically Renew in Your Node.js App

If you're using Node.js, modify your app to auto-renew using tokenRenewSelf().
🔹 Node.js Auto-Renew Script
```sh
const vault = require("node-vault")({ endpoint: "http://127.0.0.1:8200" });

async function authenticateAndFetchSecrets() {
    const role_id = process.env.VAULT_ROLE_ID;
    const secret_id = process.env.VAULT_SECRET_ID;

    try {
        // Authenticate with Vault
        const auth = await vault.approleLogin({ role_id, secret_id });
        vault.token = auth.auth.client_token;
        console.log("🔑 Token obtained, scheduling renewal...");

        // Schedule auto-renewal before expiration
        renewToken(auth.auth.client_token, auth.auth.lease_duration);

        // Fetch secrets
        const result = await vault.read("secrets/data/prod");
        console.log("✅ Secrets loaded successfully!");
        return result.data.data;
    } catch (err) {
        console.error("❌ Failed to fetch secrets from Vault:", err);
        process.exit(1);
    }
}

// Auto-renew the token periodically
async function renewToken(clientToken, leaseDuration) {
    setTimeout(async () => {
        try {
            const renewResponse = await vault.tokenRenewSelf();
            console.log("🔄 Token renewed successfully!");
            renewToken(clientToken, renewResponse.auth.lease_duration);
        } catch (err) {
            console.error("⚠️ Failed to renew token:", err);
            process.exit(1); // Exit if renewal fails
        }
    }, (leaseDuration - 60) * 1000); // Renew 1 min before expiration
}

// Start the process
authenticateAndFetchSecrets();
```
📌 How This Works
- Logs in with role_id and secret_id.
- Schedules a token renewal 1 minute before expiration.
- Fetches secrets using the renewed token.
- Ensures continuous access without requiring a long-lived token.

🚀 Best Practice Recommendation
- ✅ Use token_period instead of token_ttl for an auto-renewable token.
- ✅ Use a renewal script if you have to handle token expiry manually.
- 🚫 Avoid long-lived tokens (token_ttl=0) unless absolutely necessary.
