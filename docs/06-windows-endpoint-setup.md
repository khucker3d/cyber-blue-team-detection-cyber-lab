# Windows Endpoint Setup

## Overview

The Windows endpoint represents a standard enterprise workstation inside the Acer Blue Team CyberLab.

It is used to practice:

* Windows 11 administration
* Active Directory domain membership
* Domain authentication
* Group Policy application
* Security event generation
* Endpoint monitoring
* Wazuh agent deployment
* Splunk data collection
* PowerShell logging
* File integrity monitoring
* Incident investigation
* Detection validation

The endpoint acts as the primary monitored system in the lab.

All hostnames, domain names, IP addresses, usernames, passwords, product identifiers, and network values shown in this public document are sanitized examples.

---

## Endpoint Role

The Windows endpoint simulates a user workstation in a small enterprise environment.

Its responsibilities include:

* Authenticating users through Active Directory
* Receiving Group Policy
* Generating Windows security events
* Running endpoint monitoring agents
* Producing logs for SIEM analysis
* Supporting controlled security exercises
* Providing a target for authorized testing
* Preserving evidence for investigations
* Validating detection rules and monitoring workflows

The endpoint is not intended to contain personal files, production credentials, or sensitive real-world data.

---

## Public Example Configuration

The public documentation uses the following placeholders:

```text
Hostname: WIN11TARGET
Domain: cyberlab.example
NetBIOS domain: CYBERLAB
Host-only address: 192.0.2.20
Subnet prefix: /24
Preferred DNS server: 192.0.2.10
Default gateway: none for isolated operation
Domain controller: DC01
```

The `192.0.2.0/24` range is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Architecture Placement

The endpoint communicates with the domain controller and monitoring systems through the VMware host-only network.

```text
                         Acer Windows Host
                                 |
                         VMware Workstation
                                 |
                         Host-Only Network
                                 |
          _______________________|_______________________
         |                       |                       |
       DC01                 WIN11TARGET              SIEM Systems
 Active Directory         Windows Endpoint        Wazuh / Splunk
 DNS and Identity         Monitored System        Log Collection
```

A temporary VMware NAT adapter may be enabled when the endpoint requires updates or software installation.

---

## Security Boundary

The endpoint is treated as a controlled monitored system.

It may intentionally generate suspicious-looking activity during authorized exercises, but it should remain separated from:

* Personal files
* Personal passwords
* Production accounts
* Business systems
* Household devices
* Publicly exposed services
* Real customer or employer data

The endpoint should not be bridged to the physical home network during normal lab operation.

---

## Prerequisites

Before creating the Windows endpoint, confirm that the following are available:

* VMware Workstation
* A configured host-only network
* A functioning domain controller
* Internal DNS hosted by the domain controller
* A supported Windows 11 installation image
* Adequate host CPU, memory, and storage
* A planned static or reserved internal address
* A local administrator password
* A domain user account for validation
* A snapshot strategy
* A monitoring deployment plan

The domain controller and DNS should be validated before the endpoint is joined to the domain.

---

## Virtual Machine Creation

Create a dedicated Windows 11 virtual machine.

Suggested public VM name:

```text
WIN11TARGET
```

General virtual hardware considerations:

| Resource           | Guidance                                                    |
| ------------------ | ----------------------------------------------------------- |
| Virtual processors | Start modestly and increase if required                     |
| Memory             | Enough for Windows 11, monitoring agents, and exercises     |
| Storage            | Sufficient for Windows, updates, logs, tools, and snapshots |
| Network adapter    | Host-only                                                   |
| Optical drive      | Windows 11 ISO during installation                          |
| TPM                | Enable if required by Windows 11                            |
| Secure Boot        | Enable where supported and compatible                       |

The exact allocation depends on the Acer host’s available resources.

---

## Virtual Disk Planning

The endpoint disk must support:

* Windows 11
* Operating system updates
* Endpoint agents
* Security tools
* Event logs
* Temporary evidence
* Lab applications
* Snapshot growth

A dynamically growing virtual disk is generally acceptable for a personal lab.

Maintain adequate free space because low storage can cause:

* Update failures
* Logging failures
* Agent instability
* Slow performance
* Snapshot problems
* File integrity monitoring errors

---

## Network Adapter Configuration

The endpoint should initially use one host-only adapter.

Recommended VMware settings:

```text
Network Adapter 1
Type: Custom
VMnet: designated host-only network
Connected: enabled
Connect at power on: enabled
```

Do not use bridged networking for the normal endpoint configuration.

A temporary NAT adapter may be added later.

---

## Windows 11 Installation

1. Create the virtual machine in VMware Workstation.
2. Attach the Windows 11 installation ISO.
3. Start the VM.
4. Select the required language and keyboard layout.
5. Choose the intended Windows 11 edition.
6. Accept the license terms.
7. Select a custom installation.
8. Select the virtual disk.
9. Complete the operating system installation.
10. Create a temporary local account if required.
11. Set a strong local administrator password.
12. Complete the initial Windows setup.
13. Install VMware Tools.
14. Restart the endpoint.
15. Confirm that the virtual hardware operates correctly.

Do not publish product keys, activation identifiers, or installation-specific credentials.

---

## Local Account Strategy

The endpoint should retain a known local administrator account for recovery and troubleshooting.

Public example:

```text
.\localadmin
```

