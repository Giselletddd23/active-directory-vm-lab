▶[Watch Me Pt1](https://www.loom.com/share/a594b85b530e46f2819f6e7422f4aa82)

▶▶[Pt2](https://www.loom.com/share/67423c589dac4af5b5b746894923b5a7)

▶▶▶[The Finale](https://www.loom.com/share/010914f84cbf4e90b90339fd749e5043)
# Active Directory Home Lab
### Building and Managing a Windows Domain from Scratch

![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202025-0078D4?style=flat-square&logo=windows)
![Environment](https://img.shields.io/badge/Environment-Azure%20%7C%20VirtualBox-232F3E?style=flat-square&logo=amazonaws)
![Level](https://img.shields.io/badge/Level-Beginner%20Friendly-4CAF50?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-lab.local-6A4C9C?style=flat-square)

---

## Overview

This lab walks through building a fully functional Active Directory (AD) domain from scratch using Windows Server 2025 — the same identity infrastructure used in enterprise environments globally. Every task in this lab maps directly to real-world job responsibilities in IT support, sysadmin, cloud engineering, and security roles.

> **Why this matters:** Active Directory is the most targeted system in ransomware attacks and the backbone of every enterprise Windows environment. Understanding how to build it is the foundation of defending it.

---

## Architecture

```
Forest: lab.local
│
└── Domain Controller (lab.local)
    ├── DNS Server
    ├── AD Domain Services (AD DS)
    └── Group Policy Management Console (GPMC)
        │
        ├── OU: IT          → Group: IT_Admins      → User: alice.chen
        ├── OU: Finance     → Group: Finance_Users  → User: bob.patel
        ├── OU: HR          → Group: HR_Users       → User: carol.jones
        ├── OU: Sales       → Group: Sales_Users    → User: david.smith
        └── OU: Computers   → Domain-joined workstations
            │
            └── GPO: IT Security Policy (linked to IT OU)
                    ├── Min password length: 12 characters
                    ├── Password complexity: Enabled
                    ├── Screen lock timer: 900 seconds (15 min)
                    └── USB/Removable storage: Denied
```

---

## What This Lab Covers

| Step | Task | Real-World Application |
|------|------|------------------------|
| 1 | Provision the VM (Azure or VirtualBox) | Infrastructure setup and cost management |
| 2 | Install AD Domain Services role | Server role management |
| 3 | Promote server to Domain Controller | Forest and domain creation |
| 4 | Build OU structure, groups, and users | Identity and access management (IAM) |
| 5 | Configure Group Policy Objects (GPOs) | Centralised policy enforcement |
| 6 | Perform common help desk tasks | Day-one IT support operations |

---

## Prerequisites

**Option A — Azure (Recommended)**
- A free Azure account — [azure.microsoft.com/free](https://azure.microsoft.com/free)
- No local hardware requirements; the VM runs in Microsoft's datacentre

**Option B — VirtualBox (Local)**
- Host machine with **8GB RAM minimum** (4GB for VM, 4GB for host OS)
- **60GB free disk space**
- Quad-core CPU with virtualisation enabled in BIOS
- [VirtualBox](https://www.virtualbox.org) + [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)

---

## VM Configuration (Azure)

| Setting | Value | Reason |
|---------|-------|--------|
| Image | Windows Server 2025 Datacenter — Gen2 | Latest server OS; 180-day eval licence included |
| Size | Standard_B2s (2 vCPU / 4GB RAM) | Smallest size that runs AD comfortably |
| Region | East US | Cheapest region; best free-tier availability |
| Auth | Password | Required for RDP access |
| Inbound port | RDP (3389) | Required to connect from local machine |
| OS disk | Standard SSD | Included in free tier |

> **Cost tip:** Stop (do not delete) the VM after each session. A B2s VM costs ~$0.05/hour. Stopping it pauses compute billing and preserves your $200 free credit.

---

## Lab Steps

### Step 1 — Provision the VM

Spin up the VM using the Azure configuration table above, or configure VirtualBox with the Windows Server 2025 Evaluation ISO.

**Fix clipboard before connecting via RDP:**
1. Open Remote Desktop on your local machine
2. Enter the VM's public IP → click **Show Options**
3. Go to **Local Resources** tab → check **Clipboard**
4. Connect — copy/paste now works in both directions

> Download the `.rdp` file from the Azure portal instead of using the browser console. The native RDP client handles clipboard and performance better.

---

### Step 2 — Install Active Directory Domain Services

RDP into the VM. Open **Server Manager** (launches automatically on login).

**Via GUI:** `Manage → Add Roles and Features → Server Roles → Active Directory Domain Services → Add Features → Install`

**Via PowerShell:**
```powershell
# Install AD DS role and management tools
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Install Group Policy Management Console (required for Step 5)
Install-WindowsFeature -Name GPMC
```

> Install GPMC now. Without it, you will not see **Group Policy Management** in Server Manager's Tools menu and will hit a wall mid-lab.

---

### Step 3 — Promote the Server to a Domain Controller

Installing the role does not create a domain. Promotion is the step that builds the forest, creates the domain, and makes this server the authoritative DNS and identity server for everything that joins it.

**Via GUI:** Click the yellow flag in Server Manager → `Promote this server to a domain controller → Add a new forest → Root domain name: lab.local → Set DSRM password → Install`

**Via PowerShell:**
```powershell
Import-Module ADDSDeployment

Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

> The server restarts automatically after promotion. Log back in as `LAB\Administrator`.

---

### Step 4 — Build the Organisational Structure

Open **Active Directory Users and Computers (ADUC)** from `Server Manager → Tools`.

#### Create Organisational Units

```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

#### Create Security Groups

```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

#### Create Users and Assign Group Memberships

> **Important:** Run the entire block below as one operation. The `$password` variable must be defined before the `New-ADUser` commands execute. Select all, then paste the full block into PowerShell and press Enter.

```powershell
# Define password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Create users in their respective OUs
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Assign users to their department security groups
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

### Step 5 — Configure Group Policy

Open **Group Policy Management** from `Server Manager → Tools`. This is a separate tool from ADUC — GPOs are not created inside Active Directory Users and Computers.

**Create the GPO:**
`Right-click IT OU → Create a GPO in this domain and link it here → Name: IT Security Policy → Right-click → Edit`

**Configure these settings inside the GPO Editor:**

| Policy Path | Setting | Value |
|-------------|---------|-------|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | 12 |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | Enabled |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | 900 seconds |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | Enabled |

**Verify the GPO is applying:**
```powershell
# On the target machine, force a policy refresh
gpupdate /force

# Check which policies are applied
gpresult /r
```

---

### Step 6 — Common Help Desk Tasks

These are the most frequent real-world tasks for any IT support or sysadmin role. Practice each one on the test accounts.

#### Reset a Password

```powershell
# Reset password and force user to change it at next login
Set-ADAccountPassword -Identity "bob.patel" `
  -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)

Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

#### Unlock a Locked Account

```powershell
Unlock-ADAccount -Identity "carol.jones"
```

#### Disable an Account (Offboarding)

```powershell
# Disable — preserves account history and group memberships for audit
Disable-ADAccount -Identity "david.smith"

# Find all currently disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

#### Audit — Find Inactive Accounts

```powershell
# Find active accounts with no login in the last 90 days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Check group membership for a specific user
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification

Run these checks to confirm the lab is correctly configured:

```powershell
# Confirm the Domain Controller is running
Get-ADDomainController

# Confirm all OUs exist
Get-ADOrganizationalUnit -Filter *

# Confirm all users are enabled
Get-ADUser -Filter {Enabled -eq $true}

# Confirm group membership
Get-ADGroupMember -Identity "IT_Admins"

# Confirm GPO is linked to the IT OU
Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'
```

| Check | Expected Result |
|-------|-----------------|
| `Get-ADDomainController` | Returns DC info with forest `lab.local` |
| `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs |
| `Get-ADUser -Filter {Enabled -eq $true}` | Lists all 4 test accounts |
| `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| PowerShell prompts for `Name:` when creating users | You ran `New-ADUser` before defining `$password`. Run the entire script block at once — the `$password` line must come first. |
| Cannot copy/paste into the VM | Open RDP client → Show Options → Local Resources → check Clipboard. Or download the `.rdp` file from the Azure portal and open with the native Remote Desktop app. |
| Promotion fails with DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting. |
| Cannot RDP after domain join | Log in as `LAB\Administrator`, not just `Administrator`. |
| GPO not applying | Run `gpupdate /force` on the target machine, then `gpresult /r` to confirm. |
| User cannot log in after creation | Confirm the account is Enabled and check `ChangePasswordAtLogon` status. |
| ADUC not showing in Tools menu | Run `dsa.msc` from the Run dialog, or execute `Add-WindowsFeature RSAT-ADDS`. |

---

## Skills Demonstrated

- Windows Server 2025 administration
- Active Directory Domain Services (AD DS) deployment
- Forest and domain creation
- Organisational Unit (OU) design
- Role-based access control with Security Groups
- Group Policy Object (GPO) authoring and enforcement
- User lifecycle management: creation, password reset, account unlock, offboarding
- PowerShell automation for AD administration
- Audit and compliance reporting

---

## Relevance to Cloud Roles

This lab builds directly transferable knowledge to Microsoft cloud environments:

| On-Premises AD | Microsoft Entra ID (Azure AD) Equivalent |
|----------------|------------------------------------------|
| Organisational Units | Administrative Units |
| Security Groups | Security Groups / Microsoft 365 Groups |
| Group Policy Objects | Conditional Access Policies, Intune Policies |
| Domain User Accounts | Cloud Identities / Synced Identities (Entra Connect) |
| Domain Join | Entra ID Join / Hybrid Join |

---

## References

- [Microsoft — Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Microsoft — Group Policy Overview](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-overview)
- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Azure Free Account](https://azure.microsoft.com/free)
- [VirtualBox Download](https://www.virtualbox.org)
- [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
