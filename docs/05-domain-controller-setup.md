# Domain Controller Setup

## Overview

The domain controller provides the identity, authentication, DNS, and centralized management foundation for the Acer Blue Team CyberLab.

The server represents a simplified enterprise identity system and supports hands-on practice with:

* Windows Server administration
* Active Directory Domain Services
* Internal DNS
* Domain authentication
* User and group management
* Computer account management
* Group Policy
* Windows security events
* Security monitoring
* Identity-related incident investigation

This document describes a sanitized deployment process suitable for a public GitHub repository.

All domain names, IP addresses, usernames, passwords, file paths, product keys, and system identifiers are placeholders.

---

## Domain Controller Role

The domain controller is responsible for:

* Hosting the CyberLab Active Directory domain
* Authenticating domain users and computers
* Providing internal DNS resolution
* Managing directory objects
* Applying Group Policy
* Maintaining domain security logs
* Supporting endpoint domain membership
* Generating identity-related telemetry
* Providing a controlled environment for defensive security exercises

The domain controller is one of the most sensitive systems in the CyberLab.

Changes should be documented and protected with stable snapshots or backups.

---

## Public Example Configuration

The public documentation uses the following placeholder values:

```text
Server hostname: DC01
Domain name: cyberlab.example
NetBIOS name: CYBERLAB
Host-only address: 192.0.2.10
Subnet prefix: /24
Preferred DNS server: 192.0.2.10
Default gateway: none for isolated operation
```

The `192.0.2.0/24` network is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Architecture Placement

The domain controller is connected primarily to the VMware host-only network.

```text
Acer Windows Host
        |
        | VMware Host-Only Network
        |
       DC01
        |
        | DNS, Kerberos, LDAP, SMB, RPC, Group Policy
        |
  Windows Endpoint
```

The domain controller does not require:

* Direct public internet exposure
* Router port forwarding
* Bridged networking
* Access from untrusted household devices

A temporary NAT adapter may be enabled for updates when required.

---

## Security Boundary

The domain controller controls:

* Domain identities
* Computer trust relationships
* Authentication policy
* Group membership
* Administrative privileges
* Internal DNS records
* Group Policy configuration

Compromise or misconfiguration of the domain controller can affect every domain-joined system.

For this reason:

* Use dedicated lab credentials.
* Do not reuse personal passwords.
* Do not use production data.
* Limit administrator access.
* Keep the server isolated.
* Take snapshots before major changes.
* Maintain recovery documentation.
* Do not expose management services publicly.

---

## Prerequisites

Before building the domain controller, confirm that the following are available:

* VMware Workstation
* A configured host-only VMware network
* A supported Windows Server installation image
* Sufficient CPU, memory, and storage
* A planned internal domain name
* A planned static IP address
* A unique local administrator password
* A secure Directory Services Restore Mode password
* Adequate host disk space
* A clean baseline snapshot strategy

---

## Domain Naming Considerations

The internal domain name should be selected before Active Directory is installed.

For public documentation, use a reserved example domain such as:

```text
cyberlab.example
```

The `.example` top-level domain is intended for documentation and examples.

Avoid publishing the actual internal domain.

---

## Recommended Naming Practices

A lab domain name should be:

* Easy to recognize
* Separate from production identities
* Consistent across documentation
* Free from personal names
* Free from street or location information
* Clearly associated with the lab
* Unlikely to conflict with a public domain

Avoid names such as:

```text
<PERSONAL_NAME>.local
<HOME_ADDRESS>.local
<REAL_COMPANY_DOMAIN>
```

---

## Single-Label Domain Names

Avoid creating a domain with only one label, such as:

```text
CYBERLAB
```

Use a fully qualified domain name instead:

```text
cyberlab.example
```

The NetBIOS name may still use the shorter form:

```text
CYBERLAB
```

---

## Virtual Machine Creation

Create a dedicated Windows Server virtual machine.

Suggested public VM name:

```text
DC01
```

Recommended virtual hardware considerations:

| Resource           | General Guidance                                        |
| ------------------ | ------------------------------------------------------- |
| Virtual processors | Start modestly and increase only if needed              |
| Memory             | Enough for Windows Server, AD DS, DNS, and updates      |
| Storage            | Sufficient for the OS, logs, updates, and snapshots     |
| Network adapter    | Host-only                                               |
| Optical drive      | Windows Server ISO during installation                  |
| Firmware           | Use a supported VMware default                          |
| TPM                | Optional unless required by the selected server version |

The exact resource values depend on the physical host.

---

## Virtual Disk Configuration

A dynamically growing virtual disk is generally sufficient for a small personal CyberLab.

The disk should have enough capacity for:

* Windows Server
* Updates
* Active Directory database
* DNS data
* Event logs
* Group Policy files
* Temporary installation files
* Snapshot growth

Avoid filling the virtual disk close to capacity.

Low disk space can cause:

* Update failures
* Directory service problems
* Logging failures
* Snapshot growth issues
* General instability

---

## Network Adapter Configuration

The domain controller should initially use one host-only adapter.

Recommended VMware settings:

```text
Network Adapter 1
Type: Custom
VMnet: designated host-only network
Connected: enabled
Connect at power on: enabled
```

Do not use bridged networking for the standard deployment.

A temporary NAT adapter can be added after the initial domain configuration if updates are required.

---

## Windows Server Installation

1. Create the virtual machine.
2. Attach the Windows Server ISO.
3. Start the virtual machine.
4. Select the appropriate language and keyboard layout.
5. Choose the intended Windows Server edition.
6. Select the desktop experience option when a graphical interface is desired.
7. Accept the license terms.
8. Choose a custom installation.
9. Select the virtual disk.
10. Complete the operating system installation.
11. Set a strong local Administrator password.
12. Sign in after installation.
13. Install VMware Tools.
14. Restart the server if required.
15. Confirm the virtual hardware is functioning.

