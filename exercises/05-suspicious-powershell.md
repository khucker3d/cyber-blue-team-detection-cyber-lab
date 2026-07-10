
# Suspicious PowerShell

## Exercise Overview

This exercise generates controlled, harmless PowerShell activity on WIN11TARGET and validates the resulting telemetry across:

* Windows Event Viewer
* PowerShell Operational logs
* Windows Security logs
* Wazuh
* Splunk
* Optional Sysmon telemetry
* Investigation notes and evidence

The objective is to determine:

* Which user launched PowerShell
* Which process launched it
* Which command was executed
* Whether the command ran elevated
* Whether script content was logged
* Whether Wazuh and Splunk captured the activity
* Whether the available telemetry was sufficient for investigation
* Which benign administrative actions could resemble suspicious behavior

The activity is intentionally safe and does not create persistence, disable security controls, download payloads, or contact external systems.

---

## Exercise ID

```
EX-05
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* PowerShell telemetry analysis
* Windows process investigation
* Parent-child process correlation
* Command-line auditing
* Script Block Logging review
* Wazuh event validation
* Splunk SPL development
* Optional Sysmon analysis
* False-positive analysis
* Detection tuning
* Evidence preservation

---

## Authorization Boundary

This exercise must use:

* WIN11TARGET
* A dedicated non-administrative CyberLab test account
* Harmless local commands
* The VMware host-only network
* Wazuh and Splunk monitoring
* Synthetic files and directories
* An approved test time window

Do not use:

* Encoded or obfuscated payloads
* Credential-dumping commands
* Download cradles
* Remote command execution
* Security-control bypasses
* Persistence mechanisms
* Real malware
* Personal files
* External systems
* Employer or school systems

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Domain: cyberlab.example
Test account: powershell.test
Test directory: C:\CyberLab-Test\PowerShell
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Review PowerShell logging configuration
* Generate safe PowerShell telemetry
* Identify process creation events
* Identify PowerShell Operational events
* Identify the parent process
* Review the command line
* Determine whether the process ran elevated
* Compare Security, PowerShell, Wazuh, Splunk, and Sysmon data
* Distinguish benign administration from suspicious execution
* Measure ingestion delay
* Document missing telemetry and tuning opportunities

---

## Required Systems

* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on WIN11TARGET
* Splunk Universal Forwarder where configured

Optional:

* DC01 for Group Policy validation
* Sysmon on WIN11TARGET

KALI-TEST is not required.

---

## Required Account

Use a dedicated non-administrative test account:

```
powershell.test
```

An authorized administrator may be required to validate audit policy and logging configuration.

The test account should not be a member of:

* Domain Admins
* Local Administrators
* Remote Management Users
* Other privileged administrative groups

---

## Required Monitoring

Before starting, confirm:

* Windows Security logging is active
* Process creation auditing is enabled where required
* Command-line process auditing is enabled where required
* PowerShell Operational logging is enabled
* Script Block Logging is enabled where required
* Wazuh agent is active
* Splunk is running
* Relevant Windows event channels are collected
* System time is synchronized
* A stable snapshot exists

---

## Safety Notes

* Use only the commands documented in this exercise.
* Run the exercise under a standard user first.
* Do not download or execute remote content.
* Do not disable Defender, logging, or the firewall.
* Do not use encoded commands.
* Do not create scheduled tasks or services.
* Do not add exclusions.
* Do not use real credentials in commands.
* Remove temporary files during cleanup.
* Preserve evidence before changing logging configuration.

---

## Expected Event Flow

```
Test User Launches PowerShell
            |
            v
PowerShell Process Created
            |
            v
Safe Command Executed
            |
            v
Security / PowerShell / Sysmon Events
            |
            v
Wazuh and Splunk Ingestion
            |
            v
