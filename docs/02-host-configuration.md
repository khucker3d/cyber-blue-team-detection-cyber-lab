# Host Configuration

## Overview

The Acer Windows 11 workstation serves as the physical host for the Blue Team CyberLab.

The host provides:

* Virtual machine compute resources
* Virtual storage
* VMware virtual networking
* Snapshot management
* Administrative access
* Controlled internet connectivity
* Local backup and recovery support

Because every CyberLab virtual machine depends on the host, the physical workstation is a critical part of the lab’s security and availability architecture.

This document covers the public, sanitized host configuration used to support the virtualized environment.

Specific hardware identifiers, user account names, storage paths, license information, and internal network values have been removed or replaced with placeholders.

---

## Host Role

The Acer system is dedicated primarily to Blue Team learning and CyberLab administration.

Its responsibilities include:

* Running VMware Workstation
* Hosting Windows and Linux virtual machines
* Managing host-only and NAT virtual networks
* Storing ISO images and installation packages
* Managing virtual machine snapshots
* Providing browser access to security dashboards
* Supporting PowerShell-based administration
* Maintaining lab documentation and evidence
* Transferring installation files into isolated virtual machines
* Preserving recoverable lab states

The host should not be treated as an untrusted testing target.

Controlled security exercises should remain inside the virtual lab unless a separate host-testing exercise is specifically planned and documented.

---

## Host Operating System

The virtualization host runs Windows 11.

Windows 11 provides:

* VMware Workstation support
* PowerShell administration
* Windows Defender security controls
* BitLocker support on compatible systems
* Local firewall management
* Event logging
* Device management
* Browser-based access to SIEM interfaces
* File sharing between the host and lab systems when explicitly configured

The host operating system should remain fully patched and supported.

---

## Host Hardware Considerations

A multi-system CyberLab can place significant demand on the host.

Important hardware resources include:

* Processor cores
* System memory
* Solid-state storage
* Available disk space
* Cooling
* Network connectivity
* Power stability

Security monitoring platforms such as Wazuh and Splunk generally require more resources than standard endpoint virtual machines.

The host must retain sufficient resources for Windows 11 while multiple virtual machines are running.

---

## Example Host Capacity Planning

The table below provides a general example rather than the exact operational configuration.

| Resource        | Recommended Consideration                                               |
| --------------- | ----------------------------------------------------------------------- |
| Processor       | Modern multi-core 64-bit processor with virtualization support          |
| Memory          | At least 16 GB; 32 GB or more preferred for multiple security platforms |
| Storage         | SSD or NVMe storage strongly preferred                                  |
| Free disk space | Maintain substantial free space for VM disks, logs, and snapshots       |
| Networking      | Stable wired or wireless connection for updates and administration      |
| Cooling         | Adequate ventilation during extended VM workloads                       |
| Power           | Reliable power and safe shutdown procedures                             |

The actual number of virtual machines that can run simultaneously depends on workload, memory allocation, storage performance, and host activity.

---

## Firmware and Virtualization Requirements

Hardware virtualization must be enabled in the system firmware.

Depending on the processor and firmware, the setting may be labeled:

```
Intel Virtualization Technology
Intel VT-x
AMD-V
SVM Mode
Virtualization Technology
```

Additional features may include:

```
Intel VT-d
AMD IOMMU
NX/XD
Secure Boot
Trusted Platform Module
```

The exact firmware interface varies by manufacturer and model.

---

## Verify Virtualization in Windows

Virtualization status can be checked through Task Manager.

1. Open **Task Manager**.
2. Select **Performance**.
3. Select **CPU**.
4. Confirm that **Virtualization** shows as enabled.

PowerShell can also be used to review relevant system information:

```
systeminfo
```

Look for the Hyper-V requirements section and virtualization-related capabilities.

This command does not require administrator privileges for basic system information.

---

## VMware Workstation

VMware Workstation is the primary virtualization platform for the CyberLab.

It provides:

* Virtual machine creation
* Virtual hardware configuration
* Host-only networking
* NAT networking
* ISO mounting
* Virtual disk support
* Snapshot management
* Console access
* Shared clipboard options
* USB device passthrough
* Virtual network editing

The exact VMware product version may change over time and is therefore not hard-coded into the public documentation.

---

## VMware Installation

The installation process should use the official VMware distribution source.

General installation workflow:

1. Download the current supported VMware Workstation installer.
2. Verify that the installer came from an official source.
3. Run the installer.
4. Review optional features before accepting defaults.
5. Complete the installation.
6. Restart Windows if requested.
7. Launch VMware Workstation.
8. Confirm that the application opens successfully.
9. Review the default virtual networks.
10. Create a test virtual machine or inspect an existing VM.