The `.\` prefix explicitly selects the local computer account rather than the domain.

The actual local account name should be excluded from public documentation when it could identify the operational environment.

---

## Local Administrator Purpose

The local administrator account can be used when:

* Domain authentication is unavailable
* DNS is misconfigured
* The domain controller is offline
* The secure channel is broken
* Group Policy blocks normal access
* The endpoint must be removed from the domain
* Network configuration must be repaired

The local administrator password should be:

* Unique
* Stored securely
* Different from domain passwords
* Excluded from scripts
* Excluded from screenshots
* Tested before the domain join

---

## VMware Tools

Install VMware Tools to improve:

* Network drivers
* Display resizing
* Time synchronization
* Mouse integration
* Graceful shutdown
* Clipboard behavior
* Guest management

After installation:

1. Restart Windows.
2. Confirm the display functions correctly.
3. Confirm the VMware network adapter appears.
4. Confirm VMware can initiate a guest shutdown.
5. Review host-to-guest integration features.
6. Disable unnecessary integration for higher-risk exercises.

---

## Initial Endpoint Configuration

Before joining the domain:

1. Confirm the Windows edition supports domain membership.
2. Set the correct timezone.
3. Confirm the system time.
4. Rename the computer.
5. Configure the host-only address.
6. Configure the domain controller as DNS.
7. Install required updates.
8. Confirm the local administrator account works.
9. Validate communication with DC01.
10. Create a clean pre-domain snapshot.

---

## Windows Edition Requirement

Windows 11 Home does not support traditional Active Directory domain joining.

Use an edition such as:

* Windows 11 Pro
* Windows 11 Enterprise
* Windows 11 Education

Review the edition:

```powershell
Get-ComputerInfo |
    Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
```

Or:

```powershell
winver
```

---

## Rename the Endpoint

Public example hostname:

```text
WIN11TARGET
```

Run PowerShell as Administrator:

```powershell
Rename-Computer `
    -NewName "WIN11TARGET" `
    -Restart
```

Administrator privileges are required.

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
WIN11TARGET
```

---

## Identify the Network Adapter

Review adapters:

```powershell
Get-NetAdapter
```

Review IP configuration:

```powershell
Get-NetIPConfiguration
```

The VMware adapter name may vary.

Record the correct interface alias before applying a static address.

---

## Rename the Adapter

A descriptive name reduces confusion.

Example:

```powershell
Rename-NetAdapter `
    -Name "<CURRENT_ADAPTER_NAME>" `
    -NewName "CyberLab-Internal"
```

Administrator privileges are required.

Verify:

```powershell
Get-NetAdapter
```

---

## Configure a Static Internal Address

Run PowerShell as Administrator.

Example:

```powershell
New-NetIPAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -IPAddress "192.0.2.20" `
    -PrefixLength 24
```

Do not add a default gateway to the isolated host-only adapter unless routing is intentionally required.

---

## Configure Domain DNS

The endpoint must use the domain controller as its DNS server.

```powershell
Set-DnsClientServerAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -ServerAddresses "192.0.2.10"
```

Administrator privileges are required.

Do not configure a public resolver as the primary DNS server on a domain-joined endpoint.

---

## Review Network Configuration

```powershell
ipconfig /all
```

```powershell
Get-NetIPConfiguration `
    -InterfaceAlias "CyberLab-Internal"
```

```powershell
Get-DnsClientServerAddress `
    -InterfaceAlias "CyberLab-Internal"
```

Confirm:

* The endpoint has the correct internal address.
* The subnet is correct.
* No unintended default gateway exists.
* DNS points to the domain controller.
* No additional adapter is overriding DNS.

---

## Remove Incorrect IP Settings

Review existing IPv4 addresses:

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

---

## Validate Basic Connectivity

Test the domain controller by address:

```powershell
ping 192.0.2.10
```

Test the DNS service port:

```powershell
Test-NetConnection 192.0.2.10 -Port 53
```

Test the domain controller by hostname:

```powershell
ping DC01
```

A failed ping may only indicate that ICMP is blocked.

Service-port testing is more meaningful for domain validation.

---

## Validate DNS Resolution

Resolve the domain controller:

```powershell
Resolve-DnsName DC01.cyberlab.example
```

Resolve Active Directory service records:

```powershell
Resolve-DnsName `
    "_ldap._tcp.dc._msdcs.cyberlab.example" `
    -Type SRV
```

Resolve Kerberos service records:

```powershell
Resolve-DnsName `
    "_kerberos._tcp.cyberlab.example" `
    -Type SRV
```

The results should reference DC01 and its host-only address.

---

## Validate Domain Discovery

Use `nltest`:

```powershell
nltest /dsgetdc:cyberlab.example
```

Expected information includes:

* Domain controller hostname
* Domain name
* Site name
* Address
* Directory service capabilities

The output should not reference a NAT address.

---

## Check Time Synchronization

Review the endpoint clock:

```powershell
Get-Date
```

Review Windows Time status:

```powershell
w32tm /query /status
```

Before domain joining, the endpoint may synchronize from:

* VMware Tools
* The Windows host
* A temporary internet time source

After the domain join, it should normally synchronize through the domain hierarchy.

---

## Windows Updates

The endpoint should be updated before or shortly after domain joining.

A temporary NAT adapter may be used.

Safe workflow:

1. Shut down the VM.
2. Add a VMware NAT adapter.
3. Start the VM.
4. Confirm the host-only settings remain intact.
5. Confirm the NAT adapter receives an address.
6. Install Windows updates.
7. Restart as required.
8. Confirm internal DNS still points to DC01.
9. Disconnect or remove the NAT adapter.
10. Revalidate domain connectivity.

---

## Temporary NAT Adapter

The temporary NAT adapter should use VMware DHCP.

Recommended design:

```text
Host-only adapter:
- Static internal address
- DNS set to DC01
- No default gateway