Process and Command Investigation
```

---

## Expected Windows Event IDs

The exact events depend on the configured audit policy.

| Event ID | Log                    | Description                 |
| -------: | ---------------------- | --------------------------- |
|     4688 | Security               | A new process was created   |
|     4103 | PowerShell Operational | Module logging event        |
|     4104 | PowerShell Operational | Script block logging event  |
|      400 | Windows PowerShell     | Engine started              |
|      403 | Windows PowerShell     | Engine stopped              |
|      600 | Windows PowerShell     | Provider lifecycle activity |
|        1 | Sysmon Operational     | Process creation            |
|        3 | Sysmon Operational     | Network connection          |
|       11 | Sysmon Operational     | File creation               |

Not every event is expected in every configuration.

---

## Important Fields

Review:

* User
* Security identifier
* Process name
* New process ID
* Parent process ID
* Parent process name
* Command line
* Integrity level
* Token elevation
* Script block text
* Script block ID
* Host application
* PowerShell version
* Working directory
* File path
* Timestamp
* Reporting host

---

## Investigation Questions

The final report should answer:

* Which user launched PowerShell?
* What was the parent process?
* Which PowerShell executable was used?
* What command was executed?
* Was the command line captured?
* Was script block content captured?
* Did the process run elevated?
* Was a file created or modified?
* Did Sysmon record the activity?
* Did Wazuh collect or alert on it?
* Did Splunk index the event?
* Were timestamps aligned?
* Was any unexpected network activity present?
* Which fields were missing?
* What legitimate activity could resemble the test?
* What would justify higher severity?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

Suggested pre-exercise snapshot:

```
PRE-EX05-Suspicious-PowerShell
```

A new snapshot is recommended before changing audit or Group Policy settings.

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

## Confirm Domain Connectivity

On WIN11TARGET:

```
Test-ComputerSecureChannel -Verbose
```

Expected:

```
True
```

Confirm the domain controller:

```
nltest /dsgetdc:cyberlab.example
```

---

## Confirm the Test Account Is Nonprivileged

On DC01 or from an authorized administrative session:

```
Get-ADPrincipalGroupMembership `
    -Identity "powershell.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

On WIN11TARGET, after signing in as the test account:

```
whoami
```

```
whoami /groups
```

Confirm the account is not a local administrator.

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

# Logging Configuration Validation

## Confirm Process Creation Auditing

Run from an administrator PowerShell session:

```
auditpol /get /subcategory:"Process Creation"
```

Expected success auditing:

```
Success
```

If the policy is controlled by Group Policy, update the appropriate GPO rather than making an undocumented local change.

---

## Confirm Command-Line Auditing Policy

The following policy should be enabled when command-line capture is required:

```
Computer Configuration
└── Administrative Templates
    └── System
        └── Audit Process Creation
            └── Include command line in process creation events
```

Review effective policy:

```
gpresult /h "C:\CyberLab-Test\PowerShell\gpresult.html"
```

Administrator privileges may be required.

Do not publish the unsanitized report.

---

## Confirm PowerShell Operational Log

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-PowerShell/Operational" |
Select-Object `
    LogName,
    IsEnabled,
    RecordCount,
    LogMode,
    MaximumSizeInBytes
```

Expected:

```
IsEnabled: True
```

---

## Confirm Script Block Logging Policy

Group Policy path:

```
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Windows PowerShell
            └── Turn on PowerShell Script Block Logging
```

Optional settings may include invocation start and stop logging, but additional event volume should be considered.

---

## Confirm Module Logging Policy

Group Policy path:

```
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Windows PowerShell
            └── Turn on Module Logging
```

Module names should be selected deliberately.

Using `*` increases visibility but may generate substantial noise.

---

## Apply Group Policy

After approved policy changes:

```
gpupdate /force
```

Review:

```
gpresult /r
```

A restart may be required depending on the setting.

---

## Optional Registry Validation

Run as an administrator:

```
Get-ItemProperty `
    -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
    -ErrorAction SilentlyContinue
```

```
Get-ItemProperty `
    -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" `
    -ErrorAction SilentlyContinue
```

```
Get-ItemProperty `
    -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
    -ErrorAction SilentlyContinue
```

Registry values should be treated as implementation details, not the only source of policy truth.

---

## Optional Sysmon Validation

Check for the Sysmon service:

```
Get-Service |
    Where-Object DisplayName -Match "Sysmon"
```

Confirm the log:

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-Sysmon/Operational" `
    -ErrorAction SilentlyContinue
```

Sysmon is optional for this exercise.

---

## Create the Test Directory

Run on WIN11TARGET:

```
New-Item `
    -Path "C:\CyberLab-Test\PowerShell" `
    -ItemType Directory `
    -Force
```

Confirm:

```
Get-Item "C:\CyberLab-Test\PowerShell"
```

---

## Start the Exercise Record

```
Exercise ID: EX-05
Test user: powershell.test
Endpoint: WIN11TARGET
Start time:
Process creation auditing:
Command-line auditing:
Script Block Logging:
Module Logging:
Sysmon:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Test 1: Service Enumeration

Sign in to WIN11TARGET as the non-administrative test account.

Open PowerShell normally.

Run:

```
Get-Service |
    Sort-Object Status |
    Select-Object -First 10
```

This generates benign PowerShell activity without changing the system.

Record:

```
Test 1 time:
User:
PowerShell executable:
Observed output:
```

---

## Test 2: Child Process Creation

Run:

```
Start-Process notepad.exe
```

This creates a simple parent-child process relationship:

```
powershell.exe
      |
      v
notepad.exe
```

Record the time.

Close Notepad after telemetry is confirmed.

---

## Test 3: Controlled File Creation

Run:

```
Set-Content `
    -Path "C:\CyberLab-Test\PowerShell\exercise-output.txt" `
    -Value "Authorized PowerShell telemetry exercise."
```

Then:

```
Get-Content `
    -Path "C:\CyberLab-Test\PowerShell\exercise-output.txt"
```

This provides safe file and command telemetry.

---

## Test 4: Benign System Information Query

Run:

```
Get-CimInstance Win32_OperatingSystem |
    Select-Object `
        Caption,
        Version,
        LastBootUpTime
