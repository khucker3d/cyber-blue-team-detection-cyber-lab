# Log Ingestion and Validation

## Overview

Log ingestion connects the systems inside the Acer Blue Team CyberLab to its security-monitoring platforms.

The purpose of ingestion is not merely to move logs from one system to another. A reliable logging pipeline must confirm that:

* The source generated the expected event.
* The collection agent received the event.
* The event reached the monitoring server.
* The platform parsed the event correctly.
* The timestamp is accurate.
* The source system is identified correctly.
* Important fields are searchable.
* Expected detections or alerts are generated.
* The analyst can reconstruct what happened.
* The validation process can be repeated.

This document describes the sanitized workflow used to validate telemetry from Windows and Linux systems through Wazuh and Splunk.

All IP addresses, hostnames, domain names, usernames, account identifiers, tokens, ports, and file paths shown in this public document are placeholders.

---

## Project Goals

The log-ingestion process supports the following goals:

* Collect endpoint security events
* Validate SIEM connectivity
* Verify Windows event forwarding
* Confirm Wazuh agent health
* Confirm Splunk forwarder health
* Test timestamp accuracy
* Validate source metadata
* Identify parsing problems
* Compare raw events with SIEM records
* Develop repeatable detection exercises
* Document telemetry gaps
* Preserve evidence for investigation

---

## Public Example Environment

The public documentation uses example values such as:

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Kali testing system: KALI-TEST
Internal domain: cyberlab.example
Documentation subnet: 192.0.2.0/24
```

The `192.0.2.0/24` range is reserved for documentation.

These values do not represent the operational CyberLab.

---

## High-Level Ingestion Architecture

```
                         Event Sources
                               |
          _____________________|_____________________
         |                     |                     |
       DC01               WIN11TARGET          Linux Systems
 Active Directory        Windows Endpoint      Server Logs
         |                     |                     |
         |                     |                     |
         |               Wazuh Agent                |
         |               Splunk Forwarder           |
         |                     |                     |
         |_____________________|_____________________|
                               |
                 Security Monitoring Platforms
                               |
                 ______________________________
                |                              |
              Wazuh                         Splunk
        Alerts and agent data       Searchable indexed events
                |                              |
                |______________  ______________|
                               \/
                      Student Investigation
```

---

## Ingestion Pipeline

A complete logging pipeline includes several stages:

```
Event Generation
      |
      v
Local Log Source
      |
      v
Collection Agent or Input
      |
      v
Network Transport
      |
      v
SIEM Receiver
      |
      v
Parsing and Normalization
      |
      v
Indexing or Alert Evaluation
      |
      v
Search, Detection, and Investigation
```

Each stage should be validated independently.

---

## Source Systems

The CyberLab may generate logs from:

* Windows Server domain controller
* Windows 11 endpoint
* Wazuh server
* Splunk server
* Kali Linux
* Windows Defender
* Windows Firewall
* PowerShell
* Active Directory
* DNS
* Group Policy
* Sysmon when installed
* Custom security scripts
* File integrity monitoring
* Packet-capture or network-monitoring tools

Not every log source must be collected immediately.

Collection should be introduced according to the lab’s detection objectives.

---

## Core Validation Principle

A dashboard result alone does not prove that the complete pipeline works correctly.

Validation should compare:

1. The action performed.
2. The event recorded on the source.
3. The collection-agent status.
4. The network connection.
5. The event stored by the SIEM.
6. The fields extracted by the SIEM.
7. The alert or detection result.
8. The investigation conclusion.

This process is referred to as end-to-end validation.

---

## Known-Test-Event Method

A known test event is a harmless action performed intentionally so that its telemetry can be traced.

Suitable examples include:

* Successful sign-in
* Single failed sign-in
* Test account lockout
* User account creation
* Security group membership change
* Harmless PowerShell command
* Process creation
* File creation
* File modification
* File deletion
* Windows Defender test detection
* Firewall block
* Controlled network connection
* Limited authorized port scan

The action, time, source, expected logs, and cleanup should be documented before testing.

---

## Validation Record Template

```
Test name:

Objective:

Source system:

Target system:

Account:

Start time:

Stop time:

Action performed:

Expected local event:

Expected Wazuh result:

Expected Splunk result:

Observed local result:

Observed Wazuh result:

Observed Splunk result:

Timestamp difference:

Parsing or field issues:

Cleanup:

