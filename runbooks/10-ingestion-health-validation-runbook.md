# Ingestion Health Validation Runbook

## Runbook Overview

This runbook provides a structured process for investigating telemetry ingestion failures, delays, gaps, duplication, and data-quality problems within the CyberLab.

It applies to situations involving:

* A monitored endpoint no longer sending events
* Wazuh agent disconnection
* Splunk Universal Forwarder connection failure
* Missing Windows Event Logs
* Missing Wazuh File Integrity Monitoring events
* Missing Sysmon or Microsoft Defender telemetry
* Events arriving in the wrong index
* Incorrect host, source, or sourcetype assignment
* Excessive ingestion delay
* Duplicate events
* Future-dated or incorrectly timestamped events
* Collector recovery after a controlled or unexpected interruption
* Failed ingestion-health validation exercises

The runbook helps the analyst determine:

* Whether the source event was generated
* Whether the event exists locally
* Whether the collection agent read the event
* Whether the endpoint could reach the receiving server
* Whether the receiver accepted the data
* Whether the event was indexed
* Whether the event was searchable
* Whether important fields were parsed correctly
* Whether events were delayed, duplicated, or routed incorrectly
* Whether collection recovered without data loss
* Which telemetry-pipeline layer caused the failure

---

## Runbook ID

```
RB-10
```

---

## Related Exercise

```
EX-10 – Ingestion Health Validation
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Detection engineers
* SIEM administrators
* Windows administrators
* Linux administrators
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Windows Event Log collection
* Wazuh agent telemetry
* Wazuh File Integrity Monitoring
* Wazuh manager, indexer, and dashboard services
* Splunk Universal Forwarder
* Splunk receiving and indexing
* Sysmon telemetry
* Microsoft Defender events
* VMware host-only network transport
* Controlled ingestion validation markers

This runbook does not authorize:

* Clearing logs before evidence collection
* Deleting indexes
* Deleting Wazuh agents as an initial troubleshooting step
* Reinstalling collectors before the cause is understood
* Disabling security logging to reduce volume
* Removing input checkpoints
* Restoring snapshots before preserving current evidence
* Testing systems outside the CyberLab
* Publishing internal addresses, credentials, tokens, or certificates

---

## Primary Telemetry Path

```
Event-Generating Activity
          |
          v
Local Event Source
          |
          v
Collection Agent or Forwarder
          |
          v
CyberLab Network Transport
          |
          v
Receiver or Manager
          |
          v
Indexer and Storage
          |
          v
Search, Alert, and Dashboard
```

A healthy dashboard does not prove that every layer is operating correctly.

---

## Validation Layers

| Layer      | Primary Question                          |
| ---------- | ----------------------------------------- |
| Activity   | Did the expected action occur?            |
| Source     | Was an event written locally?             |
| Collection | Did the agent or forwarder read it?       |
| Transport  | Could the endpoint reach the receiver?    |
| Receiver   | Did the server accept the data?           |
| Indexing   | Was the event stored?                     |
| Search     | Can the event be found?                   |
| Parsing    | Are key fields accurate?                  |
| Timing     | Did the event arrive promptly?            |
| Recovery   | Did collection resume after interruption? |

---

## Primary Data Sources

Use the available data sources in this order:

1. Exact validation marker or expected event
2. Local Windows Event Logs
3. Wazuh agent log
4. Splunk Universal Forwarder log
5. Endpoint service state
6. Network reachability tests
7. Wazuh agent status
8. Wazuh manager and indexer logs
9. Splunk internal logs
10. Splunk receiver and index status
11. System time and timezone configuration
12. VMware network configuration
13. Disk and resource health
14. Change and maintenance records

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh agent disconnected alert
* Splunk forwarder missing-data alert
* Host silence detection
* Source-channel silence detection
* Excessive ingestion-delay detection
* Missing expected validation marker
* User or administrator report
* Failed CyberLab exercise
* Dashboard data gap
* Detection rule suddenly returning zero events
* Unexpected duplicate-event increase
* Incorrect timestamp or host-field observation

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Affected endpoint:
Affected event source:
Expected index:
Expected sourcetype:
Expected Wazuh agent:
Last known event time:
Last known ingestion time:
Observed delay:
Collector service:
Receiver:
Alert severity:
Analyst:
```

---

## Initial Analyst Questions

Determine immediately:

* Is one source affected or several?
* Is one endpoint affected or the entire environment?
* Does the source event exist locally?
* Is the endpoint powered on?
* Is the collection service running?
* Is the receiver reachable?
* Is the event missing or merely difficult to find?
* Did a configuration change occur?
* Is system time correct?
* Is disk space available?
* Did a snapshot rollback occur?
* Is maintenance in progress?
* Is the issue causing a security-monitoring blind spot?

---

# Severity Classification

## Informational

Use informational severity when:

* A validation marker was delayed briefly
* The affected event source is noncritical
* Maintenance explains the interruption
* Collection recovered automatically
* No meaningful data loss occurred

---

## Low

Use low severity when:

* One noncritical event channel is delayed
* One lab endpoint is temporarily silent
* The issue is caused by a known shutdown or restart
* Data remains available locally
* Recovery occurs without data loss

---

## Medium

Use medium severity when:

* One endpoint unexpectedly stops reporting
* A required event channel is missing
* Events are routed to the wrong index
* Field parsing prevents a detection from functioning
* Ingestion delay is significant
* Duplicate events materially affect searches
* Recovery requires administrative action

---

## High

Use high severity when:

* Security log ingestion stops
* Microsoft Defender or Sysmon telemetry disappears
* Multiple endpoints stop reporting
* The Wazuh manager, indexer, or Splunk receiver is unavailable
* Data loss is suspected
* A monitoring blind spot exists during suspicious activity
* Timestamp errors make the incident timeline unreliable
* A collector appears to have been disabled without authorization

---

## Critical

Use critical severity when:

* Monitoring fails during an active security incident
* Domain controller authentication telemetry is unavailable
* Multiple security platforms lose data simultaneously
* Confirmed telemetry destruction or tampering occurs
* Index data is corrupted or inaccessible
* An attacker disables collection or logging
* The analyst cannot determine the scope of lost visibility

---

# Initial Triage

## Step 1: Define the Missing Data

Identify exactly what is expected.

Record:

```
Expected host:
Expected event channel:
Expected event ID:
Expected provider:
Expected marker:
Expected file path:
Expected Wazuh rule:
Expected Splunk index:
Expected source:
Expected sourcetype:
Expected time window:
```

Avoid beginning with a broad statement such as “Splunk is not working.”

---

## Step 2: Confirm the Investigation Time Window

Record absolute timestamps and timezone.

```
First expected event:
Last known healthy event:
First missing event:
Alert generation time:
Current time:
Timezone:
```

Use a wider search window when time synchronization is uncertain.

---

## Step 3: Confirm System Time

### Windows Endpoint

```
Get-Date
```

```
w32tm /query /status
```

```
w32tm /query /source
```

### Linux Server

```
timedatectl
```

Record:

```
Endpoint time:
Wazuh server time:
Splunk server time:
Largest difference:
Timezone mismatch:
```

Clock drift can make healthy ingestion appear missing or delayed.

---

## Step 4: Confirm Endpoint Identity

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
hostnamectl
```

Confirm the identity matches:

* Wazuh agent name
* Splunk host field
* Asset inventory
* VMware virtual machine
* Expected source system

---

## Step 5: Confirm Endpoint Power and Network State

### Windows

```
Get-NetAdapter |
    Select-Object Name, Status, InterfaceDescription
```

```
Get-NetIPConfiguration
```

### Linux

```
ip address
```

```
ip route
```

Review:

* Host-only adapter
* NAT adapter where temporarily required
* Incorrect bridged adapter
* Source address
* Default route
* Duplicate address
* Disconnected virtual adapter

---

# Source Event Validation

## Confirm the Event Exists Locally

Do not troubleshoot the collector until the source event is confirmed.

### Application Event Example

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Application"
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<VALIDATION_MARKER>"
}
```

### Security Event Example

Run as an authorized administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = <EVENT_ID>
        StartTime = (Get-Date).AddHours(-1)
    }
```

### PowerShell Event Example

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-1)
    }
```

### Defender Event Example

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Windows Defender/Operational"
        StartTime = (Get-Date).AddHours(-1)
    }
```

---

## Confirm the Event Channel Is Enabled

```
Get-WinEvent -ListLog "<EVENT_LOG_NAME>" |
    Select-Object `
        LogName,
        IsEnabled,
        RecordCount,
        MaximumSizeInBytes,
        LogMode
```

Review:

* Channel enabled state
* Record count
* Maximum size
* Overwrite behavior
* Access permissions
* Recent events

---

## Source Failure Indicators

The problem is at the source layer when:

* The test action did not generate an event
* Audit policy is disabled
* The event channel is disabled
* The expected event ID assumption is incorrect
* The event log is inaccessible
* The system time is wrong
* The event was overwritten before collection
* The local application failed before writing the event

---

## Source-Layer Response

* Correct audit or application configuration
* Enable the required event channel
* Correct the validation method
* Increase log size where justified
* Repair time synchronization
* Generate a new harmless validation marker
* Confirm the event locally before continuing

Do not modify security policy broadly during active incident response without approval.

---

# Wazuh Agent Investigation

## Confirm Wazuh Service State

Run on the monitored Windows endpoint:

```
Get-Service WazuhSvc
```

Alternative discovery:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Record:

```
Service name:
Status:
Start type:
Last restart:
```

---

## Review Wazuh Agent Log

Typical location:

```
C:\Program Files (x86)\ossec-agent\ossec.log
```

Review recent entries:

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 200
```

Look for:

* Successful manager connection
* Connection refused
* Reconnection attempts
* Invalid configuration
* Event-channel subscription failure
* Queue warnings
* Authentication failure
* Duplicate agent identity
* File Integrity Monitoring errors

---

## Confirm Wazuh Manager Configuration

Review privately:

```
Select-String `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" `
    -Pattern "<address>|<port>|<protocol>|<localfile>|<syscheck>"
```

Validate:

* Manager address
* Manager port
* Protocol
* Event-channel configuration
* File Integrity Monitoring paths
* XML syntax
* Exclusions
* Agent name

Do not publish operational manager addresses or enrollment values.

---

## Confirm Wazuh Network Reachability

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
Source address:
Interface used:
```

A successful TCP test proves network reachability, not successful agent authentication or indexing.

---

## Confirm Agent Status in Wazuh

Review:

* Active
* Disconnected
* Never connected
* Pending
* Last keepalive
* Agent name
* Agent ID
* Operating system
* Manager assignment

Investigate duplicate names or IDs, especially after:

* Snapshot rollback
* VM cloning
* Agent reinstallation
* Host rename

---

## Restart the Wazuh Agent

Restart only after preserving logs and configuration evidence.

Run as an authorized administrator:

```
Restart-Service WazuhSvc
```

Confirm:

```
Get-Service WazuhSvc
```

Then review:

```
Get-Content `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Tail 100
```

---

## Wazuh Agent Failure Indicators

* Service stopped
* Invalid `ossec.conf`
* Manager unreachable
* Incorrect manager address
* Agent key mismatch
* Duplicate agent identity
* Event channel not configured
* FIM path excluded
* Queue saturation
* Snapshot rollback
* Insufficient permissions

---

# Splunk Universal Forwarder Investigation

## Confirm Forwarder Service State

Run on the monitored Windows endpoint:

```
Get-Service SplunkForwarder
```

Alternative discovery:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Record:

```
Service name:
Status:
Start type:
Last restart:
```

---

## Review Splunk Forwarder Log

Typical location:

```
C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log
```

Review recent entries:

```
Get-Content `
    -Path "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" `
    -Tail 200
```

Look for:

* TcpOutputProc errors
* Connection refused
* Active destination unavailable
* SSL or certificate error
* Blocked queue
* Indexer acknowledgement issue
* Input failure
* Permission error
* Parsing or checkpoint problem
* Configuration syntax issue

---

## Confirm Forwarding Destination

Run from an authorized administrative session:

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Review:

* Active forwards
* Inactive forwards
* Destination address
* Destination port
* Connection status

Do not publish operational receiver details.

---

## Confirm Configured Inputs

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list monitor
```

Review relevant configuration files privately:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf
```

Also review installed app configurations where applicable.

---

## Confirm Splunk Network Reachability

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
Source address:
Interface used:
```

---

## Restart the Splunk Forwarder

Restart only after preserving logs and configuration evidence.

```
Restart-Service SplunkForwarder
```

Confirm:

```
Get-Service SplunkForwarder
```

Then confirm the active destination:

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

---

## Splunk Forwarder Failure Indicators

* Service stopped
* No active destination
* Receiver unreachable
* Receiver port incorrect
* Input stanza disabled
* Incorrect index assignment
* Event channel typo
* Configuration conflict
* Blocked processing queue
* SSL mismatch
* Snapshot rollback
* Checkpoint or state problem
* Insufficient event-log permissions

---

# Network Transport Investigation

## Confirm the Intended Route

### Windows

```
Get-NetRoute |
    Sort-Object DestinationPrefix
```

### Linux

```
ip route
```

Confirm the endpoint uses the expected CyberLab network.

---

## Test Name Resolution

When hostnames are used:

```
Resolve-DnsName "<COLLECTOR_HOSTNAME>"
```

Also test the configured address directly.

A DNS failure may affect a collector configured by hostname while direct IP connectivity still works.

---

## Test Collector Ports

```
Test-NetConnection `
    -ComputerName <COLLECTOR_SERVER> `
    -Port <COLLECTOR_PORT> `
    -InformationLevel Detailed
```

Review:

* Name resolution
* Remote address
* Source address
* Interface
* TCP result

---

## Review Endpoint Firewall

```
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

Review rules related to the collector or service:

```
Get-NetFirewallRule |
    Where-Object {
        $_.DisplayName -match "Wazuh|Splunk|Forwarder"
    } |
Select-Object `
    DisplayName,
    Enabled,
    Direction,
    Action,
    Profile
```

Do not disable the firewall as a troubleshooting shortcut.

---

## Transport Failure Indicators

* Wrong collector address
* Wrong receiver port
* VMware adapter disconnected
* Incorrect route
* Firewall block
* DNS failure
* Collector listener unavailable
* Duplicate address
* Network-profile mismatch
* Bridged or NAT adapter used unintentionally

---

# Wazuh Server Investigation

## Confirm Docker Services

Run on WAZUH-SERVER:

```
sudo docker ps
```

From the Wazuh Compose directory:

```
sudo docker compose ps
```

Confirm expected services such as:

* Wazuh manager
* Wazuh indexer
* Wazuh dashboard

---

## Review Container Status

```
sudo docker ps \
    --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Look for:

* Exited containers
* Restart loops
* Unhealthy status
* Missing port mappings
* Unexpected uptime reset

---

## Review Wazuh Manager Logs

```
sudo docker logs \
    --tail 200 \
    <WAZUH_MANAGER_CONTAINER>
```

Look for:

* Agent connection failure
* Authentication failure
* Queue warnings
* Decoder or rule error
* Indexer connection failure
* Disk issue
* Certificate issue

---

## Review Wazuh Indexer Logs

```
sudo docker logs \
    --tail 200 \
    <WAZUH_INDEXER_CONTAINER>
```

Look for:

* Unassigned shards
* Read-only index
* Disk watermark
* Authentication failure
* Cluster health problem
* Index creation failure
* Certificate issue

---

## Review Wazuh Dashboard Logs

```
sudo docker logs \
    --tail 200 \
    <WAZUH_DASHBOARD_CONTAINER>
```

A dashboard failure does not necessarily mean event ingestion stopped.

---

## Confirm Disk Space

```
df -h
```

```
df -i
```

Review:

* Root volume
* Docker storage
* Wazuh index storage
* Available inodes

Low disk space can cause indexing failures or read-only indexes.

---

## Wazuh Server Failure Indicators

* Manager container stopped
* Indexer unavailable
* Dashboard-only failure
* Indexer disk watermark reached
* Unassigned shards
* Certificate or authentication failure
* Container restart loop
* Queue saturation
* Manager-to-indexer communication failure

---

# Splunk Server Investigation

## Confirm Splunk Service State

The CyberLab installation currently uses:

```
/opt/splunk
```

Check status:

```
sudo /opt/splunk/bin/splunk status
```

The current lab startup method may use:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

This reflects the current lab state.

A future hardening phase should migrate Splunk to a dedicated service account.

---

## Confirm Receiver Listener

