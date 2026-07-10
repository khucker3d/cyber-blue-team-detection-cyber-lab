# Kali Linux Testing System

## Overview

Kali Linux provides the authorized security-testing workstation for the Acer Blue Team CyberLab.

The system is used to generate controlled activity that can be observed and investigated through:

* Windows Event Viewer
* Wazuh
* Splunk
* Host-based firewall logs
* Packet captures
* Future network-monitoring tools

The purpose of the Kali system is not simply to run offensive tools. Its primary role is to help validate whether the lab’s defensive controls can:

* Detect activity
* Collect useful telemetry
* Correlate related events
* Distinguish expected testing from suspicious behavior
* Support a documented investigation
* Recover safely after an exercise

All testing is restricted to systems owned by the lab operator or systems for which explicit authorization has been granted.

This public document uses sanitized hostnames, IP addresses, usernames, interface names, command targets, and network details.

---

## System Role

The Kali system represents a controlled testing workstation operating inside the CyberLab.

It may be used to perform:

* Host discovery
* Port and service discovery
* DNS testing
* Web-service inspection
* Authentication validation
* Packet capture
* Network troubleshooting
* Traffic generation
* Detection-rule validation
* Incident-response exercises
* Defensive tool verification

The system should not be treated as a general-purpose personal workstation.

It should not contain:

* Personal credentials
* Personal documents
* Employer data
* Customer information
* Production configuration
* Real-world target lists
* Unauthorized captured traffic
* Malware samples outside an approved isolated workflow

---

## Public Example Configuration

The public documentation uses placeholder values such as:

```
Hostname: KALI-TEST
Internal address: 192.0.2.50
Internal network: 192.0.2.0/24
Domain controller: 192.0.2.10
Windows endpoint: 192.0.2.20
Wazuh server: 192.0.2.30
Splunk server: 192.0.2.40
```

The `192.0.2.0/24` range is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Architecture Placement

Kali connects primarily to the VMware host-only CyberLab network.

```
                         Acer Windows Host
                                 |
                         VMware Workstation
                                 |
                         Host-Only Network
                                 |
         ________________________|________________________
        |                        |                        |
     KALI-TEST               WIN11TARGET             SIEM Systems
 Authorized Tester        Monitored Endpoint       Wazuh / Splunk
```

A temporary VMware NAT adapter may be enabled for:

* Operating system updates
* Package installation
* Repository access
* Tool installation
* Time synchronization

The NAT adapter should be disconnected when external connectivity is not required.

---

## Security Boundary

The Kali system should interact only with:

* CyberLab virtual machines
* Lab services intentionally made available
* Dedicated testing accounts
* Synthetic files and data
* Authorized physical lab devices when explicitly documented

It should not scan or test:

* Public internet systems
* Employer systems
* Neighboring wireless networks
* Household devices outside the approved scope
* ISP infrastructure
* Third-party cloud services
* Systems that the operator does not own or have permission to assess

The presence of a tool in Kali does not create authorization to use it against an external target.

---

## Ethical Use Statement

Every exercise should have a defined scope.

Before testing, document:

```
Exercise:
Purpose:
Authorized systems:
Excluded systems:
Permitted techniques:
Start time:
Expected telemetry:
Cleanup:
Rollback plan:
```

Stop the exercise if:

* The target address is uncertain.
* Traffic reaches an unintended network.
* A bridged adapter is active unexpectedly.
* A command could affect availability beyond the approved target.
* Credentials or data from outside the lab appear.
* The impact is greater than expected.
* Recovery procedures are unavailable.

---

## Deployment Status

The Kali system may be deployed as:

* A dedicated VMware virtual machine
* A rebuilt testing VM
* A temporary live system
* A separate physical red-team workstation connected only during authorized exercises

For the Acer CyberLab, a VMware-based Kali system provides the simplest integration with the host-only lab network.

A separate physical Kali workstation can be documented as a future extension, but it should not be represented as part of the virtual architecture unless it is actually connected and validated.

---

## Prerequisites

Before creating or connecting the Kali system, confirm that the following are available:

* VMware Workstation
* Configured host-only and NAT networks
* Supported Kali Linux installation media
* Adequate CPU, memory, and storage
* A planned internal address
* A non-root administrative user
* A secure password
* Known authorized targets
* Stable domain controller and endpoint systems
* Active Wazuh or Splunk monitoring
* A clean snapshot strategy
* A written exercise scope

---

## Virtual Machine Creation

Create a dedicated Kali Linux virtual machine.

Suggested public VM name:

```
KALI-TEST
```

General resource guidance:

| Resource           | Guidance                                                 |
| ------------------ | -------------------------------------------------------- |
| Virtual processors | Moderate allocation                                      |
| Memory             | Enough for Kali tools, browser use, and packet analysis  |
| Storage            | Sufficient for the OS, packages, captures, and snapshots |
| Adapter 1          | Host-only                                                |
| Adapter 2          | NAT when required                                        |
| Operating system   | Supported 64-bit Kali Linux release                      |
| Desktop            | Lightweight desktop environment where practical          |

Exact resource values depend on the Acer host and the number of simultaneously running virtual machines.

---

## Resource Planning

Kali itself may run comfortably with moderate resources, but some activities increase demand.

Resource-intensive activities include:

* Large packet captures
* Browser-based testing
* Vulnerability scanning
* Password-auditing exercises
* Multiple concurrent tools
* Large wordlists
* Database-backed tools
* Graphical network analysis

The host must retain enough memory and processor capacity for:

* DC01
* WIN11TARGET
* Wazuh
* Splunk
* VMware Workstation
* Windows 11 host activity

Do not allocate all available host resources to Kali.

---

## Storage Planning

Kali storage may be consumed by:

* Package updates
* Tool dependencies
* Wordlists
* Packet captures
* Scan output
* Screenshots
* Browser data
* Temporary files
* Evidence exports
* VM snapshots

Large captures and wordlists should be reviewed regularly.

