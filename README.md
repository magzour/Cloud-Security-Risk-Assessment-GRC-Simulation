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
  * **Username:** `azureadmin`
  * **Password:** Strong password (saved securely)
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
| **Admin Account** | `azureadmin` (Local Admin) |
| **Assessment Type** | Non-intrusive GRC configuration review |

### 3. Asset, Threat, and Vulnerability Breakdown