Conclusion:
```

Use neutral placeholder account names in public documentation.

---

## Time Synchronization

Accurate time is required to correlate events across systems.

The following systems should use aligned clocks:

* Acer host
* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* KALI-TEST

Time differences can cause:

* Events appearing outside the selected dashboard range
* Incorrect investigation timelines
* Kerberos authentication failures
* Misleading alert order
* Apparent ingestion delays
* Duplicate or out-of-order events
* Certificate errors
* Failed event correlation

---

## Windows Time Validation

On a Windows system:

```
Get-Date
```

Review Windows Time:

```
w32tm /query /status
```

Review the configured source:

```
w32tm /query /source
```

Domain-joined endpoints should normally synchronize through the domain hierarchy.

---

## Linux Time Validation

```
date
```

```
timedatectl
```

Review synchronization:

```
timedatectl timesync-status
```

The exact command may differ when another time service is used.

---

## Record the Timezone

When documenting an exercise, record the timezone.

Example:

```
Exercise start: YYYY-MM-DD HH:MM:SS TZ
Exercise stop: YYYY-MM-DD HH:MM:SS TZ
```

Splunk commonly displays time according to user or server preferences.

Wazuh dashboard time display may also differ from the source event format.

Always verify whether the displayed time is:

* UTC
* Local server time
* Browser-local time
* Source-system time

---

## Event Time and Ingestion Time

Two timestamps are especially important:

### Event Time

The time the source system says the event occurred.

### Ingestion Time

The time the monitoring platform received or indexed the event.

A delay between these values may indicate:

* Agent outage
* Forwarder backlog
* Network interruption
* Parsing issue
* Incorrect source clock
* SIEM resource pressure
* Delayed service startup

---

## Splunk Ingestion Delay

Splunk commonly provides:

* `_time` for event time
* `_indextime` for index time

Example search:

```spl
index=<INDEX_NAME>
| eval ingestion_delay_seconds=_indextime-_time
| table _time _indextime ingestion_delay_seconds host source sourcetype
| sort - _time
```

Summary by host:

```spl
index=<INDEX_NAME>
| eval ingestion_delay_seconds=_indextime-_time
| stats
    count
    avg(ingestion_delay_seconds) as average_delay
    max(ingestion_delay_seconds) as maximum_delay
    by host
```

Large or negative values should be investigated.

---

## Wazuh Timestamp Validation

When reviewing a Wazuh alert, compare:

* Source event timestamp
* Agent timestamp
* Manager receipt time
* Dashboard display time
* Test exercise time

If the alert appears outside the expected time range, check:

* Endpoint clock
* Manager clock
* Browser timezone
* Dashboard time selector
* Agent backlog
* Snapshot restoration history

---

## Windows Event Sources

Important Windows sources include:

```
Windows Logs
├── Security
├── System
├── Application
├── Setup
└── Forwarded Events
```

Additional sources include:

```
Applications and Services Logs
└── Microsoft
    └── Windows
        ├── PowerShell
        ├── Windows Defender
        ├── GroupPolicy
        ├── Sysmon
        ├── DNS-Server
        └── Windows Firewall
```

The exact available channels depend on installed roles and software.

---

## Confirm a Windows Event Locally

Review recent Security events:

```
Get-WinEvent `
    -LogName Security `
    -MaxEvents 20
```

Administrator privileges may be required to read the Security log.

Filter by time:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName  = "Security"
        StartTime = (Get-Date).AddMinutes(-15)
    }
```

Filter by event identifier:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id      = <EVENT_ID>
        StartTime = (Get-Date).AddMinutes(-15)
    }
```

---

## Event Viewer Validation

Event Viewer can be used to verify:

* Event identifier
* Timestamp
* Computer name
* Account
* Source address
* Logon type
* Process
* Command line
* Failure reason
* Group change
* Object path

The event should be confirmed locally before concluding that a SIEM ingestion problem exists.

If no local event was generated, the problem may be:

* Audit policy
* Incorrect test procedure
* Wrong event channel
* Unsupported telemetry
* Missing Group Policy
* Log disabled
* Event overwritten

---

## Audit Policy Validation

Review effective Windows auditing:

```
auditpol /get /category:*
```

Review a specific subcategory:

```
auditpol /get /subcategory:"Process Creation"
```

Important categories include:

* Account Logon
* Account Management
* Detailed Tracking
* Logon and Logoff
* Object Access
* Policy Change
* Privilege Use
* System

The required category must be enabled before the related event can be collected.

---

## Group Policy Validation

Review applied policy:

```
gpresult /r
```

Create an HTML report:

```
gpresult /h "<REPORT_PATH>"
```

Force an update:

```
gpupdate /force
```

Confirm that the expected audit and PowerShell policies apply to the correct organizational unit.

---

## Wazuh Agent Status

On the Windows endpoint:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm:

* The service exists.
* The startup type is appropriate.
* The service is running.

A running service does not prove that the agent is connected to the manager.

---

## Wazuh Connectivity Validation

Test the required internal service port:

```
Test-NetConnection <WAZUH_SERVER> -Port <WAZUH_AGENT_PORT>
```

Review:

* `TcpTestSucceeded`
* Remote address
* Source address
* Interface used

The source should use the host-only network rather than an unintended NAT interface.

---

## Wazuh Dashboard Agent Validation

In the Wazuh dashboard, confirm:

* Agent name
* Agent address
* Operating system
* Status
* Last keepalive
* Manager
* Group
* Version

Expected status:

```
Active
```

A recently active agent may briefly appear disconnected after:

* Endpoint shutdown
* Manager restart
* Network interruption
* Snapshot restoration
* Agent-service restart

---

## Wazuh Event Validation

After generating a known event:

1. Confirm the event locally.
2. Record the source timestamp.
3. Confirm the Wazuh agent remains active.
4. Search the dashboard using a narrow time range.
5. Filter by agent.
6. Review the rule description.
7. Review the raw event.
8. Compare usernames and hostnames.
9. Review rule level and framework mappings.
10. Record whether the expected alert was created.

---

## Raw Event Versus Alert

Wazuh may:

* Collect a raw event
* Decode the event
* Match it against a rule
* Generate an alert
* Assign severity
* Apply MITRE or compliance metadata

A collected event may not always generate a visible security alert.

Possible reasons include:

* No rule matches the event.
* The rule level is below the dashboard filter.
* The event is stored in another location.
* The event source is not configured.
* The decoder failed.
* The time range excludes the event.

The absence of an alert should not immediately be interpreted as an ingestion failure.

---

## Wazuh File Integrity Validation

A dedicated test directory should be monitored.

Example:

```
C:\CyberLab-Test\Files
```

Create a test file:

```
New-Item `
    -Path "C:\CyberLab-Test\Files\fim-validation.txt" `
    -ItemType File `
    -Force
```

