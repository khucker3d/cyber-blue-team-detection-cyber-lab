# Network Reconnaissance Detection Runbook

## Runbook Overview

This runbook provides a structured process for investigating suspected network reconnaissance activity in the CyberLab.

It applies to alerts involving:

* Port scanning
* Service discovery
* Host discovery
* Repeated connection attempts across multiple ports
* One source contacting multiple systems
* ICMP or TCP-based discovery
* Nmap or similar scanning behavior
* Windows Firewall blocks associated with scanning
* Sysmon network telemetry
* Wazuh or Splunk reconnaissance alerts
* Approved CyberLab validation exercises

The runbook helps the analyst determine:

* Which system initiated the activity
* Which systems were targeted
* Which ports and protocols were tested
* Whether the source was authorized
* Whether the activity was limited to discovery
* Whether successful connections followed
* Whether authentication attempts, exploitation, or lateral movement occurred
* Whether the activity represents approved testing, troubleshooting, inventory collection, malware behavior, or unauthorized reconnaissance
* Whether containment or escalation is required

---

## Runbook ID

```text
RB-09
```

---

## Related Exercise

```text
EX-09 – Network Reconnaissance Detection
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Network security analysts
* Windows administrators
* Detection engineers
* Incident responders
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Internal CyberLab reconnaissance
* Host-only VMware networks
* TCP connection scanning
* Limited UDP discovery
* ICMP discovery
* Service enumeration
* Windows Defender Firewall events
* Sysmon network events
* Packet captures
* Wazuh alerts
* Splunk detections

This runbook does not authorize:

* Scanning systems outside the CyberLab
* Scanning household, employer, school, or internet systems without authorization
* Expanding the scan scope during investigation
* Exploiting discovered services
* Disabling firewall or monitoring controls
* Publishing internal addressing or exposed-service details
* Assuming every multi-port connection pattern is malicious

---

## Primary Data Sources

Use the available sources in this order:

1. Original reconnaissance alert
2. Windows Defender Firewall events
3. Windows Firewall text logs
4. Sysmon network events
5. Packet capture
6. Wazuh events and alerts
7. Splunk indexed telemetry
8. Source-system process telemetry
9. Target-system service state
10. Authentication and account logs
11. Change, maintenance, or exercise records
12. User or administrator confirmation

---

## Key Windows and Sysmon Event IDs

| Event ID | Source   | Description                                                 |
| -------: | -------- | ----------------------------------------------------------- |
|     5152 | Security | Windows Filtering Platform blocked a packet                 |
|     5153 | Security | A restrictive filter blocked a packet                       |
|     5156 | Security | Windows Filtering Platform permitted a connection           |
|     5157 | Security | Windows Filtering Platform blocked a connection             |
|     5158 | Security | Windows Filtering Platform permitted a bind to a local port |
|     5159 | Security | Windows Filtering Platform blocked a bind to a local port   |
|        1 | Sysmon   | Process creation                                            |
|        3 | Sysmon   | Network connection                                          |
|       22 | Sysmon   | DNS query                                                   |
|     4624 | Security | Successful logon                                            |
|     4625 | Security | Failed logon                                                |
|     4648 | Security | Explicit credentials used                                   |
|     4672 | Security | Special privileges assigned                                 |

Event availability depends on audit policy, Sysmon configuration, firewall logging, and the scan method.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh reconnaissance alert
* Splunk correlation search
* Windows Firewall block alert
* Sysmon network event
* Packet-capture review
* Uptime or availability anomaly
* Administrator observation
* User report
* CyberLab exercise

---

## Minimum Alert Information

Record:

```text
Alert time:
Alert source:
Alert name:
Source system:
Source address:
Destination systems:
Destination addresses:
Destination ports:
Protocol:
Attempt count:
Unique target count:
Time window:
Reporting host:
Associated process:
Alert severity:
Analyst:
```

Do not assume the alert has correctly identified the source and destination roles.

---

## Initial Analyst Questions

Determine immediately:

* Is the source inside the CyberLab?
* Is the source authorized to perform scanning?
* Is there an approved exercise or maintenance record?
* How many destination systems were contacted?
* How many ports were tested?
* Did the scan target one host or many hosts?
* Were connections blocked, allowed, or mixed?
* Did any authentication attempts follow?
* Did exploitation or service-specific activity follow?
* Is the activity still occurring?
* Is the source system otherwise behaving suspiciously?
* Are multiple endpoints reporting the same source?

---

# Severity Classification

## Informational

Use informational severity when:

* The activity is part of an approved CyberLab exercise
* The source is an authorized testing system
* The target scope is documented
* Only approved ports and systems were tested
* No exploitation or authentication activity followed

---

## Low

Use low severity when:

* A known management or inventory tool contacts several ports
* The source is expected
* The activity matches documented administration
* No unusual process or follow-on activity exists
* The traffic remains within an approved scope

---

## Medium

Use medium severity when:

* Authorization is not immediately confirmed
* One source contacts many ports on one host
* One source contacts several internal hosts
* The source is unusual
* The activity occurs outside expected hours
* Firewall blocks are repeated
* The process responsible is unknown

---

## High

Use high severity when:

* The source is unauthorized
* A user workstation performs broad scanning
* Reconnaissance is followed by failed authentication
* Administrative services are targeted
* Multiple systems are scanned
* The activity includes service enumeration
* The source launches unusual tools or scripts
* The activity resembles preparation for lateral movement

---

## Critical

Use critical severity when:

* Reconnaissance is followed by exploitation
* Privileged authentication succeeds
* Multiple systems are compromised
* Domain controllers or security infrastructure are targeted
* The source is a confirmed compromised endpoint
* Reconnaissance is part of active lateral movement
* Security controls are disabled or bypassed

---

# Initial Triage

## Step 1: Confirm the Alert Time

Record the exact timestamp and timezone from:

* Source endpoint
* Target endpoint
* Wazuh
* Splunk
* Packet capture
* Analyst workstation

Use absolute timestamps.

```text
YYYY-MM-DD HH:MM:SS Timezone
```

---

## Step 2: Validate the Source and Destination Roles

Confirm:

* Source hostname
* Source address
* Destination hostname
* Destination address
* VMware adapter used
* Network segment
* Whether NAT or bridged networking was active
* Whether the address belonged to the expected system at the time

### Windows

```powershell
hostname
```

```powershell
Get-NetIPConfiguration
```

### Linux

```bash
hostname
```

```bash
ip address
```

```bash
ip route
```

Do not rely on an IP address alone when DHCP, snapshots, or cloned VMs may be involved.

---

## Step 3: Confirm the Event Exists Locally

On the Windows target, run from an authorized administrator session:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5156, 5157
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Filter for the suspected source address:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5156, 5157
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

If no matching events exist, review:

* Audit policy
* Firewall text logging
* Sysmon network telemetry
* Wrong target system
* Wrong time range
* Traffic that did not reach the endpoint
* Allowed traffic without corresponding auditing

---

## Step 4: Review the Windows Firewall Text Log

Determine the configured log path:

```powershell
Get-NetFirewallProfile |
    Select-Object `
        Name,
        Enabled,
        LogFileName,
        LogBlocked,
        LogAllowed,
        LogMaxSizeKilobytes
```