```

This demonstrates a system-information query commonly used by administrators and scripts.

It does not establish malicious discovery behavior by itself.

---

## Optional Test 5: Script File Execution

Create a simple script:

```
@'
Get-Date
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 5
'@ |
Set-Content `
    -Path "C:\CyberLab-Test\PowerShell\safe-test.ps1"
```

Execute:

```
& "C:\CyberLab-Test\PowerShell\safe-test.ps1"
```

This test is useful for validating Script Block Logging.

Do not change the execution policy solely for this exercise.

If the script is blocked by policy, document the prevention event and continue using interactive commands.

---

## Record the Exercise Activity

```
Service enumeration time:
Notepad launch time:
File creation time:
System query time:
Optional script execution time:
PowerShell window elevated:
Unexpected errors:
```

---

# Local Process Validation

## Search for Event ID 4688

Run as an administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe|notepad\.exe"
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Event Viewer Procedure

1. Open **Event Viewer**.
2. Navigate to **Windows Logs**.
3. Select **Security**.
4. Choose **Filter Current Log**.
5. Enter event ID `4688`.
6. Use a narrow time range.
7. Locate the PowerShell and Notepad events.
8. Review the **General** tab.
9. Review the XML under **Details**.
10. Record process, parent, account, and command-line fields.

---

## Review Event ID 4688 Fields

Review:

* Creator subject
* New process ID
* New process name
* Token elevation type
* Mandatory label
* Creator process ID
* Parent process name
* Process command line
* Timestamp

Command-line data may be blank when the required policy is not enabled.

---

## Token Elevation Types

Common values include:

| Value    | General Meaning |
| -------- | --------------- |
| `%%1936` | Full token      |
| `%%1937` | Elevated token  |
| `%%1938` | Limited token   |

Interpret the value with the account, integrity level, and User Account Control context.

---

## Search for PowerShell Process Creation

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "\\powershell\.exe|\\pwsh\.exe"
}
```

---

## Search for Notepad Child Process

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "\\notepad\.exe"
}
```

Compare the parent process identifier with the PowerShell process.

---

## Record Process Findings

```
PowerShell event time:
PowerShell process path:
PowerShell process ID:
PowerShell parent process:
PowerShell command line:
User:
Token elevation:
Integrity level:
Notepad process ID:
Notepad parent process:
```

---

# PowerShell Operational Log Validation

## Search for Event ID 4104

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4104
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Search for Exercise Content

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4104
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "CyberLab-Test|Get-Service|Start-Process|Get-CimInstance"
}
```

---

## Search for Event ID 4103

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Review Event ID 4104 Fields

Review:

* Script block text
* Script block ID
* Message sequence
* Path
* User context
* Timestamp
* Host name

Long scripts may be split across multiple records.

---

## Review Event ID 4103 Fields

Review:

* Command name
* Command type
* Module
* User
* Host application
* Pipeline execution details
* Parameter bindings

Module Logging can generate a high event volume.

---

## Record PowerShell Findings

```
Event ID 4104 present:
Script block text captured:
Script path:
Script block ID:
Event ID 4103 present:
Module:
Command:
User:
Host application:
```

---

# Optional Sysmon Validation

## Search for Sysmon Event ID 1

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe|notepad\.exe"
}
```

---

## Review Sysmon Process Fields

Review:

* Process GUID
* Process ID
* Image
* File version
* Description
* Command line
* Current directory
* User
* Logon ID
* Integrity level
* Hashes
* Parent process GUID
* Parent process ID
* Parent image
* Parent command line

Sysmon usually provides richer process context than Event ID `4688`.

---

## Search for Sysmon File Creation

If the Sysmon configuration collects file creation events:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 11
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "exercise-output\.txt|safe-test\.ps1"
}
```

---

## Confirm No Unexpected Network Event

If Sysmon Event ID `3` is enabled:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 3
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

The documented exercise commands should not require external network access.

Investigate unexpected PowerShell network activity.

---

# Wazuh Validation

## Confirm Agent Health

In the Wazuh dashboard, confirm:

* WIN11TARGET is active
* Last keepalive is recent
* The time range includes the exercise
* No agent identity changed

---

## Search by Process Name

Search for:

```
powershell.exe
```

Also review:

```
pwsh.exe
notepad.exe
```

The executable depends on the installed PowerShell version.

---

## Search by Event ID

Review:

```
4688
4103
4104
```

Optional Sysmon searches:

```
1
3
11
```

Use the event channel and provider to distinguish identical numeric IDs.

---

## Search by Test Path

Search for:

```
C:\CyberLab-Test\PowerShell
```

This may locate script block, process, and file telemetry.

---

## Review Wazuh Fields

Record:

* Agent
* Event provider
* Event channel
* Event ID
* Rule
* Rule level
* User
* Process image
* Parent process
* Command line
* Script block text
* File path
* Timestamp
* MITRE ATT&CK mapping
* Compliance mappings

---

## Wazuh Validation Questions

* Did Wazuh collect the PowerShell process event?
* Did it collect the command line?
* Did it collect Event ID `4104`?
* Was the script block readable?
* Was the user parsed correctly?
* Was the parent process present?
* Was the rule severity appropriate?
* Did harmless PowerShell activity generate excessive alert severity?
* Were optional Sysmon events collected?
* Was the event received promptly?

---

## Wazuh Detection Gap Record

```
Process telemetry present:
Command line present:
Script Block Logging present:
Parent process present:
User present:
File telemetry present:
Alert present:
Rule:
Rule level:
Missing fields:
Improvement:
```

A PowerShell process alone should not automatically be classified as malicious.

---

# Splunk Validation

## Search for PowerShell Events

```spl
index=windows earliest=-30m
(
    "powershell.exe" OR
    "pwsh.exe"
)
| table
    _time
    host
    source
    sourcetype
    EventCode
    user
    ProcessName
    ParentProcessName
    CommandLine
    Message
