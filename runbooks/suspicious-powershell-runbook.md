# Suspicious PowerShell Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating suspicious or unusual PowerShell activity on Windows systems.

It applies to alerts involving:

* PowerShell process execution
* Encoded or obfuscated commands
* PowerShell launched by an unusual parent process
* PowerShell downloading or transferring content
* PowerShell creating files or changing configuration
* PowerShell spawning child processes
* PowerShell running with elevated privileges
* PowerShell execution from unusual directories
* Script Block Logging alerts
* Wazuh or Splunk detections involving PowerShell
* Microsoft Defender alerts associated with PowerShell
* Sysmon process or network events involving PowerShell

The runbook helps the analyst determine:

* Who executed PowerShell
* Which system executed it
* Which parent process launched it
* What command or script ran
* Whether the command was encoded or obfuscated
* Whether files, processes, accounts, services, tasks, or network connections were created
* Whether the activity was authorized
* Whether the execution represents administration, automation, troubleshooting, malware, persistence, defense evasion, or another security concern
* Whether containment or escalation is required

---

## Runbook ID

```
RB-05
```

---

## Related Exercise

```
EX-05 – Suspicious PowerShell
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

* `powershell.exe`
* `pwsh.exe`
* PowerShell scripts
* PowerShell remoting
* PowerShell launched through another application
* PowerShell-based administrative tools
* PowerShell module activity
* Encoded or obfuscated commands
* PowerShell network activity
* PowerShell-related file and process changes

This runbook does not authorize:

* Executing unknown commands to determine what they do
* Decoding and running suspicious payloads
* Connecting to untrusted infrastructure
* Disabling logging or endpoint protection
* Deleting scripts before evidence is preserved
* Testing suspicious commands outside the isolated CyberLab
* Uploading sensitive scripts or event logs to public services

---

## Primary Data Sources

Use the available sources in this order:

1. PowerShell Operational logs
2. Windows process-creation events
3. Sysmon process and network events
4. Microsoft Defender events
5. Windows Security logs
6. Wazuh events and alerts
7. Splunk indexed events
8. File-system and FIM events
9. Scheduled-task and service events
10. Network and firewall logs
11. User and account context
12. Change-management or administrative records

---

## Key Windows and Sysmon Event IDs

| Event ID | Source                 | Description                    |
| -------: | ---------------------- | ------------------------------ |
|     4103 | PowerShell Operational | Module logging                 |
|     4104 | PowerShell Operational | Script block logging           |
|      400 | Windows PowerShell     | Engine started                 |
|      403 | Windows PowerShell     | Engine stopped                 |
|      600 | Windows PowerShell     | Provider activity              |
|     4688 | Security               | A new process was created      |
|     4624 | Security               | Successful logon               |
|     4648 | Security               | Explicit credentials used      |
|     4672 | Security               | Special privileges assigned    |
|     4698 | Security               | Scheduled task created         |
|     4697 | Security               | Service installed              |
|     7045 | System                 | Service installed              |
|        1 | Sysmon                 | Process creation               |
|        3 | Sysmon                 | Network connection             |
|       11 | Sysmon                 | File creation                  |
|       13 | Sysmon                 | Registry value set             |
|       22 | Sysmon                 | DNS query                      |
|       23 | Sysmon                 | File deletion archived         |
|       26 | Sysmon                 | File deletion detected         |
|     1116 | Defender               | Threat detected                |
|     1117 | Defender               | Remediation action taken       |
|     5001 | Defender               | Real-time protection disabled  |
|     5007 | Defender               | Defender configuration changed |

Not every system will produce every event.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh PowerShell alert
* Splunk correlation search
* Microsoft Defender alert
* Sysmon alert
* Windows Event Viewer review
* File integrity alert
* Suspicious process alert
* Network connection alert
* Scheduled-task alert
* User or administrator report
* Detection exercise

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Reporting host:
User:
PowerShell executable:
Process ID:
Parent process:
Parent process ID:
Command line:
Script path:
Integrity level:
Source address:
Destination address:
Alert severity:
Analyst:
```

Do not assume the command line is complete or correctly parsed.

---

## Initial Analyst Questions

Determine immediately:

* Is the user authorized to run PowerShell?
* Is the process elevated?
* Is the parent process expected?
* Is the command line visible?
* Is the command encoded or obfuscated?
* Did PowerShell access the network?
* Did it create or modify files?
* Did it create accounts, services, or scheduled tasks?
* Did it launch another process?
* Did Microsoft Defender detect anything?
* Is the activity still running?
* Is there an approved script, maintenance task, or exercise?
* Is the same behavior occurring on other systems?

---

# Severity Classification

## Informational

Use informational severity when:

* The activity is part of an approved CyberLab exercise
* The command is harmless and documented
* The user and source are expected
* No elevated or persistent changes occurred
* No suspicious network connection occurred

---

## Low

Use low severity when:

* A known administrator ran a common diagnostic command
* The parent process is expected
* The script path is approved
* The command is readable and limited in scope
* No suspicious child process or network activity exists

---

## Medium

Use medium severity when:

* The command is unusual but not clearly malicious
* PowerShell is launched by an uncommon parent process
* The script runs from a temporary or user-writable directory
* The command is partially encoded
* A remote system or administrative share is involved
* The user does not normally use PowerShell
* The activity occurs outside expected hours

---

## High

Use high severity when:

* The command is heavily encoded or obfuscated
* PowerShell downloads or executes remote content
* PowerShell launches from an Office application, browser, script host, or unknown executable
* PowerShell creates persistence
* PowerShell modifies security controls
* PowerShell accesses credentials
* PowerShell runs as SYSTEM unexpectedly
* PowerShell is followed by remote access or lateral movement
* Microsoft Defender detects a threat
* The source is unknown or unauthorized

---

## Critical

Use critical severity when:

* PowerShell activity is part of confirmed malware execution
* Credentials or secrets are stolen
* Domain-wide changes occur
* Multiple systems show the same malicious behavior
* Security controls are disabled
* Ransomware or destructive activity follows
* Privileged persistence is established
* A domain administrator or equivalent account is compromised

---

# Initial Triage

## Step 1: Confirm the Event Time

Record the exact event time and timezone from:

* Endpoint
* Wazuh
* Splunk
* Defender
* Analyst workstation

Use absolute timestamps.

```
YYYY-MM-DD HH:MM:SS Timezone
```

---

## Step 2: Confirm the Process Exists or Existed

On the endpoint, run as an authorized administrator:

```
Get-CimInstance Win32_Process |
    Where-Object {
        $_.Name -match "powershell|pwsh"
    } |
Select-Object `
    ProcessId,
    ParentProcessId,
    Name,
    ExecutablePath,
    CommandLine,
    CreationDate
```

If the process is still running, do not terminate it before preserving volatile evidence unless immediate containment is required.

---

## Step 3: Confirm Local PowerShell Events

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    LevelDisplayName,
    Message
```

Search by user, command, or script name:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<SEARCH_TERM>"
}
```

---

## Step 4: Confirm Process-Creation Telemetry

### Windows Security Event ID 4688

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

### Sysmon Event ID 1

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

---

## Step 5: Record Process Context

Record:

```
PowerShell executable:
Executable path:
Process ID:
Process GUID:
Parent process:
Parent process ID:
Parent process GUID:
User:
Integrity level:
Command line:
Current directory:
Hashes:
Signature:
Start time:
End time:
```

---

# Command-Line Analysis

## Review the Executable

Expected paths commonly include:

```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
C:\Program Files\PowerShell\7\pwsh.exe
```

Increase concern when:

* The executable uses a lookalike filename
* It runs from a user profile
* It runs from a temporary directory
* It runs from a network share
* The executable is unsigned or unexpectedly modified

---

## Review Common Command-Line Options

| Option                    | Meaning                         | Investigation Context                                        |
| ------------------------- | ------------------------------- | ------------------------------------------------------------ |
| `-NoProfile`              | Does not load profiles          | Common in automation and malicious execution                 |
| `-NonInteractive`         | No interactive prompt           | Common in scripts and automation                             |
| `-WindowStyle Hidden`     | Hides the window                | Suspicious when combined with other indicators               |
| `-ExecutionPolicy Bypass` | Bypasses policy for the process | Often legitimate in deployment tools, but high-value context |
| `-EncodedCommand`         | Executes Base64-encoded command | Requires decoding and review                                 |
| `-Command`                | Runs a command                  | Common                                                       |
| `-File`                   | Runs a script file              | Review script path and hash                                  |
| `-NoLogo`                 | Suppresses banner               | Low-risk by itself                                           |
| `-Sta`                    | Single-threaded apartment       | Context-dependent                                            |
| `-Mta`                    | Multithreaded apartment         | Context-dependent                                            |

No single option proves malicious activity.

---

## Suspicious Command Patterns

Increase concern when the command includes:

* `EncodedCommand`
* `FromBase64String`
* `Invoke-Expression`
* `IEX`
* `DownloadString`
* `DownloadFile`
* `Invoke-WebRequest`
* `curl`
* `wget`
* `WebClient`
* `HttpClient`
* `Start-BitsTransfer`
* `Net.WebClient`
* `Reflection.Assembly`
* `Add-MpPreference`
* `Set-MpPreference`
* `DisableRealtimeMonitoring`
* `New-ScheduledTask`
* `Register-ScheduledTask`
* `New-Service`
* `sc.exe`
* `Add-LocalGroupMember`
* `Add-ADGroupMember`
* `New-ADUser`
* `Invoke-Command`
* `Enter-PSSession`
* `New-PSSession`
* `Get-Credential`
* `ConvertTo-SecureString`
* `Export-Clixml`
* `reg.exe`
* `rundll32.exe`
* `mshta.exe`

These patterns require context and are not automatically malicious.

---

# Safe Decoding and Deobfuscation

## Preserve the Original Command

Save the exact original command before transformation.

```
Original command:
Source event:
Collection time:
SHA-256 of exported evidence:
```

---

## Do Not Execute the Decoded Result

Decode text only.

Do not:

* Pipe decoded content to `Invoke-Expression`
* Save and run the decoded script
* Connect to referenced infrastructure
* Execute extracted commands
* Open suspicious attachments

---

## Decode a Standard PowerShell EncodedCommand

PowerShell commonly uses UTF-16LE for `-EncodedCommand`.

Use an isolated analysis session:

```
$Encoded = "<BASE64_VALUE>"

[System.Text.Encoding]::Unicode.GetString(
    [System.Convert]::FromBase64String($Encoded)
)
```

Record the decoded text as evidence.

---

## Decode UTF-8 Base64 When Required

```
$Encoded = "<BASE64_VALUE>"

