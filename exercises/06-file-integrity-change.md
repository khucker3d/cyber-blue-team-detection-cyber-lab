# File Integrity Change

## Exercise Overview

This exercise creates, modifies, renames, and deletes a controlled test file on WIN11TARGET to validate file integrity monitoring across:

* Windows file-system activity
* Wazuh File Integrity Monitoring
* Splunk
* Optional Sysmon telemetry
* Evidence hashing
* Investigation notes
* Cleanup validation

The objective is to determine:

* Which file changed
* What type of change occurred
* Which system generated the change
* Which user performed the action
* Whether file hashes changed
* Whether Wazuh detected the activity
* Whether Splunk received related telemetry
* Whether the monitoring configuration provided enough context
* Whether the test directory was restored safely

All activity must remain inside an approved CyberLab test directory.

---

## Exercise ID

```
EX-06
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* File integrity monitoring
* File hash validation
* Windows file-system analysis
* Wazuh FIM investigation
* Splunk SPL development
* Optional Sysmon analysis
* Change timeline reconstruction
* Evidence preservation
* Detection tuning
* Cleanup verification

---

## Authorization Boundary

This exercise must use:

* WIN11TARGET
* A dedicated test directory
* Synthetic text files
* A non-administrative test account where possible
* The VMware host-only network
* Wazuh and Splunk monitoring
* Approved filenames and content

Do not use:

* Windows system directories
* Active Directory files
* Application binaries
* Personal documents
* Security-tool configuration files
* Password databases
* Browser profiles
* Production data
* Any system outside the CyberLab

---

## Public Example Environment

```
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Test account: file.test
Test directory: C:\CyberLab-Test\FIM
Primary test file: fim-test.txt
Renamed test file: fim-test-renamed.txt
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Review the Wazuh FIM configuration
* Confirm the test path is monitored
* Create a controlled test file
* Record its initial hash
* Modify the file and compare hashes
* Rename the file
* Delete the file
* Identify the related Wazuh events
* Review related Splunk telemetry
* Use optional Sysmon file events
* Build a complete file-change timeline
* Distinguish authorized and suspicious file changes
* Document monitoring gaps

---

## Required Systems

* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on WIN11TARGET
* Splunk Universal Forwarder where configured

Optional:

* Sysmon on WIN11TARGET

DC01 and KALI-TEST are not required for the standard exercise.

---

## Required Account

Use a dedicated standard account:

```
file.test
```

An authorized administrator may be required to:

* Review Wazuh configuration
* Restart the Wazuh agent
* Review protected event logs
* Configure optional Sysmon collection

The exercise file operations should be performed without administrative privileges whenever possible.

---

## Required Monitoring

Before starting, confirm:

* Wazuh agent is active
* The test directory is included in FIM monitoring
* Wazuh dashboard is accessible
* Splunk is running
* Relevant Wazuh or Windows events are searchable
* Optional Sysmon file events are configured where used
* System time is synchronized
* A stable snapshot exists

---

## Safety Notes

* Use only the approved test directory.
* Do not monitor the entire system drive for this exercise.
* Use synthetic file content.
* Do not place credentials or personal information in test files.
* Record the file hash before and after modification.
* Preserve evidence before deletion.
* Do not delete the monitored directory until collection is complete.
* Do not disable FIM during cleanup.
* Stop immediately if unrelated directories generate unexpected high-volume alerts.

---

## Expected Activity Flow

```
Test File Created
       |
       v
Initial Hash Recorded
       |
       v
File Modified
       |
       v
Hash Changes
       |
       v
File Renamed
       |
       v
File Deleted
       |
       v
Wazuh / Splunk / Sysmon Review
       |
       v
Timeline and Cleanup Validation
```

---

## Expected Telemetry

Potential evidence includes:

* File creation
* File modification
* File rename
* File deletion
* File path
* File size
* Modification time
* Previous hash
* Current hash
* User where available
* Process where available
* Wazuh rule
* Wazuh alert severity
* Sysmon file event
* Splunk indexed event

Not every data source will provide every field.

---

## Relevant Event Sources

### Wazuh FIM

Wazuh may provide:

* Added file
* Modified file
* Deleted file
* Checksum changes
* File attributes
* User and process context when whodata is configured

### Sysmon

Potential event IDs include:

| Event ID | Description                |
| -------: | -------------------------- |
|       11 | FileCreate                 |
|       15 | FileCreateStreamHash       |
|       23 | FileDelete archived        |
|       26 | FileDelete detected        |
|        2 | File creation time changed |

Availability depends on the Sysmon configuration and version.

### Windows Security Auditing

Windows Object Access auditing may provide file events when:

* File-system auditing is enabled
* A suitable SACL exists on the directory
* The relevant subcategories are enabled

This is optional for the standard exercise.

---

## Investigation Questions

The final report should answer:

* Which file was created?
* Which user created it?
* What was the initial hash?
* What content change was made?
* What was the new hash?
* Did Wazuh detect the creation?
* Did Wazuh detect the modification?
* Did Wazuh detect the rename?
* Did Wazuh detect the deletion?
* Was user context available?
* Was process context available?
* Did Sysmon produce related events?
* Did Splunk index the events?
* Were timestamps aligned?
* Was the Wazuh severity appropriate?
* What legitimate activity could produce similar changes?
* What additional context would improve the investigation?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

Suggested pre-exercise snapshot:

```
PRE-EX06-File-Integrity-Change
```

A new snapshot is recommended before changing Wazuh FIM or Sysmon configuration.

---

## Confirm System Time

On WIN11TARGET:

```
Get-Date
```

```
w32tm /query /status
```

```
w32tm /query /source
```

On Wazuh and Splunk:

```
timedatectl
```

Record any time difference.

---

## Confirm Wazuh Agent Status

On WIN11TARGET:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm the endpoint is active in the Wazuh dashboard.

---

## Confirm Splunk Forwarder Status

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm recent endpoint telemetry:

```spl
index=windows host=WIN11TARGET earliest=-15m
| head 20
```

---

## Confirm Optional Sysmon Status

```
Get-Service |
    Where-Object DisplayName -Match "Sysmon"
```

Confirm the log exists:

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-Sysmon/Operational" `
    -ErrorAction SilentlyContinue
```

Sysmon is optional for this exercise.

---

# Wazuh FIM Configuration Review

## Locate the Agent Configuration

The Wazuh agent configuration is typically stored at:

```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

The exact path may differ by installation.

Administrator privileges are required to edit this file.

---

## Back Up the Configuration

Run from an administrator PowerShell session:

```
Copy-Item `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" `
    -Destination "C:\Program Files (x86)\ossec-agent\ossec.conf.pre-ex06.bak"
```

Do not publish the operational configuration without sanitization.

---

## Review the Syscheck Section

Look for a configuration block similar to:

```xml
<syscheck>
  <frequency>43200</frequency>
  <directories check_all="yes">C:\CyberLab-Test\FIM</directories>
</syscheck>
```

A more responsive lab configuration may use real-time monitoring where supported:

```xml
<syscheck>
  <directories
    check_all="yes"
    realtime="yes"
    whodata="yes">C:\CyberLab-Test\FIM</directories>
</syscheck>
```

Support for `realtime` and `whodata` depends on the operating system, agent version, and audit configuration.

---

## Monitoring Scope Guidance

For this exercise, monitor only:

```
C:\CyberLab-Test\FIM
```

Avoid broad paths such as:

```
C:\
C:\Windows
C:\Program Files
```

Broad monitoring creates excessive noise and may affect performance.

---

## Validate the Configuration

After an approved configuration change, restart the Wazuh agent:

```
Restart-Service WazuhSvc
```

Administrator privileges are required.

Confirm:

```
Get-Service WazuhSvc
```

Expected state:

```
Running
```

---

## Review the Agent Log

Typical path:

```
C:\Program Files (x86)\ossec-agent\ossec.log
```

Review recent entries:

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 100
```

Confirm the monitored path was accepted.

---

## Confirm the Test Directory

Create the directory as the standard test user:

```
New-Item `
    -Path "C:\CyberLab-Test\FIM" `
    -ItemType Directory `
    -Force
```

Confirm:

```
Get-Item "C:\CyberLab-Test\FIM"
```

---

## Confirm Directory Permissions

```
Get-Acl "C:\CyberLab-Test\FIM" |
    Format-List
```

Confirm the test account can create and modify files without granting unnecessary permissions.

Do not publish unsanitized access-control output if it contains real identities.

---

## Start the Exercise Record

```
Exercise ID: EX-06
Test account: file.test
Endpoint: WIN11TARGET
Test directory: C:\CyberLab-Test\FIM
Start time:
Wazuh realtime monitoring:
Wazuh whodata:
Sysmon:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Test 1: Create the File

Run as the standard test user:

```
Set-Content `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" `
    -Value "Authorized file integrity exercise."
```

Record the time immediately.

---

## Confirm the File Exists

```
Get-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" |
Select-Object `
    Name,
    FullName,
    Length,
    CreationTime,
    LastWriteTime
```

---

## Record the Initial Hash

```
Get-FileHash `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" `
    -Algorithm SHA256
```

Record:

```
Creation time:
Initial size:
Initial SHA-256:
File owner:
```

---

## Record the File Owner

```
(Get-Acl "C:\CyberLab-Test\FIM\fim-test.txt").Owner
```

---

## Wait for Collection

When real-time monitoring is enabled, allow sufficient time for the event to arrive.

When only scheduled FIM is enabled, the change may not appear until the next scan.

Do not repeatedly recreate the file while waiting.

---

## Test 2: Modify the File

Append controlled content:

```
Add-Content `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" `
    -Value "Controlled modification."
