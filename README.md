# Blue Team CyberLab

## Project Overview

The Blue Team CyberLab is a virtualized cybersecurity environment designed for hands-on practice with:

* Windows system administration
* Active Directory
* Security monitoring
* Security information and event management
* Endpoint telemetry
* Log collection and analysis
* Detection engineering
* Incident investigation
* Network traffic analysis
* Defensive security testing

The lab runs on an Acer Windows 11 workstation using VMware Workstation. It contains multiple isolated virtual machines representing common systems found in a small enterprise environment.

This repository documents the architecture, deployment process, security controls, validation procedures, troubleshooting methods, and defensive security exercises used throughout the project.

All public documentation has been sanitized to remove internal IP addresses, usernames, passwords, host-specific identifiers, and other sensitive configuration details.

---

## Project Goals

The primary goal of this project is to build a reusable Blue Team training environment that supports practical security exercises without affecting production systems or the physical home network.

The lab was designed to provide experience with:

* Building and managing virtualized infrastructure
* Configuring isolated and internet-enabled virtual networks
* Deploying Windows Server and Active Directory
* Joining Windows endpoints to a domain
* Installing and managing SIEM platforms
* Forwarding and analyzing Windows security events
* Validating security alerts
* Creating repeatable investigation workflows
* Practicing detection engineering concepts
* Documenting operational and recovery procedures

---

## Lab Components

| Component                        | Purpose                                                                |
| -------------------------------- | ---------------------------------------------------------------------- |
| Windows 11 Host                  | Physical virtualization host                                           |
| VMware Workstation               | Virtual machine and network management                                 |
| Windows Server Domain Controller | Active Directory, DNS, authentication, and policy management           |
| Windows 11 Endpoint              | Domain-joined monitored workstation                                    |
| Wazuh Server                     | Endpoint monitoring, alerting, compliance visibility, and log analysis |
| Splunk Enterprise                | Log searching, dashboards, correlation, and investigation practice     |
| Kali Linux                       | Authorized testing and controlled activity generation                  |
| Host-Only Network                | Isolated internal lab communication                                    |
| NAT Network                      | Controlled internet access for updates and software installation       |

---

## Architecture

The CyberLab uses separate VMware virtual networks to balance isolation and usability.

```text
                         Internet
                            |
                     Physical Router
                            |
                  Windows 11 Host System
                            |
                   VMware Workstation
              _____________|_____________
             |                           |
        NAT Network                 Host-Only Network
             |                           |
      Controlled Internet         Isolated Lab Traffic
             |                           |
        Wazuh Server -------------- Domain Controller
        Splunk Server ------------- Windows Endpoint
        Kali Linux ---------------- Security Systems
```

The exact network ranges used in the operational lab are intentionally omitted from the public repository.

Documentation examples use reserved documentation address ranges such as:

```text
192.0.2.0/24
198.51.100.0/24
203.0.113.0/24
```

These ranges are reserved for documentation and should not be copied directly into a production network.

---

## Virtual Machines

### Domain Controller

The Windows Server domain controller provides:

* Active Directory Domain Services
* Internal DNS
* Centralized identity management
* Domain authentication
* Group Policy testing
* Windows security event generation

The public documentation uses placeholder values such as:

```text
Domain: cyberlab.example
Hostname: DC01
Address: 192.0.2.10
```

These values do not represent the operational environment.

### Windows Endpoint

The Windows endpoint represents a domain-joined employee workstation.

It is used to generate and investigate activity such as:

* Successful and failed logons
* Account lockouts
* User creation
* Group membership changes
* Administrative actions
* PowerShell execution
* Process creation
* Remote access activity
* File integrity events
* Endpoint security alerts

### Wazuh Server

Wazuh provides:

* Endpoint agent management
* Security event collection
* File integrity monitoring
* Vulnerability visibility
* Configuration assessment
* MITRE ATT&CK mappings
* Compliance-related event categorization
* Alert investigation

### Splunk Enterprise

Splunk is used for:

* Searching collected events
* Building dashboards
* Creating detection queries
* Reviewing Windows event data
* Correlating activity across systems
* Practicing security investigation workflows

### Kali Linux

Kali Linux is used only within the authorized lab environment.

Its purpose is to generate controlled activity for defensive monitoring, including:

* Network discovery
* Service enumeration
* Authentication testing
* Traffic generation
* Security tool validation

No testing is performed against systems outside the owner-controlled CyberLab.

---

## Network Design