Add content:

```
Set-Content `
    -Path "C:\CyberLab-Test\Files\fim-validation.txt" `
    -Value "Authorized file integrity validation."
```

Modify the file:

```
Add-Content `
    -Path "C:\CyberLab-Test\Files\fim-validation.txt" `
    -Value "Controlled modification."
```

Delete after validation:

```
Remove-Item `
    -Path "C:\CyberLab-Test\Files\fim-validation.txt"
```

---

## File Integrity Fields

Review:

* Agent
* File path
* Change type
* Previous hash
* Current hash
* File size
* User where available
* Modification time
* Rule
* Alert level

File creation, modification, and deletion may appear as separate records.

---

## Splunk Universal Forwarder Status

On the Windows endpoint:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm that the Universal Forwarder service is running.

The exact service name depends on the installation.

---

## Splunk Forwarder Connectivity

Test the receiver:

```
Test-NetConnection `
    <SPLUNK_SERVER> `
    -Port <SPLUNK_RECEIVER_PORT>
```

A successful connection confirms that the endpoint can reach the listening port.

It does not confirm that:

* The forwarder configuration is correct.
* The selected logs are monitored.
* The target index exists.
* Events are parsed correctly.

---

## Splunk Receiver Validation

On the Splunk server:

```
sudo ss -tulpn |
    grep <SPLUNK_RECEIVER_PORT>
```

Confirm that the receiver is listening on the intended interface.

The Linux firewall should limit access to the internal CyberLab network.

---

## Splunk Forwarder Configuration

Important forwarder configuration may include:

* `inputs.conf`
* `outputs.conf`
* `server.conf`

A sanitized output template:

```ini
[tcpout]
defaultGroup = cyberlab_indexers

[tcpout:cyberlab_indexers]
server = <SPLUNK_SERVER>:<SPLUNK_RECEIVER_PORT>
```

A sanitized Windows event input template:

```ini
[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Application]
disabled = 0
index = windows
```

Exact syntax should be validated against the installed forwarder version.

---

## Splunk Forwarder Logs

The Universal Forwarder maintains internal logs within its installation directory.

Important information may include:

* Connection attempts
* Receiver status
* Input discovery
* File monitoring
* Parsing warnings
* Authentication errors
* Configuration errors

Operational forwarder logs may contain:

* Internal IP addresses
* Hostnames
* File paths
* Server identifiers
* Usernames

Sanitize excerpts before publication.

---

## Splunk Data Validation

Begin with a narrow search:

```spl
index=windows host=WIN11TARGET earliest=-15m
```

Review metadata:

```spl
index=windows host=WIN11TARGET earliest=-15m
| table _time host source sourcetype EventCode user _raw
| sort - _time
```

Count by source:

```spl
index=windows host=WIN11TARGET earliest=-15m
| stats count by source sourcetype
| sort - count
```

Count by event identifier:

```spl
index=windows host=WIN11TARGET earliest=-15m
| stats count by EventCode
| sort - count
```

---

## Validate Splunk Metadata

Every event should be reviewed for:

* `index`
* `host`
* `source`
* `sourcetype`
* `_time`
* `_indextime`

Incorrect metadata can make useful data difficult to find.

Examples of problems include:

* Endpoint data assigned to the Splunk server host
* Generic or incorrect sourcetype
* Data sent to the wrong index
* Source path missing
* Timestamp parsed from the wrong field
* Duplicate events

---

## Host Field Validation

Search:

```spl
index=windows
| stats count by host
```

Expected public host examples:

```
DC01
WIN11TARGET
```

If the host field is incorrect, review:

* Forwarder hostname
* Input configuration
* Host override
* Clone or snapshot history
* Duplicate forwarder identity

---

## Sourcetype Validation

Search:

```spl
index=windows
| stats count by sourcetype
```

The sourcetype should accurately describe the event format.

An incorrect sourcetype can cause:

* Missing fields
* Incorrect timestamps
* Failed searches
* Detection mismatches
* CIM mapping problems
* Poor dashboard behavior

---

## Source Validation

Search:

```spl
index=windows
| stats count by source
```

The source should identify the event channel or monitored file.

Examples may include:

```
WinEventLog:Security
WinEventLog:System
WinEventLog:Application
```

The exact values depend on the collection method.

---

## Index Validation

List event counts by index:

```spl
index=* earliest=-15m
| stats count by index
| sort - count
```

Use this temporarily for troubleshooting.

Normal searches should specify the expected index.

---

## Data Quality Checks

A useful ingestion pipeline should be evaluated for:

* Completeness
* Accuracy
* Timeliness
* Consistency
* Uniqueness
* Searchability
* Context

### Completeness

Are the expected events present?

### Accuracy

Do the fields match the source event?

### Timeliness

Did the event arrive within an acceptable delay?

### Consistency

Are equivalent events parsed the same way?

### Uniqueness

Are events duplicated?

### Searchability

Can the analyst find the event reliably?

### Context

Are the host, user, process, source, and network fields available?

---

## Data Quality Record

```
Data source:

Expected events:

Received events:

Missing events:

Duplicate events:

Average delay:

Maximum delay:

Timestamp issues:

Host-field issues:

Sourcetype issues:

Missing fields:

Parsing problems:

Recommended changes:
```

---

## Windows Authentication Validation

Authentication telemetry may exist on:

* The endpoint
* The domain controller
* Wazuh
* Splunk

A complete investigation may require comparing all four.

---

## Successful Logon Test

1. Confirm DC01 is running.
2. Confirm the endpoint uses DC01 for DNS.
3. Sign out of the endpoint.
4. Sign in with a designated standard domain account.
5. Record the exact time.
6. Confirm the event locally.
7. Search Wazuh.
8. Search Splunk.
9. Compare the account and logon type.
10. Sign out and document the result.

---

## Failed Logon Test

1. Use a dedicated test account.
2. Confirm the account is not an administrator.
3. Enter an incorrect password once.
4. Record the time.
5. Sign in successfully afterward if appropriate.
6. Review the endpoint event.
7. Review the domain controller event.
8. Review Wazuh.
9. Review Splunk.
10. Document the failure reason and source.

Avoid locking out critical accounts.

---

## Account Lockout Test

1. Review the domain lockout policy.
2. Use a disposable test account.
3. Confirm a separate administrator account is available.
4. Record the account’s initial state.
5. Perform only the required number of invalid attempts.
6. Confirm the lockout.
7. Review DC01 events.
8. Search Wazuh and Splunk.
9. Investigate the source before unlocking.
10. Restore access and document cleanup.

---

## Example Splunk Authentication Search

Generic failed-logon search:

```spl
index=windows EventCode=<FAILED_LOGON_EVENT>
| table _time host user src_ip LogonType FailureReason
| sort - _time
```

Generic successful-logon search:

```spl
index=windows EventCode=<SUCCESSFUL_LOGON_EVENT>
| table _time host user src_ip LogonType
| sort - _time
```

Field names depend on the event sourcetype and installed add-ons.

---

## Account Management Validation

Useful identity events include:

* User created
* User enabled
* User disabled
* Password reset
* User deleted
* Group membership changed
* Computer account created
* Administrative group modified

These actions should use temporary lab accounts.

---

## Test User Creation

On DC01, create a temporary nonprivileged test account through the documented identity-management procedure.

Record:

* Administrator account
* Target account
* Time
* Organizational unit
* Expected local event
* Expected Wazuh alert
* Expected Splunk record

Delete or disable the account after the exercise.

---

## Splunk User-Creation Search

```spl
index=windows EventCode=<USER_CREATED_EVENT>
| table _time host SubjectUserName TargetUserName
| sort - _time
```

Review:

* Actor
* Target
* Source system
* Time
* Related group changes
* Later authentication

---

## Group Membership Validation

Add a temporary test account to a noncritical lab group.

Then review:

* Account performing the change
* Target group
* Added member
* Domain controller
* Timestamp
* Follow-up actions

Remove the account from the group during cleanup.

---

## PowerShell Telemetry Validation

PowerShell telemetry may come from:

* Security process-creation events
* PowerShell Operational log
* Script block logging
* Module logging
* Sysmon
* Wazuh
* Splunk

A harmless command should be used.

---

## Harmless PowerShell Test

```
Get-Service |
    Sort-Object Status |
    Select-Object -First 10
```

Record:

* User
* Parent process
* Process name
* Command line
* Start time
* Security event
* PowerShell event
* Wazuh result
* Splunk result

---

## PowerShell Local Validation

Review PowerShell Operational events:

```
Get-WinEvent `
    -LogName "Microsoft-Windows-PowerShell/Operational" `
    -MaxEvents 20
```

Review recent process-creation events:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = <PROCESS_CREATION_EVENT>
        StartTime = (Get-Date).AddMinutes(-15)
    }
```

Command-line data appears only when the relevant audit setting is enabled.

---

## Splunk PowerShell Search

```spl
index=windows source=*PowerShell*
| table _time host user EventCode Message
| sort - _time
```

With process telemetry:

```spl
index=windows process_name="powershell.exe"
| table _time host user parent_process process_name process_command_line
| sort - _time
```

The available fields depend on parsing and Sysmon deployment.

---

## Process Creation Validation

A process-creation event should identify:

* Process name
* Process identifier
* Parent process
* User
* Command line where enabled
* Integrity level where available
* Hashes when supplied by Sysmon
* Host
* Time

Use benign built-in tools for validation.

Example:

```
Start-Process notepad.exe
```

Close the application after the event is confirmed.

---

## Windows Defender Validation

A Defender validation should use an officially documented harmless antivirus test mechanism.

The workflow should confirm:

* Defender is enabled.
* The test artifact is created inside the lab.
* Defender detects it.
* The local Defender event is recorded.
* Wazuh receives the event.
* Splunk receives the event if configured.
* The artifact is quarantined or removed.
* Defender returns to a healthy state.

