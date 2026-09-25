# Cloud Security Risk Assessment: GRC Lab Simulation

This project simulates a real-world cloud security risk assessment from a **Governance, Risk, and Compliance (GRC)** perspective. Using a simple Azure-hosted Windows virtual machine, I evaluated the system's setup, identified assets, mapped realistic threats and vulnerabilities, scored risks using a $5 \times 5$ matrix, and aligned security controls to **NIST CSF** and **CIS Critical Security Controls**.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Lab Setup & Prerequisites](#lab-setup--prerequisites)
- [Hands-On Lab Steps](#hands-on-lab-steps)
  - [Phase 1: Setting Up the Lab](#phase-1-setting-up-the-lab)
  - [Phase 2: Finding Assets, Threats, and Vulnerabilities](#phase-2-finding-assets-threats-and-vulnerabilities)
  - [Phase 3: Calculating Risk Scores](#phase-3-calculating-risk-scores)
  - [Phase 4: Mapping Controls to Security Frameworks](#phase-4-mapping-controls-to-security-frameworks)
- [Final Deliverable: GRC Risk Assessment Report](#final-deliverable-grc-risk-assessment-report)
  - [1. Executive Summary](#1-executive-summary)
  - [2. Scope & Target Environment](#2-scope--target-environment)
  - [3. Asset, Threat, and Vulnerability Breakdown](#3-asset-threat-and-vulnerability-breakdown)
  - [4. Risk Register](#4-risk-register)
  - [5. Framework Mapping Table](#5-framework-mapping-table)
  - [6. Recommendations & Action Plan](#6-recommendations--action-plan)
  - [7. Conclusion & Risk Summary](#7-conclusion--risk-summary)

---

## Project Overview

The goal of this project is to practice how IT and GRC analysts assess security risk in enterprise cloud environments. Instead of running active penetration tests or exploits, this lab focuses on auditing system configurations, finding security weaknesses, calculating risk scores based on likelihood and impact, and recommending standard, auditable security fixes mapped to NIST and CIS frameworks.

---

## Lab Setup & Prerequisites

* **Cloud Platform:** Microsoft Azure (Free account)
* **Target System:** 1x Azure Virtual Machine (`Windows Server 2022 Datacenter`)
* **Tools Used:** Azure Portal, PowerShell, Windows Event Viewer, Excel / Google Sheets
* **Method:** Non-intrusive configuration audit and risk review

---

## Hands-On Lab Steps

### Phase 1: Setting Up the Lab

#### 1. Azure Account Setup
1. Sign up for an Azure free account at [azure.microsoft.com/free](https://azure.microsoft.com/free).
2. Complete the standard phone and credit card verification steps (you will not be charged on the free tier).
3. Log in to the [Azure Portal](https://portal.azure.com).

<p align="center">
  <img src="https://github.com/user-attachments/assets/6e327028-6ad6-418d-84d6-c00b7a224549" alt="Azure Portal Sign-up" width="850"/>
</p>

#### 2. Create the Resource Group
1. Search for **Resource groups** in the top search bar and click **Create**.
2. Enter the following settings:
   * **Subscription:** `Default`
   * **Resource Group Name:** `GRC-Lab`
   * **Region:** Closest region to you (e.g., `West US 2`)
3. Click **Review + Create**, then click **Create**.

<p align="center">
  <img src="https://github.com/user-attachments/assets/8ec65c72-0993-4b41-9f0e-6e7a183fb142" alt="Azure Resource Groups" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/c7fbe6e8-8091-4417-9588-8b05e10d2ed8" alt="Creating GRC-Lab Resource Group" width="600"/>
</p>

#### 3. Deploy the Virtual Machine
Create a simple target VM inside the `GRC-Lab` resource group:

<p align="center">
  <img src="https://github.com/user-attachments/assets/2dfbdc3c-f06c-493d-a29d-60df6c939f0b" alt="Virtual Machines Search" width="700"/>
</p>

* **Configuration Details:**
  * **Subscription:** `Default`
  * **Resource Group:** `GRC-Lab`
  * **Virtual Machine Name:** `GRC-WIN-VM01`
  * **Region:** Same as resource group (e.g., `West US 2`)
  * **Image:** `Windows Server 2022 Datacenter - x64 Gen2`
  * **Size:** `Standard_DC1s_v3` (keeps lab costs low)
* **Administrator Account:**
  * **Username:** `azureuser`
  * **Password:** Password of your choice (saved securely)
* **Inbound Port Rules:**
  * **Public Inbound Ports:** Select *Allow selected ports*
  * **Allowed Ports:** `RDP (3389)` *(opened intentionally to simulate public exposure)*
* Click **Review + Create**, then click **Create**.

#### 4. Test RDP Connection
1. Open `GRC-WIN-VM01` in the portal, click **Connect** $\rightarrow$ **RDP**, and download the connection file.
2. Sign in using the `azureadmin` credentials to verify that you can reach the desktop.

<p align="center">
  <img src="https://github.com/user-attachments/assets/72b52203-1842-4f57-96df-9a5f7f2ad5e9" alt="Azure VM RDP Connect Blade" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/9ab2bd03-5c58-47a2-8479-4040e90e8636" alt="Windows Server Desktop" width="600"/>
</p>

---

### Phase 2: Finding Assets, Threats, and Vulnerabilities

A standard GRC assessment breaks down into three key questions:
* **Assets:** What are we protecting?
* **Threats:** What could realistically harm or compromise it?
* **Vulnerabilities:** What weaknesses make that harm possible?

#### 1. Check the Operating System & Host
Under `GRC-WIN-VM01` $\rightarrow$ **Settings** $\rightarrow$ **Properties**, check the OS version, patch level, and general machine settings.

<p align="center">
  <img src="https://github.com/user-attachments/assets/68db4218-d543-46c5-ab43-07a4a37fb3eb" alt="Azure VM Properties" width="700"/>
</p>

#### 2. Check Network Ports and Inbound Access
Go to **Networking** $\rightarrow$ **Network Settings** to review the Network Security Group (NSG) rules:
* Look for open ports that face the public internet.
* Notice that port `3389` (RDP) allows inbound traffic from `Any` source (`*` or `Internet`), which exposes the login screen to automated bots.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9d4c7b75-1067-42e5-bf69-c5f1858ffaf2" alt="NSG Inbound Port Rules" width="850"/>
</p>

#### 3. Audit Local Accounts and Privileges
Check what admin accounts exist on the machine:
1. Go to **Operations** $\rightarrow$ **Run Command** $\rightarrow$ `RunPowerShellScript`.
2. Run `Get-LocalUser` to list all local accounts.

<p align="center">
  <img src="https://github.com/user-attachments/assets/3754e4db-a679-4f5c-ab13-71d509a4c736" alt="Running Get-LocalUser in Run Command" width="850"/>
</p>

3. Inside the VM desktop session, open **Computer Management** $\rightarrow$ **Local Users and Groups** $\rightarrow$ **Groups** $\rightarrow$ **Administrators** to confirm which accounts have full control.

<p align="center">
  <img src="https://github.com/user-attachments/assets/cfe4a50b-4390-4733-8539-691a1b90b736" alt="Local Users and Groups" width="700"/>
</p>

#### 4. Review System Logging and Monitoring
Check whether the system is set up to detect suspicious activity:
* **Cloud Level:** In the Azure portal, click **Monitoring** on the left menu for `GRC-WIN-VM01` to see if logs are sent to a monitoring workspace.
* **OS Level:** On the VM, open **Event Viewer** $\rightarrow$ **Windows Logs** $\rightarrow$ **Security** to confirm whether login attempts and system events are being recorded locally.

<p align="center">
  <img src="https://github.com/user-attachments/assets/3aee0dab-6d25-4a66-805a-4b190d29dcaf" alt="Azure Monitoring Unconfigured" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/ef220f56-ea90-4b36-8acc-736169beace0" alt="Event Viewer Security Logs" width="600"/>
</p>

---

### Phase 3: Calculating Risk Scores

Rather than guessing how serious an issue is, risk assessments use a simple formula to prioritize what needs fixing first:

$$\text{Risk Score} = \text{Likelihood} \times \text{Impact}$$

#### Scoring Guidelines

| Rating | Likelihood (How likely is it?) | Impact (How bad would it be?) |
| :---: | :--- | :--- |
| **1** | **Rare:** Very unlikely to happen; needs unusual conditions. | **Minimal:** Little to no real effect on operations. |
| **2** | **Unlikely:** Possible, but requires high effort or insider access. | **Minor:** Minor issue; easily fixed with no sensitive data lost. |
| **3** | **Possible:** Known attack type; frequently seen online. | **Moderate:** Some disruption or localized system compromise. |
| **4** | **Likely:** System is directly exposed and easy to target. | **High:** Severe impact; administrative access lost or data compromised. |
| **5** | **Almost Certain:** Zero protection on an active, open target. | **Critical:** Total takeover of the system or cloud account. |

---

### Phase 4: Mapping Controls to Security Frameworks

Security recommendations need to point back to recognized standards so auditors and leadership can see why a fix is necessary.

* **NIST CSF:** Focuses on the core functions: **Identify**, **Protect**, **Detect**, **Respond**, and **Recover**.
* **CIS Controls:** A list of direct, technical steps to block the most common cyber attacks.
* **Control Categories:**
  * **Preventive:** Stops the attack before it happens (e.g., firewall rules, MFA).
  * **Detective:** Alerts you when suspicious activity happens (e.g., log monitoring, SIEM).
  * **Corrective:** Fixes the issue after it happens (e.g., patching, backups).

---

## Final Deliverable: GRC Risk Assessment Report

### 1. Executive Summary

This report outlines the findings from a cloud security risk assessment performed on an Azure virtual machine. The goal was to audit the environment, identify key assets and realistic threats, score risks, and suggest standard security controls mapped to industry frameworks.

The overall risk posture for this system is rated **Medium to High**. The biggest issues found were an open RDP management port exposed to the internet, no multi-factor authentication (MFA) on the admin account, default operating system settings with no extra hardening, and no centralized log forwarding. These weaknesses make the system vulnerable to brute-force attacks and credential theft.

### 2. Scope & Target Environment

| Setting | Details |
| :--- | :--- |
| **Scope** | Single cloud virtual machine |
| **Cloud Provider** | Microsoft Azure |
| **Resource Group** | `GRC-Lab` |
| **Virtual Machine** | `GRC-WIN-VM01` |
| **Operating System** | Windows Server 2022 Datacenter |
| **Remote Access** | Remote Desktop Protocol (RDP) on Port 3389 |
| **Internet Facing** | Yes (Public IP assigned) |
| **Admin Account** | `azureuser` (Local Admin) |
| **Assessment Type** | Non-intrusive GRC configuration review |

### 3. Asset, Threat, and Vulnerability Breakdown

```text
   [ Internet ] 
        │
   (Port 3389)  <-- Brute-Force / Credential Stuffing
        ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ Network Security Group (NSG)                                │
 │ Rule: Allow Inbound from Any (0.0.0.0/0)                    │
 └──────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ Azure VM: GRC-WIN-VM01                                      │
 │  ├── Account: azureadmin (Password only / No MFA)           │
 │  ├── Logs: Stored locally only (No SIEM)                    │
 │  └── Config: Default Windows Server settings                │
 └─────────────────────────────────────────────────────────────┘
```

| Asset | What It Does | Threat | Vulnerability |
| :--- | :--- | :--- | :--- |
| **Public RDP (Port 3389)** | Remote access into the VM | Brute-force attacks and password guessing | NSG allows traffic from any IP address |
| **Admin Account** | Full control over the Windows OS | Credential theft or compromised password | Only protected by password (No MFA) |
| **Virtual Machine OS** | Runs server roles and services | Known exploits and unpatched bugs | Default build with no custom hardening |
| **Windows Security Logs** | Records logins and system events | Attacker activities go unnoticed | Logs stay on the machine; no centralized SIEM |
| **Azure Subscription** | Controls cloud billing and resources | Unauthorized configuration changes | Single admin account without role limits |

### 4. Risk Register

$$\text{Risk Score} = \text{Likelihood} \times \text{Impact}$$

| Asset | Threat | Likelihood | Impact | Risk Score | Risk Level | Reason |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **RDP Port** | Brute-force attack | 4 | 4 | **16** | **High** | Port 3389 is wide open to the internet and constantly targeted by bots. |
| **Admin Account** | Password compromise | 3 | 5 | **15** | **High** | If this password leaks, the attacker gets full control over the machine. |
| **Windows OS** | Unpatched exploits | 3 | 4 | **12** | **Medium** | Default OS installations often have known vulnerabilities that need patching. |
| **Azure Account** | Unauthorized cloud changes | 2 | 5 | **10** | **Medium** | If the cloud login is compromised, the attacker can alter or delete resources. |
| **Event Logs** | Undetected intrusion | 3 | 3 | **9** | **Medium** | Local logs can easily be cleared or ignored without a centralized dashboard. |

### 5. Framework Mapping Table

| Finding | Recommended Control | Control Type | NIST CSF Mapping | CIS Controls Mapping |
| :--- | :--- | :---: | :--- | :--- |
| **Open RDP Port** | Restrict NSG rules to specific management IPs or use Azure Bastion. | Preventive | **PR.AC-5:** Network access is controlled | **CIS 4.4:** Restrict external-facing ports |
| **Single-Factor Admin** | Enforce Multi-Factor Authentication (MFA) and strong password rules. | Preventive | **PR.AC-7:** Multi-factor authentication is used | **CIS 6.3:** Require MFA for remote access |
| **Default OS Settings** | Apply regular OS patch cycles and CIS benchmark hardening. | Corrective | **PR.IP-12:** Vulnerability management plan | **CIS 7.4:** Automated patch management |
| **Broad Azure Access** | Use Role-Based Access Control (RBAC) to enforce least privilege. | Preventive | **PR.AC-4:** Access permissions are managed | **CIS 5.4:** Restrict administrative privileges |
| **Local-Only Logging** | Stream system and security logs to Azure Log Analytics or a SIEM. | Detective | **DE.CM-1:** Networks and systems are monitored | **CIS 8.2:** Centralize audit log storage |

### 6. Recommendations & Action Plan

1. **Step 1 (Immediate):** Update the Azure NSG to remove the open `0.0.0.0/0` rule on port 3389. Restrict access strictly to a known management IP address.
2. **Step 2 (Immediate):** Enable MFA on the Azure administrative account and require strong passphrases.
3. **Step 3 (Short-Term):** Replace public RDP access entirely with Azure Bastion or an encrypted VPN tunnel.
4. **Step 4 (Short-Term):** Turn on automatic updates and follow the CIS Windows Server benchmark to disable unneeded services.
5. **Step 5 (Ongoing):** Connect the VM's logs to an Azure Log Analytics workspace or Microsoft Sentinel so failed logins trigger automated alerts.

### 7. Conclusion & Risk Summary

Even though this lab was built around a single virtual machine, it demonstrates the most common security gaps found in real cloud environments: open management ports, password-only logins, and a lack of centralized log collection. By closing public RDP access, turning on MFA, and forwarding logs to a central workspace, the risk level of this deployment drops from **High** down to an acceptable, defensible **Low**.
