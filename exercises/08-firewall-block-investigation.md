# Firewall Block Investigation

## Exercise Overview

This exercise generates a controlled blocked network connection between authorized CyberLab systems and validates the resulting telemetry across:

* Windows Defender Firewall
* Windows Event Viewer
* Windows Firewall text logs
* Wazuh
* Splunk
* Optional Sysmon telemetry
* Optional packet capture
* Investigation notes and evidence

The objective is to determine:

* Which system initiated the connection
* Which destination system and port were targeted
* Which firewall rule or policy blocked the traffic
* Whether the blocked traffic was inbound or outbound
* Which process generated the connection
* Whether Windows recorded the block
* Whether Wazuh and Splunk received the event
* Whether the block was expected
* Whether the firewall remained correctly configured after testing

All activity must remain inside the authorized VMware host-only CyberLab network.

---

## Exercise ID

```
EX-08
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Windows Defender Firewall administration
* Network-connection troubleshooting
* Firewall event investigation
* Source and destination analysis
* Port and protocol identification
* Wazuh event validation
* Splunk SPL development
* Optional Sysmon analysis
* Packet capture correlation
* Evidence preservation
* Firewall-rule cleanup

---

## Authorization Boundary

This exercise must use:

* WIN11TARGET
* An authorized source system such as KALI-TEST
* The VMware host-only CyberLab network
* A temporary test firewall rule
* A noncritical test port
* Wazuh and Splunk monitoring
* A documented start and stop time

Do not use:

* External IP addresses
* Public websites
* Household devices
* The physical home network
* School or employer systems
* Production services
* Domain-controller authentication ports
* Wazuh management ports
* Splunk management ports
* Remote administration ports required for recovery
* Broad deny-all rules

---

## Preferred Exercise Design

The preferred design is:

1. Start a harmless temporary listener on WIN11TARGET.
2. Confirm the listener is reachable from KALI-TEST.
3. Create a narrow inbound Windows Firewall block rule for that port.
4. Repeat the connection attempt.
5. Confirm the connection is blocked.
6. Investigate the resulting telemetry.
7. Remove the temporary rule.
8. Confirm connectivity is restored.

This approach provides a known-good baseline before generating the blocked event.

---

## Public Example Environment

```
Source system: KALI-TEST
Destination system: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Test account: firewall.test
Test protocol: TCP
Test port: <TEST_PORT>
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Review the Windows Firewall profile state
* Review firewall audit configuration
* Enable narrow dropped-packet logging
* Create a temporary firewall rule safely
* Generate a controlled blocked connection
* Identify source and destination addresses
* Identify source and destination ports
* Determine the direction of the blocked traffic
* Identify the associated application where possible
* Locate the event in Windows logs
* Locate the event in Wazuh
* Locate the event in Splunk
* Compare firewall, Sysmon, and packet evidence
* Remove the temporary rule
* Confirm connectivity is restored

---

## Required Systems

* WIN11TARGET
* KALI-TEST
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on WIN11TARGET
* Splunk Universal Forwarder where configured

Optional:

* Sysmon on WIN11TARGET
* Wireshark or tcpdump on an authorized CyberLab system

DC01 is not required for the standard version of this exercise.

---

## Required Accounts

### Standard Test Account

Used for ordinary endpoint activity where possible.

```
firewall.test
```

### Authorized Windows Administrator

Required for:

* Firewall logging configuration
* Temporary firewall-rule creation
* Firewall-rule removal
* Protected event-log access

### Kali Standard User

Used for the connection test.

Root privileges are required only for packet capture or privileged networking functions.

---

## Required Monitoring

Before starting, confirm:

* Windows Defender Firewall is enabled
* The active firewall profile is known
* Dropped-packet logging is configured
* Filtering Platform auditing is enabled where used
* Wazuh agent is active
* Splunk is running
* Relevant Windows channels are collected
* Optional Sysmon is active where used
* System time is synchronized
* Stable snapshots exist

---

## Safety Notes

* Use one temporary rule limited to one port.
* Use an unused, noncritical test port.
* Do not block DNS, Kerberos, LDAP, SMB, RDP, Wazuh, or Splunk ports.
* Do not disable the firewall.
* Do not use a broad deny-all rule.
* Confirm local administrative access before testing.
* Record the original firewall state.
* Remove the temporary rule after evidence collection.
* Confirm the listener is stopped during cleanup.
* Stop immediately if management access is affected.

---

## Expected Event Flow

```
KALI-TEST Connection Attempt
            |
            v
WIN11TARGET Network Stack
            |
            v
Windows Defender Firewall Rule
            |
            v
Connection Blocked
            |
            v
Firewall Log / Filtering Platform Event
            |
            v
Wazuh and Splunk Ingestion
            |
            v
Student Investigation
            |
            v
Temporary Rule Removed
            |
            v
Connectivity Restored
```

---

## Relevant Windows Event IDs

Windows Filtering Platform events may include:

| Event ID | Description                                                                                  |
| -------: | -------------------------------------------------------------------------------------------- |
|     5152 | The Windows Filtering Platform blocked a packet                                              |
|     5153 | A more restrictive Windows Filtering Platform filter blocked a packet                        |
|     5155 | The Windows Filtering Platform blocked an application or service from listening              |
|     5157 | The Windows Filtering Platform blocked a connection                                          |
|     5159 | The Windows Filtering Platform blocked a bind to a local port                                |
|     4946 | A change was made to the Windows Firewall exception list; a rule was added                   |
|     4947 | A Windows Firewall rule was modified                                                         |
|     4948 | A Windows Firewall rule was deleted                                                          |
|     4950 | A Windows Firewall setting changed                                                           |
|     4956 | Windows Firewall changed the active profile                                                  |
|     5025 | The Windows Firewall service was stopped                                                     |
|     5031 | Windows Firewall blocked an application from accepting incoming connections                  |
|     2004 | Windows Firewall rule added through the firewall operational channel, depending on version   |
|     2006 | Windows Firewall rule deleted through the firewall operational channel, depending on version |

The exact events depend on:

* Windows version
* Audit policy
* Firewall profile
* Logging configuration
* Connection direction
* Rule creation method

---

## Optional Sysmon Event IDs