```

Record the modification time.

---

## Confirm the Modified Content

```
Get-Content `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt"
```

Expected content:

```
Authorized file integrity exercise.
Controlled modification.
```

---

## Record the Modified Hash

```
Get-FileHash `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" `
    -Algorithm SHA256
```

Record:

```
Modification time:
Modified size:
Modified SHA-256:
Hash changed:
```

The modified hash should differ from the initial hash.

---

## Test 3: Rename the File

```
Rename-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt" `
    -NewName "fim-test-renamed.txt"
```

Record the rename time.

---

## Confirm the Renamed File

```
Get-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt" |
Select-Object `
    Name,
    FullName,
    Length,
    CreationTime,
    LastWriteTime
```

Depending on the telemetry source, a rename may appear as:

* Original file deleted
* New file added
* Rename event
* Multiple related events

Document the actual behavior.

---

## Test 4: Delete the File

Preserve the required evidence first.

Then:

```
Remove-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt"
```

Record the deletion time.

---

## Confirm Deletion

```
Test-Path `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt"
```

Expected:

```
False
```

---

## Record the Exercise Activity

```
Creation time:
Initial SHA-256:
Modification time:
Modified SHA-256:
Rename time:
Deletion time:
Unexpected errors:
```

---

# Local Validation

## Review File Properties During the Test

Before deletion, capture:

```
Get-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt" |
Select-Object `
    FullName,
    Length,
    CreationTime,
    LastWriteTime,
    Attributes
```

---

## Compare the Hashes

Example record:

| Stage    | SHA-256           |
| -------- | ----------------- |
| Initial  | `<INITIAL_HASH>`  |
| Modified | `<MODIFIED_HASH>` |

Expected:

```
Initial hash and modified hash are different.
```

---

## Optional Windows Object Access Auditing

Object Access auditing is not required for the standard exercise.

When enabled, review:

```
auditpol /get /subcategory:"File System"
```

A SACL must also exist on the test directory.

Potential Security events include:

| Event ID | Description                             |
| -------: | --------------------------------------- |
|     4656 | A handle to an object was requested     |
|     4663 | An attempt was made to access an object |
|     4660 | An object was deleted                   |
|     4658 | The handle to an object was closed      |

These events can be high volume and should be configured narrowly.

---

# Wazuh Validation

## Confirm Agent Health

In the Wazuh dashboard, confirm:

* WIN11TARGET is active
* Last keepalive is recent
* The selected time range includes the exercise
* No agent identity changed

---

## Search by Test Directory

Search for:

```
C:\CyberLab-Test\FIM
```

Also search for:

```
fim-test.txt
fim-test-renamed.txt
```

---

## Review File Creation

Locate the event representing the new file.

Record:

```
Event time:
Agent:
File path:
Action:
File size:
Hash:
User:
Process:
Rule:
Rule level:
```

---

## Review File Modification

Locate the modification event.

Review:

* Previous checksum
* Current checksum
* File size change
* Modification time
* User
* Process
* Rule description
* Severity

---

## Review Rename Behavior

Determine whether Wazuh recorded the rename as:

* Delete plus add
* Direct rename
* Multiple events
* No distinct rename context

Record the result.

---

## Review File Deletion

Locate the deletion event.

Record:

```
Deletion event time:
Original file path:
User:
Process:
Previous hash:
Rule:
Rule level:
```

---

## Wazuh Validation Questions

* Did Wazuh detect creation?
* Did Wazuh detect modification?
* Did the hash change appear?
* Did Wazuh detect the rename?
* Did Wazuh detect deletion?
* Was the test user identified?
* Was the responsible process identified?
* Was the event received promptly?
* Was the severity appropriate?
* Did one file action generate duplicate alerts?
* Did the path use consistent formatting?

---

## Wazuh Detection Gap Record

```
Creation detected:
Modification detected:
Rename detected:
Deletion detected:
Initial hash present:
Modified hash present:
User present:
Process present:
Alert present:
Rule level:
Missing fields:
Improvement:
```

---

# Splunk Validation

## Search for the Test Path

```spl
index=* earliest=-30m
(
    "C:\\CyberLab-Test\\FIM"
    OR "fim-test.txt"
    OR "fim-test-renamed.txt"
)
| table
    _time
    host
    source
    sourcetype
    EventCode
    user
    file
    path
    action
    Message
| sort _time
```

Replace the broad index with the actual Wazuh or Windows index when known.

---

## Search Wazuh-Forwarded FIM Events

When Wazuh alerts are forwarded into Splunk:

```spl
index=<WAZUH_INDEX> earliest=-30m
| search
    "fim-test.txt"
    OR "fim-test-renamed.txt"
| table
    _time
    agent.name
    rule.id
    rule.level
    rule.description
    syscheck.path
    syscheck.event
    syscheck.md5_before
    syscheck.md5_after
    syscheck.sha256_before
    syscheck.sha256_after
    syscheck.audit.user.name
    syscheck.audit.process.name
| sort _time
```

Field names depend on the integration and event format.

---

## Search Optional Sysmon File Creation

```spl
index=windows EventCode=11 source="*Sysmon*" earliest=-30m
| search
    TargetFilename="*\\CyberLab-Test\\FIM\\*"
| table
    _time
    host
    User
    Image
    ProcessId
    TargetFilename
    CreationUtcTime
| sort _time
```

---

## Search Optional Sysmon File Deletion

```spl
index=windows earliest=-30m
(
    EventCode=23 OR
    EventCode=26
)
| search
    TargetFilename="*\\CyberLab-Test\\FIM\\*"
| table
    _time
    host
    EventCode
    User
    Image
    ProcessId
    TargetFilename
    Hashes
| sort _time
```

Availability depends on the Sysmon configuration.

---

## Build a File-Change Timeline

```spl
index=* earliest=-30m
(
    "fim-test.txt"
    OR "fim-test-renamed.txt"
)
| eval file_path=coalesce(
    'syscheck.path',
    TargetFilename,
    file,
    path
)
| eval file_action=coalesce(
    'syscheck.event',
    action,
    case(
        EventCode=11, "created",
        EventCode=23, "deleted and archived",
        EventCode=26, "deleted"
    )
)
| table
    _time
    host
    source
    EventCode
    file_action
    file_path
    user
    Image
    'syscheck.sha256_before'
    'syscheck.sha256_after'
| sort _time
```

Adjust indexes and field names for the actual deployment.

---

## Compare Before and After Hashes

```spl
index=<WAZUH_INDEX> earliest=-30m
| search "fim-test.txt"
| table
    _time
    'syscheck.path'
    'syscheck.event'
    'syscheck.sha256_before'
    'syscheck.sha256_after'
    'syscheck.size_before'
    'syscheck.size_after'
| sort _time
```

---

## Measure Ingestion Delay

```spl
index=* earliest=-30m
(
    "fim-test.txt"
    OR "fim-test-renamed.txt"
)
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    source
    EventCode
| sort _time
```

---

## Splunk Validation Questions

* Which index contained the events?
* Was the file path parsed?
* Was the action parsed?
* Were before and after hashes present?
* Was the user present?
* Was the responsible process present?
* Did the rename appear clearly?
* Was the deletion event captured?
* Was ingestion timely?
* Were duplicate events present?
* Could the data support a reliable detection?

---

# Optional Sysmon Validation

## Search for File Creation Locally

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 11
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "CyberLab-Test\\FIM"
}
```

---

## Search for File Deletion Locally

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 23, 26
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "CyberLab-Test\\FIM"
}
```

---

## Review Sysmon Fields

Review:

* Rule name
* Event time
* Process GUID
* Process ID
* User
* Image
* Target filename
* Hashes
* Archived state

Sysmon may provide process context that Wazuh baseline FIM lacks.

---

# Investigation

## Build the Timeline

| Time     | System      | Action   | File                 | User      | Analyst Note              |
| -------- | ----------- | -------- | -------------------- | --------- | ------------------------- |
| `<TIME>` | WIN11TARGET | Created  | fim-test.txt         | file.test | Initial file created      |
| `<TIME>` | WIN11TARGET | Hashed   | fim-test.txt         | file.test | Initial SHA-256 recorded  |
| `<TIME>` | WIN11TARGET | Modified | fim-test.txt         | file.test | Controlled text appended  |
| `<TIME>` | WIN11TARGET | Hashed   | fim-test.txt         | file.test | Modified SHA-256 recorded |
| `<TIME>` | WIN11TARGET | Renamed  | fim-test-renamed.txt | file.test | Filename changed          |
| `<TIME>` | WIN11TARGET | Deleted  | fim-test-renamed.txt | file.test | Cleanup action            |
| `<TIME>` | Wazuh       | Alerted  | Test path            | file.test | FIM telemetry received    |
| `<TIME>` | Splunk      | Indexed  | Test path            | file.test | Timeline searchable       |

---

## Root Cause

For this controlled exercise:

```
Authorized creation, modification, rename, and deletion of a synthetic file inside the approved CyberLab FIM test directory during Exercise EX-06.
```

---

## Why File Changes Can Be Security-Relevant

File changes may indicate:

* Normal user activity
* Software installation
* Configuration updates
* Patch deployment
* Log rotation
* Backup operations
* Script execution
* Persistence
* Web shell placement
* Unauthorized configuration change
* Malware activity
* Evidence destruction

The file path, process, user, hash, and timing determine context.

