# Ingestion Health Validation

## Exercise Overview

This exercise validates the complete telemetry path from event generation on CyberLab endpoints to searchable data in Wazuh and Splunk.

The exercise confirms whether:

* Source systems generate known events
* Local logs contain the expected records
* Wazuh agents and Splunk forwarders are running
* Events traverse the CyberLab network
* Wazuh and Splunk receive the events
* Hostnames, timestamps, event IDs, and source fields are correct
* Ingestion delay remains within an acceptable range
* Duplicate, missing, stale, or misrouted events are identified
* Monitoring resumes correctly after a controlled collection interruption

The purpose is to validate the monitoring pipeline itself rather than one specific security behavior.

---

## Exercise ID

```
EX-10
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Telemetry pipeline validation
* Windows Event Log analysis
* Linux log analysis
* Wazuh agent troubleshooting
* Splunk Universal Forwarder troubleshooting
* SIEM search validation
* Timestamp and timezone analysis
* Ingestion-delay measurement
* Duplicate-event identification
* Data-quality assessment
* Monitoring recovery validation
* Evidence preservation

---

## Authorization Boundary

This exercise must use:

* CyberLab systems only
* Known harmless validation events
* Existing monitoring agents and forwarders
* The VMware host-only network
* Temporary NAT only when updates are required
* Approved administrative accounts
* Documented maintenance windows

Do not:

* Generate real malicious activity
* Clear operational logs before evidence collection
* Delete SIEM indexes
* Remove Wazuh agents
* Reinstall forwarders as the first troubleshooting step
* Disable monitoring on multiple systems simultaneously
* Change production or household systems
* Test systems outside the CyberLab

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Kali testing system: KALI-TEST
Domain: cyberlab.example
Documentation subnet: 192.0.2.0/24
Splunk Windows index: windows
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Define the expected telemetry path
* Generate known Windows validation events
* Confirm events at the source
* Verify Wazuh agent health
* Verify Splunk forwarder health
* Confirm network connectivity to collectors
* Locate known events in Wazuh
* Locate known events in Splunk
* Measure source-to-SIEM delay
* Identify missing or incomplete fields
* Identify duplicate data
* Distinguish event time from ingestion time
* Validate recovery after a controlled interruption
* Document pipeline health and detection gaps

---

## Required Systems

The standard exercise uses:

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

Optional systems:

* KALI-TEST
* Additional monitored Windows endpoints
* Linux endpoints
* Sysmon-enabled systems

---

## Required Accounts

### Standard Test Account

Used to generate harmless user-level events.

```
ingestion.test
```

### Authorized Windows Administrator

Used only for:

* Service validation
* Event-log access
* Controlled agent or forwarder restart
* Network diagnostics
* Configuration review

### Authorized Linux Administrator

Used for:

* Wazuh service review
* Splunk service review
* Docker inspection
* Local log inspection
* Port validation

Commands requiring elevated access are identified throughout the exercise.

---

## Required Monitoring Components

Confirm the expected components for the environment.

### Windows Endpoint

* Windows Event Log
* Wazuh agent
* Splunk Universal Forwarder
* Optional Sysmon
* Windows Defender Firewall

### Wazuh Server

* Wazuh manager
* Wazuh indexer
* Wazuh dashboard
* Docker services where applicable

### Splunk Server

* Splunk Enterprise
* Splunk Web
* Receiving port
* Search indexes
* Internal logs

---

## Safety Notes

* Generate one known event at a time.
* Record the exact generation time.
* Confirm the event locally before troubleshooting the SIEM.
* Do not stop both Wazuh and Splunk collection simultaneously.
* Keep controlled interruptions brief.
* Preserve pre-interruption evidence.
* Restart only the component being tested.
* Confirm recovery before continuing.
* Do not use snapshot restoration until evidence is preserved.
* Do not expose management ports outside the CyberLab.

---

## Telemetry Path

```
User or System Activity
          |
          v
Local Event Source
          |
          v
Wazuh Agent / Splunk Universal Forwarder
          |
          v
VMware Host-Only Network
          |
          v
Wazuh Manager / Splunk Receiver
          |
          v
Indexer or Search Storage
          |
          v
Dashboard and Analyst Search
```

A healthy dashboard alone does not prove this complete path is functioning.

---

## Validation Layers

The exercise validates each layer separately.

| Layer     | Validation Question                      |
| --------- | ---------------------------------------- |
| Activity  | Did the test action occur?               |
| Source    | Was the event written locally?           |
| Collector | Did the agent or forwarder read it?      |
| Transport | Could the endpoint reach the server?     |
| Receiver  | Did the server accept the data?          |
| Indexing  | Was the event stored correctly?          |
| Search    | Could the event be found?                |
| Parsing   | Were important fields extracted?         |
| Timing    | Did the event arrive promptly?           |
| Recovery  | Did ingestion resume after interruption? |

---

## Validation Events

Use simple, harmless events that are easy to identify.

| Validation Event             | Typical Source             | Purpose                        |
| ---------------------------- | -------------------------- | ------------------------------ |
| Successful sign-in           | Security                   | Authentication pipeline        |
| Failed sign-in               | Security                   | Known security event           |
| Process creation             | Security or Sysmon         | Process telemetry              |
| Test file creation           | Wazuh FIM or Sysmon        | File telemetry                 |
| Defender health query        | PowerShell or Defender log | Endpoint protection visibility |
| Service start or stop        | System                     | Service telemetry              |
| Custom Application log event | Application                | Unique searchable marker       |

Not every event source must be tested in one run.

---

## Preferred Exercise Design

The standard exercise uses three validation markers:

* One unique Application log event
* One controlled failed logon
* One controlled file integrity change

This provides coverage across:

* General Windows event collection
* Security event collection
* File integrity monitoring

---

## Investigation Questions

The final report should answer:

* Which validation events were generated?
* Did each event appear locally?
* Did Wazuh receive each event?
* Did Splunk receive each event?
* Which host identity was assigned?
* Which source and sourcetype were assigned?
* Which Wazuh agent identity was assigned?
* Were timestamps accurate?
* What was the ingestion delay?
* Were events duplicated?
* Were any fields missing?
* Did collection recover after interruption?
* Were events created during the interruption recovered?
* Did either platform lose data?
* Which pipeline layer caused any failure?
* What improvements are required?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

Suggested snapshot name:

```
PRE-EX10-Ingestion-Health-Validation
```

A new snapshot is recommended before changing agent, forwarder, receiver, or index configuration.

---

## Define the Exercise Window

Record:

```
Exercise start:
Expected completion:
Systems included:
Validation events:
Controlled interruption planned:
Maximum interruption duration:
Expected Wazuh indexes:
Expected Splunk indexes:
```

Use a narrow time window to reduce unrelated event noise.

---

## Confirm System Time

### WIN11TARGET and DC01

```
Get-Date
```

```
w32tm /query /status
```

```
w32tm /query /source
```

### Wazuh and Splunk

```
timedatectl
```

Record:

```
DC01 time:
WIN11TARGET time:
Wazuh time:
Splunk time:
Largest observed difference:
```

Significant time differences must be resolved before measuring ingestion delay.

---

## Confirm Host Identity

### Windows

```
hostname
```

```
$env:COMPUTERNAME
```

```
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain
```

### Linux

```
hostname
```

```
hostnamectl
```

Confirm that the names match the expected SIEM host and agent identities.

---

## Confirm VMware Network Configuration

### Windows

```
Get-NetIPConfiguration
```

```
Get-NetAdapter |
    Select-Object Name, Status, InterfaceDescription