Do not record product keys, activation identifiers, or passwords in the public repository.

---

## VMware Tools

VMware Tools improves:

* Network driver support
* Display behavior
* Time synchronization
* Graceful shutdown
* Mouse integration
* Guest management

Install VMware Tools through VMware Workstation using the trusted installation media associated with the product.

After installation:

1. Restart the server.
2. Confirm the network adapter is available.
3. Confirm the display resizes correctly.
4. Confirm VMware can initiate a guest shutdown.
5. Review time synchronization behavior.

---

## Initial Server Configuration

Before promoting the server:

1. Confirm the correct Windows edition.
2. Set the correct timezone.
3. Confirm the system time.
4. Rename the computer.
5. Configure the static host-only address.
6. Configure DNS.
7. Install updates where practical.
8. Review the firewall.
9. Confirm the server can restart normally.
10. Create a clean baseline snapshot.

---

## Rename the Server

A domain controller should use a stable hostname.

Public example:

```text
DC01
```

### PowerShell Method

Run PowerShell as Administrator:

```powershell
Rename-Computer -NewName "DC01" -Restart
```

Administrator privileges are required.

The server restarts immediately when `-Restart` is included.

---

## Verify the Hostname

After restart:

```powershell
hostname
```

Or:

```powershell
Get-ComputerInfo |
    Select-Object CsName, WindowsProductName, WindowsVersion
```

Expected public example:

```text
DC01
```

---

## Identify the Network Adapter

Review network interfaces:

```powershell
Get-NetAdapter
```

Review detailed configuration:

```powershell
Get-NetIPConfiguration
```

The host-only adapter may initially use DHCP or an automatically assigned address.

Record the correct adapter alias before configuring a static address.

---

## Rename the Network Adapter

Renaming the adapter can reduce confusion.

Example:

```powershell
Rename-NetAdapter `
    -Name "<CURRENT_ADAPTER_NAME>" `
    -NewName "CyberLab-Internal"
```

Administrator privileges are required.

Confirm the new name:

```powershell
Get-NetAdapter
```

---

## Configure a Static IP Address

Run PowerShell as Administrator.

Example:

```powershell
New-NetIPAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -IPAddress "192.0.2.10" `
    -PrefixLength 24
```

For the isolated host-only adapter, do not add a default gateway unless the architecture specifically requires routing.

---

## Remove an Existing IP Configuration

If the adapter already has an incorrect static address:

```powershell
Get-NetIPAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -AddressFamily IPv4
```

Remove only the incorrect address:

```powershell
Remove-NetIPAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -IPAddress "<INCORRECT_ADDRESS>" `
    -Confirm:$false
```

Administrator privileges are required.

Use caution to avoid removing the address from the wrong adapter.

---

## Configure DNS Before Promotion

The future domain controller should use its own static IP address as its preferred DNS server.

```powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -ServerAddresses "192.0.2.10"
```

Administrator privileges are required.

Do not configure a public DNS server as the preferred resolver on the domain controller.

External lookups will later be handled through DNS forwarders.

---

## Review the Static Configuration

```powershell
Get-NetIPConfiguration `
    -InterfaceAlias "CyberLab-Internal"
```

```powershell
Get-DnsClientServerAddress `
    -InterfaceAlias "CyberLab-Internal"
```

```powershell
ipconfig /all
```

Confirm:

* The address is correct.
* The subnet prefix is correct.
* The adapter has no unintended gateway.
* The DNS server points to the future domain controller address.
* No second adapter is creating an unexpected route.

---

## Test Host-Only Connectivity

From the Windows Server VM, test the VMware host adapter:

```powershell
ping <VMWARE_HOST_ADAPTER>
```

From the Acer host, test the server:

```powershell
ping <DOMAIN_CONTROLLER>
```

A failed ping may be caused by Windows Firewall blocking ICMP.

Also test the network path using:

```powershell
Test-NetConnection <DOMAIN_CONTROLLER>
```

Do not disable the firewall broadly simply to make ping work.

---

## Windows Updates

Windows Server should be updated before or soon after promotion.

Updates may require temporary internet access.

A safe approach is:

1. Shut down the VM.
2. Add a temporary VMware NAT adapter.
3. Start the VM.
4. Confirm the host-only adapter still has the static address.
5. Confirm the NAT adapter receives a VMware address.
6. Install updates.
7. Restart as required.
8. Confirm Active Directory and DNS remain healthy.
9. Disconnect or remove the NAT adapter when finished.

---

## Temporary NAT Adapter Design

The temporary NAT adapter should normally use:

```text
Addressing: VMware DHCP
Default gateway: VMware NAT gateway
Purpose: updates and software downloads
```

The host-only adapter should retain:

```text
Static internal address
No default gateway
Internal DNS configuration
```

Avoid configuring two default gateways.

---

## Multi-Homed Domain Controller Warning

Domain controllers can behave unpredictably when multiple network adapters are active.

Potential issues include:

* Registration of the NAT address in DNS
* Clients resolving DC01 to the wrong address
* Incorrect default route
* Authentication failures
* Group Policy failures
* DNS queries leaving through the wrong adapter
* Services binding to the wrong address

The preferred long-term design is a single internal adapter unless multi-homing is specifically required.

---

## Prevent NAT Address Registration in DNS

When a temporary NAT adapter is used, prevent it from registering its address in internal DNS.

Graphical method:

1. Open the NAT adapter properties.
2. Open **Internet Protocol Version 4** properties.
3. Select **Advanced**.
4. Open the **DNS** tab.
5. Clear **Register this connection’s addresses in DNS**.
6. Apply the changes.

PowerShell method:

```powershell
Set-DnsClient `
    -InterfaceAlias "<NAT_ADAPTER>" `
    -RegisterThisConnectionsAddress $false
```

Administrator privileges are required.

---

## Install Active Directory Domain Services

Open PowerShell as Administrator.

Install the AD DS role and management tools:

```powershell
Install-WindowsFeature `
    -Name AD-Domain-Services `
    -IncludeManagementTools
```

Administrator privileges are required.

Review the result and confirm that a restart is not unexpectedly required before continuing.

---

## Verify Role Installation

```powershell
Get-WindowsFeature AD-Domain-Services
```

Expected state:

```text
Installed
```

The DNS role is normally installed as part of the new forest promotion process.

---

## Promote the Server to a Domain Controller

Create a new forest using the sanitized example domain:

```powershell
Install-ADDSForest `
    -DomainName "cyberlab.example" `
    -DomainNetbiosName "CYBERLAB" `
    -InstallDNS `
    -Force
```

The command prompts for a Directory Services Restore Mode password.

Administrator privileges are required.

The server restarts after promotion.

---

## Directory Services Restore Mode Password

The Directory Services Restore Mode password is used for specific recovery operations.

It should be:

* Unique
* Strong
* Stored in a password manager
* Different from the domain Administrator password
* Excluded from scripts
* Excluded from screenshots
* Excluded from GitHub
* Available during disaster recovery

Do not place this password in notes stored inside the domain controller.

---

## First Sign-In After Promotion

After restart, sign in using the domain Administrator account.

Public example:

```text
CYBERLAB\Administrator
```

The real password should never appear in public documentation.

Confirm that Server Manager recognizes the machine as a domain controller.

---

## Verify Domain Controller Status

Run PowerShell as Administrator or as an account with appropriate domain permissions:

```powershell
Get-ADDomain
```

```powershell
Get-ADForest
```

```powershell
Get-ADDomainController
```

Expected information includes:

* Domain name
* Forest name
* NetBIOS name
* Domain controller hostname
* Site name
* Functional levels

---

## Verify Installed Roles

```powershell
Get-WindowsFeature |
    Where-Object InstallState -eq "Installed"
```

Confirm that the following are present:

* Active Directory Domain Services
* DNS Server
* Group Policy Management
* Required management tools

---

## Verify Active Directory Services

```powershell
Get-Service NTDS, DNS, Netlogon, Kdc
```

Expected services should be running.

Relevant services include:

| Service  | Purpose                                          |
| -------- | ------------------------------------------------ |
| NTDS     | Active Directory Domain Services                 |
| DNS      | Domain Name System                               |
| Netlogon | Domain authentication and secure channel support |
| Kdc      | Kerberos Key Distribution Center                 |

---

## Active Directory Database and SYSVOL

Important domain controller data includes:

```text
Active Directory database: NTDS.dit
SYSVOL: Group Policy and logon-related files
```

These should not be manually edited.

SYSVOL is normally available through administrative shares after successful promotion.

Review shares:

```powershell
Get-SmbShare
```

Expected domain-related shares include:

```text
NETLOGON
SYSVOL
```

---

## Verify DNS Zones

Open DNS Manager or use PowerShell:

```powershell
Get-DnsServerZone
```

Expected zones include:

* The internal domain zone
* The `_msdcs` zone
* Reverse lookup zones if created
* Supporting system zones

The internal zone should be Active Directory-integrated.

---

## Verify DNS Service Records

Active Directory relies on DNS service records.

Query domain controller records:

```powershell
Resolve-DnsName `
    -Name "_ldap._tcp.dc._msdcs.cyberlab.example" `
    -Type SRV
```

Query Kerberos records:

```powershell
Resolve-DnsName `
    -Name "_kerberos._tcp.cyberlab.example" `
    -Type SRV
```

Expected results should reference the domain controller.

---

## Verify Hostname Resolution

```powershell
Resolve-DnsName "DC01.cyberlab.example"
```

```powershell
nslookup DC01.cyberlab.example
```

The result should return the host-only address rather than a NAT address.

---

## DNS Forwarders

DNS forwarders allow the domain controller to resolve external domains through an approved upstream resolver.

The Windows endpoint continues to query the domain controller.

```text
Windows Endpoint
        |
        | Internal DNS request
        v
      DC01 DNS
        |
        | External query when required
        v
Approved Upstream Resolver
```

---

## Configure a DNS Forwarder

Run PowerShell as Administrator:

```powershell
Add-DnsServerForwarder `
    -IPAddress "<APPROVED_UPSTREAM_DNS>" `
    -PassThru
```

Use an upstream resolver appropriate for the environment.

Do not publish the actual home-network resolver configuration unless it is intentionally public.

---

## Review DNS Forwarders

```powershell
Get-DnsServerForwarder
```

Confirm:

* Only intended forwarders are present.
* No mistyped addresses exist.
* External resolution works when internet access is available.
* Internal queries remain authoritative on the domain controller.

---

## Test External DNS Resolution

With temporary NAT access available:

```powershell
Resolve-DnsName example.com
```

A successful external lookup verifies the forwarding path.

It does not replace internal Active Directory DNS validation.

---

## Reverse Lookup Zone

A reverse lookup zone allows IP addresses to resolve back to hostnames.

It is useful for:

* Troubleshooting
* Log interpretation
* Network analysis
* SIEM enrichment
* Administrative tools

