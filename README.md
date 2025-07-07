# Azure Lab: Domain Controller and Client VM Setup

This lab will guide you through the process of setting up a Windows Server 2022 Domain Controller and a Windows 10 Client within Microsoft Azure. These virtual machines will be used for subsequent labs.

---

## Step 1: Create Resource Group and Network Infrastructure

### Create a Resource Group
- Use a unique name, e.g., `RG-LabDomain`
- Select your preferred Azure region

### Create a Virtual Network and Subnet
- Name: `VNET-LabDomain`
- Subnet: `Subnet-Lab` (e.g., 10.0.0.0/24)

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

### Post-Creation Configuration
1. Set DNS to `DC-1`’s Private IP
   - Go to `Client-1`'s Network Interface settings in Azure
   - Set DNS server to `DC-1`’s Static Private IP

2. Restart `Client-1` from the Azure Portal

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

---

## Lab Completion Notes

- Do not delete the VMs. They will be used in future labs.
- To save Azure costs, stop the VMs in the Azure Portal when not in use.