[System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String($Encoded)
)
```

Use UTF-8 only when evidence indicates that encoding.

---

## Decode Without Running PowerShell

On Linux:

```
printf '%s' '<BASE64_VALUE>' |
    base64 --decode
```

For UTF-16LE content:

```
printf '%s' '<BASE64_VALUE>' |
    base64 --decode |
    iconv -f UTF-16LE -t UTF-8
```

---

## Review Obfuscation Indicators

Look for:

* String concatenation
* Character-code conversion
* Reversed strings
* Excessive escape characters
* Environment-variable reconstruction
* Aliases replacing full command names
* Backticks inserted into keywords
* Compressed content
* Nested Base64
* Dynamic method invocation
* Reflection
* Deliberately meaningless variable names

Document each transformation separately.

---

# Parent-Process Investigation

## Common Expected Parents

Possible legitimate parent processes include:

* `explorer.exe`
* `cmd.exe`
* `WindowsTerminal.exe`
* `powershell.exe`
* `pwsh.exe`
* Approved management agent
* Approved deployment tool
* Scheduled-task engine
* Authorized service process

---

## High-Risk Parent Processes

Increase concern when PowerShell is launched by:

* `winword.exe`
* `excel.exe`
* `powerpnt.exe`
* `outlook.exe`
* `mshta.exe`
* `wscript.exe`
* `cscript.exe`
* `rundll32.exe`
* `regsvr32.exe`
* Browser executable
* PDF reader
* Archive utility
* Unknown executable
* User-writable binary
* Web server process

---

## Validate the Parent Process

```
Get-Process `
    -Id <PARENT_PROCESS_ID> `
    -ErrorAction SilentlyContinue
```

Historical parent information should come from Event ID `4688` or Sysmon Event ID `1`, because process IDs may be reused.

Record:

```
Parent executable:
Parent command line:
Parent user:
Parent hash:
Parent signature:
Parent start time:
Parent’s parent process:
```

---

## Build the Process Tree

Splunk example:

```spl
index=windows earliest=-1h
(
    EventCode=1 OR
    EventCode=4688
)
| search
    Image="*powershell.exe"
    OR Image="*pwsh.exe"
    OR NewProcessName="*powershell.exe"
    OR NewProcessName="*pwsh.exe"
| table
    _time
    host
    User
    ParentImage
    ParentCommandLine
    Image
    CommandLine
    ProcessId
    ParentProcessId
    ProcessGuid
    ParentProcessGuid
```

---

# User and Privilege Context

## Identify the User

Review:

* Domain or local identity
* Standard or administrative account
* Interactive or service context
* Recent password changes
* Recent lockout
* Recent privileged group addition
* Normal workstation
* Normal PowerShell usage

---

## Review Account Properties

For a domain user:

```
Get-ADUser `
    -Identity "<USER_ACCOUNT>" `
    -Properties `
        Enabled,
        LockedOut,
        LastLogonDate,
        PasswordLastSet,
        MemberOf |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    LastLogonDate,
    PasswordLastSet,
    MemberOf
```

---

## Review Successful Logon Context

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624, 4672
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<USER_ACCOUNT>"
}
```

Determine:

* Logon type
* Source system
* Source address
* Logon ID
* Whether special privileges were assigned

---

## Review Integrity Level

Common integrity levels include:

* Low
* Medium
* High
* System

Increase concern when:

* A standard user unexpectedly launches high-integrity PowerShell
* PowerShell runs as SYSTEM without an approved service or management tool
* Privilege follows a recent group-membership change

---

# Script and File Investigation

## Identify the Script Path

When `-File` or a script path is present, record:

```
Full path:
Filename:
Extension:
Creation time:
Modification time:
Owner:
Size:
SHA-256:
Signature:
Origin:
```

---

## Review File Properties

```
Get-Item `
    -Path "<SCRIPT_PATH>" |
Select-Object `
    FullName,
    Length,
    CreationTime,
    LastWriteTime,
    Attributes
```

---

## Hash the Script

```
Get-FileHash `
    -Path "<SCRIPT_PATH>" `
    -Algorithm SHA256
```

---

## Review Digital Signature

```
Get-AuthenticodeSignature `
    -FilePath "<SCRIPT_PATH>"
```

An unsigned script is not automatically malicious, but the signature adds context.

---

## Review Alternate Data Streams

```
Get-Item `
    -Path "<SCRIPT_PATH>" `
    -Stream *
```

A `Zone.Identifier` stream may indicate internet origin.

---

## Read the Script Safely

Use a text viewer.

```
Get-Content `
    -Path "<SCRIPT_PATH>" `
    -Raw
```

Do not dot-source, import, or execute the script.

---

## High-Risk File Locations

Increase concern when scripts execute from:

* `%TEMP%`
* `%APPDATA%`
* `%LOCALAPPDATA%`
* Downloads
* Public user directories
* Recycle Bin
* Browser cache
* Network shares
* Startup folders
* Newly mounted media
* Hidden directories

---

## Search for Related File Events

### Wazuh FIM

Search by script path or filename.

### Sysmon

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 11, 23, 26
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<SCRIPT_NAME>|<SCRIPT_DIRECTORY>"
}
```

---

# Network Investigation

## Search Sysmon Network Events

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 3
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

Record:

```
Source address:
Source port:
Destination address:
Destination port:
Protocol:
Initiated:
Process ID:
Process GUID:
User:
```

---

## Search DNS Queries

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 22
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

---

## Review Active Connections

When the process is still running:

```
Get-NetTCPConnection `
    -OwningProcess <PROCESS_ID> `
    -ErrorAction SilentlyContinue
