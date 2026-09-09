# Microsoft Entra ID + Intune Cloud-Only Endpoint Management Lab

Building and securing a complete Microsoft 365 tenant with no on-premises Active Directory — identity, device enrollment, configuration, compliance, and access control delivered entirely from the cloud.

![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Microsoft Intune](https://img.shields.io/badge/Microsoft%20Intune-0078D4?style=flat&logo=microsoft&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat&logo=powershell&logoColor=white)
![Microsoft Graph](https://img.shields.io/badge/Microsoft%20Graph-2C2C2C?style=flat&logo=microsoft&logoColor=white)
![Windows 11](https://img.shields.io/badge/Windows%2011%20Pro-0078D6?style=flat&logo=windows11&logoColor=white)
![Conditional Access](https://img.shields.io/badge/Conditional%20Access-107C10?style=flat&logo=microsoft&logoColor=white)

---

## At a Glance

| | |
|---|---|
| **Architecture** | Cloud-only. No Domain Controller, no Entra Connect, no Group Policy. |
| **Tenant domain** | `basitcloudlab.com`, added and DNS-verified in Entra ID |
| **Licensing** | Microsoft 365 Business Premium (Entra ID P1 + Intune) |
| **Identity** | 5 licensed employee accounts across 5 security groups |
| **Automation** | Microsoft Graph PowerShell SDK for users, groups, membership, and licensing |
| **Access control** | 5 Conditional Access policies, MFA, named locations, device compliance |
| **Endpoint** | `CLIENT-INTUNE01` — Windows 11 Pro, Entra joined, Intune managed |
| **Provisioning** | Windows Autopilot user-driven deployment profile |

---

## Table of Contents

**Design**
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Traditional vs Cloud-Only Identity](#traditional-vs-cloud-only-identity)
- [Project Lifecycle](#project-lifecycle)

**Tenant Foundation**
- [Custom Domain](#custom-domain)
- [Microsoft 365 Business Premium](#microsoft-365-business-premium)
- [Employee Structure](#employee-structure)

**Identity Automation with Microsoft Graph PowerShell**
- [Microsoft Graph PowerShell](#microsoft-graph-powershell)
- [Install Microsoft Graph PowerShell](#install-microsoft-graph-powershell)
- [Connect to Microsoft Graph](#connect-to-microsoft-graph)
- [Understanding Mg Commands](#understanding-mg-commands)
- [Bulk User Creation with PowerShell](#bulk-user-creation-with-powershell)
- [Verify Users](#verify-users)
- [Security Groups](#security-groups)
- [Graph Permissions for Group Management](#graph-permissions-for-group-management)
- [Bulk Group Creation](#bulk-group-creation)
- [Verify Groups](#verify-groups)
- [Understanding Entra Object IDs](#understanding-entra-object-ids)
- [Add a User to a Group](#add-a-user-to-a-group)
- [Department Mapping with Hashtable](#department-mapping-with-hashtable)
- [Automated Department Membership](#automated-department-membership)
- [GRP-All-Employees Membership](#grp-all-employees-membership)

**Licensing**
- [Microsoft 365 License Assignment Failure](#microsoft-365-license-assignment-failure)
- [Set Usage Location with Graph PowerShell](#set-usage-location-with-graph-powershell)
- [Business Premium License Assignment](#business-premium-license-assignment)

**Identity Security**
- [Emergency Access Administrator](#emergency-access-administrator)
- [MFA](#mfa)
- [Conditional Access Policies](#conditional-access-policies)

**Endpoint Enrollment**
- [Intune Automatic Enrollment](#intune-automatic-enrollment)
- [Entra Join vs Intune Enrollment](#entra-join-vs-intune-enrollment)
- [Windows 11 Test Device](#windows-11-test-device)
- [Windows OOBE and Cloud Enrollment](#windows-oobe-and-cloud-enrollment)
- [Windows Hello for Business](#windows-hello-for-business)
- [Verify Entra Join](#verify-entra-join)
- [Intune Device Enrollment Verification](#intune-device-enrollment-verification)
- [Device Sync](#device-sync)

**Configuration, Compliance, and Access**
- [Configuration Profiles](#configuration-profiles)
- [Verify Defender from Windows](#verify-defender-from-windows)
- [Windows Update Management](#windows-update-management)
- [Configuration Profile Status](#configuration-profile-status)
- [Compliance Policy](#compliance-policy)
- [Compliance Report](#compliance-report)
- [Configuration vs Compliance vs Conditional Access](#configuration-vs-compliance-vs-conditional-access)
- [CA-Require-Compliant-Device](#ca-require-compliant-device)

**Provisioning**
- [Windows Autopilot](#windows-autopilot)

**Reference**
- [Troubleshooting Summary](#troubleshooting-summary)
- [Useful Commands](#useful-commands)
- [Key Concepts Learned](#key-concepts-learned)
- [Project Outcome](#project-outcome)
- [Important Lab Note](#important-lab-note)

---

## Project Overview

This project documents the design and implementation of a fully cloud-managed Microsoft identity and endpoint environment for a fictional small organization.

The goal was to understand how a modern organization can operate without a traditional on-premises Active Directory environment.

This lab does **not** use:

- Active Directory Domain Services
- Domain Controllers
- Entra Connect
- Traditional Group Policy
- Internal Active Directory DNS
- On-premises endpoint management infrastructure

Instead, the environment uses:

- Microsoft Entra ID
- Microsoft Intune
- Microsoft 365 Business Premium
- Conditional Access
- Multifactor Authentication
- Windows Hello for Business
- Microsoft Defender
- Windows Update for Business
- Microsoft Graph PowerShell
- Windows Autopilot
- Windows 11 Pro

The project was designed as a realistic small-business cloud environment where identity, endpoint management, security, compliance, and access control are managed entirely from Microsoft cloud services.

---

## Architecture

```text
INTERNET
   |
   +-----------------------------+
   |                             |
   v                             v
Microsoft Entra ID           Microsoft Intune
------------------           ----------------
Users                        Enrollment
Groups                       Configuration
MFA                          Compliance
Conditional Access           Defender
Device Identity              Windows Update
Admin Roles                  Autopilot
   |                             |
   +-------------+---------------+
                 |
                 v
          CLIENT-INTUNE01
          Windows 11 Pro
          ----------------
          Entra Joined
          Intune Managed
          Defender Managed
          Update Managed
          Compliance Checked
          Windows Hello
                 |
                 v
           Microsoft 365
```

---

## Traditional vs Cloud-Only Identity

A traditional or hybrid environment may look like:

```text
Windows Client
      |
      v
Domain Controller
      |
      +-- Active Directory
      +-- DNS
      +-- Group Policy
      |
      v
Entra Connect
      |
      v
Microsoft Entra ID
```

This project instead uses a fully cloud-managed architecture:

```text
Windows Client
      |
      +----------------------+
      |                      |
      v                      v
Microsoft Entra ID      Microsoft Intune
      |                      |
  Identity               Device Management
  Authentication         Configuration
  MFA                    Compliance
  Conditional Access     Security
                         Updates
                         Autopilot
```

There is no traditional Domain Controller in this architecture.

---

## Custom Domain

The organization uses:

```text
basitcloudlab.com
```

The domain was added to Microsoft Entra ID and verified through DNS.

This allowed professional user identities such as:

```text
iqra@basitcloudlab.com
basit@basitcloudlab.com
sara@basitcloudlab.com
umer@basitcloudlab.com
hamdan@basitcloudlab.com
```

instead of relying only on the default Microsoft tenant domain.

---

## Microsoft 365 Business Premium

A Microsoft 365 Business Premium trial was activated for the project.

The lab used Business Premium because it provided the services required for the cloud-native environment, including:

- Microsoft Entra ID P1
- Microsoft Intune
- Conditional Access
- Microsoft 365 applications and services
- Microsoft Defender capabilities

The tenant provided 25 trial licenses.

Five employee accounts were licensed.

---

## Employee Structure

| Employee | Department / Role | UPN |
|---|---|---|
| Iqra Khalid | Management | iqra@basitcloudlab.com |
| Basit Aftab Raza | IT / Operations | basit@basitcloudlab.com |
| Sara Ahmed | Finance | sara@basitcloudlab.com |
| Umer Shah | Staff | umer@basitcloudlab.com |
| Hamdan Malik | Staff | hamdan@basitcloudlab.com |

A separate cloud administrator account was retained for tenant administration.

---

## Microsoft Graph PowerShell

Microsoft Graph PowerShell was used for repetitive Entra administration.

The management flow was:

```text
Administrator Workstation
        |
        v
    PowerShell
        |
        v
Microsoft Graph PowerShell SDK
        |
        v
  Microsoft Graph API
        |
        v
 Microsoft Entra ID
```

---

## Install Microsoft Graph PowerShell

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

Explanation:

- `Install-Module` installs additional PowerShell functionality.
- `Microsoft.Graph` is the Microsoft Graph PowerShell SDK.
- `-Scope CurrentUser` installs the module for the current Windows user.

---

## Connect to Microsoft Graph

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

The connection was verified using:

```powershell
Get-MgContext
```

This displays important session information such as:

- Account
- Tenant ID
- Authentication type
- Granted permissions

This is useful before modifying a tenant because it verifies:

```text
WHO am I?
WHICH tenant am I connected to?
WHAT permissions does this session have?
```

---

## Understanding Mg Commands

Microsoft Graph PowerShell commands follow normal PowerShell naming conventions:

```text
Verb-Noun
```

Example:

```powershell
Get-MgUser
```

Breakdown:

```text
Get = Retrieve
Mg  = Microsoft Graph
User = Entra user object
```

Examples used during the lab included:

```powershell
Get-MgUser
New-MgUser
Update-MgUser
Get-MgGroup
New-MgGroup
Get-MgDevice
```

---

## Bulk User Creation with PowerShell

One user was first created manually through the Entra Admin Center to understand the GUI workflow.

The remaining users were created using Microsoft Graph PowerShell.

A password profile was created:

```powershell
$passwordProfile = @{
    Password = "TemporaryPassword"
    ForceChangePasswordNextSignIn = $true
}
```

The employee objects were stored inside an array:

```powershell
$users = @(
    @{
        DisplayName       = "Basit Aftab Raza"
        UserPrincipalName = "basit@basitcloudlab.com"
        MailNickname      = "basit"
    }
    @{
        DisplayName       = "Sara Ahmed"
        UserPrincipalName = "sara@basitcloudlab.com"
        MailNickname      = "sara"
    }
    @{
        DisplayName       = "Umer Shah"
        UserPrincipalName = "umer@basitcloudlab.com"
        MailNickname      = "umer"
    }
    @{
        DisplayName       = "Hamdan Malik"
        UserPrincipalName = "hamdan@basitcloudlab.com"
        MailNickname      = "hamdan"
    }
)
```

A `foreach` loop created each Entra user:

```powershell
foreach ($user in $users) {

    New-MgUser `
        -DisplayName $user.DisplayName `
        -UserPrincipalName $user.UserPrincipalName `
        -MailNickname $user.MailNickname `
        -AccountEnabled:$true `
        -PasswordProfile $passwordProfile
}
```

This introduced several PowerShell concepts:

- Variables
- Arrays
- Hashtables
- `foreach` loops
- Objects
- Object properties
- Graph commands
- PowerShell parameters

---

## Verify Users

```powershell
Get-MgUser -All |
Select-Object DisplayName, UserPrincipalName
```

---

## Security Groups

The following Entra security groups were created:

```text
GRP-All-Employees
GRP-Management
GRP-Finance
GRP-IT
GRP-Staff
```

The group structure was:

```text
GRP-All-Employees
├── Iqra Khalid
├── Basit Aftab Raza
├── Sara Ahmed
├── Umer Shah
└── Hamdan Malik

GRP-Management
└── Iqra Khalid

GRP-Finance
└── Sara Ahmed

GRP-IT
└── Basit Aftab Raza

GRP-Staff
├── Umer Shah
└── Hamdan Malik
```

Groups were configured as:

```text
Group type: Security
Membership type: Assigned
Entra role assignable: No
```

---

## Graph Permissions for Group Management

```powershell
Connect-MgGraph `
    -Scopes "User.ReadWrite.All","Group.ReadWrite.All"
```

The granted scopes were verified with:

```powershell
(Get-MgContext).Scopes
```

---

## Bulk Group Creation

```powershell
$groups = @(
    "GRP-Management",
    "GRP-Finance",
    "GRP-IT",
    "GRP-Staff"
)

foreach ($group in $groups) {

    New-MgGroup `
        -DisplayName $group `
        -MailEnabled:$false `
        -MailNickname ($group -replace "-", "") `
        -SecurityEnabled:$true
}
```

The expression:

```powershell
$group -replace "-", ""
```

removes hyphens from a string.

---

## Verify Groups

```powershell
Get-MgGroup -All |
Select-Object DisplayName, SecurityEnabled, MailEnabled
```

---

## Understanding Entra Object IDs

Microsoft Entra objects have unique Object IDs.

Graph uses these unique IDs to create relationships between directory objects.

User Object IDs:

```powershell
Get-MgUser -All |
Select-Object DisplayName, UserPrincipalName, Id
```

Group Object IDs:

```powershell
Get-MgGroup -All |
Select-Object DisplayName, Id
```

---

## Add a User to a Group

```powershell
$user = Get-MgUser `
    -Filter "userPrincipalName eq 'basit@basitcloudlab.com'"

$group = Get-MgGroup `
    -Filter "displayName eq 'GRP-IT'"

New-MgGroupMemberByRef `
    -GroupId $group.Id `
    -OdataId "https://graph.microsoft.com/v1.0/directoryObjects/$($user.Id)"
```

---

## Department Mapping with Hashtable

```powershell
$departmentMap = @{
    "iqra@basitcloudlab.com"   = "GRP-Management"
    "basit@basitcloudlab.com"  = "GRP-IT"
    "sara@basitcloudlab.com"   = "GRP-Finance"
    "umer@basitcloudlab.com"   = "GRP-Staff"
    "hamdan@basitcloudlab.com" = "GRP-Staff"
}
```

A hashtable stores key/value pairs.

PowerShell's `.GetEnumerator()` was used to process each key/value pair independently:

```powershell
foreach ($entry in $departmentMap.GetEnumerator()) {
    Write-Host "User:" $entry.Key
    Write-Host "Group:" $entry.Value
}
```

---

## Automated Department Membership

```powershell
foreach ($entry in $departmentMap.GetEnumerator()) {

    $user = Get-MgUser `
        -Filter "userPrincipalName eq '$($entry.Key)'"

    $group = Get-MgGroup `
        -Filter "displayName eq '$($entry.Value)'"

    New-MgGroupMemberByRef `
        -GroupId $group.Id `
        -OdataId "https://graph.microsoft.com/v1.0/directoryObjects/$($user.Id)"

    Write-Host "Added $($user.DisplayName) to $($group.DisplayName)"
}
```

---

## GRP-All-Employees Membership

```powershell
$allEmployees = @(
    "iqra@basitcloudlab.com",
    "basit@basitcloudlab.com",
    "sara@basitcloudlab.com",
    "umer@basitcloudlab.com",
    "hamdan@basitcloudlab.com"
)

$allGroup = Get-MgGroup `
    -Filter "displayName eq 'GRP-All-Employees'"

foreach ($upn in $allEmployees) {

    $user = Get-MgUser `
        -Filter "userPrincipalName eq '$upn'"

    New-MgGroupMemberByRef `
        -GroupId $allGroup.Id `
        -OdataId "https://graph.microsoft.com/v1.0/directoryObjects/$($user.Id)"

    Write-Host "Added $($user.DisplayName) to GRP-All-Employees"
}
```

---

## Microsoft 365 License Assignment Failure

Initial license assignment failed because:

```text
Usage Location missing
```

Microsoft requires a valid usage location before many cloud licenses can be assigned.

---

## Set Usage Location with Graph PowerShell

```powershell
foreach ($upn in $allEmployees) {

    Update-MgUser `
        -UserId $upn `
        -UsageLocation "US"

    Write-Host "Set usage location to US for $upn"
}
```

Verification:

```powershell
Get-MgUser `
    -All `
    -Property DisplayName,UserPrincipalName,UsageLocation |
Where-Object {
    $_.UserPrincipalName -like "*@basitcloudlab.com"
} |
Select-Object DisplayName,UserPrincipalName,UsageLocation
```

---

## Business Premium License Assignment

Microsoft 365 Business Premium was assigned to all five employees.

This enabled:

- Microsoft Intune
- Entra ID P1
- Conditional Access
- Microsoft 365 applications
- Defender capabilities

---

## Emergency Access Administrator

Before enforcing Conditional Access, an emergency account was created and assigned:

```text
Global Administrator
```

The account was excluded from broad Conditional Access policies and was not used for normal daily administration.

---

## MFA

Multifactor Authentication was configured for employees using Microsoft Authenticator.

---

## Conditional Access Policies

### CA001 - Require MFA for All Users

```text
Users:
All users

Exclude:
Emergency Access Admin

Target:
All resources

Grant:
Require multifactor authentication
```

The policy was initially deployed in Report-only and then enabled after testing.

### CA002 - Block Legacy Authentication

A policy was created to evaluate/block legacy authentication clients and was left in Report-only during the lab.

### CA003 - Require MFA for Admin Roles

Privileged roles were targeted separately, including Global Administrator, User Administrator, Conditional Access Administrator, Intune Administrator, and Security Administrator.

The emergency admin was excluded.

### Named Location

A named location was created:

```text
NL-US
```

representing:

```text
United States
```

### Location-Based Conditional Access

A Report-only location policy was built with:

```text
Include:
Any location

Exclude:
NL-US

Grant:
Block access
```

---

## Intune Automatic Enrollment

Path:

```text
Intune Admin Center
    |
    v
Devices
    |
    v
Enrollment
    |
    v
Windows
    |
    v
Automatic Enrollment
```

The setting changed from:

```text
MDM User Scope = None
```

to:

```text
MDM User Scope = All
```

This means eligible licensed users who join Windows devices to Entra can automatically enroll those devices into Intune.

---

## Entra Join vs Intune Enrollment

These are separate processes:

```text
Microsoft Entra Join
    |
    +-- Creates device identity

Intune Enrollment
    |
    +-- Gives Intune management control
```

---

## Windows 11 Test Device

A Windows 11 Pro VM was created using VirtualBox.

Hostname:

```text
CLIENT-INTUNE01
```

Virtual hardware configuration included:

- Windows 11 Pro
- UEFI
- Secure Boot
- TPM 2.0
- Virtual CPU
- Virtual RAM
- Virtual disk

---

## Windows OOBE and Cloud Enrollment

During Windows setup, `Set up for work or school` was selected.

A licensed cloud employee account was used.

This triggered:

```text
Entra authentication
      |
      v
Microsoft Entra Join
      |
      v
Automatic Intune enrollment
```

---

## Windows Hello for Business

Windows Hello was configured during device setup and a PIN was created.

The PIN is device-specific and can use TPM protection.

---

## Verify Entra Join

```powershell
dsregcmd /status
```

Important results included:

```text
AzureAdJoined : YES
DeviceAuthStatus : SUCCESS
Device Name : CLIENT-INTUNE01
```

This confirmed the device was Microsoft Entra joined.

---

## Intune Device Enrollment Verification

The device appeared in Intune as:

```text
CLIENT-INTUNE01
Managed by: Intune
```

---

## Device Sync

Portal:

```text
Devices
   |
   v
CLIENT-INTUNE01
   |
   v
Sync
```

Windows:

```text
Settings
   |
   v
Accounts
   |
   v
Access work or school
   |
   v
Connected organization
   |
   v
Info
   |
   v
Sync
```

---

## Configuration Profiles

Configuration profiles answer:

> How should this Windows device be configured?

### WIN-CFG-Corporate-Baseline

A Settings Catalog profile configured:

```text
Max Inactivity Time Device Lock
```

### WIN-CFG-Security-Baseline

Defender settings included:

```text
Turn off Microsoft Defender Antivirus = Disabled
Turn off Real-time Protection = Disabled
```

Because these are negatively worded controls, `Disabled` means Defender and Real-time Protection remain enabled.

---

## Verify Defender from Windows

```powershell
Get-MpComputerStatus |
Select-Object AntivirusEnabled,RealTimeProtectionEnabled
```

Expected:

```text
AntivirusEnabled          True
RealTimeProtectionEnabled True
```

Windows Security also displayed:

```text
This setting is managed by your administrator
```

---

## Windows Update Management

Windows Update was managed from Intune's Windows Updates area.

An Update Ring was created with settings including:

- Microsoft product updates: Allow
- Windows drivers: Allow
- Quality update deferral: 0 days
- Feature update deferral: 7 days
- Upgrade Windows 10 devices to Windows 11: No
- Feature update uninstall period: 10 days
- Pre-release builds: Not configured
- Pause Windows Updates: Disabled
- Check for updates: Enabled
- Feature update deadline: 7 days
- Quality update deadline: 3 days
- Grace period: 2 days
- Auto reboot before deadline: No

The endpoint displayed organization-managed Windows Update behavior.

---

## Configuration Profile Status

Intune reported:

```text
WIN-CFG-Corporate-Baseline
Status: Succeeded

WIN-CFG-Security-Baseline
Status: Succeeded
```

---

## Compliance Policy

A Windows compliance policy was created:

```text
WIN-COMP-Corporate-Compliance
```

### Device Health Requirements

- BitLocker: Require
- Secure Boot: Require
- Code Integrity: Require

### Device Security Requirements

- Firewall: Require
- TPM: Require
- Antivirus: Require
- Antispyware: Require

### Defender Compliance Requirements

- Microsoft Defender Antimalware
- Security Intelligence up-to-date
- Real-time Protection

### Noncompliance Action

```text
Mark device noncompliant
Immediately
```

---

## Compliance Report

Example results included:

```text
Firewall                           Compliant
Antivirus                          Compliant
Antispyware                        Compliant
BitLocker                          Compliant
Code Integrity                     Compliant
Defender Antimalware               Compliant
Real-time Protection               Compliant
Secure Boot                        Compliant
TPM                                Compliant
Security Intelligence Up-to-date   Not Compliant
```

This demonstrated how Intune administrators can identify the exact compliance requirement causing a failure.

---

## Configuration vs Compliance vs Conditional Access

```text
CONFIGURATION PROFILE
        |
        v
Configure device
        |
        v
COMPLIANCE POLICY
        |
        v
Evaluate device
        |
        +-- Compliant
        |
        +-- Noncompliant
        |
        v
CONDITIONAL ACCESS
        |
        v
Allow / Block cloud access
```

Easy mental model:

```text
CONFIGURE
   |
   v
CHECK
   |
   v
DECIDE ACCESS
```

---

## CA-Require-Compliant-Device

Grant control:

```text
Require device to be marked as compliant
```

The policy was deployed in Report-only.

### Troubleshooting - Network Not Matched

An initial sign-in showed:

```text
Report-only: Not applied
```

because the network condition did not match due to a named-location exclusion.

### Troubleshooting - Resource Not Matched

Another event showed:

```text
Resource:
Microsoft Graph

Result:
Not matched
```

because the sign-in targeted a different resource than those originally selected.

### Troubleshooting - Device Unknown

Another event showed:

```text
Device:
Unknown

Require compliant device:
Not satisfied

Report-only:
Failure
```

The endpoint itself was verified healthy using:

```powershell
dsregcmd /status
```

and showed:

```text
AzureAdJoined : YES
DeviceAuthStatus : SUCCESS
Device Name : CLIENT-INTUNE01
```

This demonstrated that a user simply owning a compliant device is not enough. Conditional Access must associate the current sign-in with the specific managed device.

### Successful Compliance-Based Conditional Access Test

A later sign-in successfully presented the device identity.

The sign-in evaluation showed:

```text
Policy:
CA-Require-Compliant-Device

User:
Matched

Resource:
Matched

Device Platform:
Matched

Device ID:
Recognized

Grant Control:
Require compliant device

Result:
Satisfied

Report-only:
Success
```

This validated:

```text
CLIENT-INTUNE01
      |
      v
Intune Compliance
      |
      v
Microsoft Entra
      |
      v
Conditional Access
      |
      v
Require compliant device
      |
      v
Satisfied
```

---

## Windows Autopilot

A Windows Autopilot deployment profile was created:

```text
AP-Corporate-User-Driven
```

Configuration included:

```text
Deployment mode:
User-driven

Join type:
Microsoft Entra joined

User account type:
Standard

Microsoft Software License Terms:
Hide

Privacy settings:
Hide
```

The setting:

```text
Convert all targeted devices to Autopilot
```

was changed to:

```text
Yes
```

This allows eligible existing Intune-managed devices targeted by the profile to register with Windows Autopilot.

Registration may take time.

### User-Driven Autopilot Flow

```text
Company device
      |
      v
Hardware registered with Autopilot
      |
      v
Employee turns device on
      |
      v
Connects to Internet
      |
      v
Microsoft recognizes hardware
      |
      v
Autopilot deployment profile downloaded
      |
      v
Corporate OOBE
      |
      v
Employee enters Entra credentials
      |
      v
Microsoft Entra Join
      |
      v
Automatic Intune enrollment
      |
      v
Configuration profiles
      |
      v
Defender policies
      |
      v
Windows Update policies
      |
      v
Compliance evaluation
      |
      v
Managed corporate endpoint
```

### Manual Entra Join vs Autopilot

Without Autopilot:

```text
New Windows computer
      |
      v
Normal Windows OOBE
      |
      v
Employee manually selects
"Set up for work or school"
      |
      v
Entra credentials
      |
      v
Entra Join
      |
      v
Automatic Intune enrollment
```

With Autopilot:

```text
Company-registered Windows computer
      |
      v
Connect to Internet
      |
      v
Autopilot recognizes hardware
      |
      v
Corporate deployment profile
      |
      v
Controlled OOBE
      |
      v
Employee authentication
      |
      v
Entra Join
      |
      v
Intune enrollment
```

Autopilot is not required for Intune enrollment. Its purpose is to automate and standardize the provisioning experience.

---

## Troubleshooting Summary

### License Assignment Failure

Cause:

```text
Missing Usage Location
```

Resolution:

```powershell
Update-MgUser `
    -UserId "user@basitcloudlab.com" `
    -UsageLocation "US"
```

### Intune Policy Partial Success

The device sync initially reported partial success, so individual policy status was checked rather than relying only on the summary banner.

### Defender Compliance Issue

The compliance report showed:

```text
Security Intelligence Up-to-date
Not Compliant
```

while most other checks passed.

### Conditional Access - Network Not Matched

A Named Location exclusion caused the sign-in to fall outside policy scope.

### Conditional Access - Resource Not Matched

A Microsoft Graph sign-in did not match the originally selected target resources.

### Conditional Access - Device Unknown

A sign-in initially did not carry a usable managed-device identity, so the compliant-device grant control could not be satisfied.

A later sign-in correctly presented the device ID and returned:

```text
Report-only: Success
```

---

## Useful Commands

Install Graph:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

Connect:

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

Connect for users and groups:

```powershell
Connect-MgGraph `
    -Scopes "User.ReadWrite.All","Group.ReadWrite.All"
```

Check connection:

```powershell
Get-MgContext
```

Show scopes:

```powershell
(Get-MgContext).Scopes
```

List users:

```powershell
Get-MgUser -All
```

Display user properties:

```powershell
Get-MgUser -All |
Select-Object DisplayName,UserPrincipalName,Id
```

List groups:

```powershell
Get-MgGroup -All |
Select-Object DisplayName,Id
```

Find user:

```powershell
Get-MgUser `
    -Filter "userPrincipalName eq 'sara@basitcloudlab.com'"
```

Find group:

```powershell
Get-MgGroup `
    -Filter "displayName eq 'GRP-Finance'"
```

Set usage location:

```powershell
Update-MgUser `
    -UserId "sara@basitcloudlab.com" `
    -UsageLocation "US"
```

Verify Entra Join:

```powershell
dsregcmd /status
```

Verify Defender:

```powershell
Get-MpComputerStatus |
Select-Object AntivirusEnabled,RealTimeProtectionEnabled
```

---

## Key Concepts Learned

### Microsoft Entra ID

- Cloud users
- Groups
- Authentication
- MFA
- Administrative roles
- Device identity
- Conditional Access

### Microsoft Intune

- Device enrollment
- Configuration profiles
- Compliance
- Endpoint security
- Windows Update
- Device inventory
- Remote device management
- Autopilot

### Configuration Profiles

> How should the endpoint be configured?

### Compliance Policies

> Does the endpoint meet the company's security requirements?

### Conditional Access

> Based on user, device, resource, location, and authentication context, should access be allowed?

### Windows Autopilot

> How should a company-owned Windows device be provisioned when an employee receives or resets it?

---

## Project Lifecycle

```text
Custom Domain
      |
      v
Cloud Users
      |
      v
Security Groups
      |
      v
Microsoft Graph Automation
      |
      v
Microsoft 365 Licensing
      |
      v
MFA
      |
      v
Conditional Access
      |
      v
Windows 11
      |
      v
Microsoft Entra Join
      |
      v
Automatic Intune Enrollment
      |
      v
Configuration Profiles
      |
      v
Defender Security
      |
      v
Windows Update Management
      |
      v
Compliance Policy
      |
      v
Conditional Access with Device Compliance
      |
      v
Windows Autopilot
```

---

## Project Outcome

This project provided hands-on experience building a fully cloud-managed Microsoft identity and Windows endpoint environment.

The lab connected:

```text
Microsoft Entra ID
        +
Microsoft Graph PowerShell
        +
Microsoft Intune
        +
Conditional Access
        +
Microsoft Defender
        +
Windows Update
        +
Device Compliance
        +
Windows Autopilot
```

The most important lesson was understanding how the systems work together:

```text
Entra authenticates the user
        |
        v
Intune manages the device
        |
        v
Configuration profiles configure it
        |
        v
Compliance policies evaluate it
        |
        v
Conditional Access uses those signals
        |
        v
Microsoft cloud access is allowed or denied
```

---

## Important Lab Note

This project was created as a personal hands-on lab.

It demonstrates Microsoft Entra ID, Microsoft Intune, Conditional Access, Microsoft Graph PowerShell, Windows endpoint management, compliance, Microsoft Defender, Windows Update, and Windows Autopilot concepts in a controlled environment.

It does not represent production administration experience.

The Windows Autopilot deployment profile and existing-device conversion workflow were configured during the lab.

The final validation planned for the follow-up session is:

```text
Wipe
  |
  v
Autopilot OOBE
  |
  v
New employee authentication
  |
  v
Entra Join
  |
  v
Automatic Intune enrollment
```

Once CLIENT-INTUNE01 completes Windows Autopilot registration, the device can be wiped and reprovisioned through the Autopilot user-driven OOBE to validate the full provisioning workflow.