```

### Linux

```
ip address
```

```
ip route
```

Confirm:

* The host-only adapter is active.
* The expected CyberLab route exists.
* No unintended bridged adapter is active.
* Temporary NAT usage is understood.
* Collector addresses are reachable through the intended interface.

---

# Source Health Validation

## Confirm Windows Event Logs

Run on WIN11TARGET:

```
Get-WinEvent -ListLog Security |
    Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes
```

```
Get-WinEvent -ListLog System |
    Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes
```

```
Get-WinEvent -ListLog Application |
    Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes
```

Confirm all required logs are enabled and have available capacity.

---

## Review Recent Local Events

```
Get-WinEvent `
    -LogName Application `
    -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName
```

```
Get-WinEvent `
    -LogName Security `
    -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName
```

This establishes that the local event system is functioning.

---

## Confirm Security Log Access

Administrative access is generally required.

```
Get-WinEvent `
    -LogName Security `
    -MaxEvents 1
```

If access is denied, use an authorized administrative session rather than changing log permissions.

---

# Wazuh Agent Validation

## Confirm the Windows Service

Run on the monitored Windows endpoint:

```
Get-Service WazuhSvc
```

Expected:

```
Status: Running
```

Alternative discovery:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

---

## Review the Wazuh Agent Log

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

Look for:

* Successful connection
* Reconnection
* Invalid configuration
* Event-channel errors
* Queue errors
* Manager communication errors
* Repeated disconnects

---

## Confirm the Wazuh Manager Address

Review the agent configuration privately:

```
Select-String `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" `
    -Pattern "<address>|<port>|<protocol>"
```

Do not publish operational manager addresses or enrollment values.

---

## Confirm Network Reachability to Wazuh

From WIN11TARGET:

```
Test-NetConnection `
    -ComputerName <WAZUH_SERVER> `
    -Port <WAZUH_AGENT_PORT>
```

Record:

```
Remote address:
Remote port:
TCP test succeeded:
Interface used:
Source address:
```

Use the port configured by the actual deployment.

---

## Confirm Agent Status in Wazuh

In the Wazuh dashboard, confirm:

* Agent status is active
* Agent name is correct
* Agent ID is stable
* Operating system is correct
* Last keepalive is recent
* Manager assignment is correct
* No duplicate agent identity exists

Do not publish the operational agent ID where it is considered sensitive.

---

# Splunk Forwarder Validation

## Confirm the Windows Service

Run on WIN11TARGET:

```
Get-Service SplunkForwarder
```

Alternative:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Expected:

```
Status: Running
```

---

## Locate the Forwarder Installation

A common path is:

```
C:\Program Files\SplunkUniversalForwarder
```

Confirm:

```
Get-Item `
    -Path "C:\Program Files\SplunkUniversalForwarder" `
    -ErrorAction SilentlyContinue
```

The actual path may vary.

---

## Review the Forwarder Log

Typical path:

```
C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log
```

Review recent entries:

```
Get-Content `
    -Path "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" `
    -Tail 100
```

Look for:

* Connection established
* Connection refused
* TcpOutputProc errors
* Indexer unavailable
* SSL errors
* Parsing errors
* Blocked queues
* Input configuration errors

---

## Review Forwarding Status

Run from an administrator Command Prompt or PowerShell session:

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Expected output should identify an active forwarding destination.

Do not publish the operational receiver address.

---

## Review Configured Inputs

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list monitor
```

For Windows Event Log inputs, review the configuration files privately.

Common configuration location:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

---

## Confirm Network Reachability to Splunk

```
Test-NetConnection `
    -ComputerName <SPLUNK_SERVER> `
    -Port <SPLUNK_RECEIVING_PORT>
```

Record:

```
Remote address:
Remote port:
TCP test succeeded:
Interface used:
Source address:
```

The common receiving port is not assumed in public documentation. Use the actual configured value privately.

---

# Server Health Validation

## Wazuh Server Health

The exact commands depend on the deployment.

### Docker Deployment

Run on WAZUH-SERVER:

```
sudo docker ps
```

```
sudo docker compose ps
```

Run the Docker Compose command from the Wazuh deployment directory.

Review container health:

```
sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Confirm the manager, indexer, and dashboard components are running.

---

## Review Wazuh Container Logs

```
sudo docker logs --tail 100 <WAZUH_MANAGER_CONTAINER>
```

```
sudo docker logs --tail 100 <WAZUH_INDEXER_CONTAINER>
```

```
sudo docker logs --tail 100 <WAZUH_DASHBOARD_CONTAINER>
```

Look for:

* Indexer connection errors
* Authentication failures
* Disk errors
* Certificate errors
* Repeated restarts
* Queue saturation
* Agent communication errors

---

## Splunk Server Health

The current lab deployment uses Splunk under:

```
/opt/splunk
```

Check status:

```
sudo /opt/splunk/bin/splunk status
```

The lab’s current startup method may use:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

This reflects the current lab state.

A future hardening phase should migrate Splunk to a dedicated service account.

---

## Review Splunk Internal Health

In Splunk Web, review:

* Monitoring Console
* Indexing status
* Forwarder management where configured
* License status
* Disk usage
* Search health
* Internal warnings

Search internal logs:

```spl
index=_internal earliest=-30m
| stats count by host source sourcetype
```

---

## Confirm Splunk Receiving Port

On SPLUNK-SERVER:

```
sudo ss -lntp
```

Filter privately for the configured receiving port:

```
sudo ss -lntp | grep "<SPLUNK_RECEIVING_PORT>"
```

Confirm Splunk is listening on the expected interface and port.

---

## Confirm Disk Space

### Wazuh and Splunk Linux Servers

```
df -h
```

```
df -i
```

Review:

* Root file system
* Docker storage
* Wazuh index storage
* Splunk index storage
* Available inodes

Low disk space can cause delayed or failed ingestion.

---

# Validation Marker 1: Custom Application Event

## Purpose

A custom Application log event provides a unique searchable marker with a known timestamp and identifier.

This is useful because it can be distinguished from normal background events.

---

## Generate a Unique Marker

Run on WIN11TARGET from an administrator PowerShell session.

First create the event source if required:

```
$Source = "CyberLab-Ingestion-Test"

if (-not [System.Diagnostics.EventLog]::SourceExists($Source)) {
    New-EventLog `
        -LogName Application `
        -Source $Source
}
```

Create a unique marker:

```
$Marker = "EX10-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1001 `
    -EntryType Information `
    -Message "Authorized CyberLab ingestion validation marker: $Marker"

$Marker
```

