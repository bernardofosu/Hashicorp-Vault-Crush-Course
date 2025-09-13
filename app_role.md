# ✅ Secure Fix: Use Vault for MongoDB Credentials in Node.js
Ensure You're Authenticated
- First, confirm you're authenticated with a valid token using the following command:
```sh
vault login
```
If you're using a root token (often used for initial setup), it will look like this:
```sh
vault login <your-root-token>
```
Check the Current Token Permissions
You can check your token's capabilities using:
```sh
vault token lookup
```
Rather than storing credentials in environment variables or a config file, use **Vault’s AppRole** for secure, temporary access.

## 1️⃣ Create a Vault Policy for Read-Only Access
Define a policy that allows only **read** access to MongoDB credentials:

```bash
vault policy write mongo-readonly - <<EOF
path "secret/data/mongodb" {
  capabilities = ["read"]
}
EOF
vault policy list
```

## 2️⃣ Create a Vault Role for Node.js
Enable AppRole authentication and bind it to the `mongo-readonly` policy:

```bash
vault auth enable approle
vault write auth/approle/role/nodejs-app token_policies=mongo-readonly
vault list auth/approle/role
```

Retrieve the **Role ID** and **Secret ID** for your Node.js app:

```bash
vault read auth/approle/role/nodejs-app/role-id
vault write -f auth/approle/role/nodejs-app/secret-id
```

## 3️⃣ Modify Node.js App to Use Vault
Instead of hardcoding credentials, let Node.js dynamically retrieve them from Vault.

### 🔹 Secure Node.js Configuration (`vault.js`)
```javascript
const vault = require('node-vault')({ endpoint: 'http://127.0.0.1:8200' });

async function getVaultToken() {
    const role_id = process.env.VAULT_ROLE_ID;
    const secret_id = process.env.VAULT_SECRET_ID;

    const result = await vault.approleLogin({ role_id, secret_id });
    return result.auth.client_token;
}

async function getMongoCredentials() {
    const token = await getVaultToken();
    vault.token = token;
    const secrets = await vault.read('secret/data/mongodb');
    return secrets.data.data; // Vault KV v2 stores data under `data`
}

module.exports = getMongoCredentials;
```

## 4️⃣ Use Retrieved Credentials in Your MongoDB Connection
Modify your `db.js` file to use secrets from Vault:

```javascript
const mongoose = require('mongoose');
const getMongoCredentials = require('./vault');

async function connectDB() {
    const { username, password, uri } = await getMongoCredentials();
    
    const mongoURI = `mongodb+srv://${username}:${password}@${uri}`;
    
    await mongoose.connect(mongoURI, { useNewUrlParser: true, useUnifiedTopology: true });
    console.log('Connected to MongoDB securely via Vault!');
}

connectDB().catch(console.error);
```

## 🎯 Why This is Secure
✅ **No hardcoded credentials** in environment variables or files  
✅ **Short-lived tokens** instead of long-lived secrets  
✅ **Fine-grained access control** via Vault policies  

With this approach, your Node.js app dynamically retrieves MongoDB credentials securely from Vault without exposing them in code. 🚀🔒



### ✅ Example API Request

Here’s how the request would look using curl:
```sh
curl --request POST \
  --data '{"role_id": "<your-role-id>", "secret_id": "<your-secret-id>"}' \
  http://127.0.0.1:8200/v1/auth/approle/login
```
🛡 Successful Response Example
```sh
auth = {
  "request_id": "bdce8aab-b9c8-be1a-edeb-71b63eaa6e90",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "[VAULT_TOKEN_EXAMPLE]",
    "accessor": "R6ANnvu9JrEcNHl9f5BGm5W3",
    "policies": [
      "default",
      "mongo-readonly"
    ],
    "token_policies": [
      "default",
      "mongo-readonly"
    ],
    "metadata": {
      "role_name": "nodejs-app"
    },
    "lease_duration": 2764800,
    "renewable": true,
    "entity_id": "50ae0a45-6242-1e50-8215-7ae9a855c46a",
    "token_type": "service",
    "orphan": true,
    "mfa_requirement": null,
    "num_uses": 0
  },
  "mount_type": ""
}
```
client_token: This is the token used for further API requests.

lease_duration: How long the token is valid (e.g., 3600 seconds).

renewable: If true, the token can be renewed.


```sh
const vault = require("node-vault")({
    endpoint: process.env.VAULT_ADDR || "http://127.0.0.1:8200",
});

