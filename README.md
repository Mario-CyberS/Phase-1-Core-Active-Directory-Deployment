# Phase 1 — Core Active Directory Deployment  
This phase covers installing and configuring the **Active Directory Domain Services (AD DS)** role, promoting your Windows Server VM (`DC01`) to a Domain Controller, and setting up supporting infrastructure such as **DNS**, **DHCP**, and the base **OU structure**.  

---

## 🎯 Objective  
To deploy and configure a fully functional Active Directory Domain Controller in a Windows Server 2025 environment, complete with DNS, DHCP, and an organized OU structure for domain management.

---

## 🔍 Why Build an AD Core?  
This setup establishes the foundation for identity and access management (IAM). It enables:  
- Centralized authentication and account management  
- Network-level DNS and DHCP control  
- Hierarchical organization of users, computers, and groups  
- A scalable foundation for future GPO, SSO, and PAM configurations  

---

## 📚 Skills Learned  
- Installing AD DS and DNS roles in Windows Server  
- Promoting a server to a Domain Controller  
- Setting up reverse lookup zones and DHCP scopes  
- Assigning static IPs for domain controllers  
- Creating Organizational Units (OUs) for logical structure  

---

## 🛠️ Tools Used  
<div>
  <a href="https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025" target="_blank">
    <img src="https://img.shields.io/badge/-Windows_Server_2025-0078D4?style=for-the-badge&logo=windows&logoColor=white"/>
  </a>
  <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc" target="_blank">
    <img src="https://img.shields.io/badge/-Active_Directory-3333CC?style=for-the-badge&logo=microsoft&logoColor=white"/>
  </a>
  <a href="https://learn.microsoft.com/en-us/windows-server/networking/dhcp/dhcp-top" target="_blank">
    <img src="https://img.shields.io/badge/-DHCP_Server-008272?style=for-the-badge"/>
  </a>
  <a href="https://learn.microsoft.com/en-us/windows-server/networking/dns/dns-top" target="_blank">
    <img src="https://img.shields.io/badge/-DNS_Server-4479A1?style=for-the-badge"/>
  </a>
</div>

---

## 📝 Deployment Steps  

### 1️⃣ Install the AD DS Role  
On your Server VM (`DC01`):  
1. Open **Server Manager**  
2. Click **Manage → Add Roles and Features**  
3. Choose:  
   - **Role-based installation**  
   - Select your local server from the server pool  
4. Check ✅ **Active Directory Domain Services**  
5. Accept prompts for additional tools and DNS Server installation  
6. Click **Install** (restart if required)  

📸 *Screenshot Example:* Add Roles Wizard with AD DS selected  

---

### 2️⃣ Promote to Domain Controller  
After installation, in **Server Manager → Notifications (⚠️)**:  
1. Click **Promote this server to a domain controller**  
2. Choose **Add a new forest**  
3. Root domain name: `lab.local`  
4. Set the **DSRM password**  
5. Leave defaults:  
   - Functional Level: *Windows Server 2025*  
   - DNS: *Enabled*  
   - Global Catalog: *Enabled*  
6. NetBIOS name: `LAB`  
7. Accept default paths for SYSVOL and CNFG  
8. Click **Install**, then **Reboot**

📸 *Screenshot Examples:*  
- Domain setup wizard  
- System properties showing domain: `lab.local`

---

### 3️⃣ Configure DNS (Reverse Lookup Zone)  
1. Open **DNS Manager** → Expand your server  
2. Right-click **Reverse Lookup Zones → New Zone**  
3. Choose:  
   - Zone Type: *Primary zone*  
   - Replication: *To all DNS servers in domain*  
   - Network ID: e.g. `192.168.163`  
   - Dynamic updates: *Allow only secure dynamic updates*  
4. Finish and confirm both **Forward** and **Reverse** zones appear  

📸 *Screenshot Example:* Forward & Reverse zones visible  

---

### 4️⃣ Configure DHCP  
Before installing DHCP, assign your Domain Controller a **static IP address**.  

#### ✅ Step 1 — Get Current Network Info  
In **CMD**:
```powershell
ipconfig
```
Example output:
```powershell
IPv4 Address . . . . . : 192.168.163.128
Subnet Mask . . . . . : 255.255.255.0
Default Gateway . . . : 192.168.163.2
DNS Servers . . . . . : 127.0.0.1
```
#### ✅ Step 2 — Choose a Static IP
Pick one outside the DHCP pool, e.g.
```powershell
192.168.163.10
```
#### ✅ Step 3 — Set Static IP
1. Open Network & Internet Settings → Ethernet
2. Edit IPv4 properties:
IP address: 192.168.163.10
Subnet mask: 255.255.255.0
Default gateway: 192.168.163.2
Preferred DNS: 127.0.0.1
3. Disable IPv6
#### ✅ Add Localhost DNS Record
1. Open DNS Manager → lab.local → New Host (A)
2. Name: localhost
3. IP: 127.0.0.1
4. Check: Create associated PTR record
Then register DNS:
```powershell
ipconfig /registerdns
ipconfig /flushdns
```
Test:
```powershell
nslookup localhost
nslookup 127.0.0.1
nslookup <your-static-ip>
```
#### ✅ Step 6 — Connectivity Test
```powershell
ping <gateway-ip>
ping <servername>
ping lab.local
```
Everything should resolve successfully.

### 🎯 Install and Configure DHCP
1. Install DHCP Server (same steps as AD DS, Install it on same server)
2. Open DHCP Console
### Authorize DHCP in Active Directory
1. In Server Manager → Notifications ⚠️
2. Click Complete DHCP Configuration
You're in Server Manager now, your server should already be authorized but check.
Do this:
1. Right-click your server name under DHCP
2. Click DHCP Manager
3. Inside DHCP Manager:
Right-click your DHCP server name (WIN-xxxx…)
Select ✅ Authorize
4. Wait ~10 seconds
5. Right-click again → Refresh
You should see the server icon turn green ✔

#### ✅ Step 1 — Create DHCP Scope

Path: IPv4 → Right-click → New Scope

Setting:    |  Value:
Scope Name	|  Lab-LAN-Scope
Start IP	  |  192.168.163.50
End IP	    |  192.168.163.200
Subnet Mask	|  255.255.255.0
Router	    |  192.168.163.2
Domain Name	|  lab.local

Skip WINS and finish the wizard.
Activate the scope if not automatically enabled.

#### ✅ Step 2 — Activate the Scope
Right-click the new scope → Activate (If not already)

#### ✅ Step 3 — Add DNS Forwarders (VERY Important)
Because your DC only knows internal DNS — you must forward internet lookups.
In DNS Manager:
1. Right-click your server name → Properties
2. Go to Forwarders tab
3. Add:
```powershell
8.8.8.8
1.1.1.1
```
Click OK
This ensures clients can resolve internet domains.

#### ✅ Step 4 — Final DNS Functional Check
Run in PowerShell on DC:
Tools → Windows PowerShell
```powershell
Resolve-DnsName google.com
```
Should return IP addresses ✅

#### 5️⃣ Create OU Structure
Go to AD DS and right click on server
Inside Active Directory Users & Computers:
Right-click domain → New → Organizational Unit
Create OUs for each of these:

OU:                  | Purpose:
LAB Users            | Regular accounts
LAB Computers        | Workstations
LAB Servers          | Future server machines
LAB Groups           | Security groups
LAB Service Accounts | Service users
LAB Admins           | Secured OU

---

### 👨‍💻 Author
Mario Tagaras | Cyber Security Specialist