---

## Benign Indicators

Indicators supporting a benign conclusion include:

* Approved test directory
* Known test account
* Documented change time
* Harmless text content
* Expected PowerShell process
* No protected path involved
* No privilege escalation
* No network activity
* Cleanup matching the exercise plan

---

## Suspicious Indicators

Concern would increase when:

* A protected system file changes
* An executable appears in a startup path
* A script is created in a temporary directory
* An unexpected process modifies security configuration
* The user is unknown
* The process runs as SYSTEM
* Hashes match known malicious indicators
* Many files change rapidly
* File changes occur across multiple hosts
* The file is deleted immediately after execution
* Monitoring configuration is changed

---

## Process Context Analysis

When process information is available, review:

* Executable path
* Parent process
* User
* Integrity level
* Command line
* Signature
* Hash
* Working directory

A file change without process context may require correlation with process creation events.

---

## User Context Analysis

When user information is available, determine:

* Standard or administrative account
* Interactive or service context
* Expected access to the path
* Account age
* Related authentication
* Related privilege changes

---

## Hash Analysis

A changed hash confirms that file contents changed.

It does not explain:

* Who changed it
* Why it changed
* Whether it is malicious
* Which process performed the change

Hash evidence should be combined with user, process, path, and timeline context.

---

## False-Positive Analysis

Legitimate file changes may come from:

* Windows Update
* Antivirus
* Software installers
* Configuration management
* Log rotation
* User applications
* Backup software
* Temporary files
* Browser caches
* Security agents
* Administrative scripts

FIM scope and exclusions should be tuned carefully.

---

# Detection Development

## Basic Wazuh FIM Search in Splunk

```spl
index=<WAZUH_INDEX> earliest=-24h
| search 'syscheck.event' IN ("added", "modified", "deleted")
| table
    _time
    agent.name
    rule.level
    rule.description
    syscheck.event
    syscheck.path
    syscheck.sha256_before
    syscheck.sha256_after
| sort - _time
```

---

## Monitor Sensitive Paths

```spl
index=<WAZUH_INDEX> earliest=-24h
| eval monitored_path=lower('syscheck.path')
| where like(monitored_path, "c:\\windows\\system32\\%")
    OR like(monitored_path, "c:\\programdata\\microsoft\\windows\\start menu\\programs\\startup\\%")
    OR like(monitored_path, "c:\\users\\%\\appdata\\roaming\\microsoft\\windows\\start menu\\programs\\startup\\%")
| table
    _time
    agent.name
    syscheck.event
    syscheck.path
    syscheck.audit.user.name
    syscheck.audit.process.name
    rule.level
```

Paths should be adapted and normalized for the actual event format.

---

## Detect Executable Creation in a User-Writable Path

```spl
index=* earliest=-24h
| eval file_path=lower(
    coalesce(
        'syscheck.path',
        TargetFilename,
        file,
        path
    )
)
| where match(file_path, "\\.(exe|dll|ps1|bat|cmd|vbs|js)$")
| where like(file_path, "c:\\users\\%\\downloads\\%")
    OR like(file_path, "c:\\users\\%\\appdata\\local\\temp\\%")
    OR like(file_path, "c:\\programdata\\%")
| table
    _time
    host
    user
    Image
    file_path
    action
    rule.description
```

A matching extension or path is context, not proof of malicious behavior.

---

## Detect Rapid File Changes

```spl
index=<WAZUH_INDEX> earliest=-15m
| bin _time span=5m
| stats
    count as file_changes
    dc('syscheck.path') as unique_files
    values('syscheck.event') as actions
    by _time agent.name
| where file_changes >= <CHANGE_THRESHOLD>
```

Thresholds should be based on normal behavior.

---

## Detect Changes by an Unusual Process

```spl
index=<WAZUH_INDEX> earliest=-24h
| eval process=lower('syscheck.audit.process.name')
| where isnotnull(process)
| where NOT process IN (
    "explorer.exe",
    "powershell.exe",
    "notepad.exe",
    "<APPROVED_PROCESS>"
)
| table
    _time
    agent.name
    syscheck.event
    syscheck.path
    syscheck.audit.user.name
    process
    rule.level
```

Allowlists should be narrow and reviewed.

---

## Wazuh Rule Concept: Sensitive File Change

```xml
<group name="syscheck,file_integrity,sensitive_path,">
  <rule id="<CUSTOM_RULE_ID>" level="10">
    <if_group>syscheck</if_group>
    <field name="syscheck.path" type="pcre2">(?i)^c:\\windows\\system32\\</field>
    <description>File integrity change detected in a sensitive Windows path</description>
    <mitre>
      <id>T1105</id>
    </mitre>
  </rule>
</group>
```

The ATT&CK mapping must match the actual behavior being detected.

A generic file change should not be mapped automatically to ingress tool transfer.

