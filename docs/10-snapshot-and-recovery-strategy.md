# Snapshot and Recovery Strategy

## Overview

The Acer Blue Team CyberLab uses VMware snapshots, configuration backups, powered-off virtual machine copies, and documented restore procedures to protect the lab from failed changes, broken configurations, and exercise-related instability.

The recovery strategy is designed to support:

* Safe experimentation
* Repeatable defensive exercises
* Configuration rollback
* Failed-update recovery
* Domain and endpoint repair
* SIEM recovery
* Evidence preservation
* Change validation
* Disaster-recovery practice
* Long-term lab maintenance

Snapshots are useful, but they are not complete backups.

A reliable recovery plan combines several protection methods and validates that restored systems still function correctly.

All hostnames, IP addresses, usernames, file paths, snapshot names, backup locations, and internal configuration values in this public document are sanitized examples.

---

## Recovery Goals

The recovery strategy supports the following goals:

* Restore a failed virtual machine quickly
* Reverse unsafe or unsuccessful changes
* Preserve known-good lab milestones
* Protect important configurations outside the active VM
* Recover from snapshot corruption or host storage failure
* Maintain Active Directory consistency
* Restore Wazuh and Splunk services
* Reconnect monitoring agents and forwarders
* Preserve exercise evidence before rollback
* Document recovery decisions and outcomes

---

## Systems Covered

The recovery strategy applies to:

* Acer Windows 11 virtualization host
* DC01 domain controller
* WIN11TARGET Windows endpoint
* WAZUH-SERVER
* SPLUNK-SERVER
* KALI-TEST
* VMware virtual networks
* ISO and installer storage
* Wazuh configuration
* Splunk configuration
* Exercise evidence
* Documentation and runbooks

---

## Recovery Architecture

```text
Active CyberLab
      |
      | Stable milestone
      v
VMware Snapshot
      |
      | Independent protection
      v
Powered-Off VM Copy or Export
      |
      | Configuration preservation
      v
Application and System Backups
      |
      | Operational guidance
      v
Recovery Runbooks and Validation
```

Each layer protects against a different type of failure.

---

## Recovery Methods

The CyberLab uses four primary recovery methods:

* VMware snapshots
* Cold copies or VM exports
* Application and system configuration backups
* Documentation and recovery runbooks

No single method should be treated as sufficient on its own.

---

## Snapshot Definition

A VMware snapshot preserves the state of a virtual machine at a specific point in time.

Depending on the snapshot options, it may preserve:

* Virtual disk state
* Virtual machine configuration
* Memory state
* Device state

Snapshots are useful for:

* Testing configuration changes
* Installing monitoring agents
* Applying Group Policy
* Adding software
* Performing detection exercises
* Testing recovery procedures
* Creating stable milestones

---

## Snapshot Limitations

Snapshots are not independent backups because they usually remain stored with the active virtual machine.

A snapshot does not protect against:

* Host disk failure
* VM-directory deletion
* File corruption affecting the entire VM
* Ransomware on the host
* Loss of the physical laptop
* Storage-controller failure
* Accidental deletion of the VM directory
* Failure of the external filesystem containing the VM
* Theft or physical damage

Snapshots should therefore be combined with separate backups.

---

## Snapshot Chain Risks

A snapshot chain contains the base virtual disk and one or more delta disks.

Long chains can cause:

* Increased storage use
* Slower disk performance
* Longer startup and shutdown
* More complex recovery
* Greater risk during consolidation
* Confusion about the active state
* Larger backup requirements

Snapshots should represent meaningful milestones rather than every minor change.

---

## Snapshot Types

### Powered-Off Snapshot

Created after the guest operating system is shut down cleanly.

Advantages:

* Cleaner disk state
* No suspended memory state
* Lower risk of application inconsistency
* Easier long-term milestone management
* More suitable for stable baselines

### Running Snapshot Without Memory

Captures disk state while the VM is running but does not preserve active memory.

Advantages:

* Faster than a shutdown-based milestone
* Useful before some short tests

Risks:

* Applications may have pending writes
* Databases may require recovery
* Services may restart inconsistently

### Snapshot with Memory

Preserves the running state and active memory.

Advantages:

* Fast return to the exact session
* Useful for short-lived testing

Risks:

* Larger snapshot
* Potential time drift
* Stale network sessions
* Inconsistent long-term state
* Credentials or sensitive data preserved in memory
* Greater restoration complexity

Stable project milestones should normally use powered-off snapshots.

---

## Snapshot Naming Standard

Use clear, consistent snapshot names.

Recommended format:

```text
<SEQUENCE>-<SYSTEM>-<MILESTONE>-<DATE>
```

Examples:

```text
01-DC01-Clean-Install-YYYY-MM-DD
02-DC01-Domain-Promoted-YYYY-MM-DD
03-WIN11TARGET-Domain-Joined-YYYY-MM-DD
04-WAZUH-Agent-Validated-YYYY-MM-DD
05-SPLUNK-Web-Validated-YYYY-MM-DD
06-KALI-Exercise-Ready-YYYY-MM-DD
```

For pre-exercise snapshots:

```text
PRE-EX01-Failed-Logon-YYYY-MM-DD
PRE-EX04-Group-Change-YYYY-MM-DD
PRE-EX09-Network-Recon-YYYY-MM-DD
```

---

## Snapshot Description Standard

Each snapshot should include a short description.

Example:

```text
System:
Milestone:
Date:
Services validated:
Known limitations:
Reason for snapshot:
Rollback conditions:
```

A snapshot name alone may not provide enough recovery context.

---

## Snapshot Register

Maintain a private snapshot register.

Example:

| Snapshot                     | System        | Date       | Purpose                     | Validation               | Retention |
| ---------------------------- | ------------- | ---------- | --------------------------- | ------------------------ | --------- |
| 01-DC01-Clean-Install        | DC01          | YYYY-MM-DD | Base Windows Server state   | Boot validated           | Long-term |
| 02-DC01-Domain-Promoted      | DC01          | YYYY-MM-DD | AD DS and DNS working       | `dcdiag` passed          | Long-term |
| 03-WIN11TARGET-Domain-Joined | WIN11TARGET   | YYYY-MM-DD | Domain membership           | Secure channel passed    | Long-term |
| 04-WAZUH-Agent-Validated     | WAZUH-SERVER  | YYYY-MM-DD | Agent and dashboard working | Test event received      | Long-term |
| 05-SPLUNK-Web-Validated      | SPLUNK-SERVER | YYYY-MM-DD | Splunk Web operational      | Port and login validated | Long-term |

Do not publish operational snapshot notes containing internal addresses or credentials.

---

## Recommended Snapshot Milestones

### DC01

```text
01-DC01-Clean-Install
02-DC01-Patched-Baseline
03-DC01-Static-Network
04-DC01-Domain-Promoted
05-DC01-DNS-Validated
06-DC01-OUs-and-Users
07-DC01-GPO-Baseline
08-DC01-Endpoint-Join-Validated
```

### WIN11TARGET

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

### WAZUH-SERVER

```text
01-WAZUH-Ubuntu-Clean-Install
02-WAZUH-Ubuntu-Patched
03-WAZUH-Network-Configured
04-WAZUH-Docker-Installed
05-WAZUH-Stack-Deployed
06-WAZUH-Dashboard-Validated
07-WAZUH-Agent-Enrolled
08-WAZUH-Windows-Events-Validated
09-WAZUH-FIM-Validated
10-WAZUH-Exercise-Ready
```

### SPLUNK-SERVER

```text
01-SPLUNK-Ubuntu-Clean-Install
02-SPLUNK-Ubuntu-Patched
03-SPLUNK-Network-Configured
04-SPLUNK-Package-Installed
05-SPLUNK-Web-Validated
06-SPLUNK-Test-Data-Ingested
07-SPLUNK-Receiver-Enabled
08-SPLUNK-Forwarder-Connected
09-SPLUNK-Windows-Events-Validated
10-SPLUNK-Exercise-Ready
```

### KALI-TEST

```text
01-KALI-Clean-Install
02-KALI-Patched-Baseline
03-KALI-VMware-Tools
04-KALI-Network-Configured
05-KALI-Core-Tools-Validated
06-KALI-Packet-Capture-Validated
07-KALI-SIEM-Test-Validated
08-KALI-Exercise-Ready
```

---

## Safe Snapshot Procedure

For a stable milestone:

1. End active exercises.
2. Save required evidence.
3. Confirm no update or installation is running.
4. Validate system health.
5. Stop application services when appropriate.
6. Shut down the guest operating system cleanly.
7. Confirm the VM shows as powered off.
8. Create the snapshot.
9. Apply the naming standard.
10. Add a description.
11. Restart the VM.
12. Revalidate critical services.
13. Record the snapshot in the register.

---

## Health Validation Before Snapshot

Before creating a stable snapshot, confirm the system is not already broken.

### DC01

```powershell
dcdiag
```

```powershell
Get-Service NTDS, DNS, Netlogon, Kdc
```

### WIN11TARGET

```powershell
Test-ComputerSecureChannel -Verbose
```

```powershell
Get-NetIPConfiguration
```

### WAZUH-SERVER

```bash
docker compose ps
```

```bash
df -h
```

### SPLUNK-SERVER

```bash
sudo /opt/splunk/bin/splunk status
```

```bash
df -h
```

### KALI-TEST

```bash
ip address
ip route
timedatectl
```

A snapshot of an unhealthy system may preserve the problem.

---

## Application-Aware Shutdown

Some systems should have services stopped before the operating system is shut down.