Do not store the only copy of important investigation evidence inside a disposable testing VM.

---

## Network Adapter Design

The recommended configuration uses two adapters.

### Host-Only Adapter

```
Purpose: Authorized CyberLab testing
Addressing: Static or VMware DHCP
Default gateway: none
```

### NAT Adapter

```
Purpose: Updates and package downloads
Addressing: VMware DHCP
Default gateway: VMware NAT gateway
```

Only the NAT adapter should normally provide the default route.

---

## Why Bridged Networking Is Avoided

A bridged Kali VM may gain direct access to:

* Home computers
* Mobile devices
* Smart-home devices
* Network infrastructure
* Printers
* Cameras
* Storage appliances
* Other physical systems

This creates unnecessary risk.

The standard CyberLab architecture uses:

* Host-only networking for testing
* NAT for temporary outbound access
* No bridged adapter

Bridged networking should only be introduced through a separately documented and authorized physical-network exercise.

---

## Kali Linux Installation

1. Create the VMware virtual machine.
2. Attach the Kali Linux installation ISO.
3. Start the VM.
4. Select the graphical or text installation method.
5. Choose the language and keyboard layout.
6. Configure the hostname.
7. Create a non-root user.
8. Set a strong password.
9. Configure the virtual disk.
10. Select the intended desktop environment.
11. Select the required tool collections.
12. Install the bootloader.
13. Complete the installation.
14. Restart the VM.
15. Remove or disconnect the installation ISO.
16. Install VMware guest integration packages.
17. Apply updates.
18. Create a clean baseline snapshot.

Do not publish passwords, disk identifiers, or personal usernames.

---

## User Account Model

Modern Kali installations support a normal user with `sudo` privileges.

Public placeholder:

```
<STUDENT_USER>
```

Use the normal account for:

* Browsing documentation
* Reviewing files
* Running nonprivileged tools
* Writing notes
* Examining captures that are readable by the user
* Basic network testing

Use `sudo` only when required.

---

## Root and Administrative Guidance

Elevated privileges may be required for:

* Package installation
* Network configuration
* Raw packet capture
* Interface mode changes
* Protected file access
* Service management
* Firewall configuration
* Certain scanning options
* Packet injection in an authorized wireless lab

Avoid opening a persistent root shell for routine work.

Prefer:

```
sudo <COMMAND>
```

over:

```
sudo su
```

or prolonged direct root sessions.

---

## Set the Hostname

Example:

```
sudo hostnamectl set-hostname KALI-TEST
```

Review `/etc/hosts`:

```
sudo nano /etc/hosts
```

Example:

```
127.0.0.1       localhost
127.0.1.1       KALI-TEST
```

Verify:

```
hostnamectl
```

---

## Install VMware Guest Tools

Kali generally uses `open-vm-tools`.

Update package information:

```
sudo apt update
```

Install the guest tools:

```
sudo apt install open-vm-tools open-vm-tools-desktop
```

Restart:

```
sudo reboot
```

Guest tools may improve:

* Display resizing
* Mouse integration
* Graceful shutdown
* Clipboard integration
* Time synchronization
* VMware device support

Host-guest integration features should be limited during higher-risk exercises.

---

## Shared Clipboard and Drag-and-Drop

Clipboard sharing and drag-and-drop cross the VM isolation boundary.

They can transfer:

* Commands
* Credentials
* URLs
* Files
* Malicious content
* Evidence

For routine setup, these features may be convenient.

For higher-risk exercises:

* Disable shared clipboard.
* Disable drag-and-drop.
* Avoid sharing the host user profile.
* Use a dedicated transfer method.
* Scan transferred files.
* Remove temporary shares after use.

---

## Shared Folder Risks

VMware shared folders should not expose:

* Personal documents
* Password-manager exports
* Browser profiles
* SSH keys
* Git credentials
* Cloud-storage directories
* Primary project repositories
* Employer data

Use a dedicated transfer directory.

Example public placeholder:

```
<HOST_TRANSFER_DIRECTORY>
```

Configure it as read-only when possible.

---

## Verify Network Interfaces

```
ip address
```

```
ip link
```

```
ip route
```

Identify:

* Host-only interface
* NAT interface
* Current addresses
* Default route
* Interface state

Interface names may resemble:

```
eth0
eth1
ens33
ens37
```

The actual names depend on the VM configuration.

---

## Example Network Design

```
Host-only interface:
Address: 192.0.2.50/24
Gateway: none
Purpose: Internal testing

NAT interface:
Address: VMware DHCP
Gateway: VMware NAT gateway
Purpose: Updates and downloads
```

---

## NetworkManager

Kali commonly uses NetworkManager.

Review devices:

```
nmcli device status
```

Review connections:

```
nmcli connection show
```

Review current settings:

```
nmcli device show
```

Administrative privileges are normally required to modify system network connections.

---

## Configure a Static Internal Address

Example:

```
sudo nmcli connection modify "<HOST_ONLY_CONNECTION>" \
    ipv4.method manual \
    ipv4.addresses "192.0.2.50/24" \
    ipv4.gateway "" \
    ipv4.dns "192.0.2.10"
```

Restart the connection:

```
sudo nmcli connection down "<HOST_ONLY_CONNECTION>"
sudo nmcli connection up "<HOST_ONLY_CONNECTION>"
```

Verify:

```
ip address
ip route
```

The documentation values are placeholders.

---

## DNS Design

Kali may use the domain controller for internal name resolution during Active Directory exercises.

Example:

```
Internal DNS: 192.0.2.10
Internal domain: cyberlab.example
```

When Kali is used only for IP-based testing, internal DNS may not be required.

External package resolution may use NAT-provided DNS when the NAT adapter is enabled.

---

## DNS Validation

Query the domain controller:

```
dig @192.0.2.10 cyberlab.example
```

Query DC01:

```
dig @192.0.2.10 DC01.cyberlab.example
```

Query Active Directory service records:

```
dig @192.0.2.10 _ldap._tcp.dc._msdcs.cyberlab.example SRV
```

Alternative:

```
nslookup DC01.cyberlab.example 192.0.2.10
```

---

## Validate Internal Connectivity

Test the Windows endpoint:

```
ping -c 4 192.0.2.20
```

Test the domain controller:

```
ping -c 4 192.0.2.10
```

Test a specific service:

```
nc -vz 192.0.2.20 <PORT>
```

A failed ping does not prove that a service is unavailable because ICMP may be blocked.

---

## Validate Internet Connectivity

With the NAT adapter enabled:

```
ping -c 4 1.1.1.1
```

Test DNS:

```
dig example.com
```

Test HTTPS:

```
curl -I https://example.com
```

After updates or package installation, disconnect the NAT adapter when external access is unnecessary.

---

## Update Kali

Update package metadata:

```
sudo apt update
```

Review available upgrades:

```
apt list --upgradable
```

Apply the standard rolling-distribution upgrade:

```
sudo apt full-upgrade
```

Remove unneeded packages when appropriate:

```
sudo apt autoremove
```

Restart if required:

```
sudo reboot
```

Review package changes before confirming large upgrades.

---

## Kali Metapackages

Kali tools are organized into metapackages.

Common examples include:

```
kali-linux-core
kali-tools-top10
kali-linux-default
kali-linux-large
kali-linux-everything
```

Installing every available tool is not necessary for a Blue Team validation VM.

A smaller installation:

* Reduces disk use
* Reduces package-management complexity
* Reduces attack surface
* Makes the system easier to maintain
* Encourages deliberate tool selection

---

## Recommended Tool Scope

Useful categories for this CyberLab include:

* Network discovery
* Port and service inspection
* Packet capture
* DNS utilities
* Web inspection
* Authentication testing
* Traffic generation
* File hashing
* Log inspection
* Scripting
* Basic enumeration

Install tools according to exercise requirements rather than installing everything by default.

---

## Core Utility Examples

Useful nonintrusive tools may include:

```
ip
ss
ping
traceroute
dig
nslookup
curl
wget
nc
tcpdump
Wireshark
Nmap
OpenSSL
sha256sum
jq
Python
```

Each tool should be used only within the approved exercise scope.

---

## Verify Installed Tools

Check a tool version:

```
nmap --version
```

```
wireshark --version
```

```
tcpdump --version
```

Locate a command:

```
which nmap
```

Review package information:

```
apt show nmap
```

---

## Tool Documentation

Before using a tool:

```
man <COMMAND>
```

Or:

```
<COMMAND> --help
```

Review:

* Target syntax
* Default behavior
* Timing
* Output options
* Privilege requirements
* Potential impact
* Whether the command performs active testing

Do not copy commands without understanding their scope.

---

## Target Verification

Before every active test, verify the target address.

Example:

```
ip neigh
```

```
arp -n
```

```
dig <TARGET_HOSTNAME>
```

Confirm that the address belongs to the intended CyberLab VM.

Do not rely only on memory.

---

## Authorized Target File

A simple private target file can reduce mistakes.

Example:

```
# Authorized CyberLab targets
192.0.2.10    DC01
192.0.2.20    WIN11TARGET
192.0.2.30    WAZUH-SERVER
192.0.2.40    SPLUNK-SERVER
```

Do not publish the operational version.

The public repository may include a sanitized template.

---

## Scope Guardrails

Before active testing:

1. Confirm the host-only adapter.
2. Confirm no bridged adapter is connected.
3. Confirm the target address.
4. Confirm the exercise objective.
5. Confirm monitoring systems are active.
6. Confirm snapshots exist.
7. Confirm the expected service impact.
8. Record the start time.
9. Use the narrowest practical command.
10. Stop after the validation objective is met.

---

## Safe Discovery Testing

A narrow host-discovery exercise may be used to verify:

* Reachability
* ARP behavior
* Endpoint firewall response
* Packet-capture visibility
* SIEM telemetry

Prefer a specific authorized target rather than scanning the entire subnet.

Example:

```
ping -c 4 <AUTHORIZED_TARGET>
```

Or:

```
nmap -sn <AUTHORIZED_TARGET>
```

Do not substitute a broad external range.

---

## Controlled Port Discovery

A limited scan can determine whether a specific service is reachable.

Example:

```
nmap -sT -p <APPROVED_PORTS> <AUTHORIZED_TARGET>
```

This uses a TCP connect scan and does not require raw-packet privileges.

Use a short approved port list rather than all ports unless the exercise specifically requires broader enumeration.

---

## Elevated Scanning

Some scan types require root privileges because they use raw packets.

Example:

```
sudo nmap <AUTHORIZED_OPTIONS> <AUTHORIZED_TARGET>
```

Root privileges increase both capability and risk.

Before using elevated scan options:

* Understand the technique.
* Limit the target.
* Limit the ports.
* Confirm the system can tolerate the traffic.
* Confirm monitoring is active.
* Record the command.
* Stop after validation.

---

## Avoid Aggressive Defaults

Broad or aggressive scan combinations may:

* Generate substantial traffic
* Trigger services unexpectedly
* Increase log volume
* Cause endpoint performance issues
* Produce confusing results
* Reach unintended targets if scope is wrong

Begin with the least invasive test that answers the question.

---

## Service Enumeration

Service inspection may include:

* TCP connection testing
* Banner review
* HTTP header review
* TLS certificate review
* DNS lookup
* Protocol-specific client connections

Examples:

```
nc -vz <AUTHORIZED_TARGET> <PORT>
```

```
curl -I http://<AUTHORIZED_TARGET>:<PORT>
```

```
openssl s_client \
    -connect <AUTHORIZED_TARGET>:<TLS_PORT>
```

Do not authenticate with real personal or production credentials.

---

## Web-Service Testing

For a lab web service, begin with passive or low-impact checks:

```
curl -I http://<AUTHORIZED_TARGET>:<PORT>
```

