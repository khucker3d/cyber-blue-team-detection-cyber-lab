# Network Reconnaissance Detection

## Exercise Overview

This exercise generates a controlled, narrow network reconnaissance pattern from KALI-TEST against an authorized CyberLab target and validates the resulting telemetry across:

* Windows Defender Firewall
* Windows Security logs
* Optional Sysmon telemetry
* Optional packet capture
* Wazuh
* Splunk
* Investigation notes and evidence

The exercise is designed to answer:

* Which system initiated the scan
* Which host was targeted
* Which ports were probed
* Whether the ports were open, closed, or filtered
* Which telemetry sources observed the activity
* Whether the SIEMs detected the reconnaissance pattern
* Whether the available evidence distinguished a scan from ordinary connection attempts
* Which additional network controls would improve visibility

All activity must remain inside the VMware host-only CyberLab network.

---

## Exercise ID

```
EX-09
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Authorized reconnaissance testing
* Nmap scan interpretation
* Windows Firewall analysis
* Network-event correlation
* Source and destination analysis
* Wazuh event validation
* Splunk SPL development
* Optional Sysmon analysis
* Packet capture analysis
* Detection-gap assessment
* Evidence preservation

---

## Authorization Boundary

This exercise must use:

* KALI-TEST as the source system
* WIN11TARGET as the primary destination
* The VMware host-only CyberLab network
* A narrow, documented port list
* A low scan rate
* Approved test times
* Wazuh and Splunk monitoring
* Synthetic or lab-generated evidence

Do not scan:

* The physical home network
* The ISP gateway
* Omada infrastructure
* The Raspberry Pi utility server
* The NAS
* Personal devices
* Public IP addresses
* School systems
* Employer systems
* Cloud systems
* Any host outside the CyberLab
* Any target without explicit authorization

---

## Preferred Exercise Design

The preferred test uses:

* One authorized source
* One authorized destination
* Five or fewer TCP ports
* TCP connect scanning
* No service-version detection
* No operating-system fingerprinting
* No vulnerability scripts
* No UDP scanning
* No host discovery outside the known target
* A slow or normal scan rate

This creates enough activity for defensive validation without producing unnecessary traffic.

---

## Public Example Environment

```
Source system: KALI-TEST
Destination system: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Documentation subnet: 192.0.2.0/24
Approved ports: <PORT_1>, <PORT_2>, <PORT_3>
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Define and document an authorized scan scope
* Validate source and target network configuration
* Establish a known service baseline
* Run a narrow TCP scan
* Interpret open, closed, and filtered results
* Review Windows Firewall telemetry
* Review relevant Windows events
* Review optional Sysmon events
* Capture and interpret scan traffic
* Locate the activity in Wazuh
* Locate the activity in Splunk
* Build a reconnaissance timeline
* Identify detection gaps
* Distinguish authorized scanning from suspicious reconnaissance

---

## Required Systems

* KALI-TEST
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on WIN11TARGET
* Splunk Universal Forwarder where configured

Optional:

* Sysmon on WIN11TARGET
* Wireshark or tcpdump
* A dedicated IDS such as Suricata or Zeek in a later phase

DC01 is not required for the standard exercise.

---

## Required Accounts

### Kali Standard User

Used to run the authorized scan.

Root access is not required for a TCP connect scan.

### Windows Standard User

Used for endpoint validation where possible.

### Authorized Windows Administrator

Required only for:

* Firewall logging
* Audit-policy validation
* Protected event-log access
* Temporary listener management where elevation is required

---

## Required Monitoring

Before starting, confirm:

* KALI-TEST and WIN11TARGET are on the host-only network
* No bridged adapter is active
* Windows Defender Firewall is enabled
* Dropped-packet logging is configured where used
* Filtering Platform auditing is enabled where used
* Wazuh agent is active
* Splunk is running
* Optional Sysmon is active where used
* System time is synchronized
* A stable snapshot exists

---

## Safety Notes

* Use only the approved destination.
* Use only the approved port list.
* Do not scan the full subnet.
* Do not use `-A`.
* Do not use `-O`.
* Do not use `-sV` for the standard exercise.
* Do not use NSE vulnerability scripts.
* Do not use aggressive timing.
* Do not use spoofing or decoys.
* Do not use evasion options.
* Stop when the planned scan completes.
* Record the exact command and time.
* Confirm no unintended system was contacted.

---

## Expected Event Flow

```
KALI-TEST
    |
    | Narrow TCP scan
    v
WIN11TARGET
    |
    | Open / closed / filtered responses
    v
Windows Firewall and Network Stack
    |
    v
Windows / Sysmon / Packet Telemetry
    |
    v
Wazuh and Splunk
    |
    v
Reconnaissance Investigation
```

---

## Expected Scan Results

A port may appear as:

| State            | General Meaning                                             |
| ---------------- | ----------------------------------------------------------- |
| Open             | An application accepted the connection                      |
| Closed           | The host responded, but no service was listening            |
| Filtered         | A firewall or other control prevented a clear response      |
| Unfiltered       | The port was reachable, but state determination was limited |
| Open or Filtered | The scan could not distinguish between the two              |

The scan result should be compared with the actual endpoint listener and firewall state.

---

## Relevant Windows Event Sources

Potential telemetry includes:

| Event ID | Source   | Description                                                 |
| -------: | -------- | ----------------------------------------------------------- |
|     5152 | Security | Windows Filtering Platform blocked a packet                 |
|     5153 | Security | Restrictive filter blocked a packet                         |
|     5156 | Security | Windows Filtering Platform permitted a connection           |
|     5157 | Security | Windows Filtering Platform blocked a connection             |
|     5031 | Security | Firewall blocked an application from accepting a connection |
|        3 | Sysmon   | Network connection                                          |
|        1 | Sysmon   | Process creation                                            |

Permitted-connection events can be high volume and should be enabled carefully.

---

## Important Fields

Review:

* Source system
* Source address
* Source port
* Destination system
* Destination address
* Destination port
* Protocol
* Direction
* Application
* Process ID
* Firewall action
* Event count
* Unique port count
* First seen
* Last seen
* Scan duration
* Packet flags
* Connection result

---

## Investigation Questions

The final report should answer:

* Which system initiated the scan?
* Which system was targeted?
* Which ports were included?
* Which ports were open?
* Which ports were closed?
* Which ports were filtered?
* How long did the scan last?
* How many connection attempts occurred?
* Did Windows Firewall log the traffic?
* Did Windows Security events record the traffic?
* Did Sysmon record any successful connections?
* Did Wazuh collect or alert on the scan?
* Did Splunk show a multi-port pattern?
* Did the packet capture show SYN and response behavior?
* Was the activity distinguishable from ordinary network use?
* What data source provided the strongest evidence?
* What telemetry was missing?
* What would increase the severity?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* KALI-TEST
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

Suggested pre-exercise snapshot:

```
PRE-EX09-Network-Reconnaissance
```

A new snapshot is recommended before changing firewall or audit settings.

---

## Confirm System Time

On KALI-TEST:

```
timedatectl
```

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

## Confirm KALI-TEST Network Configuration

Run as a normal user:

```
ip address
```

```
ip route
```

Confirm:

* Host-only adapter is active
* Expected internal address is assigned
* No unintended bridged adapter exists
* Default route is understood
* The target belongs to the authorized lab network

---

## Confirm WIN11TARGET Network Configuration

```
Get-NetIPConfiguration
```

```
Get-NetAdapter |
    Select-Object `
        Name,
        InterfaceDescription,
        Status,
        LinkSpeed
```

```
Get-NetRoute |
    Sort-Object DestinationPrefix, RouteMetric
```

Confirm the intended host-only interface and target address.

---

## Confirm Basic Reachability

From KALI-TEST:

```
ping -c 4 <WINDOWS_ENDPOINT>
```

A failed ping does not necessarily mean the host is unreachable because ICMP may be blocked.

Use a known service connection where appropriate.

---

## Confirm Windows Firewall State

Run as an administrator:

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        DefaultInboundAction,
        DefaultOutboundAction,
        LogBlocked,
        LogAllowed,
        LogFileName
```

All intended profiles should remain enabled.

---

## Confirm the Active Firewall Profile

```
Get-NetConnectionProfile |
    Select-Object `
        InterfaceAlias,
        NetworkCategory,
        IPv4Connectivity,
        IPv6Connectivity
```

Record the active profile for the host-only interface.

---

## Confirm Firewall Logging

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        LogBlocked,
        LogAllowed,
        LogFileName,
        LogMaxSizeKilobytes
```

Use for the exercise:

```
LogBlocked: True
```

Allowed-connection logging is optional and may produce high volume.

---

## Confirm Filtering Platform Auditing

Run as an administrator:

```
auditpol /get /subcategory:"Filtering Platform Packet Drop"
```

```
auditpol /get /subcategory:"Filtering Platform Connection"
```

Document whether failure and success auditing are enabled.

---

## Confirm Wazuh Agent Status

On WIN11TARGET:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm the endpoint is active in Wazuh.

---

## Confirm Splunk Forwarder Status

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm recent events:

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

Confirm the operational log:

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-Sysmon/Operational" `
    -ErrorAction SilentlyContinue
```

---

# Target Baseline

## Select the Approved Port List

Use a short list such as:

```
<PORT_1>
<PORT_2>
<PORT_3>
<PORT_4>
<PORT_5>
```

The list should contain a mix of:

* One known open test port
* One known closed port
* One intentionally filtered port where practical

Do not use critical services unless the test specifically requires them.

---

## Optional Temporary Listener

To create a known open port on WIN11TARGET, start a harmless temporary listener.

Example with Python:

```
python -m http.server <OPEN_TEST_PORT> `
    --directory "C:\CyberLab-Test"
```

Confirm:

```
Get-NetTCPConnection `
    -LocalPort <OPEN_TEST_PORT> `
    -State Listen
```

Record the process and port.

---

## Identify Current Listeners

Run on WIN11TARGET:

```
Get-NetTCPConnection -State Listen |
    Sort-Object LocalPort |
    Select-Object `
        LocalAddress,
        LocalPort,
        OwningProcess
```

Resolve a specific process:

```
Get-Process -Id <PROCESS_ID>
```

Do not publish the complete operational listener inventory without sanitization.

---

## Test One Known Port

From KALI-TEST:

```
nc -vz -w 3 <WINDOWS_ENDPOINT> <OPEN_TEST_PORT>
```

Expected:

```
Connection succeeded
```

This creates a known open-port baseline.

---

## Confirm a Closed Port

Choose an unused test port and confirm no listener exists:

```
Get-NetTCPConnection `
    -LocalPort <CLOSED_TEST_PORT> `
    -ErrorAction SilentlyContinue