### Wazuh

From the deployment directory:

```bash
docker compose down
```

Do not use:

```bash
docker compose down -v
```

unless deleting application data is intentional.

### Splunk

```bash
sudo /opt/splunk/bin/splunk stop
```

### Windows Systems

Use the Windows shutdown process rather than VMware power-off.

### Kali

Stop packet captures and testing tools before shutdown.

---

## Snapshot Retention Categories

Snapshots may be classified as:

### Baseline

Long-term clean or foundational state.

### Milestone

Important completed configuration stage.

### Pre-Change

Temporary protection before a significant change.

### Pre-Exercise

Temporary protection before a potentially disruptive exercise.

### Troubleshooting

Short-lived state created during investigation.

### Recovery Test

Created specifically to validate restoration.

Baseline and milestone snapshots are usually retained longer than troubleshooting snapshots.

---

## Snapshot Review Schedule

### Weekly

* Review newly created snapshots.
* Confirm names and descriptions.
* Remove accidental duplicates.
* Check host free space.

### Monthly

* Review snapshot-chain length.
* Identify obsolete pre-change snapshots.
* Confirm milestone snapshots remain valid.
* Update the snapshot register.

### Before Major Changes

* Confirm a stable snapshot exists.
* Confirm an independent backup exists where needed.
* Record the rollback point.

---

## Snapshot Deletion Safety

Deleting a snapshot can trigger consolidation.

Before deletion:

1. Confirm the snapshot is no longer required.
2. Confirm a later stable snapshot exists.
3. Confirm adequate host disk space.
4. Close unnecessary applications.
5. Avoid shutting down the host during consolidation.
6. Monitor VMware progress.
7. Validate the VM afterward.
8. Record the removal.

Do not manually delete snapshot files from the VM directory.

---

## Snapshot Consolidation

VMware may report that disks require consolidation.

Possible causes include:

* Snapshot removal
* Interrupted snapshot operation
* Backup software
* File-lock issue
* Incomplete disk merge

Before consolidation:

* Back up the VM directory if possible.
* Confirm host disk space.
* Shut down the VM.
* Close other resource-intensive applications.
* Avoid interrupting the process.

After consolidation, start and validate the VM.

---

## Restore Decision Process

Before restoring a snapshot, determine whether rollback is the correct recovery method.

Ask:

* Is the failure limited to one VM?
* Is the desired snapshot known-good?
* Will rollback remove important evidence?
* Will the restore create domain trust issues?
* Will the restore affect agent or forwarder identity?
* Will recent indexed data be lost?
* Is a configuration repair safer than a full rollback?
* Does another dependent VM need to be restored to a matching point?

Snapshot restoration should be deliberate rather than automatic.

---

## Preserve Evidence Before Restore

Before reverting:

1. Record the failure time.
2. Capture relevant screenshots.
3. Export logs.
4. Save configuration files.
5. Save packet captures.
6. Export SIEM search results.
7. Hash important evidence.
8. Record the current VM state.
9. Store evidence outside the VM.
10. Confirm the evidence is accessible.

Restoring a snapshot may erase the most useful evidence of the failure.

---

## General Snapshot Restore Procedure

1. Stop active work.
2. Preserve evidence.
3. Record the current snapshot state.
4. Shut down the VM when possible.
5. Select the intended snapshot.
6. Confirm the snapshot description.
7. Revert the VM.
8. Start the system.
9. Confirm time and timezone.
10. Confirm network configuration.
11. Confirm required services.
12. Generate a harmless validation event.
13. Confirm monitoring and dependencies.
14. Document the outcome.

---

## Post-Restore Validation

Every restored VM should be validated.

### General

* Hostname
* Date and time
* Timezone
* IP address
* Route table
* DNS
* Disk space
* System services
* VMware adapter state

### Domain Systems

* Domain controller availability
* DNS
* Secure channel
* Group Policy
* User sign-in

### Monitoring Systems

* Wazuh containers
* Wazuh agents
* Splunk service
* Splunk forwarder
* Index availability
* New event ingestion

---

## Domain Controller Snapshot Risks

Domain controllers require special care.

Potential issues include:

* Time rollback
* Authentication failures
* Computer account password mismatch
* Stale DNS records
* SYSVOL problems
* Kerberos issues
* Directory inconsistencies
* Misaligned event timelines

Modern virtualization safeguards reduce some risks, but snapshots do not replace Active Directory-aware backups.

---

## DC01 Restore Validation

After restoring DC01:

1. Confirm system time.
2. Confirm the static internal address.
3. Confirm DNS settings.
4. Confirm AD DS services.
5. Run:

```powershell
dcdiag
```

6. Confirm:

```powershell
Get-Service NTDS, DNS, Netlogon, Kdc
```

7. Confirm SYSVOL and NETLOGON:

```powershell
net share
```