| Event ID | Description        |
| -------: | ------------------ |
|        1 | Process creation   |
|        3 | Network connection |
|       22 | DNS query          |

Sysmon Event ID `3` records successful network connections when enabled. It may not record a connection that is blocked before establishment.

Its absence during a blocked attempt may be expected.

---

## Important Event Fields

Review:

* Application name
* Process ID
* User
* Direction
* Source address
* Source port
* Destination address
* Destination port
* Protocol
* Interface
* Layer name
* Filter runtime ID
* Filter origin
* Rule name
* Firewall profile
* Event timestamp
* Reporting host

---

## Investigation Questions

The final report should answer:

* Which source system initiated the connection?
* Which destination system was targeted?
* Which port and protocol were used?
* Was the traffic inbound or outbound?
* Was the destination listener active?
* Which firewall rule blocked the traffic?
* Which firewall profile was active?
* Did the Windows firewall text log record the drop?
* Did Windows Event Viewer record the block?
* Was the process or application identified?
* Did Wazuh collect the event?
* Did Splunk collect the event?
* Did a packet capture show the failed connection?
* Was the block expected?
* Was connectivity restored after cleanup?
* What fields were missing?
* What would make the event higher priority?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* WIN11TARGET
* KALI-TEST
* WAZUH-SERVER
* SPLUNK-SERVER

Suggested pre-exercise snapshot:

```
PRE-EX08-Firewall-Block-Investigation
```

A snapshot is recommended before changing firewall policy or audit settings.

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

On KALI-TEST, Wazuh, and Splunk:

```
timedatectl
```

Record any time difference.

---

## Confirm Host-Only Network Scope

On KALI-TEST:

```
ip address
```

```
ip route
```

On WIN11TARGET:

```
Get-NetIPConfiguration
```

```
Get-NetRoute |
    Sort-Object DestinationPrefix, RouteMetric
```

Confirm:

* Both systems use the host-only CyberLab network.
* No bridged adapter is active.
* The intended test address is identified.
* The test does not target an external interface.

---

## Confirm Basic Connectivity

From KALI-TEST:

```
ping -c 4 <WINDOWS_ENDPOINT>
```

A successful ping confirms basic network reachability when ICMP is permitted.

Ping does not validate the test TCP port.

---

## Confirm Windows Firewall Is Enabled

Run on WIN11TARGET as an administrator:

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        DefaultInboundAction,
        DefaultOutboundAction,
        LogAllowed,
        LogBlocked,
        LogFileName,
        LogMaxSizeKilobytes
```

Expected:

```
Enabled: True
```

Do not disable the firewall for the exercise.

---

## Identify the Active Firewall Profile

```
Get-NetConnectionProfile |
    Select-Object `
        InterfaceAlias,
        NetworkCategory,
        IPv4Connectivity,
        IPv6Connectivity
```

Record whether the host-only network is using:

* DomainAuthenticated
* Private
* Public

The temporary rule should apply to the active profile or be explicitly scoped.

---

## Confirm the Firewall Service

```
Get-Service MpsSvc
```

Expected:

```
Running
```

---

## Back Up the Firewall Configuration

Run as an administrator:

```
netsh advfirewall export "<BACKUP_PATH>\firewall-pre-ex08.wfw"
```

Store the backup privately.

This provides a recovery option but should not replace careful rule design.

---

# Firewall Logging Configuration

## Review Dropped-Packet Logging

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        LogBlocked,
        LogAllowed,
        LogFileName,
        LogMaxSizeKilobytes
```

The recommended exercise state is:

```
LogBlocked: True
```

Logging allowed traffic is optional and may generate unnecessary volume.

---

## Enable Dropped-Packet Logging

Run as an administrator only when required:

```
Set-NetFirewallProfile `
    -Profile Domain,Private,Public `
    -LogBlocked True
```

This changes firewall logging across all listed profiles.

A narrower change may target only the active profile.

Example:

```
Set-NetFirewallProfile `
    -Profile Private `
    -LogBlocked True
```

Record the original value before changing it.

---

## Review the Firewall Log Path

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        LogFileName
```

A common path is:

```
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
```

The actual path may vary.

Administrator access may be required to read the file.

---

## Confirm the Log Exists

```
Get-Item `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -ErrorAction SilentlyContinue
```

If the file does not yet exist, it may be created after the first logged event.

---

# Audit Policy Validation

## Review Filtering Platform Auditing

Run as an administrator:

```
auditpol /get /subcategory:"Filtering Platform Packet Drop"
```

```
auditpol /get /subcategory:"Filtering Platform Connection"
```

For this exercise, successful and failure auditing may be configured according to the lab policy.

The blocked connection typically requires failure auditing.

---

## Enable Audit Policy Through Group Policy

Preferred Group Policy path:

```
Computer Configuration
└── Windows Settings
    └── Security Settings
        └── Advanced Audit Policy Configuration
            └── System Audit Policies
                └── Object Access
                    ├── Audit Filtering Platform Packet Drop
                    └── Audit Filtering Platform Connection
```

Use Group Policy when the endpoint is domain managed.

Avoid undocumented local policy drift.

---

## Apply Group Policy

```
gpupdate /force
```

Review effective policy:

```
auditpol /get /category:"Object Access"
```

A restart is not usually required for audit policy application, but validate the effective state before testing.

---

# Monitoring Validation

## Confirm Wazuh Agent Status

On WIN11TARGET:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm WIN11TARGET is active in the Wazuh dashboard.

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

Confirm the event channel:

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-Sysmon/Operational" `
    -ErrorAction SilentlyContinue
```

---

## Start the Exercise Record

```
Exercise ID: EX-08
Source system: KALI-TEST
Destination system: WIN11TARGET
Protocol: TCP
Test port:
Active firewall profile:
Dropped-packet logging:
Filtering Platform auditing:
Wazuh agent status:
Splunk forwarder status:
Sysmon status:
Start time:
```

---

# Baseline Listener Setup

## Select a Test Port

Choose a noncritical port that is:

* Not currently in use
* Not required by CyberLab services
* Above the well-known system port range where practical
* Documented in the exercise record

Example placeholder:

```
<TEST_PORT>
```

Do not reuse:

* `53`
* `88`
* `389`
* `445`
* `3389`
* `8000`
* Wazuh communication ports
* Splunk forwarding or management ports

---

## Confirm the Port Is Unused

Run on WIN11TARGET:

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -ErrorAction SilentlyContinue
```

