# Enterprise Active Directory & Remote Administration Lab
##  Project Overview
This project demonstrates the deployment and configuration of an Active Directory (AD) environment, focusing on centralizing client management. It covers joining a client machine to a domain, configuring Windows Defender Firewall for remote administration, structuring Organizational Units (OUs), applying Group Policy Objects (GPOs), and executing remote configurations via the Microsoft Management Console (MMC).

##  Environment Setup
* **Domain Controller (DC):** Windows Server 2025 (Also the DNS server) 
* **Client Machine:** Windows 11 Enterprise
* **Domain Name:** `soclab.local`
* **Network:** Both machines attached to the same virtual network (NAT).

##  Network Topology
| Hostname | Role | IP Address | Operating System |
| :--- | :--- | :--- | :--- |
| `WIN-2QNL6415PHV` | Domain Controller / DNS | `192.168.18.129` (Static) | Windows Server 2025 |
| `DESKTOP-IC11PIP` | Domain Endpoint | `192.168.18.130` (DHCP) | Windows 11 Enterprise |

---

##  Phase 1: Active Directory Domain Controller Provisioning

**Actions Taken:**
* **Infrastructure Foundation:** Deployed a Windows Server 2025 virtual machine to act as the central identity and access management hub for the enterprise lab.
* **Core Role Installation:** Installed Active Directory Domain Services (AD DS) to enable centralized authentication, policy enforcement, and security telemetry generation.
* **Network Stabilization:** Configured a static IP address on the primary network adapter to ensure reliable DNS resolution and prevent network isolation for future endpoints.
* **Domain Promotion:** Successfully promoted the server to the authoritative Domain Controller for the newly established `soclab.local` forest root domain. 
* **Enterprise Architecture Alignment:** Documented critical production environment design concepts, emphasizing the necessity of static addressing for core infrastructure and the principles of High Availability (HA) fault tolerance.
   
---

##  Phase 2: Joining the Client Computer to the Domain

**Actions Taken:**
* **Trust Establishment:** Joined a Windows 11 enterprise endpoint to the `soclab.local` domain to enable centralized management, authentication, and Group Policy enforcement.
* **DNS Configuration:** Manually configured the client's primary DNS to point directly to the Domain Controller, establishing the crucial name resolution required for Active Directory communication.
* **Advanced AD Hardening:** Mitigated a critical default Active Directory vulnerability by using ADSI Edit to set the `ms-DS-MachineAccountQuota` (MAQ) to `0`, preventing standard unprivileged users from joining rogue devices to the network.
* **Troubleshooting & Validation:** Navigated common domain join errors by validating network connectivity and verified the successful application of the domain suffix post-reboot.
* **Enterprise Architecture Alignment:** Outlined production deployment realities, contrasting manual DNS configuration with DHCP Server Options and defining best practices for Active Directory object staging.

---

##  Phase 3: Organizational Unit (OU) & Group Policy (GPO) Configuration

**Actions Taken:**
* **Hierarchy Definition:** Created a dedicated `IT department` Organizational Unit (OU) within Active Directory to establish a logical, manageable structure for enterprise endpoints.
* **Asset Staging:** Relocated the Windows 11 computer object from the unmanaged default `Computers` container into the targeted IT OU to bring it under management scope.
* **Policy Initialization:** Engineered and linked a new Group Policy Object (GPO) named `Remote Administration policy` specifically to the IT OU to centralize and deploy security configurations.
* **Application & Validation:** Bypassed standard 90-minute GPO background refresh cycles by executing `gpupdate /force` on the client, and validated successful policy application using `gpresult /r`.
* **Enterprise Architecture Alignment:** Highlighted critical Active Directory safeguards, including accidental deletion protection (ACL modification) and the importance of Tiered Administration Models for restricting lateral movement.

---

##  Phase 4: Security Hardening & Remote Administration Configuration

**Actions Taken:**
* **Network Trust Verification:** Verified Network Location Awareness (NLA) successfully authenticated with the Domain Controller, classifying the endpoint under the `Domain` profile to ensure accurate firewall rule processing.
* **Service Prerequisites:** Audited and validated that mandatory underlying background services (RPC, RPC Endpoint Mapper, Task Scheduler, and Event Log) were actively running to support remote management protocols.
* **Access Control & User Rights:** Configured User Rights Assignment via Group Policy to explicitly grant "Access this computer from the network" to Domain Admins, ensuring network logon authorization.
* **Centralized Firewall Hardening:** Engineered and deployed highly restricted inbound Windows Defender Firewall rules via GPO, explicitly scoping all remote management traffic solely to the Domain Controller's IP address.
* **Enterprise Architecture Alignment:** Contextualized the strict IP scoping as a simulation of Privileged Access Workstation (PAW) architecture, and highlighted how GPO-enforced immutable baselines actively prevent local GUI tampering.

---

## Phase 5: Remote Management Execution & Negative Testing (Validation)

**Actions Taken:**
* **Baseline Validation:** Successfully established a targeted remote MMC connection from the Domain Controller to the Windows 11 endpoint, confirming the precise execution of the dynamic RPC firewall rules configured in Phase 4.
* **Automated Remediation (SOAR):** Engineered an event-driven, self-healing scheduled task (`SOC-Firewall-Enforcer`) that detects unauthorized firewall profile changes (Event ID 2082) and automatically re-enables the firewall in milliseconds, simulating advanced active defense.
* **Negative Testing:** Proved the efficacy of the "Implicit Deny" network architecture by attempting to connect unauthorized snap-ins (e.g., Windows Defender Firewall management), verifying that the endpoint actively dropped the unapproved dynamic RPC requests.
* **Protocol Fallback Analysis:** Discovered and documented a critical SMB fallback vector, demonstrating how the Service Control Manager bypasses blocked dynamic RPC ports via Named Pipes (`\pipe\svcctl`) when broad File Sharing rules are enabled.
* **Enterprise Architecture Alignment:** Contextualized the lab findings within modern Security Operations Center (SOC) practices, highlighting the importance of SOAR, Remote UAC authentication restrictions, and deep packet inspection to uncover architectural blind spots.

---

> **Security Operations Context:** This project bridges the gap between systems administration and security engineering. Understanding deep Active Directory mechanics, Windows RPC/SMB protocol behaviors, and immutable GPO baselines is critical for modern Security Operations Center (SOC) monitoring and enterprise vulnerability management. By moving beyond basic configuration to implement SOAR-driven active defenses (auto-healing firewalls) and validating "Implicit Deny" architectures through negative testing, this lab demonstrates how to aggressively reduce the internal attack surface, eliminate architectural blind spots, and secure legitimate administrative access against lateral movement.