8. Resolve the internal domain.
9. Confirm WIN11TARGET secure channel.
10. Review Event Viewer.
11. Confirm Wazuh and Splunk receive new domain events.

---

## Domain Member Restore Risks

Restoring a domain-joined endpoint may create a mismatch between:

* The endpoint’s stored computer-account password
* The password stored by Active Directory

This may break the secure channel.

Symptoms include:

* Domain sign-in failure
* Group Policy failure
* Network showing Public instead of DomainAuthenticated
* Trust relationship error
* SIEM agent confusion after identity changes

---

## WIN11TARGET Restore Validation

After restoring WIN11TARGET:

1. Confirm system time.
2. Confirm host-only IP configuration.
3. Confirm DNS points to DC01.
4. Confirm domain discovery.
5. Test:

```powershell
Test-ComputerSecureChannel -Verbose
```

6. Run:

```powershell
gpupdate /force
```

7. Confirm the Wazuh agent.
8. Confirm the Splunk forwarder.
9. Generate a harmless test event.
10. Confirm ingestion.

If the secure channel is broken, repair it through the documented endpoint recovery process.

---

## Wazuh Snapshot Risks

Restoring Wazuh may cause:

* Loss of recent alerts
* Agent-key mismatch
* Duplicate agent state
* Certificate inconsistency
* Time rollback
* Index corruption
* Stale dashboard state
* Missing custom rules
* Restored credentials

---

## Wazuh Restore Validation

1. Confirm system time.
2. Confirm host-only and NAT interfaces.
3. Confirm the default route.
4. Confirm Docker:

```bash
sudo systemctl status docker
```

5. Move into the deployment directory.
6. Start or review the stack:

```bash
docker compose up -d
docker compose ps
```

7. Review recent logs.
8. Open the dashboard.
9. Confirm agents reconnect.
10. Generate a new endpoint event.
11. Confirm the event appears.
12. Review for duplicates.

---

## Splunk Snapshot Risks

Restoring Splunk may cause:

* Loss of recently indexed data
* Duplicate events
* Forwarder reconnection issues
* Stale credentials
* License inconsistencies
* Index corruption
* Changed server identity
* Missing saved searches
* Ownership problems

---

## Splunk Restore Validation

1. Confirm system time.
2. Confirm the internal address.
3. Confirm disk space.
4. Start Splunk using the documented runtime account.
5. Review status:

```bash
sudo /opt/splunk/bin/splunk status
```

6. Confirm port `8000`.
7. Open Splunk Web.
8. Review internal logs.
9. Confirm indexes.
10. Confirm the forwarder reconnects.
11. Generate a new test event.
12. Compare `_time` and `_indextime`.

---

## Kali Snapshot Risks

Restoring Kali may reintroduce:

* Old target lists
* Outdated tools
* Stale evidence
* Old SSH keys
* Sensitive shell history
* Incorrect network adapters
* Enabled NAT or bridged adapters
* Previously deleted credentials

---

## Kali Restore Validation

1. Confirm the date and time.
2. Confirm the host-only adapter.
3. Confirm NAT state.
4. Confirm no bridged adapter is active.
5. Review the authorized target list.
6. Review shell history.
7. Remove stale evidence.
8. Apply required updates.
9. Confirm Wazuh and Splunk are already running.
10. Perform a harmless connectivity test.

---

## Coordinated Restore Scenarios

Some recovery events affect more than one VM.

Examples include:

* DC01 and WIN11TARGET restored to different dates
* Wazuh restored without restoring agent state
* Splunk restored while the forwarder retains newer configuration
* Network subnet changed after snapshots were created
* Group Policy restored to an earlier state
* Kali restored with an outdated target list

When dependencies exist, document whether related systems must be restored to compatible states.

---

## Recommended Restore Order

For a full lab recovery:

1. Acer Windows host
2. VMware Workstation
3. VMware host-only and NAT networks
4. DC01
5. WIN11TARGET
6. WAZUH-SERVER
7. SPLUNK-SERVER
8. Wazuh agent
9. Splunk forwarder
10. KALI-TEST
11. Exercise data and evidence
12. End-to-end validation

Identity and DNS should be restored before dependent domain systems.

---

## Cold VM Backup

A cold backup is a copy or export of a powered-off virtual machine.

It protects against failures that snapshots cannot address.

A cold backup may include:

* VM configuration
* Virtual disks
* Snapshot metadata when intentionally retained
* NVRAM
* Logs
* Supporting files

---

## Safe Cold Backup Procedure

1. End active exercises.
2. Save required evidence.
3. Stop application services.
4. Shut down the guest operating system.
5. Confirm the VM is powered off.
6. Close VMware Workstation if required.
7. Copy or export the VM directory.
8. Store it on separate media.
9. Record the backup date and milestone.
10. Verify the copied files.
11. Restart the original VM.
12. Validate the system.