Alternative:

```
netstat -ano |
    Select-String ":<TEST_PORT>"
```

Expected before starting the listener:

```
No active listener on the selected port
```

---

## Start a Temporary Listener

One safe option is a short Python HTTP server when Python is installed:

```
python -m http.server <TEST_PORT> `
    --directory "C:\CyberLab-Test"
```

Run the listener from a standard user session where possible.

Do not expose the service beyond the host-only network.

Alternative tools may be used if they are already approved in the lab.

---

## Confirm the Listener

In another PowerShell window:

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -State Listen
```

Record:

```
Listener process:
Listener process ID:
Local address:
Local port:
Start time:
```

---

## Identify the Listener Process

```
$Connection = Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -State Listen

Get-Process `
    -Id $Connection.OwningProcess
```

Confirm the process is the intended temporary listener.

---

# Baseline Connectivity Test

## Test the Port from KALI-TEST

Run as a normal user:

```
nc -vz <WINDOWS_ENDPOINT> <TEST_PORT>
```

Expected baseline:

```
Connection succeeded
```

Alternative:

```
curl http://<WINDOWS_ENDPOINT>:<TEST_PORT>/
```

Use only when the temporary listener is an HTTP service.

---

## Record Baseline Connectivity

```
Baseline test time:
Source address:
Destination address:
Destination port:
Protocol:
Result:
```

The exercise should not continue until the port is confirmed reachable.

---

# Temporary Firewall Rule

## Create the Block Rule

Run on WIN11TARGET as an administrator:

```
New-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block" `
    -Description "Temporary inbound block rule for authorized CyberLab Exercise EX-08" `
    -Direction Inbound `
    -Action Block `
    -Protocol TCP `
    -LocalPort <TEST_PORT> `
    -Profile <ACTIVE_PROFILE> `
    -Enabled True
```

Replace `<ACTIVE_PROFILE>` with the appropriate profile.

Do not use `Any` unless the exercise design requires it and the risk is understood.

---

## Confirm the Rule

```
Get-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block" |
Select-Object `
    DisplayName,
    Enabled,
    Direction,
    Action,
    Profile,
    Description
```

Review the port filter:

```
Get-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block" |
Get-NetFirewallPortFilter
```

Confirm:

* Direction is inbound.
* Action is block.
* Protocol is TCP.
* Local port is the selected test port.
* Profile matches the active network.
* No unrelated port is affected.

---

## Record the Rule Creation

```
Rule creation time:
Rule name:
Direction:
Action:
Protocol:
Local port:
Profile:
Administrator:
```

---

# Exercise Procedure

## Generate the Blocked Connection

From KALI-TEST:

```
nc -vz -w 5 <WINDOWS_ENDPOINT> <TEST_PORT>
```

Expected result may include:

```
Connection timed out
```

or:

```
Connection failed
```

The exact message depends on firewall behavior and the testing utility.

---

## Record the Blocked Attempt

```
Attempt time:
Source system:
Source address:
Destination system:
Destination address:
Destination port:
Protocol:
Observed result:
```

Perform only the number of attempts required for evidence.

One to three attempts are sufficient.

---

## Confirm the Listener Is Still Running

On WIN11TARGET:

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -State Listen
```

This confirms that the connection failed because of the firewall rule rather than because the service stopped.

---

## Stop Repeating the Test

Once the blocked event is confirmed:

1. Stop additional connection attempts.
2. Record the final attempt time.
3. Preserve the firewall-rule state.
4. Review the firewall text log.
5. Review Event Viewer.
6. Review Wazuh.
7. Review Splunk.
8. Capture optional packet evidence.

---

# Windows Firewall Text Log Validation

## Review Recent Firewall Log Entries

Run as an administrator:

```
Get-Content `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Tail 100
```

Look for:

* `DROP`
* Source address
* Destination address
* Source port
* Destination port
* Protocol
* Direction

---

## Search for the Test Port

```
Select-String `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Pattern "<TEST_PORT>"
```

---

## Typical Firewall Log Fields

A Windows firewall log may include:

```
date
time
action
protocol
src-ip
dst-ip
src-port
dst-port
size
tcpflags
tcpsyn
tcpack
tcpwin
icmptype
icmpcode
info
path
```

The header in the log is the source of truth for the exact field order.

---

## Record Firewall Log Findings

```
Log entry time:
Action:
Protocol:
Source address:
Destination address:
Source port:
Destination port:
Direction:
Path or application:
```

---

# Windows Event Viewer Validation

## Search for Event ID 5157

Run as an administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5157
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Search for the Test Port

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5157
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<TEST_PORT>"
}
```

---

## Review Rule-Change Events

Search for firewall rule changes:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4946, 4947, 4948
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

The temporary rule creation may appear in a different operational channel depending on the Windows version.

---

## Review Windows Firewall Operational Logs

List relevant channels:

```
Get-WinEvent -ListLog *Firewall* |
    Select-Object `
        LogName,
        IsEnabled,
        RecordCount
```

Common channels may include:

```
Microsoft-Windows-Windows Firewall With Advanced Security/Firewall
Microsoft-Windows-Windows Firewall With Advanced Security/ConnectionSecurity
```

Search the Firewall channel:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall"
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    LevelDisplayName,
    Message
```

---

## Event Viewer Procedure

1. Open **Event Viewer**.
2. Navigate to **Windows Logs**.
3. Select **Security**.
4. Filter for `5152`, `5153`, and `5157`.
5. Use the exercise time range.
6. Open the matching event.
7. Review the **General** tab.
8. Review XML under **Details**.
9. Record network and application fields.
10. Review the Windows Firewall operational channel.
11. Save sanitized evidence.

---

## Review Event ID 5157 Fields

Review:

* Application information
* Process ID
* Application name
* Direction
* Source address
* Source port
* Destination address
* Destination port
* Protocol
* Layer name
* Filter runtime ID

---

## Record Windows Event Findings

```
Event ID:
Event time:
Application:
Process ID:
Direction:
Source address:
Source port:
Destination address:
Destination port:
Protocol:
Layer:
Filter runtime ID:
Rule or filter context:
```

---

# Wazuh Validation

## Confirm Agent Health