```

Expected:

```
No active listener
```

---

## Optional Filtered Port

A temporary narrow firewall rule may be used to block one approved test port.

Use the process from the firewall block exercise and remove the rule during cleanup.

Do not create multiple broad block rules.

---

## Start the Exercise Record

```
Exercise ID: EX-09
Source system: KALI-TEST
Destination system: WIN11TARGET
Scan type: TCP connect
Approved ports:
Known open port:
Known closed port:
Known filtered port:
Firewall profile:
Firewall logging:
Filtering Platform auditing:
Sysmon:
Wazuh status:
Splunk status:
Start time:
```

---

# Exercise Procedure

## Review the Planned Command

The standard scan command is:

```
nmap -sT -Pn -p <PORT_LIST> <WINDOWS_ENDPOINT>
```

Options:

| Option | Purpose                                                  |
| ------ | -------------------------------------------------------- |
| `-sT`  | Uses a full TCP connect scan                             |
| `-Pn`  | Treats the known target as online without host discovery |
| `-p`   | Limits the scan to the approved ports                    |

Do not add unrelated options.

---

## Run the Narrow Scan

From KALI-TEST as a normal user:

```
nmap \
    -sT \
    -Pn \
    -p <PORT_LIST> \
    <WINDOWS_ENDPOINT>
```

Record:

```
Scan start time:
Scan end time:
Source address:
Destination address:
Command:
Ports scanned:
Open ports:
Closed ports:
Filtered ports:
Nmap duration:
```

---

## Save the Scan Output

Save a normal text report:

```
nmap \
    -sT \
    -Pn \
    -p <PORT_LIST> \
    -oN ex09-nmap-results.txt \
    <WINDOWS_ENDPOINT>
```

Store the raw result privately until sanitized.

---

## Stop After the Planned Scan

Do not:

* Expand the port range
* Scan the subnet
* Add version detection
* Add OS detection
* Run scripts
* Increase the timing rate
* Repeat the scan unnecessarily

One or two controlled runs are sufficient.

---

# Local Windows Validation

## Review Firewall Text Logs

Typical path:

```
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
```

Review recent entries:

```
Get-Content `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Tail 200
```

Search for the KALI-TEST source address privately:

```
Select-String `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Pattern "<SOURCE_IP>"
```

Search by approved destination ports:

```
Select-String `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Pattern "<PORT_1>|<PORT_2>|<PORT_3>"
```

---

## Review Filtering Platform Block Events

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5157
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Search for the Source Address

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5157
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<SOURCE_IP>"
}
```

---

## Review Permitted Connection Events

When success auditing is enabled:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5156
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<SOURCE_IP>"
}
```

Event ID `5156` may create substantial volume.

Use a narrow time window and filter.

---

## Record Windows Findings

```
Firewall log entries:
Blocked event IDs:
Permitted event IDs:
Source address:
Unique destination ports:
Open-port evidence:
Closed-port evidence:
Filtered-port evidence:
Application context:
Direction:
Event count:
```

---

# Optional Sysmon Validation

## Review Network Connections

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 3
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<SOURCE_IP>"
}
```

---

## Review the Known Listener

Search for connections to the open test port:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 3
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<OPEN_TEST_PORT>"
}
```

Sysmon may record successful connections but not closed or filtered attempts.

---

## Review Sysmon Fields

Record:

* Image
* Process ID
* User
* Source address
* Source port
* Destination address
* Destination port
* Protocol
* Initiated
* Timestamp

For inbound activity, the event may represent the local listening application.

---

# Optional Packet Capture

## Start the Capture on KALI-TEST

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <WINDOWS_ENDPOINT> \
    and tcp \
    -w ex09-reconnaissance.pcap
```

Root privileges are required.

---

## Run the Approved Scan

In another terminal:

```
nmap \
    -sT \
    -Pn \
    -p <PORT_LIST> \
    <WINDOWS_ENDPOINT>
```

Stop the capture immediately after the scan.

---

## Review the Capture

```
tcpdump \
    -nn \
    -r ex09-reconnaissance.pcap