The lab uses two primary VMware network types.

### Host-Only Network

The host-only network provides isolated communication between lab systems.

It is used for:

* Active Directory traffic
* DNS communication
* Endpoint-to-SIEM traffic
* Administrative access
* Controlled attack simulation
* Log forwarding
* Detection exercises

The host-only network is not directly reachable from the internet.

### NAT Network

The NAT network provides controlled outbound internet access through the host computer.

It is used temporarily for:

* Operating system updates
* Package installation
* Security tool downloads
* Repository access
* Time synchronization

Virtual machines do not require direct inbound exposure from the home network or internet.

### Multi-Homed Systems

Some infrastructure systems may use more than one virtual network adapter.

For example, a SIEM server may use:

* One adapter for isolated lab communication
* One adapter for controlled software updates

Routing, DNS, and adapter priority must be configured carefully to prevent connectivity issues or unintended exposure.

---

## Deployment Order

The environment was built in the following order:

1. Prepare the Windows 11 virtualization host.
2. Install VMware Workstation.
3. Create the host-only and NAT virtual networks.
4. Deploy the Windows Server domain controller.
5. Configure Active Directory and DNS.
6. Deploy the Windows 11 endpoint.
7. Join the endpoint to the domain.
8. Deploy the Wazuh server.
9. Install the Wazuh agent on the endpoint.
10. Validate endpoint enrollment and event ingestion.
11. Deploy Splunk Enterprise.
12. Configure test data ingestion.
13. Deploy or connect the Kali testing system.
14. Perform controlled validation exercises.
15. Create clean virtual machine snapshots.
16. Document the final architecture and recovery procedures.

---

## Validation

Each major component is validated before continuing to the next deployment phase.

### Domain Validation

Validation checks include:

* The domain controller responds on the internal network.
* Internal DNS resolves the lab domain.
* The Windows endpoint can locate the domain controller.
* The endpoint successfully joins the domain.
* Domain users can authenticate.
* Windows security logs record authentication activity.

### Wazuh Validation

Validation checks include:

* The Wazuh server services are running.
* The management dashboard is reachable.
* The Windows endpoint agent is enrolled.
* The endpoint reports as active.
* Security events appear in the dashboard.
* Test activity produces the expected alerts.
* MITRE ATT&CK and compliance metadata are visible where applicable.

### Splunk Validation

Validation checks include:

* Splunk services start successfully.
* The web interface is reachable.
* The correct listening interfaces are configured.
* Test events can be ingested.
* Searches return expected results.
* Time fields and source types are interpreted correctly.

### Network Validation

Typical validation commands include:

```powershell
ipconfig /all
ping <lab-system>
nslookup <lab-domain>
Test-NetConnection <lab-system> -Port <service-port>
```

Linux validation may include:

```bash
ip address
ip route
ping -c 4 <lab-system>
ss -tulpn
curl -I http://<lab-system>:<port>
```

Administrative or root permissions should only be used when required by the specific command or service.

---

## Example Security Exercises

The lab supports repeatable exercises such as:

### Failed Logon Investigation

Generate controlled failed authentication attempts and review:

* Windows event identifiers
* Source account
* Source workstation
* Failure reason
* Authentication package
* Alert severity
* Related Wazuh or Splunk events

### Account Lockout Investigation

Trigger a test account lockout and identify:

* The affected user
* The system generating the failures
* The number and timing of failed attempts
* The lockout event
* Related authentication events
* Required remediation steps

### Suspicious PowerShell Activity

Execute a harmless PowerShell test command and review:

* Process creation events
* Command-line logging
* Parent process
* User context
* Script block telemetry, when enabled
* SIEM detection results

### User and Privilege Changes

Create a temporary test user or modify group membership, then investigate:

* Who performed the change
* Which account was modified
* Which group was affected
* Whether elevated privileges were granted
* Whether the change was authorized

### Network Reconnaissance Detection

Run a controlled scan against authorized lab systems and review:

* Source system
* Destination systems
* Ports contacted
* Scan timing
* Firewall or endpoint events
* Detection opportunities

---

## Snapshot and Recovery Strategy

Virtual machine snapshots are taken at stable milestones.

Recommended snapshot points include:

* Clean operating system installation
* Post-update baseline
* Domain controller configured
* Endpoint joined to the domain
* SIEM server installed
* Agent enrollment validated
* Splunk installation validated
* Pre-exercise baseline
* Final documented configuration