In the Wazuh dashboard, confirm:

* WIN11TARGET is active
* Last keepalive is recent
* The selected time range includes the exercise
* The endpoint identity is correct

---

## Search by Event ID

Search for:

```
5152
5153
5157
4946
4948
```

Also search the Windows Firewall operational event IDs generated by the installed Windows version.

---

## Search by Test Port

Search for:

```
<TEST_PORT>
```

Use a narrow time range.

---

## Search by Source Address

Search for the sanitized internal source address corresponding to KALI-TEST.

Do not publish the operational address.

---

## Review Wazuh Fields

Record:

* Agent
* Event provider
* Event channel
* Event ID
* Rule ID
* Rule description
* Rule level
* Application
* Process ID
* Direction
* Source address
* Source port
* Destination address
* Destination port
* Protocol
* User
* Timestamp
* MITRE ATT&CK mapping
* Compliance mappings

---

## Wazuh Validation Questions

* Did Wazuh collect the blocked connection event?
* Did it collect the temporary rule creation event?
* Was the direction parsed correctly?
* Were source and destination addresses parsed correctly?
* Was the destination port parsed correctly?
* Was the application identified?
* Was the rule severity appropriate?
* Was the event received promptly?
* Did repeated attempts create duplicate or excessive alerts?
* Did the rule-removal event appear during cleanup?

---

## Wazuh Detection Gap Record

```
Blocked event present:
Rule creation event present:
Direction parsed:
Source address parsed:
Destination address parsed:
Destination port parsed:
Protocol parsed:
Application present:
Process ID present:
Rule:
Rule level:
Missing fields:
Recommended improvement:
```

A blocked connection is not automatically malicious.

---

# Splunk Validation

## Search for Filtering Platform Events

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| table
    _time
    host
    EventCode
    Application
    ProcessID
    Direction
    SourceAddress
    SourcePort
    DestAddress
    DestPort
    Protocol
    Message
| sort _time
```

Field names depend on the installed Windows add-on.

---

## Search by Test Port

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| search "<TEST_PORT>"
| table
    _time
    host
    EventCode
    Application
    Direction
    SourceAddress
    SourcePort
    DestAddress
    DestPort
    Protocol
| sort _time
```

---

## Search for Firewall Rule Changes

```spl
index=windows earliest=-30m
(
    EventCode=4946 OR
    EventCode=4947 OR
    EventCode=4948
)
| table
    _time
    host
    EventCode
    user
    RuleName
    RuleId
    Profile
    Direction
    Action
    Message
| sort _time
```

---

## Search Windows Firewall Operational Events

```spl
index=windows earliest=-30m
source="*Windows Firewall With Advanced Security*"
| table
    _time
    host
    EventCode
    Level
    user
    Message
| sort _time
```

Adjust the source or sourcetype for the deployment.

---

## Search Ingested Firewall Text Logs

When `pfirewall.log` is ingested into Splunk:

```spl
index=<FIREWALL_INDEX> earliest=-30m
action=DROP
dest_port=<TEST_PORT>
| table
    _time
    host
    action
    protocol
    src_ip
    src_port
    dest_ip
    dest_port
    direction
    path
| sort _time
```

Field extraction depends on the log input and sourcetype.

---

## Build a Firewall Timeline

```spl
index=windows earliest=-30m
(
    EventCode=4946 OR
    EventCode=4947 OR
    EventCode=4948 OR
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval activity=case(
    EventCode=4946, "Firewall rule added",
    EventCode=4947, "Firewall rule modified",
    EventCode=4948, "Firewall rule deleted",
    EventCode=5152, "Packet blocked",
    EventCode=5153, "Packet blocked by restrictive filter",
    EventCode=5157, "Connection blocked",
    true(), "Related firewall event"
)
| table
    _time
    host
    EventCode
    activity
    user
    Application
    Direction
    SourceAddress
    SourcePort
    DestAddress
    DestPort
    Protocol
    RuleName
| sort _time
```

---

## Normalize Network Fields

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval src_port=coalesce(SourcePort, src_port)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=coalesce(DestPort, dest_port)
| eval application=coalesce(Application, ProcessName)
| table
    _time
    host
    EventCode
    application
    Direction
    src_ip
    src_port
    dest_ip
    dest_port
    Protocol
| sort _time
```

Review raw events before making permanent field mappings.

---

## Count Blocks by Source

```spl
index=windows earliest=-24h
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_port=coalesce(DestPort, dest_port)
| stats
    count as blocked_events
    dc(dest_port) as unique_ports
    values(dest_port) as destination_ports
    values(host) as destination_hosts
    by src_ip
| sort - blocked_events
```

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| search "<TEST_PORT>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    EventCode
    SourceAddress
    DestPort
| sort _time
```

---

## Splunk Validation Questions

* Which index contained the event?
* Which event ID represented the block?
* Was the source address extracted?
* Was the destination port extracted?
* Was the direction extracted?
* Was the application or process present?
* Was the firewall rule change visible?
* Was ingestion timely?
* Were duplicate events present?
* Could the event be correlated with the rule creation?
* Could repeated blocks from one source be detected reliably?

---

# Optional Sysmon Validation

## Review Process Creation

If the temporary listener was started through PowerShell or Python, Sysmon may record the process.

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "python\.exe|powershell\.exe"
}
```

---

## Review Network Connections

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 3
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<TEST_PORT>"
}
```

A successful baseline connection may appear.

The blocked attempt may not appear if the connection was never established.

---

## Splunk Sysmon Search

```spl
index=windows EventCode=3 source="*Sysmon*" earliest=-30m
| search DestinationPort=<TEST_PORT> OR SourcePort=<TEST_PORT>
| table
    _time
    host
    User
    Image
    ProcessId
    SourceIp
    SourcePort
    DestinationIp
    DestinationPort
    Protocol
    Initiated
| sort _time
```

Use this to compare the successful baseline connection with the blocked attempt.

---

# Optional Packet Capture

## Start the Capture

On KALI-TEST:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <WINDOWS_ENDPOINT> \
    and tcp port <TEST_PORT> \
    -w <CAPTURE_FILE>.pcap
