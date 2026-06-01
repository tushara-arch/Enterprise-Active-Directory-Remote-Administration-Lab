# Phase 4: Security Hardening & Remote Administration Prerequisites

## Objective
Prepare the enterprise endpoint for secure remote management by configuring necessary Windows internal services, user rights, and highly restricted firewall policies via GPO. This phase ensures the machine can accept remote commands without exposing a broad network attack surface.

## Actions Taken

### 1. Network Profile & Adapter Settings
Before the firewall processes any rules, the Windows operating system must explicitly trust the network connection.
* **Network Location Awareness (NLA):** Verified the target machine’s network adapter successfully authenticated with the Domain Controller on boot, classifying its active connection profile as `Domain`. 
*(Security Context: If the adapter fails to reach the DC, it defaults to the `Public` profile, which will drop incoming remote connections regardless of GPO firewall rules).*

### 2. Mandatory Background Services
A remote RPC call fails instantly if the listening service is disabled. Verified the following services were set to **Automatic** and currently **Running** on the Windows 11 client:
* **Task Scheduler (`Schedule`):** The core engine required to read or write remote tasks.
* **Windows Event Log (`EventLog`):** Required for the MMC console UI to render the "History" tab without crashing.
* **RPC Endpoint Mapper (`RpcEptMapper`):** Acts as the directory service that tells the MMC console which dynamic port the Task Scheduler is currently listening on.

### 3. User Rights Assignment (Local Security Policy)
Even with perfect network configurations, Windows evaluates the local security policy to authorize the network logon.
* Accessed the GPO and navigated to: **Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment**.
* **Access this computer from the network:** Ensured that `SOCLAB\Domain Admins` (and Authenticated Users) were explicitly granted this right.
* **Deny access to this computer from the network:** Verified that no administrative groups were listed here, as a "Deny" rule silently overrides all firewall and permission settings, resulting in an immediate "Access Denied."

### 4. Centralized Firewall Hardening via GPO
Instead of configuring the firewall locally, strict access controls were enforced centrally to prevent accidental exposure.
* In the GPMC, navigated to: **Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security**.
* Created and enabled the following Inbound Rules, explicitly restricting the **Remote IP** scope to the Domain Controller's IP (`192.168.18.129`) to prevent lateral movement from peer workstations:
    * **Predefined:** Remote Scheduled Tasks Management
    * **Predefined:** Remote Event Log Management
    * **Custom:** TCP Port 135 (RPC Endpoint Mapper)
    * **Custom:** TCP Port 445 (SMB/Named Pipes - Required for Task Scheduler UI & File/Printer Sharing)

## ✅ Validation & Telemetry
* **Verification:** Ran `gpupdate /force` on the client, then opened `wf.msc` (Windows Defender Firewall) locally on the client to verify the scoped rules successfully propagated from the Domain Controller and were actively enforced on the `Domain` profile.