Record the marker exactly.

---

## Confirm the Marker Locally

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Application"
        ProviderName = "CyberLab-Ingestion-Test"
        Id = 1001
        StartTime = (Get-Date).AddMinutes(-10)
    } |
Select-Object `
    TimeCreated,
    Id,
    ProviderName,
    MachineName,
    Message
```

Do not continue until the marker is visible locally.

---

## Record Marker Details

```
Marker:
Generation time:
Local event time:
Event ID:
Provider:
Computer:
Local validation result:
```

---

# Validation Marker 2: Controlled Failed Logon

## Review the Lockout Policy

Before generating the event:

```
Get-ADDefaultDomainPasswordPolicy |
    Select-Object `
        LockoutThreshold,
        LockoutDuration,
        LockoutObservationWindow
```

Use one failed attempt only.

---

## Generate the Failed Authentication

Use the approved nonprivileged test account and enter one intentionally incorrect password.

Record:

```
Test account:
Source system:
Attempt time:
Authentication method:
Observed result:
```

---

## Confirm Event ID 4625 Locally

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddMinutes(-10)
    } |
Where-Object {
    $_.Message -match "ingestion\.test"
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Confirm the account remains unlocked.

---

# Validation Marker 3: Controlled File Change

## Confirm the FIM Test Directory

Use:

```
C:\CyberLab-Test\Ingestion
```

Create it when required:

```
New-Item `
    -Path "C:\CyberLab-Test\Ingestion" `
    -ItemType Directory `
    -Force
```

The path must be included in Wazuh File Integrity Monitoring for FIM validation.

---

## Create the Test File

```
$FileMarker = "EX10-FILE-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Set-Content `
    -Path "C:\CyberLab-Test\Ingestion\ingestion-marker.txt" `
    -Value $FileMarker

$FileMarker
```

Record:

```
File marker:
Creation time:
File path:
Expected Wazuh action:
```

---

## Confirm the File Locally

```
Get-Item `
    -Path "C:\CyberLab-Test\Ingestion\ingestion-marker.txt" |
Select-Object `
    FullName,
    Length,
    CreationTime,
    LastWriteTime
```

Hash the file:

```
Get-FileHash `
    -Path "C:\CyberLab-Test\Ingestion\ingestion-marker.txt" `
    -Algorithm SHA256
```

---

# Wazuh Ingestion Validation

## Search for the Application Marker

Search the Wazuh dashboard for the unique marker:

```
EX10-<TIMESTAMP>
```

Also search by:

```
CyberLab-Ingestion-Test
```

Confirm:

* Correct agent
* Correct event ID
* Correct provider
* Correct message
* Correct timestamp

Wazuh may collect the event without generating a high-severity alert.

---

## Search for the Failed Logon

Search by:

```
ingestion.test
```

and:

```
4625
```

Review:

* Agent
* Event ID
* Account
* Failure reason
* Logon type
* Source address
* Rule
* Timestamp

---

## Search for the File Marker

Search for:

```
ingestion-marker.txt
```

or the unique file marker.

Review:

* File path
* Action
* Hash
* Agent
* Rule
* Timestamp
* User and process where available

---

## Record Wazuh Results

| Marker            | Received | Agent Correct | Timestamp Correct | Fields Complete    | Alert Generated |
| ----------------- | -------- | ------------- | ----------------- | ------------------ | --------------- |
| Application event | Yes / No | Yes / No      | Yes / No          | Yes / Partial / No | Yes / No        |
| Failed logon      | Yes / No | Yes / No      | Yes / No          | Yes / Partial / No | Yes / No        |
| File change       | Yes / No | Yes / No      | Yes / No          | Yes / Partial / No | Yes / No        |

---

## Wazuh Validation Questions

* Did every expected event arrive?
* Did the correct agent report each event?
* Was the host identity stable?
* Were timestamps accurate?
* Were the expected fields parsed?
* Did the application event require a custom decoder?
* Did the failed logon create an alert?
* Did the file event include user or process context?
* Were duplicate events present?
* Were any events delayed?
* Did event ordering match the source timeline?

---

# Splunk Ingestion Validation

## Search for the Application Marker

```spl
index=windows earliest=-30m
"EX10-"
| table
    _time
    _indextime
    host
    source
    sourcetype
    EventCode
    ProviderName
    Message
| sort _time
```

Use the exact marker when available:

```spl
index=windows earliest=-30m
"<EXACT_MARKER>"
```

---

## Search for the Failed Logon

```spl
index=windows EventCode=4625 earliest=-30m
| search "ingestion.test"
| table
    _time
    _indextime
    host
    source
    sourcetype
    EventCode
    user
    LogonType
    FailureReason
    Message
| sort _time
```

---

## Search for the File Marker

When Wazuh FIM events are forwarded into Splunk:

```spl
index=<WAZUH_INDEX> earliest=-30m
| search
    "ingestion-marker.txt"
    OR "EX10-FILE-"
| table
    _time
    _indextime
    agent.name
    rule.id
    rule.level
    rule.description
    syscheck.event
    syscheck.path
    syscheck.sha256_after
    syscheck.audit.user.name
    syscheck.audit.process.name
| sort _time
```

When file telemetry enters Splunk through Sysmon or another source, adjust the index and fields.

---

## Confirm Host and Source Assignment

```spl
index=windows earliest=-30m
(
    "EX10-"
    OR "ingestion.test"
)
| stats
    count
    values(source) as sources
    values(sourcetype) as sourcetypes
    values(EventCode) as event_codes
    by host
```

Confirm:

* The host is correct.
* The source is expected.
* The sourcetype is expected.
* The event code is parsed.
* No unexpected host alias exists.

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
(
    "EX10-"
    OR "ingestion.test"
)
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    source
    sourcetype
    EventCode
    Message
| sort _time
```

Record:

```
Minimum delay:
Maximum delay:
Average delay:
Largest delayed event:
```

---

## Summarize Delay

```spl
index=windows earliest=-30m
(
    "EX10-"
    OR "ingestion.test"
)
| eval ingestion_delay_seconds=_indextime-_time
| stats
    count
    min(ingestion_delay_seconds) as minimum_delay
    avg(ingestion_delay_seconds) as average_delay
    max(ingestion_delay_seconds) as maximum_delay
    by host source sourcetype
```

---

## Identify Future-Dated Events

```spl
index=windows earliest=-24h
| eval clock_difference=_time-now()
| where clock_difference > 60
| table
    _time
    host
    source
    EventCode
    clock_difference
    Message
| sort - clock_difference
```

Future-dated events may indicate clock or timezone problems.

---

## Identify Excessively Delayed Events

```spl
index=windows earliest=-24h
| eval ingestion_delay_seconds=_indextime-_time
| where ingestion_delay_seconds > <DELAY_THRESHOLD>
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    source
    sourcetype
    EventCode
