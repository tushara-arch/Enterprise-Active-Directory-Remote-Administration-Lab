# Phase 1: Active Directory Domain Controller Provisioning

## Objective
Establish the foundational identity and access management infrastructure for the enterprise lab environment. This phase involves configuring a Windows Server 2025 virtual machine as the primary Domain Controller (DC) to facilitate centralized authentication, policy enforcement, and DNS resolution. 

This infrastructure is critical for simulating a realistic enterprise environment where security telemetry (event logs, authentication traffic, Group Policy updates) can be generated, centralized, and monitored.

## Actions Taken

### 1. Role Installation
* Accessed **Server Manager** on the newly deployed Windows Server 2025 instance.
* Successfully installed the core infrastructure roles:
    * **Active Directory Domain Services (AD DS)**
    
 <img width="3840" height="2160" alt="Screenshot (17)" src="https://github.com/user-attachments/assets/4dcb1781-474b-4c1c-91cb-910a0ffea4ec" />

### 2. Network Configuration & Prerequisite Resolution
During the initial prerequisite validation for domain promotion, the system flagged a warning regarding the physical network adapter lacking a static IP address. To ensure network stability and reliable DNS resolution for future endpoints:
* Transitioned the primary Ethernet adapter from DHCP to a static manual configuration.
<img width="3840" height="2160" alt="Screenshot (20)" src="https://github.com/user-attachments/assets/54d93656-2c24-4f2f-be15-3ddebd2dc8a6" />
<img width="3332" height="1777" alt="Screenshot (21)" src="https://github.com/user-attachments/assets/602edaff-1ef8-48e5-add0-c7b005a94a0d" />
<img width="3840" height="2160" alt="Screenshot (23)" src="https://github.com/user-attachments/assets/7fda72d4-fc14-42fa-85f1-6c82ada36e98" />
This is standard behavior when provisioning a new forest root domain (like `soclab.local`) in an isolated lab. The system is attempting to contact a parent DNS server to register the new domain. Because this is a standalone internal environment, no parent server exists.So we can ignore this warning and safely proceed with the installation.

### 3. Domain Promotion & Configuration
With the static IP configured, the AD DS Configuration Wizard prerequisite checks passed successfully. The server was then promoted to a Domain Controller with the following specifications:
<img width="3840" height="2160" alt="Screenshot (19)" src="https://github.com/user-attachments/assets/b13d9665-f1ae-427d-83f2-b5c115260cb0" />

### 4. Finalization
* Initiated the installation. Following the automatic reboot, the server successfully came online as the authoritative Domain Controller for `soclab.local`.
<img width="3838" height="1961" alt="image" src="https://github.com/user-attachments/assets/f8e18a5e-666a-4338-aa91-7c15dc83f72a" />

## 🏢 Enterprise Design Considerations
* **Static Addressing for Core Infrastructure:** Assigning a static IP address to the Domain Controller is a strict requirement, not an option. Because this server acts as the primary DNS authority for the `soclab.local` domain, dynamic IP changes would cause widespread name resolution failures, isolate endpoints, and immediately disrupt the routing of security telemetry across the network.
* **High Availability & Redundancy:** While this lab provisions a single Domain Controller because of limitations of compute resources, production enterprise environments deploy multiple Domain Controllers across different physical or geographical sites. This ensures fault tolerance—if one DC experiences a hardware failure or network outage, secondary DCs seamlessly take over to ensure authentication and directory services remain uninterrupted.