```

---

## Wireshark Display Filters

Traffic involving the target:

```
ip.addr == <WINDOWS_ENDPOINT> && tcp
```

SYN packets:

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Reset responses:

```
tcp.flags.reset == 1
```

Specific destination ports:

```
tcp.dstport == <PORT_1> || tcp.dstport == <PORT_2> || tcp.dstport == <PORT_3>
```

---

## Interpret TCP Responses

Typical behavior:

| Result   | Packet Pattern                                  |
| -------- | ----------------------------------------------- |
| Open     | SYN, SYN-ACK, ACK                               |
| Closed   | SYN, RST or RST-ACK                             |
| Filtered | SYN with no response or ICMP filtering response |

The precise behavior depends on the firewall and operating system.

---

## Packet Capture Security

The raw capture may contain:

* Internal addresses
* MAC addresses
* Hostnames
* Timestamps
* Other nearby traffic

Store it privately.

Publish only sanitized screenshots or synthetic diagrams.

---

# Wazuh Validation

## Confirm Agent Health

In Wazuh, confirm:

* WIN11TARGET is active
* Last keepalive is recent
* The selected time range includes the scan
* The agent identity is correct

---

## Search by Source Address

Search privately for the KALI-TEST address.

Do not publish the operational value.

---

## Search by Event ID

Review:

```
5152
5153
5156
5157
```

Also review relevant firewall operational events.

---

## Search by Destination Ports

Search for the approved port values individually.

A single scan may create several related events.

---

## Review Wazuh Fields

Record:

* Agent
* Event provider
* Event channel
* Event ID
* Rule ID
* Rule level
* Rule description
* Source address
* Source port
* Destination address
* Destination port
* Direction
* Protocol
* Application
* Event count
* Timestamp
* MITRE ATT&CK mapping

---

## Wazuh Validation Questions

* Did Wazuh collect blocked probes?
* Did Wazuh collect permitted connections?
* Was the source address parsed?
* Were multiple destination ports visible?
* Did Wazuh create one alert per packet or connection?
* Did any correlation rule identify a scan pattern?
* Was the rule severity appropriate?
* Was the application context available?
* Was the event received promptly?
* Did the platform distinguish scanning from ordinary traffic?

---

## Wazuh Detection Gap Record

```
Blocked probes present:
Permitted connections present:
Source address parsed:
Destination ports parsed:
Unique port count visible:
Correlation alert present:
Application context present:
Rule:
Rule level:
Missing fields:
Improvement:
```

Wazuh may collect individual firewall events without correlating them into a reconnaissance alert.

---

# Splunk Validation

## Search for Source Activity

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| search SourceAddress="<SOURCE_IP>"
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

## Normalize Network Fields

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval src_port=coalesce(SourcePort, src_port)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
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

---

## Identify One Source Targeting Multiple Ports

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| stats
    count as connection_events
    dc(dest_port) as unique_ports
    values(dest_port) as destination_ports
    values(EventCode) as event_codes
    values(host) as destination_hosts
    min(_time) as first_seen
    max(_time) as last_seen
    by src_ip
| where unique_ports >= <PORT_THRESHOLD>
| eval duration_seconds=last_seen-first_seen
| convert ctime(first_seen) ctime(last_seen)
```

For this exercise, set the threshold below or equal to the approved number of test ports.

---

## Restrict to One Destination

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| stats
    count as connection_events
    dc(dest_port) as unique_ports
    values(dest_port) as destination_ports
    values(EventCode) as event_codes
    min(_time) as first_seen
    max(_time) as last_seen
    by src_ip dest_ip
| where unique_ports >= <PORT_THRESHOLD>
| eval duration_seconds=last_seen-first_seen
| convert ctime(first_seen) ctime(last_seen)
```

---

## Compare Allowed and Blocked Probes

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval result=case(
    EventCode=5156, "Allowed",
    EventCode=5152 OR EventCode=5153 OR EventCode=5157, "Blocked",
    true(), "Other"
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| search src_ip="<SOURCE_IP>"
| stats
    count
    values(EventCode) as event_codes
    by dest_port result
| sort dest_port result
```

---

## Build the Scan Timeline

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval src_port=coalesce(SourcePort, src_port)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| eval result=case(
    EventCode=5156, "Allowed",
    EventCode=5152 OR EventCode=5153 OR EventCode=5157, "Blocked",
    true(), "Other"
)
| search src_ip="<SOURCE_IP>"
| table
    _time
    host
    EventCode
    result
    src_ip
    src_port
    dest_ip
    dest_port
    Protocol
    Application
| sort _time
```

---

## Compare Nmap and SIEM Results

Create a comparison table:

|       Port | Nmap Result | Windows Event                | Wazuh            | Splunk           | Packet Evidence |
| ---------: | ----------- | ---------------------------- | ---------------- | ---------------- | --------------- |
| `<PORT_1>` | Open        | Permitted                    | Present          | Present          | SYN-ACK         |
| `<PORT_2>` | Closed      | Limited or permitted context | Present / Absent | Present / Absent | RST             |
| `<PORT_3>` | Filtered    | Blocked                      | Present          | Present          | No response     |

The exact mapping depends on the endpoint and firewall configuration.

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| search SourceAddress="<SOURCE_IP>"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    EventCode
    DestPort
| sort _time
```

---

## Splunk Validation Questions

* Which event IDs represented the scan?
* Was the source address extracted?
* Were all destination ports visible?
* Could the scan duration be calculated?
* Were both permitted and blocked attempts visible?
* Was the destination application identified?
* Were duplicate events present?
* Was ingestion timely?
* Did the multi-port search identify the activity?
* Could the search distinguish one user action from a broader scan?
* Which threshold produced useful results without excessive noise?

---

# Investigation

## Build the Timeline

| Time     | System           | Activity                     | Result      | Analyst Note                |
| -------- | ---------------- | ---------------------------- | ----------- | --------------------------- |
| `<TIME>` | WIN11TARGET      | Known listener validated     | Open        | Baseline service            |
| `<TIME>` | KALI-TEST        | Nmap scan started            | In progress | Approved port list          |
| `<TIME>` | WIN11TARGET      | Connection attempts received | Mixed       | Open, closed, filtered      |
| `<TIME>` | Windows Firewall | Probe blocked or permitted   | Recorded    | Filtering Platform evidence |
| `<TIME>` | Wazuh            | Events received              | Present     | Individual telemetry        |
| `<TIME>` | Splunk           | Multi-port pattern detected  | Present     | Reconnaissance correlation  |
| `<TIME>` | KALI-TEST        | Nmap scan completed          | Complete    | No scope expansion          |
| `<TIME>` | WIN11TARGET      | Temporary listener stopped   | Complete    | Cleanup                     |

---

## Root Cause

For this controlled exercise:

```
Authorized narrow TCP reconnaissance from KALI-TEST against WIN11TARGET during CyberLab Exercise EX-09.
```

