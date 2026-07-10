# File Integrity Change Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating unexpected or security-relevant file-system changes on Windows systems.

It applies to alerts involving:

* File creation
* File modification
* File deletion
* File renaming
* Permission or ownership changes
* Hash changes
* Changes to monitored configuration files
* Changes to startup locations
* Changes to scripts or executables
* Wazuh File Integrity Monitoring alerts
* Sysmon file events
* Splunk detections involving monitored files
* Microsoft Defender alerts associated with changed files

The runbook helps the analyst determine:

* Which file changed
* What type of change occurred
* When the change occurred
* Which user and process performed the action
* Whether the file is sensitive
* Whether the new content is trusted
* Whether the change was authorized
* Whether the file was executed
* Whether persistence, defense evasion, or application compromise is involved
* Whether containment or escalation is required

---

## Runbook ID

```
RB-06
```

---

## Related Exercise

```
EX-06 – File Integrity Change
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Windows administrators
* Detection engineers
* Incident responders
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Windows operating-system files
* Application configuration files
* Security-tool configuration
* Scripts
* Executables
* Startup folders
* Scheduled-task files
* Web content
* Service configuration
* Monitored CyberLab test directories
* Sensitive user or administrative files

This runbook does not authorize:

* Opening or executing an unknown file
* Restoring a file before evidence is preserved
* Deleting suspicious files before hashing and collection
* Disabling FIM to suppress alerts
* Uploading sensitive files to public scanning services
* Modifying systems outside the authorized CyberLab
* Assuming every file change is malicious

---

## Primary Data Sources

Use the available sources in this order:

1. Wazuh File Integrity Monitoring event
2. Local file metadata
3. Sysmon file events
4. Windows process-creation telemetry
5. PowerShell Operational logs
6. Microsoft Defender events
7. Windows Security and System logs
8. Splunk indexed telemetry
9. Application or service logs
10. Change-management records
11. Backup and version history
12. User or administrator confirmation

---

## Key Event Sources

| Event ID | Source     | Description                             |
| -------: | ---------- | --------------------------------------- |
|        1 | Sysmon     | Process creation                        |
|        2 | Sysmon     | File creation time changed              |
|       11 | Sysmon     | File created                            |
|       15 | Sysmon     | FileCreateStreamHash                    |
|       23 | Sysmon     | File deletion archived                  |
|       26 | Sysmon     | File deletion detected                  |
|     4663 | Security   | An attempt was made to access an object |
|     4670 | Security   | Permissions on an object were changed   |
|     4688 | Security   | A new process was created               |
|     4103 | PowerShell | Module logging                          |
|     4104 | PowerShell | Script block logging                    |
|     1116 | Defender   | Threat detected                         |
|     1117 | Defender   | Remediation action taken                |
|     5007 | Defender   | Configuration changed                   |

Wazuh FIM events use Wazuh rule and `syscheck` fields rather than a single Windows event ID.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh FIM alert
* Splunk correlation search
* Sysmon file event
* Microsoft Defender alert
* Application outage
* Configuration review
* Administrator report
* User report
* Scheduled integrity scan
* CyberLab exercise

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Reporting host:
File path:
Filename:
Change type:
Previous hash:
Current hash:
Previous size:
Current size:
User:
Process:
Alert severity:
Analyst:
```

Do not assume that the alert contains complete process or user attribution.

---

## Initial Analyst Questions

Determine immediately:

* Is the file sensitive?
* Is the change still occurring?
* Was the file created, modified, renamed, or deleted?
* Is the user authorized to change it?
* Is the responsible process known?
* Is the file signed?
* Did Microsoft Defender detect the file?
* Was the file executed?
* Does the change create persistence?
* Does the change weaken security controls?
* Is there an approved deployment or maintenance record?
* Are similar changes occurring on other systems?

---

# Severity Classification

## Informational

Use informational severity when:

* The change is part of an approved CyberLab exercise
* The file is inside a dedicated test directory
* The user and process are expected
* No sensitive configuration is affected
* Cleanup is completed

---

## Low

Use low severity when:

* A known application changes a noncritical file
* The user is authorized
* The change matches a documented update
* The process and path are expected
* No suspicious follow-on activity exists

---

## Medium

Use medium severity when:

* The change is not documented
* The file is in a user-writable or temporary directory
* Process attribution is incomplete
* The file is a script or executable
* The hash is unknown
* The modification occurs outside expected hours
* The same file changes repeatedly

---

## High

Use high severity when:

* A startup or persistence location changes
* An executable or script is placed in a sensitive directory
* A security configuration file changes unexpectedly
* Permissions are weakened
* The file is unsigned or suspicious
* An unusual process performed the change
* The file is executed immediately after creation
* Defender detects associated malicious content
* Several endpoints receive the same file

---

## Critical

Use critical severity when:

* A confirmed malicious file is executed
* Security tools are disabled or tampered with
* Domain-wide scripts or policies are modified
* Ransomware or destructive file activity occurs
* Critical system binaries are replaced
* Privileged persistence is established
* The change is part of confirmed widespread compromise

---

# Initial Triage

## Step 1: Confirm the Alert Time

Record the exact event time and timezone from:

* Endpoint
* Wazuh
* Splunk
* Sysmon
* Microsoft Defender
* Analyst workstation

Use absolute timestamps.

```
YYYY-MM-DD HH:MM:SS Timezone
```

---

## Step 2: Confirm the File State

On the affected endpoint:

```
Test-Path -LiteralPath "<FILE_PATH>"
```

When the file exists:

```
Get-Item -LiteralPath "<FILE_PATH>" -Force |
    Select-Object `
        FullName,
        Length,
        CreationTime,
        LastWriteTime,
        LastAccessTime,
        Attributes
```

Do not open or execute an unknown file.

---

## Step 3: Record File Metadata

Record:

```
Full path:
Filename:
Extension:
File exists:
Size:
Creation time:
Modification time:
Last access time:
Attributes:
Owner:
Previous hash:
Current hash:
Digital signature:
```

---

## Step 4: Hash the Current File

```
Get-FileHash `
    -LiteralPath "<FILE_PATH>" `
    -Algorithm SHA256
```

When possible, compare:

* Current hash
* Previous FIM hash
* Trusted baseline hash
* Software-vendor hash
* Backup copy hash

A hash difference proves the content changed but does not determine intent.

---

## Step 5: Review File Ownership

```
Get-Acl -LiteralPath "<FILE_PATH>" |
    Select-Object Owner, AccessToString
```

Store detailed access-control information privately.

---

## Step 6: Preserve the Current File

When the file may be suspicious, copy it to a restricted evidence location without executing it:

```
Copy-Item `
    -LiteralPath "<FILE_PATH>" `
    -Destination "<EVIDENCE_PATH>\preserved-file" `
    -Force
```

Do not use the normal Git repository, shared drive, or public cloud folder as an evidence location.

---

# Wazuh FIM Investigation

## Review the FIM Event

Record the available `syscheck` fields:

```
Agent:
Rule ID:
Rule level:
Rule description:
Event type:
File path:
File size before:
File size after:
SHA-256 before:
SHA-256 after:
MD5 before:
MD5 after:
Permissions before:
Permissions after:
Owner before:
Owner after:
User:
Process:
Timestamp:
```

Field availability depends on the Wazuh version and whether `whodata` is enabled.

---

## Common FIM Change Types

Possible actions include:

```
added
modified
deleted
```

A rename may appear as:

* One deletion event
* One addition event
* Related events close together in time

Correlate filenames, hashes, timestamps, user, and process.

---

## Confirm the Monitored Directory

Review the Wazuh agent configuration privately.

Typical Windows configuration file:

```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Confirm:

* The path is monitored
* The intended recursion level applies
* Real-time monitoring is enabled where expected
* `whodata` is enabled where supported
* Exclusions do not hide the file
* The configuration is valid

Do not publish operational monitoring exclusions.

---

## Wazuh Search by Path

Search for:

```
<FILE_PATH>
```

Also search by:

```
<FILENAME>
<SHA256_HASH>
```

Use a time range beginning before the first change.

---

## Review Related Wazuh Events

Search for:

* Process creation
* PowerShell execution
* Defender detection
* User logon
* Scheduled-task creation
* Service installation
* Registry changes
* Firewall activity
* Additional file changes

---

## Wazuh Investigation Questions

* Was the file added, modified, or deleted?
* Was the previous hash available?
* Was the current hash available?
* Was the user identified?
* Was the process identified?
* Were permissions or ownership changed?
* Did the event occur in real time?
* Were repeated changes grouped or duplicated?
* Was the rule severity appropriate?
* Were related process or Defender events visible?

---

# Local File Analysis

## Review the File Type

Do not trust the extension alone.

Review basic metadata:

```
Get-Item -LiteralPath "<FILE_PATH>" |
    Select-Object Name, Extension, Length, Attributes
```

For scripts and text configuration files, inspect content only with a text viewer.

Do not load, import, dot-source, or execute the file.

---

## Review the Digital Signature

For supported file types:

```
Get-AuthenticodeSignature `
    -FilePath "<FILE_PATH>"
```

Record:

```
Signature status:
Signer:
Certificate subject:
Certificate issuer:
Timestamp:
```

An unsigned file is not automatically malicious.

A valid signature does not guarantee that the change was authorized.

---

## Review Alternate Data Streams

```
Get-Item `
    -LiteralPath "<FILE_PATH>" `
    -Stream * `
    -ErrorAction SilentlyContinue
```

A `Zone.Identifier` stream may indicate that the file originated from another security zone.

---

## Review File Content Safely

For a text-based file:

```
Get-Content `
    -LiteralPath "<FILE_PATH>" `
    -Raw
