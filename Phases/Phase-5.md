# Phase 5: Remote Management Execution & Negative Testing (Validation)

## Objective
Validate the enterprise security baseline established in Phase 4 by remotely connecting to the client endpoint via the Microsoft Management Console (MMC). This phase demonstrates successful remote execution (creating a scheduled task) while simultaneously proving the efficacy of the strict firewall rules through **Negative Testing**—confirming that unauthorized management tools are actively blocked.

## Actions Taken

### 1. Remote MMC Connection (Positive Test)
* Logged into the Domain Controller as a Domain Admin.
* Executed `mmc.exe` to open a blank console.
* Navigated to **File > Add/Remove Snap-in**, selected **Task Scheduler**, and explicitly targeted the remote endpoint: `DESKTOP-IC11PIP`.
* Successfully connected and loaded the remote task library, confirming the GPO firewall rules successfully opened the required dynamic RPC ports for this specific service.
<img width="3840" height="2160" alt="Screenshot (45)" src="https://github.com/user-attachments/assets/6ec70063-bb71-4b23-acac-54c051106e38" />


### 2. Remote Task Execution (Auto-Healing Firewall Scenario)
To demonstrate advanced SOC automation, a self-healing task was engineered remotely to prevent the endpoint's firewall from being maliciously disabled.

* **The Object Picker Block (Unexpected Negative Test):** During initial task creation, an attempt to change the executing user account triggered a connection error: *"The program cannot open the required dialog box because it cannot determine whether the computer... is joined to a domain."*<img width="3840" height="2160" alt="Screenshot (44)" src="https://github.com/user-attachments/assets/3a961d19-0d2a-4384-b561-dae36567f89e" />
  
  * **Security Context:** This error inadvertently proved the efficacy of the Phase 4 baseline. The strict firewall actively dropped the Object Picker's Named Pipes/SMB request (TCP Port 445) required to verify domain accounts.
* **Baseline Adjustment (Resolution):** To resolve this and allow remote account querying, a targeted **File and Printer Sharing (SMB-In)** rule was added to the central GPO. <img width="3840" height="2160" alt="image" src="https://github.com/user-attachments/assets/3c41153e-6d30-4ea6-8fef-909177070dc9" />
* **Privilege Escalation:** With SMB communication permitted to the Domain Controller, the task (`SOC-Firewall-Enforcer`) was successfully configured to run as `NT AUTHORITY\SYSTEM` with highest privileges so it executes silently in the background.
* **Event-Driven Trigger:** Configured the task to trigger instantly upon the logging of **Event ID 2082** (generated when a Windows Defender Firewall profile setting has changed) within the `Microsoft-Windows-Windows Firewall With Advanced Security` log.
* **Automated Remediation:** Set the action to launch `netsh` with the arguments `advfirewall set allprofiles state on`, forcing the Domain, Private, and Public profiles immediately back to an active state.  <img width="3845" height="2165" alt="Task Creation" src="https://github.com/user-attachments/assets/50ef692f-ab63-4058-8759-5a2996db0589" />
* **Live Execution Demonstration:** The automated response was actively validated on the endpoint. As demonstrated below, when the Windows Defender Firewall is manually disabled, the scheduled task immediately detects the event and executes the remediation command, instantly forcing the firewall back to an active and secure state without manual SOC intervention.<img width="3845" height="2165" alt="Verification" src="https://github.com/user-attachments/assets/8b518b16-6440-41d8-8ba0-ea61f880d482" />
  


### 3. Validating the "Implicit Deny" (Negative Testing)
To prove that the endpoint is not globally exposed to all remote administration, an attempt was made to access a management tool that was *not* explicitly allowed in the Phase 4 firewall GPO.
* Returned to **File > Add/Remove Snap-in** and attempted to add the **Services** snap-in, targeting the same remote endpoint (`DESKTOP-IC11PIP`).
* **Result:** The console hung temporarily before throwing an error: *"Cannot open the Service Control Manager database on DESKTOP-IC11PIP. Error 1722: The RPC server is unavailable."*
* **Security Context:** This successfully proves the network "Least Privilege" model. Even though the Domain Controller is highly trusted and successfully connected to Task Scheduler, the Windows 11 firewall explicitly dropped the connection attempt to the Service Control Manager because the `Remote Service Management` rules were intentionally omitted from the GPO.
<img width="3840" height="2160" alt="Screenshot (49)" src="https://github.com/user-attachments/assets/903a22b8-08d8-467d-8ba2-8c57496aa3ef" />

### 4. Unintended Access Discovery (The SMB Fallback Vector)
To further test the baseline, an attempt was made to load the **Services** snap-in, which was expected to fail since the specific "Remote Service Management" RPC rules were intentionally omitted.
* **Result:** The Services snap-in successfully connected and populated the remote service list.
* **Security Context (The Explanation):** This reveals a critical enterprise security mechanism. While the primary protocol for the Service Control Manager (SCM) is dynamic RPC, it has a built-in fallback. Because the **File and Printer Sharing (SMB-In)** rule (TCP Port 445) was explicitly enabled to allow the Object Picker to function during task creation, the SCM bypassed the blocked dynamic RPC ports and established the connection using **Named Pipes over SMB** (`\pipe\svcctl`). This demonstrates to a SOC analyst how enabling a broad protocol like SMB can inadvertently expose secondary administrative vectors, highlighting the importance of strict protocol monitoring and deep packet inspection.<img width="3840" height="2160" alt="image" src="https://github.com/user-attachments/assets/a0f6b9b1-af00-471e-b9c7-091008b71ff5" />
* **Alternative Strict Configuration (RPC Only):** If the SMB-In rule was strictly disabled (blocking the Named Pipe fallback), remote services.msc access could still be successfully established by enabling only the Remote Service Management (RPC) dynamic port rule, intentionally ignoring its grouped (NP-In) and (RPC-EPMAP) predefined rules.
    * **The Reason:** SCM requires two steps: querying the Endpoint Mapper (Port 135) and connecting to the dynamic service port. Because the Remote Scheduled Tasks Management (RPC-EPMAP) rule and the COM+ rule configured in Phase 4 already explicitly open TCP Port 135, the Endpoint Mapper is already actively routing traffic for the management IP. Therefore, the specific RPC-EPMAP rule for Remote Service Management is entirely redundant. Enabling just its (RPC) rule allows the dynamic high port to open, permitting standard TCP RPC to function flawlessly without exposing SMB (Port 445).


## ⚠️ Troubleshooting: Remote Execution Exceptions
* **Error: "The RPC Server is unavailable" (Error 0x800706BA / 1722)**
    * **Expected (Negative Test):** As demonstrated above, this error is *expected* when accessing unapproved snap-ins (like Services) because the firewall drops the dynamic port request.
* **Error: "Access is Denied"**
    * **Cause:** Attempting to execute the MMC connection using an account lacking local administrative privileges or missing the "Access this computer from the network" right.(In my case I was using a Local Administrator account initially)
    * **Resolution:** Ensured the session was authenticated using the Domain Admin account.
  <img width="3840" height="2160" alt="Screenshot (41)" src="https://github.com/user-attachments/assets/84695fc9-7a70-41ea-9357-dd79fa0ce06b" />