---

## Why Reconnaissance Matters

Network reconnaissance may be used to identify:

* Live hosts
* Open ports
* Remote services
* Administrative interfaces
* File-sharing services
* Database services
* Security products
* Potential attack paths
* Segmentation weaknesses

Reconnaissance can also be legitimate when performed by:

* Network administrators
* Vulnerability scanners
* Monitoring systems
* Asset inventory tools
* Security teams
* Troubleshooting workflows

Context is required.

---

## Benign Indicators

Indicators supporting an authorized test conclusion include:

* Known KALI-TEST source
* One approved destination
* Small documented port list
* Normal scan timing
* No service or OS detection
* No vulnerability scripts
* No subnet scan
* Approved test window
* Matching exercise record
* Immediate stop after completion

---

## Suspicious Indicators

Concern would increase when:

* One source targets many ports
* One source targets many hosts
* The source is unknown
* The scan occurs outside approved hours
* Administrative ports are targeted
* The scan follows credential compromise
* The source uses high-speed or evasive methods
* Version detection or exploit scripts are used
* The scan crosses network segments
* The activity continues after blocks
* Successful access follows reconnaissance

---

## Scope Analysis

Confirm:

* One source
* One destination
* Approved ports only
* Host-only network
* No public or physical-network targets
* No unintended DNS resolution
* No service scripts
* No additional discovery

Compare the Nmap output with packet capture or logs when possible.

---

## Port-State Analysis

### Open

An open port indicates:

* The host was reachable
* A process was listening
* The firewall allowed the connection

Review the owning process and whether the service is expected.

### Closed

A closed port indicates:

* The host was reachable
* No service was listening
* The operating system returned a rejection

Closed ports still reveal that a host exists.

### Filtered

A filtered port indicates:

* A firewall or network control prevented state determination
* The probe may have been dropped
* No response or an ICMP filtering response was observed

Filtered does not necessarily mean the service is absent.

---

## Process Analysis

For open ports, identify the listener:

```
Get-NetTCPConnection `
    -LocalPort <OPEN_PORT> `
    -State Listen
```

Then:

```
Get-Process -Id <PROCESS_ID>
```

Review:

* Executable
* Service
* User
* Expected purpose
* Binding address
* Exposure
* Firewall rule

---

## False-Positive Analysis

Legitimate multi-port activity may come from:

* Vulnerability scanners
* Monitoring systems
* Asset discovery
* Endpoint management
* Backup software
* Network troubleshooting
* Load balancers
* Health checks
* Application deployment
* Security validation

A detection should consider approved scanners and schedules without suppressing all scanning behavior.

---

# Detection Development

## Basic Multi-Port Detection

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| stats
    count as connection_events
    dc(dest_port) as unique_ports
    values(dest_port) as destination_ports
    min(_time) as first_seen
    max(_time) as last_seen
    by src_ip dest_ip
| where unique_ports >= <PORT_THRESHOLD>
| eval duration_seconds=last_seen-first_seen
| convert ctime(first_seen) ctime(last_seen)
```

---

## One Source Scanning Multiple Hosts

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_ip=coalesce(DestAddress, dest_ip, host)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| stats
    count as connection_events
    dc(dest_ip) as unique_hosts
    dc(dest_port) as unique_ports
    values(dest_ip) as destination_hosts
    values(dest_port) as destination_ports
    by src_ip
| where unique_hosts >= <HOST_THRESHOLD>
```

---

## Fast Sequential Port Activity

```spl
index=windows earliest=-5m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_ip=coalesce(DestAddress, dest_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| stats
    count as connection_events
    dc(dest_port) as unique_ports
    min(_time) as first_seen
    max(_time) as last_seen
    by src_ip dest_ip
| eval duration_seconds=last_seen-first_seen
| where unique_ports >= <PORT_THRESHOLD>
    AND duration_seconds <= <DURATION_THRESHOLD>
```

---

## Sensitive-Port Reconnaissance

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| eval dest_port=tonumber(coalesce(DestPort, dest_port))
| where dest_port IN (
    22,
    53,
    88,
    135,
    139,
    389,
    445,
    636,
    3389,
    5985,
    5986
)
| stats
    count as connection_events
    dc(dest_port) as unique_sensitive_ports
    values(dest_port) as sensitive_ports
    values(host) as destination_hosts
    by src_ip
| where unique_sensitive_ports >= <PORT_THRESHOLD>
```

Adjust ports to the actual environment.

A port list alone does not establish malicious intent.

---

## Approved Scanner Lookup

A mature implementation may maintain:

```
scanner_ip
scanner_name
approved_owner
approved_schedule
approved_targets
ticket_reference
```

Example search concept:

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval src_ip=coalesce(SourceAddress, src_ip)
| lookup approved_scanners scanner_ip as src_ip OUTPUT scanner_name approved_schedule
| where isnull(scanner_name)
| stats
    count
    dc(DestPort) as unique_ports
    values(DestPort) as destination_ports
    by src_ip host
| where unique_ports >= <PORT_THRESHOLD>
```

Approved scanners should not be ignored completely. Their behavior should still be monitored.

---

## Wazuh Rule Concept: Repeated Firewall Events

```xml
<group name="windows,firewall,reconnaissance,">
  <rule id="<CUSTOM_RULE_ID>" level="9" frequency="<EVENT_COUNT>" timeframe="<SECONDS>">
    <if_matched_sid><BASE_FIREWALL_EVENT_RULE_ID></if_matched_sid>
    <same_source_ip />
    <description>Repeated Windows Firewall activity from one source may indicate reconnaissance</description>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>
</group>
```