Use technique mappings carefully and only when context supports them.

---

## Better ATT&CK Context

Depending on the real behavior, file changes may relate to:

* T1036 — Masquerading
* T1105 — Ingress Tool Transfer
* T1547 — Boot or Logon Autostart Execution
* T1562.001 — Impair Defenses
* T1070.004 — File Deletion

The controlled exercise itself is authorized and does not establish any of these techniques.

---

## Detection Severity Guidance

| Pattern                                              | Suggested Context           |
| ---------------------------------------------------- | --------------------------- |
| Text file changed in approved test directory         | Informational or Low        |
| Configuration file changed by approved administrator | Low or Medium               |
| Executable created in user startup path              | High                        |
| Security-tool configuration changed unexpectedly     | High                        |
| System binary modified                               | Critical review             |
| Many files changed rapidly                           | High investigation priority |
| File created and deleted immediately after execution | Medium or High              |
| Unknown process changes protected file               | High                        |
| FIM configuration disabled or altered                | Critical review             |

Severity should reflect path sensitivity, process, user, scale, and impact.

---

# Evidence Collection

## Evidence

Preserve:

* Wazuh FIM configuration
* Test directory permissions
* Initial file properties
* Initial SHA-256 hash
* Modified file properties
* Modified SHA-256 hash
* Wazuh creation event
* Wazuh modification event
* Wazuh rename behavior
* Wazuh deletion event
* Splunk timeline
* Optional Sysmon events
* Cleanup confirmation
* Investigation notes

---

## Export Wazuh Evidence

Export a narrow set of results from the dashboard or index.

Suitable formats include:

* CSV
* JSON
* Screenshot
* Saved query result

Review every field before publication.

---

## Export Optional Sysmon Events

```
wevtutil epl `
    "Microsoft-Windows-Sysmon/Operational" `
    "<EVIDENCE_PATH>\WIN11TARGET-Sysmon-Operational.evtx"
```

Raw event logs should remain private.

---

## Save Hash Records

Example:

```
File: fim-test.txt
Stage: Initial
SHA-256: <INITIAL_HASH>
Collected:
Collected by:
```

```
File: fim-test.txt
Stage: Modified
SHA-256: <MODIFIED_HASH>
Collected:
Collected by:
```

---

## Hash Evidence Files

Windows:

```
Get-FileHash `
    -Path "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

Linux:

```
sha256sum <EVIDENCE_FILE>
```

---

## Evidence Record

```
Exercise: EX-06
File:
Source:
Collection time:
Description:
SHA-256:
Student analyst:
Storage location:
```

---

# Cleanup and Recovery

## Preserve Evidence Before Cleanup

Before removing the final file:

1. Save initial and modified hashes.
2. Capture Wazuh events.
3. Save Splunk results.
4. Save optional Sysmon events.
5. Record file properties.
6. Record the rename behavior.
7. Confirm the deletion test is still required.

---

## Confirm the Test File Is Deleted

```
Test-Path `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt"
```

Expected:

```
False
```

Also confirm the original name is absent:

```
Test-Path `
    -Path "C:\CyberLab-Test\FIM\fim-test.txt"
```

Expected:

```
False
```

---

## Review the Test Directory

```
Get-ChildItem `
    -Path "C:\CyberLab-Test\FIM" `
    -Force
```

Confirm no unintended files remain.

---

## Retain or Remove the Directory

Retain the directory when it is the permanent approved FIM test path.

To remove it when no longer needed:

```
Remove-Item `
    -Path "C:\CyberLab-Test\FIM" `
    -Force
```

Do not remove the path from Wazuh configuration unintentionally.

---

## Restore the Wazuh Configuration if Temporary

If the test path was added only for this exercise:

1. Preserve the evidence.
2. Restore the backed-up configuration.
3. Restart the Wazuh agent.
4. Confirm agent health.
5. Confirm the configuration is valid.

Example restore:

```
Copy-Item `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.conf.pre-ex06.bak" `
    -Destination "C:\Program Files (x86)\ossec-agent\ossec.conf" `
    -Force
```

Then:

```
Restart-Service WazuhSvc
```

Administrator privileges are required.

---

## Confirm Monitoring Health

```
Get-Service WazuhSvc
```

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm the endpoint remains active in both monitoring platforms.

---

## Final State Record