Do not copy an actively running VM as the only backup.

---

## Cold Backup Naming Standard

```text
<SYSTEM>-<MILESTONE>-<YYYY-MM-DD>
```

Examples:

```text
DC01-Domain-Validated-YYYY-MM-DD
WIN11TARGET-Exercise-Ready-YYYY-MM-DD
WAZUH-SERVER-Agent-Validated-YYYY-MM-DD
SPLUNK-SERVER-Ingestion-Validated-YYYY-MM-DD
KALI-TEST-Baseline-YYYY-MM-DD
```

---

## Backup Storage

Independent backups should be stored outside the active VM directory.

Possible storage locations include:

* External USB drive
* NAS
* Separate internal drive
* Encrypted backup volume
* Offline archive
* Approved cloud backup for sanitized, encrypted content

Do not place the only backup on the same physical disk as the active VM.

---

## Backup Encryption

Backups may contain:

* Password hashes
* Domain data
* Usernames
* Internal addresses
* Agent keys
* Splunk configuration
* Wazuh certificates
* Event logs
* Investigation evidence

Backup storage should use encryption when practical.

Protect:

* BitLocker recovery keys
* Archive passwords
* Encryption keys
* NAS credentials
* Cloud backup credentials

Do not store the recovery key only inside the encrypted backup.

---

## Backup Verification

A backup is not trustworthy until it has been checked.

Verification may include:

* Confirming the archive opens
* Confirming files are readable
* Comparing file sizes
* Calculating hashes
* Restoring to a test location
* Starting a copied VM
* Running service validation
* Confirming evidence files

---

## Backup Hashing

Windows:

```powershell
Get-FileHash `
    -Path "<BACKUP_FILE>" `
    -Algorithm SHA256
```

Linux:

```bash
sha256sum <BACKUP_FILE>
```

Record:

```text
Backup:
System:
Milestone:
Date:
Storage location:
SHA-256:
Verification:
```

---

## Application Configuration Backups

VM backups protect the whole system, but configuration backups provide faster recovery and version tracking.

Important targets include:

### DC01

* Group Policy backups
* Active Directory system state
* DNS configuration records
* OU and group documentation
* PowerShell administration scripts

### WIN11TARGET

* Agent configuration
* Forwarder configuration
* Sysmon configuration
* Audit policy records
* Firewall configuration
* Test scripts

### Wazuh

* Compose files
* Environment templates
* Custom rules
* Custom decoders
* Manager configuration
* Dashboard exports
* Certificate configuration
* Agent deployment notes

### Splunk

* `/opt/splunk/etc`
* Custom applications
* Saved searches
* Dashboards
* Lookup files
* Index configuration
* Input and output configuration

### Kali

* Exercise scripts
* Authorized-target templates
* Tool configuration
* Evidence notes
* Synthetic wordlists

---

## Active Directory System State Backup

System state provides a more appropriate Active Directory recovery method than snapshots alone.

Install Windows Server Backup:

```powershell
Install-WindowsFeature Windows-Server-Backup
```

Run a system state backup:

```powershell
wbadmin start systemstatebackup `
    -backuptarget:<BACKUP_DESTINATION> `
    -quiet
```

Administrator privileges are required.

The backup destination should not be the same virtual disk as DC01.

---

## Group Policy Backup

Back up all Group Policy Objects:

```powershell
Backup-GPO `
    -All `
    -Path "<GPO_BACKUP_PATH>"
```

Appropriate domain permissions are required.

Store the backup securely because it may reveal internal configuration.

---

## Wazuh Configuration Backup

From the Wazuh host, preserve:

* Compose files
* Custom rules
* Custom decoders
* Configuration templates
* Certificate configuration
* Change records

Example archive:

```bash
tar -czf \
    <BACKUP_PATH>/wazuh-config-<DATE>.tar.gz \
    <WAZUH_CONFIG_PATH>
```

Use `sudo` when protected files require it.

Do not upload secrets or private keys to GitHub.

---

## Splunk Configuration Backup

Stop Splunk before a consistent full configuration backup when practical.

Example:

```bash
sudo tar -czf \
    <BACKUP_PATH>/splunk-config-<DATE>.tar.gz \
    /opt/splunk/etc
```

This archive may contain sensitive authentication and license information.

Store it securely.

---

## Evidence Backup

Exercise evidence should be separated from disposable VM states.

Important evidence may include:

* `.evtx` logs
* Packet captures
* Splunk exports
* Wazuh alerts
* Screenshots
* Investigation notes
* Hash records
* Timelines
* Detection queries
* Change records

Save the evidence before reverting a snapshot.

---

## Evidence Preservation Workflow

1. Identify the evidence.
2. Export it from the source.
3. Copy it to a controlled location.
4. Calculate a SHA-256 hash.
5. Record source and collection time.
6. Preserve the original.
7. Create a sanitized copy for GitHub.
8. Confirm the original remains private.
9. Perform the restore.
10. Confirm the evidence is still available.

