# 📌 Difference Between result.data.data.PORT vs. result.data.NODE_ENV in Vault KV v2

The difference arises because Vault KV Secrets Engine v2 introduces an additional data layer when retrieving secrets.

## 1️⃣ KV Version 1 (KV v1)
- Data is directly stored under `result.data`.

### Example:
```js
const result = await vault.read("secrets/prod");
console.log(result.data.PORT);  // 3000
console.log(result.data.NODE_ENV);  // "production"
```
#### The values are accessed as `result.data.KEY_NAME`.In KV v1, the structure would look like this:
```sh
{
  "request_id": "...",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "PORT": 3000,
    "NODE_ENV": "production",
    "USER": "nana"
  }
}
---

## 2️⃣ KV Version 2 (KV v2) → Your Case
- Data is wrapped inside another `data` field.

### Example:
```js
const result = await vault.read("secrets/data/prod"); // Note the "data" path
console.log(result.data.data.PORT);  // 3000
console.log(result.data.data.NODE_ENV);  // "production"
```

### KV v2 JSON Structure:
```json
{
  "request_id": "...",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "data": {
      "PORT": 3000,
      "NODE_ENV": "production",
      "USER": "nana"
    },
    "metadata": {
      "created_time": "...",
      "version": 1
    }
  }
}
```
- The secrets are under `data.data` instead of just `data`.

---

## 🔄 Fix for KV v2
If you're using KV v2, update your code:
```js
const result = await vault.read("secrets/data/prod");
const secrets = result.data.data;  // Extract actual secrets

console.log(secrets.PORT);  // 3000
console.log(secrets.NODE_ENV);  // "production"
```

If you're using KV v1, use:
```js
const result = await vault.read("secrets/prod");
const secrets = result.data;

console.log(secrets.PORT);  // 3000
console.log(secrets.NODE_ENV);  // "production"
```

---

## 🛠 How to Check Your Vault KV Version
Run:
```sh
vault secrets list -detailed
```
Look at the **options** column:
- `map[version:2]` → You're using KV v2 (Use `result.data.data.KEY`).
- No version → You're using KV v1 (Use `result.data.KEY`).

---

## 📂 KV v2 Path Structure
In **KV v2**, secrets are stored under `data/`, so if you're using `vault kv get`, the correct command is:
```sh
vault kv get secrets/prod
```
But if you're using the **raw API** (or from code like Node.js), you must include **`data/`**:
```sh
vault read secrets/data/prod
```

---

## 🔍 Retrieve Secrets from KV v2
```sh
vault kv get secrets/prod
```
or explicitly in JSON format:
```sh
vault kv get -format=json secrets/prod
```

---

## 🔄 Retrieve a Specific Key
If you want to fetch just **one value** (e.g., `PORT`):
```sh
vault kv get -field=PORT secrets/prod
```
This will return:
```sh
3000
```

---

## 💡 Retrieve Previous Versions of Secrets (KV v2 Only)
### Get metadata (including versions)
```sh
vault kv metadata get secrets/prod
```

### Retrieve a specific version (e.g., version 2)
```sh
vault kv get -version=2 secrets/prod
```

---

## 🗑️ Retrieve Deleted or Destroyed Secrets
### Undelete (soft-deleted secret)
```sh
vault kv undelete -versions=1 secrets/prod
```

### Check metadata for deleted versions
```sh
vault kv metadata get secrets/prod
```

### Destroy (permanently delete)
```sh
vault kv destroy -versions=1 secrets/prod
```

