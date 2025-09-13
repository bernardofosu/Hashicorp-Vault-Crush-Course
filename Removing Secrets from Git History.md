# 🧹 Removing Secrets from Git History

When GitHub blocks your push because of leaked secrets, you have **3 main options** to clean history. 🚀


## ❌ GitHub Push Error

```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Push cannot contain secrets
remote: —— HashiCorp Vault Service Token ——————————————
remote:   commit: 8a47917d87514b044173b26d2c21f74c10da974c
remote:   path: app_role.md:118
error: failed to push some refs to 'https://github.com/bernardofosu/Hashicorp-Vault-Crush-Course.git'
```

## 🔑 Option 1) **BFG Repo-Cleaner** (no Python needed; works on Win/Linux)

⚡ Fast and simple, but requires **Java** (`java -version`).

### 🐧 Linux / macOS (bash) ✅ *(I used this and it worked)*

```bash
# 0️⃣ From your repo root
cd /path/to/your/repo

# 1️⃣ Download BFG (one-time)
curl -L -o bfg.jar https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar

# 2️⃣ Make a regex replacement file (replace any Vault token starting with hvs.)
cat > replacements.txt << 'EOF'
regex:\bhvs\.[A-Za-z0-9._-]+==>[VAULT_TOKEN_EXAMPLE]
EOF

# 3️⃣ Run BFG to replace matches across history
java -jar bfg.jar --replace-text replacements.txt

# 4️⃣ Cleanup and push rewritten history
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force-with-lease origin main
```

### 🪟 Windows (PowerShell)

```powershell
# 0️⃣ From your repo root
Set-Location "D:\Hashicorp Vault Crush Course"

# 1️⃣ Download BFG (one-time)
Invoke-WebRequest -Uri "https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar" -OutFile "bfg.jar"

# 2️⃣ Create the regex replacement file
@"
regex:\bhvs\.[A-Za-z0-9._-]+==>[VAULT_TOKEN_EXAMPLE]
"@ | Set-Content -Encoding ASCII replacements.txt

# 3️⃣ Run BFG
java -jar .\bfg.jar --replace-text .\replacements.txt

# 4️⃣ Cleanup and push rewritten history
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force-with-lease origin main
```

---

## 🐍 Option 2) **git filter-repo** (modern Git tool)

Requires **pip**.

### 🪟 Windows (PowerShell)

```powershell
# Enable pip for PyPy
& "C:\pypy3.11-v7.3.20-win64\python.exe" -m ensurepip
& "C:\pypy3.11-v7.3.20-win64\python.exe" -m pip install --upgrade pip
& "C:\pypy3.11-v7.3.20-win64\python.exe" -m pip install git-filter-repo

# Repo root
Set-Location "D:\Hashicorp Vault Crush Course"

# Replacement regex file
@"
regex:\bhvs\.[A-Za-z0-9._-]+==>[VAULT_TOKEN_EXAMPLE]
"@ | Set-Content -Encoding ASCII replacements.txt

# Run filter-repo
git filter-repo --replace-text replacements.txt

# Push rewritten history
git push --force-with-lease origin main
```

### 🐧 Linux / macOS (bash)

```bash
pipx install git-filter-repo   # or: pip install --user git-filter-repo

cd /path/to/your/repo
cat > replacements.txt << 'EOF'
regex:\bhvs\.[A-Za-z0-9._-]+==>[VAULT_TOKEN_EXAMPLE]
EOF

git filter-repo --replace-text replacements.txt
git push --force-with-lease origin main
```

---

## ✂️ Option 3) **Interactive Rebase** (surgical)

Good if **only one commit** has the secret.

### 🐧 Linux / macOS (bash)

```bash
cd /path/to/your/repo

git commit -am "Replace client_token with placeholder in docs"
git rebase -i --root   # mark 8a47917d… as `edit`
git reset HEAD^
git add .
git commit -m "Rewrite: remove leaked token from history"
git rebase --continue
git push --force-with-lease origin main
```

### 🪟 Windows (PowerShell)

```powershell
Set-Location "D:\Hashicorp Vault Crush Course"

git commit -am "Replace client_token with placeholder in docs"
git rebase -i --root   # mark 8a47917d… as `edit`
git reset HEAD^
git add .
git commit -m "Rewrite: remove leaked token from history"
git rebase --continue
git push --force-with-lease origin main
```

---

## ✅ Verify Cleanup

```bash
git grep -nE '\bhvs\.'
git log -p -S 'hvs.' -- app_role.md
```

👉 No hits = push will succeed.

---

## 🔒 Final Security Step

If that token was ever real:

```bash
vault token revoke <the-leaked-token>
# or revoke the AppRole SecretID that produced it
```

---

✨ I already tried **Option 1 (BFG on Linux)** and it worked.
Still need to test 🪟 Windows + 🐍 git filter-repo + ✂️ interactive rebase.