| sort - ingestion_delay_seconds
```

Set the threshold according to the lab’s expected collection interval.

---

## Splunk Validation Questions

* Did every expected event arrive?
* Which index stored each event?
* Was the host field correct?
* Was the source field correct?
* Was the sourcetype correct?
* Was the EventCode extracted?
* Was the event timestamp accurate?
* Was the ingestion delay acceptable?
* Were duplicate records present?
* Were events assigned to an unexpected index?
* Were event fields consistent across hosts?
* Was the marker searchable using exact text?

---

# Duplicate Event Analysis

## Identify Exact Duplicate Markers

```spl
index=windows earliest=-30m
"EX10-"
| stats
    count
    values(source) as sources
    values(sourcetype) as sourcetypes
    by host EventCode Message
| where count > 1
```

---

## Identify Raw-Event Duplicates

```spl
index=windows earliest=-30m
| stats
    count
    values(source) as sources
    by host _time _raw
| where count > 1
| sort - count
```

Use a narrow time range because grouping by `_raw` can be expensive.

---

## Common Duplicate Causes

* Same event channel configured twice
* Local and app-level configuration overlap
* Duplicate forwarder inputs
* Wazuh and native Windows collection both forwarded to the same index
* Snapshot rollback
* Reinstalled forwarder with stale state
* Forwarder replay after output interruption
* Multiple collection agents reading the same source

Similar endpoint and domain-controller events are not automatically duplicates.

---

# Missing Event Analysis

## Confirm the Event Exists Locally

Before changing the collector, confirm:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "<EXPECTED_LOG>"
        Id = <EXPECTED_EVENT_ID>
        StartTime = (Get-Date).AddMinutes(-30)
    }
```

If the event is not local, the problem is event generation or audit configuration.

---

## Confirm the Input Is Configured

### Wazuh

Review relevant `localfile` or event-channel entries in the agent configuration.

### Splunk

Review:

* `inputs.conf`
* Active monitored inputs
* Windows Event Log stanzas
* Target index
* Disabled status

Do not publish operational credentials or server addresses.

---

## Confirm Collection Logs

### Wazuh Agent

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 200
```

### Splunk Forwarder

```
Get-Content `
    -Path "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" `
    -Tail 200
```

---

## Confirm Receiver Reachability

```
Test-NetConnection `
    -ComputerName <COLLECTOR_SERVER> `
    -Port <COLLECTOR_PORT>
```

A successful TCP test does not prove application-level ingestion, but a failed test identifies a transport problem.

---

## Search Broadly Before Declaring Data Missing

### Splunk

```spl
index=* earliest=-30m
"<EXACT_MARKER>"
```

Then review:

```spl
index=* earliest=-30m
| stats count by index host source sourcetype
```

### Wazuh

Remove narrow filters such as:

* Agent
* Rule level
* Event ID
* Time range
* Security module
* Search field

An event may be ingested but hidden by the selected dashboard view.

---

# Controlled Interruption Test

## Purpose

The optional interruption test validates whether collection recovers after an agent or forwarder restart.

This is not a destructive outage test.

Stop only one collection component at a time.

---

## Choose the Component

Select one:

```
Wazuh agent
Splunk Universal Forwarder
```

Do not stop both simultaneously.

Record:

```
Selected component:
Interruption start:
Maximum interruption:
Expected behavior:
Recovery validation marker:
```

---

# Wazuh Agent Recovery Test

## Preserve the Baseline

Before stopping the agent:

* Confirm active status
* Save the latest keepalive time
* Generate and confirm one baseline event
* Record the local time

---

## Stop the Wazuh Agent

Run on WIN11TARGET as an administrator:

```
Stop-Service WazuhSvc
```

Confirm:

```
Get-Service WazuhSvc
```

Expected:

```
Status: Stopped
```

---

## Generate a Local Event While Stopped

Create a second unique Application marker:

```
$RecoveryMarker = "EX10-WAZUH-RECOVERY-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1002 `
    -EntryType Information `
    -Message "Authorized Wazuh recovery validation marker: $RecoveryMarker"

$RecoveryMarker
```

Confirm it exists locally.

---

## Restart the Wazuh Agent

```
Start-Service WazuhSvc
```

Confirm:

```
Get-Service WazuhSvc
```

Expected:

```
Status: Running
```

---

## Validate Wazuh Recovery

Confirm:

* Agent returns to active status
* Keepalive updates
* No new duplicate agent appears
* Recovery marker becomes searchable
* Delay is recorded
* Events resume in chronological order where possible
* Agent log shows successful reconnection

Record:

```
Interruption start:
Marker generation time:
Agent restart time:
Agent active time:
Marker arrival time:
Recovery delay:
Marker recovered:
Data loss observed:
```

---

# Splunk Forwarder Recovery Test

## Preserve the Baseline

Before stopping the forwarder:

* Confirm active destination
* Generate a baseline marker
* Confirm it appears in Splunk
* Record the current time

---

## Stop the Splunk Forwarder

Run on WIN11TARGET as an administrator:

```
Stop-Service SplunkForwarder
```

Confirm:

```
Get-Service SplunkForwarder
```

Expected:

```
Status: Stopped
```

---

## Generate an Event While Stopped

```
$RecoveryMarker = "EX10-SPLUNK-RECOVERY-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1003 `
    -EntryType Information `
    -Message "Authorized Splunk recovery validation marker: $RecoveryMarker"

$RecoveryMarker
```

Confirm the event exists locally.

---

## Restart the Splunk Forwarder

```
Start-Service SplunkForwarder
```

Confirm:

```
Get-Service SplunkForwarder
```

---

## Confirm Forwarding Status

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Confirm an active destination.

---

## Validate Splunk Recovery

Search for the exact marker:

```spl
index=windows earliest=-30m
"<EXACT_RECOVERY_MARKER>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    source
    sourcetype
    EventCode
    Message
```

Record:

```
Interruption start:
Marker generation time:
Forwarder restart time:
Forwarder connected time:
Marker indexed time:
Recovery delay:
Marker recovered:
Duplicate marker:
Data loss observed:
```

---

## Interruption Test Safety

* Keep the interruption brief.
* Stop only the selected service.
* Do not modify checkpoint files.
* Do not delete forwarder state.
* Do not clear Windows logs.
* Do not restore a snapshot during the test.
* Confirm full recovery before moving to another component.

---

# Wazuh Server Investigation

## Review Active Agents

In the dashboard, review:

* Active agents
* Disconnected agents
* Never-connected agents
* Duplicate names
* Last keepalive
* Operating system
* Manager assignment

A disconnected agent may indicate:

* Endpoint powered off
* Agent stopped
* Network failure
* Manager failure
* Firewall block
* Enrollment problem
* Snapshot identity conflict

---

## Review Manager Logs

In a Docker deployment:

```
sudo docker logs --tail 200 <WAZUH_MANAGER_CONTAINER>
```

Look for:

* Agent authentication failures
* Duplicate agent names
* Connection resets
* Queue warnings
* Decoder errors
* Rule errors
* Indexer communication failures

---

## Review Indexer Health

Use the approved Wazuh dashboard or deployment-specific health command.

Review:

* Cluster health
* Unassigned shards
* Disk usage
* Index status
* Authentication errors
* Repeated index failures

Do not publish credentials or internal certificate details.

---

## Review Wazuh Event Volume

Compare recent event counts by agent and event type.

Look for:

* Sudden drop to zero
* Unexpected spike
* One silent agent
* One noisy agent
* Repeated identical alerts
* Missing Windows channels

---

# Splunk Server Investigation

## Search Internal Forwarder Connections

```spl
index=_internal earliest=-30m
source="*metrics.log"
group=tcpin_connections
| stats
    latest(_time) as last_seen
    values(sourceIp) as source_ips
    values(version) as versions
    by hostname
| convert ctime(last_seen)
```

Field availability depends on the Splunk version and configuration.

---

## Search for Forwarding Errors

```spl
index=_internal earliest=-30m
(
    "TcpOutputProc"
    OR "Connection refused"
    OR "Cooked connection"
    OR "blocked"
    OR "thruput"
)
| table _time host source log_level component message
| sort _time
```

---

## Review Index Volume

```spl
index=_internal earliest=-24h
source="*metrics.log"
group=per_index_thruput
| stats
    sum(kb) as total_kb
    by series
| sort - total_kb
```

This helps identify indexes with no recent ingestion.

---

## Review Host Volume

```spl
index=windows earliest=-24h
| timechart span=15m count by host
```

A host dropping suddenly to zero may indicate an ingestion problem.

---

## Review Source Volume

```spl
index=windows earliest=-24h
| timechart span=15m count by source
```

This can reveal one missing event channel while other sources continue ingesting.

---

## Review Latest Event by Host

```spl
index=windows
| stats
    latest(_time) as latest_event
    latest(_indextime) as latest_index_time
    count
    by host
| eval event_age_seconds=now()-latest_event
| convert ctime(latest_event) ctime(latest_index_time)
| sort - event_age_seconds
```

---

# Data Quality Validation

## Hostname Accuracy

Confirm that:

* DC01 events use the DC01 identity
* WIN11TARGET events use the endpoint identity
* No old hostname remains after rename
* No duplicate host value represents the same machine
* IP addresses are not incorrectly used as permanent hostnames

---

## Source Accuracy

Confirm expected source names such as:

```
WinEventLog:Security
WinEventLog:System
WinEventLog:Application
Microsoft-Windows-PowerShell/Operational
Microsoft-Windows-Windows Defender/Operational
Microsoft-Windows-Sysmon/Operational
```

Actual source formatting depends on the collector.

---

## Sourcetype Accuracy

Confirm that Windows events use the intended Windows event sourcetype.

Incorrect sourcetypes may cause:

* Missing field extraction
* Incorrect timestamps
* Search failures
* CIM incompatibility
* Broken dashboards

---

## Event ID Accuracy

Confirm the event ID is parsed into a searchable field such as:

```
EventCode
event_id
win.system.eventID
```

The exact field varies by platform.

---

## User Field Accuracy

A generic `user` field may represent:

* Subject user
* Target user
* Process user
* Account being authenticated
* Account performing a change

Review the raw event before relying on a generic normalized field.

---

## Timestamp Accuracy

Compare:

* Local event time
* Wazuh event time
* Splunk `_time`
* Splunk `_indextime`
* Dashboard timezone
* System timezone

Do not calculate ingestion delay using a manually copied display time when raw timestamp fields are available.

---

# Investigation

## Build the Validation Timeline

| Time     | System      | Layer    | Marker or Event              | Result   |
| -------- | ----------- | -------- | ---------------------------- | -------- |
| `<TIME>` | WIN11TARGET | Activity | Application marker generated | Complete |
| `<TIME>` | WIN11TARGET | Source   | Application event confirmed  | Present  |
| `<TIME>` | Wazuh       | Indexing | Marker searchable            | Present  |
| `<TIME>` | Splunk      | Indexing | Marker searchable            | Present  |
| `<TIME>` | WIN11TARGET | Activity | Failed logon generated       | Complete |
| `<TIME>` | WIN11TARGET | Source   | Event ID 4625 confirmed      | Present  |
| `<TIME>` | Wazuh       | Indexing | Failed logon searchable      | Present  |
| `<TIME>` | Splunk      | Indexing | Failed logon searchable      | Present  |
| `<TIME>` | WIN11TARGET | Activity | File marker created          | Complete |
| `<TIME>` | Wazuh       | FIM      | File event searchable        | Present  |
| `<TIME>` | Collector   | Recovery | Controlled interruption      | Complete |
| `<TIME>` | Collector   | Recovery | Recovery marker received     | Present  |

---

## Root Cause Classification

When all validation succeeds:

```
The CyberLab telemetry pipeline operated as expected. Known validation events were generated locally, collected by the configured agents, transported across the host-only network, indexed by Wazuh and Splunk, and made available for investigation.
```

When an issue is found, classify it by layer:

```
Activity generation
Source logging
Agent or forwarder
Network transport
Receiver
Indexing
Parsing
Search configuration
Timestamp handling
Dashboard filtering
```

---

## Pipeline Failure Examples

### Source Failure

The action occurred, but no local event was written.

Possible causes:

* Audit policy disabled
* Wrong event channel
* Event source not configured
* Test method did not produce the expected event

### Collection Failure

The event exists locally but does not leave the endpoint.

Possible causes:

* Agent or forwarder stopped
* Input not configured
* Permission failure
* Corrupt configuration
* Queue issue

### Transport Failure

The collector cannot reach the server.

Possible causes:

* Wrong address
* Wrong port
* Firewall
* Route
* VMware adapter
* Server listener unavailable

### Receiver Failure

The network connection succeeds, but the server does not accept data.

Possible causes:

* Receiver disabled
* Authentication failure
* Certificate failure
* Manager problem
* Indexer problem

### Search Failure

The event is indexed but not found by the original search.

Possible causes:

* Wrong index
* Wrong host
* Wrong time range
* Field extraction issue
* Dashboard filter
* Timezone difference

---

## False-Positive Analysis

Pipeline health monitoring can produce false alarms when:

* Endpoint is intentionally powered off
* VM snapshot is being taken
* Maintenance is in progress
* Agent was intentionally restarted
* Network adapter was temporarily removed
* Indexing is delayed during startup
* Test events were not generated
* A host is normally inactive

Maintenance context should be included without suppressing genuine monitoring failures.

---

# Detection Development

## Splunk: Hosts with No Recent Events

```spl
index=windows earliest=-24h
| stats
    latest(_time) as latest_event
    count
    by host
| eval event_age_minutes=round((now()-latest_event)/60, 2)
| where event_age_minutes > <EXPECTED_SILENCE_MINUTES>
| convert ctime(latest_event)
| sort - event_age_minutes
```

---