Review:

* Status code
* Server header
* Redirects
* Content type
* Security headers
* TLS use
* Authentication requirement

Browser developer tools may also be used for inspection.

Do not perform destructive web testing against services that are not specifically deployed for that purpose.

---

## Splunk Validation Example

Check whether Splunk Web is reachable:

```
curl -I http://<SPLUNK_SERVER>:8000
```

A response confirms that the service is reachable.

It does not authorize:

* Password guessing
* Session attacks
* Vulnerability exploitation
* Configuration changes
* Denial-of-service testing

---

## Wazuh Dashboard Validation Example

Use the intended internal dashboard address.

```
curl -k -I https://<WAZUH_SERVER>
```

The `-k` option disables certificate validation.

Use it only when the lab intentionally uses a known self-signed certificate and the target identity has already been verified.

Do not normalize bypassing certificate verification for production systems.

---

## Authentication Testing

Authentication exercises should use:

* Dedicated test accounts
* Known lab passwords
* Documented attempt limits
* Approved services
* Defined lockout expectations
* Monitoring on both endpoint and domain controller

Do not use:

* Personal passwords
* Leaked credential lists
* Employer credentials
* Real usernames from outside the lab
* Unbounded password guessing
* Administrative accounts unless specifically required

---

## Failed Authentication Exercise

A controlled exercise may generate one or a small number of failed sign-in attempts.

Document:

* Test account
* Source system
* Target system
* Protocol
* Attempt count
* Start time
* Expected Windows event
* Expected Wazuh result
* Expected Splunk result
* Cleanup

Avoid locking out critical administrative accounts.

---

## Account Lockout Exercise

Before testing account lockout:

1. Confirm the lockout policy.
2. Use a disposable test account.
3. Confirm another administrator account remains available.
4. Confirm DC01 is healthy.
5. Confirm Wazuh and Splunk are collecting events.
6. Record the planned attempt count.
7. Stop immediately after the lockout occurs.
8. Investigate before unlocking.
9. Restore access.
10. Document the exercise.

---

## Credential Safety

Never include passwords directly in:

* GitHub documentation
* Shell history
* Screenshots
* Packet captures intended for publication
* Script arguments
* Shared clipboard notes
* Public repositories
* Exercise filenames

Use interactive prompts or secure credential storage where possible.

---

## Shell History

Review recent commands:

```
history
```

Search for potentially sensitive terms:

```
history |
    grep -Ei 'password|token|key|secret'
```

The shell history file may contain commands from previous sessions.

Its path commonly resembles:

```
~/.bash_history
```

or another shell-specific history file.

Do not publish raw shell history.

---

## Disable or Limit History for a Sensitive Command

For a single controlled session, history behavior may be changed, but this should not be used to conceal unauthorized actions.

A safer practice is to avoid placing secrets in command arguments.

Document the action without documenting the secret.

---

## Packet Capture

Packet capture helps validate:

* ARP
* DNS
* ICMP
* TCP handshakes
* Service communication
* Authentication traffic
* Forwarder connections
* Agent communication
* Firewall behavior

Capture only the traffic required for the exercise.

---

## Identify the Capture Interface

```
ip link
```

List interfaces visible to `tcpdump`:

```
sudo tcpdump -D
```

Select the host-only interface rather than the NAT interface when investigating CyberLab traffic.

---

## Basic Packet Capture

Capture traffic to or from one authorized target:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <AUTHORIZED_TARGET>
```

Limit by port:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <AUTHORIZED_TARGET> \
    and port <PORT>
```

Stop the capture as soon as the required evidence is collected.

---

## Save a Packet Capture

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <AUTHORIZED_TARGET> \
    -w <CAPTURE_FILE>.pcap
```

Packet captures may contain:

* Internal addresses
* Hostnames
* Usernames
* Session data
* Authentication exchanges
* Unencrypted content
* Service information

Treat captures as sensitive evidence.

---

## Packet Capture Permissions

Raw packet capture normally requires root or Linux capabilities.

Wireshark may be configured to allow members of a dedicated group to capture without running the full graphical application as root.

Do not run the complete Wireshark graphical interface as root unless there is a specific justified reason.

---

## Wireshark

Wireshark can be used to inspect:

* ARP
* DNS
* ICMP
* TCP
* TLS
* Kerberos
* LDAP
* SMB
* HTTP
* Splunk or Wazuh communication metadata

Example display filters:

```
arp
dns
icmp
tcp
kerberos
ldap
smb2
http
tls
```

Protocol visibility depends on encryption, capture location, and configuration.

---

## Packet Capture Evidence Workflow

1. Define the expected traffic.
2. Select the correct interface.
3. Apply a narrow capture filter.
4. Start the capture.
5. Perform the test.
6. Stop the capture.
7. Record the capture time.
8. Calculate a hash.
9. Store the original securely.
10. Use a sanitized screenshot for public documentation.

---

## Calculate a Capture Hash

```
sha256sum <CAPTURE_FILE>.pcap
```

Record:

```
File:
Source system:
Interface:
Collection time:
Exercise:
SHA-256:
Student analyst:
```

Do not rename or modify the original after hashing without creating a new hash record.

---

## DNS Testing

Useful DNS validation commands include:

```
dig <HOSTNAME>
```

```
dig @<DNS_SERVER> <HOSTNAME>
```

```
nslookup <HOSTNAME> <DNS_SERVER>
```

For Active Directory:

```
dig @<DOMAIN_CONTROLLER> \
    _ldap._tcp.dc._msdcs.<LAB_DOMAIN> \
    SRV
```

DNS testing is useful for both troubleshooting and detection exercises.

---

## Traffic Generation

The Kali system may generate harmless network activity to validate:

* Endpoint firewall logging
* Packet capture
* Wazuh alerting
* Splunk ingestion
* Network baselines
* Service availability

Examples include:

* Ping
* DNS queries
* TCP connection attempts
* HTTP requests
* TLS handshakes
* Limited approved scans

Use predictable tests with clear expected results.

---

## Detection Validation Workflow

A complete defensive validation exercise should follow this path:

```
KALI-TEST
    |
    | Authorized activity
    v