```

Do not use this command for large, binary, or potentially dangerous files.

Review suspicious content in an isolated analysis environment.

---

## Compare with a Trusted Copy

For text files:

```
Compare-Object `
    -ReferenceObject (Get-Content "<TRUSTED_COPY>") `
    -DifferenceObject (Get-Content "<FILE_PATH>")
```

For binary files, compare hashes and metadata rather than using text comparison.

---

# Process Attribution

## Review Sysmon File Creation

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 11
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILENAME>")
}
```

Review:

* Image
* Process ID
* Process GUID
* Target filename
* User
* Creation time

---

## Review Sysmon File Deletion

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 23, 26
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILENAME>")
}
```

Event ID `23` may archive deleted content when configured.

---

## Review Process Creation

### Sysmon

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<PROCESS_NAME>|<FILE_PATH>"
}
```

### Windows Security

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<PROCESS_NAME>|<FILE_PATH>"
}
```

---

## Record Process Context

```
Process image:
Process command line:
Process ID:
Process GUID:
Parent image:
Parent command line:
User:
Integrity level:
Hash:
Signature:
Start time:
```

---

## Common Legitimate Processes

Possible expected processes include:

* Approved software installer
* Application updater
* Text editor
* Configuration-management agent
* Backup client
* Security product
* PowerShell
* Command Prompt
* Deployment tool
* Authorized administrative utility

A legitimate executable can still make an unauthorized change.

---

## High-Risk Processes

Increase concern when the change is performed by:

* Office application
* Browser
* Script host
* `mshta.exe`
* `rundll32.exe`
* `regsvr32.exe`
* Unknown executable
* Unsigned executable in a user-writable path
* Unexpected SYSTEM process
* PowerShell with encoded or hidden commands

---

# PowerShell Investigation

## Search Script Block Logging

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<FILENAME>|<FILE_PATH>"
}
```

Review commands such as:

* `Set-Content`
* `Add-Content`
* `Out-File`
* `New-Item`
* `Copy-Item`
* `Move-Item`
* `Remove-Item`
* `Set-Acl`
* `Invoke-WebRequest`
* `Start-BitsTransfer`
* Archive extraction commands

Transition to the suspicious PowerShell runbook when the execution itself is suspicious.

---

# Windows Object Access Auditing

## Review Event ID 4663

When object-access auditing is configured:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4663
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILE_PATH>")
}
```

Review:

* Subject user
* Process name
* Object name
* Access mask
* Accesses
* Handle ID
* Timestamp

This event depends on both audit policy and the file’s audit ACL.

---

## Review Permission Changes

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4670
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILE_PATH>")
}
```

Unexpected permission weakening can be more significant than a simple content change.

---

# Microsoft Defender Investigation

## Review Defender Events

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Windows Defender/Operational"
        Id = 1116, 1117, 5007
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<FILENAME>|<FILE_PATH>"
}
```

---

## Review Defender Threat History

```
Get-MpThreatDetection |
    Select-Object `
        InitialDetectionTime,
        LastThreatStatusChangeTime,
        ThreatID,
        ActionSuccess,
        Resources
```

Do not restore a suspicious file from quarantine.

---

## Review Defender Health

```
Get-MpComputerStatus |
    Select-Object `
        AntivirusEnabled,
        BehaviorMonitorEnabled,
        OnAccessProtectionEnabled,
        RealTimeProtectionEnabled,
        AntivirusSignatureVersion,
        AntivirusSignatureLastUpdated
```

---

# File Execution Investigation

## Determine Whether the File Executed

Search:

* Sysmon Event ID `1`
* Windows Event ID `4688`
* PowerShell logs
* Application logs
* Prefetch where appropriate
* Defender events
* Child-process activity
* Network connections

Sysmon example:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILE_PATH>")
}
```

---

## Review Follow-On Activity

When execution occurred, review:

* Child processes
* Network connections
* DNS queries
* Registry changes
* Scheduled tasks
* Services
* Account changes
* Group changes
* Additional files
* Defender alerts

Execution significantly increases severity.

---

# Persistence Investigation

## Startup Folders

Review:

```
Get-ChildItem `
    -Path `
        "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup",
        "$env:ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp" `
    -Force
```

---

## Scheduled Tasks

```
Get-ScheduledTask |
    Where-Object {
        $_.Actions.Execute -match "<FILENAME>"
        -or $_.Actions.Arguments -match "<FILE_PATH>"
    }
```

Search Event ID `4698`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4698
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<FILENAME>|<FILE_PATH>"
}
```

---

## Services

```
Get-CimInstance Win32_Service |
    Where-Object {
        $_.PathName -match "<FILENAME>|<FILE_PATH>"
    } |
Select-Object `
    Name,
    DisplayName,
    State,
    StartMode,
    StartName,
    PathName
```

Search Event ID `7045`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "System"
        Id = 7045
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<FILENAME>|<FILE_PATH>"
}
```

---