Do not upload the test artifact to GitHub.

---

## Windows Firewall Validation

A controlled connection to a blocked port may generate firewall telemetry when logging is enabled.

Before testing:

* Confirm blocked-connection logging is enabled.
* Confirm the destination is an authorized lab system.
* Confirm the test will not affect a needed service.
* Record the time.

Review firewall settings:

```
Get-NetFirewallProfile |
    Select-Object Name, LogAllowed, LogBlocked, LogFileName
```

---

## Network Activity Validation

Kali may generate controlled traffic toward the Windows endpoint.

Examples include:

* ICMP request
* DNS lookup
* TCP connection test
* HTTP request
* Limited port scan

The objective is to determine which systems observe the traffic.

---

## Single-Port Connection Test

From Kali:

```
nc -vz <AUTHORIZED_TARGET> <APPROVED_PORT>
```

Review:

* Kali output
* Target firewall log
* Windows event logs
* Wazuh
* Splunk
* Packet capture

Not every connection attempt creates a native Windows security event.

This may reveal a telemetry gap that requires:

* Firewall logging
* Sysmon
* Network IDS
* Packet capture
* Additional endpoint telemetry

---

## Limited Scan Validation

```
nmap -sT \
    -p <APPROVED_PORT_LIST> \
    <AUTHORIZED_TARGET>
```

Keep the scope narrow.

Record:

* Source
* Target
* Port list
* Start time
* End time
* Expected logs
* Actual logs
* Detection gaps

---

## Packet Capture Validation

A packet capture can confirm that traffic occurred even when no SIEM event exists.

On Kali or another authorized sensor:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <AUTHORIZED_TARGET> \
    -w <CAPTURE_FILE>.pcap
```

Stop the capture after the test.

Treat the capture as sensitive evidence.

---

## Hash the Capture

```
sha256sum <CAPTURE_FILE>.pcap
```

Record:

```
File:
Capture interface:
Source:
Target:
Collection time:
Exercise:
SHA-256:
```

---

## Wazuh and Splunk Comparison

Wazuh and Splunk may present the same event differently.

| Capability                | Wazuh                                          | Splunk                                              |
| ------------------------- | ---------------------------------------------- | --------------------------------------------------- |
| Endpoint agent status     | Built-in agent visibility                      | Forwarder status through internal logs and searches |
| Rule-based alerts         | Prebuilt and custom Wazuh rules                | Saved searches and alerts                           |
| Raw-event searching       | Available through indexed data and event views | Core platform capability                            |
| File integrity monitoring | Built-in agent capability                      | Requires appropriate data source                    |
| MITRE mapping             | Included with many rules                       | Added through detections, apps, or analyst mapping  |
| Flexible statistics       | Available                                      | Strong SPL-based analysis                           |
| Dashboard creation        | Available                                      | Strong customizable dashboards                      |
| Data normalization        | Wazuh decoders and fields                      | Sourcetypes, field extraction, and CIM              |

The platforms should be treated as complementary learning tools rather than assumed to produce identical results.

---

## Cross-Platform Correlation

For one known test event, record:

```
Source event timestamp:

Wazuh timestamp:

Splunk timestamp:

Wazuh agent:

Splunk host:

Wazuh rule:

Splunk event code:

User:

Source address:

Process:

Command line:

File path:

Investigation conclusion:
```

This comparison helps identify parsing and timing differences.

---

## Expected Telemetry Matrix

| Exercise                |        Local Windows |  DC01 |                       Wazuh |                      Splunk | Packet Capture |
| ----------------------- | -------------------: | ----: | --------------------------: | --------------------------: | -------------: |
| Successful domain logon |                  Yes | Often |                    Expected |                    Expected |       Optional |
| Failed domain logon     |                  Yes | Often |                    Expected |                    Expected |       Optional |
| Account lockout         |             Possible |   Yes |                    Expected |                    Expected |       Optional |
| User creation           |              On DC01 |   Yes |                    Expected |                    Expected |             No |
| Group membership change |              On DC01 |   Yes |                    Expected |                    Expected |             No |
| Harmless PowerShell     |                  Yes |    No |           Depends on policy |       Expected if collected |             No |
| File modification       | File-system evidence |    No |           Expected with FIM |           Only if collected |             No |
| Limited port scan       | Depends on telemetry |    No | May require added telemetry | May require added telemetry |            Yes |
| Defender test           |                  Yes |    No |                    Expected |       Expected if collected |             No |

“Expected” means the pipeline is intended to collect it, not that every default rule will create an alert.

---

## Missing Event Investigation

When an expected event is missing, determine where the pipeline failed.

### Stage 1: Action

Was the intended action actually performed?

### Stage 2: Source

Did the source log record the event?

### Stage 3: Agent or Input

Was the log source configured for collection?

### Stage 4: Service

Was the agent or forwarder running?

### Stage 5: Network

Could the source reach the receiver?

### Stage 6: Receiver

Was the SIEM listening?

### Stage 7: Parsing

Was the event assigned the correct format?

### Stage 8: Storage

Was the event indexed or retained?

### Stage 9: Search

Did the query use the correct time range and fields?

### Stage 10: Alerting

Did a rule or detection exist?

---

## Troubleshooting Decision Flow

```
Expected event missing
        |
        v