```

Root privileges are required.

---

## Generate One Blocked Attempt

In another terminal:

```
nc -vz -w 5 <WINDOWS_ENDPOINT> <TEST_PORT>
```

Stop the capture immediately afterward.

---

## Review with tcpdump

```
tcpdump \
    -nn \
    -r <CAPTURE_FILE>.pcap
```

---

## Expected Packet Behavior

Depending on the firewall action, the capture may show:

* TCP SYN transmitted
* No SYN-ACK returned
* Retransmitted SYN packets
* Connection timeout
* TCP reset in some configurations

The capture confirms the network behavior but does not identify the Windows firewall rule by itself.

---

## Wireshark Display Filter

```
tcp.port == <TEST_PORT>
```

More specific:

```
ip.addr == <WINDOWS_ENDPOINT> && tcp.port == <TEST_PORT>
```

---

## Packet Capture Security

The capture may contain:

* Internal addresses
* MAC addresses
* Hostnames
* Timestamps
* Other nearby traffic

Store the raw capture privately.

Publish only a sanitized screenshot or synthetic example.

---

# Investigation

## Build the Timeline

| Time     | System      | Event                 | Result     | Analyst Note                |
| -------- | ----------- | --------------------- | ---------- | --------------------------- |
| `<TIME>` | WIN11TARGET | Listener started      | Listening  | Approved test service       |
| `<TIME>` | KALI-TEST   | Baseline connection   | Successful | Port reachable before block |
| `<TIME>` | WIN11TARGET | Firewall rule created | Active     | Temporary inbound block     |
| `<TIME>` | KALI-TEST   | Connection attempted  | Blocked    | Expected test result        |
| `<TIME>` | WIN11TARGET | Firewall event        | Recorded   | Filtering Platform block    |
| `<TIME>` | Wazuh       | Alert or event        | Received   | Firewall telemetry ingested |
| `<TIME>` | Splunk      | Indexed event         | Searchable | Block timeline created      |
| `<TIME>` | WIN11TARGET | Firewall rule removed | Complete   | Cleanup action              |
| `<TIME>` | KALI-TEST   | Validation connection | Successful | Connectivity restored       |

---

## Root Cause

For this controlled exercise:

```
An authorized inbound connection from KALI-TEST was blocked by a temporary Windows Defender Firewall rule created for CyberLab Exercise EX-08.
```

---

## Why Firewall Blocks Matter

Firewall blocks may represent:

* Normal denied background traffic
* Service discovery
* Misconfigured applications
* Unauthorized remote access
* Port scanning
* Malware communication
* Lateral movement attempts
* Policy enforcement
* Incorrect firewall rules
* Host hardening
* Segmentation controls

A block event demonstrates prevention, but the context determines whether the activity is suspicious.

---

## Benign Indicators

Indicators supporting an authorized test conclusion include:

* Known CyberLab source
* Approved test window
* One documented destination port
* Known temporary listener
* Temporary rule with clear description
* Low event volume
* No related suspicious process
* No additional target ports
* Cleanup matching the exercise plan

---

## Suspicious Indicators

Concern would increase when:

* One source targets many ports
* One source targets many hosts
* The source is unknown
* The target port is associated with remote administration
* The traffic occurs outside an approved window
* The source follows failed authentication activity
* The traffic originates from an unexpected process
* Blocks occur across multiple endpoints
* The source continues after repeated denial
* Firewall rules are disabled or modified afterward

---

## Direction Analysis

Confirm whether the blocked traffic was:

* Inbound to WIN11TARGET
* Outbound from WIN11TARGET

Inbound and outbound blocks have different meanings.

An outbound block may indicate:

* Application misconfiguration
* Denied update traffic
* Policy enforcement
* Suspicious external communication

An inbound block may indicate:

* Scanning
* Unauthorized service access
* Misconfigured management traffic
* Expected segmentation

---

## Process Analysis

When an application is available, review:

* Executable path
* Process ID
* User
* Parent process
* Command line
* Signature
* Hash
* Expected network behavior

For inbound blocks, the application may represent the listening service or the Windows kernel networking context.

---

## Rule Analysis

Review:

* Rule name
* Description
* Direction
* Action
* Protocol
* Port
* Profile
* Program scope
* Service scope
* Local address
* Remote address
* Enabled state
* Creation time
* Creator

A clearly named temporary rule is easier to investigate and remove safely.

---

## False-Positive Analysis

Legitimate blocked traffic may come from:

* Vulnerability scanners
* Monitoring tools
* Software discovery
* Endpoint-management tools
* Backup software
* Printers
* File-sharing clients
* Network troubleshooting
* Misconfigured services
* Old application settings
* Security validation

A high volume of expected blocks may still require firewall or application tuning.

---

# Detection Development

## Basic Filtering Platform Block Search

```spl
index=windows earliest=-24h
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| table
    _time
    host
    EventCode
    Application
    Direction
    SourceAddress
    SourcePort
    DestAddress
    DestPort
    Protocol
| sort - _time
```

---

## One Source Targeting Multiple Ports

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_port=coalesce(DestPort, dest_port)
| stats
    count as blocked_events
    dc(dest_port) as unique_ports
    values(dest_port) as destination_ports
    values(host) as destination_hosts
    by src_ip
| where unique_ports >= <PORT_THRESHOLD>
```

This may indicate reconnaissance but requires context.

---

## One Source Targeting Multiple Hosts

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| stats
    count as blocked_events
    dc(host) as unique_hosts
    values(host) as targeted_hosts
    values(DestPort) as destination_ports
    by src_ip
| where unique_hosts >= <HOST_THRESHOLD>
```

---

## Repeated Blocks to a Sensitive Port

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5157
)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| where dest_port IN (
    22,
    135,
    139,
    445,
    3389,
    5985,
    5986
)
| stats
    count as blocked_events
    values(SourceAddress) as source_addresses
    values(Application) as applications
    by host dest_port
| where blocked_events >= <BLOCK_THRESHOLD>
```

Ports should be adapted to the environment.

A port number alone does not establish malicious intent.

---

## Firewall Rule Added by an Unapproved Actor

```spl
index=windows earliest=-24h
(
    EventCode=4946 OR
    EventCode=4947
)
| eval actor=coalesce(SubjectUserName, user)
| where NOT actor IN (
    "<APPROVED_ADMIN_1>",
    "<APPROVED_ADMIN_2>"
)
| table
    _time
    host
    EventCode
    actor
    RuleName
    Direction
    Action
    Profile
```