Administrator privileges are normally required to install VMware Workstation and its networking drivers.

Routine VM operation can generally be performed from a standard user account, depending on local permissions and configuration.

---

## Hyper-V and Virtualization Conflicts

Windows virtualization features can affect VMware behavior.

Potentially relevant Windows features include:

* Hyper-V
* Virtual Machine Platform
* Windows Hypervisor Platform
* Windows Sandbox
* Windows Subsystem for Linux 2
* Core Isolation
* Memory Integrity
* Credential Guard

Modern VMware versions may operate alongside some Microsoft virtualization features, but performance or compatibility can vary.

Possible symptoms include:

* Virtual machines failing to start
* Reduced performance
* Nested virtualization failures
* Incompatible device warnings
* VMware using a different virtualization engine
* Network adapter problems

Changes to Windows security or virtualization features should not be made casually.

Before disabling a protection such as Memory Integrity or Credential Guard:

1. Confirm that the feature is causing the issue.
2. Record the original configuration.
3. Review the security impact.
4. Make the smallest necessary change.
5. Validate VMware functionality.
6. Restore the protection when practical.

Administrator privileges are required to enable or disable most Windows virtualization and security features.

---

## Dedicated Lab Directory

Virtual machines should be stored in a dedicated parent directory.

Example sanitized structure:

```
C:\CyberLab\
├── ISOs\
├── Installers\
├── Virtual-Machines\
├── Backups\
├── Exports\
├── Evidence\
└── Documentation\
```

An alternative dedicated drive may also be used:

```
D:\CyberLab\
```

The public repository should avoid displaying a path containing a personal Windows username.

For example, avoid publishing:

```
C:\Users\<PERSONAL_USERNAME>\Documents\Virtual Machines\
```

Use a neutral path in screenshots and documentation where possible.

---

## Recommended Directory Structure

```
<CYBERLAB_ROOT>\
├── ISOs\
│   ├── Windows\
│   ├── Linux\
│   └── Security-Tools\
│
├── Installers\
│   ├── VMware\
│   ├── Wazuh\
│   ├── Splunk\
│   └── Endpoint-Agents\
│
├── Virtual-Machines\
│   ├── DC01\
│   ├── WIN11TARGET\
│   ├── WAZUH-SERVER\
│   ├── SPLUNK-SERVER\
│   └── KALI-TEST\
│
├── Backups\
│   ├── Configurations\
│   ├── VM-Exports\
│   └── Application-Backups\
│
├── Evidence\
│   ├── Screenshots\
│   ├── Event-Logs\
│   ├── Packet-Captures\
│   └── Investigation-Notes\
│
└── Documentation\
    ├── Architecture\
    ├── Runbooks\
    ├── Troubleshooting\
    └── Change-Records\
```

This structure separates installation media, virtual machines, evidence, backups, and documentation.

---

## ISO Management

Installation media should be stored in a dedicated directory.

Examples include:

* Windows Server ISO
* Windows 11 ISO
* Ubuntu Server ISO
* Kali Linux ISO
* VMware Tools images
* Recovery media

Recommended practices:

* Download ISOs only from official sources.
* Preserve the original filename where practical.
* Record the download date.
* Record the operating system version.
* Verify published checksums when available.
* Remove outdated or unsupported images.
* Avoid uploading commercial operating system media to GitHub.
* Do not commit license keys or activation data.

---

## Verify File Hashes

PowerShell can calculate a SHA-256 hash:

```
Get-FileHash -Path "<PATH_TO_ISO>" -Algorithm SHA256
```

Example:

```
Get-FileHash -Path "C:\CyberLab\ISOs\Linux\example.iso" -Algorithm SHA256
```

Compare the result with the checksum published by the software vendor.

This command normally does not require administrator privileges when the current user can read the file.

---

## Installer Management

Installers may include:

* VMware Workstation
* Splunk Enterprise
* Wazuh agents
* Sysmon
* Browser packages
* Windows utilities
* Linux packages transferred for offline installation

Installer storage practices should include:

* Use official vendor sources.
* Retain version information.
* Record checksum values when available.
* Remove installers that are no longer required.
* Scan downloaded files.
* Avoid executing installers directly from temporary browser directories when a managed storage location is available.

---

## Host Storage Planning

Virtual machines consume storage through:

* Base virtual disks
* Operating system updates
* Application logs
* SIEM indexes
* Snapshots
* Suspended memory files
* Temporary files
* Packet captures
* Exported appliances