This is a conceptual example.

The deployed rule must be tested against normal background traffic and actual decoded fields.

---

## MITRE ATT&CK Context

This exercise relates primarily to:

```
T1046 – Network Service Discovery
```

The authorized exercise simulates discovery behavior but does not establish malicious intent.

---

## Detection Severity Guidance

| Pattern                                   | Suggested Context           |
| ----------------------------------------- | --------------------------- |
| Narrow scan from approved test host       | Informational or Low        |
| Known scanner scans approved target       | Low or Medium               |
| Unknown source scans several ports        | Medium                      |
| Unknown source scans many ports rapidly   | High                        |
| One source scans several hosts            | High                        |
| Administrative ports targeted             | Increased severity          |
| Reconnaissance follows account compromise | High                        |
| Scan followed by successful remote access | Critical review             |
| Scan crosses restricted network segments  | High                        |
| Evasive or spoofed scanning behavior      | High investigation priority |

Severity should reflect scope, source, speed, target sensitivity, and subsequent activity.

---

# Evidence Collection

## Evidence

Preserve:

* Authorized scope record
* KALI-TEST network configuration
* WIN11TARGET network configuration
* Approved port list
* Listener baseline
* Nmap command
* Nmap output
* Scan start and end time
* Firewall text-log entries
* Windows Filtering Platform events
* Wazuh results
* Splunk multi-port search
* Optional Sysmon events
* Optional packet capture
* Final listener and firewall state
* Investigation notes

---

## Hash the Nmap Result

```
sha256sum ex09-nmap-results.txt
```

---

## Export Windows Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\WIN11TARGET-Security.evtx"
```

Raw Security logs should remain private.

---

## Copy the Firewall Log

```
Copy-Item `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Destination "<EVIDENCE_PATH>\pfirewall-ex09.log"
```

---

## Export Optional Sysmon Events

```
wevtutil epl `
    "Microsoft-Windows-Sysmon/Operational" `
    "<EVIDENCE_PATH>\WIN11TARGET-Sysmon-Operational.evtx"
```

---

## Export Splunk Results

Export the narrow scan timeline as:

* CSV
* JSON
* Raw events

Review and sanitize every field before publication.

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
Exercise: EX-09
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

Before stopping temporary services or removing test rules:

1. Save the Nmap output.
2. Export relevant Windows events.
3. Copy the firewall log.
4. Save Wazuh results.
5. Save Splunk results.
6. Save optional Sysmon events.
7. Save the packet capture.
8. Record the target listener state.
9. Confirm the scan stayed within scope.

---

## Stop the Temporary Listener

Return to the listener terminal and press:

```
Ctrl+C
```

Confirm the port is no longer listening:

```
Get-NetTCPConnection `
    -LocalPort <OPEN_TEST_PORT> `
    -ErrorAction SilentlyContinue
```

Expected:

```
No active listener
```

---

## Remove Any Temporary Firewall Rule

```
Remove-NetFirewallRule `
    -DisplayName "<TEMPORARY_RULE_NAME>" `
    -ErrorAction SilentlyContinue
```

Confirm:

```
Get-NetFirewallRule `
    -DisplayName "<TEMPORARY_RULE_NAME>" `
    -ErrorAction SilentlyContinue
```

Expected:

```
No matching rule
```

---

## Confirm Firewall Health

```
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        DefaultInboundAction,
        DefaultOutboundAction,
        LogBlocked
```

All intended profiles should remain enabled.

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

Confirm both platforms remain healthy.

---

## Confirm No Scan Process Remains

On KALI-TEST:

```
pgrep -a nmap
```

Expected:

```
No active Nmap process
```

---

## Review Scan Scope

Confirm:

```
Only one authorized destination scanned:
Only approved ports scanned:
No subnet scan performed:
No service detection used:
No OS detection used:
No scripts used:
No external traffic generated:
```

---

## Final State Record