```
sudo ss -lntp |
    grep "<SPLUNK_RECEIVING_PORT>"
```

Confirm:

* Expected port
* Listening state
* Correct process
* Intended network interface

---

## Review Splunk Internal Logs

```
index=_internal earliest=-30m
| table _time host source sourcetype component log_level message
| sort _time
```

Search common forwarding and indexing errors:

```
index=_internal earliest=-30m
(
    "TcpOutputProc"
    OR "connection refused"
    OR "blocked"
    OR "queue"
    OR "index"
    OR "license"
    OR "disk"
)
| table _time host component log_level message
| sort _time
```

---

## Confirm Recent Forwarder Connections

```
index=_internal earliest=-30m
source="*metrics.log"
group=tcpin_connections
| stats
    latest(_time) as last_seen
    values(sourceIp) as source_addresses
    values(version) as versions
    by hostname
| convert ctime(last_seen)
| sort - last_seen
```

Field availability depends on the Splunk version and configuration.

---

## Confirm Index Activity

```
index=_internal earliest=-24h
source="*metrics.log"
group=per_index_thruput
| stats
    sum(kb) as total_kb
    by series
| sort - total_kb
```

Review whether the expected index has recent volume.

---

## Confirm Disk Space

```
df -h
```

```
df -i
```

Review:

* Root file system
* Splunk index path
* Log storage
* Available inodes

---

## Splunk Server Failure Indicators

* Splunk service stopped
* Receiver port not listening
* License problem
* Index unavailable
* Disk full
* Parsing queue blocked
* Indexing queue blocked
* Incorrect index configuration
* Forwarder connections absent
* Search-head time issue
* Internal errors or repeated restarts

---

# Search-Layer Investigation

## Search All Splunk Indexes

Before declaring an event missing:

```
index=* earliest=-2h
"<EXACT_VALIDATION_MARKER>"
```

Search by:

* Exact marker
* Event ID
* Provider
* Hostname
* Filename
* Account
* Source address

---

## Review Event Distribution

```
index=* earliest=-2h
| stats count by index host source sourcetype
| sort index host source
```

This may reveal that the event was routed incorrectly.

---

## Search Wazuh Broadly

Remove narrow filters involving:

* Agent
* Rule level
* Module
* Event ID
* Time range
* Dashboard category
* Alert-only views

An event can be collected without generating a high-level alert.

---

## Search Failure Indicators

The problem is at the search layer when:

* Data is indexed but the query uses the wrong field
* The wrong index is selected
* The time range excludes the event
* A dashboard filter hides the event
* The event has an unexpected sourcetype
* The host field changed
* The event timestamp is parsed incorrectly

---

# Data Quality Investigation

## Validate Host Assignment

Splunk:

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
| stats
    count
    values(source) as sources
    values(sourcetype) as sourcetypes
    by host
```

Confirm:

* Expected hostname
* No old hostname
* No duplicate host identity
* No IP address incorrectly used as the permanent host value

---

## Validate Source and Sourcetype

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
| table
    _time
    _indextime
    host
    index
    source
    sourcetype
    EventCode
    ProviderName
    Message
```

Incorrect sourcetypes may cause:

* Missing field extraction
* Timestamp errors
* Detection failures
* Dashboard failures
* CIM incompatibility

---

## Validate Wazuh Agent Identity

Review:

* Agent name
* Agent ID
* Manager
* Operating system
* Source host
* Last keepalive

A newly generated agent ID after reinstallation or snapshot restoration may cause duplicate identities.

---

## Validate Field Extraction

Important fields may include:

```
EventCode
ProviderName
TargetUserName
SubjectUserName
SourceNetworkAddress
LogonType
ProcessName
CommandLine
agent.name
rule.id
rule.level
syscheck.path
```

Always compare normalized fields with the raw event.

---

# Ingestion Delay Analysis

## Calculate Splunk Delay

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    index
    source
    sourcetype
    Message
```

---

## Summarize Delay by Host and Source

```
index=* earliest=-24h
| eval ingestion_delay_seconds=_indextime-_time
| stats
    count
    min(ingestion_delay_seconds) as minimum_delay
    avg(ingestion_delay_seconds) as average_delay
    perc95(ingestion_delay_seconds) as p95_delay
    max(ingestion_delay_seconds) as maximum_delay
    by host source sourcetype
| sort - maximum_delay
```

---

## Search Excessively Delayed Events

```
index=* earliest=-24h
| eval ingestion_delay_seconds=_indextime-_time
| where ingestion_delay_seconds > <DELAY_THRESHOLD_SECONDS>
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    index
    source
    sourcetype
    EventCode
| sort - ingestion_delay_seconds
```

---

## Search Future-Dated Events

```
index=* earliest=-24h
| eval future_offset_seconds=_time-now()
| where future_offset_seconds > 60
| table
    _time
    host
    source
    sourcetype
    future_offset_seconds
| sort - future_offset_seconds
```

---

## Common Delay Causes

* Clock drift
* Timezone mismatch
* Forwarder queueing
* Wazuh queueing
* Receiver unavailable
* Indexer resource pressure
* Disk pressure
* Scheduled collection interval
* VM suspension
* Network interruption
* Incorrect timestamp extraction
* Events generated before a snapshot pause

---

# Duplicate Event Investigation

## Search Exact Marker Duplicates

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
| stats
    count
    values(index) as indexes
    values(source) as sources
    values(sourcetype) as sourcetypes
    by host EventCode Message
| where count > 1
```

---

## Search Raw Duplicates

Use a narrow time range:

```
index=* earliest=-30m
| stats
    count
    values(index) as indexes
    values(source) as sources
    by host _time _raw
| where count > 1
| sort - count
```

---

## Common Duplicate Causes

* Same event channel configured more than once
* Multiple Splunk apps collect the same log
* Native collection and Wazuh forwarding overlap
* Duplicate forwarders
* Snapshot rollback
* Reinstalled forwarder state
* Receiver replay
* Search joins or append operations
* Same data stored in multiple indexes

Similar events from an endpoint and domain controller are not automatically duplicates.

---

# Missing Channel Investigation

## Confirm Other Channels Continue Reporting

```
index=windows earliest=-2h
| stats
    latest(_time) as latest_event
    count
    by host source
| eval event_age_minutes=round((now()-latest_event)/60, 2)
| convert ctime(latest_event)
| sort - event_age_minutes
```

If Application and System logs continue but Security stops, the issue is likely source- or input-specific rather than total forwarder failure.

---

## Review Wazuh Event-Channel Configuration

Confirm the required channel exists in `ossec.conf`.

Potential examples include:

```
Security
System
Application
Microsoft-Windows-PowerShell/Operational
Microsoft-Windows-Windows Defender/Operational
Microsoft-Windows-Sysmon/Operational
```

---

## Review Splunk Windows Event Log Inputs

Review active stanzas in `inputs.conf`.

Confirm:

* Correct channel name
* `disabled = 0`
* Correct index
* Correct rendering mode
* No duplicate stanza
* App and local configuration precedence

---

# Controlled Recovery Validation

## Purpose

A controlled recovery test verifies that an agent or forwarder:

* Reconnects successfully
* Resumes collection
* Recovers queued events
* Does not create duplicates
* Does not lose events

Perform this test only when it is safe and approved.

---

## Wazuh Recovery Test

### Stop the Agent

Run as an authorized administrator:

```
Stop-Service WazuhSvc
```

### Generate a Unique Local Marker

```
$Marker = "RB10-WAZUH-RECOVERY-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1010 `
    -EntryType Information `
    -Message "Authorized Wazuh recovery validation marker: $Marker"

$Marker
```

Confirm it exists locally.

### Start the Agent

```
Start-Service WazuhSvc
```

### Validate Recovery

Confirm:

* Agent returns to active
* Keepalive updates
* Marker becomes searchable
* No duplicate agent appears
* Recovery delay is recorded
* No data loss is observed

---

## Splunk Recovery Test

### Stop the Forwarder

```
Stop-Service SplunkForwarder
```

### Generate a Unique Local Marker

```
$Marker = "RB10-SPLUNK-RECOVERY-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1011 `
    -EntryType Information `
    -Message "Authorized Splunk recovery validation marker: $Marker"

$Marker
```

### Start the Forwarder

```
Start-Service SplunkForwarder
```

### Validate Recovery

```
index=* earliest=-1h
"<EXACT_RECOVERY_MARKER>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    index
    source
    sourcetype
    EventCode
    Message
```

Confirm:

* Destination becomes active
* Marker is indexed
* Delay is measured
* No duplicate is present
* No data loss is observed

---

# Investigation Decision Tree

```
Ingestion Health Alert
          |
          v
Does the expected event exist locally?
          |
     +----+----+
     |         |
    No        Yes
     |         |
Investigate   Is the agent or
source event  forwarder running?
generation         |
              +----+----+
              |         |
             No        Yes
              |         |
         Preserve logs  Can the collector
         and restore    reach the receiver?
         service             |
                        +----+----+
                        |         |
                       No        Yes
                        |         |
                 Investigate     Is the event
                 network path    indexed anywhere?
                                      |
                                 +----+----+
                                 |         |
                                No        Yes
                                 |         |
                         Review receiver, Review index,
                         indexing and    host, source,
                         disk health     sourcetype,
                                         time and search
```

---

# Common Investigation Scenarios

## Scenario 1: Endpoint Was Powered Off

Indicators:

* Agent disconnected at shutdown time
* No new local events
* VM intentionally stopped
* No unexpected service failure

Response:

* Confirm planned shutdown
* Document the maintenance context
* Confirm collection resumes after startup
* Close as expected interruption

---

## Scenario 2: Wazuh Agent Service Stopped

Indicators:

* Wazuh agent disconnected
* `WazuhSvc` stopped
* Local events continue
* Manager remains healthy

Response:

* Preserve the agent log
* Determine why the service stopped
* Restart the service
* Validate a new marker
* Escalate unauthorized service stoppage

---

## Scenario 3: Splunk Forwarder Has No Active Destination

Indicators:

* Service is running
* `list forward-server` shows no active destination
* Receiver port test fails or Splunk listener is absent
* `splunkd.log` shows TcpOutputProc errors

Response:

* Validate receiver address and port
* Confirm Splunk listener
* Correct transport or configuration
* Validate queued-event recovery

---

## Scenario 4: Events Are in the Wrong Index

Indicators:

* Exact marker found through `index=*`
* Expected index has no event
* Host and source appear valid
* Input-level index assignment is incorrect

Response:

* Preserve the incorrectly routed event
* Correct the input or routing configuration
* Validate with a new marker
* Do not rely permanently on `index=*`

---

## Scenario 5: Security Events Missing but Other Logs Continue

Indicators:

* Application and System logs are present
* Security events exist locally
* Security input or channel is missing
* Forwarder and network remain healthy

Response:

* Review permissions
* Review event-channel input
* Review Wazuh or Splunk configuration
* Restart only after preserving evidence
* Validate with a known harmless Security event

---

## Scenario 6: Events Arrive with Large Delay

Indicators:

* `_indextime` is much later than `_time`
* Collector queue warnings
* Server resource or disk pressure
* Receiver interruption occurred

Response:

* Determine where queueing occurred
* Correct network, receiver, or capacity issue
* Confirm backlog drains
* Monitor delay until normal

---

## Scenario 7: Duplicate Events

Indicators:

* Exact same raw event appears multiple times
* Same source configured in more than one input
* Multiple indexes receive the same event
* Snapshot or forwarder state changed

Response:

* Preserve duplicate examples
* Identify every collection path
* Remove only the unintended path
* Validate with a unique marker
* Confirm detection counts normalize

---

## Scenario 8: Snapshot Rollback Created Identity Conflict

Indicators:

* Duplicate Wazuh agent
* Old hostname returns
* Splunk forwarder replays data
* Event timestamps precede current VM state
* Agent key or checkpoint mismatch

Response:

* Preserve current evidence
* Identify the authoritative VM state
* Correct agent and forwarder identity
* Avoid repeated snapshot restoration
* Validate new telemetry end to end

---

## Scenario 9: Wazuh Dashboard Loads but Events Are Missing

Indicators:

* Dashboard accessible
* Manager or indexer errors present
* Agents may appear active
* Searches return stale results

Response:

* Review manager and indexer health
* Check disk and cluster state
* Confirm new marker indexing
* Do not treat dashboard availability as ingestion proof

---

## Scenario 10: Splunk Web Loads but Receiver Is Down

Indicators:

* Splunk Web accessible
* Receiver port not listening
* Forwarders show inactive destination
* Internal logs report connection problems

Response:

* Restore or enable the receiver
* Confirm network access
* Validate forwarder reconnection
* Measure backlog recovery delay

---

# Containment

## When Containment Is Not Required

Containment may not be required when:

* Maintenance explains the interruption
* No unauthorized changes occurred
* Data remains locally available
* Collection recovers without loss
* No active incident depends on the missing telemetry

Document why containment was unnecessary.

---

## Endpoint Containment

Consider isolation when:

* A collector was disabled without authorization
* Logging was altered during suspicious activity
* Malware may be interfering with telemetry
* The endpoint is actively compromised
* Evidence tampering is suspected

Do not isolate a central collector without reviewing operational impact.

---

## Account Containment

With appropriate authorization:

* Restrict the account that disabled logging
* Revoke suspicious administrative sessions
* Reset compromised credentials
* Remove unauthorized privilege

---

## Configuration Containment

* Preserve current configurations
* Stop unauthorized deployment automation
* Prevent policy from repeatedly applying a bad configuration
* Restrict access to collector and receiver settings
* Avoid broad emergency changes that obscure the original cause

---

# Eradication and Correction

## Source-Layer Correction

* Restore audit policy
* Enable required log channels
* Correct application logging
* Increase log capacity
* Repair system time
* Correct local permissions

---

## Agent or Forwarder Correction

* Correct manager or receiver address
* Correct port
* Correct input stanza
* Correct agent identity
* Correct permissions
* Remove duplicate input
* Repair configuration syntax
* Restart the affected service

Reinstall only when other documented recovery steps fail and evidence is preserved.

---

## Network Correction

* Restore VMware adapter
* Correct route
* Correct DNS
* Correct narrow firewall rule
* Restore receiver listener
* Remove duplicate address

---

## Server Correction

* Restart failed container or service
* Resolve disk-space issue
* Restore indexer health
* Correct certificate or authentication problem
* Correct index configuration
* Resolve queue blockage
* Restore receiver configuration

---

# Recovery

## Generate a Final Validation Marker

Run on the affected endpoint:

```
$FinalMarker = "RB10-FINAL-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-EventLog `
    -LogName Application `
    -Source "CyberLab-Ingestion-Test" `
    -EventId 1012 `
    -EntryType Information `
    -Message "Final ingestion recovery validation marker: $FinalMarker"

$FinalMarker
```

Confirm the marker:

1. Exists locally
2. Appears in Wazuh
3. Appears in Splunk
4. Uses the correct host
5. Uses the correct index
6. Uses the correct source and sourcetype
7. Has an accurate timestamp
8. Has acceptable ingestion delay
9. Is not duplicated

---

## Confirm Wazuh Health

```
Get-Service WazuhSvc
```

Confirm:

* Agent active
* Keepalive recent
* Correct agent identity
* Manager and indexer healthy
* New marker searchable

---

## Confirm Splunk Health

```
Get-Service SplunkForwarder
```

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

Confirm:

* Forwarder running
* Active destination present
* Receiver listening
* New marker indexed
* Correct index and fields

---

## Confirm No Continuing Gap

Splunk:

```
index=* earliest=-1h
| stats
    latest(_time) as latest_event
    count
    by host source
| eval event_age_minutes=round((now()-latest_event)/60, 2)
| convert ctime(latest_event)
| sort - event_age_minutes
```

Review Wazuh agent status and recent events for the affected endpoint.

---

# Escalation Criteria

Escalate immediately when:

* Security logs are unavailable during an active incident
* Multiple endpoints stop reporting
* Domain controller telemetry is missing
* Wazuh manager or indexer remains unavailable
* Splunk receiver or indexes remain unavailable
* Data loss is confirmed
* Logging was disabled without authorization
* Evidence suggests telemetry tampering
* Time errors invalidate the incident timeline
* Duplicate or corrupted data affects incident decisions
* Recovery fails
* The analyst cannot determine the scope of lost visibility

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
First observed:
Last known healthy event:
Affected hosts:
Affected event sources:
Expected indexes:
Collector states:
Receiver states:
Network test results:
Disk status:
Time synchronization status:
Data loss suspected:
Data loss confirmed:
Duplicate data:
Incorrect routing:
Containment completed:
Recovery attempted:
Final validation result:
Evidence locations:
Outstanding questions:
Recommended next action:
```