NAT adapter:
- DHCP address
- VMware default gateway
- Temporary outbound access
```

Avoid configuring two default gateways.

---

## Prevent NAT DNS Registration

A temporary NAT adapter should not register its address in internal DNS.

PowerShell:

```powershell
Set-DnsClient `
    -InterfaceAlias "<NAT_ADAPTER>" `
    -RegisterThisConnectionsAddress $false
```

Administrator privileges are required.

Also review the adapter’s DNS settings so it does not become the preferred resolver for domain operations.

---

## Disable the NAT Adapter When Finished

```powershell
Disable-NetAdapter `
    -Name "<NAT_ADAPTER>" `
    -Confirm:$false
```

Re-enable when required:

```powershell
Enable-NetAdapter `
    -Name "<NAT_ADAPTER>" `
    -Confirm:$false
```

Administrator privileges are required.

The adapter can also be disconnected through VMware settings.

---

## Create the Pre-Domain Snapshot

Before joining the domain:

1. Confirm Windows starts correctly.
2. Confirm the local administrator login works.
3. Confirm the hostname is correct.
4. Confirm the static address.
5. Confirm DNS points to DC01.
6. Confirm the domain can be discovered.
7. Shut down Windows cleanly.
8. Create a powered-off snapshot.
9. Name the snapshot clearly.
10. Restart the endpoint.

Example:

```text
01-WIN11TARGET-Pre-Domain-Join
```

---

## Domain Join Prerequisites

Before joining the domain, confirm:

* DC01 is running.
* Active Directory services are healthy.
* DNS is running.
* The endpoint resolves the domain.
* The endpoint discovers DC01.
* The endpoint time is close to DC01.
* A domain account with join permissions is available.
* The local administrator password is known.
* A pre-domain snapshot exists.

---

## Join the Domain Through PowerShell

Run PowerShell as Administrator:

```powershell
Add-Computer `
    -DomainName "cyberlab.example" `
    -Credential "CYBERLAB\<DOMAIN_JOIN_ACCOUNT>" `
    -Restart
```

A credential prompt appears.

Administrator privileges are required on the endpoint, and the supplied domain account must be allowed to join computers.

---

## Join the Domain Through Settings

1. Open **Settings**.
2. Select **System**.
3. Select **About**.
4. Open the advanced rename or domain settings.
5. Select **Domain or workgroup**.
6. Choose **Domain**.
7. Enter the internal domain name.
8. Provide authorized domain credentials.
9. Confirm the welcome message.
10. Restart the endpoint.

The exact interface may vary by Windows build.

---

## First Domain Sign-In

After restart, sign in with a domain user.

Public example:

```text
CYBERLAB\validation.user
```

Or:

```text
validation.user@cyberlab.example
```

To sign in with the local recovery account:

```text
.\localadmin
```

Confirm the correct account context before entering credentials.

---

## Verify Domain Membership

```powershell
Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsDomainRole
```

Or:

```powershell
systeminfo |
    findstr /B /C:"Domain"
```

Expected public example:

```text
Domain: cyberlab.example
```

---

## Verify the Logged-In Identity

```powershell
whoami
```

Expected public example:

```text
cyberlab\validation.user
```

Review group memberships:

```powershell
whoami /groups
```

The output may contain security identifiers and should be sanitized before publication.

---

## Verify the Computer Account

On DC01:

```powershell
Get-ADComputer `
    -Identity "WIN11TARGET" `
    -Properties Enabled, DistinguishedName, LastLogonDate
```

Confirm:

* The computer account exists.
* It is enabled.
* The hostname is correct.
* It is located in the expected container or OU.

---

## Move the Endpoint into the Workstations OU

Newly joined computers normally appear in the default Computers container.

Move the endpoint into the dedicated Workstations OU.

On DC01:

```powershell
Get-ADComputer "WIN11TARGET" |
    Move-ADObject `
        -TargetPath "OU=Workstations,OU=CyberLab,DC=cyberlab,DC=example"
```

Appropriate domain permissions are required.

Verify:

```powershell
Get-ADComputer `
    -Identity "WIN11TARGET" `
    -Properties DistinguishedName
```

---

## Apply Group Policy

On the endpoint:

```powershell
gpupdate /force
```

Review applied policies:

```powershell
gpresult /r
```

Generate an HTML report:

```powershell
gpresult /h "C:\Temp\gpresult.html"
```

Create the destination directory first if required.

The report may contain domain names, usernames, groups, and internal paths.

Sanitize it before publication.

---

## Validate Group Policy

Confirm that intended policies apply, such as:

* Audit policy
* PowerShell logging
* Process creation logging
* Windows Defender settings
* Firewall rules
* Event log sizing
* Screen lock settings
* Security baseline settings

Review Group Policy operational logs:

```text
Event Viewer
└── Applications and Services Logs
    └── Microsoft
        └── Windows
            └── GroupPolicy
                └── Operational