A common default path is:

```text
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
```

Review recent entries:

```powershell
Get-Content `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Tail 200
```

Search for the source address:

```powershell
Select-String `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Pattern "<SOURCE_IP>"
```

Firewall text logs may not provide process attribution.

---

## Step 5: Establish the Initial Scope

Record:

```text
First observed:
Last observed:
Source system:
Source address:
Destination systems:
Destination addresses:
Unique destination ports:
Protocols:
Blocked attempts:
Allowed attempts:
Total attempts:
Associated process:
Authorization status:
```

---

# Pattern Analysis

## One Source, Many Ports

This pattern may indicate a vertical port scan.

Example:

```text
One source address
One destination address
Many destination ports
Short time window
```

Possible causes include:

* Nmap scan
* Vulnerability scanner
* Service inventory tool
* Troubleshooting
* Malware service discovery

---

## One Source, Many Hosts

This pattern may indicate horizontal host or service discovery.

Example:

```text
One source address
Many destination addresses
Same destination port
Short time window
```

Possible causes include:

* Searching for SMB
* Searching for RDP
* Searching for SSH
* Asset inventory
* Worm-like propagation
* Lateral movement preparation

---

## Many Sources, One Host

Possible explanations include:

* Distributed scanning
* Several approved test systems
* Background network traffic
* Misconfigured automation
* One highly visible service
* Multiple compromised systems

---

## Sequential Port Pattern

A sequence such as:

```text
20
21
22
23
24
25
```

may suggest systematic scanning.

However, tools may:

* Randomize ports
* Use common-port lists
* Retry ports
* Interleave targets
* Scan ports in parallel

Do not require strict numerical order.

---

## Common Service Targets

| Port | Typical Service       | Investigation Context             |
| ---: | --------------------- | --------------------------------- |
|   22 | SSH                   | Linux administration              |
|   53 | DNS                   | DNS service or discovery          |
|   80 | HTTP                  | Web service                       |
|  135 | RPC                   | Windows service discovery         |
|  139 | NetBIOS               | Windows networking                |
|  443 | HTTPS                 | Web or management service         |
|  445 | SMB                   | File sharing and lateral movement |
| 3389 | RDP                   | Remote administration             |
| 5985 | WinRM HTTP            | PowerShell remoting               |
| 5986 | WinRM HTTPS           | Secure PowerShell remoting        |
| 8000 | Alternate web service | Application-specific              |
| 8080 | Alternate HTTP        | Application-specific              |

A port number alone does not prove the exact service.

---

# Source-System Investigation

## Confirm the User

### Windows Source