Snapshots can grow quickly, especially on systems with active log ingestion.

The host should maintain sufficient free space for:

* Normal Windows operation
* VMware temporary files
* Virtual disk growth
* Snapshot creation
* Security platform indexing
* Emergency exports

A lab should not operate with the host drive nearly full.

---

## Storage Monitoring

Storage can be reviewed through File Explorer or PowerShell.

```
Get-Volume
```

A simplified view:

```
Get-PSDrive -PSProvider FileSystem
```

To inspect the size of a CyberLab directory:

```
Get-ChildItem "<CYBERLAB_ROOT>" -Recurse -File |
    Measure-Object -Property Length -Sum
```

For a readable size calculation:

```
$size = (
    Get-ChildItem "<CYBERLAB_ROOT>" -Recurse -File |
    Measure-Object -Property Length -Sum
).Sum

"{0:N2} GB" -f ($size / 1GB)
```

These commands normally do not require administrator privileges when the current user has access to the directory.

---

## Virtual Machine Storage

Each virtual machine should have its own directory.

Example:

```
<CYBERLAB_ROOT>\Virtual-Machines\DC01\
```

A VM directory may contain:

* Configuration files
* Virtual disks
* Snapshot metadata
* Memory state files
* Logs
* Lock files
* NVRAM data

Files inside an active VM directory should not be manually modified unless the action is part of a documented recovery procedure.

---

## Do Not Synchronize Active VMs with Consumer Cloud Storage

Actively running virtual machines should generally not be placed inside a folder continuously synchronized by services such as:

* OneDrive
* Dropbox
* Google Drive
* Consumer backup synchronization tools

Continuous synchronization can cause:

* File locking
* Partial uploads
* Corrupted VM states
* Snapshot inconsistencies
* Excessive bandwidth use
* Large cloud storage consumption

A safer approach is to export or copy a powered-off VM into a designated backup location.

---

## Virtual Machine Naming

Use consistent virtual machine names.

Recommended public naming:

```
DC01
WIN11TARGET
WAZUH-SERVER
SPLUNK-SERVER
KALI-TEST
```

Names should communicate system role without including:

* Personal names
* Physical location
* Home address references
* Public IP information
* Account names
* Sensitive project identifiers

Consistency improves troubleshooting, screenshots, diagrams, and documentation.

---

## VM Resource Allocation

Each virtual machine should receive only the resources it needs.

General allocation considerations:

| System            |              CPU |   Memory | Storage Priority |
| ----------------- | ---------------: | -------: | ---------------- |
| Domain Controller |  Low to moderate | Moderate | Moderate         |
| Windows Endpoint  |         Moderate | Moderate | Moderate         |
| Wazuh Server      | Moderate to high |     High | High             |
| Splunk Server     | Moderate to high |     High | High             |
| Kali Linux        |         Moderate | Moderate | Moderate         |

The exact values depend on:

* Host capacity
* Workload
* Number of simultaneous VMs
* SIEM data volume
* Operating system requirements

Avoid assigning every available processor core or most of the host memory to virtual machines.

The host must remain responsive.

---

## CPU Allocation Guidance

Over-allocation can reduce performance.

A virtual machine assigned many virtual processors does not automatically perform better.

Potential problems include:

* Increased scheduling delay
* Host contention
* Reduced responsiveness
* Longer suspend and resume times
* Poor performance when multiple VMs run together

Start with a modest allocation and increase it only after observing sustained resource pressure.

---

## Memory Allocation Guidance

Memory is often the primary limiting factor in a multi-VM CyberLab.

The host requires memory for:

* Windows 11
* VMware Workstation
* Browsers
* Documentation tools
* Security dashboards
* Background services

The virtual machines require memory for:

* Operating systems
* Databases
* Search indexes
* Security agents
* Dashboards
* Containers

Wazuh and Splunk should not be allocated memory based solely on their ability to start. They also need enough memory to remain responsive during indexing and searches.

---

## Dynamic Versus Preallocated Virtual Disks

VMware virtual disks may be configured to grow as needed or be preallocated.

### Growable Disks

Advantages:

* Consume less space initially
* Easier to deploy
* Suitable for many lab systems

Disadvantages:

* Can become fragmented
* May expand unexpectedly
* Require active free-space monitoring

### Preallocated Disks

Advantages:

* Predictable storage consumption
* Potential performance consistency

Disadvantages:

* Consume full capacity immediately
* Require more host storage
* Take longer to create or copy