WIN11TARGET or Lab Service
    |
    | Endpoint and network telemetry
    v
Wazuh / Splunk
    |
    | Alert or searchable event
    v
Student Investigation
```

The activity is successful only when the telemetry and investigation outcome are validated.

---

## Exercise Documentation Template

```
Title:

Objective:

Authorized source:
KALI-TEST

Authorized target:

Preconditions:

Command or action:

Expected target behavior:

Expected Windows telemetry:

Expected Wazuh telemetry:

Expected Splunk telemetry:

Observed result:

False positives or limitations:

Cleanup:

Evidence:

Conclusion:
```

---

## Safe Exercise: Single-Port Connection Test

### Objective

Confirm that a known service is reachable and determine what telemetry is created.

### Procedure

1. Confirm the target address.
2. Confirm the approved port.
3. Start monitoring or packet capture.
4. Run:

```
nc -vz <AUTHORIZED_TARGET> <APPROVED_PORT>
```

5. Record the time.
6. Review target logs.
7. Review Wazuh.
8. Review Splunk.
9. Stop the capture.
10. Document the result.

---

## Safe Exercise: Limited Port Scan

### Objective

Determine whether a short list of approved services is visible and whether the scan produces useful telemetry.

### Procedure

1. Confirm the target VM.
2. Confirm the target is not a production system.
3. Confirm Wazuh and Splunk are active.
4. Run:

```
nmap -sT \
    -p <APPROVED_PORT_LIST> \
    <AUTHORIZED_TARGET>
```

5. Save the output.
6. Review target firewall logs.
7. Review packet capture.
8. Search Wazuh and Splunk.
9. Record detection gaps.
10. End the exercise.

---

## Save Nmap Output

Normal output:

```
nmap \
    -sT \
    -p <APPROVED_PORT_LIST> \
    <AUTHORIZED_TARGET> \
    -oN <OUTPUT_FILE>.txt
```

XML output:

```
nmap \
    -sT \
    -p <APPROVED_PORT_LIST> \
    <AUTHORIZED_TARGET> \
    -oX <OUTPUT_FILE>.xml
```

Do not publish output containing operational addresses or MAC identifiers without sanitization.

---

## Safe Exercise: DNS Query Validation

### Objective

Confirm internal DNS behavior and inspect related network traffic.

```
dig @<DOMAIN_CONTROLLER> \
    <WINDOWS_ENDPOINT>.<LAB_DOMAIN>