async function getVaultToken() {
    try {
        console.log("🔑 Authenticating with Vault using AppRole...");

        const role_id = process.env.VAULT_ROLE_ID;
        const secret_id = process.env.VAULT_SECRET_ID;

        if (!role_id || !secret_id) {
            throw new Error("❌ VAULT_ROLE_ID or VAULT_SECRET_ID is missing!");
        }

        const auth = await vault.approleLogin({ role_id, secret_id });
        vault.token = auth.auth.client_token;

        console.log("✅ Vault token obtained successfully!");
        
        // Ensure lease_duration is a valid number
        const leaseDuration = auth.auth.lease_duration || 3600; // Default to 1 hour if undefined
        renewToken(auth.auth.client_token, leaseDuration);

        return auth.auth.client_token;
    } catch (error) {
        console.error("❌ Vault authentication failed:", error);
        process.exit(1);
    }
}

// 🔄 Auto-renew token periodically with safe timeout
async function renewToken(clientToken, leaseDuration) {
    // Renew 1 min before expiry, but prevent overflow
    const safeTimeout = Math.min((leaseDuration - 60) * 1000, 2147483647);

    console.log(`🔄 Scheduling token renewal in ${safeTimeout / 1000} seconds.`);

    setTimeout(async () => {
        try {
            const renewResponse = await vault.tokenRenewSelf();
            console.log("🔄 Vault token renewed successfully!");

            // Ensure lease_duration is valid before reusing it
            const newLeaseDuration = renewResponse.auth.lease_duration || 3600;
            renewToken(clientToken, newLeaseDuration);
        } catch (error) {
            console.error("⚠️ Failed to renew token:", error);
            process.exit(1);
        }
    }, safeTimeout);
}

async function getSecrets() {
    try {
        console.log("🔄 Fetching secrets from Vault... Using AppRole authentication");

        if (!vault.token) {
            await getVaultToken();
        }

        const result = await vault.read("secrets/data/prod");

        console.log("✅ Secrets loaded successfully!");

        return {
            NODE_ENV: result.data.data.NODE_ENV || "production",
            PORT: result.data.data.PORT || 3000,
            USER: result.data.data.USER,
            PASSWORD: result.data.data.PASSWORD,
            CONN_STR: result.data.data.CONN_STR,
            CONN_LOCAL_STR: result.data.data.CONN_LOCAL_STR,
        };
    } catch (err) {
        console.error("❌ Failed to fetch secrets from Vault:", err);
        process.exit(1);
    }
}

module.exports = getSecrets;
```



# 🛡️ Overview

This Node.js script uses the `node-vault` library to authenticate with **HashiCorp Vault** using **AppRole** authentication. It handles three primary tasks:

1. **Authenticate using AppRole** to obtain a Vault token.
2. **Renew the token** if it's renewable and before it expires.
3. **Fetch secrets** from Vault using the token.

---

## 📦 Importing Vault
```javascript
const vault = require("node-vault")({
    endpoint: process.env.VAULT_ADDR || "http://127.0.0.1:8200",
});
```
- This imports the `node-vault` library for Vault interactions.
- The Vault address is taken from the `VAULT_ADDR` environment variable.
- If not set, it defaults to `http://127.0.0.1:8200`.

---

## 🔑 Authenticating with AppRole
```javascript
async function getVaultToken() {
    try {
        console.log("🔑 Authenticating with Vault using AppRole...");

        const role_id = process.env.VAULT_ROLE_ID;
        const secret_id = process.env.VAULT_SECRET_ID;
```
- **AppRole** is used for machines or applications to authenticate securely.
- `role_id` and `secret_id` are credentials for the AppRole, stored in environment variables.

---