```

---

## Network Indicators of Concern

Increase severity when:

* Destination is unknown
* Destination is outside the CyberLab
* The connection uses an unusual port
* A script downloads executable content
* PowerShell contacts several destinations
* DNS requests use unusual or newly observed domains
* Network activity is followed by file creation or process execution
* The destination is associated with another alert

Do not access or browse an unknown destination during triage.

---

# Child-Process Investigation

## Search for Child Processes

Using Sysmon:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<POWERSHELL_PROCESS_GUID>|<POWERSHELL_PROCESS_ID>"
}
```

---

## High-Risk Child Processes

Increase concern when PowerShell launches:

* `cmd.exe`
* `rundll32.exe`
* `regsvr32.exe`
* `mshta.exe`
* `certutil.exe`
* `bitsadmin.exe`
* `schtasks.exe`
* `sc.exe`
* `net.exe`
* `net1.exe`
* `wmic.exe`
* `vssadmin.exe`
* `bcdedit.exe`
* `wevtutil.exe`
* `whoami.exe`
* `nltest.exe`
* `dsquery.exe`
* `procdump.exe`
* Unknown executable

Many of these tools are legitimate administrative utilities. Context is required.

---

## Review Child Process Details

Record:

```
Child executable:
Child command line:
Child process ID:
Child process GUID:
User:
Integrity level:
Hash:
Signature:
Start time:
Network activity:
Files created:
```

---

# Persistence Investigation

## Scheduled Tasks

Search for task creation:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4698
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "powershell|pwsh|<SCRIPT_NAME>"
}
```

Review current tasks:

```
Get-ScheduledTask |
    Where-Object {
        $_.Actions.Execute -match "powershell|pwsh"
        -or $_.Actions.Arguments -match "<SCRIPT_NAME>"
    }
```

---

## Services

Search for service installation:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "System"
        Id = 7045
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "powershell|pwsh|<SCRIPT_NAME>"
}
```

Review services:

```
Get-CimInstance Win32_Service |
    Where-Object {
        $_.PathName -match "powershell|pwsh|<SCRIPT_NAME>"
    } |
Select-Object `
    Name,
    DisplayName,
    State,
    StartMode,
    StartName,
    PathName
```

---

## Registry Persistence

Review Sysmon Event ID `13`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 13
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "Run|RunOnce|Winlogon|PowerShell|<SCRIPT_NAME>"
}
```

---

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

## PowerShell Profiles

Review profile locations:

```
$PROFILE | Format-List *
```

Common profile paths may contain commands that execute whenever PowerShell starts.

Do not alter profile files before preserving them.

---

# Security-Control Investigation

## Review Microsoft Defender Events

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Windows Defender/Operational"
        Id = 1116, 1117, 5001, 5004, 5007
        StartTime = (Get-Date).AddHours(-4)
    } |
Select-Object `
    TimeCreated,
    Id,
    LevelDisplayName,
    Message
```

---

## Review Defender Status

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

## Review Defender Preferences

Run as an authorized administrator:

```
Get-MpPreference |
    Select-Object `
        DisableRealtimeMonitoring,
        DisableBehaviorMonitoring,
        DisableScriptScanning,
        ExclusionPath,
        ExclusionProcess,
        ExclusionExtension
```

Do not publish operational exclusion details.

---

## High-Risk Defender Changes

Increase severity when PowerShell:

* Disables real-time protection
* Disables script scanning
* Adds exclusions
* Changes cloud-delivered protection
* Changes sample-submission settings
* Stops Defender services
* Deletes Defender history
* Attempts to bypass AMSI

---

# Wazuh Investigation

## Search by Executable

Search for:

```
powershell.exe
pwsh.exe
```

---

## Search by User

```
<USER_ACCOUNT>
```

---

## Search by Host

```
<WINDOWS_ENDPOINT>
```

---

## Search by Event ID

Review:

```
4103
4104
4688
1
3
11
13
22
1116
1117
5001
5007
4698
7045
```

---

## Review Wazuh Fields

Record:

```
Agent:
Rule ID:
Rule description:
Rule level:
Event ID:
User:
Image:
Parent image:
Command line:
Parent command line:
Process ID:
Process GUID:
Integrity level:
Script path:
Destination address:
Destination port:
File created:
Defender result:
Timestamp:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Was the full command line collected?
* Was Script Block Logging available?
* Was the parent process visible?
* Was the user parsed correctly?
* Was elevation visible?
* Was network activity visible?
* Were file changes visible?
* Were child processes visible?
* Did Defender generate a related alert?
* Did Wazuh correlate the related events?
* Was the severity appropriate?
* Were any fields truncated?

---

# Splunk Investigation

## Basic PowerShell Search

```spl
index=windows earliest=-24h
(
    ProcessName="*powershell.exe"
    OR ProcessName="*pwsh.exe"
    OR Image="*powershell.exe"
    OR Image="*pwsh.exe"
    OR NewProcessName="*powershell.exe"
    OR NewProcessName="*pwsh.exe"
    OR EventCode=4103
    OR EventCode=4104
)
| table
    _time
    host
    EventCode
    User
    ParentImage
    Image
    CommandLine
    ScriptBlockText
    Message
| sort _time
```

---

## Search Suspicious Command Patterns