---

# Evidence Collection

## Minimum Evidence

Preserve:

* Original alert
* Exact validation marker
* Local source event
* Event-channel state
* Wazuh service state
* Wazuh agent log
* Wazuh agent status
* Splunk forwarder service state
* Splunk forwarder log
* Active destination status
* Network test results
* Wazuh server logs
* Splunk internal logs
* Disk-space results
* Time-synchronization results
* Search results
* Delay calculations
* Duplicate-event examples
* Configuration changes
* Recovery validation
* Analyst notes

---

## Export Local Event Logs

```
wevtutil epl `
    Application `
    "<EVIDENCE_PATH>\Application.evtx"
```

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

Raw logs may contain unrelated sensitive information.

Store them privately.

---

## Preserve Wazuh Agent Log

```
Copy-Item `
    -Path "C:\Program Files (x86)\ossec-agent\ossec.log" `
    -Destination "<EVIDENCE_PATH>\wazuh-agent-ossec.log" `
    -Force
```

---

## Preserve Splunk Forwarder Log

```
Copy-Item `
    -Path "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" `
    -Destination "<EVIDENCE_PATH>\splunk-forwarder-splunkd.log" `
    -Force
```

---

## Export a Narrow Event Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Application"
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Where-Object {
    $_.Message -match "RB10-|EX10-"
} |
Select-Object `
    TimeCreated,
    Id,
    ProviderName,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\ingestion-markers.csv" `
    -NoTypeInformation
```

---

## Hash Evidence

### Windows

```
Get-FileHash `
    -LiteralPath "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

### Linux

```
sha256sum <EVIDENCE_FILE>
```

---

## Evidence Record

```
Runbook: RB-10
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

Common benign causes include:

* Endpoint powered off
* VM suspended
* Planned reboot
* Snapshot operation
* Agent upgrade
* Forwarder upgrade
* Receiver maintenance
* Index maintenance
* Temporary network interruption
* Delayed scheduled collection
* Timezone display difference
* Incorrect dashboard filter
* Wrong search index
* Approved CyberLab recovery test

A benign interruption should still be documented when it creates a meaningful monitoring gap.

---

# Detection Gaps

Document whether the investigation lacked:

* Expected-host inventory
* Latest event time by host
* Latest event time by source
* Agent disconnect alert
* Forwarder connection alert
* Receiver health monitoring
* Disk-capacity monitoring
* Ingestion-delay monitoring
* Future-event detection
* Duplicate-event detection
* Index-routing validation
* Host-field validation
* Sourcetype validation
* Synthetic heartbeat events
* Maintenance-window context
* Data-loss confirmation process
* Recovery validation process

---

# Detection Improvement Recommendations

Potential improvements include:

* Create recurring synthetic heartbeat events
* Monitor last event by host
* Monitor last event by source channel
* Alert on Wazuh agent disconnections
* Monitor Splunk forwarder connections
* Alert when security logs stop reporting
* Monitor ingestion-delay percentiles
* Detect future-dated events
* Detect exact duplicates
* Detect unexpected index routing
* Maintain an expected-host inventory
* Maintain expected index and sourcetype mappings
* Add Wazuh manager and indexer health checks
* Add Splunk receiver-port monitoring
* Add disk-space and inode alerts
* Document maintenance windows
* Validate ingestion after every configuration change
* Create backup and restoration procedures for collector configuration

---

# MITRE ATT&CK Context

An ingestion failure does not automatically represent adversary behavior.

When logging or monitoring is intentionally disabled by an attacker, the activity may relate to:

```
T1562.001 – Impair Defenses
```

When event logs are cleared, activity may relate to:

```
T1070.001 – Clear Windows Event Logs
```

Apply ATT&CK mappings only when the observed behavior supports them.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Affected endpoint:
Affected source:
Expected event:
Last known healthy event:
First missing event:
Local event present:
Collector service state:
Collector log findings:
Receiver reachable:
Receiver state:
Index state:
Search result:
Host field:
Source:
Sourcetype:
Timestamp accuracy:
Ingestion delay:
Duplicate events:
Data loss:
Root cause:
Severity:
Containment:
Correction:
Recovery:
Detection gaps:
```

---

## Root Cause Categories

Select one:

```
Endpoint powered off
VM suspended or snapshot operation
Local event not generated
Event channel disabled
Audit policy misconfiguration
Wazuh agent stopped
Wazuh agent configuration error
Wazuh identity conflict
Splunk forwarder stopped
Splunk input configuration error
Splunk destination unavailable
Network transport failure
Receiver unavailable
Indexer failure
Disk-capacity issue
Incorrect index routing
Incorrect host or sourcetype
Timestamp parsing problem
Duplicate collection path
Planned maintenance
Unauthorized logging impairment
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The expected event and source are identified.
* The local event is confirmed or its absence is explained.
* Agent and forwarder states are known.
* Collector logs are reviewed.
* Receiver reachability is validated.
* Wazuh manager and indexer health are known.
* Splunk receiver and index health are known.
* Host, source, sourcetype, and index routing are validated.
* Timestamp accuracy and ingestion delay are reviewed.
* Duplicate events are assessed.
* Data loss is confirmed or reasonably excluded.
* Required corrective actions are completed.
* A final validation marker reaches both platforms.
* Monitoring confirms stable recovery.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The telemetry ingestion issue was investigated across the source, collection, transport, receiver, indexing, parsing, and search layers. Local event generation, Wazuh agent health, Splunk forwarder health, network connectivity, receiver status, index routing, timestamps, ingestion delay, and duplicate handling were reviewed. The issue was classified as <CLASSIFICATION>. Required corrective and recovery actions were completed, a final validation marker was received by the expected monitoring platforms, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Operational IP addresses
* Internal domain names
* Real usernames
* Sensitive hostnames
* Wazuh agent IDs
* Wazuh manager addresses
* Splunk receiver addresses
* Receiver ports where sensitive
* Enrollment information
* Authentication tokens
* Certificates
* Internal URLs
* Complete configuration files
* Raw Security logs
* Internal index names where sensitive
* Evidence-storage paths
* VMware identifiers