Did the event occur locally?
        |
     No | Yes
        | 
        |        Is the collection service running?
        |                  |
Fix audit policy       No | Yes
or test method            |
                         Start or repair service
                                  |
                                  v
                         Is the receiver reachable?
                                  |
                               No | Yes
                                  |
                         Fix network/firewall
                                          |
                                          v
                               Is the event indexed?
                                          |
                                       No | Yes
                                          |
                                Fix input/parsing
                                                  |
                                                  v
                                      Fix search or detection
```

---

## Troubleshooting: Local Event Is Missing

Check:

* Correct event channel
* Audit policy
* Group Policy
* Event-log service
* Test account and action
* Required administrative privileges
* Operating system support
* Event log retention
* Event overwritten by high volume

Commands:

```
auditpol /get /category:*
```

```
gpresult /r
```

```
Get-Service EventLog
```

---

## Troubleshooting: Wazuh Agent Is Inactive

Check:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Test connectivity:

```
Test-NetConnection <WAZUH_SERVER> -Port <WAZUH_AGENT_PORT>
```

Possible causes include:

* Agent service stopped
* Wazuh manager unavailable
* Wrong manager address
* Host-only adapter disconnected
* Firewall rule
* Duplicate agent
* Restored snapshot
* Agent-key mismatch
* Time problem

---

## Troubleshooting: Wazuh Event Is Not Alerted

Check:

* The event source is collected.
* The event appears locally.
* The agent is active.
* A decoder recognizes the event.
* A rule exists.
* The rule level is visible.
* The time filter is correct.
* The selected agent is correct.

The event may require a custom rule if the default ruleset does not alert on it.

---

## Troubleshooting: Splunk Forwarder Is Not Running

Review:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Restart when appropriate:

```
Restart-Service <SPLUNK_FORWARDER_SERVICE>
```

Administrator privileges are required.

Review the forwarder logs if the service stops again.

---

## Troubleshooting: Splunk Receiver Is Unreachable

From Windows:

```
Test-NetConnection `
    <SPLUNK_SERVER> `
    -Port <SPLUNK_RECEIVER_PORT>
```

On the server:

```
sudo ss -tulpn |
    grep <SPLUNK_RECEIVER_PORT>
```

Check:

* Splunk service
* Receiver configuration
* Ubuntu firewall
* Endpoint firewall
* VMware adapter
* Wrong address
* NAT versus host-only path
* Port conflict

---

## Troubleshooting: Splunk Search Returns Nothing

Begin with:

```spl
index=<EXPECTED_INDEX> earliest=-24h
```

Then inspect metadata:

```spl
index=<EXPECTED_INDEX> earliest=-24h
| stats count by host source sourcetype
```

Check:

* Time range
* Index
* Host value
* Source
* Sourcetype
* Event identifier field
* User permissions
* Forwarder status
* Receiver status

Add filters one at a time.

---

## Troubleshooting: Wrong Splunk Hostname

Possible causes include:

* Cloned VM identity
* Incorrect forwarder hostname
* Host override
* Stale configuration
* Snapshot restoration
* Data being collected locally on the server

Review the forwarder configuration and internal Splunk logs.

Do not modify historical events to conceal the issue.

Correct the configuration for future data.

---

## Troubleshooting: Wrong Timestamp

Possible causes include:

* Incorrect source clock
* Wrong timezone
* Incorrect sourcetype
* Timestamp extracted from message content incorrectly
* Missing year
* Snapshot rollback
* Browser timezone
* Delayed forwarding

Compare `_time` and `_indextime` in Splunk.

Compare source and dashboard timestamps in Wazuh.

---

## Troubleshooting: Duplicate Events

Possible causes include:

* Same channel collected twice
* Local and forwarded input overlap
* Multiple forwarders
* Duplicate agent configuration
* Replayed files
* Snapshot restoration
* Re-uploaded dataset
* Multiple Splunk inputs targeting the same source

Splunk duplicate review:

```spl
index=<INDEX_NAME>
| stats count by _time host source EventCode _raw
| where count > 1
```

Use caution because legitimately repeated events may have identical fields.

---

## Troubleshooting: Missing Fields

Possible causes include:

* Incorrect sourcetype
* Missing add-on
* Parsing failure
* Different event format
* Field exists only inside raw XML
* Wazuh decoder limitation
* Data collected from a different channel

Review the raw event before creating manual extractions.

Do not create a field extraction based on one sample without testing additional events.

---

## Troubleshooting: High Ingestion Delay

Check:

* Endpoint CPU and memory
* Agent or forwarder service
* Network path
* SIEM resource use
* Disk space
* Splunk queues
* Wazuh indexer health
* Large backlog after shutdown
* Source time

Record whether the delay is:

* Continuous
* Intermittent
* Host-specific
* Source-specific
* Related to server restarts

---

## Troubleshooting: Event Volume Is Too High

Possible causes include:

* Broad PowerShell logging
* Sysmon configuration
* Process creation auditing
* Repeated failed authentication
* File integrity monitoring of noisy directories
* Duplicate collection
* Firewall allowed-connection logging
* Large scan range
* Debug logging

Reduce volume by tuning the source rather than deleting evidence indiscriminately.

---

## Log Retention

Retention should be planned according to:

* Disk capacity
* Event volume
* Exercise duration
* Investigation requirements
* Snapshot size
* Privacy
* Backup strategy

The lab does not need enterprise-scale retention.

Important evidence should be exported before data expires or the environment is reset.

---

## Windows Event Log Retention

Review log size:

```
Get-WinEvent -ListLog Security |
    Select-Object LogName, MaximumSizeInBytes, RecordCount, LogMode