A reverse zone is helpful but not always required for a small lab.

---

## Create a Reverse Lookup Zone

Example PowerShell command:

```powershell
Add-DnsServerPrimaryZone `
    -NetworkId "192.0.2.0/24" `
    -ReplicationScope "Domain"
```

Administrator privileges are required.

The documentation network is only an example.

---

## Create a PTR Record

```powershell
Add-DnsServerResourceRecordPtr `
    -Name "10" `
    -ZoneName "2.0.192.in-addr.arpa" `
    -PtrDomainName "DC01.cyberlab.example"
```

The reverse zone name depends on the actual subnet.

Verify:

```powershell
Resolve-DnsName "192.0.2.10"
```

---

## Active Directory Sites and Services

A single-domain-controller lab normally uses the default site.

Open:

```text
Active Directory Sites and Services
```

Confirm that DC01 appears under:

```text
Default-First-Site-Name
```

The default site name can be renamed for clarity, but this is optional.

Example public site name:

```text
CyberLab-Site
```

---

## Associate the Lab Subnet with the Site

Creating a subnet object improves directory topology documentation.

Example:

```powershell
New-ADReplicationSubnet `
    -Name "192.0.2.0/24" `
    -Site "CyberLab-Site"
```

Appropriate domain permissions are required.

Use the operational subnet privately, not the documentation subnet.

---

## Organizational Unit Design

Organizational units allow accounts and systems to be grouped logically.

Example structure:

```text
CyberLab
├── Users
├── Administrators
├── Service Accounts
├── Workstations
├── Servers
├── Security Groups
└── Disabled Objects
```

This is more manageable than leaving every object in the default containers.

---

## Create the Root Organizational Unit

```powershell
New-ADOrganizationalUnit `
    -Name "CyberLab" `
    -Path "DC=cyberlab,DC=example" `
    -ProtectedFromAccidentalDeletion $true
```

Appropriate domain permissions are required.

---

## Create Child Organizational Units

```powershell
$base = "OU=CyberLab,DC=cyberlab,DC=example"

"Users",
"Administrators",
"Service Accounts",
"Workstations",
"Servers",
"Security Groups",
"Disabled Objects" |
ForEach-Object {
    New-ADOrganizationalUnit `
        -Name $_ `
        -Path $base `
        -ProtectedFromAccidentalDeletion $true
}
```

Verify:

```powershell
Get-ADOrganizationalUnit -Filter *
```

---

## User Account Strategy

Use separate accounts for separate purposes.

Suggested account categories:

* Standard domain user
* Domain administration account
* Test user
* Service account
* Disabled test account
* Temporary incident exercise account

Do not use the built-in domain Administrator account for normal daily administration.

---

## Example Account Naming

Public placeholder examples:

```text
student.user
student.admin
test.user01
svc.monitoring
```

Avoid using real personal names, personal email addresses, or production naming conventions in public screenshots.

---

## Create a Standard Domain User

Create a secure password interactively:

```powershell
$password = Read-Host `
    "Enter temporary password" `
    -AsSecureString
```

Create the account:

```powershell
New-ADUser `
    -Name "Student User" `
    -GivenName "Student" `
    -Surname "User" `
    -SamAccountName "student.user" `
    -UserPrincipalName "student.user@cyberlab.example" `
    -Path "OU=Users,OU=CyberLab,DC=cyberlab,DC=example" `
    -AccountPassword $password `
    -Enabled $true `
    -ChangePasswordAtLogon $true
```

Appropriate domain permissions are required.

Do not place passwords directly in PowerShell scripts committed to GitHub.

---

## Create a Dedicated Administrative Account

```powershell
$adminPassword = Read-Host `
    "Enter administrative password" `
    -AsSecureString

New-ADUser `
    -Name "Student Admin" `
    -SamAccountName "student.admin" `
    -UserPrincipalName "student.admin@cyberlab.example" `
    -Path "OU=Administrators,OU=CyberLab,DC=cyberlab,DC=example" `
    -AccountPassword $adminPassword `
    -Enabled $true
```

Add the account to the required administrative group:

```powershell
Add-ADGroupMember `
    -Identity "Domain Admins" `
    -Members "student.admin"
```

Membership in Domain Admins grants extensive privileges and should be used only when necessary.

---

## Least Privilege

Administrative accounts should not be used for:

* General browsing
* Reading email
* Routine workstation activity
* Downloading untrusted tools
* Running normal applications
* Performing user-level security exercises

Use a standard account for normal activity and elevate only for administrative tasks.

---

## Security Group Strategy

Security groups simplify access management.

Example groups:

```text
GG-CyberLab-Users
GG-CyberLab-Admins
GG-SIEM-Analysts
GG-Remote-Management
GG-Test-Accounts
```

The `GG` prefix can indicate a global security group.

A naming convention should be documented and used consistently.

---

## Create a Security Group

```powershell
New-ADGroup `
    -Name "GG-SIEM-Analysts" `
    -SamAccountName "GG-SIEM-Analysts" `
    -GroupCategory Security `
    -GroupScope Global `
    -Path "OU=Security Groups,OU=CyberLab,DC=cyberlab,DC=example"
```

Add a user:

```powershell
Add-ADGroupMember `
    -Identity "GG-SIEM-Analysts" `
    -Members "student.user"
