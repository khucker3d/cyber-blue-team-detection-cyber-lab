# CyberLab Architecture

## Overview

The Acer Blue Team CyberLab is a virtualized security training environment designed for defensive security practice, system administration, monitoring, log analysis, and controlled testing.

The environment runs on a Windows 11 host using VMware Workstation. Multiple virtual machines are connected through isolated and internet-enabled virtual networks to simulate a small enterprise environment.

The architecture separates:

* Identity services
* Monitored endpoints
* Security monitoring platforms
* Testing systems
* Administrative access
* Internet access
* Internal lab traffic

All addressing, domain names, usernames, and system identifiers shown in this public document are sanitized examples.

---

## Architecture Goals

The environment was designed around the following goals:

* Isolate security exercises from the physical home network
* Support Active Directory and Windows authentication testing
* Provide controlled internet access for updates and software installation
* Collect and analyze endpoint security telemetry
* Generate safe test activity for detection validation
* Support repeatable Blue Team exercises
* Allow rapid recovery through virtual machine snapshots
* Avoid exposing lab services to the public internet
* Maintain documentation suitable for a public portfolio

---

## High-Level Architecture

```text
                               Internet
                                  |
                         Physical Home Router
                                  |
                       Acer Windows 11 Host
                                  |
                         VMware Workstation
                                  |
             _____________________|_____________________
            |                                           |
       VMware NAT                                  Host-Only Network
       Network                                     Internal CyberLab
            |                                           |
            |                        ___________________|___________________
            |                       |                   |                   |
      Wazuh Server                DC01           Windows Endpoint       Kali Linux
      Splunk Server        Domain Controller       WIN11TARGET        Test System
            |                       |                   |                   |
            |_______________________|___________________|___________________|
                                    |
                          Security Monitoring Traffic
```

The exact operational network ranges are intentionally excluded from the public repository.

---

## Physical Host

### Host Role

The Acer Windows 11 system serves as the physical virtualization host.

Its responsibilities include:

* Running VMware Workstation
* Hosting all CyberLab virtual machines
* Providing local administrative access
* Managing virtual networks
* Storing virtual disks and snapshots
* Providing controlled internet access through NAT
* Supporting backup and recovery operations

### Host Security Considerations

The host system should be treated as part of the CyberLab security boundary.

Recommended controls include:

* Keep Windows fully updated.
* Use full-disk encryption where available.
* Use a strong local account password.
* Protect the Windows administrator account.
* Avoid storing lab credentials in plain text.
* Keep VMware Workstation updated.
* Do not expose VMware management services externally.
* Store virtual machines in a dedicated directory.
* Back up important configuration files.
* Avoid running unnecessary services on the host.

The host should not use the same credentials as the lab domain.

---

## Virtualization Platform

VMware Workstation provides the following functionality:

* Virtual machine creation
* Virtual hardware management
* Virtual network configuration
* Snapshot management
* NAT connectivity
* Host-only networking
* ISO mounting
* Virtual disk management
* Console access
* Resource allocation

Each virtual machine is assigned only the network adapters required for its role.

---

## Virtual Machines

## Domain Controller

### Public Example Identity

```text
Hostname: DC01
Domain: cyberlab.example
Example address: 192.0.2.10
```

These values are documentation placeholders and do not represent the live environment.

### Responsibilities

The domain controller provides:

* Active Directory Domain Services
* Internal DNS
* Domain authentication
* User and group management
* Computer account management
* Group Policy testing
* Windows security event generation
* Centralized identity services

### Network Placement

The domain controller is placed primarily on the host-only network.

It does not require direct inbound access from:

* The internet
* The physical home network
* Untrusted devices

Temporary outbound access may be enabled when required for:

* Windows updates
* Software installation
* Time synchronization
* Administrative maintenance

### Security Importance

The domain controller is the most sensitive virtual machine in the lab because it controls:

* User authentication
* Computer trust relationships
* DNS for the lab domain
* Administrative privileges
* Domain security policy

Administrative access should be limited to designated lab administrator accounts.

---

## Windows Endpoint

### Public Example Identity

```text
Hostname: WIN11TARGET
Domain: cyberlab.example
Example address: 192.0.2.20
```

### Responsibilities

The Windows endpoint represents a typical enterprise workstation.

It is used for:

* Domain authentication
* Security event generation
* Endpoint monitoring
* Agent deployment
* File integrity testing
* PowerShell logging
* Process monitoring
* Account activity testing
* Incident investigation exercises

### Network Placement

The endpoint communicates with the domain controller over the host-only network.

Required traffic may include:

* DNS
* Kerberos
* LDAP
* SMB
* RPC
* Group Policy
* Windows authentication
* SIEM agent communication

The endpoint may also use NAT temporarily for:

* Operating system updates
* Agent downloads
* Software installation
* Security tool deployment

### Monitoring Role

The Windows endpoint generates telemetry for:

* Wazuh
* Splunk
* Windows Event Viewer
* Future Sysmon integration
* Detection engineering exercises

---

## Wazuh Server

### Public Example Identity

```text
Hostname: WAZUH-SERVER
Example internal address: 192.0.2.30
Example management URL: https://<WAZUH_SERVER>
```

### Responsibilities

The Wazuh server provides:

* Endpoint agent management
* Security event collection
* Alert generation
* File integrity monitoring
* Security configuration assessment
* Vulnerability visibility
* MITRE ATT&CK mappings
* Compliance-related event tagging
* Dashboard-based investigation

### Network Placement

The Wazuh server may use two virtual network adapters.

#### Internal Adapter

The internal adapter connects to the host-only network.

It supports:

* Agent enrollment
* Endpoint telemetry
* Internal administration
* Dashboard access
* Alert review
* Log ingestion

#### NAT Adapter

The NAT adapter provides controlled outbound internet access.

It may be used for:

* Package downloads
* Repository access
* Container image downloads
* Operating system updates
* Threat intelligence updates
* Time synchronization

The NAT adapter should not be treated as a trusted path into the lab.

### Multi-Homed Server Considerations

A system with more than one network adapter can experience:

* Incorrect default routes
* DNS resolution conflicts
* Traffic leaving through the wrong adapter
* Dashboard access failures
* Agent enrollment problems
* Package installation failures
* Unexpected source addresses

Only one adapter should normally provide the default gateway.

The internal host-only adapter should generally not define an internet gateway.

---

## Splunk Server

### Public Example Identity

```text
Hostname: SPLUNK-SERVER
Example internal address: 192.0.2.40
Example management URL: http://<SPLUNK_SERVER>:8000
```

### Responsibilities

Splunk Enterprise is used for:

* Log searching
* Event correlation
* Dashboard creation
* Windows event analysis
* Security investigation
* Search Processing Language practice
* Detection query development
* Timeline reconstruction

### Network Placement

Splunk communicates with systems inside the host-only network.

It may receive data from:

* Windows endpoints
* Linux systems
* Security tools
* Forwarders
* Imported test datasets
* Manually uploaded logs

A NAT adapter may be enabled temporarily for software installation or updates.

### Management Access

The Splunk web interface should remain accessible only from trusted lab administration systems.

It should not be:

* Port-forwarded through the home router
* Exposed directly to the internet
* Published through a public reverse proxy
* Accessible from untrusted wireless networks

---

## Kali Linux Test System

### Public Example Identity

```text
Hostname: KALI-TEST
Example address: 192.0.2.50
```

### Responsibilities

Kali Linux is used as an authorized testing system.

Its purpose is to generate controlled activity for defensive validation, including:

* Host discovery
* Port scanning
* Service enumeration
* Authentication testing
* Network traffic generation
* Detection rule validation
* Packet capture exercises

### Network Placement

Kali is connected to the host-only CyberLab network when performing security exercises.

A NAT adapter may be used for:

* Tool installation
* Package updates
* Repository access

Internet access is not required during most internal exercises.

### Ethical Boundary

Testing is limited to:

* CyberLab virtual machines
* Systems owned by the operator
* Services intentionally deployed for testing
* Explicitly authorized targets

The testing system must not be used against third-party systems without authorization.

---

## Virtual Network Design

The CyberLab uses two main VMware network types:

* Host-only
* NAT

---

## Host-Only Network

### Purpose

The host-only network acts as the primary internal CyberLab network.

It provides communication between:

* Domain controller
* Windows endpoint
* Wazuh server
* Splunk server
* Kali Linux
* Acer host management interfaces

### Characteristics

The host-only network:

* Is isolated from the physical network by default
* Does not directly route to the internet
* Supports internal lab communication
* Reduces exposure to home devices
* Provides a predictable testing environment
* Supports repeatable security exercises

### Example Addressing

```text
Documentation subnet: 192.0.2.0/24
Gateway: Not required for isolated operation
Domain Controller: 192.0.2.10
Windows Endpoint: 192.0.2.20
Wazuh Server: 192.0.2.30
Splunk Server: 192.0.2.40
Kali Linux: 192.0.2.50
```

The `192.0.2.0/24` range is reserved for documentation and example use.

It should not be interpreted as the actual CyberLab subnet.

---

## NAT Network

### Purpose

The NAT network provides controlled outbound connectivity through the Windows host.

It is used for:

* Software downloads
* Package installation
* Operating system updates
* Repository access
* Time synchronization
* Temporary internet-dependent tasks

### Characteristics

The NAT network:

* Uses the Windows host for outbound translation
* Does not normally expose virtual machines directly to the home network
* Allows internet access without bridged networking
* Reduces the need for inbound firewall changes
* Can be disconnected when no longer needed

### Security Practice

The NAT adapter should be disabled when a system does not require internet access.

This reduces:

* Unnecessary exposure
* Accidental outbound traffic
* Software update interruptions during exercises
* Routing complexity
* DNS conflicts

---

## Why Bridged Networking Is Avoided

Bridged networking places a virtual machine directly onto the physical network.

That would allow the virtual machine to interact more directly with:

* Home computers
* Mobile devices
* Smart home equipment
* Network infrastructure
* Other trusted systems

The CyberLab avoids bridged networking unless there is a specific, documented requirement.

Host-only and NAT networking provide sufficient connectivity for most exercises while maintaining a stronger isolation boundary.

---

## Network Adapter Matrix

| System            | Host-Only Adapter |      NAT Adapter | Default Use            |
| ----------------- | ----------------: | ---------------: | ---------------------- |
| Domain Controller |               Yes |         Optional | Identity and DNS       |
| Windows Endpoint  |               Yes |         Optional | Monitored workstation  |
| Wazuh Server      |               Yes |              Yes | Monitoring and updates |
| Splunk Server     |               Yes |         Optional | Log analysis           |
| Kali Linux        |               Yes |         Optional | Controlled testing     |
| Windows Host      |  VMware interface | Physical network | Lab administration     |

Optional NAT adapters should only be enabled when needed.

---

## Trust Zones

The architecture can be divided into several trust zones.

## Physical Home Network

This zone contains:

* Physical router
* Personal devices
* Smart home systems
* Mobile devices
* Non-lab workstations

The CyberLab should not assume that access to the physical network is necessary.

---

## Virtualization Host Zone

This zone contains the Acer Windows host and VMware Workstation.

The host acts as:

* The administrative entry point
* The storage platform
* The network boundary
* The NAT provider
* The snapshot manager

Compromise of the host could affect every virtual machine in the CyberLab.

---

## Identity Zone

This zone contains the domain controller.

It provides:

* Authentication
* DNS
* Directory services
* Security policy
* Administrative trust

Access to this zone should be tightly controlled.

---

## Endpoint Zone

This zone contains the Windows workstation.

It represents a monitored user system where:

* User activity occurs
* Security events are generated
* Agents collect telemetry
* Test incidents are created
* Investigation workflows are practiced

---

## Security Monitoring Zone

This zone contains Wazuh and Splunk.

It is used for:

* Log collection
* Event analysis
* Detection development
* Alert investigation
* Dashboard access
* Monitoring validation

Administrative interfaces should be limited to trusted users.

---

## Testing Zone

This zone contains Kali Linux and other future testing systems.

It should be treated as less trusted than the monitoring and identity zones.

Testing systems should only receive the access required for a specific exercise.

---

## Primary Traffic Flows

## Domain Authentication Flow

```text
Windows Endpoint
      |
      | DNS query
      v
Domain Controller
      |
      | Domain discovery
      | Kerberos authentication
      | LDAP requests
      | Group Policy
      v
Windows Endpoint
```

The endpoint must use the domain controller as its DNS server to reliably locate domain services.

---

## Wazuh Telemetry Flow

```text
Windows Endpoint
      |
      | Security events
      | File integrity events
      | Agent status
      | Configuration data
      v
Wazuh Server
      |
      | Indexing and analysis
      v
Wazuh Dashboard
```