```spl
index=windows earliest=-24h
(
    EventCode=1 OR
    EventCode=4688 OR
    EventCode=4103 OR
    EventCode=4104
)
| eval command=lower(
    coalesce(
        CommandLine,
        Process_Command_Line,
        ScriptBlockText,
        Message
    )
)
| where like(command, "%encodedcommand%")
    OR like(command, "%frombase64string%")
    OR like(command, "%invoke-expression%")
    OR like(command, "%downloadstring%")
    OR like(command, "%downloadfile%")
    OR like(command, "%invoke-webrequest%")
    OR like(command, "%start-bitstransfer%")
    OR like(command, "%disablebookrealtimemonitoring%")
| table
    _time
    host
    User
    ParentImage
    Image
    command
| sort _time
```

Correct any field or spelling differences for the deployment.

---

## Search Unusual Parent Processes

```spl
index=windows earliest=-24h
(
    EventCode=1 OR
    EventCode=4688
)
| eval image=lower(coalesce(Image, NewProcessName))
| eval parent=lower(coalesce(ParentImage, Creator_Process_Name))
| where like(image, "%powershell.exe")
    OR like(image, "%pwsh.exe")
| where like(parent, "%winword.exe")
    OR like(parent, "%excel.exe")
    OR like(parent, "%powerpnt.exe")
    OR like(parent, "%outlook.exe")
    OR like(parent, "%mshta.exe")
    OR like(parent, "%wscript.exe")
    OR like(parent, "%cscript.exe")
    OR like(parent, "%rundll32.exe")
| table
    _time
    host
    User
    parent
    image
    CommandLine
    ParentCommandLine
| sort _time
```

---

## Search PowerShell Network Activity

```spl
index=windows EventCode=3 earliest=-24h
| search
    Image="*powershell.exe"
    OR Image="*pwsh.exe"
| table
    _time
    host
    User
    Image
    ProcessId
    ProcessGuid
    SourceIp
    SourcePort
    DestinationIp
    DestinationPort
    Protocol
    Initiated
| sort _time
```

---

## Search PowerShell DNS Activity

```spl
index=windows EventCode=22 earliest=-24h
| search
    Image="*powershell.exe"
    OR Image="*pwsh.exe"
| table
    _time
    host
    User
    Image
    ProcessId
    ProcessGuid
    QueryName
    QueryStatus
    QueryResults
| sort _time
```

---

## Search PowerShell File Creation

```spl
index=windows EventCode=11 earliest=-24h
| search
    Image="*powershell.exe"
    OR Image="*pwsh.exe"
| table
    _time
    host
    User
    Image
    ProcessId
    ProcessGuid
    TargetFilename
    CreationUtcTime
| sort _time
```

---

## Search PowerShell Child Processes

```spl
index=windows EventCode=1 earliest=-24h
| eval parent=lower(ParentImage)
| where like(parent, "%powershell.exe")
    OR like(parent, "%pwsh.exe")
| table
    _time
    host
    User
    ParentImage
    ParentCommandLine
    Image
    CommandLine
    ProcessId
    ProcessGuid
| sort _time
```

---

## Build a PowerShell Timeline

```spl
index=windows earliest=-24h
(
    EventCode=1 OR
    EventCode=3 OR
    EventCode=11 OR
    EventCode=13 OR
    EventCode=22 OR
    EventCode=4103 OR
    EventCode=4104 OR
    EventCode=4688 OR
    EventCode=4698 OR
    EventCode=7045 OR
    EventCode=1116 OR
    EventCode=1117 OR
    EventCode=5001 OR
    EventCode=5007
)
| search
    "powershell.exe"
    OR "pwsh.exe"
    OR "<SCRIPT_NAME>"
    OR "<PROCESS_GUID>"
| eval activity=case(
    EventCode=1 OR EventCode=4688, "Process creation",
    EventCode=3, "Network connection",
    EventCode=11, "File creation",
    EventCode=13, "Registry change",
    EventCode=22, "DNS query",
    EventCode=4103, "PowerShell module activity",
    EventCode=4104, "PowerShell script block",
    EventCode=4698, "Scheduled task created",
    EventCode=7045, "Service installed",
    EventCode=1116, "Defender detection",
    EventCode=1117, "Defender remediation",
    EventCode=5001 OR EventCode=5007, "Defender configuration change",
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
    ScriptBlockText
    TargetFilename
    DestinationIp
    DestinationPort
    QueryName
    Message
| sort _time
```

---

# Investigation Decision Tree

```
Suspicious PowerShell Alert
             |
             v
Does the event exist locally?
             |
        +----+----+
        |         |
       No        Yes
        |         |
Check host,     Is the user and
time range,     parent process expected?
parsing               |
                 +----+----+
                 |         |
                Yes        No
                 |         |
Review command,     Increase severity
script and change         |
record                    v
                 Is the command encoded,
                 obfuscated or remote?
                         |
                    +----+----+
                    |         |
                   Yes        No
                    |         |
              Decode safely   Review files,
              without running child processes,
                              privilege and scope
                    |
                    v
             Did it create persistence,
             access credentials, disable
             security, or contact an
             unknown destination?
                    |
               +----+----+
               |         |
              Yes        No
               |         |
        Contain and      Determine whether
        escalate         activity was authorized
```

---

# Common Investigation Scenarios

## Scenario 1: Approved Administrative Command

Indicators:

* Known administrator
* Expected workstation
* Readable command
* Approved maintenance or troubleshooting
* No suspicious network or file activity
* No persistence
* No security-control changes

Response:

* Validate the administrative purpose
* Preserve the command and event
* Close as authorized activity

---

## Scenario 2: Approved CyberLab Script

Indicators:

* Script is stored in the approved test directory
* Exercise record exists
* Harmless commands only
* Standard test account
* No external connection
* Cleanup completed

Response:

* Confirm scope
* Preserve telemetry
* Classify as authorized test activity

---

## Scenario 3: Encoded but Legitimate Automation

Indicators:

* Known management tool
* Signed or approved script
* Expected parent process
* Approved deployment window
* Known destination systems
* Reproducible administrative purpose

Response:

* Decode and validate safely
* Confirm source and owner
* Document why encoding is used
* Tune carefully without broadly suppressing encoded commands

---

## Scenario 4: Office Application Launches PowerShell

Indicators:

* `winword.exe`, `excel.exe`, or similar parent
* Hidden or encoded command
* Download or child process
* Script in temporary directory

Response:

* Treat as high severity
* Preserve process and document evidence
* Isolate the endpoint
* Review email, document, browser, and network activity
* Review Defender results
* Escalate for malware investigation

---

## Scenario 5: PowerShell Downloads a File

Indicators:

* `Invoke-WebRequest`
* `DownloadString`
* `DownloadFile`
* `WebClient`
* `Start-BitsTransfer`
* Network connection followed by file creation

Response:

* Preserve URL or destination as evidence
* Do not access it
* Hash the downloaded file
* Isolate the endpoint when unauthorized
* Review execution and persistence
* Escalate based on file behavior and source

---

## Scenario 6: PowerShell Creates Persistence

Indicators:

* Scheduled task
* New service
* Registry Run key
* Startup-folder item
* PowerShell profile modification
* WMI subscription

Response:

* Treat as high severity
* Preserve persistence configuration
* Isolate where appropriate
* Remove only after evidence collection
* Review all related sessions and accounts

---

## Scenario 7: PowerShell Modifies Defender

Indicators:

* Defender exclusion added
* Real-time protection disabled
* Script scanning disabled
* Defender service stopped
* Security logs deleted

Response:

* Treat as high or critical
* Preserve Defender and PowerShell events
* Isolate endpoint
* Restore security controls through approved procedure
* Investigate preceding process and authentication activity

---

## Scenario 8: PowerShell Runs as SYSTEM

Indicators:

* SYSTEM user
* Service, scheduled task, or management agent parent
* High-integrity execution
* Broad system changes

Response:

* Validate the service or task
* Review who created or modified it
* Confirm whether SYSTEM execution is expected
* Escalate unknown or recently created mechanisms

---

# Containment

## When Containment Is Not Required

Containment may not be necessary when:

* The activity is clearly authorized
* The command and script are understood
* User and parent process are expected
* No suspicious network, file, or persistence activity exists
* The event is part of an approved exercise

Document why containment was unnecessary.

---

## Process Containment

When authorized and evidence is preserved:

```
Stop-Process `
    -Id <PROCESS_ID> `
    -Force
```

Use process termination only when:

* Harmful activity is continuing
* Isolation alone is insufficient
* The process can be safely terminated
* Required volatile evidence has been collected

---

## Endpoint Containment

Options include:

* Disconnect the VM’s host-only adapter
* Disable the network adapter
* Isolate through an endpoint platform
* Block external connectivity
* End unauthorized sessions
* Stop the responsible scheduled task or service

Do not power off the system before considering volatile evidence.

---

## Account Containment

With appropriate authorization:

* Disable the affected user
* Reset credentials
* Revoke active sessions
* Remove unauthorized privilege
* Restrict remote access
* Disable the actor account

---

## Network Containment

* Block the destination address
* Block the destination domain
* Block the destination port
* Restrict PowerShell remoting
* Restrict WinRM
* Disable an unnecessary listener
* Limit access to approved management hosts

Use narrow controls and document rollback steps.

---

# Eradication

## Remove Unauthorized Scripts

Before removal:

* Record path
* Record timestamps
* Record owner
* Hash the file
* Preserve a secure copy
* Review related files

Then remove through the approved response process.

---

## Remove Persistence

Depending on findings:

* Delete unauthorized scheduled tasks
* Remove unauthorized services
* Remove Run or RunOnce entries
* Restore modified PowerShell profiles
* Remove startup-folder artifacts
* Remove WMI persistence
* Remove unauthorized remoting configuration

Validate each change separately.

---

## Restore Security Controls

Confirm:

* Defender real-time protection enabled
* Script scanning enabled
* No unauthorized exclusions
* Firewall enabled
* PowerShell logging enabled
* Wazuh agent active
* Splunk forwarder active

---

## Address Compromised Accounts

* Disable or restrict account
* Reset credentials
* Revoke sessions
* Review privilege changes
* Review successful logons
* Review account creation
* Review lateral movement
* Restore only after scope is understood

---

# Recovery

## Confirm PowerShell Activity Stopped

```
Get-CimInstance Win32_Process |
    Where-Object {
        $_.Name -match "powershell|pwsh"
    } |
Select-Object `
    ProcessId,
    ParentProcessId,
    Name,
    CommandLine,
    CreationDate