```

---

## Password Policy

The default domain password policy should be reviewed.

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Review:

* Minimum password length
* Password history
* Maximum password age
* Minimum password age
* Complexity requirement
* Reversible encryption
* Account lockout threshold
* Lockout duration
* Lockout observation window

---

## Account Lockout Policy

Account lockout testing is useful for Blue Team exercises, but the policy should be configured deliberately.

An overly strict policy can cause accidental administrative lockouts.

An overly permissive policy may not generate useful test events.

Before changing it:

1. Create a known-good administrator account.
2. Confirm access to the VM console.
3. Create a snapshot.
4. Document the original values.
5. Apply the test policy.
6. Validate the expected events.
7. Restore or revise the policy after testing.

---

## Example Lab Lockout Policy

The following values are examples only:

```text
Lockout threshold: 5 invalid attempts
Lockout duration: 15 minutes
Observation window: 15 minutes
```

Apply with PowerShell:

```powershell
Set-ADDefaultDomainPasswordPolicy `
    -Identity "cyberlab.example" `
    -LockoutThreshold 5 `
    -LockoutDuration "00:15:00" `
    -LockoutObservationWindow "00:15:00"
```

Appropriate domain permissions are required.

---

## Group Policy Management

Group Policy provides centralized configuration for domain systems.

Open Group Policy Management:

```text
gpmc.msc
```

Avoid placing every setting directly into the Default Domain Policy or Default Domain Controllers Policy.

Create separate policies for specific purposes where practical.

---

## Suggested Lab Group Policies

Examples include:

* Windows audit policy
* PowerShell logging
* Account lockout policy
* Windows Defender settings
* Firewall settings
* Screen lock policy
* Event log sizing
* Script block logging
* Process command-line auditing
* Remote management restrictions

Each policy should have:

* A clear name
* A defined scope
* A documented purpose
* A rollback plan
* A validation procedure

---

## Example GPO Naming Standard

```text
GPO-Audit-Policy
GPO-PowerShell-Logging
GPO-Windows-Defender
GPO-Workstation-Baseline
GPO-Event-Log-Sizing
```

Avoid names such as:

```text
New Group Policy Object
Test
Policy 2
Final Policy
```

Clear names improve troubleshooting.

---

## Create a Group Policy Object

```powershell
New-GPO -Name "GPO-Audit-Policy"
```

Link it to the Workstations OU:

```powershell
New-GPLink `
    -Name "GPO-Audit-Policy" `
    -Target "OU=Workstations,OU=CyberLab,DC=cyberlab,DC=example"
```

Appropriate permissions are required.

---

## Audit Policy Goals

The lab should generate sufficient telemetry for monitoring and investigation.

Important categories include:

* Account logon
* Account management
* Logon and logoff
* Policy change
* Privilege use
* Process creation
* System events
* Directory service access
* Object access where appropriate

Avoid enabling every possible audit subcategory without understanding the storage impact.

---

## Advanced Audit Policy

Advanced Audit Policy is normally configured through Group Policy.

Example navigation:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Advanced Audit Policy Configuration
                └── Audit Policies
```

Useful categories for lab exercises include:

* Audit Credential Validation
* Audit User Account Management
* Audit Security Group Management
* Audit Logon
* Audit Account Lockout
* Audit Special Logon
* Audit Process Creation
* Audit PowerShell activity through supporting policies
* Audit Policy Change

---

## Include Command Line in Process Creation Events

To improve process investigation, enable command-line logging for process creation events.

Group Policy path:

```text
Computer Configuration
└── Administrative Templates
    └── System
        └── Audit Process Creation
            └── Include command line in process creation events
```

This can expose sensitive command-line arguments.

Do not enter real passwords or secrets on the command line.

---

## PowerShell Logging

Useful PowerShell logging features include:

* Module logging
* Script block logging
* Transcription
* Process creation auditing

Group Policy path:

```text
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Windows PowerShell
```

PowerShell logs may contain sensitive commands and data.

Treat them as security evidence.

---

## Event Log Sizing

Default event log sizes may be too small for a monitoring lab.

Important logs include:

* Security
* System
* Application
* Windows PowerShell
* PowerShell Operational
* Directory Service
* DNS Server
* Group Policy Operational

Log sizing should balance:

* Retention needs
* Disk space
* Event volume
* SIEM forwarding
* Exercise duration

---

## Review Event Log Configuration

```powershell
wevtutil gl Security
```

PowerShell example:

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName, MaximumSizeInBytes, RecordCount, IsEnabled
```

Administrative privileges may be required for some log configuration changes.

---

## Domain Controller Firewall

Windows Defender Firewall should remain enabled.

The Active Directory role creates required rules for services such as:

* DNS
* Kerberos
* LDAP
* SMB
* RPC
* Active Directory Web Services

Review profiles:

```powershell
Get-NetFirewallProfile
```

Review enabled rules related to Active Directory:

```powershell
Get-NetFirewallRule |
    Where-Object DisplayGroup -Match "Active Directory|DNS|File and Printer Sharing"
```

Do not disable the firewall as a permanent workaround.

---

## Domain Controller Time Synchronization

Accurate time is required for Kerberos and event correlation.

In a single-domain-controller lab, DC01 holds the Primary Domain Controller Emulator role.

Review domain roles:

```powershell
netdom query fsmo
```

Review time status:

```powershell
w32tm /query /status
```

Review configuration:

```powershell
w32tm /query /configuration
```

---

## PDC Emulator Time Source

The PDC Emulator should use a reliable time source when external connectivity is available.

Other domain members normally synchronize through the domain hierarchy.

A disconnected lab may temporarily rely on:

* VMware host time synchronization
* The Windows host clock
* A configured NTP source available through NAT

Avoid conflicting time sources.

---

## Configure an External Time Source

Run from an elevated command prompt or PowerShell:

```powershell
w32tm /config `
    /manualpeerlist:"<APPROVED_NTP_SOURCE>" `
    /syncfromflags:manual `
    /reliable:yes `
    /update