Use placeholders such as:

```
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<COLLECTOR_SERVER>
<COLLECTOR_PORT>
<WAZUH_AGENT_PORT>
<SPLUNK_RECEIVING_PORT>
<EVENT_LOG_NAME>
<EVENT_ID>
<VALIDATION_MARKER>
<INDEX_NAME>
<SOURCETYPE>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Wazuh agent active or disconnected state
* Splunk forwarder active destination
* Exact local validation marker
* Wazuh marker search
* Splunk marker search
* Ingestion-delay calculation
* Host and source mapping
* Duplicate-event result
* Receiver listener status
* Final recovery marker

Before publishing:

1. Crop unrelated information.
2. Remove operational addresses.
3. Remove real usernames.
4. Remove Wazuh agent IDs.
5. Remove internal URLs.
6. Remove tokens and certificate details.
7. Remove unrelated events.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Affected endpoint identified
* Expected source identified
* Expected event or marker identified
* Initial severity assigned

## Source Validation

* Test action confirmed
* Event confirmed locally
* Event channel enabled
* Audit policy reviewed where relevant
* Local event timestamp validated
* Log capacity reviewed

## Wazuh Agent Review

* Wazuh service state confirmed
* Agent log reviewed
* Manager configuration reviewed
* Network reachability tested
* Agent status reviewed
* Duplicate agent identity checked
* Event-channel or FIM configuration reviewed
* Agent restarted only after evidence preservation

## Splunk Forwarder Review

* Forwarder service state confirmed
* `splunkd.log` reviewed
* Active destination checked
* Inputs reviewed
* Receiver reachability tested
* Index assignment reviewed
* Forwarder restarted only after evidence preservation

## Network Review

* VMware adapters reviewed
* Source address confirmed
* Route reviewed
* DNS reviewed where relevant
* Endpoint firewall reviewed
* Receiver ports tested

## Wazuh Server Review

* Containers reviewed
* Manager logs reviewed
* Indexer logs reviewed
* Dashboard logs reviewed where relevant
* Cluster or index health reviewed
* Disk and inode usage reviewed

## Splunk Server Review

* Splunk service status reviewed
* Receiver listener reviewed
* Internal logs reviewed
* Forwarder connections reviewed
* Index volume reviewed
* Disk and inode usage reviewed
* License state reviewed where relevant

## Data Quality Review

* Event searched across all indexes
* Wazuh broad search completed
* Correct host confirmed
* Correct agent confirmed
* Correct source confirmed
* Correct sourcetype confirmed
* Correct index confirmed
* Field extraction reviewed
* Timestamp accuracy reviewed
* Ingestion delay calculated
* Duplicate events checked

## Recovery

* Root cause corrected
* Collector service healthy
* Receiver healthy
* Final marker generated
* Final marker found in Wazuh
* Final marker found in Splunk
* Correct routing validated
* Delay acceptable
* No unexpected duplicate found
* Data loss assessed

## Closure

* Timeline completed
* Evidence preserved
* Evidence hashes recorded
* Root cause documented
* Severity finalized
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Confirm Local Event

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "<EVENT_LOG_NAME>"
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<VALIDATION_MARKER>"
}
```

## Confirm Wazuh Agent

```
Get-Service WazuhSvc
```

## Confirm Splunk Forwarder

```
Get-Service SplunkForwarder
```

## Confirm Splunk Destination

```
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" `
    list forward-server
```

## Confirm Collector Port

```
Test-NetConnection `
    -ComputerName <COLLECTOR_SERVER> `
    -Port <COLLECTOR_PORT>
```

## Search All Splunk Indexes

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
```

## Measure Ingestion Delay

```
index=* earliest=-2h
"<VALIDATION_MARKER>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    index
    source
    sourcetype
```

## Immediate Escalation Indicators

```
Security logs unavailable
Domain controller telemetry missing
Multiple endpoints silent
Receiver or indexer unavailable
Confirmed data loss
Unauthorized collector shutdown
Logging or audit policy disabled
Telemetry tampering suspected
Timeline invalid due to clock errors
Monitoring blind spot during active incident
Recovery failure
```

---

# Summary

This runbook provides a repeatable process for investigating telemetry ingestion and data-quality failures.

A complete investigation should determine:

* Whether the expected event was generated
* Whether it exists locally
* Whether the collection agent or forwarder is healthy
* Whether the receiver is reachable
* Whether the receiver accepted the data
* Whether the event was indexed
* Whether it can be found through search
* Whether host, source, sourcetype, index, and agent identity are correct
* Whether timestamps and ingestion delay are accurate
* Whether events were duplicated
* Whether data loss occurred
* Whether collection recovered correctly
* Whether the monitoring environment returned to a healthy state

The investigation is complete only after the failed pipeline layer is identified, required corrective action is completed, a final validation marker reaches the expected monitoring platforms, evidence is preserved, and detection improvements are documented.