## Registry Autoruns

Review relevant Sysmon Event ID `13` events:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 13
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<FILENAME>|<FILE_PATH>|Run|RunOnce"
}
```

---

# Sensitive File Categories

## Operating-System Files

Increase severity when changes affect:

* Windows system binaries
* Drivers
* Authentication components
* Security providers
* Boot configuration
* System libraries

Do not replace or restore system files manually without a validated recovery process.

---

## Security Configuration

High-value files include:

* Wazuh configuration
* Splunk forwarder configuration
* Microsoft Defender-related configuration
* Firewall configuration exports
* Sysmon configuration
* Audit-policy scripts
* Security-monitoring scripts

A monitoring configuration change may create a blind spot.

---

## Scripts and Automation

Review:

* PowerShell scripts
* Batch files
* Python scripts
* Login scripts
* Deployment scripts
* Backup scripts
* Administrative automation

Determine whether the modification changes behavior, credentials, destinations, or privilege.

---

## Web and Application Files

Unexpected changes may indicate:

* Unauthorized deployment
* Web shell
* Application compromise
* Defacement
* Supply-chain modification
* Misconfigured update

Review application logs and service activity.

---

## Credential and Key Material

Treat changes involving the following as sensitive:

* Private keys
* Certificates
* Password databases
* Credential exports
* Configuration files containing secrets
* Tokens
* SSH authorized keys

Do not publish their contents.

---

# Wazuh Investigation Searches

## Search by File Path

```
<FILE_PATH>
```

## Search by Hash

```
<SHA256_HASH>
```

## Search by User

```
<USER_ACCOUNT>
```

## Search by Process

```
<PROCESS_NAME>
```

## Search Related Activity

Review the same host and time window for:

```
PowerShell
Process creation
Defender detection
Scheduled task creation
Service installation
Registry modification
Authentication
Privilege changes
```

---

# Splunk Investigation

## Basic File-Change Search

```
index=<WAZUH_INDEX> earliest=-24h
| search
    syscheck.path="<FILE_PATH>"
    OR syscheck.path="*<FILENAME>*"
| table
    _time
    agent.name
    rule.id
    rule.level
    rule.description
    syscheck.event
    syscheck.path
    syscheck.size_before
    syscheck.size_after
    syscheck.sha256_before
    syscheck.sha256_after
    syscheck.audit.user.name
    syscheck.audit.process.name
| sort _time
```

Adjust fields for the actual Wazuh-to-Splunk ingestion design.

---

## Sysmon File Creation Search

```
index=windows EventCode=11 earliest=-24h
| search TargetFilename="*<FILENAME>*"
| table
    _time
    host
    User
    Image
    ProcessId
    ProcessGuid
    TargetFilename
| sort _time
```

---

## File Deletion Search

```
index=windows earliest=-24h
(
    EventCode=23 OR
    EventCode=26
)
| search TargetFilename="*<FILENAME>*"
| table
    _time
    host
    EventCode
    User
    Image
    ProcessId
    ProcessGuid
    TargetFilename
    Hashes
| sort _time
```

---

## Permission Change Search

```
index=windows EventCode=4670 earliest=-24h
| search ObjectName="*<FILENAME>*"
| table
    _time
    host
    SubjectUserName
    ProcessName
    ObjectName
    OldSd
    NewSd
    Message
| sort _time
```

---

## Correlate File and Process Events

```
index=windows earliest=-24h
(
    EventCode=1 OR
    EventCode=11 OR
    EventCode=23 OR
    EventCode=26 OR
    EventCode=4688
)
| search
    "*<FILENAME>*"
    OR "*<PROCESS_NAME>*"
| eval activity=case(
    EventCode=1 OR EventCode=4688, "Process creation",
    EventCode=11, "File creation",
    EventCode=23 OR EventCode=26, "File deletion",
    true(), "Related activity"
)
| table
    _time
    host
    EventCode
    activity
    User
    ParentImage
    Image
    CommandLine
    TargetFilename
    Hashes