Snapshots are created only while the virtual machines are in a safe state.

For important infrastructure changes, the preferred process is:

1. Shut down the virtual machine cleanly.
2. Confirm that the virtual disk is no longer active.
3. Create a clearly named snapshot.
4. Add a description explaining the milestone.
5. Start the virtual machine.
6. Confirm that services still operate correctly.

Snapshots improve recovery but are not considered a replacement for independent backups.

---

## Security Controls

The lab follows several defensive design principles:

* No production credentials are used.
* No business or customer data is stored in the lab.
* Virtual machines are isolated from production systems where possible.
* Internet access is limited to systems that require it.
* No lab services are intentionally exposed to the public internet.
* Default passwords are changed.
* Administrative access is restricted.
* Test accounts are separate from normal user accounts.
* Snapshots are created before high-risk configuration changes.
* Security exercises are limited to systems owned and controlled by the lab operator.
* Screenshots are reviewed and sanitized before publication.

---

## Public Sanitization Standards

The following information is removed or replaced before content is committed to GitHub:

* Real internal IP addresses
* Public IP addresses
* MAC addresses
* Router and gateway identifiers
* Wireless network names
* Email addresses
* Personal usernames
* Passwords
* API keys
* Enrollment tokens
* Agent registration keys
* Session cookies
* Internal domain names
* Device serial numbers
* VMware identifiers
* License information
* Browser bookmarks
* File paths containing personal names
* QR codes
* Notification webhook addresses

Placeholder examples should be clearly labeled.

```text
<SIEM_SERVER_IP>
<DOMAIN_NAME>
<WINDOWS_ENDPOINT>
<ADMIN_USERNAME>
<REDACTED_TOKEN>
```

Documentation-only IP ranges should be used when an example address is necessary.

---

## Troubleshooting Areas

Common issues encountered during the project included:

* Virtual machines receiving addresses from the wrong VMware network
* Incorrect DNS settings preventing domain joins
* Multi-adapter routing conflicts
* SIEM services not starting after reboot
* Web interfaces listening on unexpected interfaces
* Firewall rules blocking management ports
* Windows agents failing to enroll
* Incorrect system time affecting log correlation
* Splunk installation package transfer issues
* Services requiring elevated startup permissions
* Host-only systems being unable to reach package repositories

Each troubleshooting document records:

* Symptoms
* Likely causes
* Diagnostic commands
* Corrective actions
* Validation steps
* Lessons learned

---

## Skills Demonstrated

This project demonstrates practical experience with:

* VMware Workstation
* Windows 11 administration
* Windows Server
* Active Directory
* DNS
* Group Policy concepts
* Linux administration
* Wazuh
* Splunk
* SIEM deployment
* Log ingestion
* Windows Event Viewer
* Endpoint agent management
* Network troubleshooting
* PowerShell
* Bash
* Security monitoring
* Detection engineering
* Incident investigation
* Snapshot management
* Technical documentation
* Security sanitization

---

## Lessons Learned

Several engineering lessons became clear during the project:

* DNS configuration is critical to Active Directory reliability.
* Network isolation should be designed before deploying services.
* Multi-homed virtual machines require careful routing and DNS planning.
* Each service should be validated independently before adding additional complexity.
* Snapshots should be created at known-good milestones.
* Successful installation does not automatically mean successful integration.
* Log timestamps and system time must be synchronized.
* Troubleshooting steps should be documented while the issue is being resolved.
* Public documentation requires a separate sanitization review.
* Recovery procedures are as important as deployment procedures.

---

## Planned Improvements

Future improvements may include:

* Sysmon deployment
* Centralized Windows event forwarding
* Additional Wazuh custom rules
* Splunk dashboards
* Sigma rule testing
* Suricata or Zeek integration
* Vulnerability scanning
* Linux endpoint monitoring
* Automated log generation
* Incident response playbooks
* Detection-as-code workflows
* Active Directory attack-path monitoring
* Network segmentation exercises
* Automated environment health checks
* Infrastructure backup documentation

---

## Ethical Use Statement

This CyberLab is intended exclusively for education, defensive security research, and authorized testing.

All security testing must be limited to systems that the operator owns or has explicit permission to assess.

The techniques documented in this repository should not be used against third-party systems, networks, services, accounts, or data without authorization.

---

## Disclaimer

This repository documents a personal educational lab.

The configurations and procedures may require modification for other environments. They should not be treated as production deployment guidance without additional security review, testing, and organizational approval.