```

---

## Verify Domain Secure Channel

```powershell
Test-ComputerSecureChannel -Verbose
```

Expected result:

```text
True
```

A false result may indicate:

* Snapshot rollback mismatch
* Computer account password mismatch
* DNS failure
* Domain controller unavailability
* Broken domain trust

---

## Repair a Secure Channel

When appropriate:

```powershell
Test-ComputerSecureChannel `
    -Repair `
    -Credential "CYBERLAB\<AUTHORIZED_ADMIN>"
```

Administrator privileges and suitable domain permissions are required.

Do not repair the channel before documenting the symptoms during an investigation exercise.

---

## Windows Firewall

Windows Defender Firewall should remain enabled.

Review profiles:

```powershell
Get-NetFirewallProfile
```

Review the current network profile:

```powershell
Get-NetConnectionProfile
```

After domain join, the host-only network should normally use the DomainAuthenticated profile when DC01 is available.

---

## Domain Network Profile Validation

Expected example:

```text
NetworkCategory: DomainAuthenticated
```

If the adapter remains Public:

* Confirm DNS points to DC01.
* Confirm the domain controller is reachable.
* Confirm Network Location Awareness is running.
* Confirm the secure channel.
* Restart the endpoint if required.
* Review related event logs.

Do not manually force the DomainAuthenticated profile.

Windows determines it through domain detection.

---

## Windows Defender

Microsoft Defender Antivirus should remain enabled unless a specific controlled exercise requires otherwise.

Review status:

```powershell
Get-MpComputerStatus
```

Important fields include:

* Antivirus enabled
* Real-time protection enabled
* Behavior monitoring enabled
* Signature age
* Last scan time
* Tamper protection status where reported

The command may require elevation for complete information.

---

## Update Defender Signatures

Run PowerShell as Administrator:

```powershell
Update-MpSignature
```

Internet access may be required.

After updating, review:

```powershell
Get-MpComputerStatus |
    Select-Object AntivirusSignatureLastUpdated, AntivirusSignatureVersion
```

---

## Security Exclusions

Do not create broad Defender exclusions for:

* The entire endpoint disk
* The user profile
* Downloads
* PowerShell
* Temporary folders
* Monitoring directories without justification

An exclusion reduces security visibility.

When an exclusion is required for a controlled test:

1. Document the reason.
2. Limit the path or process.
3. Record the original configuration.
4. Perform the test.
5. Remove the exclusion.
6. Validate Defender protection.
7. Record the result.

---

## Local Administrators Group

Review local administrators:

```powershell
Get-LocalGroupMember `
    -Group "Administrators"
```

Expected entries may include:

* Local recovery administrator
* Domain administrative group
* Approved lab administration account

Remove unnecessary members.

The output may reveal real usernames and should be sanitized.

---

## Add a Domain Group to Local Administrators

Example:

```powershell
Add-LocalGroupMember `
    -Group "Administrators" `
    -Member "CYBERLAB\GG-CyberLab-Admins"
```

Administrator privileges are required.

Use a security group rather than adding multiple users individually where practical.

---

## Standard User Testing

Most endpoint activity should be performed through a standard domain user.

A standard account is appropriate for:

* Routine sign-ins
* Browser activity
* File operations
* Failed logon tests
* PowerShell exercises that do not require elevation
* Application execution
* File integrity monitoring
* Security event generation

Administrative credentials should be used only when the task requires them.

---

## User Profile Creation

The first domain sign-in creates a local Windows profile.

The profile may contain:

* Desktop files
* User registry settings
* Application data
* Browser data
* PowerShell history
* Temporary evidence
* Exercise artifacts

Do not place personal information in the domain profile.

Use neutral lab content.

---

## Suggested Test Directory

Create a dedicated test directory:

```powershell
New-Item `
    -ItemType Directory `
    -Path "C:\CyberLab-Test" `
    -Force
```

Possible subdirectories:

```text
C:\CyberLab-Test\
├── Files\
├── Scripts\
├── Logs\
├── Evidence\
└── Temporary\
```

Administrator privileges are required when writing directly under `C:\` depending on the context.

A user-owned path may be used when elevation is unnecessary.

---

## Test Data Safety

Use only synthetic or harmless files.

Suitable content includes:

* Sample text files
* Dummy CSV files
* Generated logs
* Benign PowerShell scripts
* Test configuration files
* Public datasets
* Non-sensitive documents

Do not use:

* Personal documents
* Tax or financial records
* Health records
* Real credentials
* Employer data
* Customer information
* Private email
* Real security incidents

---

## Windows Event Logging

The endpoint provides logs for:

* Authentication
* Account activity
* Process creation
* PowerShell
* System changes
* Service activity
* Defender
* Firewall
* Group Policy
* Application events

Primary logs include:

```text
Windows Logs
├── Application
├── Security
├── Setup
├── System
└── Forwarded Events
```

---

## Review the Security Log

PowerShell:

```powershell
Get-WinEvent `
    -LogName Security `
    -MaxEvents 20
```

Administrator privileges may be required to read the Security log.

Filter recent events:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        StartTime = (Get-Date).AddMinutes(-30)
    }
```

---

## Audit Policy Validation

Review effective audit policy:

```powershell
auditpol /get /category:*
```

