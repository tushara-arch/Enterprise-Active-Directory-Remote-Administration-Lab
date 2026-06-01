# Enterprise Active Directory & Remote Administration Lab
##  Project Overview
This project demonstrates the deployment and configuration of an Active Directory (AD) environment, focusing on centralizing client management. It covers joining a client machine to a domain, configuring Windows Defender Firewall for remote administration, structuring Organizational Units (OUs), applying Group Policy Objects (GPOs), and executing remote configurations via the Microsoft Management Console (MMC).

##  Environment Setup
* **Domain Controller (DC):** Windows Server 2025 (Also the DNS server) 
* **Client Machine:** Windows 11 Enterprise
* **Domain Name:** `soclab.local`
* **Network:** Both machines attached to the same virtual network (NAT).

## 🗺️ Network Topology
| Hostname | Role | IP Address | Operating System |
| :--- | :--- | :--- | :--- |
| `WIN-2QNL6415PHV` | Domain Controller / DNS | `192.168.18.129` (Static) | Windows Server 2025 |
| `DESKTOP-IC11PIP` | Domain Endpoint | `192.168.18.130` (DHCP) | Windows 11 Enterprise |

---

##  Phase 1: Active Directory Domain Controller Setup

**Actions Taken:**
1. Installed the **Active Directory Domain Services (AD DS)** and **DNS Server** roles via Server Manager.
2. Promoted the server to a Domain Controller and configured the new forest root domain.
   
---

##  Phase 2: Joining the Client Computer to the Domain

**Actions Taken:**
1. Configured the Windows 11 client's preferred DNS server to use the Domain Controller's IP address, enabling Active Directory name resolution.
2. Successfully joined the client machine to the soclab.local domain using Domain Admin credentials.
3. Restarted the system to complete the domain join process.
4. Verified successful domain membership and communication with Active Directory services..

> **⚠️ Common Errors & Troubleshooting**
> * **Error:** *"An Active Directory Domain Controller (AD DC) for the domain could not be contacted."*
> * **Cause:** The client machine could not resolve the domain name because its DNS settings were still pointing to the default external router/ISP.
> * **Fix:** Changed the client's IPv4 DNS server directly to the Domain Controller's IP address (`192.168.1.10`).

---

##  Phase 3: Organizational Unit (OU) & Group Policy (GPO) Configuration

**Actions Taken:**
1. Opened **Active Directory Users and Computers (ADUC)**.
2. Created a new OU named `IT_Department` and moved the client computer object into this OU.
3. Opened **Group Policy Management Console (GPMC)**.
4. Created a new GPO named `Firewall_Remote_Admin_Policy` and linked it to the `IT_Department` OU.

> **⚠️ Common Errors & Troubleshooting**
> * **Error:** *GPO settings are not reflecting on the client machine.*
> * **Cause:** Group Policy refreshes natively every 90 minutes; it does not apply instantly. Alternatively, the policy was applied to the wrong OU level.
> * **Fix:** Verified the computer object was in the correct OU. Ran `gpupdate /force` on the client machine via an elevated command prompt to force the policy pull. Checked status using `gpresult /r`.

---

##  Phase 4: Firewall Configuration for Remote MMC

**Actions Taken:**
To use MMC to control the client remotely, specific firewall rules must be enabled. I configured these centrally via the GPO created in Phase 3 (`Computer Configuration` -> `Policies` -> `Windows Settings` -> `Security Settings` -> `Windows Defender Firewall`).

**Enabled Rules:**
* COM+ Network Access (DCOM-In)
* Remote Event Log Management (NP-In, RPC, RPC-EPMAP)
* Remote Service Management (NP-In, RPC, RPC-EPMAP)
* Windows Management Instrumentation (WMI-In)

> **⚠️ Common Errors & Troubleshooting**
> * **Error:** *MMC Error: "The RPC Server is unavailable" (Error 0x800706BA).*
> * **Cause:** The Windows Defender Firewall on the client is blocking remote RPC dynamic ports, or the File and Printer Sharing rules are disabled.
> * **Fix:** Ensured the GPO specifically allowed inbound traffic for **Remote Administration** and **File and Printer Sharing**. Once the GPO applied, the MMC connection succeeded.

---

##  Phase 5: Remote Management via MMC & Task Scheduler

**Actions Taken:**
1. Logged into the Domain Controller (or an admin workstation) as a Domain Admin.
2. Opened `mmc.exe` and added the **Computer Management** snap-in.
3. Directed the snap-in to connect to the remote client computer.
4. Navigated to **Task Scheduler** within the remote MMC session.
5. Created a new scheduled task remotely to change a system setting (running under the `SYSTEM` or `Domain Admin` context).

> **⚠️ Common Errors & Troubleshooting**
> * **Error:** *"Access is Denied" when trying to connect via MMC.*
> * **Cause:** Attempting to connect using a standard user account rather than an account with local administrative privileges on the target machine.
> * **Fix:** Ensured I was logged into the host machine with my Domain Admin account, which by default is added to the local Administrators group of all domain-joined computers. 
> * **Error:** *Task Scheduler: "The network path was not found."*
> * **Cause:** The Remote Registry service was not running on the client machine.
> * **Fix:** Opened `services.msc` within the remote MMC, located the **Remote Registry** service, and started it.

---

##  Security Operations Context
Understanding these Active Directory mechanics and Windows internals is crucial for Security Operations Center (SOC) monitoring and enterprise vulnerability management. Properly configuring and securing RPC, WMI, and remote management interfaces demonstrates how to balance the reduction of the network attack surface while maintaining legitimate administrative access.