---

## Firewall Service Stopped

```spl
index=windows earliest=-24h
EventCode=5025
| table
    _time
    host
    user
    Message
| sort - _time
```

Unexpected firewall service stoppage should receive higher priority than a routine blocked packet.

---

## Wazuh Rule Concept: Blocked Connection

```xml
<group name="windows,firewall,blocked_connection,">
  <rule id="<CUSTOM_RULE_ID>" level="5">
    <field name="win.system.eventID">5152|5153|5157</field>
    <description>Windows Defender Firewall blocked network traffic</description>
  </rule>
</group>
```

Validate the actual decoded field names.

A single blocked connection should normally remain low or informational unless additional context increases severity.

---

## Wazuh Rule Concept: Repeated Blocks

```xml
<group name="windows,firewall,repeated_blocks,">
  <rule id="<CUSTOM_RULE_ID>" level="9" frequency="<EVENT_COUNT>" timeframe="<SECONDS>">
    <if_matched_sid><BASE_FIREWALL_BLOCK_RULE_ID></if_matched_sid>
    <same_source_ip />
    <description>Repeated Windows Firewall blocks from the same source</description>
  </rule>
</group>
```

This is a conceptual example.

Frequency correlation must be tested against normal background traffic.

---

## MITRE ATT&CK Context

Blocked network traffic may support analysis of techniques such as:

* T1046 — Network Service Discovery
* T1021 — Remote Services
* T1210 — Exploitation of Remote Services
* T1071 — Application Layer Protocol
* T1095 — Non-Application Layer Protocol

The ATT&CK mapping should reflect the observed behavior rather than the firewall block alone.

The authorized exercise does not establish malicious intent.

---

## Detection Severity Guidance

| Pattern                                           | Suggested Context           |
| ------------------------------------------------- | --------------------------- |
| One block from known test host                    | Informational or Low        |
| Repeated blocks to one noncritical port           | Low or Medium               |
| One source targets many ports                     | Medium or High              |
| One source targets many hosts                     | High investigation priority |
| Repeated blocks to administrative services        | High                        |
| Unknown external source reaches internal endpoint | High                        |
| Outbound block from unusual process               | High                        |
| Firewall rule changed by unapproved account       | High                        |
| Firewall service disabled                         | Critical review             |
| Blocks followed by successful unauthorized access | Critical                    |

Severity should reflect source, destination, process, frequency, privilege, and impact.

---

# Evidence Collection

## Recommended Evidence

Preserve:

* Firewall profile baseline
* Dropped-packet logging state
* Audit-policy state
* Listener process
* Successful baseline connection
* Temporary firewall rule
* Blocked connection output
* Firewall text-log entry
* Filtering Platform event
* Rule-creation event
* Wazuh result
* Splunk timeline
* Optional Sysmon result
* Optional packet capture
* Rule-removal event
* Restored connectivity
* Final firewall state

---

## Export Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\WIN11TARGET-Security.evtx"
```

Raw Security logs may contain unrelated sensitive events.

Store them privately.

---

## Export Firewall Operational Events

```
wevtutil epl `
    "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall" `
    "<EVIDENCE_PATH>\WIN11TARGET-Firewall.evtx"
```

The channel name should be verified on the installed Windows version.

---

## Copy the Firewall Text Log

```
Copy-Item `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Destination "<EVIDENCE_PATH>\pfirewall-ex08.log"
```

Administrator privileges may be required.

---

## Export Splunk Results

Export the narrow firewall timeline as:

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
Exercise: EX-08
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

Before removing the firewall rule:

1. Capture the rule configuration.
2. Export relevant Windows events.
3. Copy the firewall text log.
4. Save Wazuh results.
5. Save Splunk results.
6. Save optional packet evidence.
7. Confirm the listener is still active.
8. Record the blocked state.

---

## Remove the Temporary Rule

Run on WIN11TARGET as an administrator:

```
Remove-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block"
```

---

## Confirm the Rule Is Removed

```
Get-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block" `
    -ErrorAction SilentlyContinue
```

Expected:

```
No matching rule
```

---

## Confirm the Rule-Removal Event

Search:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4948
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

The deletion event may appear in a Windows Firewall operational channel instead, depending on the version and audit configuration.

---

## Retest Connectivity

From KALI-TEST:

```
nc -vz -w 5 <WINDOWS_ENDPOINT> <TEST_PORT>
```

Expected:

```
Connection succeeded
```

This confirms:

* The listener is active.
* The temporary block was removed.
* Network connectivity is restored.

---

## Stop the Temporary Listener

On WIN11TARGET, return to the listener terminal and stop it with:

```
Ctrl+C
```

Then confirm:

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -ErrorAction SilentlyContinue
```

Expected:

```
No active listener
```

---

## Restore Firewall Logging if It Was Temporary

If dropped-packet logging was enabled only for this exercise, restore the original setting.

Example:

```
Set-NetFirewallProfile `
    -Profile Private `
    -LogBlocked False
```

Only restore the setting when it was previously disabled and the change is documented.

Keeping blocked-packet logging enabled may be preferable for a security lab.

---

## Confirm Firewall Health

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        DefaultInboundAction,
        DefaultOutboundAction,
        LogBlocked,
        LogFileName
