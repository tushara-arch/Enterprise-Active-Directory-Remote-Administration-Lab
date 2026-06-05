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


##  Troubleshooting: GPO Settings Not Reflecting on Client
* **Cause 1 (Native Refresh Intervals):** Group Policy does not apply instantly. Windows endpoints natively refresh their policy from the Domain Controller every 90 minutes (with a randomized up to 30-minute offset).
    * **Resolution & Verification:** Executed `gpupdate /force` via an elevated command prompt on the Windows 11 client to force an immediate pull of the latest AD directory configurations. Once completed, ran `gpresult /r` to verify the application, confirming that `Remote Administration policy` successfully appeared under the "Applied Group Policy Objects" list for the computer context.

<img width="3840" height="2160" alt="Screenshot (38)" src="https://github.com/user-attachments/assets/6b289db0-e2ee-49b4-b6fa-495b219384e8" />


##  Enterprise Design Considerations
* **Accidental Deletion Protection:** The checkbox selected during the OU creation modifies the AD object's access control list (ACL) to explicitly deny the "Delete" permission to everyone, including Domain Admins. This is a critical enterprise safeguard against catastrophic script errors or accidental clicks that could orphan thousands of endpoints.
* **Tiered Administration Models:** While this lab uses a flat `IT department` OU structure, modern enterprise security architectures dictate highly granular OU hierarchies. Endpoints, servers, and domain controllers are strictly separated into different OUs to ensure lateral movement paths are restricted.
