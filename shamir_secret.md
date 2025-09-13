# 🔐 Shamir’s Secret Sharing (SSS) in Vault

Shamir’s Secret Sharing (SSS) is a cryptographic algorithm used by HashiCorp Vault to protect the **Master Key** that encrypts Vault’s data. It allows the key to be split into multiple shares, requiring a **threshold** number of shares to reconstruct the key and unseal Vault.

## 🤔 Why Does Vault Use Shamir’s Secret Sharing?
Vault does **not** store the Master Key anywhere. Instead, when Vault is **initialized**:

- The Master Key is **split** into **N** unseal keys (shares).
- A **threshold (T)** number of keys is required to unseal Vault (e.g., **3 out of 5**).
- These keys are distributed to **trusted administrators**, who must collaborate to unseal Vault.

## 🎭 Example of Shamir’s Secret Sharing
Imagine you have a **treasure chest** that requires **3 out of 5 keys** to open:

- 🏴‍☠️ The **chest** represents **Vault**.
- 🔑 The **keys** represent **unseal keys**.
- Any **3 keys together** can open the chest, but **fewer than 3** cannot.

This ensures:
✅ **Security**: No single person can access the data alone.
✅ **Redundancy**: If some keys are lost, the system can still be unlocked.
✅ **Trust Model**: It enforces **collaboration** among administrators.

## 🔓 How Unsealing Works
Each time **Vault restarts**, it is **sealed** and requires **manual unsealing**:

1️⃣ An admin enters **Unseal Key 1** → **Progress: 1/3**
2️⃣ Another admin enters **Unseal Key 2** → **Progress: 2/3**
3️⃣ A third admin enters **Unseal Key 3** → **Vault is unsealed** 🎉

## 🚀 Alternatives to Shamir’s Secret Sharing
In **production**, manually unsealing Vault is **inconvenient**. You can enable **Auto-Unseal** using:

- ☁️ **AWS KMS** (Key Management Service)
- ☁️ **Google Cloud KMS**
- 🔒 **HSM (Hardware Security Module)**

Using these methods, Vault **automatically unseals** upon restart, improving efficiency and security. 🔐🚀