```

Review:

* Query source
* DNS server
* Response
* Record type
* Response time
* Packet capture
* DNS server logs where available

---

## Safe Exercise: HTTP Request Validation

### Objective

Confirm a lab web interface is reachable and identify what telemetry is produced.

```
curl -I http://<AUTHORIZED_TARGET>:<PORT>
```

Review:

* TCP connection
* HTTP status
* Server response
* Target logs
* Firewall logs
* Packet capture
* Wazuh or Splunk data

Do not attempt authentication bypass or exploit testing unless the service was deployed specifically for that exercise.

---

## Safe Exercise: Failed Logon Generation

Use a dedicated test account and an approved authentication service.

The exercise should generate only the minimum required failures.

Review:

* Source address
* Target account
* Failure reason
* Authentication protocol
* Domain controller events
* Endpoint events
* Wazuh alerts
* Splunk searches
* Lockout status

Do not include a real password in the command or documentation.

---

## Detection Comparison

The same test activity may appear differently across tools.

| Source               | Possible Visibility                             |
| -------------------- | ----------------------------------------------- |
| Windows Event Viewer | Local operating-system events                   |
| Domain Controller    | Authentication and account events               |
| Wazuh                | Parsed alerts, rule level, MITRE mapping        |
| Splunk               | Searchable raw events, correlations, dashboards |
| Wireshark            | Packet-level communication                      |
| Firewall log         | Allowed or blocked connections                  |
| Kali output          | Tester-side results                             |

Comparing these views helps identify telemetry gaps.

---

## Exercise Timing

Record exact start and stop times.

Example:

```
Exercise start: YYYY-MM-DD HH:MM:SS TZ
Exercise stop: YYYY-MM-DD HH:MM:SS TZ
```

Accurate timestamps make it easier to:

* Search Splunk
* Filter Wazuh
* Review Event Viewer
* Locate packet-capture frames
* Reconstruct timelines
* Separate test activity from background events

---

## System Time

Review:

```
timedatectl
```

Check synchronization:

```
timedatectl status
```

Kali time should align with:

* DC01
* WIN11TARGET
* Wazuh
* Splunk
* Acer host

Snapshot restoration may cause clock changes.

Validate time before each exercise.

---

## Local Firewall

Kali may use firewall technologies such as:

* nftables
* iptables compatibility tools
* UFW when installed
* NetworkManager firewall integration

Review nftables:

```
sudo nft list ruleset
```

Review listening services:

```
sudo ss -tulpn
```

Kali should not expose unnecessary services to the CyberLab.

---

## Disable Unnecessary Services

Review active services:

```
systemctl --type=service --state=running
```

Before disabling a service:

1. Identify its purpose.
2. Confirm it is unnecessary.
3. Record the current state.
4. Stop the service.
5. Disable it when appropriate.
6. Validate the system.
7. Maintain rollback instructions.

Example:

```
sudo systemctl stop <SERVICE>
sudo systemctl disable <SERVICE>
```

---

## SSH Service

Kali does not need to expose SSH unless remote administration is required.

Check:

```
sudo systemctl status ssh
```

If not needed:

```
sudo systemctl disable --now ssh
```

If required:

* Restrict it to the host-only network.
* Disable root login.
* Use a strong account.
* Prefer key-based authentication.
* Limit firewall access.
* Stop it after the exercise if no longer needed.

---

## Browser Safety

The Kali browser may be used for:

* Vendor documentation
* Internal lab interfaces
* Web inspection
* GitHub
* Research

Recommended practices:

* Use a dedicated lab profile.
* Do not sign in to personal email.
* Do not save personal passwords.
* Do not sync personal browser data.
* Clear downloads.
* Review history before screenshots.
* Hide bookmarks.
* Avoid mixing personal browsing with lab testing.

---

## Package and Repository Safety

Install packages only from:

* Official Kali repositories
* Trusted vendor sources
* Reviewed project repositories
* Known package files with verified hashes

Avoid:

* Unverified installation scripts
* Random binary downloads
* Abandoned repositories
* Commands copied without review
* Adding unnecessary third-party repositories

A security tool can itself be malicious or compromised.

---

## Git Repository Review

Before running a downloaded script:

```
git status
```

```
git log --oneline -5
```

Review the source:

```
less <SCRIPT_FILE>
```

Calculate a hash when preserving evidence:

```
sha256sum <SCRIPT_FILE>
```

Do not execute code simply because it is described as a security tool.

---

## Python Environments

Avoid installing every Python tool globally.

Use:

* Virtual environments
* `pipx`
* Project-specific environments
* Distribution packages when available

Example:

```
python3 -m venv <VENV_DIRECTORY>
```

Activate:

```
source <VENV_DIRECTORY>/bin/activate
```

Install only the required package.

---

## Wordlists

Kali may include or support large wordlists.

Wordlists can consume substantial disk space and may contain real leaked passwords.

For a defensive lab:

* Prefer synthetic wordlists.
* Use only dedicated test credentials.
* Do not publish leaked credential data.
* Do not use real password dumps.
* Store large files outside the public repository.
* Delete unnecessary copies.

---

## Password Auditing Scope

Password auditing should be limited to:

* Lab-created accounts
* Lab-created hashes
* Synthetic passwords
* Explicitly approved authentication services
* Defined attempt limits

Do not test:

* Personal online accounts
* Employer credentials
* Third-party services
* Public login portals
* Passwords obtained from breaches

---

## Exploit Frameworks

Kali includes tools capable of exploitation.

The presence of these tools does not mean they are required for every exercise.

Before using an exploit framework:

1. Confirm the target is deliberately vulnerable or a disposable lab system.
2. Confirm the exact vulnerability.
3. Create snapshots.
4. Isolate the target.
5. Document the payload and expected effect.
6. Confirm monitoring is active.
7. Avoid persistence.
8. Avoid data destruction.
9. Restore the system afterward.
10. Document the defensive findings.

Exploit testing should be documented separately from basic Kali setup.

---

## Malware and Payload Safety

Do not introduce live malware into the standard CyberLab without a dedicated malware-analysis architecture.

The normal Acer CyberLab includes connected identity and monitoring systems and should not be assumed safe for unrestricted malware execution.

Safer validation options include:

* Harmless antivirus test files
* Synthetic logs
* Benign scripts
* Prebuilt public datasets
* Detection simulation frameworks designed for labs
* Manual creation of expected event patterns

A dedicated malware-analysis lab requires stronger isolation and separate documentation.

---

## Denial-of-Service Testing

Availability-impacting tests are not part of the standard Kali setup.

Do not perform:

* Flooding
* Resource exhaustion
* Service-crash testing
* Network saturation
* Fork-bomb activity
* Destructive storage tests

unless a disposable target and isolated exercise have been specifically designed.

The default CyberLab goal is telemetry validation, not service disruption.

---

## Evidence Directory

Create a dedicated location:

```
mkdir -p ~/CyberLab-Evidence
```

Suggested structure:

```
~/CyberLab-Evidence/
├── Commands/
├── Captures/
├── Scans/
├── Screenshots/
├── Hashes/
└── Notes/
```

The operational username and path should be removed from public screenshots.

---

## Evidence File Naming

Recommended format:

```
<DATE>-<EXERCISE>-<SOURCE>-<TYPE>
```

Examples:

```
YYYY-MM-DD-port-validation-kali-nmap.txt
YYYY-MM-DD-dns-test-kali-packet-capture.pcap
YYYY-MM-DD-authentication-test-kali-notes.md
```

Avoid including passwords, internal addresses, or personal identifiers in filenames.

---

## Command Logging

Record the exact authorized command used during an exercise.

A private exercise record should include:

```
Command:
Source:
Target:
Date and time:
Purpose:
Expected result:
Observed result:
Impact:
Cleanup:
```

Before publishing, replace operational values with placeholders.

---

## Terminal Recording

A terminal session may be recorded using tools such as `script`.

Example:

```
script <SESSION_LOG>.txt
```

Exit the recording:

```
exit
```

Terminal recordings may contain:

* Usernames
* Paths
* Addresses
* Commands
* Credentials entered visibly
* Sensitive output

Review and sanitize before publication.

---

## Screenshot Preparation

Before taking Kali screenshots:

1. Close unrelated terminals.
2. Clear personal browser data.
3. Hide bookmarks.
4. Remove notification content.
5. Use a neutral wallpaper if desired.
6. Avoid showing personal usernames.
7. Remove operational IP addresses.
8. Remove MAC addresses.
9. Remove shell history.
10. Crop the image to the relevant tool output.
11. Permanently redact sensitive values.
12. Reopen the final image to verify it.

---

## Snapshot Strategy

Recommended milestones:

```
01-KALI-Clean-Install
02-KALI-Patched-Baseline
03-KALI-VMware-Tools
04-KALI-Network-Configured
05-KALI-Core-Tools-Validated
06-KALI-Packet-Capture-Validated
07-KALI-SIEM-Test-Validated
08-KALI-Exercise-Ready
```

Create important stable snapshots while the VM is powered off.

---

## Safe Snapshot Procedure

1. End active tests.
2. Stop packet captures.
3. Save required evidence.
4. Remove temporary credentials.
5. Close testing tools.
6. Disconnect the NAT adapter if appropriate.
7. Shut down Kali cleanly.
8. Confirm the VM is powered off.
9. Create the snapshot.
10. Restart and validate the network.

---

## Snapshot Restore Risks

Restoring Kali may cause:

* Time rollback
* Lost evidence
* Old tool versions
* Stale SSH keys
* Reused scan output
* Incorrect network configuration
* Duplicate files
* Shell-history confusion
* Reappearance of deleted credentials
* Outdated authorized-target lists

After restoration:

1. Confirm the date and time.
2. Confirm network adapters.
3. Confirm no bridged adapter is active.
4. Confirm the target list.
5. Apply required updates.
6. Confirm monitoring systems are active.
7. Review shell history.
8. Remove stale evidence.
9. Generate a new test event.
10. Document the restoration.

---

## Backup Strategy

Important Kali backup targets include:

* Custom scripts
* Exercise documentation
* Sanitized configurations
* Packet-capture notes
* Hash records
* Custom wordlists containing only synthetic data
* Tool configuration
* Authorized-target templates

The entire Kali VM may also be preserved through a powered-off cold copy.

Do not back up secrets into a public repository.

---

## Cold Backup Procedure

1. End active exercises.
2. Save evidence outside the VM when required.
3. Shut down Kali.
4. Confirm the VM is powered off.
5. Copy or export the VM directory.
6. Store the backup on separate media.
7. Record the milestone.
8. Verify the copied files.
9. Restart the original VM.
10. Validate network isolation.

---

## Startup Procedure

1. Start DC01.
2. Confirm DNS and time.
3. Start WIN11TARGET.
4. Start Wazuh.
5. Start Splunk.
6. Confirm monitoring and ingestion.
7. Start KALI-TEST.
8. Confirm the host-only adapter.
9. Confirm the NAT adapter state.
10. Confirm the authorized target.
11. Record the exercise start time.
12. Begin testing.

Kali should generally start after the monitored systems are ready.

---

## Shutdown Procedure

1. Stop active commands.
2. Stop packet captures.
3. Save testing output.
4. Record the exercise stop time.
5. Confirm required telemetry reached the SIEM.
6. Remove temporary files and credentials.
7. Disconnect NAT if appropriate.
8. Shut down Kali.
9. Investigate and document the events.
10. Shut down dependent systems according to the CyberLab procedure.

---

## Validation Checklist

### Operating System

* Kali starts successfully.
* The hostname is correct.
* A normal administrative user exists.
* Root is not used for routine work.
* Guest tools are installed.
* The system time is correct.
* Required updates are installed.

### Networking

* The host-only adapter is connected.
* The internal address is correct.
* The NAT adapter is enabled only when required.
* Only the NAT adapter provides a default route.
* No bridged adapter is active.
* Authorized targets are reachable.
* Unintended physical-network access is absent.

### Tools

* Required utilities are installed.
* Tool versions are documented where useful.
* Commands are understood before use.
* Output directories exist.
* Packet capture works.
* DNS tools work.
* Limited connection testing works.

### Safety

* The exercise scope is documented.
* Targets are verified.
* Personal credentials are absent.
* Monitoring systems are active.
* Stable snapshots exist.
* Cleanup and rollback steps exist.
* Evidence handling is defined.

### Monitoring

* Kali traffic can be seen in packet captures.
* The target records expected telemetry.
* Wazuh receives relevant events.
* Splunk can search the related data.
* Timestamps align.
* The investigation outcome is documented.

---

## Troubleshooting: Kali Has No Internal Connectivity

Check:

1. The VM is running.
2. The host-only adapter is connected.
3. The correct VMnet is selected.
4. The interface is enabled.
5. The address and subnet are correct.
6. The target system is running.
7. The target firewall allows the expected traffic.
8. The VMware host-only adapter exists.

Commands:

```
ip address
ip route
nmcli device status
```

Test:

```
ping -c 4 <AUTHORIZED_TARGET>
```

---

## Troubleshooting: Kali Has No Internet Access

Check:

* NAT adapter connection
* VMware DHCP address
* Default route
* DNS configuration
* VMware NAT service
* Acer host connectivity
* Host VPN behavior
* Local firewall

Commands:

```
ip address
ip route
resolvectl status
```

Test separately:

```
ping -c 4 1.1.1.1
```

```
dig example.com
```

Do not add a gateway to the host-only adapter.

---

## Troubleshooting: DNS Fails

Review:

```
resolvectl status
```

```
cat /etc/resolv.conf
```

Test the internal server directly:

```
dig @<DOMAIN_CONTROLLER> <LAB_DOMAIN>
```

Possible causes include:

* Incorrect DNS server
* DC01 offline
* NAT DNS overriding internal DNS
* Incorrect search domain
* NetworkManager configuration
* Missing Active Directory records

---

## Troubleshooting: Scan Finds Nothing

Possible explanations include:

* Wrong target
* Target powered off
* Firewall blocking probes
* Service not listening
* Incorrect network
* NAT address used instead of host-only address
* Command syntax
* Insufficient privileges for the chosen scan type

Validate the target with:

```
ip neigh
```

```
nc -vz <AUTHORIZED_TARGET> <KNOWN_PORT>
```

Do not immediately broaden the scan to unrelated networks.

---

## Troubleshooting: Tool Requires Root

Some tools require raw-socket or capture privileges.

Confirm the reason before using:

```
sudo <COMMAND>
```

Do not run the entire desktop session as root.

For Wireshark capture permissions, use the distribution-supported group or capability configuration rather than launching the graphical interface as root.

---

## Troubleshooting: Packet Capture Shows No Traffic

Check:

* Correct interface
* Correct capture filter
* Correct target
* Traffic actually generated
* VPN or NAT path
* Interface state
* Permission to capture

List interfaces:

```
sudo tcpdump -D
```

Test without a restrictive filter briefly:

```
sudo tcpdump -i <HOST_ONLY_INTERFACE> -c 20
```

Then reapply a narrow filter.

---

## Troubleshooting: Wazuh Does Not Alert

Check:

1. The target generated a local event.
2. The Wazuh agent is active.
3. The event channel is collected.
4. A decoder recognizes the event.
5. A rule exists.
6. The dashboard time filter is correct.
7. The test was significant enough to match a rule.

Not every connection or scan produces a Wazuh alert by default.

The absence of an alert may indicate a detection-development opportunity rather than platform failure.

---

## Troubleshooting: Splunk Shows No Related Event

Check:

* Target event source
* Universal Forwarder
* Receiver listener
* Target index
* Time range
* Host field
* Event identifier
* Ingestion delay
* Timestamp alignment

Begin with a broader search limited to the target and timeframe, then narrow the query.

---

## Troubleshooting: Too Much Event Noise

Possible causes include:

* Scan scope too broad
* Repeated connection attempts
* Large port range
* High scan rate
* Excessive packet capture
* Verbose logging
* Monitoring broad directories
* Background endpoint activity

Reduce:

* Target count
* Port count
* Scan rate
* Exercise duration
* Collected event sources
* Capture scope

The goal is useful telemetry, not maximum volume.

---

## Troubleshooting: Wrong Network Was Scanned

Stop immediately.

Then:

1. Save the command for the incident record.
2. Record the actual target range.
3. Disconnect Kali from the network if necessary.
4. Confirm no further scan process is running.
5. Review affected systems.
6. Determine whether any non-lab system was contacted.
7. Document the cause.
8. Correct the adapter or target list.
9. Add a preventive control.
10. Do not repeat the test until scope is verified.

Possible preventive controls include:

* No bridged adapter
* Authorized-target file
* Firewall egress restrictions
* Narrow commands
* Pre-exercise checklist
* Separate test network

---

## Public Sanitization Standards

Remove or replace:

* Real internal IP addresses
* Real physical-network addresses
* MAC addresses
* Personal usernames
* Shell prompts containing names
* Home-directory paths
* Host VPN information
* Wireless network names
* External IP addresses
* Target lists
* Passwords
* Hashes derived from real credentials
* Session tokens
* SSH private keys
* Scan results exposing operational services
* Packet captures
* Browser history
* Personal bookmarks
* Tool configuration containing secrets

Use placeholders such as:

```
<KALI_SYSTEM>
<AUTHORIZED_TARGET>
<WINDOWS_ENDPOINT>
<DOMAIN_CONTROLLER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<HOST_ONLY_INTERFACE>
<NAT_INTERFACE>
<APPROVED_PORTS>
<LAB_DOMAIN>
<STUDENT_USER>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing screenshots from Kali:

1. Verify the target is sanitized.
2. Remove internal addresses.
3. Remove MAC addresses.
4. Hide personal usernames.
5. Hide terminal history.
6. Remove SSH keys or fingerprints where sensitive.
7. Close unrelated browser tabs.
8. Hide bookmarks.
9. Remove external network details.
10. Crop to the relevant output.
11. Permanently redact sensitive values.
12. Verify the final image independently.

Suitable public screenshots may include:

* Sanitized interface configuration
* Limited authorized scan
* Packet-capture example
* DNS validation
* Splunk service-reachability test
* Wazuh test traffic
* SIEM correlation result
* Synthetic exercise output

---

## Skills Demonstrated

This deployment demonstrates:

* Kali Linux administration
* VMware virtual machine deployment
* Linux networking
* NetworkManager
* Host-only and NAT architecture
* DNS validation
* Authorized network discovery
* Port and service testing
* Packet capture
* Wireshark
* Nmap
* Linux permissions
* Tool-scope management
* Evidence collection
* SIEM validation
* Detection engineering support
* Ethical testing boundaries
* Snapshot management
* Recovery planning
* Documentation sanitization

---

## Lessons Learned

Key lessons include:

* Kali should be treated as a controlled testing system, not just a tool collection.
* Host-only networking is the safest default for CyberLab exercises.
* Bridged networking creates unnecessary scope risk.
* The target should be verified before every active command.
* Narrow tests produce cleaner and more useful telemetry.
* Root privileges should be used only when the technique requires them.
* Packet captures may contain more sensitive information than screenshots.
* The absence of an alert does not always mean telemetry is absent.
* Wazuh and Splunk provide different views of the same activity.
* Accurate timestamps are essential for correlation.
* Monitoring systems should start before Kali.
* Evidence must be preserved before reverting a snapshot.
* Public scan output requires careful sanitization.
* A written scope is a technical control, not merely an administrative formality.