The exact ports and registration credentials are intentionally omitted from the public architecture document.

---

## Splunk Ingestion Flow

```text
Windows or Linux Source
      |
      | Forwarded logs or imported data
      v
Splunk Server
      |
      | Parsing
      | Indexing
      | Search
      v
Splunk Web Interface
```

Data ingestion methods may vary by exercise.

---

## Controlled Test Flow

```text
Kali Linux
      |
      | Authorized scan or test activity
      v
Windows Endpoint or Lab Service
      |
      | Security and network telemetry
      v
Wazuh / Splunk
      |
      | Detection and investigation
      v
Analyst Review
```

All generated activity should be:

* Planned
* Authorized
* Documented
* Reversible
* Limited to the lab

---

## Administrative Flow

```text
Acer Windows Host
      |
      | VMware console
      | Browser administration
      | Remote shell
      | PowerShell
      v
CyberLab Virtual Machines
```

The Acer host is the primary administrative workstation.

Administrative access should not be exposed beyond the trusted local environment.

---

## DNS Architecture

The domain controller provides DNS for the internal lab domain.

Domain-joined systems should use the domain controller as their primary DNS server.

Using an external DNS resolver directly on a domain-joined endpoint can cause:

* Domain join failures
* Domain controller discovery failures
* Group Policy failures
* Authentication delays
* Incorrect name resolution

The domain controller may forward external DNS requests to an approved upstream resolver when internet resolution is required.

### Example Flow

```text
Windows Endpoint
      |
      | Internal DNS request
      v
Domain Controller
      |
      | External lookup when required
      v
Approved Upstream Resolver
```

---

## Routing Design

The host-only network is intended primarily for internal communication.

The NAT adapter provides the default route for systems that require internet access.

A multi-homed system should generally follow this model:

```text
Host-Only Adapter
- Internal static address
- No default gateway
- Internal DNS as required

NAT Adapter
- DHCP or assigned NAT address
- Default gateway provided by VMware
- External resolution as required
```

Incorrect gateway placement may cause traffic to use the wrong network adapter.

---

## Firewall Design

Each system should maintain its local firewall.

Only required traffic should be permitted.

Examples include:

* Domain authentication traffic
* DNS
* SIEM agent communication
* Splunk ingestion
* Administrative web interfaces
* SSH where applicable
* Remote management when explicitly enabled

Broad firewall disabling should be avoided.

Temporary firewall changes should be:

1. Documented.
2. Limited in scope.
3. Tested.
4. Reversed after troubleshooting.
5. Replaced with a specific rule.

---

## Time Synchronization

Accurate system time is essential for:

* Kerberos authentication
* Log correlation
* Incident timelines
* SIEM alerts
* Event sequencing
* Certificate validation

All lab systems should maintain consistent time.

Large time differences can cause:

* Domain authentication failures
* Incorrect event ordering
* Misleading SIEM timelines
* Agent communication problems
* Certificate errors

Time synchronization should be validated before troubleshooting complex authentication or logging failures.

---

## Resource Allocation

Virtual machine resource assignments should balance performance with host capacity.

Typical considerations include:

* Processor cores
* Memory
* Virtual disk size
* Disk growth mode
* Network adapters
* Snapshot storage
* Background services

Security monitoring platforms typically require more memory and storage than standard endpoint systems.

The host should retain enough resources to remain stable while all required virtual machines are running.

---

## Snapshot Architecture

Snapshots are used to preserve known-good configuration states.

Recommended milestones include:

* Clean operating system installation
* Fully patched baseline
* Domain controller deployment
* Domain configuration complete
* Endpoint domain join complete
* Wazuh installation complete
* Wazuh agent enrollment complete
* Splunk installation complete
* Pre-exercise state
* Post-validation state

Snapshot names should describe the state clearly.

Example:

```text
01-Clean-Install
02-Patched-Baseline
03-Domain-Configured
04-Agent-Enrolled
05-SIEM-Validated
```

Snapshots should not be used as the only form of backup.

---

## Availability Dependencies

The systems have several important dependencies.

### Windows Endpoint Dependencies

The endpoint depends on:

* Domain controller availability
* Internal DNS
* Correct system time
* Host-only network connectivity
* SIEM server availability for telemetry forwarding

