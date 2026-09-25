# Cloud Security Risk Assessment: GRC Simulation Lab

A simulated enterprise cloud security risk assessment executed from a **Governance, Risk, and Compliance (GRC)** perspective. This project models an audit of an Azure-hosted cloud asset, evaluating configuration posture, identifying threats and vulnerabilities, scoring risks using a quantitative 5×5 matrix, and aligning mitigation controls with **NIST CSF** and **CIS Critical Security Controls**.

---

## Table of Contents
- [Executive Overview](#executive-overview)
- [Lab Environment & Prerequisites](#lab-environment--prerequisites)
- [Hands-On Lab Walkthrough](#hands-on-lab-walkthrough)
  - [Phase 1: Environment Provisioning](#phase-1-environment-provisioning)
  - [Phase 2: Asset, Threat, and Vulnerability Assessment](#phase-2-asset-threat-and-vulnerability-assessment)
  - [Phase 3: Quantitative Risk Scoring](#phase-3-quantitative-risk-scoring)
  - [Phase 4: Framework Alignment](#phase-4-framework-alignment)
- [Formal GRC Deliverable: Cloud Security Assessment Report](#formal-grc-deliverable-cloud-security-assessment-report)
  - [1. Executive Summary](#1-executive-summary)
  - [2. Scope & Target Architecture](#2-scope--target-architecture)
  - [3. Identified Assets & Threat Profiles](#3-identified-assets--threat-profiles)
  - [4. Enterprise Risk Register](#4-enterprise-risk-register)
  - [5. Framework Control Mapping](#5-framework-control-mapping)
  - [6. Strategic Mitigation Roadmap](#6-strategic-mitigation-roadmap)
  - [7. Overall Posture Assessment](#7-overall-posture-assessment)

---

## Executive Overview

The purpose of this project is to simulate real-world GRC and security operations workflows in an enterprise cloud setting. By assessing a single cloud-hosted virtual machine, this assessment evaluates the attack surface, identifies critical assets, examines threat scenarios, quantifies organizational exposure using a standard $5 \times 5$ risk scoring methodology, and maps defensible controls against industry standards (**NIST CSF v1.1/2.0** and **CIS Controls v8**).

---

## Lab Environment & Prerequisites

* **Cloud Platform:** Microsoft Azure (Free-tier eligible subscription)
* **Compute Target:** 1x Azure Virtual Machine (`Windows Server 2022 Datacenter`)
* **Tooling:** Azure Portal, PowerShell, Windows Event Viewer, Excel / Google Sheets
* **Auditing Methodology:** Non-exploitative configuration review and security posture assessment

---

## Hands-On Lab Walkthrough

### Phase 1: Environment Provisioning

#### 1. Account Setup & Portal Access
1. Sign up for an Azure account at [azure.microsoft.com/free](https://azure.microsoft.com/free).
2. Complete SMS and payment method verification (free trial avoids charges unless upgraded).
3. Authenticate to the [Azure Management Portal](https://portal.azure.com).

<p align="center">
  <img src="https://github.com/user-attachments/assets/6e327028-6ad6-418d-84d6-c00b7a224549" alt="Azure Portal Sign-up Screen" width="850"/>
</p>

#### 2. Resource Group Creation
1. Search for **Resource groups** in the top search bar and select **Create**.
2. Specify deployment parameters:
   * **Subscription:** `Default`
   * **Resource Group Name:** `GRC-Lab`
   * **Region:** Nearest deployment zone (e.g., `West US 2`)
3. Select **Review + Create** and finalize the resource group.

<p align="center">
  <img src="https://github.com/user-attachments/assets/8ec65c72-0993-4b41-9f0e-6e7a183fb142" alt="Azure Resource Groups blade" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/c7fbe6e8-8091-4417-9588-8b05e10d2ed8" alt="Configuring GRC-Lab Resource Group" width="600"/>
</p>

#### 3. Virtual Machine Deployment
Deploy a standard target VM inside the newly created resource group:

<p align="center">
  <img src="https://github.com/user-attachments/assets/2dfbdc3c-f06c-493d-a29d-60df6c939f0b" alt="Virtual Machines Search" width="700"/>
</p>

* **Basic Configuration:**
  * **Subscription:** `Default`
  * **Resource Group:** `GRC-Lab`
  * **Virtual Machine Name:** `GRC-WIN-VM01`
  * **Region:** Same as Resource Group (e.g., `West US 2`)
  * **Image:** `Windows Server 2022 Datacenter - x64 Gen2`
  * **Size:** `Standard_DC1s_v3` (or equivalent cost-effective lab SKU)
* **Administrator Credentials:**
  * **Username:** `azureadmin`
  * **Password:** Robust complex passphrase (documented securely)
* **Inbound Connectivity:**
  * **Public Inbound Ports:** Select *Allow selected ports*
  * **Selected Inbound Ports:** `RDP (3389)` *(simulating intentional exposure for assessment)*
* Select **Review + Create**, then confirm deployment.

#### 4. Baseline Connectivity Validation
1. Open `GRC-WIN-VM01` in the portal, select **Connect** $\rightarrow$ **RDP**, and download the connection `.rdp` file.
2. Authenticate using configured administrator credentials to establish desktop session access.

<p align="center">
  <img src="https://github.com/user-attachments/assets/72b52203-1842-4f57-96df-9a5f7f2ad5e9" alt="Azure VM Connect RDP blade" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/9ab2bd03-5c58-47a2-8479-4040e90e8636" alt="Windows Server Desktop Initial Session" width="600"/>
</p>

---

### Phase 2: Asset, Threat, and Vulnerability Assessment

A proper GRC posture evaluation analyzes three foundational tiers:
* **Assets:** What critical services, data, and access vectors require protection?
* **Threats:** What adversaries or circumstances could realistically compromise them?
* **Vulnerabilities:** What configuration gaps or architectural weaknesses allow threats to materialize?

#### 1. OS & Compute Asset Verification
Inspect system configuration parameters under `GRC-WIN-VM01` $\rightarrow$ **Settings** $\rightarrow$ **Properties / Operating System** to confirm build baseline and patch level.

<p align="center">
  <img src="https://github.com/user-attachments/assets/68db4218-d543-46c5-ab43-07a4a37fb3eb" alt="Azure VM OS Properties" width="700"/>
</p>

#### 2. Network Attack Surface & Inbound Exposure Analysis
Navigate to **Networking** $\rightarrow$ **Network Settings** to review Network Security Group (NSG) rules:
* Examine inbound rules for public listeners.
* Confirm that TCP port `3389` (RDP) allows traffic sourced from `*` / `Internet` / `Any`, creating an unthrottled brute-force vector.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9d4c7b75-1067-42e5-bf69-c5f1858ffaf2" alt="NSG Inbound Port Rules Showing Open RDP" width="850"/>
</p>

#### 3. Identity and Privilege Audit
Evaluate local privilege escalation and credential boundaries:
1. Navigate to **Operations** $\rightarrow$ **Run Command** $\rightarrow$ `RunPowerShellScript`.
2. Execute `Get-LocalUser` to inventory local identity stores and detect active administrative accounts.

<p align="center">
  <img src="https://github.com/user-attachments/assets/3754e4db-a679-4f5c-ab13-71d509a4c736" alt="PowerShell Run Command executing Get-LocalUser" width="850"/>
</p>

3. Within the active RDP session, launch `Computer Management (compmgmt.msc)` $\rightarrow$ **Local Users and Groups** $\rightarrow$ **Groups** $\rightarrow$ **Administrators** to verify local administrator assignments.

<p align="center">
  <img src="https://github.com/user-attachments/assets/cfe4a50b-4390-4733-8539-691a1b90b736" alt="Windows Local Users and Groups configuration" width="700"/>
</p>

#### 4. Monitoring & Telemetry Review
Audit detective security visibility at both the hypervisor and guest OS layers:
* **Cloud Telemetry:** Navigate to `GRC-WIN-VM01` $\rightarrow$ **Monitoring**. Verify if continuous diagnostics and Azure Monitor collection are enabled.
* **Guest OS Auditing:** Within the VM, open `Event Viewer (eventvwr.msc)` $\rightarrow$ **Windows Logs** $\rightarrow$ **Security** to confirm event auditing is generating valid Event IDs (e.g., 4624, 4625).

<p align="center">
  <img src="https://github.com/user-attachments/assets/3aee0dab-6d25-4a66-805a-4b190d29dcaf" alt="Azure Monitoring Unconfigured State" width="600"/>
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/ef220f56-ea90-4b36-8acc-736169beace0" alt="Windows Security Event Viewer Logs" width="600"/>
</p>

---

### Phase 3: Quantitative Risk Scoring

To remove subjective ambiguity from assessment findings, risks are calculated using an industry-standard qualitative-to-quantitative formula:

$$\text{Risk Score} = \text{Likelihood} \times \text{Impact}$$

#### Calibration Scoring Rubrics

| Scale | Likelihood Criteria | Impact Severity Criteria |
| :---: | :--- | :--- |
| **1** | **Rare:** Attack vector is theoretically possible but requires extreme resource expenditure. | **Minimal:** Little to no operational, economic, or regulatory effect. |
| **2** | **Unlikely:** Vulnerability exists but requires specialized prerequisites or elevated access. | **Minor:** Isolated disruption to non-essential systems; zero data exposure. |
| **3** | **Possible:** Attack methods are well-documented; known exploits circulate publicly. | **Moderate:** Partial service unavailability; local privilege compromise. |
| **4** | **Likely:** Exposed, internet-facing vector subject to daily opportunistic automated scans. | **High:** Extensive disruption; significant data loss; lateral movement potential. |
| **5** | **Almost Certain:** Active zero-barrier exposure (e.g., unauthenticated publicly routable port). | **Critical:** Complete system compromise; domain takeover; regulatory non-compliance. |

---

### Phase 4: Framework Alignment

Technical risk mitigations must be defensible, auditable, and mapped directly to governance frameworks rather than implemented as unverified ad-hoc changes.

* **NIST Cybersecurity Framework (CSF):** Core functions include **Identify (ID)**, **Protect (PR)**, **Detect (DE)**, **Respond (RS)**, and **Recover (RC)**.
* **CIS Critical Security Controls (v8):** Prescriptive, prioritized technical safeguards designed to eliminate top modern attack vectors.
* **Control Typology:**
  * **Preventive:** Deter or intercept attacks before compromise occurs (e.g., Network segmentation, MFA).
  * **Detective:** Identify and flag anomalous or unauthorized access while underway (e.g., SIEM, Centralized logging).
  * **Corrective:** Remediate and recover system integrity post-event (e.g., System patching, incident response backups).

---

## Formal GRC Deliverable: Cloud Security Assessment Report

### 1. Executive Summary

This cloud security risk assessment evaluated the architectural posture of an Azure compute asset from a Governance, Risk, and Compliance (GRC) perspective. The primary objective was to inventory target assets, identify threat vectors, isolate vulnerabilities, quantify business exposure, and recommend compensating controls aligned with recognized industry standards.

The assessment revealed a **Medium to High overall risk posture**. Key findings include an exposed management interface (RDP) accessible to the public internet, single-factor local administrative authentication, default unhardened OS controls, and an absence of centralized SIEM-forwarded logging. These gaps create exposure to automated credential attacks, undetected lateral movement, and unauthorized system access. Compensating controls must be implemented immediately.

### 2. Scope & Target Architecture

| Assessment Attribute | Scope Detail |
| :--- | :--- |
| **Target Scope** | Standalone Cloud Compute Workload (Single Node) |
| **Platform** | Microsoft Azure |
| **Resource Group** | `GRC-Lab` |
| **Host Name** | `GRC-WIN-VM01` |
| **Operating System** | Windows Server 2022 Datacenter (x64) |
| **Network Boundary** | Azure VNet / Subnet with attached Network Security Group |
| **Ingress Access** | Remote Desktop Protocol (TCP Port 3389) via Public IPv4 |
| **Identity Mechanism** | Local Account (`azureadmin`) with Local Administrative Rights |
| **Assessment Methodology** | Non-intrusive Configuration Audit & Control Verification |

### 3. Identified Assets & Threat Profiles