```
Test file created:
Initial hash recorded:
Test file modified:
Modified hash recorded:
Rename validated:
Deletion validated:
Test directory reviewed:
Wazuh configuration restored or retained intentionally:
Wazuh healthy:
Splunk healthy:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* An approved test directory was used.
* A synthetic file was created.
* The initial hash was recorded.
* The file was modified.
* The modified hash differed.
* Rename behavior was documented.
* The file was deleted.
* Wazuh creation telemetry was reviewed.
* Wazuh modification telemetry was reviewed.
* Wazuh deletion telemetry was reviewed.
* User and process context were assessed.
* Splunk was reviewed.
* Optional Sysmon telemetry was reviewed where available.
* Ingestion delay was assessed.
* False positives were considered.
* Cleanup was completed.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                   | Result                       |
| -------------------------- | ---------------------------- |
| File creation detected     | Pass / Fail                  |
| File modification detected | Pass / Fail                  |
| Before and after hashes    | Pass / Partial / Fail        |
| Rename behavior visible    | Pass / Partial / Fail        |
| File deletion detected     | Pass / Fail                  |
| User context               | Pass / Partial / Fail        |
| Process context            | Pass / Partial / Fail        |
| Wazuh alert quality        | Pass / Needs Tuning / Fail   |
| Splunk ingestion           | Pass / Fail / Not Configured |
| Sysmon telemetry           | Pass / Fail / Not Installed  |
| Timestamp accuracy         | Pass / Needs Review / Fail   |
| Cleanup                    | Complete / Incomplete        |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* User
* Process
* Parent process
* Command line
* Initial hash
* Modified hash
* File size
* Rename context
* Deletion context
* Path normalization
* File owner
* Integrity level
* Clear severity
* Correlation across event sources

---

## Improvements

Potential improvements include:

* Enable Wazuh real-time monitoring
* Enable whodata where supported
* Tune monitored paths
* Add exclusions for noisy temporary files
* Deploy Sysmon file events
* Collect Windows Object Access events selectively
* Normalize paths across data sources
* Add sensitive-path rules
* Add executable-creation detection
* Add rapid-change thresholds
* Build a FIM dashboard
* Create a file-change investigation runbook

---

# Troubleshooting

## No Wazuh Event Appears

Check:

* Wazuh agent status
* Test path in `ossec.conf`
* XML syntax
* Agent restart
* Realtime setting
* Scheduled scan frequency
* Dashboard time range
* Agent filter
* Indexer health
* File actually changed

Review:

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 200
```

---

## Configuration Change Breaks the Agent

Possible causes include:

* Invalid XML
* Duplicate tags
* Unsupported attributes
* Incorrect path
* Quotation errors
* Permission issues

Restore the backup configuration and restart the agent.

Do not continue editing repeatedly without validating each change.

---

## Creation Appears but Modification Does Not

Check:

* The file content actually changed
* Hashes differ
* Real-time monitoring
* Scan interval
* File exclusion
* Agent logs
* Dashboard time range

Record the local hashes before changing configuration.

---

## Rename Appears as Delete and Add

This is expected in some FIM implementations.

Document:

* Original path deleted
* New path added
* Matching file size
* Matching hash
* Close timestamps

Correlation can reconstruct the rename.

---

## User or Process Is Missing

Possible causes include:

* Whodata not enabled
* Audit policy not configured
* Unsupported file-system behavior
* Agent version
* Event-source limitation
* Action completed before monitoring initialized

Use Sysmon or Windows Object Access auditing for additional context.

---

## Whodata Does Not Work

Check:

* Agent configuration
* Windows audit policy
* Required privileges
* NTFS file system
* Agent restart
* Wazuh documentation for the installed version
* Agent log errors

Do not enable broad Object Access auditing without understanding the volume.

---

## Splunk Does Not Show the Event

Check:

* Whether Wazuh alerts are forwarded to Splunk
* Relevant index
* Host field
* Sourcetype
* Time range
* Field names
* Splunk forwarder or integration status

Begin broadly:

```spl
index=* earliest=-30m
| search "fim-test"
```

Then narrow by source and index.

---

## Sysmon Does Not Show File Events

Check:

* Sysmon service
* Sysmon configuration
* Event ID included
* Target path filters
* Event log time range
* File action supported by the configured event type

Sysmon does not log every file-system action by default.

---

## Too Many FIM Alerts Appear

Possible causes include:

* Broad monitored path
* Application cache
* Browser data
* Windows updates
* Antivirus activity
* Temporary files
* Log directories
* Repeated scans

Tune by:

* Narrowing paths
* Adding justified exclusions
* Adjusting scan frequency
* Using file type filters
* Separating high-value and noisy paths
* Reviewing baseline behavior

Do not suppress an entire drive without review.

---

## Hashes Do Not Change

Possible causes include:

* File content unchanged
* Wrong file hashed
* Modification failed
* Path changed after rename
* Hash recorded before write completed

Review the content and path, then recalculate.

---

## Test File Cannot Be Deleted

Check:

* Open application
* File handle
* Permissions
* Antivirus action
* Wrong filename after rename
* Current working directory
* Read-only attribute

Review:

```
Get-Item `
    -Path "C:\CyberLab-Test\FIM\fim-test-renamed.txt" `
    -Force
```

---

# Public Sanitization