| sort _time
```

---

## Correlate File Change and Defender Detection

```
index=windows earliest=-24h
(
    EventCode=11 OR
    EventCode=23 OR
    EventCode=26 OR
    EventCode=1116 OR
    EventCode=1117
)
| search "*<FILENAME>*"
| eval activity=case(
    EventCode=11, "File created",
    EventCode=23 OR EventCode=26, "File deleted",
    EventCode=1116, "Defender detection",
    EventCode=1117, "Defender remediation",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    User
    Image
    TargetFilename
    ThreatName
    ActionName
    Message
| sort _time
```

---

## Detect Changes in Sensitive Directories

```
index=<WAZUH_INDEX> earliest=-24h
| eval file_path=lower(syscheck.path)
| where like(file_path, "c:\\windows\\system32\\%")
    OR like(file_path, "c:\\programdata\\microsoft\\windows\\start menu\\programs\\startup\\%")
    OR like(file_path, "%\\appdata\\roaming\\microsoft\\windows\\start menu\\programs\\startup\\%")
| table
    _time
    agent.name
    syscheck.event
    syscheck.path
    syscheck.sha256_before
    syscheck.sha256_after
    syscheck.audit.user.name
    syscheck.audit.process.name
    rule.level
| sort - _time
```

---

## Detect Repeated Changes to One File

```
index=<WAZUH_INDEX> earliest=-1h
| stats
    count as change_count
    values(syscheck.event) as change_types
    values(syscheck.sha256_after) as observed_hashes
    values(syscheck.audit.user.name) as users
    values(syscheck.audit.process.name) as processes
    min(_time) as first_seen
    max(_time) as last_seen
    by agent.name syscheck.path
| where change_count >= <CHANGE_THRESHOLD>
| convert ctime(first_seen) ctime(last_seen)
| sort - change_count
```

---

## Detect File Creation Followed by Execution

```
index=windows earliest=-24h
(
    EventCode=1 OR
    EventCode=11
)
| eval file_path=coalesce(TargetFilename, Image)
| stats
    min(eval(if(EventCode=11, _time, null()))) as creation_time
    min(eval(if(EventCode=1, _time, null()))) as execution_time
    values(User) as users
    values(ParentImage) as parent_images
    values(CommandLine) as command_lines
    by host file_path
| where isnotnull(creation_time)
    AND isnotnull(execution_time)
    AND execution_time >= creation_time
| eval seconds_to_execution=execution_time-creation_time
| where seconds_to_execution <= <TIME_THRESHOLD_SECONDS>
| convert ctime(creation_time) ctime(execution_time)
```

Field normalization may be required before production use.

---

# Investigation Decision Tree

```
File Integrity Alert
        |
        v
Does the file exist?
        |
   +----+----+
   |         |
  No        Yes
   |         |
Review      Preserve metadata,
deletion,   hash and safe copy
rename or         |
remediation       v
          Is the path sensitive?
                 |
            +----+----+
            |         |
           Yes        No
            |         |
      Increase      Identify user
      severity      and process
            |         |
            +----+----+
                 |
                 v
        Is the change authorized?
                 |
            +----+----+
            |         |
           Yes        No
            |         |
    Validate change   Did the file execute,
    record and final  create persistence or
    hash              trigger Defender?
                           |
                      +----+----+
                      |         |
                     Yes        No
                      |         |
               Contain and     Assess scope,
               escalate        restore safely
```

---

# Common Investigation Scenarios

## Scenario 1: Approved Configuration Change

Indicators:

* Approved change record
* Authorized administrator
* Expected management tool
* Known maintenance window
* Trusted final hash or content
* Application remains healthy

Response:

* Validate the final file state
* Preserve evidence
* Close as authorized activity

---

## Scenario 2: Approved CyberLab Test File

Indicators:

* Dedicated test directory
* Synthetic filename
* Exercise record exists
* Standard test account
* Harmless content
* File removed during cleanup

Response:

* Confirm telemetry and cleanup
* Classify as authorized test activity

---

## Scenario 3: Application Update

Indicators:

* Signed vendor installer
* Expected application directory
* Multiple related files updated
* Version change matches release
* No suspicious parent or network activity

Response:

* Confirm update authorization
* Verify signature and version
* Document as expected software maintenance

---

## Scenario 4: Suspicious File in Startup Folder

Indicators:

* New script or executable
* User-writable source
* Unknown hash
* Unusual process created it
* File executes at sign-in

Response:

* Treat as high severity
* Preserve the file
* Isolate the endpoint where appropriate
* Review authentication, process, and network activity
* Remove persistence after evidence collection

---

## Scenario 5: Security Configuration Modified

Indicators:

* Wazuh, Splunk, Defender, firewall, or Sysmon configuration changed
* Monitoring becomes incomplete
* Actor is unknown
* Change follows suspicious PowerShell

Response:

* Treat as high severity
* Preserve old and new configuration
* Restore the approved baseline
* Investigate the actor and source
* Validate monitoring recovery

---

## Scenario 6: File Created and Executed Quickly

Indicators:

* Sysmon Event ID `11`
* Sysmon Event ID `1` follows
* Short creation-to-execution delay
* Unknown parent process
* Network activity follows

Response:

* Escalate
* Preserve file and process evidence
* Isolate endpoint
* Review child processes, persistence, and Defender results

---

## Scenario 7: File Deleted After Execution

Indicators:

* Process event followed by deletion
* Sysmon Event ID `23` or `26`
* No authorized cleanup record
* Additional artifacts remain

Response:

* Treat deletion as possible anti-forensics
* Preserve archived deletion evidence where available
* Review process tree and surrounding activity
* Escalate based on execution behavior

---

## Scenario 8: Repeated File Modification

Indicators:

* Same file changes on a regular interval
* Known service or agent process
* Application rewrites configuration or state
* High alert volume

Response:

* Confirm expected behavior
* Tune monitoring narrowly when justified
* Do not exclude the entire directory without risk review

---

# Containment

## When Containment Is Not Required

Containment may not be required when:

* The change is clearly authorized
* The file and process are trusted
* The final state matches the approved baseline
* No execution, persistence, or security impact exists
* The event is part of an approved exercise

Document why containment was unnecessary.

---

## File Containment

After evidence preservation:

* Quarantine the file through Defender where applicable
* Rename or move it to a restricted location
* Remove execute permission where appropriate
* Block its hash through an approved security control
* Prevent the responsible application from recreating it

Do not improvise quarantine procedures for critical system files.

---

## Process Containment

When harmful activity is ongoing:

```
Stop-Process `
    -Id <PROCESS_ID> `
    -Force
```

Terminate a process only after considering:

* Volatile evidence
* Service impact
* Child processes
* File locks
* Operational consequences

---

## Endpoint Containment

Options include:

* Disconnect the VM’s network adapter
* Isolate the endpoint through approved controls
* End unauthorized sessions
* Block external connectivity
* Stop the responsible task or service

Do not power off the system before considering volatile evidence.

---

## Account Containment

With appropriate authorization:

* Disable or restrict the responsible account
* Reset credentials
* Revoke sessions
* Remove unauthorized privilege
* Restrict remote access

---

# Eradication

## Remove Unauthorized File

Before removal:

* Preserve a copy
* Hash it
* Record metadata
* Record process attribution
* Review persistence
* Review related files

Then remove it through an approved response process.

---

## Restore Trusted File

Restore only from:

* Verified backup
* Trusted installation media
* Known-good package
* Version-controlled approved source
* Vendor-provided repair mechanism

Verify the restored hash and permissions.

---

## Remove Persistence

Depending on findings:

* Remove unauthorized scheduled task
* Remove unauthorized service
* Remove startup-folder entry
* Remove registry autorun
* Restore PowerShell profile
* Remove malicious shortcut
* Correct Group Policy or deployment configuration

---

## Correct Permissions

When permissions were changed:

```
Get-Acl -LiteralPath "<FILE_PATH>"
```

Restore permissions from a known-good reference or approved baseline.

Do not guess access-control entries on system files.

---

## Address Compromised Accounts

* Disable or restrict account
* Reset credentials
* Revoke sessions
* Review privilege changes
* Review successful logons
* Review other file changes
* Restore access only after scope is understood

---

# Recovery

## Validate the Restored File

```
Get-Item -LiteralPath "<FILE_PATH>" -Force |
    Select-Object `
        FullName,
        Length,
        CreationTime,
        LastWriteTime,
        Attributes
```

```
Get-FileHash `
    -LiteralPath "<FILE_PATH>" `
    -Algorithm SHA256
```

```
Get-Acl -LiteralPath "<FILE_PATH>" |
    Select-Object Owner, AccessToString
```

---

## Validate Application or Service Health

Confirm:

* Application starts correctly
* Service runs
* Configuration loads
* No unauthorized task recreates the file
* No repeated FIM alert occurs
* Monitoring remains active

---

## Confirm Defender Health

```
Get-MpComputerStatus |
    Select-Object `
        AntivirusEnabled,
        BehaviorMonitorEnabled,
        OnAccessProtectionEnabled,
        RealTimeProtectionEnabled,
        AntivirusSignatureVersion
```

---

## Confirm Wazuh FIM Health

Generate an approved harmless marker in a dedicated test directory or wait for the next scheduled scan.

Confirm:

* FIM event is generated
* User and process attribution are available where configured
* Wazuh agent is active
* Monitoring path remains configured
* No unauthorized exclusion was added

Do not retest by modifying the sensitive production-like file.

---

## Confirm Splunk Visibility

Search for the final validation event and confirm:

* Correct host
* Correct source
* Correct timestamp
* Expected fields
* Acceptable ingestion delay

---

# Escalation Criteria

Escalate immediately when:

* A critical system file changes unexpectedly
* A security configuration is modified
* The file is executed
* Persistence is created
* Microsoft Defender detects malicious content
* Permissions are weakened on a sensitive file
* A privileged or unknown account made the change
* Several systems receive the same suspicious file
* The change is followed by credential access or lateral movement
* File deletion suggests anti-forensics
* Ransomware or destructive behavior is present
* The analyst cannot determine scope

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
First observed:
Last observed:
Host:
File path:
Change type:
Previous hash:
Current hash:
File size:
Owner:
Permissions:
Responsible user:
Responsible process:
Parent process:
File executed:
Child processes:
Network activity:
Persistence:
Defender result:
Additional affected files:
Containment completed:
Evidence locations:
Outstanding questions:
Recommended next action:
```

---

# Evidence Collection

## Minimum Evidence

Preserve:

* Original alert
* Wazuh FIM event
* File metadata
* Previous and current hashes
* File ownership and permissions
* Preserved file copy
* Sysmon file event
* Process-creation event
* PowerShell event where relevant
* Defender events
* Related persistence evidence
* Splunk timeline
* Change record
* Containment and recovery actions
* Analyst notes

---

## Export Sysmon Events

```
wevtutil epl `
    "Microsoft-Windows-Sysmon/Operational" `
    "<EVIDENCE_PATH>\Sysmon-Operational.evtx"
```

---

## Export Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

---

## Export PowerShell Events

```
wevtutil epl `
    "Microsoft-Windows-PowerShell/Operational" `
    "<EVIDENCE_PATH>\PowerShell-Operational.evtx"
```

Raw logs may contain unrelated sensitive data.

Store them privately.

---

## Export File Metadata

```
Get-Item -LiteralPath "<FILE_PATH>" -Force |
    Select-Object * |
    Export-Clixml `
        -Path "<EVIDENCE_PATH>\file-metadata.xml"
```

---

## Export File Permissions

```
Get-Acl -LiteralPath "<FILE_PATH>" |
    Export-Clixml `
        -Path "<EVIDENCE_PATH>\file-acl.xml"
```

---

## Hash Evidence

Windows:

```
Get-FileHash `
    -LiteralPath "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

Linux:

```
sha256sum <EVIDENCE_FILE>
```

---

## Evidence Record

```
Runbook: RB-06
Incident or case:
Evidence file:
Source:
Collection time:
Description:
SHA-256:
Analyst:
Storage location:
```

---

# False Positives and Benign Causes

Common legitimate causes include:

* Software update
* Application configuration change
* Operating-system update
* Log rotation
* Temporary file generation
* Backup or restore operation
* Security-product update
* User document edit
* Administrative script
* Deployment automation
* Scheduled maintenance
* Approved CyberLab exercise

A legitimate change should still be reviewed for:

* Authorization
* Process ownership
* Final hash
* Permissions
* Application health
* Monitoring impact

---

# Detection Gaps

Document whether the investigation lacked:

* Previous hash
* Current hash
* Change type
* File owner
* Permissions
* User attribution
* Process attribution
* Parent process
* Command line
* File signature
* File origin
* Execution evidence
* Network activity
* Persistence context
* Defender context
* Change-management context
* Trusted-baseline comparison
* Real-time monitoring

---

# Detection Improvement Recommendations

Potential improvements include:

* Enable Wazuh real-time FIM on high-value paths
* Enable `whodata` where supported
* Define sensitive-file categories
* Maintain trusted file hashes
* Enable Sysmon file creation and deletion telemetry
* Enable process command-line logging
* Correlate file creation with execution
* Correlate file changes with PowerShell
* Correlate file changes with Defender alerts
* Alert on permission changes
* Alert on startup-folder changes
* Alert on security-tool configuration changes
* Detect repeated modification of one file
* Track changes across multiple hosts
* Maintain approved updater and installer context
* Create a file-integrity dashboard

---

# MITRE ATT&CK Context

File changes may relate to several ATT&CK techniques depending on observed behavior.

Potential examples include:

* T1547 — Boot or Logon Autostart Execution
* T1053.005 — Scheduled Task
* T1543.003 — Windows Service
* T1562.001 — Impair Defenses
* T1070 — Indicator Removal
* T1036 — Masquerading
* T1105 — Ingress Tool Transfer

A file change alone should not be mapped to a technique without supporting behavior.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Host:
File path:
Change type:
Previous hash:
Current hash:
Previous size:
Current size:
Owner:
Permission change:
Responsible user:
Responsible process:
Parent process:
Command line:
File signature:
File origin:
File executed:
Child processes:
Network activity:
Persistence:
Defender result:
Approved change:
Root cause:
Severity:
Containment:
Eradication:
Recovery:
Detection gaps:
```

---

## Root Cause Categories

Select one:

```
Approved application update
Approved configuration change
Approved administrative action
Approved CyberLab exercise
Operating-system update
Security-product update
User file modification
Misconfigured application
Faulty automation
Unauthorized file creation
Unauthorized file modification
Unauthorized permission change
Malicious file execution
Persistence attempt
Defense evasion
Indicator removal
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The original alert is confirmed.
* The affected file is identified.
* The change type is understood.
* Previous and current hashes are recorded where available.
* Ownership and permissions are reviewed.
* The responsible user and process are identified or documented as unknown.
* The file is analyzed safely.
* Execution is confirmed or excluded.
* Related process, network, and persistence activity is reviewed.
* Defender results are reviewed.
* Authorization is confirmed or disproved.
* Required containment is completed.
* Unauthorized files or changes are removed.
* Trusted content and permissions are restored.
* Application and monitoring health are validated.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The file integrity change was investigated using Wazuh FIM, local file metadata, Sysmon, Windows, PowerShell, Defender, and Splunk telemetry. The file path, change type, hashes, ownership, permissions, responsible user, process activity, execution, network behavior, and persistence indicators were reviewed. The activity was classified as <CLASSIFICATION>. Required containment, eradication, restoration, and validation actions were completed, monitoring confirmed the intended file and endpoint state, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Real usernames
* Administrative identities
* Internal IP addresses
* Domain names
* Hostnames where sensitive
* Full sensitive file paths
* Confidential file contents
* Credentials
* Tokens
* Private keys
* SIDs
* Process GUIDs
* Internal URLs
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths
* Raw unrelated events

Use placeholders such as:

```
<WINDOWS_ENDPOINT>
<FILE_PATH>
<FILENAME>
<PREVIOUS_HASH>
<CURRENT_HASH>
<USER_ACCOUNT>
<PROCESS_NAME>
<PROCESS_ID>
<PROCESS_GUID>
<PARENT_PROCESS>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Wazuh FIM alert
* Previous and current hash comparison
* File metadata
* File ownership or permission change
* Sysmon file-creation event
* Process tree
* Splunk file-change timeline
* Defender detection
* Persistence evidence
* Restored-file validation

Before publishing:

1. Crop unrelated information.
2. Remove usernames.
3. Remove operational addresses and domains.
4. Remove sensitive file contents.
5. Remove credentials, tokens, and keys.
6. Remove SIDs and process GUIDs where unnecessary.
7. Remove unrelated file paths.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Reporting host identified
* File path identified
* Change type identified
* Initial severity assigned

## File Validation

* File existence checked
* Metadata recorded
* Current hash recorded
* Previous hash recorded where available
* Owner reviewed
* Permissions reviewed
* Signature reviewed
* Alternate data streams reviewed
* Evidence copy preserved

## Wazuh Review

* FIM event reviewed
* Rule ID reviewed
* Rule level reviewed
* `syscheck.event` reviewed
* Previous and current hashes reviewed
* User attribution reviewed
* Process attribution reviewed
* Repeated changes checked

## Process Review

* Sysmon Event ID 11 reviewed
* Sysmon Event ID 23 or 26 reviewed
* Sysmon Event ID 1 reviewed
* Event ID 4688 reviewed
* Parent process identified
* Command line reviewed
* User and integrity level reviewed
* Process hash and signature reviewed

## PowerShell Review

* Event ID 4103 reviewed where relevant
* Event ID 4104 reviewed where relevant
* File-modification commands reviewed
* Encoded or obfuscated execution escalated

## Security Review

* Defender events reviewed
* Defender health confirmed
* File execution assessed
* Child processes reviewed
* Network connections reviewed
* DNS queries reviewed
* Persistence reviewed
* Permission changes reviewed
* Related files reviewed

## SIEM Review

* Wazuh search completed
* Splunk file-change search completed
* File-to-process correlation completed
* File-to-execution correlation completed
* File-to-Defender correlation completed
* Sensitive-directory context reviewed
* Ingestion delay reviewed where needed

## Response

* Severity updated
* Containment decision recorded
* File quarantined or restricted where required
* Harmful process stopped where required
* Endpoint isolated where required
* Account contained where required
* Unauthorized persistence removed
* Trusted file restored
* Permissions restored
* Application health validated

## Closure

* Timeline completed
* Evidence preserved
* Evidence hashes recorded
* Root cause documented
* Authorized or incident classification recorded
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Primary File Check

```
Get-Item -LiteralPath "<FILE_PATH>" -Force
```

## Primary Hash Check

```
Get-FileHash `
    -LiteralPath "<FILE_PATH>" `
    -Algorithm SHA256
```

## Primary Sysmon Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 11, 23, 26
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<FILENAME>")
}
```

## Primary Splunk Search

```
index=<WAZUH_INDEX> earliest=-24h
| search syscheck.path="*<FILENAME>*"
| table
    _time
    agent.name
    syscheck.event
    syscheck.path
    syscheck.sha256_before
    syscheck.sha256_after
    syscheck.audit.user.name
    syscheck.audit.process.name
    rule.level
| sort _time
```

## Immediate Escalation Indicators

```
Critical system file changed
Security configuration changed
Unknown executable or script
File executed after creation
Persistence established
Permissions weakened
Defender detection
Privileged or unknown user
Multiple endpoints affected
Deletion after execution
Destructive activity
```

---

# Summary

This runbook provides a repeatable process for investigating file integrity changes.

A complete investigation should determine:

* Which file changed
* What type of change occurred
* When the change occurred
* Which user and process performed it
* Whether hashes, ownership, or permissions changed
* Whether the file is trusted
* Whether it was executed
* Whether it created persistence
* Whether it affected security controls
* Whether Microsoft Defender detected related activity
* Whether the change was authorized
* Whether containment and restoration were required
* Whether the final file and endpoint state are healthy

The investigation is complete only after the file change is classified, unauthorized content or configuration is removed, trusted state is restored, evidence is preserved, monitoring confirms the intended state, and detection improvements are documented.