## Splunk: Source Channel Stopped Ingesting

```spl
index=windows earliest=-24h
| stats
    latest(_time) as latest_event
    count
    by host source
| eval event_age_minutes=round((now()-latest_event)/60, 2)
| where event_age_minutes > <EXPECTED_SILENCE_MINUTES>
| convert ctime(latest_event)
| sort - event_age_minutes
```

---

## Splunk: High Ingestion Delay

```spl
index=windows earliest=-1h
| eval ingestion_delay_seconds=_indextime-_time
| stats
    count
    avg(ingestion_delay_seconds) as average_delay
    max(ingestion_delay_seconds) as maximum_delay
    perc95(ingestion_delay_seconds) as p95_delay
    by host source
| where maximum_delay > <DELAY_THRESHOLD>
| sort - maximum_delay
```

---

## Splunk: Future-Dated Events

```spl
index=windows earliest=-24h
| eval future_offset_seconds=_time-now()
| where future_offset_seconds > 60
| stats
    count
    max(future_offset_seconds) as maximum_future_offset
    by host source
| sort - maximum_future_offset
```

---

## Splunk: Duplicate Marker Detection

```spl
index=windows earliest=-24h
"EX10-"
| stats
    count
    values(source) as sources
    values(sourcetype) as sourcetypes
    by host EventCode Message
| where count > 1
```

---

## Splunk: Expected Host Inventory

A mature implementation may maintain a lookup:

```
host
expected_index
expected_source
expected_event_interval
owner
maintenance_state
```

Conceptual search:

```spl
| inputlookup expected_hosts.csv
| rename host as expected_host
| join type=left expected_host
    [
        search index=windows earliest=-24h
        | stats latest(_time) as latest_event by host
        | rename host as expected_host
    ]
| eval event_age_minutes=round((now()-latest_event)/60, 2)
| where isnull(latest_event)
    OR event_age_minutes > expected_event_interval
```

---

## Wazuh Health Detection Concepts

Useful monitoring conditions include:

* Agent disconnected
* Agent never connected
* Manager unavailable
* Indexer unhealthy
* Dashboard unavailable
* Agent configuration error
* FIM stopped reporting
* Windows channel stopped reporting
* Event volume unexpectedly zero

A disconnected agent alert should include maintenance and powered-off VM context.

---

## Detection Severity Guidance

| Condition                                         | Suggested Context |
| ------------------------------------------------- | ----------------- |
| One test event delayed briefly                    | Informational     |
| One noncritical source channel delayed            | Low               |
| One endpoint agent disconnected unexpectedly      | Medium            |
| Security log ingestion stopped                    | High              |
| Multiple agents disconnected simultaneously       | High              |
| Wazuh indexer unavailable                         | High              |
| Splunk receiver unavailable                       | High              |
| Events indexed with incorrect timestamps          | Medium or High    |
| Monitoring blind spot during active investigation | Critical review   |
| Data loss confirmed                               | High              |
| Monitoring service disabled by unapproved user    | Critical          |

---

# Evidence Collection

## Evidence

Preserve:

* Exercise scope
* System time outputs
* Host identities
* Agent and forwarder service states
* Wazuh agent status
* Splunk forwarding status
* Network reachability tests
* Exact validation markers
* Local event evidence
* Wazuh search results
* Splunk search results
* Ingestion-delay calculations
* Duplicate-event results
* Controlled interruption timeline
* Recovery marker results
* Final service health
* Investigation notes

---

## Export Local Windows Events

Application log:

```
wevtutil epl `
    Application `
    "<EVIDENCE_PATH>\WIN11TARGET-Application.evtx"
```

Security log:

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\WIN11TARGET-Security.evtx"
```

Raw logs may contain unrelated sensitive information.

Store them privately.

---

## Export Narrow Event Summaries

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Application"
        ProviderName = "CyberLab-Ingestion-Test"
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    ProviderName,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\ex10-application-markers.csv" `
    -NoTypeInformation
```

---

## Export Wazuh Results

Export only the narrow exercise results as:

* CSV
* JSON
* Screenshot
* Saved query

Review all fields before publication.

---

## Export Splunk Results

Export:

* Marker timeline
* Delay statistics
* Host and source mapping
* Duplicate analysis
* Recovery marker

Suitable formats:

* CSV
* JSON
* Search screenshot

---

## Hash Evidence

### Windows

```
Get-FileHash `
    -Path "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

### Linux

```
sha256sum <EVIDENCE_FILE>
```

---

## Evidence Record

```
Exercise: EX-10
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

Before removing test artifacts:

1. Save all validation markers.
2. Export local source events.
3. Save Wazuh results.
4. Save Splunk results.
5. Record service states.
6. Record recovery timing.
7. Confirm no unresolved ingestion failure remains.

---

## Remove the Test File

```
Remove-Item `
    -Path "C:\CyberLab-Test\Ingestion\ingestion-marker.txt" `
    -Force `
    -ErrorAction SilentlyContinue
```

---

## Remove the Test Directory

When it is no longer required:

```
Remove-Item `
    -Path "C:\CyberLab-Test\Ingestion" `
    -Force `
    -ErrorAction SilentlyContinue
```

Retain the directory when it is an approved reusable FIM validation path.

---

## Retain or Remove the Custom Event Source

The custom source may be retained for future validation exercises.

To remove it from an authorized administrator session:

```
Remove-EventLog `
    -Source "CyberLab-Ingestion-Test"
```

Removing the source is optional.

Do not remove the Application event log itself.

---

## Confirm Wazuh Agent Health

```
Get-Service WazuhSvc
```

Confirm active status in the Wazuh dashboard.

---

## Confirm Splunk Forwarder Health

```
Get-Service SplunkForwarder
```

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Confirm an active destination.

---

## Confirm Wazuh Server Health

```
sudo docker compose ps
```

Run from the deployment directory.

Confirm all expected containers are running.

---

## Confirm Splunk Server Health

```
sudo /opt/splunk/bin/splunk status
```

Confirm Splunk Web and the receiver are available.

---

## Generate a Final Marker

Generate one final Application event after all services are restored:

```
$FinalMarker = "EX10-FINAL-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1004 `
    -EntryType Information `
    -Message "Final CyberLab ingestion health marker: $FinalMarker"

$FinalMarker
```

Confirm it appears in both Wazuh and Splunk.

---

## Final State Record