Growable disks are generally acceptable for a personal CyberLab if free space is monitored.

---

## Split Versus Single Virtual Disk Files

VMware may store a virtual disk as:

* One large file
* Multiple smaller files

Multiple files can simplify movement between file systems with size limitations.

A single file may be easier to manage on a modern NTFS volume.

The selected option should remain consistent unless there is a specific need to change it.

---

## Snapshot Storage Impact

Snapshots preserve changes made after a baseline state.

They can consume substantial disk space because active changes are written to delta files.

Snapshot growth is influenced by:

* Windows updates
* SIEM indexing
* Splunk ingestion
* Log generation
* Packet captures
* Software installation
* Large file transfers

Long snapshot chains should be avoided.

Snapshots should be:

* Named clearly
* Created at meaningful milestones
* Reviewed periodically
* Consolidated when appropriate
* Removed only after confirming they are no longer required

---

## Snapshot Naming Standard

Recommended format:

```
<SEQUENCE>-<SYSTEM>-<MILESTONE>-<DATE>
```

Example:

```
01-DC01-Clean-Install-YYYY-MM-DD
02-DC01-Domain-Configured-YYYY-MM-DD
03-WIN11TARGET-Domain-Joined-YYYY-MM-DD
04-WAZUH-Agent-Enrolled-YYYY-MM-DD
05-SPLUNK-Web-Validated-YYYY-MM-DD
```

Do not include usernames, passwords, internal addresses, or confidential incident details in snapshot names.

---

## Powered-Off Snapshot Practice

For major stable milestones, the preferred method is:

1. Shut down the guest operating system cleanly.
2. Confirm that the VM shows as powered off.
3. Create the snapshot.
4. Add a descriptive note.
5. Restart the VM.
6. Confirm that required services return.
7. Record the snapshot in the change log.

A powered-off snapshot avoids preserving a potentially inconsistent memory state.

Memory snapshots may still be useful for short-term testing, but they should not replace stable powered-off baselines.

---

## Host Network Interfaces

The host may contain several network interfaces, including:

* Physical Ethernet
* Physical Wi-Fi
* VPN adapter
* VMware host-only adapter
* VMware NAT adapter
* Bluetooth network adapter
* Virtualization-related adapters

The presence of multiple adapters can complicate troubleshooting.

Useful command:

```
Get-NetAdapter
```

Detailed IP configuration:

```
Get-NetIPConfiguration
```

Routing table:

```
route print
```

Or:

```
Get-NetRoute
```

These commands generally do not require administrator privileges for viewing information.

Changing adapters, routes, or interface metrics usually requires administrator privileges.

---

## VPN Considerations

A host VPN can affect CyberLab connectivity.

Possible effects include:

* DNS replacement
* Default route changes
* Blocked private network access
* Changed interface priority
* Internet access failures in NAT-connected VMs
* Dashboard access problems
* Split-tunneling conflicts

When troubleshooting unexpected network behavior:

1. Record the current VPN state.
2. Review host adapters.
3. Review DNS configuration.
4. Review the routing table.
5. Test with the VPN temporarily disconnected when safe.
6. Re-enable the VPN after testing.
7. Document any required exclusions or split-tunnel rules.

Do not disable security software permanently as a troubleshooting shortcut.

---

## Host Firewall

Windows Defender Firewall should remain enabled.

The firewall protects the host from:

* Unsolicited inbound connections
* Unintended service exposure
* Malicious or misconfigured lab traffic
* Connections from other physical network devices

Firewall rules should be specific.

Avoid broad rules that allow:

```
Any program
Any port
Any protocol
Any remote address
```

A better rule limits access by:

* Program
* Protocol
* Local port
* Remote address
* Network profile
* Direction

Administrator privileges are required to create or modify Windows Firewall rules.

---

## Review Firewall Profiles

PowerShell:

```
Get-NetFirewallProfile
```

Review enabled status:

```
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

This command normally allows read-only access without elevation.

---

## Windows Defender

Microsoft Defender Antivirus should remain enabled unless a specific incompatibility has been confirmed.

Recommended host protections include:

* Real-time protection
* Cloud-delivered protection
* Tamper protection
* Automatic sample submission
* Potentially unwanted application blocking
* Controlled folder access where compatible
* Network protection where compatible

Security exclusions should be used sparingly.

---

## Antivirus Exclusions for Virtual Machines

Virtualization platforms sometimes recommend exclusions for active VM files to reduce performance overhead.

However, excluding entire CyberLab directories creates a security visibility gap.

Before adding an exclusion:

1. Confirm that antivirus scanning is causing a measurable problem.
2. Identify the smallest necessary file type or path.
3. Consider whether the lab may contain untrusted samples.
4. Avoid excluding evidence, downloads, or malware-testing directories.
5. Document the exclusion.
6. Review it periodically.

A Blue Team learning lab should not assume that every downloaded tool or file is safe.

Administrator privileges are required to configure Defender exclusions.

---

## BitLocker

Full-disk encryption can protect virtual machine files, credentials, logs, and evidence if the laptop is lost or stolen.

BitLocker considerations include:

* Recovery key storage
* TPM support
* Boot configuration
* External drive encryption
* Backup accessibility
* Performance impact

Recovery keys should not be stored only on the encrypted device.

Do not publish recovery keys or screenshots containing them.

---

## Check BitLocker Status

From an elevated command prompt:

```
manage-bde -status
```

Or through PowerShell:

```
Get-BitLockerVolume
```

Administrator privileges may be required for complete information and any configuration changes.

---

## Host Account Separation

The host should use account separation where practical.

Possible roles include:

* Standard daily-use account
* Local administrator account
* Lab-specific service accounts
* Domain accounts inside the virtual environment

The Windows host should not be joined to the lab domain unless that is a deliberate exercise.

Using separate host and domain credentials reduces the chance of credential reuse across trust boundaries.

---

## Credential Practices

Recommended practices:

* Do not reuse personal passwords in the lab.
* Do not reuse lab administrator passwords on the physical host.
* Use unique passwords for Wazuh, Splunk, Windows, and Linux accounts.
* Store credentials in a password manager.
* Avoid plain-text credential files.
* Do not place credentials in PowerShell history.
* Remove credentials from screenshots.
* Rotate credentials after accidental exposure.
* Avoid committing configuration files containing secrets.

---

## Windows Update

The host should receive regular Windows updates.

Updates may affect:

* VMware compatibility
* Network adapters
* Hypervisor behavior
* Drivers
* Windows security features
* Reboot requirements

Before a major Windows feature update:

1. Shut down all virtual machines.
2. Confirm that important snapshots exist.
3. Back up critical VM configurations.
4. Record the current VMware version.
5. Apply the update.
6. Restart the host.
7. Test VMware Workstation.
8. Validate host-only networking.
9. Validate NAT networking.
10. Start each critical VM and verify services.

---

## Driver Updates

Relevant drivers may include:

* Chipset
* Graphics
* Storage controller
* Ethernet
* Wi-Fi
* Bluetooth
* Firmware
* BIOS or UEFI updates

Use the hardware manufacturer or trusted Windows update channels.

Avoid third-party driver download websites.

Firmware updates should be performed with reliable power and current backups.

---

## Power Configuration

Virtual machines can be affected by host sleep and hibernation.

Potential issues include:

* Interrupted services
* Time drift
* Broken network connections
* Suspended SIEM indexing
* Corrupted transfers
* Inconsistent lab state

For extended lab sessions, configure Windows power settings so the host does not unexpectedly sleep while critical virtual machines are running.

Laptop battery protection and thermal considerations should still be respected.

---

## Recommended Power Practices

* Keep the host connected to reliable power during long exercises.
* Shut down virtual machines before restarting Windows.
* Avoid forced shutdowns.
* Monitor heat during sustained SIEM workloads.
* Do not block ventilation.
* Avoid operating the laptop on soft surfaces.
* Use the guest operating system shutdown process before powering off a VM.
* Suspend VMs only for short interruptions when appropriate.

---

## Host Thermal Monitoring

Multiple active virtual machines can increase CPU temperature and fan activity.

Potential warning signs include:

* Sustained high fan speed
* Performance throttling
* Unexpected host shutdown
* VMware lag
* Slow indexing
* Unresponsive dashboards

Temperature monitoring can be performed using trusted system utilities or vendor software.

No third-party hardware utility should be granted kernel-level access without reviewing its source and reputation.

---

## Browser Administration

The host browser may be used to access:

* Wazuh Dashboard
* Splunk Web
* VMware documentation
* Internal application interfaces
* GitHub
* Confluence
* Vendor documentation

Recommended browser practices:

* Use a dedicated browser profile for the lab.
* Avoid saving lab administrator passwords in an unmanaged browser profile.
* Remove cookies before capturing screenshots when necessary.
* Review open tabs before publishing screenshots.
* Hide bookmarks and personal profile information.
* Avoid exposing internal URLs.
* Use HTTPS when supported.
* Treat self-signed certificate warnings as configuration items, not automatic permission to ignore risk.

---

## Terminal Administration

Common administrative tools include:

* PowerShell
* Windows Terminal
* Command Prompt
* SSH clients
* VMware console
* Browser-based terminals

PowerShell is preferred for repeatable Windows administration because commands can be documented and converted into scripts.

---

## PowerShell Execution Policy

Execution policy affects script behavior but is not a complete security boundary.

Check current policy:

```
Get-ExecutionPolicy -List
```

Do not globally weaken execution policy solely to run an unknown script.

A temporary process-scoped policy may be safer when a trusted script requires it:

```
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