The output shows which categories and subcategories are configured for success and failure auditing.

Important areas include:

* Logon and logoff
* Account logon
* Account management
* Process creation
* Policy change
* Privilege use
* System events

---

## Process Creation Logging

Process creation events are commonly used for detection engineering.

Validate the audit subcategory:

```powershell
auditpol /get /subcategory:"Process Creation"
```

Validate command-line inclusion through policy or by inspecting generated events.

A common process creation event is recorded in the Security log when the required audit settings are enabled.

---

## PowerShell Logging

Important PowerShell logs include:

```text
Applications and Services Logs
└── Microsoft
    └── Windows
        └── PowerShell
            └── Operational
```

Legacy Windows PowerShell events may also appear under:

```text
Windows PowerShell
```

Review recent PowerShell operational events:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-PowerShell/Operational" `
    -MaxEvents 20
```

---

## Script Block Logging Validation

Execute a harmless command:

```powershell
Get-Process |
    Select-Object -First 5
```

Then review the PowerShell Operational log.

The presence and detail of script block events depends on the applied Group Policy configuration.

PowerShell logs may contain full command content and should be treated as sensitive evidence.

---

## PowerShell History

PowerShell history can contain sensitive commands.

Review the history path:

```powershell
(Get-PSReadLineOption).HistorySavePath
```

Do not enter passwords, tokens, or API keys directly into command lines.

Before taking screenshots, review:

```powershell
Get-History
```

This does not necessarily display all history stored by PSReadLine.

---

## Windows Defender Event Log

Defender events are available under:

```text
Applications and Services Logs
└── Microsoft
    └── Windows
        └── Windows Defender
            └── Operational
```

Review recent events:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Windows Defender/Operational" `
    -MaxEvents 20
```

---

## Windows Firewall Event Logging

Firewall logging can help investigate blocked or allowed connections.

The log is commonly stored at:

```text
%SystemRoot%\System32\LogFiles\Firewall\pfirewall.log
```

Review profile logging settings:

```powershell
Get-NetFirewallProfile |
    Select-Object Name, LogAllowed, LogBlocked, LogFileName
```

Administrator privileges are required to change these settings.

---

## Enable Blocked-Connection Logging

Example:

```powershell
Set-NetFirewallProfile `
    -Profile Domain,Private,Public `
    -LogBlocked True
```

Administrator privileges are required.

Logging every allowed connection can generate significant volume and should be enabled only when needed.

---

## Event Log Sizing

Monitoring exercises can generate more events than default log sizes retain.

Review the Security log:

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName, MaximumSizeInBytes, RecordCount
```

Review PowerShell:

```powershell
Get-WinEvent `
    -ListLog "Microsoft-Windows-PowerShell/Operational" |
    Select-Object LogName, MaximumSizeInBytes, RecordCount