```
All local event sources healthy:
Wazuh agent running:
Wazuh agent active:
Splunk forwarder running:
Splunk destination active:
Wazuh server healthy:
Splunk server healthy:
Application marker received:
Failed logon received:
File marker received:
Recovery marker received:
Final marker received:
Unexpected duplicates:
Confirmed data loss:
Temporary files removed:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* System time was checked across all systems.
* Host identities were validated.
* Local event logs were healthy.
* Wazuh agent status was confirmed.
* Splunk forwarder status was confirmed.
* Collector connectivity was confirmed.
* Wazuh server health was reviewed.
* Splunk server health was reviewed.
* A unique Application marker was generated.
* The marker appeared locally.
* The marker appeared in Wazuh.
* The marker appeared in Splunk.
* A failed logon was validated.
* A file integrity event was validated where configured.
* Ingestion delay was measured.
* Host, source, and sourcetype values were reviewed.
* Duplicate events were checked.
* A controlled recovery test was completed where selected.
* The recovery marker was received.
* Final monitoring health was confirmed.
* Evidence was preserved.
* Detection gaps were documented.

---

## Data Quality Assessment

| Category                    | Wazuh                        | Splunk                       |
| --------------------------- | ---------------------------- | ---------------------------- |
| Application marker received | Pass / Fail                  | Pass / Fail                  |
| Failed logon received       | Pass / Fail                  | Pass / Fail                  |
| File event received         | Pass / Fail / Not Configured | Pass / Fail / Not Configured |
| Host identity correct       | Pass / Partial / Fail        | Pass / Partial / Fail        |
| Event ID parsed             | Pass / Partial / Fail        | Pass / Partial / Fail        |
| User field accurate         | Pass / Partial / Fail        | Pass / Partial / Fail        |
| Timestamp accurate          | Pass / Partial / Fail        | Pass / Partial / Fail        |
| Ingestion delay acceptable  | Pass / Needs Review / Fail   | Pass / Needs Review / Fail   |
| Duplicate events absent     | Pass / Needs Review / Fail   | Pass / Needs Review / Fail   |
| Recovery marker received    | Pass / Fail / Not Tested     | Pass / Fail / Not Tested     |
| Searchability               | Pass / Partial / Fail        | Pass / Partial / Fail        |

---

## Pipeline Assessment

| Layer                | Result                     | Evidence      |
| -------------------- | -------------------------- | ------------- |
| Activity generation  | Pass / Fail                | `<REFERENCE>` |
| Local source logging | Pass / Fail                | `<REFERENCE>` |
| Agent or forwarder   | Pass / Fail                | `<REFERENCE>` |
| Network transport    | Pass / Fail                | `<REFERENCE>` |
| Receiver             | Pass / Fail                | `<REFERENCE>` |
| Indexing             | Pass / Fail                | `<REFERENCE>` |
| Field parsing        | Pass / Partial / Fail      | `<REFERENCE>` |
| Timestamp handling   | Pass / Needs Review / Fail | `<REFERENCE>` |
| Recovery             | Pass / Fail / Not Tested   | `<REFERENCE>` |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Expected event channel
* Correct host identity
* Correct agent identity
* Event ID
* Provider name
* User context
* Source address
* Process context
* File hash
* Correct timestamp
* Ingestion timestamp
* Delay metrics
* Duplicate detection
* Agent disconnect alerting
* Source-channel silence alerting
* Recovery confirmation
* Index-routing visibility

---

## Improvements

Potential improvements include:

* Create a scheduled ingestion health search
* Maintain an expected-host inventory
* Monitor latest event time by host
* Monitor latest event time by source
* Alert on excessive ingestion delay
* Alert on future-dated events
* Alert on duplicate markers
* Monitor Wazuh agent disconnects
* Monitor Splunk forwarder connections
* Add disk-capacity alerts
* Add receiver-port monitoring
* Create synthetic heartbeat events
* Document maintenance windows
* Create an ingestion failure runbook
* Test recovery after every major configuration change

---

# Troubleshooting

## Event Does Not Exist Locally

Check:

* Correct test action
* Correct event channel
* Audit policy
* Event source
* Account used
* Event ID assumption
* Time range
* Administrative log access

Do not troubleshoot the SIEM until the source event is confirmed.

---

## Wazuh Agent Shows Disconnected

Check on WIN11TARGET:

```
Get-Service WazuhSvc
```

Review:

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 200
```

Test network access:

```
Test-NetConnection `
    -ComputerName <WAZUH_SERVER> `
    -Port <WAZUH_AGENT_PORT>
```

Check:

* Manager address
* Agent key
* Firewall
* Route
* Manager container
* Duplicate identity
* Snapshot rollback

---

## Wazuh Agent Is Active but Event Is Missing

Check:

* Event exists locally
* Event channel configured
* Dashboard time range
* Agent filter
* Rule-level filter
* Decoder
* Indexer health
* Event excluded by configuration
* Event stored without alerting

Search by exact marker text rather than rule only.

---

## Splunk Forwarder Service Is Stopped

Run as an administrator:

```
Start-Service SplunkForwarder
```

Confirm:

```
Get-Service SplunkForwarder
```

Then review `splunkd.log`.

Do not reinstall the forwarder as the first response.

---

## Forwarder Has No Active Destination

Review:

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Check:

* Receiver address
* Receiver port
* Splunk server listener
* Firewall
* Route
* SSL configuration
* Output configuration
* Server availability

---

## Splunk Receives Some Logs but Not Security

Check:

* Security event input stanza
* Administrative privileges
* Channel name
* Index routing
* Forwarder restart after configuration
* Event log access
* Source and sourcetype
* Search index

Search all indexes before changing the input.

---

## Event Appears in the Wrong Index

Review:

* Input-level index assignment
* Default index
* Props and transforms
* App configuration
* Deployment app
* Local override
* Index existence

Do not solve the issue by searching all indexes permanently.

Correct the routing configuration.

---

## Hostname Is Incorrect

Possible causes include:

* Forwarder `host` override
* Old system name
* Clone or snapshot identity
* Input-level host assignment
* Index-time transform
* Agent registration name

Review whether the same endpoint appears under multiple identities.

---

## Events Are Delayed

Check:

* System time
* Forwarder queue
* Agent queue
* Network connectivity
* Receiver health
* Indexer performance
* Disk space
* Docker health
* VM resource pressure
* Scheduled collection interval
* Event timestamp parsing

Determine whether the delay occurred at collection, transport, or indexing.

---

## Events Have Future Timestamps

Check:

* Windows timezone
* Linux timezone
* NTP source
* VMware guest time synchronization
* Splunk timestamp parsing
* Wazuh timestamp handling
* Daylight-saving settings

Correct the system clock before tuning searches around bad timestamps.

---

## Duplicate Events Appear

Check:

* Duplicate input stanzas
* Multiple apps collecting the same channel
* Native collection plus Wazuh forwarding
* Host clone
* Snapshot rollback
* Forwarder replay
* Same event indexed into multiple indexes
* Search joining identical data

Preserve one duplicate example before changing configuration.

---

## Recovery Marker Is Missing

Check:

* Event exists locally
* Service restarted successfully
* Agent or forwarder reconnected
* Input checkpoint
* Queue state
* Receiver availability
* Indexer health
* Search time range
* Exact marker text
* Event channel collection

Do not assume data loss until a broad search is completed.

---

## Wazuh Containers Are Running but Data Is Missing

A running container does not prove application health.

Check:

* Manager logs
* Indexer logs
* Dashboard logs
* Cluster health
* Disk space
* Authentication
* Agent connectivity
* Index creation
* Time range

---

