# 🔐 Vault: Log Out and Log In Guide

## 🚪 Log Out from Vault
To log out from Vault and remove the active token from your session, use:

```sh
unset VAULT_TOKEN
```

If you're using a custom environment variable for the token, such as `VAULT_NODE_TOKEN`, run:

```sh
unset VAULT_NODE_TOKEN
```

### ✅ Verify Logout
Check if you have successfully logged out by running:

```sh
vault token lookup
```

If you see an error like this:

```
Error making API request.
Code: 403. Errors:
* permission denied
```

It means you are successfully logged out. 🎉

### If you are still getting feedback then use this
```sh
vault token revoke -self
---

## 🔑 Log In as `node-user`
After logging out, you can log back in as a limited user with read-only access.

### 👤 Log in with `node-user`
Use the following command to authenticate:

```sh
vault login -method=userpass username=node-user password="mysecurepassword"
```

### 📌 Set the New Token
Once logged in, Vault will return a token. Set it in your environment:

```sh
export VAULT_TOKEN=<your-node-user-token>
```

Now, your Vault commands will run under `node-user` privileges, not the root user. 🚀


## 🔑 Switching Between Vault Users

### 🛠 How It Works
Vault authentication is based on the `VAULT_TOKEN` environment variable.

- When you **unset** or **revoke** a token, you lose access.
- When you **export** a new token (root or otherwise), Vault uses that token for authentication.

### 🔄 Switching Between Users

#### 1️⃣ Login as `node-user`
```sh
vault login -method=userpass username=node-user password="mysecurepassword"
export VAULT_TOKEN=<node-user-token>
```
📌 Now, you're logged in as `node-user`.

#### 2️⃣ Switch to Root User
```sh
export VAULT_TOKEN=<your-root-token>
```
📌 Now, you're acting as **root** again.

#### 3️⃣ Check Active User
```sh
vault token lookup
```
🔍 This will show the `display_name`, which tells you whether you’re using **node-user** or **root**.

---

### 🚀 How to Ensure You're Logged Out
To fully **log out** (so no token is active):
```sh
unset VAULT_TOKEN
```
Then, if you run:
```sh
vault token lookup
```
You should get an **"invalid token"** error, meaning you’re completely logged out. 🎉