```

Restart the time service:

```powershell
Restart-Service w32time
```

Request synchronization:

```powershell
w32tm /resync
```

Administrator privileges are required.

---

## VMware Time Synchronization Considerations

VMware Tools can synchronize guest time with the host.

For a domain controller, VMware time synchronization and Windows domain time should be reviewed carefully.

Potential problems include:

* Time jumps after snapshot restoration
* Conflict between NTP and VMware Tools
* Kerberos failures
* Misordered events
* SIEM timeline inaccuracies

After reverting a snapshot:

1. Check the server clock.
2. Check the timezone.
3. Verify Windows Time status.
4. Confirm the endpoint clock.
5. Confirm authentication works.
6. Confirm SIEM timestamps are correct.

---

## Domain Controller Health Check

Run:

```powershell
dcdiag
```

A more detailed report:

```powershell
dcdiag /v
```

DNS-focused testing:

```powershell
dcdiag /test:dns /v
```

The detailed output may contain internal hostnames and addresses.

Sanitize it before publication.

---

## Replication Check

A single-domain-controller lab has no partner replication, but the following tools are still useful for understanding domain health:

```powershell
repadmin /replsummary
```

```powershell
repadmin /showrepl
```

The output will be limited in a single-controller environment.

---

## Verify SYSVOL and NETLOGON

```powershell
net share
```

Confirm:

```text
SYSVOL
NETLOGON
```

If these shares are missing after promotion, investigate:

* AD DS health
* DFS Replication
* DNS
* Event logs
* Incomplete promotion

---

## Review Active Directory Event Logs

Important logs include:

```text
Applications and Services Logs
└── Directory Service
└── DNS Server
└── DFS Replication
└── Microsoft
    └── Windows
        └── GroupPolicy
        └── Kerberos
```

Also review:

* Security
* System
* Application

---

## Important Security Event Categories

The domain controller can generate events related to:

* Successful authentication
* Failed authentication
* Kerberos ticket activity
* Account lockouts
* User creation
* User deletion
* Password changes
* Security group membership
* Privileged logons
* Policy changes
* Computer account creation
* Directory service access

Exact event identifiers are documented in the related detection and investigation exercises.

---

## Create a Validation User

Create a non-administrative test account for domain validation.

Example:

```powershell
$password = Read-Host `
    "Enter temporary password" `
    -AsSecureString

New-ADUser `
    -Name "Validation User" `
    -SamAccountName "validation.user" `
    -UserPrincipalName "validation.user@cyberlab.example" `
    -Path "OU=Users,OU=CyberLab,DC=cyberlab,DC=example" `
    -AccountPassword $password `
    -Enabled $true `
    -ChangePasswordAtLogon $true
```

This account can later be used to validate the Windows endpoint domain join and authentication.

---

## Validate Account Creation

```powershell
Get-ADUser `
    -Identity "validation.user" `
    -Properties Enabled, PasswordLastSet, LastLogonDate
```

Confirm the account:

* Exists
* Is enabled
* Is in the correct OU
* Is not an administrator
* Has the expected password-change requirement

---

## Move the Domain Controller Computer Object

After promotion, DC01 normally appears in the built-in Domain Controllers OU.

This is the expected default location:

```text
OU=Domain Controllers,DC=cyberlab,DC=example
```

Do not move the domain controller into a standard Servers OU without understanding the Group Policy implications.

---

## Default Domain Policies

Two important default policies exist:

* Default Domain Policy
* Default Domain Controllers Policy

Recommended practice:

* Keep default policies focused on foundational domain settings.
* Use separate GPOs for additional security settings.
* Avoid unnecessary edits.
* Back up policies before major changes.
* Document every modification.

---

## Back Up Group Policy

Back up all GPOs:

```powershell
Backup-GPO `
    -All `
    -Path "<GPO_BACKUP_PATH>"
```

The destination should exist and be protected.

Do not commit a backup containing sensitive operational values to the public repository without review.

---

## Active Directory Recycle Bin

The Active Directory Recycle Bin can simplify recovery of deleted objects.

Check status:

```powershell
Get-ADOptionalFeature `
    -Filter 'Name -like "Recycle Bin Feature"'
```

Enable it:

```powershell
Enable-ADOptionalFeature `
    -Identity "Recycle Bin Feature" `
    -Scope ForestOrConfigurationSet `
    -Target "cyberlab.example"
```

Appropriate forest permissions are required.

Enabling the Recycle Bin cannot be reversed.

---

## System State Backup

A system state backup is more appropriate for domain controller recovery than relying only on VMware snapshots.

System state includes critical components such as:

* Active Directory
* SYSVOL
* Registry
* Boot files
* System services configuration

Windows Server Backup may need to be installed.

---

## Install Windows Server Backup

```powershell
Install-WindowsFeature Windows-Server-Backup
```

Administrator privileges are required.

Verify:

```powershell
Get-WindowsFeature Windows-Server-Backup
```

---

## System State Backup Example

A system state backup must target a suitable backup location.

Example:

```powershell
wbadmin start systemstatebackup `
    -backuptarget:<BACKUP_DESTINATION> `
    -quiet
```

Administrator privileges are required.

Do not store the only backup on the same virtual disk as the domain controller.

---

## Snapshot Strategy

Recommended domain controller snapshot milestones:

```text
01-DC01-Clean-Install
02-DC01-Patched-Baseline
03-DC01-Static-Network
04-DC01-ADDS-Installed
05-DC01-Domain-Promoted
06-DC01-DNS-Validated
07-DC01-OUs-and-Users
08-DC01-GPO-Baseline
09-DC01-Endpoint-Join-Validated
```