```powershell
quser
```

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, UserName
```

### Linux Source

```bash
who
```

```bash
w
```

Record:

```text
Logged-on user:
Account privilege:
Session type:
Login time:
Source system owner:
```

---

## Review Running Processes

### Windows

```powershell
Get-CimInstance Win32_Process |
    Select-Object `
        ProcessId,
        ParentProcessId,
        Name,
        ExecutablePath,
        CommandLine,
        CreationDate
```

### Linux

```bash
ps auxww
```

Look for:

* `nmap`
* `masscan`
* PowerShell
* Python
* Bash
* Network inventory agents
* Vulnerability scanners
* Unknown binaries
* Recently launched terminal processes

---

## Search for Nmap

### Linux

```bash
ps auxww | grep -i nmap
```

Review shell history only when authorized:

```bash
history | grep -i nmap
```

Shell history may be incomplete, modified, or unavailable.

### Windows

```powershell
Get-CimInstance Win32_Process |
    Where-Object {
        $_.CommandLine -match "nmap"
    } |
Select-Object `
    ProcessId,
    ParentProcessId,
    Name,
    CommandLine,
    CreationDate
```

---

## Review Process Creation

### Sysmon on Windows

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "nmap|masscan|powershell|python|port"
}
```

### Windows Security

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "nmap|masscan|powershell|python"
}
```

---

## Record Source Process Context

```text
Process:
Executable path:
Command line:
Process ID:
Parent process:
Parent process ID:
User:
Integrity level:
Start time:
Hash:
Signature:
```

---

## Review Source Network Connections

### Windows

```powershell
Get-NetTCPConnection |
    Sort-Object RemoteAddress, RemotePort
```

Filter by process:

```powershell
Get-NetTCPConnection `
    -OwningProcess <PROCESS_ID> `
    -ErrorAction SilentlyContinue
```

### Linux

```bash
sudo ss -tunap
```

Historical connections may require Sysmon, packet capture, firewall logs, or SIEM data.

---

# Target-System Investigation

## Review Listening Ports

Run on the target:

```powershell
Get-NetTCPConnection -State Listen |
    Sort-Object LocalPort |
    Select-Object `
        LocalAddress,
        LocalPort,
        OwningProcess
```

Map the process:

```powershell
Get-Process -Id <PROCESS_ID>
```

Alternative:

```powershell
Get-NetTCPConnection -State Listen |
ForEach-Object {
    [PSCustomObject]@{
        LocalAddress = $_.LocalAddress
        LocalPort = $_.LocalPort
        ProcessId = $_.OwningProcess
        ProcessName = (
            Get-Process `
                -Id $_.OwningProcess `
                -ErrorAction SilentlyContinue
        ).ProcessName
    }
}
```

---

## Confirm Expected Services

Record:

```text
Port:
Protocol:
Service:
Process:
Expected:
Required:
Firewall rule:
Owner:
```

An open port may be legitimate but unnecessary.

A closed port may still generate a reconnaissance signal.

---

## Review Firewall Rules

Search by port:

```powershell
Get-NetFirewallPortFilter |
    Where-Object {
        $_.LocalPort -eq "<PORT>"
    }
```

Review enabled inbound rules:

```powershell
Get-NetFirewallRule `
    -Enabled True `
    -Direction Inbound |
Select-Object `
    DisplayName,
    Action,
    Profile,
    Enabled
```

Do not publish the complete operational firewall rule set.

---

## Review Successful Connections

Search Event ID `5156` when enabled:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5156
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

A successful TCP connection does not prove authentication or exploitation.

---

# Packet-Capture Investigation

## Use Existing Capture Evidence

Prefer an existing packet capture collected during the event.

Review:

* Source address
* Destination address
* Protocol
* SYN packets
* SYN-ACK responses
* Reset responses
* ICMP unreachable responses
* Connection timing
* Retransmissions
* Service banners where captured

---

## Safe Capture on Linux

Run only within the authorized CyberLab:

```bash
sudo tcpdump \
    -i <INTERFACE> \
    -nn \
    host <SOURCE_IP> \
    -w reconnaissance-investigation.pcap
```

Use a narrow source, destination, or port filter.

Stop the capture after the required window.

---

## Wireshark Display Filters

Traffic from the suspected source:

```text
ip.src == <SOURCE_IP>
```

Traffic between source and target:

```text
ip.addr == <SOURCE_IP> && ip.addr == <TARGET_IP>
```

TCP SYN packets without ACK:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Specific destination port:

```text
tcp.dstport == <PORT>
```

---

## Packet Interpretation

### SYN Followed by SYN-ACK

Likely indicates:

* Port reachable
* TCP listener present
* Firewall permitted initial connection

### SYN Followed by RST

Likely indicates:

* Host reachable
* Port closed
* No listening service

### Repeated SYN with No Response

Possible explanations include:

* Firewall filtering
* Packet loss
* Host offline
* Routing issue
* Silent drop policy

Packet behavior should be correlated with endpoint logs.

---

# Authentication Correlation

## Search Failed Logons

On Windows targets:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

---

## Search Successful Logons

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

---

## Search Explicit Credential Use

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4648
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<SOURCE_SYSTEM>|<TARGET_ACCOUNT>"
}
```

