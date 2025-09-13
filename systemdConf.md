# 📦 HashiCorp Vault Service File Explained

This markdown provides a detailed explanation of a typical Vault systemd service configuration (`/usr/lib/systemd/system/vault.service`).

---

## 🧱 **[Unit] Section**

```ini
[Unit]
Description="HashiCorp Vault - A tool for managing secrets"
Documentation=https://developer.hashicorp.com/vault/docs
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault.d/vault.hcl
StartLimitIntervalSec=60
StartLimitBurst=3
```

### 🔎 **Explanation:**
- **Description:** A brief overview of the service.
- **Documentation:** Provides a link to the official documentation for further reference.
- **Requires & After:** Ensures Vault only starts once the network is online.
- **ConditionFileNotEmpty:** Prevents the service from starting if the Vault configuration file (`vault.hcl`) is empty or missing.
- **StartLimitIntervalSec:** If Vault fails to start, it limits retries to avoid exhausting system resources. (60 seconds)
- **StartLimitBurst:** Allows a maximum of 3 attempts within the interval.

---

## ⚙️ **[Service] Section**

```ini
[Service]
Type=notify
EnvironmentFile=/etc/vault.d/vault.env
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
```

### 🔎 **Explanation:**
- **Type=notify:** Vault uses systemd notifications to signal when it is ready.
- **EnvironmentFile:** Loads environment variables from `/etc/vault.d/vault.env`, useful for external configuration and secrets.
- **User & Group:** Runs Vault under the `vault` user and group, improving security by minimizing privileges.
- **ProtectSystem=full:** Provides read-only access to most parts of the filesystem, except essential locations.
- **ProtectHome=read-only:** Ensures Vault cannot write to user home directories, reducing the attack surface.
- **PrivateTmp=yes:** Provides Vault with a separate temporary storage (`/tmp`) to prevent interference from other processes.
- **PrivateDevices=yes:** Blocks access to device files to limit damage in case of a compromise.
- **SecureBits=keep-caps:** Ensures Vault retains its necessary Linux capabilities.
- **AmbientCapabilities=CAP_IPC_LOCK:** Allows Vault to lock memory, preventing sensitive data from being swapped to disk.
- **CapabilityBoundingSet:** Restricts capabilities to only `CAP_SYSLOG` and `CAP_IPC_LOCK` for security.
- **NoNewPrivileges=yes:** Prevents Vault from gaining additional privileges, mitigating privilege escalation attacks.

---

## 🚀 **Conclusion**

This service configuration follows best practices for securing and managing Vault. It's designed for stability, security, and reliable startup. If you need further adjustments or explanations, feel free to ask! 😊


# 🚀 Difference Between /usr and /etc for Services

The difference between `/usr` and `/etc` for services lies in their purpose and the type of files they store:

## 📂 **/usr/lib/systemd/system/**

- **Purpose:** Contains **default** service unit files that come with software packages installed on your system (e.g., Vault, Docker, Nginx).
- **Managed by:** The system's package manager (e.g., `apt`, `yum`, `dnf`).
- **Persistence:** Files here are **overwritten** when the software is updated or reinstalled.
- **Example:**
  ```
  /usr/lib/systemd/system/vault.service
  ```
- **When to Use:**
  - When the software provides its own service file.
  - **Avoid direct editing**; instead, use `/etc/systemd/system/` for customizations.

---

## 🛠 **/etc/systemd/system/**

- **Purpose:** Contains **user-defined** or customized service unit files that override the defaults from `/usr/lib/systemd/system/`.
- **Managed by:** System administrators.
- **Persistence:** Changes are **retained** even after package updates.
- **Example:**
  ```
  /etc/systemd/system/vault.service
  ```
- **When to Use:**
  - If you want to modify service behavior (e.g., environment variables, resource limits).
  - For custom services not installed via a package manager.

---

## ✅ **Key Differences Summary**

