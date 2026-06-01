Got it. That makes perfect sense for the flow of a runbook—Phase 4 is all about the **setup and hardening**, and Phase 5 is purely the **execution and proof**.

Here are both phases properly split according to your structure. Phase 4 now houses all the deep-dive prerequisites, user rights, and strict GPO firewall rules. Phase 5 is clean, focusing strictly on the MMC connection, the task creation, and the resulting SOC telemetry.

---

```markdown
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

```

---

```markdown
# Phase 5: Remote Management Execution via MMC

## Objective
Validate the enterprise security baseline established in Phase 4 by remotely connecting to the client endpoint via the Microsoft Management Console (MMC). Once connected, demonstrate remote administrative execution by creating a scheduled task without requiring a local interactive logon session.

## Actions Taken

### 1. Remote MMC Connection
* Logged into the Domain Controller as a Domain Admin.
* Executed `mmc.exe` and added the **Computer Management** snap-in.
* Directed the snap-in away from the local system to target the remote endpoint: `DESKTOP-IC11PIP`.

### 2. Remote Task Execution
* Navigated to **System Tools > Task Scheduler** within the remote MMC session.
* Created a new scheduled task remotely to change a system configuration setting.
* Configured the task to run under the `NT AUTHORITY\SYSTEM` context to ensure maximum execution privileges without requiring the interactive user to be logged in.

## ⚠️ Troubleshooting: Remote Execution Failures
* **Error: "The RPC Server is unavailable" (Error 0x800706BA)**
    * **Cause:** The RPC Endpoint Mapper (Port 135) is reachable, but the dynamic high port assigned to the Task Scheduler service is blocked, or the client dropped to a `Public` network profile.
    * **Resolution:** Verified the GPO applied correctly and that the endpoint was using the `Domain` network profile.
* **Error: "Access is Denied" (Error 0x80070005)**
    * **Cause:** Attempting to execute the MMC connection using an account lacking local administrative privileges or missing the "Access this computer from the network" right.
    * **Resolution:** Ensured the session was authenticated using the Domain Admin account.

## ✅ Validation & Telemetry
* **Verification:** Successfully executed the remote scheduled task and observed the configuration change on the client endpoint in real-time.
* **SOC Visibility:** * **Client Logs (Security):** Monitored the endpoint's Event Viewer for **Event ID 4624 (Type 3 Logon)**, verifying a network logon occurred specifically from the management IP (`192.168.18.129`).
    * Monitored for **Event ID 4698**, confirming the scheduled task was successfully created via the remote session.

```