---

## Authentication Indicators of Concern

Increase severity when reconnaissance is followed by:

* Repeated failed logons
* Successful RDP logon
* Successful SMB authentication
* PowerShell remoting
* Explicit credential use
* Privileged logon
* Account lockout
* New account creation
* Privileged group changes

Transition to the relevant identity runbook when needed.

---

# Service-Specific Follow-On Review

## SMB

Review:

* Event ID `4624` with Logon Type `3`
* Event ID `4625`
* Share access events where configured
* File creation
* Remote service creation
* Administrative share access

---

## RDP

Review:

* Event ID `4624` with Logon Type `10`
* Event ID `4625`
* Terminal Services logs
* Source address
* Session creation
* Privileged activity after logon

---

## WinRM and PowerShell Remoting

Review:

* PowerShell Operational logs
* WinRM Operational logs
* Event ID `4648`
* Remote PowerShell process creation
* Network connections to WinRM ports

---

## SSH

Review Linux authentication logs:

```bash
sudo journalctl -u ssh
```

Or where applicable:

```bash
sudo grep -i ssh /var/log/auth.log
```

Review:

* Source address
* Username
* Authentication failures
* Successful sessions
* Commands executed

---

## Web Services

Review:

* Web access logs
* Requested paths
* User agents
* Authentication attempts
* Error codes
* Unusual methods
* File upload attempts

Simple port scanning may produce only a TCP connection without an HTTP request.

---

# Wazuh Investigation

## Search by Source Address

Search for:

```text
<SOURCE_IP>
```

---

## Search by Destination

Search for:

```text
<TARGET_IP>
<TARGET_HOST>
```

---

## Search by Port

Search for:

```text
<DESTINATION_PORT>
```

---

## Search by Process or Tool

Search for:

```text
nmap
masscan
powershell
python
```

---

## Review Wazuh Fields

Record:

```text
Agent:
Rule ID:
Rule description:
Rule level:
Source address:
Source port:
Destination address:
Destination port:
Protocol:
Action:
Process:
User:
Event count:
First observed:
Last observed:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Was the source address parsed correctly?
* Were source and destination roles correct?
* Were blocked and allowed connections distinguished?
* Were repeated ports correlated?
* Were multiple targets correlated?
* Was process attribution available?
* Did authentication activity follow?
* Was the alert threshold appropriate?
* Were approved scanners excluded through context rather than broad suppression?
* Was event ingestion timely?

---

# Splunk Investigation

## Basic Firewall Search

```spl
index=windows earliest=-24h
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| search "<SOURCE_IP>"
| table
    _time
    host
    EventCode
    SourceAddress
    SourcePort
    DestAddress
    DestPort
    Protocol
    Application
    Direction
    Message
| sort _time
```

Field names depend on the Windows add-on and source configuration.

---

## One Source Scanning Many Ports

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval source_ip=coalesce(SourceAddress, src_ip)
| eval destination_ip=coalesce(DestAddress, dest_ip)
| eval destination_port=coalesce(DestPort, dest_port)
| stats
    count as connection_events
    dc(destination_port) as unique_ports
    values(destination_port) as destination_ports
    values(EventCode) as event_codes
    min(_time) as first_seen
    max(_time) as last_seen
    by source_ip destination_ip
| where unique_ports >= <PORT_THRESHOLD>
| convert ctime(first_seen) ctime(last_seen)
| sort - unique_ports
```

---

## One Source Scanning Many Hosts

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval source_ip=coalesce(SourceAddress, src_ip)
| eval destination_ip=coalesce(DestAddress, dest_ip)
| eval destination_port=coalesce(DestPort, dest_port)
| stats
    count as connection_events
    dc(destination_ip) as unique_targets
    values(destination_ip) as destination_ips
    values(destination_port) as destination_ports
    by source_ip
| where unique_targets >= <HOST_THRESHOLD>
| sort - unique_targets
```

---

## One Source Scanning One Port Across Many Hosts

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval source_ip=coalesce(SourceAddress, src_ip)
| eval destination_ip=coalesce(DestAddress, dest_ip)
| eval destination_port=coalesce(DestPort, dest_port)
| stats
    count as attempts
    dc(destination_ip) as unique_targets
    values(destination_ip) as targets
    by source_ip destination_port
| where unique_targets >= <HOST_THRESHOLD>
| sort - unique_targets
```

---

## Mixed Allowed and Blocked Connections

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval source_ip=coalesce(SourceAddress, src_ip)
| eval destination_ip=coalesce(DestAddress, dest_ip)
| eval destination_port=coalesce(DestPort, dest_port)
| eval result=case(
    EventCode=5156, "allowed",
    EventCode=5152 OR EventCode=5153 OR EventCode=5157, "blocked",
    true(), "other"
)
| stats
    count(eval(result="allowed")) as allowed
    count(eval(result="blocked")) as blocked
    dc(destination_port) as unique_ports
    values(destination_port) as ports
    by source_ip destination_ip
