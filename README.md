# Azure Lab: Domain Controller and Client VM Setup

![{93E1CCA0-D7C4-4689-92B2-7F2DA0FC7F4B}](https://github.com/user-attachments/assets/03c2604f-a76a-45e7-b636-817028c0c903)


This lab will guide you through the process of setting up a Windows Server 2022 Domain Controller and a Windows 10 Client within Microsoft Azure. These virtual machines will be used for subsequent labs.

---

## Step 1: Create Resource Group and Network Infrastructure

### Create a Resource Group
- Use a unique name, e.g., `Active-Directory-Lab`
- Select your preferred Azure region

### Create a Virtual Network and Subnet
- Name: `VNET-LabDomain`
- Subnet: `Subnet-Lab` (e.g., 10.0.0.0/24)


https://github.com/user-attachments/assets/53c4dfb5-4d58-4919-be58-edeeb9868534


---

## Step 2: Create Domain Controller VM

### VM Configuration
- Name: `DC-1`
- Image: Windows Server 2022
- Username: `labuser`
- Password: `Cyberlab123!`
- Region: Same as your Resource Group
- Virtual Network: `VNET-LabDomain`
- Subnet: `Subnet-Lab`


https://github.com/user-attachments/assets/c02cdd3d-add4-4cb0-a833-34da715f82c5




### Post-Creation Configuration
1. Set Static Private IP
   - Go to `DC-1`'s Network Interface settings in Azure
   - Set the Private IP to Static

2. Log into `DC-1` via RDP

3. Disable Windows Firewall (for testing only)
   - Open PowerShell or Command Prompt as Administrator and run:
     ```powershell
     netsh advfirewall set allprofiles state off
     ```


https://github.com/user-attachments/assets/b59e653e-c3a4-4969-8670-22d4c6b12cdf


---

## Step 3: Create Client-1 VM

### VM Configuration
- Name: `Client-1`
- Image: Windows 10
- Username: `labuser`
- Password: `Cyberlab123!`
- Region: Same as `DC-1`
- Virtual Network: `VNET-LabDomain`
- Subnet: `Subnet-Lab`

https://github.com/user-attachments/assets/26757022-0e52-4eda-bd99-c1b3550f01f7

### Post-Creation Configuration
1. Set DNS to `DC-1`’s Private IP
   - Go to `Client-1`'s Network Interface settings in Azure
   - Set DNS server to `DC-1`’s Static Private IP

2. Restart `Client-1` from the Azure Portal


https://github.com/user-attachments/assets/8a6b4407-a3eb-4611-9afb-018f6bd971ca


3. Log into `Client-1` via RDP

4. Test Network Connectivity
   - Open Command Prompt and run:
     ```cmd
     ping <DC-1_Private_IP>
     ```
   - Ensure ping replies are successful

5. Verify DNS Configuration
   - Open PowerShell and run:
     ```powershell
     ipconfig /all
     ```
   - Confirm the DNS Server entry matches `DC-1`’s private IP address


https://github.com/user-attachments/assets/19afb390-1626-434a-99b3-8253d23edfcc


---

# Azure Windows VMs Active Directory Lab

This guide walks through installing Active Directory on a Domain Controller VM (DC-1), creating users and OUs, joining a Client VM (Client-1) to the domain, and configuring Remote Desktop for non-admin users.  

---

## Prerequisites

- Two Azure VMs already created:
  - **DC-1** (Windows Server, intended as Domain Controller)
  - **Client-1** (Windows 10/11, intended as domain-joined client)
- Both VMs in the same Virtual Network/subnet.
- `labuser` account credentials on both VMs.
- Azure Portal access to start/stop VMs and configure networking.

---

## Part 1 – Install AD DS & Create Domain Accounts

### 1. Start VMs
1. In the Azure Portal, navigate to each VM (DC-1 and Client-1).
2. Click **Start** if either VM is stopped.

### 2. Install Active Directory Domain Services on DC-1
1. **RDP** into **DC-1** as `.\labuser`.
2. Open **Server Manager** → **Add roles and features**.
3. Select **Active Directory Domain Services** and install.
4. After installation, click the **flag** icon in Server Manager and choose **Promote this server to a domain controller**.
5. Choose **Add a new forest**, set **Root domain name** to `mydomain.com` (or your preferred name).
6. Complete the wizard and allow the server to **restart**.

### 3. Create Domain Admin User
1. After reboot, log in as `mydomain.com\labuser` (password: `Cyberlab123!`).
2. Open **Active Directory Users and Computers (ADUC)**.
3. Right-click your domain → **New → Organizational Unit**, name it `_EMPLOYEES`.
4. Right-click your domain → **New → Organizational Unit**, name it `_ADMINS`.
5. Under the `_ADMINS` OU, **New → User**:
   - **First name**: Jane  
   - **Last name**: Doe  
   - **User logon name**: `jane_admin`  
   - **Password**: `Cyberlab123!`  
6. Finish the wizard, then add `jane_admin` to the **Domain Admins** group:
   - Double-click `jane_admin` → **Member Of** tab → **Add…** → enter `Domain Admins`.
7. Sign out, then **RDP** back into **DC-1** as `mydomain.com\jane_admin`.

### 4. Join Client-1 to the Domain
1. Ensure **Client-1** DNS is pointed at **DC-1**’s private IP (already done).
2. In the Azure Portal, restart **Client-1**.
3. RDP into **Client-1** as the local admin (`.\labuser`).
4. Open **System Properties** → **Computer Name** → **Change…**.
5. Select **Domain**, enter `mydomain.com`, click **OK**.
6. Provide `mydomain.com\jane_admin` credentials when prompted.
7. Allow **Client-1** to restart to complete the join.
8. On **DC-1**, open **ADUC**, verify **Client-1** appears in **Computers**.
9. In ADUC, create a new OU named `_CLIENTS`.
10. Drag **Client-1** object into the `_CLIENTS` OU.

> **Tip:** When you’re done for now, you can stop both VMs in the Azure Portal to save costs.

---

## Part 2 – Remote Desktop & Bulk User Creation

### 1. Enable RDP for Domain Users on Client-1
1. In the Azure Portal, **start** DC-1 and Client-1 if they’re off.
2. RDP into **Client-1** as `mydomain.com\jane_admin`.
3. Open **System Properties** → **Remote** tab → **Select** “Allow remote connections to this computer”.
4. Click **Select Users…** → **Add…** → type `Domain Users` → **OK**.
5. All domain users can now RDP into Client-1 (no admin rights needed).

### 2. Bulk-Create Users via PowerShell Script
1. RDP into **DC-1** as `mydomain.com\jane_admin`.
2. Open **Windows PowerShell ISE** **as Administrator**.
3. **File → New**, paste your user-creation script, for example:
   ```powershell
   # Example Bulk User Creation Script
   Import-Module ActiveDirectory

   $users = @(
     @{Name='Alice Smith'; SamAccountName='alice.smith'; OU='_EMPLOYEES'; Password='P@ssw0rd1'},
     @{Name='Bob Jones';   SamAccountName='bob.jones';   OU='_EMPLOYEES'; Password='P@ssw0rd1'},
     # … add more entries here …
   )

   foreach ($u in $users) {
     New-ADUser `
       -Name         $u.Name `
       -SamAccountName $u.SamAccountName `
       -AccountPassword (ConvertTo-SecureString $u.Password -AsPlainText -Force) `
       -Enabled      $true `
       -Path         "OU=$($u.OU),DC=mydomain,DC=com" `
       -PasswordNeverExpires $false
   }