## 🔎 Validation
```javascript
if (!role_id || !secret_id) {
    throw new Error("❌ VAULT_ROLE_ID or VAULT_SECRET_ID is missing!");
}
```
- It checks if `role_id` or `secret_id` is missing and throws an error if so.

---

## 🚀 Logging In
```javascript
const auth = await vault.approleLogin({ role_id, secret_id });
vault.token = auth.auth.client_token;

console.log("✅ Vault token obtained successfully!");
```
- The script sends an AppRole login request using `approleLogin()`.
- On success, the received token (`auth.auth.client_token`) is stored for subsequent requests.

---

## ⏳ Handling Token Expiration
```javascript
const leaseDuration = auth.auth.lease_duration || 3600;
renewToken(auth.auth.client_token, leaseDuration);
```
- `lease_duration` indicates how long the token is valid.
- If `lease_duration` is `0`, it means the token never expires.
- A default value of **3600 seconds (1 hour)** is used if `lease_duration` is missing or invalid.

---

## 🔄 Token Renewal
```javascript
async function renewToken(clientToken, leaseDuration) {
    const safeTimeout = Math.min((leaseDuration - 60) * 1000, 2147483647);
```
- **Token renewal** is scheduled using `setTimeout()`.
- `safeTimeout` ensures the renewal doesn't exceed JavaScript's maximum timer limit of **2,147,483,647 ms**.
- `leaseDuration - 60` ensures renewal attempts 1 minute before the token expires.

---

## 🔄 Performing Token Renewal
```javascript
setTimeout(async () => {
    try {
        const renewResponse = await vault.tokenRenewSelf();
        console.log("🔄 Vault token renewed successfully!");
```
- `tokenRenewSelf()` requests Vault to renew the token.
- On success, the new `lease_duration` is used for the next renewal.

---

## 🛡️ Handling New Lease Duration
```javascript
const newLeaseDuration = renewResponse.auth.lease_duration || 3600;
renewToken(clientToken, newLeaseDuration);
```
- After successful renewal, the token's new lease duration is used.
- If no lease duration is provided, it defaults to **1 hour**.

---

## 🔍 Fetching Secrets from Vault
```javascript
async function getSecrets() {
    try {
        console.log("🔄 Fetching secrets from Vault... Using AppRole authentication");

        if (!vault.token) {
            await getVaultToken();
        }
```
- It checks if the Vault token exists.
- If not, it re-authenticates using `getVaultToken()`.

---

## 📥 Reading Secrets
```javascript
const result = await vault.read("secrets/data/prod");
```
- It reads secrets using the **Key-Value (KV)** secrets engine.
- The path `secrets/data/prod` contains the production environment secrets.

---

## ✅ Returning Secrets
```javascript
return {
    NODE_ENV: result.data.data.NODE_ENV || "production",
    PORT: result.data.data.PORT || 3000,
    USER: result.data.data.USER,
    PASSWORD: result.data.data.PASSWORD,
    CONN_STR: result.data.data.CONN_STR,
    CONN_LOCAL_STR: result.data.data.CONN_LOCAL_STR,
};
```
- Secrets are returned as environment variables.
- It uses default values like `"production"` for `NODE_ENV` and `3000` for `PORT` if not found.

---

## 🧑‍💻 Error Handling
```javascript
} catch (err) {
    console.error("❌ Failed to fetch secrets from Vault:", err);
    process.exit(1);
}
```
- If any error occurs while fetching secrets, it logs the error and exits the process with a failure status.

---

## 🛎️ Exporting the Module
```javascript
module.exports = getSecrets;
```
- `getSecrets()` is exported, allowing other modules to import and use it.

---

## 🚦 Final Notes

- **Vault Policies:** Ensure the AppRole has the required policies to read secrets (`mongo-readonly` in this case).
- **Token Management:** If a token is non-renewable (`renewable: false`), the renewal will fail, and re-authentication will be needed.
- **Error Monitoring:** Implement monitoring to track renewal failures and application errors.

With this implementation, your application securely authenticates, manages tokens, and retrieves secrets from Vault using AppRole authentication. 🚀

