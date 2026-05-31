# Phase 3: Organizational Unit (OU) & Group Policy (GPO) Configuration

## Objective
Establish a structured directory hierarchy and centralize security management. By default, newly joined endpoints are placed in a generic container that cannot receive targeted policies. This phase involves creating a dedicated Organizational Unit (OU) for the IT department and structuring a Group Policy Object (GPO) to explicitly manage and secure remote administration access to those endpoints.

## Actions Taken

### 1. Organizational Unit (OU) Creation
To enable targeted policy enforcement, a logical container was required to house the endpoint.
* Accessed **Active Directory Users and Computers (ADUC)** on the Domain Controller.
* Created a new Organizational Unit named `IT department` at the root of the `soclab.local` domain. 
* Ensured the enterprise failsafe "Protect container from accidental deletion" was enabled.
<img width="3840" height="2160" alt="Screenshot (31)" src="https://github.com/user-attachments/assets/ed8e8819-b63c-47eb-95ab-a4bb9d5d5f60" />


### 2. Asset Staging and Relocation
* Navigated to the default `Computers` container where the Windows 11 machine initially landed after the domain join.
* Relocated the `DESKTOP-IC11PIP` computer object into the newly created `IT department` OU, bringing it under the management scope of future departmental policies.
<img width="3840" height="2160" alt="Screenshot (32)" src="https://github.com/user-attachments/assets/7eb443e7-bbaf-439e-b78e-70bc2ebaeaf9" />


### 3. Group Policy Object (GPO) Initialization
* Opened the **Group Policy Management Console (GPMC)** to define the security configuration.
* Right-clicked the `soclab.local` domain and created a new GPO specifically named `Remote Administration policy`. 
<img width="3840" height="2160" alt="Screenshot (33)" src="https://github.com/user-attachments/assets/a75e4e38-48a5-4bf0-98d0-e7579906869e" />


### 4. Linking and Scope Definition
Policies must be explicitly linked to OUs to take effect.
* Linked the `Remote Administration policy` directly to the `IT department` OU. 
* Verified the link status was active and that the GPO scope applied to `NT AUTHORITY\Authenticated Users` (which includes the domain computers inside the OU).
<img width="3840" height="2160" alt="Screenshot (34)" src="https://github.com/user-attachments/assets/bc32ff11-7482-4e78-a2dc-afa748879391" />


## ⚠️ Troubleshooting: GPO Settings Not Reflecting on Client
* **Cause 1 (Native Refresh Intervals):** Group Policy does not apply instantly. Windows endpoints natively refresh their policy from the Domain Controller every 90 minutes (with a randomized up to 30-minute offset).
    * **Resolution:** Executed `gpupdate /force` via an elevated command prompt on the Windows 11 client to force an immediate pull of the latest AD directory configurations.
* **Cause 2 (Incorrect Policy Context):** Applying computer-based configurations to an OU that only contains User objects, or vice-versa.
    * **Resolution:** Verified in ADUC that the actual Computer object (`DESKTOP-IC11PIP`) was successfully seated inside the `IT department` OU where the policy was linked.

##  Validation & Telemetry
* **Verification:** Ran `gpresult /r` via the client's command prompt and confirmed that `Remote Administration policy` appeared under the "Applied Group Policy Objects" list for the computer context.
* **SOC Visibility:** 
    * **Domain Controller Logs:** Monitored the Security event logs for **Event ID 5136** (A directory service object was modified) to track the movement of the computer object between containers.
    * **Client Logs:** Monitored the local `Microsoft-Windows-GroupPolicy/Operational` logs for **Event ID 1502** to confirm successful Group Policy processing and retrieval of the new GPO from the `SYSVOL` share.

##  Enterprise Design Considerations
* **Accidental Deletion Protection:** The checkbox selected during the OU creation modifies the AD object's access control list (ACL) to explicitly deny the "Delete" permission to everyone, including Domain Admins. This is a critical enterprise safeguard against catastrophic script errors or accidental clicks that could orphan thousands of endpoints.
* **Tiered Administration Models:** While this lab uses a flat `IT department` OU structure, modern enterprise security architectures (like Microsoft's Enterprise Access Model) dictate highly granular OU hierarchies. Endpoints, servers, and domain controllers are strictly separated into different OUs to ensure lateral movement paths are restricted.