```
Authorized scope followed:
Nmap output preserved:
Windows events preserved:
Firewall log preserved:
Wazuh reviewed:
Splunk reviewed:
Packet capture preserved or not collected:
Temporary listener stopped:
Temporary firewall rule removed:
Firewall healthy:
Wazuh healthy:
Splunk healthy:
No Nmap process remains:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* The scope was documented before scanning.
* KALI-TEST and WIN11TARGET were on the host-only network.
* No bridged adapter was active.
* A narrow port list was used.
* A known open-port baseline was established.
* One controlled TCP connect scan was completed.
* The Nmap result was saved.
* Firewall telemetry was reviewed.
* Windows event telemetry was reviewed.
* Wazuh was reviewed.
* Splunk was reviewed.
* A multi-port pattern was evaluated.
* Optional Sysmon telemetry was reviewed where available.
* Optional packet capture was reviewed where collected.
* The temporary listener was stopped.
* Temporary firewall changes were removed.
* Final monitoring health was confirmed.
* Evidence was preserved.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                       | Result                       |
| ------------------------------ | ---------------------------- |
| Scope documented               | Pass / Fail                  |
| Host-only network confirmed    | Pass / Fail                  |
| Approved port list used        | Pass / Fail                  |
| Nmap output saved              | Pass / Fail                  |
| Open-port result validated     | Pass / Fail                  |
| Closed-port result validated   | Pass / Partial / Fail        |
| Filtered-port result validated | Pass / Partial / Not Tested  |
| Firewall text-log evidence     | Pass / Fail / Not Configured |
| Filtering Platform events      | Pass / Fail / Not Configured |
| Sysmon telemetry               | Pass / Fail / Not Installed  |
| Wazuh ingestion                | Pass / Fail                  |
| Wazuh correlation              | Pass / Needs Tuning / Absent |
| Splunk ingestion               | Pass / Fail                  |
| Splunk multi-port detection    | Pass / Needs Tuning / Fail   |
| Packet evidence                | Pass / Fail / Not Collected  |
| Timestamp accuracy             | Pass / Needs Review / Fail   |
| Cleanup                        | Complete / Incomplete        |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Source address
* Destination address
* Destination ports
* Connection result
* Allowed versus blocked status
* Application
* Process ID
* Scan duration
* Unique port count
* Correlation across events
* Approved-scanner context
* Threshold tuning
* Packet-level visibility
* Network-wide visibility
* Clear severity

---

## Improvements

Potential improvements include:

* Enable Windows Firewall blocked-packet logging
* Enable selective Filtering Platform auditing
* Ingest firewall text logs into Splunk
* Normalize source, destination, and port fields
* Add multi-port correlation
* Add multi-host correlation
* Maintain an approved-scanner lookup
* Add sensitive-port context
* Deploy Sysmon network telemetry
* Add Suricata
* Add Zeek
* Create a reconnaissance dashboard
* Create a network reconnaissance investigation runbook
* Track scan-to-authentication sequences

---

# Troubleshooting

## Nmap Reports the Host Is Down

Because the target is known and authorized, use:

```
nmap -Pn -sT -p <PORT_LIST> <WINDOWS_ENDPOINT>
```

Also check:

* Host-only adapter
* Target address
* VMware network
* Firewall
* Route
* VM power state

Do not scan the subnet to rediscover the target.

---

## All Ports Appear Filtered

Check:

* Firewall policy
* Active profile
* Target address
* Listener state
* Rule scope
* IPv4 versus IPv6
* VMware adapter
* Host reachability

Validate the known open port separately:

```
nc -vz -w 3 <WINDOWS_ENDPOINT> <OPEN_TEST_PORT>
```

---

## Known Open Port Appears Closed

Check:

* Listener still running
* Correct port
* Listener binding address
* Firewall rule
* Process crash
* Address family
* Target address

On WIN11TARGET:

```
Get-NetTCPConnection `
    -LocalPort <OPEN_TEST_PORT> `
    -State Listen
```

---

## Nmap Scans More Ports Than Planned

Stop the scan immediately.

Then:

1. Preserve the command history.
2. Confirm actual targets contacted.
3. Review packet capture or logs.
4. Confirm no external systems were reached.
5. Correct the command.
6. Document the mistake privately.
7. Repeat only after scope is revalidated.

---

## No Firewall Events Appear

Check:

* Firewall logging
* Filtering Platform auditing
* Event time range
* Correct endpoint
* Scan result
* Open versus blocked behavior
* Firewall profile
* Event IDs
* Security-log retention

Open connections may not generate blocked-packet events.

---

## No Sysmon Event Appears

Possible reasons include:

* Network Event ID `3` not enabled
* Connection blocked before establishment
* Inbound activity not captured as expected
* Sysmon filter excludes the process
* Event time range
* Source field mismatch

Sysmon is not a complete replacement for firewall or packet telemetry.

---

## Wazuh Shows Individual Events but No Scan Alert

This is a common finding.

Possible reasons include:

* No frequency rule
* No same-source correlation
* Events split across rule IDs
* Destination-port field not parsed
* Timeframe too short or long
* Rule threshold too high
* Low event volume

Document this as a detection-engineering opportunity.

---

## Splunk Search Does Not Identify Multiple Ports

Check:

* Destination port field name
* Numeric conversion
* Source-address field
* Time range
* Duplicate events
* Event types
* Index
* Host
* Raw event format

Start with:

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| table _time host EventCode SourceAddress DestAddress DestPort Message
```

---

## Packet Capture Does Not Show the Scan

Check:

* Correct capture interface
* Capture started first
* Correct BPF filter
* IPv4 versus IPv6
* Target address
* Nmap command
* Capture permissions

Run:

```
ip address
```

to confirm the interface.

---

## Scan Results and Windows Events Do Not Match

Possible causes include:

* One Nmap probe creates multiple events
* Closed ports do not generate the expected event type
* Allowed-event auditing disabled
* Firewall text logging disabled
* Sysmon sees only successful connections
* Event ingestion delay
* Duplicate data sources
* IPv6 traffic
* Field extraction problems

Use packet evidence as the network-level reference when available.

---

## Unexpected External Traffic Appears

Stop the exercise immediately.

Then:

1. Stop Nmap.
2. Preserve command output.
3. Review routing and adapter state.
4. Confirm no bridged adapter exists.
5. Review the exact command.
6. Inspect packet capture.
7. Document the incident privately.
8. Do not resume until the scope boundary is restored.

---

# Public Sanitization

Remove or replace:

* Operational IP addresses
* MAC addresses
* Real hostnames
* Real usernames
* Internal domain names
* Exact service inventory
* Sensitive open ports
* Firewall rule identifiers
* Process identifiers
* Security identifiers
* Wazuh agent identifiers
* Splunk internal URLs
* Raw packet captures
* Full Nmap output tied to operational systems
* Evidence storage paths
* Physical-network details