| where allowed > 0 AND blocked > 0
| sort - unique_ports
```

---

## Correlate Reconnaissance with Authentication

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157 OR
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4648
)
| search "<SOURCE_IP>"
| eval activity=case(
    EventCode=5152 OR EventCode=5153 OR EventCode=5157, "Blocked connection",
    EventCode=5156, "Allowed connection",
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4648, "Explicit credential use",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    SourceAddress
    DestAddress
    DestPort
    TargetUserName
    LogonType
    ProcessName
    Message
| sort _time
```

---

## Correlate Source Process and Network Activity

```spl
index=windows earliest=-30m
(
    EventCode=1 OR
    EventCode=3 OR
    EventCode=4688
)
| search
    "nmap"
    OR "masscan"
    OR "<SOURCE_PROCESS>"
| table
    _time
    host
    EventCode
    User
    ParentImage
    Image
    CommandLine
    SourceIp
    SourcePort
    DestinationIp
    DestinationPort
    ProcessId
    ProcessGuid
| sort _time
```

---

## Build a Reconnaissance Timeline

```spl
index=windows earliest=-30m
(
    EventCode=1 OR
    EventCode=3 OR
    EventCode=22 OR
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4648 OR
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| search
    "<SOURCE_IP>"
    OR "<SOURCE_PROCESS>"
| eval activity=case(
    EventCode=1 OR EventCode=4688, "Process creation",
    EventCode=3, "Network connection",
    EventCode=22, "DNS query",
    EventCode=5152 OR EventCode=5153 OR EventCode=5157, "Firewall block",
    EventCode=5156, "Firewall allow",
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4648, "Explicit credential use",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    User
    Image
    CommandLine
    SourceAddress
    DestAddress
    DestPort
    DestinationIp
    DestinationPort
    TargetUserName
    LogonType
| sort _time
```

---

# Investigation Decision Tree

```text
Network Reconnaissance Alert
            |
            v
Is the source inside the authorized environment?
            |
       +----+----+
       |         |
      No        Yes
       |         |
Escalate and   Is the source an
validate edge  approved scanner?
telemetry            |
                +----+----+
                |         |
               Yes        No
                |         |
        Validate scope,   Increase severity
        time and targets        |
                                v
                    Was activity limited
                      to discovery?
                                |
                           +----+----+
                           |         |
                          Yes        No
                           |         |
                  Review process,   Review authentication,
                  purpose and       exploitation, privilege
                  configuration     and lateral movement
                           |
                           v
               Were any successful connections
                 or authentications observed?
                           |
                      +----+----+
                      |         |
                     Yes        No
                      |         |
               Investigate      Classify, document
               follow-on use    and tune if appropriate
```

---

# Common Investigation Scenarios

## Scenario 1: Approved Nmap Exercise

Indicators:

* Approved exercise record
* Dedicated Kali testing system
* Narrow target scope
* Documented test ports
* Expected timestamps
* No exploitation or unauthorized authentication

Response:

* Validate source, target, ports, and timing
* Confirm Wazuh and Splunk detections
* Preserve sanitized evidence
* Classify as authorized test activity

---

## Scenario 2: Administrative Port Check

Indicators:

* Known administrator
* One target
* Small set of ports
* Troubleshooting or deployment context
* Expected tool or command
* No broad scan behavior

Response:

* Confirm administrative purpose
* Record the tested ports
* Close as authorized activity when appropriate

---

## Scenario 3: Vulnerability Scanner

Indicators:

* Known scanner source
* Broad but approved scope
* Scheduled time
* Scanner inventory entry
* Repeated service-specific probes
* Change or assessment record

Response:

* Validate scanner identity
* Confirm target scope
* Confirm scan window
* Avoid broadly suppressing all scanner traffic
* Maintain approved-scanner context in detections

---

## Scenario 4: User Workstation Scanning Internal Hosts

Indicators:

* Standard workstation
* User does not perform security testing
* Multiple targets or ports
* Unknown scanning process
* No approved record

Response:

* Increase severity
* Investigate source process and user
* Review downloads and PowerShell activity
* Contain the endpoint if activity is unauthorized
* Review follow-on authentication

---

## Scenario 5: Malware Service Discovery

Indicators:

* Unknown or suspicious process
* Scanning begins after a suspicious file or Defender alert
* Common administrative ports targeted
* Multiple hosts contacted
* Authentication or remote execution follows

Response:

* Treat as high severity
* Isolate the source
* Preserve process and network evidence
* Investigate malware, persistence, and credentials
* Review all contacted systems

---

## Scenario 6: Reconnaissance Followed by Failed Logons

Indicators:

* Port scanning
* RDP, SMB, SSH, or WinRM identified
* Failed authentication follows
* Same source address
* Multiple usernames may be tested

Response:

* Escalate
* Transition to the failed logon runbook
* Review spraying or brute-force patterns
* Contain unauthorized source

---

## Scenario 7: Reconnaissance Followed by Successful Logon

Indicators:

* Scanning identifies an open service
* Successful authentication follows
* Source and destination match
* Privileged or remote logon type
* Additional activity occurs afterward

Response:

* Treat as high or critical
* Isolate affected systems
* Review session activity
* Protect affected accounts
* Investigate lateral movement

---

## Scenario 8: Monitoring or Inventory Agent

Indicators:

* Known management agent
* Regular schedule
* Consistent destination list
* Documented service-discovery function
* Signed executable
* No suspicious follow-on activity

Response:

* Confirm owner and purpose
* Tune detection with specific process, source, and schedule context
* Do not suppress all multi-port behavior

---

# Containment

## When Containment Is Not Required

Containment may not be required when:

* The source is authorized
* Scope and timing are approved
* The activity is limited to documented discovery
* No authentication, exploitation, or persistence follows
* The event is part of an approved CyberLab exercise

Document why containment was unnecessary.

---

## Source Endpoint Containment

Depending on severity:

* Disconnect the source VM’s network adapter
* Disable the source network adapter
* Isolate through an endpoint security platform
* Stop the scanning process
* End the user session
* Restrict the source to an isolated network

Preserve volatile evidence before terminating the process when possible.

---

## Stop the Scanning Process

### Windows

```powershell
Stop-Process `
    -Id <PROCESS_ID> `
    -Force
```

### Linux

```bash
sudo kill <PROCESS_ID>
```

Use forceful termination only when necessary:

```bash
sudo kill -9 <PROCESS_ID>
```

Record:

```text
Process:
Process ID:
Stop time:
Administrator:
Reason:
```

---

## Account Containment

With appropriate authorization:

* Disable or restrict the responsible account
* Reset credentials
* Revoke active sessions
* Remove unauthorized privilege
* Restrict remote access

---

## Network Containment

* Block the source address
* Restrict access to targeted administrative ports
* Limit management services to approved hosts
* Disable unnecessary listeners
* Segment the source system
* Apply a temporary narrow firewall rule

Do not create broad deny rules without reviewing operational impact.

---

# Eradication

## Unauthorized Scanning Tool

* Preserve tool path, hash, and command line
* Remove unauthorized software
* Review installation source
* Review related scripts
* Review scheduled tasks and services
* Review user profile and downloads
* Validate that the tool is not recreated

---

## Malicious Process

* Isolate the endpoint
* Preserve executable and memory evidence where appropriate
* Remove malicious files
* Remove persistence
* Restore security controls
* Review credentials and sessions
* Scan other affected systems

---

## Misconfigured Management Tool

* Correct the scan scope
* Correct destination lists
* Reduce unnecessary ports
* Set an approved schedule
* Document ownership
* Validate behavior before re-enabling

---

## Unnecessary Exposed Service

* Confirm service ownership
* Stop or disable the service where appropriate
* Remove unnecessary firewall rule
* Restrict the service to approved hosts
* Validate application impact
* Document the change

---

# Recovery

## Confirm Scanning Has Stopped

### Windows Firewall Events

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5156, 5157
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

### Splunk

```spl
index=windows earliest=-30m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| search "<SOURCE_IP>"
| timechart span=5m count
```

---

## Confirm Source Process State

### Windows

```powershell
Get-CimInstance Win32_Process |
    Where-Object {
        $_.ProcessId -eq <PROCESS_ID>
    }
```

### Linux

```bash
ps -p <PROCESS_ID>
```

---

## Confirm Target Services

Review expected listening services:

```powershell
Get-NetTCPConnection -State Listen |
    Sort-Object LocalPort
```

Confirm:

* Required services remain available
* Unnecessary services are disabled
* Firewall rules match intended access
* No unauthorized listener exists

---

## Confirm Authentication State

Review:

* Failed logons stopped
* No unexplained successful logons
* No account lockouts
* No privilege changes
* No new accounts
* No suspicious remote sessions

---

## Confirm Monitoring Health

Verify:

* Wazuh agents active
* Splunk forwarders active
* Firewall logging enabled
* Sysmon operational where installed
* Time synchronization healthy
* New validation traffic is searchable

Do not repeat the full suspicious scan as a recovery test.

Use a documented narrow connection check where validation is required.

---

# Escalation Criteria

Escalate immediately when:

* The source is unauthorized
* A user workstation scans many systems
* Administrative services are targeted
* Authentication attempts follow
* A successful remote logon follows
* The scan is performed by a suspicious process
* Multiple endpoints are affected
* Domain controllers or security systems are targeted
* Malware or Defender alerts are associated
* The source creates persistence
* The activity continues after containment
* The analyst cannot determine scope

---

## Escalation Package

Provide:

```text
Incident summary:
Severity:
First observed:
Last observed:
Source system:
Source address:
Source user:
Source process:
Source command line:
Destination systems:
Destination addresses:
Unique ports:
Protocols:
Blocked connections:
Allowed connections:
Authentication attempts:
Successful authentication:
Privileged access:
Follow-on activity:
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
* Source and destination identity
* Firewall events
* Firewall text-log entries
* Sysmon network events
* Source process creation event
* Source command line
* Process hash and signature
* Packet capture where available
* Target listening-port state
* Authentication events
* Wazuh results
* Splunk timeline
* Containment and recovery actions
* Analyst notes

---

## Export Windows Security Events

```powershell
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