---

## Recovery Runbooks

The repository should include focused runbooks for common failures.

Recommended files:

```text
runbooks/
├── start-cyberlab.md
├── stop-cyberlab.md
├── restore-vm-snapshot.md
├── restore-domain-controller.md
├── repair-domain-secure-channel.md
├── recover-wazuh-stack.md
├── recover-splunk-service.md
├── validate-siem-ingestion.md
└── vm-unreachable.md
```

Runbooks should contain direct operational steps rather than long architectural explanations.

---

## Recovery Runbook Template

```text
Title:

Purpose:

Symptoms:

Affected systems:

Prerequisites:

Required access:

Evidence to preserve:

Recovery steps:

Validation:

Rollback:

Escalation conditions:

Outcome:
```

---

## Change Management Before Recovery

Before a restore or repair:

* Record the incident.
* Record the affected system.
* Record the current snapshot.
* Record the intended recovery point.
* Identify dependent systems.
* Preserve evidence.
* Confirm rollback conditions.
* Record the expected loss of recent data.

This prevents recovery work from becoming undocumented configuration drift.

---

## Recovery Record

```text
Date:

Incident:

Affected system:

Symptoms:

Current snapshot:

Selected recovery point:

Evidence preserved:

Recovery method:

Dependent systems:

Validation performed:

Data lost:

Follow-up action:

Outcome:
```

---

## Recovery Testing

A recovery plan should be tested before an actual emergency.

Possible tests include:

* Restore WIN11TARGET snapshot
* Repair secure channel
* Restore Wazuh after a stopped container
* Restore Splunk after a configuration error
* Restore a cold VM copy
* Recreate VMware virtual networks
* Restore Group Policy backup
* Validate DC01 system state procedure
* Recover evidence from external storage

---

## Recovery Test Schedule

### Monthly

* Review snapshot register.
* Validate one known-good snapshot.
* Check external backup availability.
* Confirm free storage.

### Quarterly

* Restore one noncritical VM to a test location.
* Validate application services.
* Confirm configuration backup readability.
* Review runbooks.

### After Major Architecture Changes

* Update restore order.
* Update network values.
* Update dependencies.
* Create new baseline backups.
* Retire obsolete recovery points.

---

## Restore Test Success Criteria

A recovery test succeeds when:

* The selected backup or snapshot is readable.
* The VM starts.
* Networking works.
* Time is correct.
* Required services start.
* Dependencies reconnect.
* Monitoring resumes.
* A new known event is ingested.
* No unresolved duplicate identity exists.
* The result is documented.

---

## Recovery Point Objective

Recovery Point Objective describes how much recent data may be lost.

In this lab, different systems have different priorities.

Examples:

* DC01 configuration changes: preserve major milestones
* WIN11TARGET: disposable exercise state may be lost
* Wazuh alerts: recent data may be lost after snapshot restore
* Splunk indexes: recent indexed events may be lost
* Exercise evidence: should be exported before rollback
* Documentation: should be stored outside the VMs and version-controlled

The lab should define acceptable loss for each system.

---

## Recovery Time Objective

Recovery Time Objective describes how quickly a system should return to service.

For a personal CyberLab, the goal is not production-level availability.

Practical priorities may be:

1. Restore DC01 and DNS.
2. Restore WIN11TARGET domain access.
3. Restore Wazuh.
4. Restore Splunk.
5. Restore Kali.
6. Resume exercises.

The most important objective is reliable recovery, not immediate recovery.

---

## Host Recovery

If the Acer host must be rebuilt:

1. Reinstall or repair Windows.
2. Apply host updates.
3. Reinstall VMware Workstation.
4. Recreate or restore VMware virtual networks.
5. Restore the CyberLab directory.
6. Import or open VM configuration files.
7. Validate DC01 first.
8. Validate dependent VMs.
9. Confirm snapshots.
10. Confirm backups and evidence.

---

## VMware Network Recovery

Record the private operational configuration for:

* Host-only VMnet
* NAT VMnet
* Subnets
* DHCP state
* Host adapter addresses
* NAT gateway
* Adapter assignments

Public documentation should use placeholders.

After recreating the networks:

1. Confirm host adapters.
2. Confirm VM adapter assignments.
3. Start DC01.
4. Confirm static addressing.
5. Start one endpoint.
6. Test internal communication.
7. Test NAT separately.
8. Confirm SIEM connectivity.

---

## Storage Failure Recovery

If the active VM storage fails:

* Do not rely on snapshots stored on the failed drive.
* Restore from an independent cold backup.
* Verify the backup hash when available.
* Copy the VM to healthy storage.
* Open it in VMware.
* Confirm whether the VM was moved or copied when prompted.
* Validate identity-sensitive systems carefully.
* Recreate newer configuration from documented change records.