This change affects only the current PowerShell process.

The script should still be reviewed before execution.

Some execution policy changes require administrator privileges depending on scope.

---

## File Transfer Into Virtual Machines

Installation files may need to be transferred from the host into a VM.

Possible methods include:

* ISO image
* VMware shared folder
* Drag and drop
* Clipboard transfer
* Temporary web server
* SCP
* SMB share
* USB passthrough
* Direct browser download inside the VM

Each method has different security and reliability considerations.

For isolated systems, a temporary ISO containing required installers can provide a controlled transfer method.

---

## Shared Folder Risks

VMware shared folders improve convenience but cross the isolation boundary between host and guest.

Risks include:

* Guest access to host files
* Accidental modification
* Malware reaching shared data
* Credential or documentation exposure
* Evidence contamination

Shared folders should be:

* Disabled when not needed
* Limited to a dedicated transfer directory
* Read-only where practical
* Excluded from sensitive personal directories
* Reviewed after each exercise

Do not share the entire user profile or primary document directory with an untrusted VM.

---

## Clipboard and Drag-and-Drop Risks

Clipboard sharing and drag-and-drop can also cross the host-guest boundary.

They may transfer:

* Text
* Commands
* URLs
* Credentials
* Files
* Malicious content

These features may be disabled for higher-risk exercises.

The operator should never paste a personal password into a lab VM.

---

## VMware Tools

VMware Tools improves:

* Guest display resizing
* Time synchronization
* Clipboard support
* Mouse integration
* Shutdown behavior
* Network and storage drivers

Security tradeoffs should be considered because some convenience features increase host-guest integration.

Install VMware Tools from trusted sources associated with the installed VMware product.

---

## Time Synchronization

Accurate time is important for:

* Active Directory
* Kerberos
* Wazuh alerts
* Splunk searches
* Incident timelines
* Snapshot documentation
* Log correlation

The host should maintain accurate system time.

Virtual machine time synchronization should be tested because guest systems may use:

* VMware time synchronization
* Windows Time service
* Domain hierarchy
* Network Time Protocol
* Linux systemd-timesyncd
* Chrony

Conflicting time sources can create drift.

---

## Host Logging

The Windows host records useful events through:

* Event Viewer
* Windows Defender logs
* Firewall logs
* VMware logs
* PowerShell logs
* Reliability Monitor
* Windows Update history

These sources can help diagnose:

* VMware crashes
* Driver failures
* Network problems
* Defender detections
* Unexpected shutdowns
* Application errors

---

## Reliability Monitor

Reliability Monitor provides a timeline of:

* Application failures
* Windows failures
* Driver issues
* Updates
* Warnings
* Successful installations

Open it by searching for:

```
View reliability history
```

It is useful when an issue began after an update or installation.

---

## VMware Log Files

Each VM directory contains VMware log files, commonly named:

```
vmware.log
vmware-0.log
vmware-1.log
```

These logs may contain:

* VM startup errors
* Virtual device errors
* Disk problems
* Network adapter issues
* Snapshot issues
* Permission failures

Logs should be sanitized before publication because they may include:

* Full file paths
* Hostnames
* Usernames
* Device identifiers
* VM configuration details

---

## Host Backup Strategy

The host backup strategy should protect:

* VM configuration files
* Important virtual disks
* Documentation
* Application configuration exports
* Detection rules
* Splunk searches
* Wazuh custom rules
* Screenshots
* Investigation notes
* Recovery procedures

Snapshots alone are not sufficient because they reside with the VM.

---

## Backup Types

### Configuration Backup

Includes:

* VMware VM configuration files
* Network configuration records
* Splunk configuration exports
* Wazuh custom rules
* Agent configuration
* PowerShell scripts
* Documentation

### VM Export or Cold Copy

A powered-off virtual machine can be:

* Exported
* Copied to external storage
* Archived
* Stored on a separate drive

### Evidence Backup

Includes:

* Packet captures
* Event logs
* Screenshots
* Investigation worksheets
* Query results
* Detection test records

