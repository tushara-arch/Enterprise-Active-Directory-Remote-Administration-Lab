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
 4698**, confirming the scheduled task was successfully created via the remote session.

```
