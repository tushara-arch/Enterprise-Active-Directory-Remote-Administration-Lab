# Phase 1: Active Directory Domain Controller Provisioning

## Objective
Establish the foundational identity and access management infrastructure for the enterprise lab environment. This phase involves configuring a Windows Server 2025 virtual machine as the primary Domain Controller (DC) to facilitate centralized authentication, policy enforcement, and DNS resolution. 

This infrastructure is critical for simulating a realistic enterprise environment where security telemetry (event logs, authentication traffic, Group Policy updates) can be generated, centralized, and monitored.

## Actions Taken

### 1. Role Installation
* Accessed **Server Manager** on the newly deployed Windows Server 2025 instance.
* Successfully installed the core infrastructure roles:
    * **Active Directory Domain Services (AD DS)**
    * **DNS Server**

### 2. Network Configuration & Prerequisite Resolution
During the initial prerequisite validation for domain promotion, the system flagged a warning regarding the physical network adapter lacking a static IP address. To ensure network stability and reliable DNS resolution for future endpoints:
* Transitioned the primary Ethernet adapter from DHCP to a static manual configuration.
* **IPv4 Configuration Applied:**
    * **IP Address:** `192.168.18.129`
    * **Subnet Mask:** `255.255.255.0` (/24)
    * **Default Gateway:** `192.168.18.2`
    * **Preferred DNS:** `127.0.0.1` *(Configured as the loopback address since this server will host the primary DNS zone for the domain)*

### 3. Domain Promotion & Configuration
With the static IP configured, the AD DS Configuration Wizard prerequisite checks passed successfully. The server was then promoted to a Domain Controller with the following specifications:
* **Deployment Operation:** Add a new forest
* **Root Domain Name:** `soclab.local`
* **NetBIOS Domain Name:** `SOCLAB`
* **Forest & Domain Functional Level:** Windows Server 2025
* **Additional Options:** * Global Catalog (GC) selected.
    * DNS Server selected.
* **Database, Log, and SYSVOL Folders:** Maintained default paths (`C:\WINDOWS\NTDS` and `C:\WINDOWS\SYSVOL`).

### 4. Finalization
* Initiated the installation. Following the automatic reboot, the server successfully came online as the authoritative Domain Controller for `soclab.local`.

## Next Steps
With the domain established, the next phase will involve provisioning endpoint machines (e.g., Windows 10/11), joining them to the `soclab.local` domain, and setting up the foundational Group Policy Objects (GPOs) to manage the environment.