---

## Export Sysmon Events

```powershell
wevtutil epl `
    "Microsoft-Windows-Sysmon/Operational" `
    "<EVIDENCE_PATH>\Sysmon-Operational.evtx"
```

---

## Preserve Firewall Log

```powershell
Copy-Item `
    -Path "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" `
    -Destination "<EVIDENCE_PATH>\pfirewall.log" `
    -Force
```

---

## Export Narrow Firewall Events

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5156, 5157
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\reconnaissance-firewall-events.csv" `
    -NoTypeInformation
```

---

## Preserve Packet Capture

Record:

```text
Capture file:
Interface:
Capture start:
Capture end:
Capture filter:
Source system:
SHA-256:
```

Packet captures may contain sensitive internal traffic and must remain private.

---

## Hash Evidence

### Windows

```powershell
Get-FileHash `
    -LiteralPath "<EVIDENCE_FILE>" `
    -Algorithm SHA256
```

### Linux

```bash
sha256sum <EVIDENCE_FILE>
```

---

## Evidence Record

```text
Runbook: RB-09
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

* Approved Nmap exercise
* Vulnerability scanner
* Asset inventory
* Network monitoring
* Availability monitoring
* Service discovery
* Administrative troubleshooting
* Application health checks
* Software deployment
* Backup software
* Security validation
* CyberLab exercise

A benign source should still be reviewed for:

* Authorization
* Scope
* Schedule
* Process
* Target list
* Port list
* Logging visibility
* Operational impact

---

# Detection Gaps

Document whether the investigation lacked:

* Source address
* Destination address
* Source hostname
* Destination hostname
* Destination port
* Protocol
* Connection result
* Direction
* Source process
* Source user
* Process command line
* Process hash
* Packet evidence
* Allowed-connection visibility
* Target-service context
* Authentication correlation
* Approved-scanner context
* Multi-target correlation
* Multi-port correlation
* Reliable timestamps

---

# Detection Improvement Recommendations

Potential improvements include:

* Normalize source and destination fields
* Collect blocked and allowed firewall events
* Enable firewall text logging
* Enable Sysmon network telemetry
* Maintain an approved-scanner inventory
* Detect one source contacting many ports
* Detect one source contacting many hosts
* Detect one service port across many hosts
* Correlate scans with failed logons
* Correlate scans with successful logons
* Correlate scans with process creation
* Add privileged-service context
* Add target criticality
* Add scan-rate thresholds
* Detect repeated scans over longer windows
* Create a reconnaissance dashboard
* Create response workflows for lateral movement

---

# MITRE ATT&CK Context

Network reconnaissance may relate to:

```text
T1046 – Network Service Discovery
```

Related techniques may include:

* T1018 — Remote System Discovery
* T1087 — Account Discovery
* T1135 — Network Share Discovery
* T1049 — System Network Connections Discovery
* T1021 — Remote Services

Apply mappings only when supported by observed behavior.

A single connection attempt does not establish reconnaissance.

---

# Documentation and Closure

## Investigation Summary

```text
Alert:
Source system:
Source address:
Source user:
Source process:
Command line:
First observed:
Last observed:
Destination systems:
Destination addresses:
Unique targets:
Unique ports:
Protocols:
Blocked connections:
Allowed connections:
Open services identified:
Authentication attempts:
Successful logons:
Privilege activity:
Follow-on behavior:
Approved activity:
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

```text
Approved CyberLab exercise
Approved vulnerability scan
Approved asset inventory
Administrative troubleshooting
Application health check
Misconfigured monitoring
Misconfigured automation
Unauthorized user scanning
Malware service discovery
Lateral movement preparation
Password attack preparation
Compromised endpoint
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The original alert is confirmed.
* The source system and address are identified.
* The destination systems and ports are identified.
* The source user and process are identified or documented as unknown.
* Authorization is confirmed or disproved.
* Blocked and allowed connections are reviewed.
* Target services are identified.
* Authentication activity is reviewed.
* Follow-on exploitation or lateral movement is excluded or escalated.
* Required containment is completed.
* Unauthorized tools or processes are removed.
* Exposed services and firewall rules are reviewed.
* Monitoring confirms the activity stopped.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```text
The suspected network reconnaissance was investigated using endpoint firewall, Sysmon, packet capture, Wazuh, Splunk, process, service, and authentication telemetry. The source, target scope, ports, connection results, responsible process, authorization context, and follow-on activity were reviewed. The activity was classified as <CLASSIFICATION>. Required containment, corrective, and recovery actions were completed, monitoring confirmed that the unauthorized or unexpected activity stopped, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Operational IP addresses
* Internal hostnames
* Domain names
* Real usernames
* Process GUIDs
* MAC addresses
* VMware adapter identifiers
* Full firewall rule names
* Complete port inventories
* Internal service banners
* Packet-capture contents
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths
* Unrelated network traffic

Use placeholders such as:

```text
<SOURCE_SYSTEM>
<SOURCE_IP>
<SOURCE_USER>
<SOURCE_PROCESS>
<PROCESS_ID>
<TARGET_SYSTEM>
<TARGET_IP>
<DESTINATION_PORT>
<PROTOCOL>
<WINDOWS_ENDPOINT>
<KALI_SYSTEM>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Firewall block event
* Sanitized firewall-log entries
* Sysmon network event
* Source process command line
* Wazuh reconnaissance alert
* Splunk multi-port correlation
* Splunk multi-host correlation
* Packet timeline with sanitized addresses
* Authentication correlation
* Final recovery validation