| **Feature**                 | **/usr/lib/systemd/system/**                     | **/etc/systemd/system/**                      |
|-----------------------------|-------------------------------------------------|---------------------------------------------|
| **Purpose**                 | Stores default service files from packages     | Stores custom or overridden service files  |
| **Managed by**              | Package Manager                                | System Administrator                        |
| **Persistence**             | Overwritten on package updates                 | Remains unchanged during updates           |
| **Use Case**                | For standard, unmodified services              | For service customization or local services |
| **Example**                 | `/usr/lib/systemd/system/vault.service`        | `/etc/systemd/system/vault.service`        |

---

🔎 **Note:**
If you want to customize Vault's service, it's best to create a copy or a symbolic link in `/etc/systemd/system/` rather than editing the original file in `/usr/lib/systemd/system/`. Let me know if you'd like guidance on that! 😊




## ✅ **Option 3: Run Vault as a Systemd Service**

If you prefer running Vault as a background service using `systemd`, ensure Vault is properly set up as a service.

### 📌 **Create or Edit the Service File:**
```bash
sudo nano /etc/systemd/system/vault.service
```

### 📌 **Example service file:**
check vault location
```sh
~$ which vault
/usr/bin/vault
```
```ini
[Unit]
Description=HashiCorp Vault - A tool for managing secrets
Documentation=https://www.vaultproject.io/docs/
After=network.target

[Service]
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
Restart=always
User=vault
Group=vault
PermissionsStartOnly=true
LimitNOFILE=65536
ProtectSystem=full
ProtectHome=read-only
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
SecureBits=keep-caps

[Install]
WantedBy=multi-user.target
```

### 📌 **Enable and Start the Service:**
If vault is part of systemd
```sh
workstation@Nana-Kwasi-Fosu:~$ sudo systemctl status vault
Warning: The unit file, source configuration file or drop-ins of vault.service changed on disk. Run 'systemctl daemon-reload' to reload units.
○ vault.service - "HashiCorp Vault - A tool for managing secrets"
     Loaded: loaded (/usr/lib/systemd/system/vault.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: https://developer.hashicorp.com/vault/docs
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault
```
# 🖥️ HashiCorp Vault Systemd Service Guide

This section explains how to create and manage a systemd service for HashiCorp Vault, allowing it to run in the background like a system-level service. Let's break it down step-by-step.

## 🛠 1. Create or Edit the Service File

```bash
sudo nano /etc/systemd/system/vault.service
```

- **sudo**: Runs the command with administrator privileges.
- **nano**: Opens the Nano text editor to create or edit a file.
- **/etc/systemd/system/vault.service**: This is the location for custom systemd service files. By creating a `vault.service` file, you define how Vault should be managed by systemd.

## 📦 2. Example Service File Breakdown

This file consists of three main sections: **[Unit]**, **[Service]**, and **[Install]**.

### 🔎 [Unit] Section

```ini
[Unit]
Description=HashiCorp Vault - A tool for managing secrets
Documentation=https://www.vaultproject.io/docs/
After=network.target
```

- **Description**: Provides a brief explanation of the service.
- **Documentation**: Offers a link to the official Vault documentation.
- **After=network.target**: Ensures that Vault starts only after the network is up.

### ⚙️ [Service] Section

```ini
[Service]
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
Restart=always
User=vault
Group=vault
PermissionsStartOnly=true
LimitNOFILE=65536
ProtectSystem=full
ProtectHome=read-only
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
SecureBits=keep-caps
```

- **ExecStart**: Specifies the command to run Vault with its configuration file.
- **Restart=always**: Restarts Vault if it crashes.
- **User and Group**: Ensures Vault runs under the `vault` user and group for security.
- **PermissionsStartOnly=true**: Ensures only the start phase runs with elevated privileges.
- **LimitNOFILE=65536**: Increases the file descriptor limit, ensuring Vault can handle a large number of connections.
- **ProtectSystem=full**: Restricts Vault’s access to system files, enhancing security.
- **ProtectHome=read-only**: Prevents Vault from modifying home directories.
- **CapabilityBoundingSet**: Grants specific Linux capabilities.
- **NoNewPrivileges=yes**: Prevents privilege escalation.
- **SecureBits=keep-caps**: Ensures that capabilities are retained without escalation.

### 🚀 [Install] Section

```ini
[Install]
WantedBy=multi-user.target
```

- **WantedBy=multi-user.target**: Ensures Vault starts in multi-user mode, typically used for server environments.

## ✅ 3. Enable and Start the Service

Once you've saved the service file, run the following commands:

### Reload Systemd to Apply Changes

```bash
sudo systemctl daemon-reload
```

- Informs systemd about the new or modified service file.

### Enable Vault to Start on Boot

```bash
sudo systemctl enable vault
```

- Ensures Vault starts automatically when the system reboots.

### Start Vault Service

```bash
sudo systemctl start vault
```

- Immediately starts the Vault server using the configuration.

### Check Vault Status

```bash
sudo systemctl status vault
```

- Displays detailed information about Vault's status, including logs and any error messages.

That's it! You now have Vault running as a systemd service. 🚀


