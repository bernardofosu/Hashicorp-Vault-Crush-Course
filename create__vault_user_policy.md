## 🎯 Step 1: Create a Vault Policy for Read-Only Access

Vault policies define what a user or application can do. Let’s create a read-only policy for your Node.js app so it can only read the database connection details but cannot modify or delete them.

### 🔐 Define the Read-Only Policy

Create a policy file called `node-app-policy.hcl`:

```hcl
path "secrets/data/prod" {
  capabilities = ["read"]
}
```

This policy:

- Grants read (`"read"`) access to `secrets/data/prod`.
- Prevents writing, deleting, or modifying secrets.

#### After writing a policy in Vault using the command:
```bash
vault policy write node-app node-app-policy.hcl
```
You can verify the policy using the following command:
```bash
vault policy list
```
This will display a list of all policies in Vault.

To view the details of a specific policy (e.g., node-app), use:
```bash
vault policy read node-app
```

### 📝 Load the Policy into Vault

Run the following command to create the policy in Vault:

```sh
vault policy write node-app node-app-policy.hcl
```

## 🎯 Step 2: Create a New Vault User with Limited Access

### 🔑 Enable Userpass Authentication (if not enabled)

```sh
vault auth enable userpass
```

### 👤 Create a User with a Limited Token

```sh
vault write auth/userpass/users/node-user password="vault123" policies="node-app"
```

- This creates a Vault user named `node-user` with the password **mysecurepassword**.
- The user is assigned the **node-app** policy, which only allows reading `secrets/data/prod`.

To check the list of users authenticated via the userpass method in Vault, use the following command:

```bash
vault list auth/userpass/users
Keys
----
node-user
```
This will display the usernames of all users registered under the userpass authentication method.

To get detailed information about a specific user (e.g., node-user):
```bash
vault read auth/userpass/users/node-user

Key                        Value
---                        -----
policies                   [node-app]
token_bound_cidrs          []
token_explicit_max_ttl     0s
token_max_ttl              0s
token_no_default_policy    false
token_num_uses             0
token_period               0s
token_policies             [node-app]
token_ttl                  0s
token_type                 default
```
#### Here's what the output means:
- 🔹 policies: [node-app] → The user has the node-app policy assigned.
- 🔹 token_policies: [node-app] → Tokens issued for this user inherit the node-app policy.
- 🔹 token_num_uses: 0 → No limit on the number of times the token can be used.
- 🔹 token_ttl: 0s → No specific TTL, meaning default settings apply.
This will show details like the policies attached to the use

## 🎯 Step 3: Login as the New User and Get a Token

Now, let’s log in as `node-user` and get a token for Node.js to use.

```sh
vault login -method=userpass username=node-user password="vault123"
```
You can get the Vault login response in JSON format by adding the -format=json flag:
```bash
vault login -method=userpass username=node-user password="vault123" -format=json
```
After login, Vault will return a token like this:

```json
{
  "request_id": "e053dcbb-e9c3-ad50-d4d4-f7",
  "lease_id": "",
  "lease_duration": 0,
  "renewable": false,
  "data": null,
  "warnings": null,
  "auth": {
    "client_token": "hvs.CAESIB3vla5f_EXZ1H-J88dnP9_zwOiL26",
    "accessor": "On60iopVt",
    "policies": [
      "default",
      "node-app"
    ],
    "token_policies": [
      "default",
      "node-app"
    ],
    "identity_policies": null,
    "metadata": {
      "username": "node-user"
    },
    "orphan": true,
    "entity_id": "bbc-0119-3649-068c-42e",
    "lease_duration": 2764800,
    "renewable": true,
    "mfa_requirement": null
  }
}
```
### ✅ 1️⃣ Set the Custom Environment Variable
Run this in your terminal before starting your app:
```sh
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_NODE_TOKEN="hvs.CAESIB3vlhMCa5f_EXZ1H-J8"
```

# 🚀 Vault Authentication Successful

You are now authenticated as **node-user**, and your Vault session will automatically use the generated token for future requests.

## 🔍 Understanding the Response

- 🔑 **token** → Your Vault authentication token (be careful not to expose it).
- 🔄 **token_renewable: true** → The token can be renewed before expiration.
- ⏳ **token_duration: 768h** → The token is valid for **32 days** unless renewed.
- 📜 **policies: ["default", "node-app"]** → This user has access to the **node-app** policy in addition to the **default** policy.

## ✅ Next Steps

### 1️⃣ Check Accessible Secrets
```bash
vault kv get secret/my-secret
```
_(This assumes the `node-app` policy allows access to `secret/my-secret`.)_

### 2️⃣ Renew the Token Manually (if needed)
```bash
vault token renew
```

### 3️⃣ Unset VAULT_TOKEN If Needed
```bash
unset VAULT_TOKEN
```
_(This ensures your session uses the newly generated token.)_

