
# ✨ Vault Authentication & Secret Management with Node.js

## 1⃣ Importing and Initializing Vault Client

```js
const vault = require("node-vault")({
    endpoint: process.env.VAULT_ADDR || "http://127.0.0.1:8200",
});
```

- **`require("node-vault")`**: Imports the `node-vault` library to interact with HashiCorp Vault.
- **Vault Initialization**: Calls `node-vault` with an options object containing `endpoint`, which is the Vault server address.
- **Environment Variable Handling**: Uses `process.env.VAULT_ADDR` if set; otherwise, defaults to `http://127.0.0.1:8200` (local Vault instance).

## 2⃣ Function: `getVaultToken()` (Authenticates using AppRole)

```js
async function getVaultToken() {
    try {
        console.log("🔑 Authenticating with Vault using AppRole...");
```

- **Logs** a message indicating that authentication is starting.

```js
        const role_id = process.env.VAULT_ROLE_ID;
        const secret_id = process.env.VAULT_SECRET_ID;

        if (!role_id || !secret_id) {
            throw new Error("❌ VAULT_ROLE_ID or VAULT_SECRET_ID is missing!");
        }
```

- **Retrieving AppRole Credentials**: Fetches `VAULT_ROLE_ID` and `VAULT_SECRET_ID` from environment variables.
- **Validation Check**: If either variable is missing, an error is thrown and execution stops.

```js
        const auth = await vault.approleLogin({ role_id, secret_id });
        vault.token = auth.auth.client_token;
```

- **Logging in with AppRole**: Uses Vault's `approleLogin()` function to authenticate and obtain a `client_token`.
- **Setting Token**: Stores the obtained `client_token` in `vault.token`, allowing further authenticated API calls.

```js
        console.log("✅ Vault token obtained successfully!");
```


performs AppRole authentication with HashiCorp Vault. Here's what happens:

Sends a request to Vault’s /v1/auth/approle/login API endpoint with:
- role_id (AppRole's ID)
- secret_id (Secret associated with the role)

Vault responds with an authentication object (auth), which typically looks like this:
```ini
{
  "request_id": "abcd-1234-efgh-5678",
  "lease_id": "",
  "renewable": true,
  "lease_duration": 3600,
  "auth": {
    "client_token": "s.abcdefghijklmnopqrstuvwxyz",
    "accessor": "abcdefgh-1234-ijkl-5678-mnopqrstuvwx",
    "policies": ["default", "my-policy"],
    "token_policies": ["default", "my-policy"],
    "metadata": {
      "role_name": "my-role"
    },
    "lease_duration": 3600,
    "renewable": true
  }
}
```
How to see this JSON?

If you want to log the full response, update your code like this:
```sh
console.log("🔍 Full Vault authentication response:", JSON.stringify(auth, null, 2));
```
If you only want to see auth.auth.client_token:

console.log("🔑 Vault Token:", auth.auth.client_token);

- **Logs** success after authentication.

```js
        const leaseDuration = auth.auth.lease_duration || 3600; // Default to 1 hour if undefined
        renewToken(auth.auth.client_token, leaseDuration);
```

- **Ensures lease_duration is available** and sets a default value.
- **Calls `renewToken()`** to refresh the Vault token before it expires.

```js
        return auth.auth.client_token;
    } catch (error) {
        console.error("❌ Vault authentication failed:", error);
        process.exit(1);
    }
}
```

- **Returns the Vault token** for further use.
- **Handles Errors**: Logs failure and exits the program if authentication fails.

## 3⃣ Function: `renewToken()` (Auto-renews the Vault Token)

```js
async function renewToken(clientToken, leaseDuration) {
    const safeTimeout = Math.min((leaseDuration - 60) * 1000, 2147483647);
```

- **Calculates Safe Renewal Time**: Ensures the renewal happens 1 minute before expiration.
- **Prevents Overflow**: The maximum timeout in JavaScript is `2147483647` ms (~24.8 days). This prevents setting an invalid timeout.

```js
    console.log(`🔄 Scheduling token renewal in ${safeTimeout / 1000} seconds.`);
```

- **Logs** when the next token renewal will happen.

```js
    setTimeout(async () => {
        try {
            const renewResponse = await vault.tokenRenewSelf();
            console.log("🔄 Vault token renewed successfully!");
```

- **Schedules a Renewal**: `setTimeout()` calls Vault’s `tokenRenewSelf()` API to extend the token's lifetime.

```js
            const newLeaseDuration = renewResponse.auth.lease_duration || 3600;
            renewToken(clientToken, newLeaseDuration);
```

- **Updates Lease Duration**: If renewal succeeds, retrieves a new lease duration and reschedules the next renewal.

```js
        } catch (error) {
            console.error("⚠️ Failed to renew token:", error);
            process.exit(1);
        }
    }, safeTimeout);
}
```

- **Handles Errors**: Logs an error and exits the program if renewal fails.

## 4⃣ Function: `getSecrets()` (Fetches secrets from Vault)

```js
async function getSecrets() {
    try {
        console.log("🔄 Fetching secrets from Vault... Using AppRole authentication");
```

- **Logs** a message indicating the start of secret retrieval.

```js
        if (!vault.token) {
            await getVaultToken();
        }
```

- **Ensures Authentication**: If `vault.token` is missing, it calls `getVaultToken()` to obtain one.

```js
        const result = await vault.read("secrets/data/prod");
```

- **Retrieves Secrets**: Uses Vault’s `read()` function to get the data stored at `secrets/data/prod`.

```js
        console.log("✅ Secrets loaded successfully!");
```

- **Logs** success after retrieving secrets.

```js
        return {
            NODE_ENV: result.data.data.NODE_ENV || "production",
            PORT: result.data.data.PORT || 3000,
            USER: result.data.data.USER,
            PASSWORD: result.data.data.PASSWORD,
            CONN_STR: result.data.data.CONN_STR,
            CONN_LOCAL_STR: result.data.data.CONN_LOCAL_STR,
        };
```

- **Extracts Secrets**: Retrieves key-value pairs from Vault.
- **Sets Defaults**: If `NODE_ENV` or `PORT` are missing, defaults to `"production"` and `3000`, respectively.

```js
    } catch (err) {
        console.error("❌ Failed to fetch secrets from Vault:", err);
        process.exit(1);
    }
}
```

- **Handles Errors**: Logs failure and exits if unable to fetch secrets.

## 5⃣ Exporting the `getSecrets` Function

```js
module.exports = getSecrets;
```

- **Makes `getSecrets()` available** for import in other files.

---

## 🔹 Summary of Key Vault Commands

| Command | Purpose |
|---------|---------|
| `require("node-vault")({...})` | Initializes Vault client |
| `vault.approleLogin({ role_id, secret_id })` | Authenticates with Vault using AppRole |
| `vault.token = auth.auth.client_token` | Stores authentication token for further requests |
| `vault.tokenRenewSelf()` | Renews the Vault token before expiry |
| `vault.read("secrets/data/prod")` | Reads secrets from Vault |
| `setTimeout(() => ..., timeout)` | Schedules token renewal automatically |

## ✨ Final Thoughts

This script ensures **secure authentication**, **automatic token renewal**, and **fetching secrets safely**. It's designed to prevent token expiration issues and ensure secrets are always available when needed.

Let me know if you have any questions or need modifications! 🚀