Evidence should be separated from disposable lab VMs when practical.

---

## Safe Cold Backup Procedure

1. Shut down the guest operating system.
2. Confirm that the VM is powered off.
3. Close VMware Workstation if required.
4. Copy or export the VM directory.
5. Store the backup on separate media.
6. Verify that the copied files are readable.
7. Record the backup date and VM state.
8. Restore-test important backups periodically.

Copying an active VM can produce an inconsistent backup.

---

## Backup Naming Standard

Example:

```
<SYSTEM>-<MILESTONE>-<YYYY-MM-DD>
```

Examples:

```
DC01-Domain-Configured-YYYY-MM-DD
WIN11TARGET-Agent-Validated-YYYY-MM-DD
WAZUH-SERVER-Baseline-YYYY-MM-DD
SPLUNK-SERVER-Clean-Install-YYYY-MM-DD
```

Do not include passwords, internal addresses, or personal identifiers.

---

## Recovery Planning

Host recovery planning should answer:

* Where are VM backups stored?
* Which VM must be restored first?
* Which snapshots are known-good?
* Are installation ISOs still available?
* Are product installers preserved?
* Are license details stored securely?
* Are passwords available in a password manager?
* Are architecture diagrams current?
* Are network settings documented?
* Can the domain controller be recovered independently?

A documented restore order reduces confusion during failure recovery.

---

## Recommended Restore Order

A typical restore order is:

1. Windows host and VMware Workstation
2. VMware virtual networks
3. Domain controller
4. Windows endpoint
5. Wazuh server
6. Splunk server
7. Kali testing system
8. Agents and data forwarding
9. Dashboards and detections
10. Validation exercises

Identity and DNS should generally be restored before dependent domain systems.

---

## Host Change Management

Significant host changes should be recorded.

Examples include:

* VMware upgrades
* Windows feature updates
* Network adapter changes
* Storage relocation
* Firewall changes
* Defender exclusions
* Virtual network changes
* Firmware updates
* Major driver updates

A simple change record should include:

```
Date:
Change:
Reason:
Systems affected:
Backup or snapshot:
Validation:
Rollback plan:
Outcome:
```

---

## Host Maintenance Schedule

### Before Each Lab Session

* Confirm available disk space.
* Confirm VMware starts.
* Confirm expected host adapters exist.
* Confirm required VMs are powered off or stable.
* Confirm the host has no pending forced restart.
* Confirm backup status for high-risk exercises.

### Weekly

* Review Windows updates.
* Review Defender status.
* Review disk usage.
* Review snapshot growth.
* Remove unnecessary installation files.
* Check VMware errors.

### Monthly

* Review backup copies.
* Review firewall changes.
* Review Defender exclusions.
* Review VM resource assignments.
* Review old snapshots.
* Test at least one recovery procedure.
* Update architecture and inventory documentation.

---

## Host Validation Checklist

### Operating System

* Windows starts normally.
* Windows activation and licensing are handled privately.
* Security updates are current.
* Defender is active.
* Firewall profiles are enabled.
* Time and timezone are correct.

### VMware

* VMware Workstation launches.
* Required virtual networks exist.
* Host-only networking works.
* NAT networking works.
* VMs can start.
* Snapshots can be created.
* VMware logs show no critical recurring errors.

### Storage

* Adequate free space remains.
* VM directories are accessible.
* Snapshot chains are manageable.
* Backups exist on separate storage.
* ISO files have known sources.

### Security

* Lab credentials are unique.
* Shared folders are limited.
* No lab service is publicly exposed.
* Sensitive screenshots are not stored in the public repository.
* The host is not using lab domain credentials.
* VPN behavior is understood and documented.

---

## Troubleshooting: VMware Will Not Start a VM

Possible causes include:

* Hardware virtualization disabled
* Hypervisor conflict
* Insufficient memory
* Locked VM files
* Corrupted snapshot chain
* Permission issue
* Incomplete Windows update
* VMware service failure

Suggested checks:

```
systeminfo
Get-Service | Where-Object DisplayName -Match "VMware"
Get-Process | Where-Object ProcessName -Match "vmware"
```

Review the VM’s `vmware.log`.

Administrative privileges may be required to restart VMware services or change Windows features.

---

## Troubleshooting: VM Has No Network Connectivity

Check:

1. The virtual adapter is connected.
2. The correct VMware network is selected.
3. The guest interface is enabled.
4. The guest has a valid IP address.
5. The VMware NAT or DHCP service is running where required.
6. The host adapter exists.
7. The host firewall is not blocking the intended traffic.
8. A VPN has not replaced the route or DNS settings.