Snapshots should be clearly dated and documented.

---

## Snapshot Safety

Virtual machine snapshots are helpful for a lab but require caution with domain controllers.

Risks include:

* Time rollback
* Authentication issues
* Directory inconsistencies
* Password mismatch with domain members
* Broken secure channels
* Stale DNS records
* Event timeline confusion

Modern Windows Server and VMware versions include safeguards, but snapshots are not a replacement for system state backups.

---

## Safe Snapshot Procedure

For major stable milestones:

1. Confirm domain health.
2. Confirm DNS health.
3. Shut down the server cleanly.
4. Confirm the VM is powered off.
5. Create a named snapshot.
6. Add a description.
7. Restart the server.
8. Run a health check.
9. Validate DNS.
10. Record the snapshot.

---

## Snapshot Restore Considerations

After restoring DC01:

1. Confirm the date and time.
2. Confirm the hostname.
3. Confirm the static address.
4. Confirm the DNS service.
5. Run `dcdiag`.
6. Confirm SYSVOL and NETLOGON.
7. Confirm domain authentication.
8. Confirm the endpoint secure channel.
9. Review event logs.
10. Confirm SIEM telemetry.

A domain member restored from a different point in time may have a broken secure channel.

---

## Domain Controller Startup Sequence

Recommended startup order:

1. Start DC01.
2. Wait for Windows Server to finish booting.
3. Confirm AD DS and DNS services.
4. Confirm the host-only address.
5. Confirm internal DNS.
6. Start the Windows endpoint.
7. Confirm domain authentication.
8. Start monitoring platforms.
9. Confirm telemetry.
10. Begin testing only after validation.

---

## Domain Controller Shutdown Sequence

The domain controller should normally be shut down after dependent domain systems.

Recommended order:

1. Stop active security exercises.
2. Shut down the Windows endpoint.
3. Stop or shut down dependent services.
4. Confirm no domain maintenance is running.
5. Shut down DC01 cleanly.
6. Confirm the VM is powered off.
7. Create a milestone snapshot if required.

---

## Validation Checklist

### Operating System

* Windows Server starts successfully.
* VMware Tools is installed.
* The hostname is correct.
* Time and timezone are correct.
* Updates are current enough for the lab.
* Windows Firewall is enabled.

### Networking

* The host-only adapter is connected.
* The static address is correct.
* The subnet is correct.
* No unintended default gateway exists.
* DNS points to the domain controller.
* No temporary NAT address is registered in internal DNS.

### Active Directory

* The domain exists.
* The forest exists.
* DC01 is listed as a domain controller.
* Required services are running.
* SYSVOL and NETLOGON shares exist.
* `dcdiag` reports no unresolved critical errors.

### DNS

* The internal zone exists.
* Domain controller SRV records resolve.
* DC01 resolves to the host-only address.
* External forwarding works when required.
* No unwanted NAT records exist.

### Administration

* A dedicated administrative account exists.
* A standard validation user exists.
* Organizational units exist.
* Security groups are documented.
* Password and lockout policies are reviewed.
* Group Policy changes are documented.

### Recovery

* A known-good snapshot exists.
* The DSRM password is stored securely.
* The restore order is documented.
* Group Policy backups exist where required.
* A system state backup plan exists.

---

## Troubleshooting: Domain Promotion Fails

Possible causes include:

* Dynamic IP configuration
* Incorrect DNS settings
* Unsupported domain name
* Pending restart
* Missing updates
* Insufficient privileges
* Existing role conflicts
* Incorrect system time
* Network adapter problems

Review:

```powershell
Get-NetIPConfiguration
Get-DnsClientServerAddress
Get-WindowsFeature AD-Domain-Services
Get-PendingReboot
```

`Get-PendingReboot` is not a default Windows PowerShell command unless a supporting module or function has been installed.

Also review Event Viewer and the AD DS deployment logs.

---

## Troubleshooting: DNS Does Not Resolve the Domain

Check:

```powershell
Get-Service DNS
Get-DnsServerZone
Resolve-DnsName "_ldap._tcp.dc._msdcs.cyberlab.example" -Type SRV
ipconfig /all
```

Possible causes include:

* DNS service stopped
* Client querying the wrong server
* Zone missing
* SRV records missing
* NAT adapter registering the wrong address
* Firewall issue
* Incomplete promotion

Restart Netlogon to trigger registration when appropriate:

```powershell
Restart-Service Netlogon
```

Then:

```powershell
ipconfig /registerdns
```

Administrator privileges are required.

---

## Troubleshooting: DC01 Resolves to the NAT Address

Symptoms include:

* Clients connect to the wrong interface
* Domain joins fail intermittently
* Authentication is unreliable
* Ping resolves to an unexpected address

Corrective actions:

1. Disable DNS registration on the NAT adapter.
2. Remove the incorrect DNS record.
3. Confirm the host-only record exists.
4. Restart Netlogon.
5. Register DNS again.
6. Clear client DNS caches.
7. Retest SRV and host records.

Review records:

```powershell
Get-DnsServerResourceRecord `
    -ZoneName "cyberlab.example" `
    -Name "DC01"
```

---

## Troubleshooting: SYSVOL or NETLOGON Missing

Check:

```powershell
Get-Service NTDS, Netlogon, DFSR
```

```powershell
dcdiag /v
```

Review:

```text
Applications and Services Logs
└── DFS Replication
└── Directory Service
```

Do not manually create SYSVOL or NETLOGON shares as a shortcut.

Resolve the underlying directory or replication issue.

---

## Troubleshooting: Kerberos Authentication Fails