| sort _time
```

---

## Search for Event ID 4688

```spl
index=windows EventCode=4688 earliest=-30m
| search "powershell.exe" OR "pwsh.exe"
| table
    _time
    host
    SubjectUserName
    NewProcessName
    NewProcessId
    ParentProcessName
    ProcessCommandLine
    TokenElevationType
    MandatoryLabel
| sort _time
```

Field names depend on the installed Windows add-on.

---

## Search for Event ID 4104

```spl
index=windows EventCode=4104 earliest=-30m
| search "CyberLab-Test" OR "Get-Service" OR "Start-Process" OR "Get-CimInstance"
| table
    _time
    host
    user
    ScriptBlockId
    Path
    ScriptBlockText
    Message
| sort _time
```

---

## Search for Event ID 4103

```spl
index=windows EventCode=4103 earliest=-30m
| search "Get-Service" OR "Start-Process" OR "Set-Content" OR "Get-CimInstance"
| table
    _time
    host
    user
    CommandName
    ModuleName
    HostApplication
    Message
| sort _time
```

---

## Build a PowerShell Timeline

```spl
index=windows earliest=-30m
(
    EventCode=4688 OR
    EventCode=4103 OR
    EventCode=4104
)
| search
    "powershell.exe"
    OR "pwsh.exe"
    OR "CyberLab-Test"
    OR "notepad.exe"