```

Changes should be applied through Group Policy where practical.

---

## Wazuh Agent Role

The Wazuh agent can collect and report:

* Windows event logs
* Security events
* File integrity changes
* System inventory
* Security configuration information
* Vulnerability-related data
* Agent health
* Selected custom logs

The exact installation process is documented in the Wazuh setup section of the repository.

---

## Wazuh Agent Prerequisites

Before agent installation:

* The Wazuh server is running.
* The endpoint can reach the Wazuh internal address.
* Required ports are allowed.
* The endpoint hostname is correct.
* Time is synchronized.
* DNS works if hostnames are used.
* Enrollment credentials are protected.
* A snapshot exists.

Do not publish enrollment tokens, registration keys, or agent secrets.

---

## Splunk Data Collection Role

Splunk may collect endpoint data through:

* Splunk Universal Forwarder
* Imported event logs
* Scripted inputs
* Test datasets
* Windows Event Log inputs
* Sysmon events
* PowerShell logs

The collection method should be documented separately from the endpoint baseline.

---

## Sysmon Planning

Sysmon can extend endpoint telemetry with:

* Process creation
* Network connections
* File creation
* Registry changes
* Driver loading
* Image loading
* DNS queries
* Named pipe activity

Sysmon should be deployed with a deliberate configuration.

Installing Sysmon with an overly broad configuration can create high event volume.

The endpoint baseline should be stable before Sysmon is added.

---

## Snapshot Strategy

Recommended endpoint milestones:

```text
01-WIN11TARGET-Clean-Install
02-WIN11TARGET-Patched-Baseline
03-WIN11TARGET-Static-Network
04-WIN11TARGET-Pre-Domain-Join
05-WIN11TARGET-Domain-Joined
06-WIN11TARGET-GPO-Validated
07-WIN11TARGET-Wazuh-Agent
08-WIN11TARGET-Splunk-Forwarder
09-WIN11TARGET-Sysmon-Baseline
10-WIN11TARGET-Exercise-Ready
```

Snapshot names should include dates in private operational records.

---

## Safe Snapshot Procedure

1. End active exercises.
2. Save required evidence.
3. Confirm no update is in progress.
4. Run a basic health check.
5. Shut down Windows cleanly.
6. Confirm the VM is powered off.
7. Create the snapshot.
8. Add a descriptive note.
9. Restart the endpoint.
10. Validate domain and monitoring connectivity.

---

## Snapshot Restore Risks

Restoring a domain-joined endpoint can cause:

* Broken secure channel
* Computer account password mismatch
* Time rollback
* Stale Group Policy
* Duplicate or missing SIEM events
* Agent registration inconsistencies
* Conflicting monitoring state

After restoration:

1. Confirm time.
2. Confirm DNS.
3. Confirm domain connectivity.
4. Test the secure channel.
5. Run Group Policy update.
6. Confirm Wazuh agent status.
7. Confirm Splunk forwarding.
8. Review event timelines.
9. Document the restoration.
10. Repair or rejoin the domain only when required.

---

## Endpoint Startup Sequence

Recommended sequence:

1. Start DC01.
2. Confirm DNS and Active Directory.
3. Start WIN11TARGET.
4. Confirm the host-only address.
5. Confirm domain resolution.
6. Sign in with a standard domain account.
7. Confirm Group Policy.
8. Confirm monitoring agents.
9. Confirm SIEM telemetry.
10. Begin the exercise.

---

## Endpoint Shutdown Sequence

1. Stop active test activity.
2. Save evidence.
3. Stop packet captures.
4. Close monitoring tools.
5. Confirm logs have been forwarded where required.
6. Sign out of test accounts.
7. Shut down Windows cleanly.
8. Confirm the VM is powered off.
9. Create a snapshot when appropriate.
10. Shut down DC01 after dependent systems.

---

## Baseline Validation Checklist

### Operating System

* Windows 11 starts normally.
* The installed edition supports domain joining.
* VMware Tools is installed.
* The hostname is correct.
* Time and timezone are correct.
* Windows updates are in an acceptable state.

### Networking

* The host-only adapter is connected.
* The internal address is correct.
* DNS points to DC01.
* No unintended default gateway exists.
* The NAT adapter is disabled when not required.
* The endpoint resolves internal names.

### Domain

* The endpoint is joined to the domain.
* The computer account exists.
* The computer is in the Workstations OU.
* Domain users can sign in.
* The secure channel is healthy.
* Group Policy applies.

### Security

* Windows Firewall is enabled.
* Microsoft Defender is enabled.
* Local administrator membership is reviewed.
* Standard user accounts are used for routine activity.
* Audit policy is active.
* PowerShell logging is validated.

### Monitoring

* Wazuh agent is active when installed.
* Splunk forwarding works when configured.
* Endpoint events appear in the monitoring platform.
* System time aligns with SIEM timestamps.
* Test events can be found and investigated.

### Recovery

* The local administrator account works.
* A pre-domain snapshot exists.
* A post-domain snapshot exists.
* Restore procedures are documented.
* Important evidence is stored outside disposable snapshots.

---

## Controlled Validation Exercises

The endpoint should generate harmless, repeatable activity.

Examples include:

* Successful domain sign-in
* Failed domain sign-in
* Account lockout
* Local administrator use
* Group membership change
* Temporary user creation
* Harmless PowerShell execution
* Service creation in a controlled exercise
* File creation and modification
* Defender test event
* Firewall block
* Controlled network scan

Each exercise should document:

* Objective
* Preconditions
* Actions
* Expected telemetry
* Validation query
* Cleanup
* Lessons learned

---

## Successful Logon Validation

1. Sign out of the endpoint.
2. Sign in with a standard domain user.
3. Confirm the account with `whoami`.
4. Record the approximate time.
5. Review the Security log.
6. Confirm the event reaches Wazuh or Splunk.
7. Record the event fields.
8. Sign out when finished.

Do not publish real account names.

---

## Failed Logon Validation

1. Use a designated test account.
2. Enter an incorrect password once.
3. Record the time.
4. Sign in successfully afterward if appropriate.
5. Review the endpoint Security log.
6. Review domain controller authentication events.
7. Search the SIEM.
8. Confirm the source workstation and account.
9. Document the failure reason.
10. Avoid locking out an administrative account.

---

## Account Lockout Validation

Use only a dedicated test account.

1. Confirm the lockout policy.
2. Confirm an administrator account remains available.
3. Record the test account state.
4. Perform the planned invalid attempts.
5. Confirm the account is locked.
6. Review endpoint and domain controller events.
7. Search Wazuh or Splunk.
8. Identify the source system.
9. Unlock the account after investigation.
10. Document cleanup.

---

## Harmless PowerShell Validation

Run a benign command:

```powershell
Get-Service |
    Sort-Object Status |
    Select-Object -First 10
```

Record:

* User
* Time
* Parent process
* Command content
* Security process-creation event
* PowerShell operational event
* SIEM detection result

Do not use encoded, obfuscated, or credential-access commands unless a later authorized exercise specifically requires them.

---

## File Integrity Validation

Create a test file:

```powershell
New-Item `
    -Path "C:\CyberLab-Test\Files\sample.txt" `
    -ItemType File `
    -Force
```

Add content:

```powershell
Set-Content `
    -Path "C:\CyberLab-Test\Files\sample.txt" `
    -Value "CyberLab file integrity validation."
```

Modify it:

```powershell
Add-Content `
    -Path "C:\CyberLab-Test\Files\sample.txt" `
    -Value "Authorized modification."
```

Delete it after the monitoring result is confirmed:

```powershell
Remove-Item `
    -Path "C:\CyberLab-Test\Files\sample.txt"