Common causes include:

* Time difference
* Incorrect DNS
* Duplicate hostnames
* Duplicate service principal names
* Broken secure channel
* Restored snapshot mismatch

Check:

```powershell
w32tm /query /status
Resolve-DnsName DC01.cyberlab.example
setspn -L DC01
```

On a domain member:

```powershell
Test-ComputerSecureChannel -Verbose
```

---

## Troubleshooting: Cannot Reach DC01 from the Acer Host

Check:

1. The VM is running.
2. The VMware adapter is connected.
3. The VM uses the intended host-only VMnet.
4. The server has the expected address.
5. The host VMware adapter exists.
6. Both systems use the same subnet.
7. Windows Firewall is not blocking the required traffic.
8. A VPN has not altered routing.

Host commands:

```powershell
Get-NetAdapter
Get-NetIPConfiguration
Get-NetRoute
```

Server commands:

```powershell
ipconfig /all
Get-NetConnectionProfile
Get-NetFirewallProfile
```

---

## Troubleshooting: Server Has No Internet During Updates

Check:

* The temporary NAT adapter is connected.
* It received a VMware DHCP address.
* It has the default gateway.
* VMware NAT services are running.
* The host has internet access.
* The host VPN is not blocking VMware traffic.
* DNS forwarding is configured correctly.

```powershell
Get-NetIPConfiguration
Get-NetRoute
Resolve-DnsName example.com
Test-NetConnection example.com -Port 443
```

Do not add a gateway to the host-only adapter simply to restore internet access.

---

## Troubleshooting: Account Lockout During Testing

If a test account becomes locked:

```powershell
Search-ADAccount -LockedOut
```

Unlock the intended account:

```powershell
Unlock-ADAccount -Identity "<TEST_USERNAME>"
```

Appropriate permissions are required.

Do not unlock an account before investigating the source of repeated failures during a detection exercise.

---

## Troubleshooting: Group Policy Does Not Apply

On the endpoint:

```powershell
gpupdate /force
```

```powershell
gpresult /r
```

Generate an HTML report:

```powershell
gpresult /h "<REPORT_PATH>"
```

Check:

* DNS
* Domain connectivity
* OU placement
* GPO link
* Security filtering
* WMI filtering
* Replication
* Event logs

---

## Public Sanitization Standards

Remove or replace:

* Real internal domain name
* Real IP addresses
* VMware subnet values
* Administrator usernames
* User principal names based on real names
* Passwords
* DSRM password
* Product keys
* Activation data
* Windows Server license details
* MAC addresses
* Security identifiers
* Group Policy backup content
* Internal DNS forwarders
* Event logs containing personal names
* File paths containing personal usernames
* VMware UUIDs
* Screenshots containing home-network information

Use placeholders such as:

```text
<DOMAIN_NAME>
<NETBIOS_NAME>
<DOMAIN_CONTROLLER>
<DOMAIN_CONTROLLER_IP>
<ADMIN_USERNAME>
<TEST_USERNAME>
<UPSTREAM_DNS>
<DSRM_PASSWORD>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing screenshots from:

* Server Manager
* Active Directory Users and Computers
* DNS Manager
* Group Policy Management
* PowerShell
* Event Viewer
* VMware Workstation

Review for:

* Domain names
* IP addresses
* Usernames
* Personal names
* Email addresses
* Security identifiers
* File paths
* Product identifiers
* Browser tabs
* Taskbar notifications
* Host network information

Crop screenshots to show only what supports the documented task.

---

## Skills Demonstrated

This deployment demonstrates:

* Windows Server administration
* Active Directory Domain Services
* DNS configuration
* Static IPv4 configuration
* VMware networking
* Domain architecture
* User and group administration
* Organizational unit design
* Group Policy
* Audit policy planning
* Windows event logging
* Time synchronization
* PowerShell administration
* Identity security
* Backup and recovery planning
* Troubleshooting
* Public documentation sanitization

---

## Lessons Learned

Key lessons include:

* Active Directory depends on correct DNS.
* A domain controller should use a stable internal address.
* Domain clients must query the domain controller for DNS.
* A temporary NAT adapter can register the wrong address in DNS.
* Multiple default gateways create unpredictable behavior.
* Time synchronization is essential for Kerberos.
* Group Policy should be separated by purpose.
* Administrative and standard user accounts should be distinct.
* Snapshots are useful but do not replace system state backups.
* The domain controller should start before dependent systems.
* Public identity documentation requires careful sanitization.
* Validation should be performed before joining the endpoint.

---

## Planned Improvements

Future improvements may include:

* Additional domain controller for replication practice
* Certificate Services lab
* Windows Event Forwarding
* Fine-grained password policies
* Privileged Access Workstation
* Local Administrator Password Solution
* Service account hardening
* Managed service accounts
* Restricted administrative tiers
* Group Policy baseline automation
* Active Directory assessment scripts
* Identity attack-path analysis
* Advanced auditing
* Sysmon deployment through Group Policy
* Automated system state backups
* DNS monitoring
* Domain controller health dashboard
* Microsoft Defender for Identity concepts
* Entra ID integration exercises

---

## Summary

The domain controller provides the central identity and DNS services required by the Acer Blue Team CyberLab.

A reliable deployment requires:

* Stable host-only networking
* Correct DNS
* Accurate time
* Controlled administrative access
* Organized directory objects
* Documented Group Policy
* Security event collection
* Snapshots and backups
* Repeatable health validation

By validating the domain controller before adding dependent systems, the CyberLab gains a stable foundation for endpoint monitoring, SIEM integration, detection engineering, and incident investigation.