```

Confirm all intended profiles remain enabled.

---

## Confirm Firewall Service Health

```
Get-Service MpsSvc
```

Expected:

```
Running
```

---

## Confirm Monitoring Health

Wazuh:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Splunk:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm both platforms remain operational.

---

## Final State Record

```
Baseline connectivity confirmed:
Temporary listener started:
Temporary block rule created:
Blocked connection confirmed:
Firewall log reviewed:
Windows events reviewed:
Wazuh reviewed:
Splunk reviewed:
Temporary rule removed:
Removal event confirmed:
Connectivity restored:
Temporary listener stopped:
Firewall logging restored or retained intentionally:
Firewall enabled:
Wazuh healthy:
Splunk healthy:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* Both systems were confirmed on the host-only network.
* Windows Defender Firewall remained enabled.
* A noncritical test port was selected.
* A temporary listener was started.
* Baseline connectivity succeeded.
* A narrow temporary firewall rule was created.
* The controlled connection was blocked.
* The listener remained active during the block.
* The firewall text log was reviewed.
* A relevant Windows event was reviewed.
* Wazuh was reviewed.
* Splunk was reviewed.
* Optional packet or Sysmon telemetry was reviewed where configured.
* The temporary rule was removed.
* Connectivity was restored.
* The listener was stopped.
* Final firewall health was confirmed.
* Evidence was preserved.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                         | Result                       |
| -------------------------------- | ---------------------------- |
| Firewall enabled before test     | Pass / Fail                  |
| Baseline connection successful   | Pass / Fail                  |
| Temporary rule created correctly | Pass / Fail                  |
| Blocked connection confirmed     | Pass / Fail                  |
| Firewall text log event          | Pass / Fail / Not Configured |
| Windows Filtering Platform event | Pass / Fail / Not Configured |
| Application context              | Pass / Partial / Fail        |
| Source address                   | Pass / Partial / Fail        |
| Destination port                 | Pass / Partial / Fail        |
| Direction                        | Pass / Partial / Fail        |
| Wazuh ingestion                  | Pass / Fail                  |
| Wazuh alert quality              | Pass / Needs Tuning / Fail   |
| Splunk ingestion                 | Pass / Fail                  |
| Splunk field quality             | Pass / Needs Tuning / Fail   |
| Packet evidence                  | Pass / Fail / Not Collected  |
| Rule removed                     | Pass / Fail                  |
| Connectivity restored            | Pass / Fail                  |
| Cleanup                          | Complete / Incomplete        |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Firewall rule name
* Filter identifier
* Application
* Process ID
* User
* Direction
* Source address
* Source port
* Destination address
* Destination port
* Protocol
* Active firewall profile
* Rule creator
* Correlation between rule creation and block
* Correlation between rule removal and restored access
* Clear severity

---

## Recommended Improvements

Potential improvements include:

* Enable dropped-packet logging
* Enable Filtering Platform auditing
* Ingest `pfirewall.log` into Splunk
* Collect Windows Firewall operational channels
* Normalize network fields
* Correlate rule changes with blocked traffic
* Detect one source targeting multiple ports
* Detect one source targeting multiple hosts
* Add sensitive-port context
* Add process context through Sysmon
* Monitor firewall-service state
* Create a firewall block investigation runbook

---

# Troubleshooting

## Baseline Connection Fails Before the Rule Is Created

Check:

* Listener process
* Correct test port
* Correct destination address
* Host-only network connectivity
* Existing firewall rules
* Active profile
* Listener binding address
* Service crash
* Route
* VMware adapter state

On WIN11TARGET:

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -State Listen
```

From KALI-TEST:

```
nc -vz -w 5 <WINDOWS_ENDPOINT> <TEST_PORT>
```

Do not create the block rule until baseline connectivity succeeds.

---

## The Connection Still Succeeds After Rule Creation

Check:

* Rule enabled state
* Correct direction
* Correct local port
* Correct protocol
* Active profile
* Rule precedence
* Listener using a different port
* Test using IPv6 instead of IPv4
* Rule scoped to the wrong program or address

Review:

```
Get-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block"
```

```
Get-NetFirewallRule `
    -DisplayName "CyberLab EX-08 Temporary Block" |
Get-NetFirewallPortFilter
```

---

## The Rule Blocks an Unintended Service

Stop the exercise immediately.

Then:

1. Preserve the rule state.
2. Remove the temporary rule.
3. Confirm service recovery.
4. Review the selected port and protocol.
5. Confirm no broad address or program scope was used.
6. Document the error privately.
7. Choose a safer test port.

---

## No Firewall Text Log Entry Appears

Check:

* `LogBlocked` enabled
* Correct firewall profile
* Correct log path
* Log directory permissions
* File size limit
* Connection actually blocked
* Event time
* IPv4 versus IPv6
* Firewall service health

Review:

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        LogBlocked,
        LogFileName,
        LogMaxSizeKilobytes
```

---

## No Event ID 5157 Appears

Check:

* Filtering Platform Connection auditing
* Security log time range
* Correct event ID
* Correct endpoint
* Audit policy controlled by Group Policy
* Event log retention
* Whether the action produced a packet-drop event instead

Review:

```
auditpol /get /subcategory:"Filtering Platform Connection"
```

```
auditpol /get /subcategory:"Filtering Platform Packet Drop"
```

Search for `5152`, `5153`, and `5157`.

---

## Application Field Is Blank or Shows System

Possible causes include:

* Traffic was blocked before application association
* Kernel networking context
* Inbound packet without an established connection
* Event-source limitation
* Listener implementation
* Field extraction problem

Correlate with:

* Listener process
* Port ownership
* Sysmon process event
* Baseline connection
* Firewall rule scope

---

## Source Address Is Missing

Possible causes include:

* Field extraction issue
* Event type
* IPv6 representation
* Firewall operational event format
* Text-log parser issue

Review raw event XML and the firewall text-log header.

---

## Wazuh Does Not Show the Event

Check:

* WIN11TARGET agent status
* Security log collection
* Firewall operational channel collection
* Event decoder
* Rule level
* Dashboard time range
* Agent filter
* Indexer health

Confirm the event exists locally before changing Wazuh.

---

## Splunk Does Not Show the Event

Check:

* Universal Forwarder service
* Security log input
* Firewall operational channel input
* Firewall text-log input
* Receiver
* Target index
* Host
* Source
* Sourcetype
* Time range
* Forwarder logs

Begin with:

```spl
index=windows host=WIN11TARGET earliest=-30m
| stats count by source sourcetype EventCode
```

---

## Packet Capture Shows No Traffic

Check:

* Correct interface
* Correct destination address
* Correct port
* Capture started before test
* Host-only adapter
* IPv4 versus IPv6
* Capture permissions
* BPF filter

Run without a port filter briefly only when necessary and authorized.

---

## Connectivity Does Not Return After Rule Removal

Check:

* Rule actually removed
* Other block rules
* Listener still running
* Listener port
* Firewall profile
* Network route
* VMware adapter
* Service process
* Address family

Confirm:

```
Get-NetFirewallRule |
    Where-Object DisplayName -Match "EX-08|Temporary Block"
```

```
Get-NetTCPConnection `
    -LocalPort <TEST_PORT> `
    -State Listen