🔐 **Stay Secure:** Always follow best practices to protect your Vault credentials! 🚀


Copy the `client_token` and use it in your Node.js app.

## 🎯 Step 4: Use the Token in Your Node.js App

Instead of using the root token, update your `config.js` to use the new token dynamically.

```js
const vault = require("node-vault")({
    endpoint: process.env.VAULT_ADDR || "http://127.0.0.1:8200",
    token: process.env.VAULT_TOKEN, // Use node-user's token
});

async function getSecrets() {
    try {
        console.log("🔄 Fetching secrets from Vault...");

        // KV v2 requires "data/" in the path
        const result = await vault.read("secrets/data/prod");

        console.log("✅ Secrets loaded successfully!");

        return result.data.data; // Extract actual secrets
    } catch (err) {
        console.error("❌ Failed to fetch secrets from Vault:", err);
        process.exit(1);
    }
}

module.exports = getSecrets;
```

## 🎯 Step 5: Store and Use the Limited Token

Instead of exporting the **root** token, store and use the **new user's token**.

```sh
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=hvs.FAKE-TOKEN-123456  # Use node-user's token
```

Now, when your **Node.js app** fetches secrets, it will:
✅ **Only have read access** to `secrets/data/prod`.
❌ **Cannot delete, write, or modify** any secret.

## 🎯 Step 6: Test the Security

### ✅ Allowed: Read Secrets

```sh
vault kv get secrets/prod
```

### ❌ Denied: Writing a Secret

```sh
vault kv put secrets/prod NEW_KEY="test"
```

You should get a **403 permission denied** error. 🎉

## 🔥 Summary

1️⃣ Created a **read-only policy** for the `secrets/data/prod` path.
2️⃣ Created a **new user (`node-user`)** with only read access.
3️⃣ Generated a **limited token** for the Node.js app.
4️⃣ Updated the **app to use the limited token**, **not the root token**.
5️⃣ Verified that the user **can only read**, preventing accidental secret modifications.


# 🔐 Vault User Login, Policy Testing, and Updates

## 🎯 Step 1: Log in as the New User
To authenticate as the new user (`node-user`), run:

```sh
vault login -method=userpass username=node-user password="mysecurepassword"
```

After logging in, you should see a response with a **client token** like this:

```json
{
  "auth": {
    "client_token": "hvs.XXXXX-XXXXX",
    "policies": ["default", "node-app"],
    "lease_duration": 2764800,
    "renewable": true
  }
}
```

🔑 **Copy the `client_token`** and use it in API requests or applications.

---

## 🔍 Step 2: Test the User's Policies

### ✅ **Test Read Access**
If the user has read access, this command should work:

```sh
vault kv get secrets/prod
```

Expected output:

```sh
======= Metadata =======
Key       Value
---       -----
PORT      3000
USER      nana
```

### ❌ **Test Write Access (Should Fail if Read-Only)**
If `node-user` only has read access, trying to write will fail:

```sh
vault kv put secrets/prod NEW_SECRET="test-value"
```

Expected error:

```sh
Error writing data to secrets/prod: permission denied
```

### ❌ **Test Policy Listing (May Be Denied)**
Users with limited access might be unable to list policies:

```sh
vault policy list
```

Expected error:

```sh
Error listing policies: permission denied
```

---

## 🔧 Step 3: Update the User's Policies
If you need to grant additional permissions, modify the policy.

### 1️⃣ **Edit the Policy File**
Open `node-app-policy.hcl` and update it:

```hcl
path "secrets/data/prod" {
  capabilities = ["read", "create", "update"]
}
```

This allows the user to:
- ✅ **Read** secrets
- ✅ **Create new** secrets
- ✅ **Update existing** secrets

### 2️⃣ **Apply the Updated Policy**
Run the following command:

```sh
vault policy write node-app node-app-policy.hcl
```

### 3️⃣ **Test Write Access Again**
Now, retry writing a secret:

```sh
vault kv put secrets/prod NEW_SECRET="test-value"
```

If successful, verify it with:

```sh
vault kv get secrets/prod
```

---

## 🔄 Step 4: Assign a New Policy to the User
If you want to **attach a different policy**, run:

```sh
vault write auth/userpass/users/node-user policies="new-policy"
```

To verify the applied policies:

```sh
vault token lookup
```

Look at the `policies` field to confirm the assigned policies.

---

## 🎉 Summary
1️⃣ **Login** as `node-user` with `vault login` 🔑
2️⃣ **Test read & write permissions** with `vault kv get/put` 🔍
3️⃣ **Modify policy if needed** (`vault policy write node-app`) 🔧
4️⃣ **Attach new policies** with `vault write auth/userpass/users/node-user` 🏗️
5️⃣ **Verify changes** with `vault token lookup` ✅

Now, your user has the correct access! 🚀



