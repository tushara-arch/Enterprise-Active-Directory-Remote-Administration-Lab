# Phase 5: Remote Management Execution & Negative Testing (Validation)

## Objective
Validate the enterprise security baseline established in Phase 4 by remotely connecting to the client endpoint via the Microsoft Management Console (MMC). This phase demonstrates successful remote execution (creating a scheduled task) while simultaneously proving the efficacy of the strict firewall rules through **Negative Testing**—confirming that unauthorized management tools are actively blocked.

## Actions Taken

### 1. Remote MMC Connection (Positive Test)
* Logged into the Domain Controller as a Domain Admin.
* Executed `mmc.exe` and added the **Computer Management** snap-in, targeting the remote endpoint: `DESKTOP-IC11PIP`.
* Successfully expanded the **System Tools** node. Both the **Event Viewer** and **Task Scheduler** snap-ins loaded data from the client seamlessly, confirming the specific GPO firewall rules for these services successfully opened the required dynamic RPC ports.

### 2. Remote Task Execution
* Navigated to **System Tools > Task Scheduler** within the remote MMC session.
* Created a new scheduled task remotely to execute a system configuration change.
* Configured the task to run under the `NT AUTHORITY\SYSTEM` context to ensure maximum execution privileges without requiring an interactive user logon.

### 3. Validating the "Implicit Deny" (Negative Testing)
To prove that the endpoint is not globally exposed to all remote administration, an attempt was made to access a management tool that was *not* explicitly allowed in the Phase 4 firewall GPO.
* Within the exact same active MMC console, clicked on the **Services and Applications > Services** node.
* **Result:** The console hung temporarily before throwing an error: *"Cannot open the Service Control Manager database on DESKTOP-IC11PIP. Error 1722: The RPC server is unavailable."*
* **Security Context:** This successfully proves the network "Least Privilege" model. Even though the Domain Controller is highly trusted and successfully connected to Task Scheduler, the Windows 11 firewall explicitly dropped the connection attempt to the Service Control Manager because the `Remote Service Management` rules were intentionally omitted from the GPO.

## ⚠️ Troubleshooting: Remote Execution Exceptions
* **Error: "The RPC Server is unavailable" (Error 0x800706BA)**
    * **Expected (Negative Test):** As demonstrated above, this error is *expected* when accessing unapproved nodes (like Services or Device Manager) because the firewall drops the dynamic port request.
    * **Unexpected (Task Scheduler Failure):** If this occurs on an *approved* service, it indicates the RPC Endpoint Mapper (Port 135) is reachable, but the specific dynamic high port is blocked, or the client dropped to a `Public` network profile.
* **Error: "Access is Denied" (Error 0x80070005)**
    * **Cause:** Attempting to execute the MMC connection using an account lacking local administrative privileges or missing the "Access this computer from the network" right.
    * **Resolution:** Ensured the session was authenticated using the Domain Admin account.

## ✅ Validation & Telemetry
* **Execution Verification:** Successfully observed the configuration change triggered by the remote scheduled task on the client endpoint in real-time.
* **SOC Visibility:** * **Client Logs (Security):** Monitored the endpoint's Event Viewer for **Event ID 4624 (Type 3 Logon)**, verifying a network logon occurred specifically from the management IP (`192.168.18.129`). Monitored for **Event ID 4698**, confirming the scheduled task was successfully created.
    * **Firewall Telemetry:** Reviewed the `pfirewall.log` (if enabled in Phase 4) to verify dropped TCP packets corresponding to the blocked **Services** connection attempt, proving the firewall's implicit deny is actively logging blocked lateral movement attempts.
```