---

## Planned Improvements

Future improvements may include:

* Separate attacker and defender host-only networks
* Internal firewall or router VM
* Dedicated vulnerable target systems
* Detection simulation framework
* Sigma-rule validation
* Sysmon-focused exercises
* Suricata integration
* Zeek network telemetry
* Centralized packet-capture sensor
* Automated scope validation
* Egress firewall restrictions
* Dedicated jump host
* Additional Linux target
* Web application testing target
* Active Directory security exercises
* Automated evidence hashing
* Reusable exercise scripts
* Sanitized sample datasets
* Detection coverage matrix
* Purple-team exercise documentation
* Separate malware-analysis environment

---

## Summary

Kali Linux provides the controlled activity-generation and network-analysis capability for the Acer Blue Team CyberLab.

A safe and useful deployment requires:

* Host-only network isolation
* Temporary NAT access only when required
* No default bridged networking
* Verified authorized targets
* Narrow, documented testing
* Limited use of root privileges
* Active Wazuh and Splunk monitoring
* Accurate time synchronization
* Evidence preservation
* Stable snapshots
* Clear cleanup and rollback procedures
* Aggressive public sanitization

The value of the Kali system is measured not by how many tools are installed, but by how effectively its authorized activity helps validate telemetry, detections, investigations, and defensive improvements.