```

Confirm only expected sessions remain.

---

## Confirm Persistence Was Removed

Review:

* Scheduled tasks
* Services
* Registry autoruns
* Startup folders
* PowerShell profiles
* WMI subscriptions

---

## Confirm Defender Health

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

## Confirm Logging Health

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-PowerShell/Operational" |
Select-Object `
    LogName,
    IsEnabled,
    RecordCount
```

Confirm Script Block Logging and process telemetry continue to generate events.

---

## Validate with a Harmless Command

After recovery, run an approved harmless command:

```
Get-Date
```

Confirm:

* Process event generated
* PowerShell event generated
* Wazuh received the event
* Splunk received the event

Do not repeat the suspicious command.

---

# Escalation Criteria

Escalate immediately when:

* PowerShell is launched by Office, a browser, or an unknown executable
* The command is encoded and unauthorized
* Remote content is downloaded or executed
* Credentials are accessed
* Defender or logging is disabled
* Persistence is created
* SYSTEM execution is unexplained
* A privileged account is involved
* Multiple endpoints show similar activity
* Lateral movement is suspected
* The activity is followed by account or group changes
* The activity is destructive
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
User:
Integrity level:
PowerShell executable:
Parent process:
Command line:
Decoded command:
Script path:
Script hash:
Child processes:
Network destinations:
Files created:
Persistence:
Defender result:
Security-control changes:
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
* Event ID `4103`
* Event ID `4104`
* Event ID `4688` or Sysmon Event ID `1`
* Full command line
* Parent process
* Process tree
* User and privilege context
* Script file and hash
* Network events
* DNS events
* File-creation events
* Child processes
* Defender events
* Persistence evidence
* Wazuh results
* Splunk timeline
* Containment and recovery actions
* Analyst notes

---

## Export PowerShell Operational Events

```
wevtutil epl `
    "Microsoft-Windows-PowerShell/Operational" `
    "<EVIDENCE_PATH>\PowerShell-Operational.evtx"
```

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

Raw logs may contain unrelated sensitive information.

Store them privately.

---

## Export a Narrow PowerShell Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\powershell-summary.csv" `
    -NoTypeInformation
```

---

## Preserve the Script

Copy it to the private evidence location without executing it:

```
Copy-Item `
    -Path "<SCRIPT_PATH>" `
    -Destination "<EVIDENCE_PATH>\quarantine-copy" `
    -Force
```

Access to preserved suspicious files should be restricted.

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
Runbook: RB-05
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

* Administrative troubleshooting
* Software deployment
* Configuration management
* Endpoint-management tools
* Login scripts
* Scheduled maintenance
* Backup operations
* Security products
* Vulnerability scanners
* System inventory
* Approved automation
* Developer activity
* CyberLab exercises

A legitimate command should still be reviewed for:

* Authorization
* Script ownership
* Least privilege
* Source host
* Network destination
* Logging visibility
* Change record

---

# Detection Gaps

Document whether the investigation lacked:

* Full command line
* Script block text
* Parent process
* Parent command line
* Process GUID
* User
* Integrity level
* Script path
* File hash
* Digital signature
* Network destination
* DNS query
* Child process
* File creation
* Registry change
* Persistence
* Defender context
* Security-control change
* Approved-script context
* Process-tree correlation

---

# Detection Improvement Recommendations

Potential improvements include:

* Enable Script Block Logging
* Enable Module Logging
* Enable command-line process auditing
* Deploy Sysmon process and network telemetry
* Normalize PowerShell executable names
* Detect unusual parent processes
* Detect encoded commands
* Detect hidden-window execution
* Detect execution-policy bypass
* Detect remote content retrieval
* Correlate PowerShell with file creation
* Correlate PowerShell with child processes
* Correlate PowerShell with Defender changes
* Correlate PowerShell with scheduled-task or service creation
* Maintain approved script and automation inventories
* Add code-signing and script-control policies
* Create a PowerShell investigation dashboard

---

# MITRE ATT&CK Context

Suspicious PowerShell activity may relate to:

```
T1059.001 – Command and Scripting Interpreter: PowerShell
```

Depending on observed behavior, related techniques may include:

* T1105 — Ingress Tool Transfer
* T1021.006 — Windows Remote Management
* T1053.005 — Scheduled Task
* T1543.003 — Windows Service
* T1547 — Boot or Logon Autostart Execution
* T1562.001 — Impair Defenses
* T1003 — OS Credential Dumping
* T1087 — Account Discovery
* T1046 — Network Service Discovery
* T1070 — Indicator Removal

Apply mappings only when the associated behavior is observed.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Host:
User:
Account privilege:
Process start:
PowerShell executable:
Parent process:
Command line:
Encoded:
Decoded command:
Script path:
Script hash:
Script signature:
Child processes:
Network destinations:
DNS queries:
Files created:
Registry changes:
Persistence:
Defender result:
Security-control changes:
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
Approved administrative command
Approved automation
Approved software deployment
CyberLab test activity
Misconfigured script
Unauthorized user activity
Malicious script execution
Remote payload retrieval
Persistence attempt
Credential access
Defense evasion
Lateral movement
Compromised administrator
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The original event is confirmed.
* The user is identified.
* The parent process is identified.
* The command line is preserved and reviewed.
* Encoded content is decoded safely where required.
* The script path and hash are documented.
* Network activity is reviewed.
* Child processes are reviewed.
* File and registry changes are reviewed.
* Persistence is reviewed.
* Defender and security-control status are reviewed.
* Authorization is confirmed or disproved.
* Required containment is completed.
* Unauthorized artifacts and persistence are removed.
* Accounts and sessions are addressed where required.
* Logging and endpoint protection are healthy.
* Monitoring confirms the activity stopped.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The PowerShell activity was investigated using PowerShell Operational, process, Sysmon, Defender, Wazuh, and Splunk telemetry. The user, parent process, command line, script content, privilege context, network activity, file changes, child processes, and persistence indicators were reviewed. The activity was classified as <CLASSIFICATION>. Required containment, eradication, and recovery actions were completed, monitoring confirmed the intended endpoint state, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Real usernames
* Administrative identities
* Internal IP addresses
* Domain names
* Hostnames where sensitive
* Full operational script paths
* Process GUIDs
* SIDs
* Internal URLs
* Tokens
* Credentials
* Encoded payloads
* Decoded malicious content
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths
* Unrelated event content

