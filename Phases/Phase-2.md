# 🔗 Phase 2: Joining the Client Computer to the Domain

## Actions Taken

### 1. DNS Configuration
Configured the Windows 11 client's primary IPv4 DNS server to point manually to the Domain Controller (`192.168.18.129`). This is a mandatory step to enable Active Directory name resolution so the client can locate the `soclab.local` domain.
*(Note: The client's IP address remains dynamically assigned via DHCP as `192.168.18.130`, but DNS is strictly manual).*

![DNS Configuration](Screenshot%20(27).jpg)

### 2. Domain Join
Navigated to **Settings > System > About > Domain or workgroup** (Advanced System Settings) and initiated the domain join to `soclab.local`.

### 3. Authentication
Successfully authenticated the join request using Domain Admin credentials, resulting in the successful domain welcome prompt.

![Domain Join Welcome](Screenshot%20(28).jpg)

### 4. Finalization
Restarted the Windows 11 system to apply the new domain membership. Upon reboot, verified the Full Device Name successfully updated to include the domain suffix (`DESKTOP-IC11PIP.soclab.local`).

![Verified Domain Membership](image_b438bf.jpg)

## ⚠️ Troubleshooting: Domain Controller Could Not Be Contacted
* **Cause:** If a client fails to find the domain, it is almost always a DNS misconfiguration where the client is still using a default NAT/ISP router for DNS instead of the Domain Controller.
* **Fix:** Verified via `ipconfig /all` (and the Windows GUI) that the primary DNS was strictly set to `192.168.18.129` before attempting the join.

## ✅ Validation & Telemetry

### Verification
Logged into the Domain Controller, opened **Active Directory Users and Computers (ADUC)**, and verified the `DESKTOP-IC11PIP` computer object successfully populated in the default `Computers` container.

### SOC Visibility
* **Domain Controller Logs:** Monitored the Security event logs for **Event ID 4741** (A computer account was created) and **Event ID 4624** (Successful Logon) to confirm the machine account established a trust relationship.
* **Client Logs:** Checked the local System logs for **NetJoin** events confirming the successful transition from a workgroup to the `soclab.local` domain.