Before publishing:

1. Crop unrelated information.
2. Replace operational IP addresses.
3. Remove usernames.
4. Remove internal hostnames and domains.
5. Remove unrelated ports and services.
6. Remove process GUIDs where unnecessary.
7. Remove packet payloads.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Source system identified
* Source address identified
* Destination systems identified
* Destination ports identified
* Initial severity assigned

## Source Validation

* Source hostname validated
* Source address validated
* Source user identified
* Source process identified
* Process command line preserved
* Parent process reviewed
* Process hash recorded where appropriate
* Authorization checked

## Target Review

* Target identities validated
* Firewall events reviewed
* Firewall text log reviewed
* Listening ports reviewed
* Services mapped
* Firewall rules reviewed
* Allowed connections reviewed
* Blocked connections reviewed

## Pattern Analysis

* Unique destination ports counted
* Unique targets counted
* Scan duration calculated
* Sequential or randomized pattern reviewed
* Vertical scan pattern assessed
* Horizontal scan pattern assessed
* Approved-scanner context reviewed

## Authentication and Follow-On Review

* Event ID 4625 reviewed
* Event ID 4624 reviewed
* Event ID 4648 reviewed
* RDP activity reviewed where relevant
* SMB activity reviewed where relevant
* SSH activity reviewed where relevant
* WinRM activity reviewed where relevant
* Exploitation indicators reviewed
* Privilege changes reviewed
* Account creation reviewed

## SIEM Review

* Wazuh alert reviewed
* Wazuh field quality assessed
* Splunk multi-port search completed
* Splunk multi-host search completed
* Allowed-versus-blocked search completed
* Authentication correlation completed
* Process correlation completed
* Timeline completed

## Response

* Severity updated
* Containment decision recorded
* Source process stopped where required
* Source endpoint isolated where required
* Account contained where required
* Network block applied where required
* Unauthorized tool removed
* Exposed services reviewed
* Firewall configuration corrected

## Closure

* Activity confirmed stopped
* Target services validated
* Authentication state reviewed
* Monitoring health validated
* Evidence preserved
* Evidence hashes recorded
* Root cause documented
* Authorized or incident classification recorded
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Primary Firewall Event Search

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 5152, 5153, 5156, 5157
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match [regex]::Escape("<SOURCE_IP>")
}
```

## Primary Source Process Search

```powershell
Get-CimInstance Win32_Process |
    Where-Object {
        $_.CommandLine -match "nmap|masscan|powershell|python"
    } |
Select-Object `
    ProcessId,
    ParentProcessId,
    Name,
    ExecutablePath,
    CommandLine,
    CreationDate
```

## Primary Splunk Search

```spl
index=windows earliest=-15m
(
    EventCode=5152 OR
    EventCode=5153 OR
    EventCode=5156 OR
    EventCode=5157
)
| eval source_ip=coalesce(SourceAddress, src_ip)
| eval destination_ip=coalesce(DestAddress, dest_ip)
| eval destination_port=coalesce(DestPort, dest_port)
| stats
    count
    dc(destination_port) as unique_ports
    values(destination_port) as ports
    by source_ip destination_ip
| where unique_ports >= <PORT_THRESHOLD>
| sort - unique_ports
```

## Immediate Escalation Indicators

```text
Unauthorized source
User workstation performing broad scan
Administrative ports targeted
Failed logons after scan
Successful logon after scan
Suspicious source process
Multiple systems targeted
Domain controller targeted
Security infrastructure targeted
Malware or Defender alert associated
Lateral movement indicators
Continuing activity
```

---

# Summary

This runbook provides a repeatable process for investigating suspected network reconnaissance.

A complete investigation should determine:

* Which system initiated the activity
* Which user and process were responsible
* Which systems and ports were targeted
* Whether connections were blocked or allowed
* Whether the activity matched an approved scanner or exercise
* Whether target services were exposed
* Whether authentication attempts followed
* Whether successful access occurred
* Whether exploitation or lateral movement followed
* Whether containment was required
* Whether unauthorized tools and access were removed
* Whether monitoring confirmed that the activity stopped

The investigation is complete only after the source, target scope, authorization, process activity, and follow-on behavior are understood, required response actions are completed, evidence is preserved, and detection improvements are documented.

