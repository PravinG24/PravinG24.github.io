---
title: "HomeCloud Lab: AD-Managed Identity & SIEM-Audited Storage"
date: 2026-05-01
categories: [Cybersecurity, Systems Administration]
tags: [Active Directory, Wazuh, SIEM, Infrastructure, Automation]
---

# HomeCloud Lab: Building an Enterprise-Grade Storage Mesh 🛡️☁️

Modern cybersecurity isn't just about blocking hackers; it’s about **Visibility**, **Identity**, and **Redundancy**. 

**HomeCloud Lab** is a private infrastructure project that transforms a standard Windows VM into a centralized, AD-governed storage vault. This setup bridges my **TUF Daily Driver** and **Mobile devices** into a secure mesh network where every file interaction is audited in real-time.

---

## 🏗️ The Architecture
The lab follows a hub-and-spoke model, utilizing a "Zero-Trust" approach by keeping management interfaces and identity controllers isolated from the physical hardware.

*   **The Identity Provider:** Windows Server VM (**DC-01**) running Active Directory.
*   **The Auditor:** **Wazuh Manager** (deployed on the MSI Host).
*   **The Clients:** ASUS TUF Laptop (Daily Driver) and Android Mobile.
*   **The Network:** **Tailscale (WireGuard)** for a secure, encrypted peer-to-peer tunnel.

---

## 🔑 Phase 1: Identity & Access (Active Directory)
The core of the lab is **Centralized Identity**. By promoting the VM to a Domain Controller, I moved away from insecure local accounts and implemented **Role-Based Access Control (RBAC)**.

I created a dedicated security group for cloud access, ensuring that even if a device is connected to the network, it cannot view files without valid Domain credentials.

> **[Active Directory Users and Computers showing the 'CloudAccess' Security Group and User accounts]**
![AD Users](/assets/ad%20users.png)

---

## 💾 Phase 2: Storage Isolation & Security
In an enterprise environment, you never mix OS files with Data files. I partitioned a dedicated **F: Drive** (900GB+) to isolate my cloud storage. 

> **[Disk Management showing the C: OS partition and the dedicated F: volume]**
![Disk Partition](/assets/disk%20partition.png)

### The "Double-Gate" Permission Model
Security is handled at two layers. To access the cloud, a user must pass:
1.  **SMB Sharing Permissions:** Controls network-level entry.
2.  **NTFS Permissions:** Controls local file-level actions (Read/Write/Modify).

> **[Folder Properties showing the 'Sharing' and 'Security' (NTFS) permissions tabs]**
![Sharing Permissions](/assets/share_permissions.png)
![Security Permissions](/assets/sec_permissions.png)


---

## 🛡️ Phase 3: SIEM Integration (Wazuh)
Visibility is the heartbeat of this project. I installed the **Wazuh Agent** on the Windows VM to monitor the `S:\HomeCloud` directory.

### The Audit Configuration
I configured the agent to use **Real-time File Integrity Monitoring (FIM)**. This ensures that the moment a file is created, modified, or deleted, an alert is generated on the Wazuh Dashboard.

> **[Screenshot of the 'ossec.conf' file highlighting the FIM directory monitoring block]**
![ossec file](/assets/ossec.png)


### Agent Telemetry
Before any data was moved, I verified the heartbeat between the Windows VM and the MSI Host (Manager).

> **[Wazuh Agent Status showing the Windows VM 'Active' and 'Running']**
![Wazuh agent](/assets/Wazuh%20Agent.png)

---

## 🌐 Phase 4: The Mobile & Daily Driver Mesh
Using **Tailscale**, I created an encrypted tunnel that allows me to mount the `CloudStorage` share on my phone and my TUF laptop simultaneously. 

*   **On the daily driver laptop:** Mapped as a persistent `Z:` drive.
*   **On the Phone:** Accessed via SMB over the Tailscale 100.x.x.x network through the Cx File Explorer App.

---

## 🚀 Phase 5: Automated Sync Engine
To keep my local work safe, I developed a **PowerShell script** that utilizes **Robocopy** for multi-threaded, incremental backups. 
```powershell
# High-speed sync from TUF Daily Driver to the DC-01 Vault
foreach ($Source in $Sources) {
    robocopy $Source $TargetDir /MIR /MT:32 /R:3 /W:5 /LOG+:"C:\Users\Pravin\backup_log.txt"
}