Use placeholders such as:

```
<USER_ACCOUNT>
<WINDOWS_ENDPOINT>
<PARENT_PROCESS>
<PROCESS_ID>
<PROCESS_GUID>
<COMMAND_LINE>
<SCRIPT_PATH>
<SCRIPT_NAME>
<SCRIPT_HASH>
<DESTINATION_IP>
<DESTINATION_DOMAIN>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4104`
* Sysmon process-creation event
* Parent and child process tree
* Sanitized command line
* Script hash and signature
* Wazuh PowerShell alert
* Splunk PowerShell timeline
* Network connection event
* Defender alert
* Persistence evidence
* Post-recovery health

Before publishing:

1. Crop unrelated information.
2. Remove usernames.
3. Remove operational addresses and domains.
4. Remove credentials and tokens.
5. Do not expose encoded or decoded malicious payloads.
6. Remove process GUIDs where unnecessary.
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
* User identified
* PowerShell process identified
* Parent process identified
* Initial severity assigned

## Process Validation

* Local event confirmed
* Event ID 4103 reviewed
* Event ID 4104 reviewed
* Event ID 4688 reviewed
* Sysmon Event ID 1 reviewed
* Full command line preserved
* Process ID recorded
* Process GUID recorded where available
* Parent command line reviewed
* Executable path reviewed
* Hash and signature reviewed

## Command Analysis

* Encoded-command indicators reviewed
* Original encoded content preserved privately
* Content decoded without execution
* Obfuscation reviewed
* Remote-content commands reviewed
* Defender-modification commands reviewed
* Account and group commands reviewed
* Persistence commands reviewed

## User and Privilege Review

* User account validated
* Privilege reviewed
* Logon source reviewed
* Event ID 4624 reviewed
* Event ID 4648 reviewed
* Event ID 4672 reviewed
* Recent group changes checked

## File and Script Review

* Script path identified
* File properties recorded
* SHA-256 recorded
* Signature reviewed
* Alternate data streams reviewed
* Script content reviewed safely
* Related file events reviewed

## Network Review

* Sysmon Event ID 3 reviewed
* Sysmon Event ID 22 reviewed
* Destination addresses reviewed
* Destination ports reviewed
* DNS queries reviewed
* External or unknown destinations escalated

## Follow-On Activity

* Child processes reviewed
* Scheduled tasks reviewed
* Services reviewed
* Registry changes reviewed
* Startup folders reviewed
* PowerShell profiles reviewed
* WMI persistence reviewed where relevant
* Defender events reviewed
* Security-control changes reviewed

## SIEM Review

* Wazuh event reviewed
* Wazuh field quality assessed
* Splunk process timeline created
* Unusual-parent search completed
* Encoded-command search completed
* Network search completed
* File-creation search completed
* Child-process search completed
* Ingestion delay reviewed where needed

## Response

* Severity updated
* Containment decision recorded
* Process stopped where required
* Endpoint isolated where required
* Account contained where required
* Network destination blocked where required
* Persistence removed
* Security controls restored
* Unauthorized files removed after preservation

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

## Primary PowerShell Event Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-1)
    }
```

## Primary Process Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe"
}
```

## Primary Splunk Search

```spl
index=windows earliest=-1h
(
    EventCode=1 OR
    EventCode=4103 OR
    EventCode=4104 OR
    EventCode=4688
)
| search
    "powershell.exe"
    OR "pwsh.exe"
| table
    _time
    host
    User
    ParentImage
    Image
    CommandLine
    ScriptBlockText
| sort _time
```

## Immediate Escalation Indicators

```
Office or browser parent process
Encoded or heavily obfuscated command
Remote payload retrieval
Unknown network destination
Credential access
Defender disabled or exclusions added
Persistence created
SYSTEM execution without explanation
Privileged account involved
Multiple endpoints affected
Destructive activity
```

---

# Summary

This runbook provides a repeatable process for investigating suspicious PowerShell activity.

A complete investigation should determine:

* Which user executed PowerShell
* Which parent process launched it
* What command or script ran
* Whether the content was encoded or obfuscated
* Whether the process was elevated
* Whether files or registry values changed
* Whether child processes were created
* Whether network connections occurred
* Whether persistence was established
* Whether security controls were changed
* Whether Microsoft Defender detected related activity
* Whether the execution was authorized or malicious
* Whether containment and eradication were required
* Whether logging and endpoint protection returned to a healthy state

The investigation is complete only after the execution chain is understood, unauthorized activity is contained and removed, evidence is preserved, monitoring confirms the intended endpoint state, and detection improvements are documented.