## Splunk Web Loads but Events Are Missing

A working web interface does not prove receiving or indexing health.

Check:

* Receiving port
* Forwarder connection
* Internal logs
* Index status
* License state
* Disk capacity
* Search index
* Event time
* Sourcetype
* Queue warnings

---

## Disk Space Is Low

Check:

```
df -h
```

```
df -i
```

Then review:

* Old indexes
* Docker logs
* Archived evidence
* VM snapshots
* Temporary files
* Retention settings

Do not delete active index data without a documented retention and recovery plan.

---

# Public Sanitization

Remove or replace:

* Operational IP addresses
* Internal domain names
* Real usernames
* Real hostnames where sensitive
* Wazuh agent IDs
* Enrollment information
* Splunk receiver addresses
* Management ports where sensitive
* Authentication tokens
* Certificate details
* Internal URLs
* Full configuration files
* Raw Security logs
* Raw SIEM events containing unrelated data
* Evidence storage paths
* MAC addresses
* VMware identifiers

Use placeholders such as:

```
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<TEST_ACCOUNT>
<INTERNAL_IP>
<WAZUH_AGENT_PORT>
<SPLUNK_RECEIVING_PORT>
<INDEX_NAME>
<SOURCETYPE>
<MARKER>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Wazuh active agent status
* Splunk active forwarding status
* Local Application marker
* Wazuh marker search
* Splunk marker search
* Ingestion-delay statistics
* Host and source mapping
* Controlled interruption timeline
* Recovery marker
* Data-quality assessment
* Final health confirmation

Before publishing:

1. Crop unrelated information.
2. Remove operational addresses.
3. Remove real usernames.
4. Remove agent IDs.
5. Remove internal URLs.
6. Remove tokens and certificate details.
7. Remove unrelated events.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
Known validation events were generated on monitored CyberLab systems during Exercise EX-10. Each event was confirmed locally and then traced through the Wazuh and Splunk collection pipelines. Agent health, forwarding status, network transport, indexing, timestamp accuracy, ingestion delay, duplicate handling, and controlled recovery were reviewed.
```

## Findings

```
Systems tested:

Application marker:

Failed-logon marker:

File marker:

Wazuh agent state:

Splunk forwarder state:

Wazuh server state:

Splunk server state:

Wazuh application-event result:

Splunk application-event result:

Wazuh failed-logon result:

Splunk failed-logon result:

Wazuh file-event result:

Splunk file-event result:

Minimum ingestion delay:

Average ingestion delay:

Maximum ingestion delay:

Duplicate events:

Incorrect host values:

Incorrect source or sourcetype values:

Controlled interruption:

Recovery marker:

Confirmed data loss:

Detection gaps:
```

## Root Cause

Successful validation:

```
The monitored event sources, collection agents, network transport, receivers, indexes, and searches operated as expected during the validation window.
```

Failure example:

```
The validation event was generated locally but was not received by the SIEM because of a failure at the <PIPELINE_LAYER> layer.
```

## Resolution

```
The affected collection component was restored, receiver connectivity was confirmed, the recovery marker was indexed, and final end-to-end ingestion was validated.
```

## Recommendations

```
- Create scheduled synthetic heartbeat events.
- Monitor last-event time by host and source.
- Alert on agent and forwarder disconnections.
- Alert on excessive ingestion delay.
- Detect future-dated and duplicate events.
- Monitor receiver and indexer health.
- Create an ingestion failure response runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshots exist
* Exercise window documented
* Systems in scope documented
* System time validated
* Host identities validated
* VMware network validated
* Local event logs healthy
* Wazuh agent running
* Splunk forwarder running
* Wazuh server healthy
* Splunk server healthy
* Disk space reviewed
* Start time recorded

## Connectivity

* Wazuh manager reachable
* Splunk receiver reachable
* Wazuh agent active
* Splunk destination active
* Expected server listeners confirmed

## Validation Events

* Unique Application marker generated
* Application marker confirmed locally
* Controlled failed logon generated
* Failed-logon event confirmed locally
* Controlled file marker created
* File marker confirmed locally
* File hash recorded

## Wazuh

* Application marker searched
* Failed logon searched
* File marker searched
* Correct agent confirmed
* Event IDs reviewed
* Timestamps reviewed
* Field quality reviewed
* Duplicate events checked

## Splunk

* Application marker searched
* Failed logon searched
* File marker searched where configured
* Correct index confirmed
* Host field reviewed
* Source field reviewed
* Sourcetype reviewed
* EventCode reviewed
* Ingestion delay measured
* Future-dated events checked
* Duplicate events checked

## Recovery

* One collection component selected
* Baseline preserved
* Component stopped briefly
* Recovery marker generated
* Component restarted
* Connection restored
* Recovery marker received
* Recovery delay recorded
* Data loss assessed

## Cleanup

* Evidence preserved
* Test file removed
* Test directory removed or retained intentionally
* Custom event source retained or removed intentionally
* Wazuh agent healthy
* Splunk forwarder healthy
* Wazuh server healthy
* Splunk server healthy
* Final marker generated
* Final marker received by both platforms
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* End-to-end telemetry validation
* Windows Event Log administration
* Wazuh agent troubleshooting
* Wazuh server health review
* Splunk Universal Forwarder troubleshooting
* Splunk receiver and index validation
* Network transport testing
* SIEM search development
* Timestamp analysis
* Ingestion-delay measurement
* Duplicate-event analysis
* Host and source normalization
* Controlled service interruption
* Recovery validation
* Evidence preservation
* Public sanitization

---

# Lessons Learned

* A working dashboard does not prove that new events are being ingested.
* A running agent does not prove that every required event channel is collected.
* A successful TCP connection does not prove that the receiver is indexing data.
* Source events should be confirmed locally before troubleshooting the SIEM.
* Unique validation markers reduce ambiguity during pipeline testing.
* Event time and ingestion time measure different parts of the pipeline.
* Clock drift can make healthy ingestion appear delayed or out of order.
* Host, source, sourcetype, and event ID values are part of data quality.
* Duplicate events can be as harmful to investigations as missing events.
* Controlled interruption testing validates queueing and recovery behavior.
* Wazuh and Splunk may receive the same event but parse it differently.
* Missing alerts do not always mean missing telemetry.
* Ingestion health should be monitored continuously rather than only during incidents.
* Final validation should use a new event after all services are restored.

---

# Summary

This exercise validates the CyberLab’s ability to collect, transport, index, search, and recover security telemetry.

A complete validation should confirm:

* The activity occurred
* The event exists locally
* The collection service is healthy
* The network path is available
* The receiver accepts data
* The event is indexed
* The event is searchable
* Important fields are parsed
* Timestamps are accurate
* Ingestion delay is acceptable
* Duplicate events are absent or explained
* Collection recovers after interruption
* No data loss is observed
* Final monitoring health is confirmed

The exercise is complete only after all expected validation markers are accounted for, evidence is preserved, temporary artifacts are removed, and pipeline improvements are documented.