```

Review other important channels individually.

Log-size changes should be applied through Group Policy where practical.

---

## Splunk Retention

Splunk retention is commonly configured per index.

Before changing retention:

1. Record current index settings.
2. Export important events.
3. Back up configuration.
4. Confirm available disk space.
5. Make one controlled change.
6. Restart only if required.
7. Confirm searches still work.
8. Document the change.

Do not delete index directories manually.

---

## Wazuh Retention

Wazuh retention depends on the deployment’s indexer and index-management configuration.

Before changing retention:

* Back up configuration.
* Confirm index health.
* Review disk space.
* Record existing policy.
* Understand which indexes are affected.
* Validate dashboard searches afterward.

---

## Evidence Export

Important test evidence may include:

* Windows `.evtx` files
* Splunk CSV exports
* Splunk screenshots
* Wazuh alert exports
* Packet captures
* PowerShell output
* Event XML
* Investigation notes
* Hash records

Evidence should be exported before snapshot rollback.

---

## Export a Windows Event Log

```
wevtutil epl Security "<EVIDENCE_PATH>\Security.evtx"
```

Administrator privileges may be required.

Raw event logs may contain sensitive information and should not be published without review.

---

## Export Splunk Results

Search results may be exported through Splunk Web as:

* CSV
* JSON
* XML
* Raw events

Review exports for:

* Internal addresses
* Usernames
* Domain names
* Session data
* Command lines
* File paths
* Personal information

---

## Evidence Hashing

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

Record:

```
File:
Source system:
Collection date:
Exercise:
SHA-256:
Student analyst:
```

---

## Baseline Logging Validation

Before advanced detection exercises, validate the baseline.

### Windows Endpoint

* Security log readable
* System log readable
* PowerShell Operational enabled
* Defender Operational enabled
* Audit policy applied
* Wazuh agent active
* Splunk forwarder active

### Domain Controller

* Security log active
* Directory Service log available
* DNS logs available where configured
* Account-management events generated
* Authentication events generated
* Time synchronized

### Wazuh

* Manager running
* Indexer healthy
* Dashboard accessible
* Endpoint active
* Recent Windows events visible
* File integrity test works

### Splunk

* Splunk service running
* Web interface accessible
* Receiver listening
* Endpoint forwarder connected
* Expected indexes exist
* Recent endpoint events searchable

---

## Startup Validation

After starting the CyberLab:

1. Start DC01.
2. Confirm Active Directory and DNS.
3. Start WIN11TARGET.
4. Confirm domain connectivity.
5. Start WAZUH-SERVER.
6. Confirm the Wazuh stack.
7. Confirm the Wazuh agent reconnects.
8. Start SPLUNK-SERVER.
9. Start Splunk.
10. Confirm the forwarder reconnects.
11. Generate a harmless known event.
12. Confirm the event in both platforms.

---

## Shutdown Validation

Before shutting down:

1. Stop active testing.
2. Record the exercise end time.
3. Confirm expected events reached the SIEMs.
4. Export important evidence.
5. Stop packet captures.
6. Sign out of test accounts.
7. Shut down Kali.
8. Shut down the Windows endpoint.
9. Stop Splunk cleanly.
10. Stop Wazuh cleanly.
11. Shut down monitoring servers.
12. Shut down DC01 last.
13. Create stable snapshots when appropriate.

---

## Snapshot Considerations

Snapshots can affect logging through:

* Time rollback
* Duplicate events
* Lost events
* Secure-channel failure
* Agent identity mismatch
* Forwarder backlog
* Restored credentials
* Stale SIEM state
* Reappearing test accounts

After restoration, always generate a new known event and confirm the complete pipeline.

---

## Post-Restore Validation

1. Confirm system time on all restored systems.
2. Confirm host-only network settings.
3. Confirm DNS.
4. Confirm the domain secure channel.
5. Confirm Wazuh agent status.
6. Confirm Splunk forwarder status.
7. Confirm server listeners.
8. Generate a harmless test event.
9. Compare local, Wazuh, and Splunk records.
10. Document any duplicate or missing data.

---

## Logging Change Management

Document changes to:

* Audit policy
* Group Policy
* Event channels
* Wazuh agent configuration
* Wazuh manager configuration
* File integrity paths
* Splunk inputs
* Splunk outputs
* Receiver ports
* Indexes
* Sourcetypes
* Field extractions
* Retention
* Firewall rules
* Time sources

Example:

```
Date:

Change:

Reason:

Data sources affected:

Previous configuration:

New configuration:

Backup or snapshot:

Validation event:

Observed result:

Rollback plan:

Outcome:
```

---

## Public Sanitization Standards

Remove or replace:

* Real internal addresses
* Real hostnames when sensitive
* Personal usernames
* Email addresses
* Internal domain names
* Security identifiers
* Agent keys
* Forwarder credentials
* Enrollment tokens
* Dashboard session identifiers
* API tokens
* Raw event exports
* Event data containing personal names
* Real file paths
* Home-directory usernames
* MAC addresses
* VMware identifiers
* Exact active receiver configuration
* Packet captures
* Authentication details
* Password-related data

Use placeholders such as:

```
<WINDOWS_ENDPOINT>
<DOMAIN_CONTROLLER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<KALI_SYSTEM>
<LAB_DOMAIN>
<INDEX_NAME>
<EVENT_ID>
<WAZUH_AGENT_PORT>
<SPLUNK_RECEIVER_PORT>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing logging screenshots:

1. Use a neutral test account.
2. Select a narrow time range.
3. Close unrelated browser tabs.
4. Hide bookmarks.
5. Clear notifications.
6. Remove internal addresses.
7. Remove domain names.
8. Remove usernames where required.
9. Remove security identifiers.
10. Remove session and token data.
11. Review raw event fields.
12. Crop to the relevant evidence.
13. Permanently redact sensitive values.
14. Reopen the final image to verify it.

---

## Suitable Public Screenshots

Useful sanitized examples include:

* Wazuh active-agent status
* Wazuh harmless authentication alert
* Wazuh file integrity event
* Splunk synthetic log search
* Splunk Windows event search
* Splunk event-count statistics
* PowerShell event validation
* Windows Event Viewer source event
* Forwarder connectivity test
* Sanitized end-to-end validation comparison

Avoid showing:

* Raw credential events
* Full operational IP addresses
* Personal account names
* License information
* Session cookies
* Enrollment commands
* Internal certificates
* Unrelated alerts

---

## Validation Checklist

### Source Event

* The action was authorized.
* The action time was recorded.
* The local event exists.
* The event identifier is known.
* The correct event channel was used.
* The event fields match the action.

### Wazuh

* Agent is active.
* Manager is reachable.
* The event source is collected.
* The event or alert appears.
* The agent identity is correct.
* The timestamp is accurate.
* The rule result is understood.

### Splunk

* Forwarder is running.
* Receiver is reachable.
* The event is indexed.
* The correct index is used.
* Host, source, and sourcetype are correct.
* Event and index times are reasonable.
* Fields are searchable.

### Correlation

* Local, Wazuh, and Splunk records represent the same action.
* Usernames and hostnames align.
* Timestamps align.
* Source and destination details align.
* Any differences are documented.
* The investigation conclusion is recorded.

### Recovery

* Important evidence was exported.
* Evidence hashes were recorded.
* A stable snapshot exists.
* Restore validation is documented.
* Sensitive evidence is excluded from GitHub.

---

## Skills Demonstrated

This work demonstrates:

* Windows event-log analysis
* Active Directory auditing
* PowerShell logging
* Wazuh agent administration
* Wazuh alert validation
* Splunk Universal Forwarder administration
* Splunk data ingestion
* SPL querying
* Timestamp analysis
* Data-quality assessment
* Log-source troubleshooting
* Detection validation
* Network connectivity testing
* Packet-capture validation
* Evidence handling
* SIEM comparison
* Change management
* Snapshot recovery validation
* Public documentation sanitization

---

## Lessons Learned

Key lessons include:

* A dashboard result does not prove that the entire logging pipeline is healthy.
* The source event should be validated before troubleshooting the SIEM.
* A running agent or forwarder may still be misconfigured.
* Network connectivity does not prove that data is being indexed.
* Correct timestamps are essential for event correlation.
* Splunk event time and index time can reveal ingestion delays.
* Wazuh may collect an event without generating a visible alert.
* Incorrect host, source, or sourcetype metadata can hide useful events.
* Domain authentication activity may appear on both the endpoint and domain controller.
* Not every network connection creates endpoint telemetry by default.
* Packet capture can confirm activity when host logs are incomplete.
* Missing alerts often reveal detection gaps rather than platform failures.
* Snapshot restoration can create duplicates, time problems, and agent inconsistencies.
* Evidence should be preserved before a lab reset.
* Public event screenshots require aggressive sanitization.

---

## Planned Improvements

Future improvements may include:

* Sysmon deployment
* Windows Event Forwarding
* Dedicated log collector
* Additional Splunk indexes
* Splunk Technology Add-on for Windows
* Common Information Model mapping
* Custom Wazuh rules
* Custom Wazuh decoders
* Sigma-rule testing
* Automated known-event generation
* Automated ingestion-delay checks
* Agent and forwarder health dashboard
* Detection coverage matrix
* Data-quality dashboard
* Event-volume baselines
* Automated evidence export
* Centralized time monitoring
* Suricata ingestion
* Zeek ingestion
* Linux endpoint telemetry
* Automated post-snapshot validation
* Reusable validation scripts

---

## Summary

Reliable log ingestion requires more than installing an agent or enabling a receiver.

The Acer Blue Team CyberLab validates logging through a repeatable process that confirms:

* The source action occurred.
* The local event exists.
* The collection service is active.
* The network path is available.
* The SIEM received the event.
* The event was parsed correctly.
* The timestamp and metadata are accurate.
* The event can be searched and investigated.
* Expected detections are understood.
* Evidence and cleanup are documented.

By validating the entire path from event generation through investigation, the CyberLab provides a dependable foundation for detection engineering, incident response, threat hunting, and security-monitoring practice.

