# Phase 2: Joining the Client Computer to the Domain

## Objective
Establish a secure trust relationship between the enterprise endpoint (Windows 11) and the central Active Directory infrastructure. This phase ensures the client machine is subject to centralized management, authentication, and Group Policy enforcement, simulating a standard corporate workstation deployment.

## Actions Taken

### 1. DNS Configuration
Before a machine can join a domain, it must be able to resolve the domain's name. To ensure reliable communication with Active Directory:
* Configured the Windows 11 client's primary IPv4 DNS server to point manually to the Domain Controller (`192.168.18.129`). 
*(Note: The client's IP address remains dynamically assigned via DHCP as `192.168.18.130`, but DNS is strictly manual).*
<img width="3840" height="2160" alt="Screenshot (27)" src="https://github.com/user-attachments/assets/89bd24d9-0553-45ff-a022-f47f796af16a" />


### 2. Domain Join & Authentication
With name resolution established, the system was configured to transition from a local workgroup to the centralized domain.
* Navigated to **Settings > System > About > Domain or workgroup** (Advanced System Settings) and initiated the domain join to `soclab.local`.
* Successfully authenticated the join request using Domain Admin credentials, resulting in the successful domain welcome prompt.
<img width="3840" height="2160" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/05bdae55-a526-4a93-8857-657dc7bf8e65" />


### 3. Finalization
* Restarted the Windows 11 system to apply the new domain membership and establish the secure machine trust account. 
* Upon reboot, verified the Full Device Name successfully updated to include the domain suffix (`DESKTOP-IC11PIP.soclab.local`).
<img width="3840" height="2160" alt="Screenshot (37)" src="https://github.com/user-attachments/assets/97ac0691-6579-4947-b34c-12f1da25f4ea" />


## ⚠️ Troubleshooting: Domain Controller Could Not Be Contacted
* **Cause:** If a client fails to find the domain during the join process, it is almost always a DNS misconfiguration where the client is still using a default NAT/ISP router for DNS instead of the internal Domain Controller.
* **Resolution:** Verified via `ipconfig /all` that the primary DNS was strictly set to `192.168.18.129` before attempting the join.

standard user credentials for verification

## 🏢 Enterprise Design Considerations
* **DHCP vs. Manual DNS:** In a production enterprise environment, endpoints typically receive their DNS settings automatically via DHCP Server Options rather than manual interface configuration. This lab uses manual assignment to explicitly demonstrate the strict dependency Active Directory has on precise DNS resolution.
* **Object Staging:** By default, newly joined computers land in the default `Computers` container, which cannot have Group Policy objects directly linked to it. Enterprise best practice dictates immediately moving this object to a structured Organizational Unit (OU)—which will be addressed in Phase 3—to ensure security baselines are enforced.