Use placeholders such as:

```
<SOURCE_SYSTEM>
<SOURCE_IP>
<DESTINATION_SYSTEM>
<DESTINATION_IP>
<PORT_LIST>
<OPEN_PORT>
<CLOSED_PORT>
<FILTERED_PORT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Nmap narrow-scan result
* Known open listener
* Windows firewall event
* Wazuh firewall telemetry
* Splunk multi-port detection
* Packet-level SYN sequence
* Port-state comparison table
* Detection-quality assessment

Before publishing:

1. Crop unrelated information.
2. Remove operational IP addresses.
3. Remove MAC addresses.
4. Remove real usernames.
5. Remove unrelated open ports.
6. Remove SIDs and process IDs where unnecessary.
7. Hide browser tabs and bookmarks.
8. Permanently redact sensitive values.
9. Reopen and inspect the final image.

Do not upload the raw packet capture.

---

# Exercise Report Template

## Executive Summary

```
A narrow, authorized TCP connect scan was performed from KALI-TEST against WIN11TARGET during CyberLab Exercise EX-09. The scan was restricted to a documented port list on the host-only network. Windows Firewall, optional Sysmon, packet, Wazuh, and Splunk telemetry were reviewed to determine whether the activity could be identified as network service discovery.
```

## Findings

```
Source system:

Destination system:

Scan command:

Approved port list:

Scan start:

Scan end:

Open ports:

Closed ports:

Filtered ports:

Firewall log result:

Windows event result:

Sysmon result:

Packet capture result:

Wazuh result:

Splunk result:

Unique ports detected:

Scan duration:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized narrow network reconnaissance performed as part of CyberLab Exercise EX-09.
```

## Resolution

```
The scan completed within the approved scope, all evidence was preserved, temporary listeners and firewall rules were removed, and the CyberLab network and monitoring systems were returned to their intended state.
```

## Recommendations

```
- Normalize firewall network fields.
- Correlate one source targeting multiple ports.
- Maintain an approved-scanner lookup.
- Add sensitive-port context.
- Deploy network IDS telemetry.
- Correlate reconnaissance with later authentication or exploitation.
- Create a network reconnaissance investigation runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshots exist
* Scan authorization documented
* Source system confirmed
* Destination system confirmed
* Host-only network confirmed
* No bridged adapter active
* Approved port list documented
* Firewall enabled
* Firewall logging reviewed
* Filtering Platform auditing reviewed
* Wazuh healthy
* Splunk healthy
* Optional Sysmon reviewed
* System time validated
* Start time recorded

## Baseline

* Known open port established
* Open listener confirmed
* Closed port confirmed
* Optional filtered port configured
* Baseline connection validated
* Listener process recorded

## Exercise

* Exact command reviewed
* TCP connect scan used
* One destination scanned
* Only approved ports scanned
* No service detection used
* No OS detection used
* No scripts used
* Scan output saved
* Start and end times recorded
* No unnecessary repeat scan performed

## Investigation

* Nmap results reviewed
* Firewall text log reviewed
* Filtering Platform events reviewed
* Permitted events reviewed where enabled
* Source address identified
* Destination ports identified
* Open, closed, and filtered states compared
* Optional Sysmon reviewed
* Optional packet capture reviewed
* Wazuh reviewed
* Splunk reviewed
* Multi-port pattern evaluated
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Nmap process stopped
* Temporary listener stopped
* Temporary firewall rule removed
* Firewall healthy
* Wazuh healthy
* Splunk healthy
* Scope reviewed
* No external target contacted
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Authorized Nmap use
* TCP port-state interpretation
* Windows Defender Firewall analysis
* Windows Filtering Platform analysis
* Optional Sysmon network analysis
* Packet capture analysis
* Wazuh event validation
* Splunk SPL
* Multi-event correlation
* Threshold development
* False-positive analysis
* Scope control
* Evidence preservation
* Detection-gap assessment
* Public sanitization

---

# Lessons Learned

* Reconnaissance testing must begin with a narrowly documented scope.
* A known open-port baseline makes scan results easier to validate.
* Open, closed, and filtered ports produce different network behavior.
* Firewall logs, Windows events, Sysmon, and packet captures provide different views.
* Sysmon may capture successful connections but not blocked probes.
* Wazuh may ingest individual firewall events without identifying the broader scan.
* Splunk can correlate one source targeting multiple destination ports.
* A small authorized scan can resemble malicious reconnaissance without context.
* Approved-scanner allowlists should add context rather than suppress all activity.
* Port count, host count, timing, target sensitivity, and subsequent behavior affect severity.
* Packet capture provides the clearest view of TCP response behavior.
* Network IDS telemetry would improve scan detection beyond endpoint-only logging.
* Cleanup must remove temporary listeners and firewall rules without weakening monitoring.

---

# Summary

This exercise validates the CyberLab’s ability to observe and investigate controlled network reconnaissance.

A complete investigation should identify:

* The source system
* The destination system
* The approved scan scope
* The ports tested
* The open, closed, and filtered results
* The Windows Firewall evidence
* The Windows event evidence
* The optional Sysmon evidence
* The packet behavior
* The Wazuh result
* The Splunk result
* The multi-port pattern
* The event timeline
* Missing telemetry
* Appropriate detection severity

The exercise is complete only after the scan remains within scope, evidence is preserved, temporary changes are removed, monitoring remains healthy, and detection improvements are documented.