---

## VM Copy Identity Prompt

VMware may ask whether a restored VM was moved or copied.

This choice can affect:

* Virtual MAC address
* VMware UUID
* Network identity
* Application licensing
* Agent identity

For recovery of the same VM, “moved” may preserve identity.

For a deliberate clone, “copied” may generate a new identity.

Choose based on the recovery objective and document the decision.

---

## Ransomware or Host Compromise

If the Acer host is suspected of compromise:

1. Stop using the host for normal administration.
2. Disconnect it from untrusted networks if necessary.
3. Preserve evidence.
4. Do not trust active VM files automatically.
5. Use known-good external backups.
6. Rebuild the host from trusted media.
7. Change credentials exposed on the host.
8. Restore VMs carefully.
9. Validate hashes and configurations.
10. Review the cause before reconnecting the environment.

A compromised host can affect every VM and backup mounted to it.

---

## Backup Rotation

A simple rotation may retain:

* Current known-good baseline
* Previous known-good milestone
* Most recent pre-change backup
* Periodic archive copy

Avoid retaining every backup indefinitely without a storage plan.

Retire obsolete backups only after:

* New backups are validated.
* Documentation is updated.
* No unresolved dependency remains.
* Important evidence is preserved.

---

## Offsite and Offline Copies

A stronger backup strategy may include:

* One local working copy
* One external backup
* One disconnected or offsite copy

For a personal lab, this can be implemented gradually.

Offline storage reduces exposure to:

* Host compromise
* Ransomware
* Accidental deletion
* Software bugs
* Power-related damage

---

## Backup Inventory

Maintain a backup inventory.

| Backup                | System        | Date       | Type             | Location                | Verified | Retention |
| --------------------- | ------------- | ---------- | ---------------- | ----------------------- | -------- | --------- |
| DC01-Domain-Validated | DC01          | YYYY-MM-DD | Cold VM copy     | External storage        | Yes      | Long-term |
| WAZUH-Agent-Validated | WAZUH-SERVER  | YYYY-MM-DD | VM export        | External storage        | Yes      | Long-term |
| Splunk configuration  | SPLUNK-SERVER | YYYY-MM-DD | Config archive   | Encrypted backup        | Yes      | Current   |
| Exercise evidence     | Multiple      | YYYY-MM-DD | Evidence archive | NAS or external storage | Yes      | Project   |

Do not publish operational storage locations.

---

## Backup Failure Conditions

A backup should not be treated as valid if:

* It cannot be opened.
* It was copied while the VM was active and is inconsistent.
* The hash does not match.
* Required files are missing.
* The storage device reports errors.
* The archive password is unavailable.
* The VM cannot start.
* Critical services fail after restore.
* The backup location is the same failed disk as the source.

---

## Troubleshooting: Snapshot Creation Fails

Possible causes include:

* Insufficient host disk space
* VM lock files
* Existing consolidation issue
* Suspended operation
* Storage permission problem
* Host filesystem error
* Backup software conflict

Actions:

1. Check host free space.
2. Shut down the VM.
3. Close VMware.
4. Reopen VMware.
5. Review VM logs.
6. Check whether consolidation is required.
7. Avoid manually deleting snapshot files.

---

## Troubleshooting: Snapshot Restore Fails

Check:

* Snapshot files exist.
* VM files are readable.
* Host storage is healthy.
* No file is locked.
* The snapshot chain is intact.
* Adequate disk space exists.
* VMware logs show the failure reason.

Do not attempt repeated restoration without preserving the VM directory.

---

## Troubleshooting: VM Starts but Service Fails

The snapshot may restore the operating system but not a healthy application state.

Check:

* System time
* Disk space
* Service account
* File ownership
* Network interfaces
* DNS
* Certificates
* Application logs
* Dependency services
* Database or index health

Validate the application separately from the VM boot.

---

## Troubleshooting: Secure Channel Broken After Restore

On WIN11TARGET:

```powershell
Test-ComputerSecureChannel -Verbose
```

Repair when appropriate:

```powershell
Test-ComputerSecureChannel `
    -Repair `
    -Credential "CYBERLAB\<AUTHORIZED_ADMIN>"
```

If repair fails, a controlled domain leave and rejoin may be required.

---

## Troubleshooting: Wazuh Agent Duplicate After Restore

Possible causes include:

* Restored agent key
* Existing newer agent record
* Duplicate hostname
* Cloned endpoint identity
* Manager rollback

Actions:

1. Identify the intended endpoint.
2. Record the current agent state.
3. Remove only the stale record.
4. Re-enroll the endpoint if required.
5. Confirm one stable active agent.
6. Generate a test event.

---

## Troubleshooting: Splunk Duplicate Events After Restore

Possible causes include:

* Forwarder replay
* Restored checkpoint state
* Re-uploaded files
* Duplicate inputs
* Restored index state
* Snapshot rollback

Review:

```spl
index=<INDEX_NAME>
| stats count by _time host source EventCode _raw
| where count > 1
```

Confirm whether the events are true duplicates before changing inputs.

---

## Troubleshooting: Backup Storage Is Full

Review:

* Old VM exports
* Duplicate snapshots
* Large packet captures
* ISO files
* SIEM indexes
* Configuration archives
* Stale evidence copies

Do not delete the only known-good backup.

Create a retention decision before removing files.

---

## Recovery Validation Checklist

### Snapshot

* Snapshot name is clear.
* Snapshot description exists.
* VM was healthy before creation.
* Snapshot is recorded.
* Host has adequate free space.
* Snapshot chain is manageable.

### Backup

* VM was powered off.
* Backup is stored separately.
* Backup files are readable.
* Hash is recorded where practical.
* Backup date and milestone are documented.
* Restore has been tested.

### Restore

* Evidence was preserved.
* Correct recovery point was selected.
* Time and networking were checked.
* Services were validated.
* Dependencies reconnected.
* New telemetry was generated.
* Outcome was documented.

### Security

* Backups are protected.
* Secrets are not committed to GitHub.
* Recovery keys are stored separately.
* Operational paths are sanitized.
* Evidence remains private.
* Public copies are redacted.

---

## Public Sanitization Standards

Remove or replace:

* Real snapshot names containing internal values
* Operational backup paths
* Personal usernames
* Internal IP addresses
* Real domain names
* Passwords
* Encryption keys
* BitLocker recovery keys
* Agent keys
* Splunk credentials
* Wazuh certificates
* NAS addresses
* External-drive serial numbers
* VMware UUIDs
* MAC addresses
* Raw event logs
* Packet captures
* Internal storage topology

Use placeholders such as:

```text
<VM_NAME>
<SNAPSHOT_NAME>
<BACKUP_PATH>
<EXTERNAL_STORAGE>
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<AUTHORIZED_ADMIN>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing snapshot or backup screenshots:

1. Hide personal Windows paths.
2. Remove internal IP addresses.
3. Remove VM UUIDs.
4. Remove MAC addresses.
5. Remove backup destination names.
6. Remove usernames.
7. Remove volume labels when sensitive.
8. Remove encryption and recovery keys.
9. Crop unrelated VM entries.
10. Permanently redact sensitive values.
11. Reopen the final image.
12. Confirm it supports the documentation objective.

---

## Skills Demonstrated

This recovery strategy demonstrates:

* VMware snapshot management
* Virtual machine backup
* Recovery planning
* Active Directory recovery considerations
* Domain secure-channel repair
* Wazuh recovery
* Splunk recovery
* Evidence preservation
* Backup verification
* Configuration backup
* Restore validation
* Change management
* Disaster-recovery planning
* Storage management
* Security sanitization
* Technical runbook development

---

## Lessons Learned

Key lessons include:

* Snapshots are not independent backups.
* Powered-off snapshots provide cleaner stable milestones.
* A snapshot should be validated before it is trusted.
* Evidence must be exported before rollback.
* Domain controllers require additional recovery planning.
* Restoring a domain member can break the secure channel.
* Wazuh restoration can affect agent identity and recent alerts.
* Splunk restoration can cause duplicate or missing events.
* Long snapshot chains increase risk and storage use.
* Cold VM copies protect against host-disk failure.
* Configuration backups allow faster targeted recovery.
* A backup is not reliable until restoration has been tested.
* Recovery order matters because identity and DNS support other systems.
* Public recovery documentation requires careful storage and credential sanitization.

---

## Planned Improvements

Future recovery improvements may include:

* Dedicated external backup drive
* NAS-based VM backup
* Automated configuration archives
* Scheduled backup verification
* Automated snapshot inventory
* Automated free-space alerts
* System state backup schedule for DC01
* Splunk configuration backup script
* Wazuh configuration backup script
* Encrypted evidence archive
* Quarterly restore testing
* UPS support
* Offline backup rotation
* Recovery dashboard
* Documented host rebuild procedure
* VMware network configuration export
* Automated post-restore validation
* Backup integrity reporting
* Formal recovery-point and recovery-time targets

---

## Summary

The Acer Blue Team CyberLab recovery strategy combines VMware snapshots, powered-off VM backups, configuration archives, evidence preservation, and documented validation procedures.

A dependable recovery process requires:

* Clear snapshot milestones
* Independent backups
* Application-aware shutdown
* Evidence preservation
* Domain-aware restoration
* Monitoring-platform validation
* Secure backup storage
* Restore testing
* Change records
* Public sanitization

By treating recovery as a tested operational capability rather than an emergency-only task, the CyberLab becomes safer to modify, easier to maintain, and more useful for long-term defensive security training.

