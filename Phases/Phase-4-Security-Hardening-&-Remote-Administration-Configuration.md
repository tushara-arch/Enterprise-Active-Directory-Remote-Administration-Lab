# Phase 4: Security Hardening & Remote Administration Configuration

## Objective
Prepare the enterprise endpoint for secure remote management by configuring necessary Windows internal services, user rights, and highly restricted firewall policies via GPO. This phase ensures the machine can accept remote commands without exposing a broad network attack surface.

## Actions Taken

### 1. Network Profile & Adapter Settings
Before the firewall processes any rules, the Windows operating system must explicitly trust the network connection.
* **Network Location Awareness (NLA):** Verified the target machine’s network adapter successfully authenticated with the Domain Controller on boot, classifying its active connection profile as `Domain`. 
*(Security Context: If the adapter fails to reach the DC, it defaults to the `Public` profile, which will drop incoming remote connections regardless of GPO firewall rules).*

### 2. Mandatory Background Services
A remote RPC call fails instantly if the listening service is disabled. Verified the following services were set to **Automatic** and currently **Running** on the Windows 11 client:
* **Task Scheduler:** The core engine required to read or write remote tasks.
* **Windows Event Log:** Required for the MMC console UI to render the "History" tab without crashing.
* **Remote Procedure Call (RPC):** The foundational transport mechanism for Windows inter-process communication. If this core service is not running, all network-based management protocols and MMC snap-ins will completely fail to establish a connection.
* **RPC Endpoint Mapper:** Acts as the directory service that tells the MMC console which dynamic port the Task Scheduler is currently listening on.

### 3. User Rights Assignment (Local Security Policy)
Even with perfect network configurations, Windows evaluates the local security policy to authorize the network logon.
* Accessed the GPO and navigated to: **Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment**.
* **Access this computer from the network:** Ensured that `SOCLAB\Domain Admins` (and Authenticated Users) were explicitly granted this right.
* **Deny access to this computer from the network:** Verified that no administrative groups were listed here, as a "Deny" rule silently overrides all firewall and permission settings, resulting in an immediate "Access Denied." 
<img width="3840" height="2160" alt="Screenshot (39)" src="https://github.com/user-attachments/assets/62ffde4e-4701-4ebc-a5bb-b925d766add4" />

### 4. Centralized Firewall Hardening via GPO
Instead of configuring the firewall locally, strict access controls were enforced centrally to prevent accidental exposure.
* In the GPMC, navigated to: **Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security**.
* Created and enabled the following Inbound Rules, explicitly restricting the **Remote IP** scope to the Domain Controller's IP (`192.168.18.129`).
<img width="3840" height="2160" alt="image" src="https://github.com/user-attachments/assets/bd41355a-41cd-4f4c-b3ec-8757ce2e4cfd" />
* **Verification:** Ran `gpupdate /force` on the client, then opened `wf.msc` (Windows Defender Firewall) locally on the client to verify the scoped rules successfully propagated from the Domain Controller and were actively enforced on the `Domain` profile.

##  Enterprise Design Considerations
* **Privileged Access Workstations (PAWs) & Tiered Admin Model:** The strict IP scoping applied to the firewall rules in this phase simulates a PAW architecture. In a mature enterprise, Domain Admins do not manage systems from generic user VLANs. Management traffic is strictly siloed to dedicated, highly secured management subnets, preventing lateral movement if a standard user endpoint is compromised.
* **Immutable Security Baselines (GPO vs. Local):** Configuring User Rights and Firewall rules via Active Directory GPOs ensures the security baseline is immutable at the endpoint level. When firewall profiles (such as the Domain profile) are enforced via GPO, the local operating system actively locks the configuration interface, preventing even local administrators from manually disabling the firewall via the GUI.