### Wazuh Dependencies

Wazuh depends on:

* Linux host services
* Required containers or components
* Correct storage permissions
* Available disk space
* Network connectivity
* Accurate time
* Correct agent configuration

### Splunk Dependencies

Splunk depends on:

* Splunk services
* Available storage
* Correct listening interfaces
* Valid data inputs
* Network connectivity
* Accurate timestamps

Understanding these dependencies helps identify the likely source of a failure.

---

## Architecture Validation

The architecture should be validated in layers.

### Layer 1: Virtual Hardware

Confirm:

* Each virtual machine starts.
* The expected network adapters are connected.
* Virtual disks are available.
* Resource allocation is sufficient.

### Layer 2: IP Connectivity

Confirm:

* Each system has the expected address.
* Systems can communicate over the host-only network.
* NAT-enabled systems can reach approved external resources.

Example commands:

```powershell
ipconfig /all
ping <LAB_SYSTEM>
Test-NetConnection <LAB_SYSTEM> -Port <PORT>
```

```bash
ip address
ip route
ping -c 4 <LAB_SYSTEM>
ss -tulpn
```

### Layer 3: DNS

Confirm:

* The endpoint uses the domain controller for DNS.
* Internal hostnames resolve.
* The lab domain resolves.
* External forwarding works when required.

```powershell
nslookup <LAB_DOMAIN>
nslookup <HOSTNAME>
```

### Layer 4: Identity

Confirm:

* The endpoint can locate the domain.
* Domain authentication succeeds.
* Group Policy can be retrieved.
* Security logs record authentication activity.

### Layer 5: Monitoring

Confirm:

* The endpoint agent is active.
* Events reach the monitoring platform.
* Dashboards are accessible.
* Test activity produces expected telemetry.

### Layer 6: Detection

Confirm:

* A controlled event is generated.
* The event is collected.
* The event is searchable.
* Relevant alerts are created.
* The investigation can be documented.

---

## Public Sanitization Boundary

The live CyberLab contains details that should not be published.

The following information is removed or replaced:

* Real internal IP addresses
* Physical network ranges
* Public IP addresses
* MAC addresses
* Personal usernames
* Email addresses
* Passwords
* Domain administrator credentials
* Agent registration credentials
* Enrollment tokens
* API keys
* Session cookies
* Internal domain names
* VMware UUIDs
* Device serial numbers
* License information
* Personal file paths
* Browser bookmarks
* Router identifiers
* Wireless network names

Public examples use placeholders such as:

```text
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<KALI_TEST_SYSTEM>
<LAB_DOMAIN>
<LAB_SUBNET>
<ADMIN_USERNAME>
<REDACTED_TOKEN>
```

Screenshots should be reviewed independently before publication.

---

## Architectural Limitations

This environment is intended for education and portfolio development.

It does not fully reproduce:

* Enterprise redundancy
* High availability
* Multi-site architecture
* Production identity governance
* Large-scale log ingestion
* Dedicated security appliances
* Enterprise backup infrastructure
* Hardware network segmentation
* Production change control
* Formal certificate management
* Centralized secrets management

These limitations are acceptable for the project because the primary objective is hands-on learning and defensive security practice.

---

## Planned Architecture Improvements

Future enhancements may include:

* Sysmon deployment
* Windows Event Forwarding
* Dedicated log collector
* Sigma rule testing
* Suricata integration
* Zeek network monitoring
* Vulnerability scanner
* Additional Linux endpoint
* Dedicated jump host
* Separate attacker network
* Internal certificate authority
* Automated configuration backups
* Detection-as-code repository
* Infrastructure health monitoring
* Centralized time source
* More granular virtual network segmentation
* Automated environment validation scripts

---

## Architecture Summary

The Acer Blue Team CyberLab uses a layered virtual architecture that separates identity, endpoint activity, security monitoring, testing, and internet access.

The host-only network provides the main isolation boundary, while VMware NAT provides controlled outbound connectivity when required.

The environment supports:

* Active Directory practice
* Windows endpoint monitoring
* SIEM administration
* Log analysis
* Detection engineering
* Controlled adversary simulation
* Incident investigation
* Troubleshooting
* Recovery testing

The architecture is intentionally documented using sanitized values so that the project can demonstrate technical knowledge without exposing the operational environment.