Remove or replace:

* Operational usernames
* Internal IP addresses
* Real monitored paths when sensitive
* Wazuh agent identifiers
* Wazuh manager address
* Splunk internal URLs
* Security identifiers
* File owner identities
* Raw configuration secrets
* Evidence storage paths
* Unrelated FIM events
* Personal filenames
* Real file hashes tied to private files

Use placeholders such as:

```
<TEST_ACCOUNT>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<TEST_DIRECTORY>
<TEST_FILE>
<INITIAL_HASH>
<MODIFIED_HASH>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Wazuh monitored-path configuration
* File creation alert
* File modification alert
* Before and after hash values
* Rename behavior
* File deletion alert
* Splunk file-change timeline
* Optional Sysmon file event
* Detection-quality table

Before publishing:

1. Crop unrelated information.
2. Remove real usernames.
3. Remove operational addresses.
4. Remove unrelated file paths.
5. Remove agent identifiers.
6. Remove SIDs.
7. Permanently redact sensitive values.
8. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
A synthetic text file was created, modified, renamed, and deleted inside an approved CyberLab test directory during Exercise EX-06. File hashes and properties were recorded, and the resulting file integrity telemetry was reviewed through Wazuh, Splunk, and optional Sysmon data. The activity was classified as authorized test behavior, and monitoring gaps were documented.
```

## Findings

```
Test account:

Endpoint:

Test directory:

Initial filename:

Renamed filename:

Creation time:

Initial SHA-256:

Modification time:

Modified SHA-256:

Rename time:

Deletion time:

Wazuh creation result:

Wazuh modification result:

Wazuh deletion result:

User context:

Process context:

Splunk result:

Sysmon result:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized file-system activity performed inside the approved CyberLab FIM test path during Exercise EX-06.
```

## Resolution

```
The test file was removed, the test directory was reviewed, temporary monitoring configuration was restored or retained intentionally, and Wazuh and Splunk health were confirmed.
```

## Recommendations

```
- Enable real-time FIM monitoring for high-value paths.
- Enable whodata where supported.
- Add process context through Sysmon.
- Normalize path and hash fields in Splunk.
- Create sensitive-path and executable-creation detections.
- Develop a file-integrity investigation runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshot exists
* Test account validated
* Test directory approved
* Wazuh agent healthy
* Test path included in FIM
* Realtime setting reviewed
* Whodata setting reviewed
* Splunk healthy
* Optional Sysmon reviewed
* System time validated
* Start time recorded

## Exercise

* Test file created
* Initial properties recorded
* Initial SHA-256 recorded
* File modified
* Modified properties recorded
* Modified SHA-256 recorded
* Hash change confirmed
* File renamed
* Rename behavior recorded
* File deleted
* Deletion confirmed

## Investigation

* Wazuh creation event reviewed
* Wazuh modification event reviewed
* Wazuh rename behavior reviewed
* Wazuh deletion event reviewed
* User context reviewed
* Process context reviewed
* Splunk reviewed
* Optional Sysmon reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Original filename absent
* Renamed filename absent
* Test directory reviewed
* Temporary Wazuh configuration restored or retained intentionally
* Wazuh healthy
* Splunk healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Wazuh File Integrity Monitoring
* File creation and modification analysis
* SHA-256 hashing
* Before-and-after comparison
* File rename correlation
* File deletion investigation
* Optional Sysmon analysis
* Splunk SPL
* User and process context review
* Timeline reconstruction
* False-positive analysis
* Detection tuning
* Evidence preservation
* Configuration rollback
* Public sanitization

---

# Lessons Learned

* File integrity monitoring should begin with a narrow, controlled path.
* A changed hash confirms modified content but does not establish intent.
* User and process context are essential for strong file-change investigations.
* Wazuh may represent a rename as a deletion and a new file.
* Real-time monitoring and scheduled scans produce different response times.
* Whodata can improve attribution but requires compatible audit configuration.
* Sysmon can add process context that baseline FIM may not provide.
* Monitoring broad temporary or cache directories creates excessive noise.
* File path sensitivity should influence severity.
* A file deletion may be cleanup, normal application behavior, or evidence removal.
* Splunk can combine Wazuh and Sysmon telemetry into one timeline.
* Cleanup should preserve the monitoring configuration unless a rollback is intentional.

---

# Summary

This exercise validates the CyberLab’s ability to detect and investigate controlled file-system changes.

A complete investigation should identify:

* The file
* The action
* The user
* The process
* The original hash
* The modified hash
* The rename behavior
* The deletion event
* The Wazuh result
* The Splunk result
* The optional Sysmon result
* The event timeline
* Missing telemetry
* Appropriate severity

The exercise is complete only after the file changes are fully documented, evidence is preserved, temporary artifacts are removed, monitoring remains healthy, and detection improvements are recorded.