| eval activity=case(
    EventCode=4688 AND like(_raw, "%powershell.exe%"), "PowerShell process created",
    EventCode=4688 AND like(_raw, "%notepad.exe%"), "Child process created",
    EventCode=4103, "PowerShell module activity",
    EventCode=4104, "PowerShell script block",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    user
    ProcessName
    ParentProcessName
    CommandLine
    ScriptBlockText
| sort _time
```

---

## Optional Sysmon Process Search

```spl
index=windows EventCode=1 source="*Sysmon*" earliest=-30m
| search Image="*powershell.exe" OR Image="*pwsh.exe" OR Image="*notepad.exe"
| table
    _time
    host
    User
    Image
    CommandLine
    ParentImage
    ParentCommandLine
    IntegrityLevel
    Hashes
| sort _time
```

Adjust the `source` or `sourcetype` for the actual deployment.

---

## Correlate PowerShell and Child Process

```spl
index=windows earliest=-30m
(
    EventCode=4688 OR
    EventCode=1
)
| search
    "powershell.exe"
    OR "pwsh.exe"
    OR "notepad.exe"
| eval process=coalesce(NewProcessName, Image)
| eval parent=coalesce(ParentProcessName, ParentImage)
| eval command_line=coalesce(ProcessCommandLine, CommandLine)
| table
    _time
    host
    user
    process
    parent
    command_line
| sort _time
```

---

## Search for Test Files

```spl
index=windows earliest=-30m
(
    "exercise-output.txt"
    OR "safe-test.ps1"
)
| table
    _time
    host
    EventCode
    user
    Image
    TargetFilename
    Path
    Message
| sort _time
```

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
(
    EventCode=4688 OR
    EventCode=4103 OR
    EventCode=4104
)
| search "powershell.exe" OR "pwsh.exe" OR "CyberLab-Test"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    EventCode
| sort _time
```

---

## Splunk Validation Questions

* Which event sources recorded PowerShell activity?
* Was the command line available?
* Was the script block text available?
* Was the parent process available?
* Was the child process visible?
* Was the test user extracted correctly?
* Was the integrity or elevation level available?
* Were test files visible?
* Did any unexpected network activity appear?
* Was ingestion timely?
* Were duplicate sources present?
* Could the searches distinguish ordinary and suspicious PowerShell behavior?

---

# Investigation

## Build the Timeline

| Time     | System      |          Event ID | Activity             | User            | Analyst Note                |
| -------- | ----------- | ----------------: | -------------------- | --------------- | --------------------------- |
| `<TIME>` | WIN11TARGET |  4688 or Sysmon 1 | PowerShell started   | powershell.test | Standard-user execution     |
| `<TIME>` | WIN11TARGET |              4104 | Service query        | powershell.test | Harmless script block       |
| `<TIME>` | WIN11TARGET |  4688 or Sysmon 1 | Notepad started      | powershell.test | Child process of PowerShell |
| `<TIME>` | WIN11TARGET | 4104 or Sysmon 11 | Test file created    | powershell.test | Approved test path          |
| `<TIME>` | Wazuh       |    Alert or event | PowerShell telemetry | powershell.test | Telemetry received          |
| `<TIME>` | Splunk      |     Indexed event | Unified timeline     | powershell.test | Event searchable            |

---

## Root Cause

For this controlled exercise:

```
Authorized execution of harmless PowerShell commands by a non-administrative CyberLab test account during Exercise EX-05.
```

---

## Why PowerShell Can Be Suspicious

PowerShell is a legitimate administration platform that can also be abused for:

* Command execution
* System discovery
* Remote administration
* File modification
* Security-control modification
* Credential access
* Persistence
* Payload delivery
* Data collection

The executable alone does not determine intent.

---

## Benign Indicators

Indicators supporting a benign conclusion include:

* Known test user
* Approved test time
* Harmless command
* Known parent process
* No external network connection
* No credential access
* No persistence
* No security-control modification
* Approved test directory
* Evidence matching the exercise plan

---

## Suspicious Indicators

Indicators that would increase concern include:

* Encoded or obfuscated command line
* Hidden-window execution
* Unusual parent process
* Office application launching PowerShell
* Browser launching PowerShell
* PowerShell running as SYSTEM unexpectedly
* Download or web-request commands
* Credential-related modules
* Defender exclusions
* Scheduled-task or service creation
* Execution from temporary directories
* Unusual network connections
* Script content inconsistent with approved administration
* Log clearing or logging modification

These indicators should be investigated in context.

---

## Parent Process Analysis

Common benign parents may include:

* `explorer.exe`
* `WindowsTerminal.exe`
* `cmd.exe`
* Approved management tools

Potentially higher-risk parents may include:

* Office applications
* Browser processes
* Script hosts
* Service processes
* Unexpected remote-management tools

A suspicious parent-child relationship is a stronger signal than `powershell.exe` alone.

---

## Elevation Analysis

Determine whether the PowerShell process ran:

* As a standard user
* With a limited administrative token
* With an elevated administrative token
* As a service account
* As `SYSTEM`

Higher privilege increases potential impact but does not establish malicious intent.

---

## Command Analysis

Evaluate:

* Command purpose
* Cmdlets used
* File paths
* Parameters
* Remote destinations
* Encoding
* Obfuscation
* Child processes
* Files created
* Registry changes
* Security-control changes

The safe exercise commands should align exactly with the documented test plan.

---

## False-Positive Analysis

Legitimate PowerShell activity may come from:

* System administrators
* Help-desk scripts
* Endpoint management
* Software deployment
* Configuration management
* Backup software
* Security tools
* Login scripts
* Monitoring agents
* Scheduled maintenance
* Developer tooling

Detection logic should include context rather than alerting critically on every PowerShell process.

---

# Detection Development

## Basic PowerShell Process Search

```spl
index=windows earliest=-24h
(
    EventCode=4688 OR
    EventCode=1
)
| search
    "powershell.exe"
    OR "pwsh.exe"
| table
    _time
    host
    user
    ProcessName
    ParentProcessName
    CommandLine
| sort - _time
```

---

## PowerShell with Unusual Parent

```spl
index=windows earliest=-24h
(
    EventCode=4688 OR
    EventCode=1
)
| eval process=lower(coalesce(NewProcessName, Image))
| eval parent=lower(coalesce(ParentProcessName, ParentImage))
| where like(process, "%powershell.exe")
    OR like(process, "%pwsh.exe")
| where like(parent, "%winword.exe")
    OR like(parent, "%excel.exe")
    OR like(parent, "%powerpnt.exe")
    OR like(parent, "%outlook.exe")
    OR like(parent, "%mshta.exe")
    OR like(parent, "%wscript.exe")
    OR like(parent, "%cscript.exe")
| table _time host user process parent CommandLine
```

This search should be tuned to the environment.

---

## Potentially Risky Command-Line Patterns

```spl
index=windows earliest=-24h
(
    EventCode=4688 OR
    EventCode=4104 OR
    EventCode=1
)
| eval command_text=lower(
    coalesce(
        ProcessCommandLine,
        CommandLine,
        ScriptBlockText,
        _raw
    )
)
| where like(command_text, "%-encodedcommand%")
    OR like(command_text, "%frombase64string%")
    OR like(command_text, "%downloadstring%")
    OR like(command_text, "%invoke-webrequest%")
    OR like(command_text, "%start-bitstransfer%")
    OR like(command_text, "%add-mppreference%")
    OR like(command_text, "%set-mppreference%")
| table
    _time
    host
    user
    EventCode
    command_text
```

This is defensive detection logic only. Matches require analyst review.

---

## PowerShell Followed by Child Process

```spl
index=windows earliest=-30m
(
    EventCode=4688 OR
    EventCode=1
)
| eval process=lower(coalesce(NewProcessName, Image))
| eval parent=lower(coalesce(ParentProcessName, ParentImage))
| where like(parent, "%powershell.exe")
    OR like(parent, "%pwsh.exe")
| table
    _time
    host
    user
    process
    parent
    CommandLine
| sort _time
```

Child processes should be evaluated based on their purpose and command line.

---

## PowerShell Network Activity

When Sysmon network events are available:

```spl
index=windows EventCode=3 source="*Sysmon*" earliest=-24h
| eval image=lower(Image)
| where like(image, "%powershell.exe")
    OR like(image, "%pwsh.exe")
| table
    _time
    host
    User
    Image
    DestinationIp
    DestinationHostname
    DestinationPort
    Protocol
| sort - _time
```

Not all PowerShell network activity is malicious.

---

## Wazuh Rule Concept: PowerShell Process

```xml
<group name="windows,powershell,process_creation,">
  <rule id="<CUSTOM_RULE_ID>" level="5">
    <field name="win.system.eventID">4688</field>
    <field name="win.eventdata.newProcessName" type="pcre2">(?i)\\(powershell|pwsh)\.exe$</field>
    <description>PowerShell process created</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>
</group>
```

Validate field names against the actual decoded event.

A basic PowerShell launch should normally be informational or low severity.

---

## Wazuh Rule Concept: Higher-Risk Command

```xml
<group name="windows,powershell,suspicious_command,">
  <rule id="<CUSTOM_RULE_ID>" level="10">
    <if_sid><BASE_POWERSHELL_RULE_ID></if_sid>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)(-enc(odedcommand)?|frombase64string|downloadstring|invoke-webrequest)</field>
    <description>PowerShell command contains a higher-risk execution pattern</description>
    <mitre>
      <id>T1059.001</id>
    </mitre>
  </rule>
</group>
```

This is a conceptual public example.

Test against benign administrative behavior before deployment.

---

## MITRE ATT&CK Context

This exercise relates primarily to:

```
T1059.001 – Command and Scripting Interpreter: PowerShell
```

The system-information query may also provide controlled discovery context, but the authorized test should not be labeled malicious solely because it resembles a technique.

---

## Detection Severity Guidance

| Pattern                                           | Suggested Context           |
| ------------------------------------------------- | --------------------------- |
| Interactive PowerShell from known administrator   | Informational or Low        |
| Standard user launches PowerShell for known task  | Low                         |
| PowerShell launches ordinary child process        | Low or Medium               |
| PowerShell with unusual parent process            | Medium or High              |
| Encoded or obfuscated command                     | High investigation priority |
| PowerShell creates persistence                    | High                        |
| PowerShell changes endpoint-protection settings   | High                        |
| PowerShell makes unusual external connection      | High                        |
| PowerShell runs as SYSTEM unexpectedly            | High                        |
| PowerShell activity followed by credential access | Critical review             |

Severity should reflect context, impact, and confidence.

---

# Evidence Collection

## Evidence

Preserve:

* Audit policy output
* Group Policy result
* Event ID `4688`
* Event ID `4103`
* Event ID `4104`
* Optional Sysmon Event ID `1`
* Parent-child process relationship
* Test file properties
* Wazuh result
* Splunk timeline
* Ingestion-delay result
* No-unexpected-network validation
* Cleanup confirmation
* Investigation notes

---

## Export Security Events

Operational example:

```
wevtutil epl Security "<EVIDENCE_PATH>\WIN11TARGET-Security.evtx"
```

Store raw logs privately.

---

## Export PowerShell Operational Events

```
wevtutil epl `
    "Microsoft-Windows-PowerShell/Operational" `
    "<EVIDENCE_PATH>\WIN11TARGET-PowerShell-Operational.evtx"
```

---

## Export Sysmon Events

When Sysmon is installed:

```
wevtutil epl `
    "Microsoft-Windows-Sysmon/Operational" `
    "<EVIDENCE_PATH>\WIN11TARGET-Sysmon-Operational.evtx"
```

---

## Export Splunk Results

Export the narrow exercise timeline as:

* CSV
* JSON
* Raw events

Review every field before publication.

---

## Hash Evidence

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
Exercise: EX-05
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

Before deleting test files:

1. Capture the process events.
2. Capture Script Block Logging events.
3. Save Wazuh evidence.
4. Save the Splunk timeline.
5. Record file properties.
6. Confirm no unexpected network activity.
7. Close the PowerShell session.
8. Close Notepad.

---

## Remove the Test Files

```
Remove-Item `
    -Path "C:\CyberLab-Test\PowerShell\exercise-output.txt" `
    -Force `
    -ErrorAction SilentlyContinue
```

```
Remove-Item `
    -Path "C:\CyberLab-Test\PowerShell\safe-test.ps1" `
    -Force `
    -ErrorAction SilentlyContinue
```

---

## Remove the Test Directory

When empty:

```
Remove-Item `
    -Path "C:\CyberLab-Test\PowerShell" `
    -Force `
    -ErrorAction SilentlyContinue
```

Retain the directory when it is an approved reusable exercise location.

---

## Confirm No PowerShell Test Process Remains

```
Get-Process `
    powershell,
    pwsh,
    notepad `
    -ErrorAction SilentlyContinue
```

Do not terminate unrelated administrative PowerShell sessions.

---

## Confirm Monitoring Health

Check Wazuh:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Check Splunk:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Generate a harmless validation event if needed.

---

## Final State Record

```
PowerShell tests completed:
Notepad closed:
Test files removed:
Unexpected network activity:
Wazuh healthy:
Splunk healthy:
Evidence preserved:
Logging configuration unchanged:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* A non-administrative test account was used.
* Only approved harmless commands were executed.
* Event ID `4688` was reviewed where enabled.
* PowerShell Operational events were reviewed.
* The command line was assessed.
* The parent process was identified.
* The child process was identified.
* The execution privilege was assessed.
* Wazuh was reviewed.
* Splunk was reviewed.
* Optional Sysmon telemetry was reviewed where available.
* Ingestion delay was measured.
* False positives were considered.
* Test files and processes were cleaned up.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                | Result                       |
| ----------------------- | ---------------------------- |
| Process creation event  | Pass / Fail / Not Configured |
| Command line captured   | Pass / Partial / Fail        |
| Parent process captured | Pass / Partial / Fail        |
| Script Block Logging    | Pass / Fail / Not Configured |
| Module Logging          | Pass / Fail / Not Configured |
| User captured           | Pass / Partial / Fail        |
| Elevation context       | Pass / Partial / Fail        |
| Sysmon telemetry        | Pass / Fail / Not Installed  |
| Wazuh ingestion         | Pass / Fail                  |
| Wazuh alert quality     | Pass / Needs Tuning / Fail   |
| Splunk ingestion        | Pass / Fail                  |
| Splunk field quality    | Pass / Needs Tuning / Fail   |
| Timestamp accuracy      | Pass / Needs Review / Fail   |
| Cleanup                 | Complete / Incomplete        |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Command line
* Parent process
* Script block text
* User
* Elevation level
* Integrity level
* Process hashes
* Working directory
* File-creation telemetry
* Network telemetry
* PowerShell version
* Source process
* Clear detection severity
* Correlation across event channels

---

## Improvements

Potential improvements include:

* Enable process command-line auditing
* Enable Script Block Logging
* Tune Module Logging
* Deploy Sysmon
* Collect Sysmon process and network events
* Normalize process and parent fields
* Create a PowerShell activity dashboard
* Add unusual-parent detection
* Add risky-command pattern detection
* Add PowerShell network-connection detection
* Add allowlists for approved management tools carefully
* Create a PowerShell investigation runbook

---

# Troubleshooting

## Event ID 4688 Does Not Appear

Check:

```
auditpol /get /subcategory:"Process Creation"
```

Review:

* Success auditing
* Group Policy
* Security log time range
* Event retention
* Correct endpoint
* Administrative access to the log

Apply approved Group Policy changes and retest.

---

## Command Line Is Missing from Event ID 4688

Check the policy:

```
Include command line in process creation events
```

Then:

```
gpupdate /force
```

Confirm effective policy with:

```
gpresult /r
```

Command-line logging may expose sensitive data. Avoid placing secrets directly on command lines.

---

## Event ID 4104 Does Not Appear

Check:

* Script Block Logging policy
* PowerShell Operational log enabled
* Group Policy application
* Event time range
* Test command
* PowerShell version
* Event log capacity

Interactive commands may generate different telemetry than script-file execution.

---

## Event ID 4103 Does Not Appear

Check:

* Module Logging policy
* Selected module names
* Group Policy application
* Event channel
* Command type
* Time range

Module Logging is not required for every PowerShell exercise.

---

## Script Execution Is Blocked

Do not weaken execution policy solely to complete the exercise.

Instead:

* Document the prevention
* Continue with interactive commands
* Validate the security event
* Review application-control policy
* Treat prevention as a useful finding

---

## Wazuh Does Not Show PowerShell Events

Check:

* Agent status
* Security log collection
* PowerShell Operational channel collection
* Sysmon channel collection
* Decoder
* Rule level
* Dashboard time range
* Agent filter
* Indexer health

Confirm the event exists locally first.

---

## Splunk Does Not Show PowerShell Events

Check:

* Forwarder service
* Security input
* PowerShell Operational input
* Sysmon input
* Receiver
* Target index
* Host
* Sourcetype
* Time range
* Field names

Begin with:

```spl
index=windows host=WIN11TARGET earliest=-30m
| stats count by source sourcetype EventCode
```

---

## Parent Process Is Missing

Possible causes include:

* Event source limitation
* Missing Sysmon
* Field extraction problem
* Incomplete process audit data
* Different PowerShell launch method

Review the raw event XML and correlate process identifiers.

---

## PowerShell Activity Produces Too Many Alerts

Possible causes include:

* Every PowerShell process assigned high severity
* Broad command-pattern rules
* Module Logging volume
* Approved administration not identified
* Duplicate event sources
* Sysmon and Security logs both ingested

Tune using:

* Parent process
* User role
* Host role
* Command content
* Execution path
* Network behavior
* Frequency
* Approved management sources

Do not suppress all PowerShell activity.

---

## Test Files Cannot Be Removed

Check:

* Open file handle
* Active Notepad session
* PowerShell session still using the path
* Permissions
* Antivirus quarantine
* Incorrect path

Close applications and confirm the file owner before forcing removal.

---

# Public Sanitization

Remove or replace:

* Operational domain name
* Real usernames
* Internal IP addresses
* Personal file paths
* Security identifiers
* Exact process command lines containing secrets
* Wazuh agent identifiers
* Splunk internal URLs
* Raw event XML
* Hashes tied to operational files where sensitive
* Evidence storage paths
* Group Policy reports containing internal data

Use placeholders such as:

```
<TEST_ACCOUNT>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<DOMAIN_NAME>
<PROCESS_PATH>
<PARENT_PROCESS>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4688`
* Event ID `4104`
* Parent-child process relationship
* Optional Sysmon Event ID `1`
* Wazuh PowerShell event
* Splunk PowerShell timeline
* Test file creation event
* Detection-quality table

Before publishing:

1. Crop unrelated content.
2. Remove real usernames.
3. Remove domain values.
4. Remove internal addresses.
5. Remove SIDs.
6. Remove sensitive command-line arguments.
7. Hide browser tabs and bookmarks.
8. Permanently redact sensitive values.
9. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
Harmless PowerShell commands were executed by a non-administrative CyberLab test account during Exercise EX-05. Process creation, command-line, script block, child-process, and optional Sysmon telemetry were reviewed locally and correlated through Wazuh and Splunk. The activity was classified as authorized test behavior, and telemetry gaps were documented.
```

## Findings

```
Test account:

Endpoint:

PowerShell executable:

Parent process:

Commands executed:

Process event:

Script block event:

Module event:

Sysmon result:

Elevation level:

Child process:

Files created:

Unexpected network activity:

Wazuh result:

Splunk result:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized execution of benign PowerShell commands during CyberLab Exercise EX-05.
```

## Resolution

```
The test PowerShell and Notepad processes were closed, temporary files were removed, monitoring health was confirmed, and evidence was preserved for analysis.
```

## Recommendations

```
- Enable command-line process auditing.
- Enable and validate Script Block Logging.
- Deploy and tune Sysmon.
- Detect unusual PowerShell parent processes.
- Correlate process, script block, file, and network telemetry.
- Create a PowerShell investigation runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshot exists
* Test account nonprivileged
* WIN11TARGET healthy
* Wazuh healthy
* Splunk healthy
* Process auditing reviewed
* Command-line auditing reviewed
* PowerShell Operational log reviewed
* Script Block Logging reviewed
* Optional Sysmon reviewed
* Test directory created
* Start time recorded

## Exercise

* Service query executed
* Notepad launched
* Test file created
* System query executed
* Optional script executed or prevention documented
* No prohibited command used
* No unexpected external connection observed

## Investigation

* Event ID 4688 reviewed
* Parent process identified
* Child process identified
* Command line reviewed
* Elevation context reviewed
* Event ID 4104 reviewed
* Event ID 4103 reviewed where configured
* Sysmon reviewed where installed
* Wazuh reviewed
* Splunk reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Notepad closed
* PowerShell test session closed
* Test files removed
* Test directory removed or retained intentionally
* Wazuh healthy
* Splunk healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* PowerShell telemetry analysis
* Windows process creation auditing
* Script Block Logging
* Module Logging
* Parent-child process analysis
* Privilege and integrity assessment
* Optional Sysmon analysis
* Wazuh validation
* Splunk SPL
* Timeline reconstruction
* File-event correlation
* Network-activity review
* False-positive analysis
* Detection tuning
* Evidence handling
* Public sanitization

---

# Lessons Learned

* PowerShell is a legitimate administration tool and should not be treated as malicious by default.
* Process creation auditing and Script Block Logging provide different types of evidence.
* Command-line data may be missing unless the required policy is enabled.
* Parent process context can be more useful than the PowerShell executable alone.
* Sysmon provides richer process context than standard process auditing.
* Script Block Logging can reveal command content even when process telemetry is limited.
* Module Logging can provide useful detail but may generate substantial noise.
* PowerShell launching a child process is not automatically suspicious.
* Unexpected network activity significantly changes the investigation context.
* Higher-risk patterns require correlation with user, parent, privilege, host, and behavior.
* Harmless test activity should still be investigated using the same structured workflow.
* Cleanup must remove temporary files without disabling the logging controls being tested.

---

# Summary

This exercise validates the CyberLab’s ability to observe and investigate PowerShell activity.

A complete investigation should identify:

* The user
* The PowerShell process
* The parent process
* The command line
* The script block
* The elevation level
* The child process
* The files created
* Any network activity
* The Wazuh result
* The Splunk result
* The event timeline
* Missing telemetry
* Appropriate detection severity

The exercise is complete only after the activity is classified, evidence is preserved, temporary artifacts are removed, monitoring remains healthy, and detection improvements are documented.