Host commands:

```
Get-NetAdapter
Get-NetIPConfiguration
Get-NetRoute
```

Guest commands:

```
ipconfig /all
```

Or:

```
ip address
ip route
```

---

## Troubleshooting: Host Disk Is Filling Quickly

Likely causes include:

* Snapshot growth
* Splunk indexes
* Wazuh data
* Packet captures
* Suspended VM memory
* Duplicate ISO files
* VM exports
* Windows update files

Actions:

1. Identify the largest directories.
2. Shut down unnecessary VMs.
3. Review snapshot usage.
4. Remove unneeded packet captures.
5. Archive old exports.
6. Remove obsolete installation media.
7. Do not manually delete active snapshot files.
8. Expand or relocate storage through a documented process.

---

## Troubleshooting: VM Performance Is Poor

Possible causes include:

* Excessive CPU allocation
* Insufficient host memory
* Host memory pressure
* Slow storage
* Antivirus scanning active VM disks
* Too many simultaneous VMs
* Long snapshot chains
* Thermal throttling
* Windows background updates

Recommended approach:

1. Measure host CPU, memory, disk, and temperature.
2. Shut down nonessential VMs.
3. Review guest resource usage.
4. Reduce over-allocated virtual CPUs.
5. Confirm adequate free disk space.
6. Review snapshot chains.
7. Test whether security scanning is causing measurable delay.
8. Avoid making broad permanent antivirus exclusions.

---

## Public Documentation Sanitization

Before publishing host screenshots or logs, remove:

* Windows username
* Microsoft account email
* Computer serial number
* Product key
* Windows activation identifiers
* Public IP address
* Home network name
* Internal IP addresses where not required
* MAC addresses
* VPN server details
* Browser bookmarks
* Personal file paths
* Recent file lists
* VMware UUIDs
* License details
* Password manager content
* Recovery keys
* Hidden notification content

Use placeholders such as:

```
<WINDOWS_HOST>
<CYBERLAB_ROOT>
<HOST_ONLY_NETWORK>
<NAT_NETWORK>
<PERSONAL_USERNAME>
<REDACTED_PATH>
```

---

## Screenshot Preparation

Before taking a public screenshot:

1. Close unrelated applications.
2. Close personal browser tabs.
3. Hide bookmarks.
4. Clear notifications.
5. Move sensitive files out of view.
6. Use neutral example paths where practical.
7. Review the full image at high zoom.
8. Crop unnecessary interface areas.
9. Redact sensitive values permanently.
10. Reopen the final image to verify the redaction.

Do not rely on a semi-transparent overlay that could be removed.

---

## Skills Demonstrated

This host configuration work demonstrates:

* Windows 11 administration
* VMware Workstation management
* Virtual resource planning
* Storage management
* Host hardening
* PowerShell usage
* Network adapter troubleshooting
* Backup planning
* Snapshot management
* Change management
* Virtualization security
* Documentation sanitization
* Recovery planning
* Lab operations

---

## Lessons Learned

Key lessons include:

* The virtualization host is part of the security boundary.
* SIEM workloads require deliberate memory and storage planning.
* Snapshots can consume more space than expected.
* Host VPNs can affect virtual network routing and DNS.
* Shared folders reduce isolation and should be limited.
* Active VM directories should not be placed in continuously synchronized folders.
* Windows updates can affect VMware drivers and networking.
* Backups must exist outside the active VM directory.
* A powered-off snapshot provides a cleaner stable milestone.
* Public documentation requires a separate review from technical validation.

---

## Planned Improvements

Future host improvements may include:

* Dedicated external backup storage
* Automated disk-space reporting
* Automated VM inventory export
* Scheduled configuration backups
* UPS support
* Additional host monitoring
* Standardized VM templates
* Automated snapshot inventory
* Host hardening checklist
* Lab-specific Windows account separation
* Secure evidence archive
* Automated validation scripts
* Documented disaster recovery test
* Dedicated storage volume for virtual machines

---

## Summary

The Acer Windows 11 host provides the compute, storage, networking, administration, and recovery foundation for the Blue Team CyberLab.

A reliable host configuration requires more than installing VMware Workstation. It requires deliberate planning for:

* Resources
* Storage
* Isolation
* Security
* Updates
* Backups
* Snapshots
* Credentials
* File transfers
* Recovery

By treating the host as critical infrastructure, the CyberLab becomes more stable, repeatable, secure, and suitable for long-term defensive security training.