```

---

## Firewall Service Stops

Treat this as an unexpected high-priority condition.

Check:

```
Get-Service MpsSvc
```

Review System and Security logs.

Do not continue the exercise until the firewall is restored and the cause is understood.

---

# Public Sanitization

Remove or replace:

* Operational IP addresses
* MAC addresses
* Real usernames
* Internal domain names
* Exact firewall profile details when sensitive
* Real rule identifiers
* Filter runtime IDs
* Security identifiers
* Internal file paths beyond safe examples
* Wazuh agent identifiers
* Splunk internal URLs
* Packet captures
* Evidence storage paths
* Unrelated firewall rules
* Public or household network information

Use placeholders such as:

```
<SOURCE_SYSTEM>
<SOURCE_IP>
<WINDOWS_ENDPOINT>
<DESTINATION_IP>
<TEST_PORT>
<PROTOCOL>
<FIREWALL_RULE>
<ACTIVE_PROFILE>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Windows Firewall profile status
* Temporary firewall rule
* Successful baseline connection
* Blocked connection result
* Firewall text-log entry
* Event ID `5157`
* Wazuh firewall event
* Splunk firewall timeline
* Packet retransmission pattern
* Rule-removal validation
* Restored connectivity

Before publishing:

1. Crop unrelated information.
2. Remove operational IP addresses.
3. Remove real usernames.
4. Remove SIDs and filter identifiers.
5. Remove unrelated firewall rules.
6. Hide browser tabs and bookmarks.
7. Remove MAC addresses from packet evidence.
8. Permanently redact sensitive values.
9. Reopen and inspect the final image.

Do not upload the raw packet capture.

---

# Exercise Report Template

## Executive Summary

```
A controlled inbound connection from KALI-TEST to a temporary service on WIN11TARGET was blocked by a narrowly scoped Windows Defender Firewall rule during CyberLab Exercise EX-08. The resulting firewall text log, Windows Filtering Platform event, Wazuh event, and Splunk telemetry were reviewed. The temporary rule was removed, connectivity was restored, and the firewall remained enabled.
```

## Findings

```
Source system:

Destination system:

Protocol:

Destination port:

Active firewall profile:

Baseline connection result:

Temporary rule:

Rule creation time:

Blocked attempt time:

Firewall text-log result:

Windows event ID:

Application:

Direction:

Source address:

Source port:

Destination address:

Destination port:

Wazuh result:

Splunk result:

Packet capture result:

Rule removal time:

Restored connection result:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
An authorized connection was blocked by a temporary Windows Defender Firewall rule created for CyberLab Exercise EX-08.
```

## Resolution

```
The temporary block rule was removed after evidence collection, the test connection was successfully revalidated, the temporary listener was stopped, and the final firewall and monitoring states were confirmed healthy.
```

## Recommendations

```
- Enable and retain dropped-packet logging where appropriate.
- Enable Windows Filtering Platform auditing.
- Ingest the Windows Firewall text log into Splunk.
- Normalize source, destination, port, direction, and process fields.
- Correlate firewall rule changes with blocked traffic.
- Detect repeated blocks across multiple ports or hosts.
- Create a firewall block investigation runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshots exist
* Source and destination systems on host-only network
* Active firewall profile identified
* Windows Firewall enabled
* Firewall service running
* Original logging state recorded
* Dropped-packet logging reviewed
* Filtering Platform auditing reviewed
* Wazuh healthy
* Splunk healthy
* Optional Sysmon reviewed
* Test port selected
* Start time recorded

## Baseline

* Test port confirmed unused
* Temporary listener started
* Listener process identified
* Baseline connection succeeded
* Baseline time recorded

## Exercise

* Temporary block rule created
* Rule scope validated
* Rule creation time recorded
* Connection attempt generated
* Connection blocked
* Listener remained active
* No unnecessary repeated attempts
* No critical service affected

## Investigation

* Firewall text log reviewed
* Filtering Platform events reviewed
* Rule-change events reviewed
* Direction identified
* Source identified
* Destination port identified
* Application or process reviewed
* Wazuh reviewed
* Splunk reviewed
* Optional Sysmon reviewed
* Optional packet capture reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Temporary rule removed
* Rule-removal event reviewed
* Connectivity restored
* Temporary listener stopped
* Firewall logging restored or retained intentionally
* Firewall remains enabled
* Firewall service healthy
* Wazuh healthy
* Splunk healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Windows Defender Firewall administration
* Windows Filtering Platform analysis
* Firewall text-log analysis
* TCP connection testing
* Source and destination correlation
* Port and protocol analysis
* Firewall rule review
* Wazuh validation
* Splunk SPL
* Optional Sysmon analysis
* Packet capture analysis
* Timeline reconstruction
* False-positive assessment
* Evidence preservation
* Configuration rollback
* Public sanitization

---

# Lessons Learned

* A successful baseline test is required before attributing failure to the firewall.
* Firewall rules should be narrow, clearly named, and temporary during exercises.
* A blocked connection is not automatically malicious.
* Windows Firewall text logs and Filtering Platform events provide different evidence.
* The text log may show packet details even when process context is unavailable.
* Event ID `5157` can provide application and connection context when auditing is enabled.
* Group Policy is preferable to undocumented local audit-policy changes.
* Sysmon may show the successful baseline connection but not the blocked attempt.
* Packet capture confirms network behavior but does not identify the firewall rule.
* Repeated blocks across ports or systems provide stronger reconnaissance context.
* Firewall-rule changes should be correlated with subsequent network behavior.
* Cleanup must verify both rule removal and restored connectivity.
* The firewall must remain enabled throughout the exercise.

---

# Summary

This exercise validates the CyberLab’s ability to observe and investigate a Windows Defender Firewall block.

A complete investigation should identify:

* The source system
* The destination system
* The protocol
* The destination port
* The firewall direction
* The active profile
* The temporary rule
* The local firewall-log result
* The Windows Filtering Platform event
* The application or process
* The Wazuh result
* The Splunk result
* The packet behavior
* The event timeline
* Missing telemetry
* Appropriate severity
* The final firewall state

The exercise is complete only after the temporary rule is removed, connectivity is restored, the listener is stopped, the firewall remains enabled, evidence is preserved, and detection improvements are documented.

