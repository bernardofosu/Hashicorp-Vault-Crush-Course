# 🔐 Unsealing Vault with Shamir's Secret Sharing

Your Vault server is currently **sealed** and using **Shamir's Secret Sharing** for unsealing. Since the threshold is set to **3**, you need to enter **three different unseal keys** to unseal it.

## 🚀 Steps to Unseal Vault

### 1️⃣ Run the Unseal Command
Run the following command to start the unsealing process:
When Exit the server its sealed the vault and when you start the server you to unsealed it.
```bash
vault status
vault operator unseal
```
You'll be prompted to enter an **unseal key**.

### 2️⃣ Enter Three Different Unseal Keys
Repeat the unseal command with **three different** unseal keys:

```bash
vault operator unseal <unseal-key-1>
vault operator unseal <unseal-key-2>
vault operator unseal <unseal-key-3>
```
🔹 Replace `<unseal-key-1>`, `<unseal-key-2>`, and `<unseal-key-3>` with the **actual unseal keys**.

### 3️⃣ Verify Vault's Status
Check if Vault is successfully unsealed:

```bash
vault status
```
✅ If unsealing was successful, the output should indicate:
- `Sealed: false` (Vault is now unsealed)

🎉 **Congratulations! Your Vault server is now unsealed and ready to use.**