```

Use only the designated test directory.

---

## Microsoft Defender Test Validation

Microsoft publishes a harmless antivirus test string commonly used to validate detection.

When using any antivirus test artifact:

* Obtain it from an official security source.
* Use it only inside the lab.
* Do not email it.
* Do not upload it to GitHub.
* Confirm Defender is active.
* Record the alert.
* Remove the artifact.
* Confirm the threat is cleared.

The exact test string should not be embedded unnecessarily in general project documentation.

---

## Endpoint Evidence Collection

Useful evidence includes:

* Exported event logs
* Sanitized screenshots
* PowerShell output
* Wazuh alerts
* Splunk search results
* Group Policy reports
* Network captures
* Investigation notes
* Timeline records
* Hash values

Evidence should be copied to a controlled location before reverting a snapshot.

---

## Export a Windows Event Log

Run PowerShell or Command Prompt as Administrator:

```powershell
wevtutil epl Security "<EVIDENCE_PATH>\Security.evtx"
```

PowerShell event export may also be performed through Event Viewer.

The exported log may contain:

* Usernames
* Domain names
* IP addresses
* Security identifiers
* Computer names
* Process command lines

Do not publish the raw file without sanitization and review.

---

## Calculate an Evidence Hash

```powershell
Get-FileHash `
    -Path "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

Record:

```text
File:
Collection date:
Source system:
SHA-256:
Exercise:
Analyst:
```

Use a neutral public analyst label such as `Student Analyst` when publishing examples.

---

## Troubleshooting: Domain Join Fails

Possible causes include:

* Endpoint DNS points to the wrong server
* Domain controller is offline
* Domain name is misspelled
* Time difference is too large
* Required ports are blocked
* Windows edition does not support joining
* Computer name conflict
* Credentials lack permission
* A pending restart exists

Check:

```powershell
ipconfig /all
Resolve-DnsName cyberlab.example
Resolve-DnsName "_ldap._tcp.dc._msdcs.cyberlab.example" -Type SRV
nltest /dsgetdc:cyberlab.example
w32tm /query /status
```

---

## Troubleshooting: Domain Name Cannot Be Resolved

Check the configured DNS servers:

```powershell
Get-DnsClientServerAddress
```

The host-only adapter should point to DC01.

Clear the cache:

```powershell
ipconfig /flushdns
```

Retest:

```powershell
Resolve-DnsName cyberlab.example
```

Do not solve the issue by assigning a public DNS resolver to the endpoint.

---

## Troubleshooting: Cannot Sign In with a Domain User

Check:

* DC01 is running.
* DNS resolves the domain.
* The user account is enabled.
* The password is correct.
* The account is not locked.
* The endpoint time is correct.
* The secure channel is healthy.
* The user is entering the correct domain context.

Use the local recovery account if domain authentication is unavailable:

```text
.\localadmin
```

---

## Troubleshooting: Network Shows Public Instead of Domain

Review:

```powershell
Get-NetConnectionProfile
```

Check:

```powershell
Resolve-DnsName DC01.cyberlab.example
Test-ComputerSecureChannel -Verbose
Get-Service NlaSvc, Netlogon
```

Possible causes include:

* DNS failure
* DC01 unavailable
* Secure channel problem
* Network Location Awareness timing
* Firewall issue

Restarting the endpoint after DC01 is fully available may resolve startup timing issues.

---

## Troubleshooting: Group Policy Does Not Apply

Run:

```powershell
gpupdate /force
```

Review:

```powershell
gpresult /r
```

Check:

* Endpoint OU placement
* GPO link
* Security filtering
* DNS
* Domain connectivity
* Network profile
* Event logs
* WMI filters
* User versus computer scope

Review the Group Policy Operational log.

---

## Troubleshooting: Secure Channel Is Broken

Test:

```powershell
Test-ComputerSecureChannel -Verbose
```

Potential causes include:

* Endpoint snapshot restoration
* Domain controller snapshot restoration
* Computer account password mismatch
* Duplicate computer account
* DNS failure
* Long offline period

Repair when appropriate:

```powershell
Test-ComputerSecureChannel `
    -Repair `
    -Credential "CYBERLAB\<AUTHORIZED_ADMIN>"
```

If repair fails, a controlled domain leave and rejoin may be required.

---

## Troubleshooting: Endpoint Has No Internet Access

Check:

* The NAT adapter is connected.
* The NAT adapter received an address.
* The NAT adapter has a default gateway.
* VMware NAT services are running.
* The Acer host has internet access.
* A host VPN is not blocking VMware traffic.
* DNS works for external names.

```powershell
Get-NetIPConfiguration
Get-NetRoute
Test-NetConnection 1.1.1.1 -Port 443
Resolve-DnsName example.com
```

Do not add a default gateway to the host-only adapter.

---

## Troubleshooting: Internal DNS Breaks When NAT Is Enabled

Possible causes include:

* NAT DNS assigned a lower metric
* Multiple DNS server lists
* NAT adapter registered in internal DNS
* Adapter order changed
* External DNS queried instead of DC01

Review:

```powershell
Get-DnsClientServerAddress
Get-NetIPInterface |
    Sort-Object InterfaceMetric
```

Correct the adapter DNS configuration and disable DNS registration on the NAT adapter.

---

## Troubleshooting: Wazuh Agent Does Not Connect

Check:

* Wazuh server is running.
* The endpoint can reach the server.
* The agent service is running.
* Enrollment completed successfully.
* The configured manager address is correct.
* Required firewall rules exist.
* Time is synchronized.
* The agent identity is not duplicated.

Review services:

```powershell
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Test the required server port:

```powershell
Test-NetConnection <WAZUH_SERVER> -Port <AGENT_PORT>
```

Do not publish agent registration credentials.

---

## Troubleshooting: Splunk Forwarder Does Not Send Data

Check:

* The forwarder service is running.
* The Splunk server is reachable.
* The receiver is enabled.
* The target address and port are correct.
* The Windows input is configured.
* The selected event logs exist.
* Firewall rules permit the connection.
* The endpoint time is correct.

Review services:

```powershell
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Review the forwarder logs privately before publishing excerpts.

---

## Troubleshooting: Security Events Are Missing

Check:

* Audit policy
* Group Policy application
* Event log status
* Log size and retention
* User privileges
* The exact action performed
* Monitoring-agent configuration
* SIEM ingestion delay
* Timestamp alignment

```powershell
auditpol /get /category:*
Get-WinEvent -ListLog Security
gpresult /r
```

Generate a simple known test event and trace it from the endpoint to the SIEM.

---

## Troubleshooting: Endpoint Performance Is Poor

Possible causes include:

* Insufficient VM memory
* Too many virtual CPUs
* Windows updates
* Defender scanning
* Wazuh activity
* Splunk forwarder activity
* Sysmon event volume
* Long snapshot chains
* Host storage pressure
* Multiple active VMs

Review:

```powershell
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10
```

```powershell
Get-Counter '\Memory\Available MBytes'
```

Use Task Manager and host performance monitoring to compare guest and host resource pressure.

---

## Public Sanitization Standards

Remove or replace:

* Real hostname
* Real domain name
* Real IP addresses
* Personal usernames
* Personal names
* Email addresses
* Passwords
* Product keys
* Activation identifiers
* MAC addresses
* Security identifiers
* Enrollment tokens
* Agent keys
* Internal server addresses
* Home-network information
* Browser profile data
* PowerShell history containing sensitive commands
* Personal file paths
* VMware UUIDs
* Notification content

Use placeholders such as:

```text
<WINDOWS_ENDPOINT>
<DOMAIN_NAME>
<DOMAIN_CONTROLLER>
<DOMAIN_CONTROLLER_IP>
<LOCAL_ADMIN>
<DOMAIN_USER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<AGENT_PORT>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing endpoint screenshots:

1. Close unrelated applications.
2. Hide personal browser tabs.
3. Hide bookmarks.
4. Clear notifications.
5. Sign in with a neutral test account.
6. Remove personal files from the desktop.
7. Review the taskbar.
8. Remove internal addresses.
9. Remove usernames and domain names.
10. Remove security identifiers.
11. Remove agent enrollment data.
12. Verify that redactions are permanent.

Suitable screenshots may include:

* Sanitized domain membership
* Group Policy results
* Event Viewer entries
* Wazuh agent status
* Splunk forwarder status
* Defender alerts
* Harmless test activity
* Sanitized PowerShell output

---

## Skills Demonstrated

This endpoint deployment demonstrates:

* Windows 11 administration
* VMware virtual machine deployment
* Static IPv4 configuration
* Active Directory domain joining
* DNS troubleshooting
* Group Policy validation
* Account separation
* Windows Defender administration
* Windows Firewall review
* Windows event logging
* PowerShell logging
* Audit policy validation
* Endpoint monitoring
* SIEM integration
* Snapshot management
* Evidence handling
* Incident investigation
* Technical documentation
* Public sanitization

---

## Lessons Learned

Key lessons include:

* Domain joining depends on correct DNS.
* The domain controller should be available before the endpoint starts.
* A local recovery administrator should be tested before the join.
* Temporary NAT adapters can interfere with domain DNS.
* Group Policy application depends on correct OU placement.
* The DomainAuthenticated network profile cannot be safely forced.
* Snapshot restoration can break the secure channel.
* Audit settings must be validated before relying on SIEM data.
* Monitoring agents should be added only after the endpoint baseline is stable.
* Standard users should generate most test activity.
* Evidence should be preserved before snapshot rollback.
* Public screenshots require a separate sanitization review.

---

## Planned Improvements

Future endpoint improvements may include:

* Sysmon deployment
* Windows Event Forwarding
* Microsoft LAPS
* Application control
* PowerShell constrained language testing
* Attack Surface Reduction rules
* Advanced Defender configuration
* Local firewall policy through Group Policy
* Automated endpoint validation
* Scheduled security health checks
* Vulnerability scanning
* Browser security exercises
* Credential Guard testing
* Device Control
* BitLocker lab configuration
* Software inventory
* Centralized configuration management
* Additional Windows endpoints
* Simulated departmental workstations
* Detection-as-code validation

---

## Summary

The Windows endpoint is the primary monitored workstation in the Acer Blue Team CyberLab.

A reliable endpoint deployment requires:

* Correct host-only networking
* Domain-controller DNS
* Accurate time
* A known local recovery account
* Successful domain membership
* Group Policy validation
* Windows security controls
* Endpoint telemetry
* Snapshot and recovery planning
* Safe, synthetic test data

By establishing a stable endpoint baseline before adding advanced monitoring, the CyberLab gains a dependable platform for Wazuh, Splunk, detection engineering, incident investigation, and controlled defensive security exercises.

