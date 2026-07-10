# Wazuh SIEM Setup

## Overview

Wazuh provides the primary endpoint monitoring and security analytics platform for the Acer Blue Team CyberLab.

The deployment is used to practice:

* SIEM administration
* Endpoint agent enrollment
* Windows event collection
* File integrity monitoring
* Security configuration assessment
* Vulnerability visibility
* Alert investigation
* MITRE ATT&CK mapping
* Compliance-related event categorization
* Detection validation
* Incident analysis
* Linux and container administration

The Wazuh server runs inside an Ubuntu virtual machine hosted by VMware Workstation.

This document describes the sanitized public deployment process. Operational IP addresses, credentials, enrollment keys, internal URLs, usernames, host-specific paths, and other sensitive values have been removed or replaced.

---

## Wazuh Role in the CyberLab

Wazuh acts as the central defensive monitoring platform for the lab.

Its responsibilities include:

* Receiving telemetry from monitored endpoints
* Managing endpoint agents
* Parsing Windows and Linux events
* Evaluating events against detection rules
* Generating security alerts
* Monitoring selected files and directories
* Assessing endpoint configuration
* Presenting alerts through a web dashboard
* Supporting investigation and validation exercises
* Mapping relevant activity to security frameworks

The Wazuh deployment is intended for education and controlled testing.

It is not exposed to the public internet.

---

## Public Example Configuration

The public documentation uses placeholder values such as:

```
Server hostname: WAZUH-SERVER
Internal address: 192.0.2.30
Internal network: 192.0.2.0/24
Dashboard URL: https://<WAZUH_SERVER>
Monitored endpoint: WIN11TARGET
Domain: cyberlab.example
```

The `192.0.2.0/24` network is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Architecture Placement

The Wazuh server communicates with monitored systems through the VMware host-only network.

A second VMware NAT adapter provides controlled outbound access for installation and updates.

```
                              Internet
                                 |
                          VMware NAT Network
                                 |
                         WAZUH-SERVER
                                 |
                       Host-Only CyberLab
                                 |
              ___________________|___________________
             |                   |                   |
           DC01             WIN11TARGET          Acer Host
      Domain Controller    Wazuh Agent       Dashboard Access
```

The internal adapter handles:

* Agent enrollment
* Endpoint telemetry
* Dashboard access
* Administrative access
* Internal monitoring traffic

The NAT adapter handles:

* Package installation
* Container image downloads
* Repository access
* Operating system updates
* Time synchronization

---

## Wazuh Components

A Wazuh all-in-one deployment generally combines several components.

| Component           | Purpose                                                      |
| ------------------- | ------------------------------------------------------------ |
| Wazuh Manager       | Receives agent data and evaluates security events            |
| Wazuh Indexer       | Stores and indexes security data                             |
| Wazuh Dashboard     | Provides browser-based searches, alerts, and visualizations  |
| Wazuh Agent         | Collects telemetry from monitored endpoints                  |
| Supporting services | Provide networking, certificates, storage, and orchestration |

In this lab, the manager, indexer, and dashboard are hosted on one Ubuntu virtual machine.

This simplifies deployment but does not provide enterprise redundancy.

---

## All-in-One Deployment Considerations

Advantages include:

* Lower infrastructure requirements
* Simpler deployment
* Easier recovery
* Fewer virtual machines
* Suitable for a small training environment
* Easier documentation

Limitations include:

* Single point of failure
* Shared compute and storage
* No high availability
* Limited scaling
* Maintenance affects all components
* Resource contention during indexing and searches

These limitations are acceptable for a personal CyberLab.

---

## Prerequisites

Before deploying Wazuh, confirm that the following are available:

* VMware Workstation
* Configured host-only and NAT networks
* Ubuntu Server installation media
* Sufficient host memory and storage
* A planned static internal address
* Working internet access through VMware NAT
* Accurate system time
* Administrative access to the Ubuntu server
* A functioning Windows endpoint
* A snapshot and recovery plan
* A secure place to store generated credentials

---

## Virtual Machine Creation

Create a dedicated Ubuntu Server virtual machine.

Suggested public VM name:

```
WAZUH-SERVER
```

General hardware considerations:

| Resource           | Guidance                                                |
| ------------------ | ------------------------------------------------------- |
| Virtual processors | Moderate allocation                                     |
| Memory             | Higher than a standard endpoint                         |
| Storage            | Sufficient for indexes, logs, containers, and snapshots |
| Adapter 1          | Host-only                                               |
| Adapter 2          | NAT                                                     |
| Operating system   | Supported 64-bit Ubuntu Server release                  |

Wazuh indexing and dashboard services can consume significant memory and storage.

The Acer host must retain enough resources for Windows and the other CyberLab virtual machines.

---

## Storage Planning

Wazuh storage consumption is affected by:

* Number of agents
* Event volume
* Alert retention
* File integrity activity
* Vulnerability data
* Indexer configuration
* Dashboard usage
* Container images
* System logs
* Snapshot growth

Maintain enough free space for normal operations and recovery.

Low disk space can cause:

* Indexer failures
* Missing alerts
* Dashboard errors
* Container restart loops
* Slow searches
* Corrupted or incomplete data
* Failed upgrades

---

## Network Adapter Design

The server uses two VMware adapters.

### Internal Adapter

```
Type: Host-only
Purpose: CyberLab communication
Addressing: Static
Default gateway: none
```

### External Adapter

```
Type: VMware NAT
Purpose: Updates and downloads
Addressing: VMware DHCP
Default gateway: VMware NAT gateway
```

Only the NAT adapter should normally provide the default route.

---

## Multi-Homed Server Risks

A dual-adapter Wazuh server can experience:

* Multiple default gateways
* Incorrect DNS selection
* Services binding to the wrong address
* Agents connecting through the wrong interface
* Dashboard access failures
* Return traffic using the wrong route
* Internal hostnames resolving incorrectly
* Docker publishing services on unintended interfaces

The network configuration should be validated before installing Wazuh.

---

## Ubuntu Server Installation

1. Create the VMware virtual machine.
2. Attach the Ubuntu Server ISO.
3. Start the VM.
4. Select the language and keyboard layout.
5. Configure the initial network interfaces.
6. Use DHCP temporarily where necessary.
7. Configure the storage layout.
8. Create a non-root administrative user.
9. Install the OpenSSH server when remote administration is required.
10. Complete the installation.
11. Restart the VM.
12. Install VMware Tools or the supported open-vm-tools package.
13. Apply operating system updates.
14. Confirm both network adapters are visible.
15. Create a clean baseline snapshot.

Do not publish the real administrative username or password.

---

## Administrative Account

Use a normal Linux user with `sudo` privileges for administration.

Public placeholder:

```
<ADMIN_USER>
```

Routine commands should run as the normal user.

Use `sudo` only for actions requiring elevated permissions, such as:

* Package installation
* Network configuration
* Service management
* Docker administration
* Firewall changes
* Wazuh configuration
* Protected log access

Avoid using the root account for normal daily administration.

---

## Verify the Operating System

```
hostnamectl
```

```
cat /etc/os-release
```

```
uname -a
```

Review:

* Hostname
* Ubuntu version
* Kernel version
* Architecture
* Virtualization status

Sanitize command output before publishing it because it may contain internal identifiers.

---

## Set the Hostname

Set the public example hostname:

```
sudo hostnamectl set-hostname WAZUH-SERVER
```

Update `/etc/hosts` if required:

```
sudo nano /etc/hosts
```

Example structure:

```
127.0.0.1       localhost
127.0.1.1       WAZUH-SERVER
```

Restart or open a new session after changing the hostname.

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

Interface names vary and may appear similar to:

```
ens33
ens37
```

Do not assume interface names without checking.

---

## Example Network Design

```
Host-only interface:
Address: 192.0.2.30/24
Gateway: none
Purpose: Internal Wazuh traffic

NAT interface:
Address: VMware DHCP
Gateway: VMware NAT gateway
Purpose: Internet access
```

Only the NAT interface should normally have a default gateway.

---

## Netplan Configuration

Ubuntu Server commonly uses Netplan.

Review available configuration files:

```
ls -l /etc/netplan
```

Open the active file:

```
sudo nano /etc/netplan/<CONFIGURATION_FILE>.yaml
```

Example sanitized configuration:

```yaml
network:
  version: 2
  ethernets:
    <HOST_ONLY_INTERFACE>:
      dhcp4: false
      addresses:
        - 192.0.2.30/24
      nameservers:
        addresses:
          - 192.0.2.10

    <NAT_INTERFACE>:
      dhcp4: true
```

The exact DNS configuration depends on whether the Wazuh server must resolve the internal Active Directory domain.

---

## Apply Netplan Safely

Use:

```
sudo netplan try
```

This provides a temporary validation period and rollback opportunity.

After confirming connectivity:

```
sudo netplan apply
```

Verify:

```
ip address
ip route
resolvectl status
```

Expected routing design:

```
default via <VMWARE_NAT_GATEWAY> dev <NAT_INTERFACE>
192.0.2.0/24 dev <HOST_ONLY_INTERFACE>
```

---

## Validate Internal Connectivity

Test DC01:

```
ping -c 4 192.0.2.10
```

Test the Windows endpoint:

```
ping -c 4 192.0.2.20
```

Test internal DNS when configured:

```
resolvectl query DC01.cyberlab.example
```

A failed ping does not necessarily mean the target service is unavailable because host firewalls may block ICMP.

---

## Validate Internet Connectivity

Test an external address:

```
ping -c 4 1.1.1.1
```

Test DNS:

```
resolvectl query example.com
```

Test HTTPS:

```
curl -I https://example.com
```

If IP connectivity works but DNS does not, investigate resolver configuration rather than routing.

---

## Update Ubuntu

```
sudo apt update
```

```
sudo apt upgrade
```

Review the packages before confirming major upgrades.

Restart when required:

```
sudo reboot
```

After restart, confirm:

```
ip address
ip route
```

---

## Install Required Utilities

Useful administration tools may include:

```
sudo apt install curl wget unzip ca-certificates gnupg lsb-release
```

Additional troubleshooting tools:

```
sudo apt install net-tools dnsutils traceroute tcpdump
```

Install only tools required for the lab.

---

## Time Synchronization

Accurate time is essential for:

* Alert timestamps
* Event correlation
* Agent communication
* Certificate validation
* Incident timelines
* Windows domain activity comparison

Review time:

```
timedatectl
```

Review synchronization:

```
timedatectl timesync-status
```

If the server uses another time service, review that service instead.

---

## Timezone Configuration

Set the correct timezone:

```
sudo timedatectl set-timezone <TIMEZONE>
```

Verify:

```
timedatectl
```

The public repository may use a placeholder rather than publishing location-specific information.

---

## Install Docker

The Wazuh deployment in this lab uses containers.

Install Docker through the supported repository or another trusted method documented for the environment.

After installation, verify:

```
docker --version
```

```
docker compose version
```

Review service status:

```
sudo systemctl status docker
```

---

## Docker Permissions

Docker normally requires root privileges.

A user may be added to the Docker group:

```
sudo usermod -aG docker <ADMIN_USER>
```

Sign out and sign back in before testing.

Verify:

```
docker ps
```

Membership in the Docker group effectively grants root-level control over the host.

Only trusted administrative users should receive this access.

---

## Docker Security Considerations

Docker access can allow a user to:

* Mount host directories
* Modify containers
* Access sensitive files
* Start privileged containers
* Control service availability
* Potentially gain root-level host access

Recommended practices:

* Limit Docker group membership.
* Do not expose the Docker API over the network.
* Use trusted container images.
* Review Compose files before use.
* Keep Docker updated.
* Avoid privileged containers unless required.
* Protect configuration files containing credentials.

---

## Wazuh Deployment Method

The lab uses the official Wazuh Docker deployment files as the basis for an all-in-one installation.

The exact repository version should be selected deliberately and recorded privately.

General workflow:

1. Obtain the supported deployment files.
2. Review the Compose configuration.
3. Generate required certificates.
4. Review generated credentials.
5. Start the Wazuh stack.
6. Confirm all containers remain healthy.
7. Access the dashboard.
8. Change default credentials when applicable.
9. Validate indexing and alerts.
10. Create a stable snapshot.

---

## Create a Deployment Directory

Example:

```
mkdir -p ~/wazuh-docker
```

Move into the directory:

```
cd ~/wazuh-docker
```

The real operational path may differ.

Do not publish paths containing a personal username.

---

## Obtain the Deployment Files

Use the official Wazuh repository or approved release package.

Example placeholder workflow:

```
git clone <OFFICIAL_WAZUH_REPOSITORY>
```

Move into the single-node deployment directory:

```
cd <WAZUH_DEPLOYMENT_DIRECTORY>
```

Review all files before running them.

Do not blindly execute scripts from an unverified source.

---

## Version Control Considerations

Record the deployed version because installation files and configuration formats may change.

Private change records should include:

```
Wazuh version:
Deployment method:
Ubuntu version:
Docker version:
Docker Compose version:
Installation date:
Snapshot name:
```

Avoid committing operational credentials or generated private keys.

---

## Review the Compose File

Before starting the stack:

```
less docker-compose.yml
```

Or:

```
nano docker-compose.yml
```

Review:

* Image versions
* Container names
* Port publishing
* Volume mappings
* Environment variables
* Password values
* Network definitions
* Certificate paths
* Resource limits

Do not publish the operational Compose file without sanitizing secrets.

---

## Generate Certificates

The deployment may include a certificate-generation Compose file or script.

General example:

```
docker compose -f generate-indexer-certs.yml run --rm generator
```

The exact filename and command depend on the selected Wazuh release.

Generated files may include:

* Certificate authority files
* Indexer certificates
* Dashboard certificates
* Manager certificates
* Private keys

Private keys must never be committed to GitHub.

---

## Certificate File Protection

Review certificate permissions:

```
find . -type f -maxdepth 4 -ls
```

Protect sensitive files from unnecessary access.

Do not publish:

* Private keys
* Certificate passwords
* Keystores
* Truststore passwords
* Internal certificate details that reveal operational names
* Generated secrets

Public documentation may explain the process without including the files.

---

## Start the Wazuh Stack

From the deployment directory:

```
docker compose up -d
```

This starts the containers in detached mode.

Depending on local permissions, `sudo` may be required:

```
sudo docker compose up -d
```

Use normal-user Docker access only when the user has intentionally been granted Docker group membership.

---

## Review Container Status

```
docker compose ps
```

Or:

```
docker ps
```

Expected components should remain running.

If a container repeatedly restarts, inspect its logs before retrying the installation.

---

## Review Container Logs

All services:

```
docker compose logs
```

Follow logs:

```
docker compose logs -f
```

Specific service:

```
docker compose logs <SERVICE_NAME>
```

Limit output when practical:

```
docker compose logs --tail 100 <SERVICE_NAME>
```

Logs may contain:

* Internal addresses
* Certificate names
* Usernames
* Configuration paths
* Error details
* Authentication failures

Sanitize before publication.

---

## Validate Listening Ports

Review host listeners:

```
sudo ss -tulpn
```

Review Docker-published ports:

```
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Confirm services are listening only where required.

A port exposed on `0.0.0.0` may be reachable through every active server interface unless blocked by the firewall.

---

## Dashboard Access

Access the dashboard from the Acer host through the internal host-only address.

Public example:

```
https://<WAZUH_SERVER>
```

The exact port may depend on the deployment.

A self-signed certificate warning may appear in a lab.

Verify:

* The address is the intended internal server.
* The certificate presented belongs to the deployment.
* The connection is not being intercepted.
* The dashboard is not exposed to the public internet.

---

## Initial Credentials

The deployment process may generate or include initial dashboard credentials.

These should be:

* Stored in a password manager
* Changed when supported
* Excluded from screenshots
* Excluded from shell history
* Excluded from GitHub
* Unique to the lab

Do not leave default credentials active longer than necessary.

---

## Credential Rotation

When changing credentials:

1. Record the affected component.
2. Back up the relevant configuration.
3. Change one credential set at a time.
4. Restart only the required services.
5. Validate the dashboard.
6. Validate indexer communication.
7. Validate agent reporting.
8. Update the password manager.
9. Remove temporary notes.
10. Record the result.

Wazuh deployments may use multiple internal credentials.

Changing one value without updating dependent components can break the stack.

---

## Host Firewall

Ubuntu firewall controls should be reviewed before exposing any management service.

Check status:

```
sudo ufw status verbose
```

If UFW is used, permit only required traffic from the host-only subnet.

Example structure:

```
sudo ufw allow from 192.0.2.0/24 to any port <DASHBOARD_PORT> proto tcp
```

```
sudo ufw allow from 192.0.2.0/24 to any port <AGENT_PORT> proto tcp
```

Use the actual required protocol for each service.

Do not copy placeholder rules without confirming the deployment requirements.

---

## Default-Deny Firewall Strategy

A controlled lab firewall strategy may include:

```
sudo ufw default deny incoming
```

```
sudo ufw default allow outgoing
```

Then add narrow rules for:

* SSH from the trusted host
* Dashboard access from the host-only network
* Agent communication from monitored endpoints
* Enrollment traffic when required

Enable only after confirming that the required management rule exists:

```
sudo ufw enable
```

A firewall mistake can disconnect SSH administration.

VMware console access should remain available for recovery.

---

## SSH Administration

SSH provides remote command-line administration from the Acer host.

Review status:

```
sudo systemctl status ssh
```

Review listening state:

```
sudo ss -tulpn | grep ssh
```

Recommended controls include:

* Restrict SSH to the host-only network.
* Use a non-root administrative account.
* Disable direct root login.
* Use key-based authentication where practical.
* Protect private keys.
* Disable password authentication only after key access is validated.

---

## SSH Hardening

Review:

```
sudo nano /etc/ssh/sshd_config
```

Relevant settings may include:

```
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
```

Do not disable password authentication until key-based access has been tested.

Validate the configuration:

```
sudo sshd -t
```

Restart:

```
sudo systemctl restart ssh
```

Keep the VMware console open during SSH configuration changes.

---

## Verify Wazuh Services

Review container status:

```
docker compose ps
```

Review recent logs:

```
docker compose logs --tail 50
```

Validate the dashboard in a browser.

Confirm:

* Login succeeds.
* The dashboard loads.
* The indexer is reachable.
* The manager is available.
* No container is restarting.
* Alerts can be viewed.
* Agent management is accessible.

---

## Wazuh Agent Deployment

The Windows endpoint requires a Wazuh agent.

The agent is installed on `WIN11TARGET` and configured to communicate with the Wazuh server’s internal address.

General workflow:

1. Confirm the Wazuh manager is running.
2. Confirm the endpoint can reach the server.
3. Obtain the supported Windows agent package.
4. Install the agent.
5. Configure the manager address.
6. Enroll or register the endpoint.
7. Start the agent service.
8. Confirm the endpoint appears in the dashboard.
9. Generate a test event.
10. Validate alert ingestion.

---

## Agent Communication Boundary

The endpoint should communicate with the internal Wazuh address.

```
WIN11TARGET
     |
     | Agent telemetry
     v
WAZUH-SERVER host-only interface
```

Do not configure the endpoint to use:

* The server’s NAT address
* A public IP address
* A bridged interface
* An internet-exposed hostname

The agent path should remain inside the CyberLab.

---

## Windows Agent Installation

Obtain the supported Windows agent installer from an official source.

The installation may be performed through:

* Graphical installer
* PowerShell
* Command Prompt
* Software deployment tooling
* Domain policy in a future exercise

The package version should be compatible with the Wazuh server.

---

## Example Windows Agent Installation

A generic silent installation pattern may resemble:

```
msiexec.exe /i "<WAZUH_AGENT_INSTALLER>" /q `
    WAZUH_MANAGER="<WAZUH_SERVER>" `
    WAZUH_AGENT_NAME="WIN11TARGET"
```

The exact installer properties depend on the agent version and deployment method.

Run from an elevated PowerShell or Command Prompt.

Do not place enrollment passwords or keys directly into documentation.

---

## Verify the Windows Agent Service

On the endpoint:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm that the service:

* Exists
* Is configured appropriately
* Is running

Start when required:

```
Start-Service <WAZUH_AGENT_SERVICE>
```

Administrator privileges are required.

---

## Agent Configuration File

The Windows agent configuration is stored in a protected installation directory.

It may contain:

* Manager address
* Event channels
* File integrity paths
* Local log definitions
* Agent settings
* Enrollment information

Do not publish the complete operational configuration without sanitization.

Before editing:

1. Back up the file.
2. Record the current agent status.
3. Make one change.
4. Validate the configuration.
5. Restart the agent.
6. Confirm telemetry returns.

---

## Agent Enrollment

Enrollment associates the endpoint with the manager.

Depending on the deployment, this may use:

* Authentication service
* Enrollment password
* Generated key
* Preconfigured installer values
* Dashboard-assisted deployment command

Protect:

* Enrollment passwords
* Registration keys
* Agent keys
* Deployment commands containing credentials
* Temporary tokens

A screenshot of an agent deployment command should be reviewed carefully before publication.

---

## Verify Agent Enrollment

In the Wazuh dashboard, confirm:

* The endpoint appears.
* The agent name is correct.
* The operating system is recognized.
* The status is active.
* The last keepalive is recent.
* The manager address is correct.
* No duplicate agent exists.

On the manager, agent status may also be reviewed through Wazuh administration tools or container commands.

---

## Duplicate Agent Identity

Duplicate agent names or keys can cause:

* One endpoint replacing another
* Repeated disconnects
* Incorrect inventory
* Alerts assigned to the wrong system
* Enrollment failures

Before re-enrolling an endpoint:

1. Confirm whether the old record is still needed.
2. Record the previous agent identifier privately.
3. Remove only the stale record.
4. Reinstall or re-enroll the endpoint.
5. Confirm a stable active status.

Do not publish real agent keys.

---

## Endpoint Event Collection

The Wazuh agent can collect selected Windows event channels.

Important sources may include:

* Security
* System
* Application
* Windows Defender Operational
* PowerShell Operational
* Sysmon Operational when installed
* Group Policy Operational
* Windows Firewall logs where configured

Event collection should match the lab’s detection goals.

Collecting every channel without planning can increase noise and storage usage.

---

## Windows Event Channel Validation

Generate a harmless event on the endpoint.

Example:

```
whoami
```

Or:

```
Get-Service |
    Select-Object -First 5
```

Then verify:

1. The event exists locally.
2. The agent is active.
3. The event reaches Wazuh.
4. The timestamp is correct.
5. The endpoint name is correct.
6. Relevant rule or decoder information appears.
7. The result is documented.

---

## File Integrity Monitoring

Wazuh file integrity monitoring can detect changes to selected files and directories.

Suitable lab paths include:

```
C:\CyberLab-Test\Files
```

Avoid initially monitoring:

* Entire system drives
* Large software directories
* Browser caches
* Temporary directories
* Frequently changing log directories
* SIEM storage paths

Overly broad monitoring can create excessive events.

---

## File Integrity Test

On the endpoint, create a file:

```
New-Item `
    -Path "C:\CyberLab-Test\Files\wazuh-test.txt" `
    -ItemType File `
    -Force
```

Add content:

```
Set-Content `
    -Path "C:\CyberLab-Test\Files\wazuh-test.txt" `
    -Value "Authorized Wazuh file integrity test."
```

Modify it:

```
Add-Content `
    -Path "C:\CyberLab-Test\Files\wazuh-test.txt" `
    -Value "Controlled modification."
```

Delete it after validation:

```
Remove-Item `
    -Path "C:\CyberLab-Test\Files\wazuh-test.txt"
```

Confirm the expected creation, modification, and deletion events.

---

## File Integrity Considerations

File integrity events should be evaluated for:

* File path
* Change type
* User context
* Before and after hash
* File size
* Timestamp
* Endpoint
* Rule level
* Frequency

A file change is not automatically malicious.

Context determines whether the activity is expected, suspicious, or unauthorized.

---

## Security Configuration Assessment

Wazuh Security Configuration Assessment can compare endpoint settings against predefined policies.

Potential checks include:

* Password settings
* Audit configuration
* Service state
* Registry values
* Firewall configuration
* Operating system settings
* Security controls

Assessment results should be treated as guidance rather than automatic proof of vulnerability.

A failed check may reflect:

* Lab design
* Unsupported operating system version
* Intentional configuration
* Policy mismatch
* Missing data
* A real hardening opportunity

---

## Vulnerability Visibility

Wazuh may identify software vulnerabilities based on:

* Installed package inventory
* Operating system data
* Vulnerability feeds
* Package versions
* Vendor advisories

Results should be validated before remediation.

Potential false or misleading results may occur because of:

* Backported security patches
* Vendor package naming
* Delayed feed updates
* Unsupported software
* Incomplete inventory
* Version parsing

Do not publicly publish a detailed vulnerability list tied to the operational lab.

---

## MITRE ATT&CK Mapping

Some Wazuh alerts include MITRE ATT&CK metadata.

This can help relate activity to:

* Tactics
* Techniques
* Detection objectives
* Investigation context
* Exercise design

A MITRE mapping does not automatically prove that an attack occurred.

It indicates that the detected behavior may relate to a known adversary technique.

---

## Compliance Metadata

Wazuh rules may include references to frameworks or standards such as:

* NIST
* PCI DSS
* HIPAA
* GDPR
* CIS
* Other policy mappings

These tags are useful for learning how technical events relate to governance and control frameworks.

They do not make the personal CyberLab formally compliant with those standards.

---

## Alert Levels

Wazuh uses rule levels to communicate severity or importance.

Higher levels generally indicate more significant activity.

However, analysts should not rely only on the numeric level.

Review:

* Event source
* User context
* System role
* Frequency
* Rule description
* Related events
* Known test activity
* Whether the event was expected

A lower-level event may still be relevant during an investigation.

---

## Dashboard Investigation Workflow

A basic investigation workflow is:

1. Identify the alert.
2. Record the timestamp.
3. Confirm the endpoint.
4. Review the rule description.
5. Review the raw event.
6. Identify the user or process.
7. Check related alerts.
8. Compare with the planned exercise.
9. Determine whether the activity is expected.
10. Record the conclusion.

Screenshots should show only sanitized information.

---

## Search and Filtering

Useful dashboard filters may include:

* Agent name
* Rule identifier
* Rule level
* Source IP
* Destination IP
* Username
* Event channel
* Process name
* File path
* MITRE technique
* Time range

Use neutral placeholder values when documenting searches publicly.

---

## Test Alert Generation

Safe test activity may include:

* Successful domain sign-in
* Failed domain sign-in
* Account lockout
* File creation
* File modification
* Harmless PowerShell execution
* Defender antivirus test event
* Group membership change
* Service state change
* Controlled network scan

Each test should include cleanup and validation steps.

---

## Failed Logon Validation

1. Use a dedicated test account.
2. Enter an incorrect password once.
3. Record the time.
4. Confirm the event locally.
5. Search Wazuh for the endpoint and timeframe.
6. Review the account name.
7. Review the source workstation.
8. Review the failure reason.
9. Confirm the activity was authorized.
10. Document the result.

Avoid locking out administrative accounts.

---

## Account Lockout Validation

1. Confirm the domain lockout policy.
2. Use a non-administrative test account.
3. Record the initial account state.
4. Generate the required failed attempts.
5. Confirm the account becomes locked.
6. Review DC01 and endpoint events.
7. Search Wazuh.
8. Identify the originating system.
9. Unlock the account after the investigation.
10. Document cleanup.

---

## PowerShell Validation

Run a harmless command:

```
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 5
```

Review:

* Process creation telemetry
* PowerShell event data
* User context
* Parent process
* Command content
* Wazuh rule mapping
* Alert level
* Timestamp

The exact result depends on enabled Windows auditing and agent configuration.

---

## Controlled Network Activity

Kali Linux may generate authorized traffic against `WIN11TARGET`.

Examples include:

* Ping
* Limited port scan
* Service connection test
* DNS query
* Authentication test

The objective is to validate whether the monitored endpoint or Wazuh records useful telemetry.

Do not scan systems outside the CyberLab.

---

## Wazuh Manager Configuration

Wazuh manager configuration should be changed only after creating a backup.

Potential settings include:

* Agent communication
* Enrollment
* Local file monitoring
* Rule directories
* Decoder directories
* Active response
* Cluster settings
* Log collection
* Email or notification integration

The exact configuration path depends on the deployment method.

In a container deployment, configuration may be bind-mounted from the host.

---

## Back Up Configuration Before Changes

Example:

```
cp <CONFIGURATION_FILE> <CONFIGURATION_FILE>.bak
```

For protected files:

```
sudo cp <CONFIGURATION_FILE> <CONFIGURATION_FILE>.bak
```

Record:

```
Date:
File:
Reason:
Original checksum:
Change:
Validation:
Rollback:
```

---

## Validate Wazuh Configuration

Use the validation method supported by the deployed version and component.

Before restarting:

* Check XML or YAML syntax.
* Review indentation.
* Confirm closing tags.
* Confirm file permissions.
* Confirm referenced paths exist.
* Confirm secret values are protected.

A syntax error can prevent a service from starting.

---

## Custom Rules

Custom Wazuh rules can support detection engineering practice.

Good custom rule development includes:

* A clear objective
* A documented data source
* A specific condition
* A meaningful severity
* Test events
* False-positive analysis
* Validation evidence
* Version control
* Rollback instructions

Custom rules should be stored separately from vendor-managed defaults where supported.

---

## Custom Rule Naming

Use a consistent rule identifier range reserved for local rules.

Example naming:

```
LAB - Repeated Failed Logons
LAB - Suspicious PowerShell Test
LAB - Monitored File Modified
LAB - Test Administrator Group Change
```

Avoid vague names such as:

```
Test Rule
Rule 1
New Alert
```

---

## Rule Testing

A custom rule should be tested against:

* Expected matching event
* Expected non-matching event
* Normal administrative behavior
* Repeated activity
* Different user contexts
* Different endpoints
* Missing fields

Document false positives and limitations.

---

## Restarting the Wazuh Stack

Restart the entire stack only when necessary:

```
docker compose restart
```

Restart a single service when possible:

```
docker compose restart <SERVICE_NAME>
```

Review status afterward:

```
docker compose ps
```

Review logs:

```
docker compose logs --tail 100 <SERVICE_NAME>
```

---

## Starting Wazuh After Host Reboot

After the Ubuntu VM starts:

```
cd <WAZUH_DEPLOYMENT_DIRECTORY>
```

Review status:

```
docker compose ps
```

Start when required:

```
docker compose up -d
```

If the Docker service is not running:

```
sudo systemctl start docker
```

Then start the stack again.

---

## Automatic Container Startup

Compose services may use restart policies.

Review the Compose file for settings such as:

```yaml
restart: always
```

Or:

```yaml
restart: unless-stopped
```

Automatic restart improves availability but does not eliminate the need for health validation.

A container can be running while the application inside it is unhealthy.

---

## Wazuh Startup Validation

After starting the VM:

1. Confirm the internal address.
2. Confirm the default route.
3. Confirm Docker is running.
4. Confirm all Wazuh containers are running.
5. Review recent errors.
6. Open the dashboard.
7. Confirm the indexer is available.
8. Confirm the manager is available.
9. Confirm agents reconnect.
10. Generate or locate a recent test event.

---

## Wazuh Shutdown Procedure

1. End active exercises.
2. Confirm endpoint events have been received.
3. Save investigation evidence.
4. Record active alerts when required.
5. Move into the deployment directory.
6. Stop the stack cleanly.
7. Confirm containers are stopped.
8. Shut down Ubuntu.
9. Confirm the VM is powered off.
10. Create a stable snapshot when appropriate.

Stop the stack:

```
docker compose down
```

Do not use commands that delete volumes unless intentional data removal is part of a documented reset.

---

## Docker Volume Warning

Commands such as the following may delete stored data:

```
docker compose down -v
```

The `-v` option removes associated named volumes.

This can erase:

* Index data
* Configuration state
* Certificates
* Application data
* Agent-related state

Do not use volume-deleting commands as a routine shutdown method.

---

## Snapshot Strategy

Recommended milestones:

```
01-WAZUH-Ubuntu-Clean-Install
02-WAZUH-Ubuntu-Patched
03-WAZUH-Network-Configured
04-WAZUH-Docker-Installed
05-WAZUH-Stack-Deployed
06-WAZUH-Dashboard-Validated
07-WAZUH-Agent-Enrolled
08-WAZUH-Windows-Events-Validated
09-WAZUH-FIM-Validated
10-WAZUH-Exercise-Ready
```

Create important snapshots while the VM is powered off.

---

## Safe Snapshot Procedure

1. End active tests.
2. Confirm all required events are indexed.
3. Record the current stack status.
4. Stop the Wazuh stack cleanly.
5. Shut down Ubuntu.
6. Confirm the VM is powered off.
7. Create the snapshot.
8. Add a descriptive note.
9. Start the VM.
10. Validate the complete stack.

---

## Snapshot Restore Risks

Restoring a Wazuh snapshot may cause:

* Lost recent alerts
* Agent key mismatch
* Duplicate agent state
* Certificate inconsistency
* Time rollback
* Index corruption
* Stale dashboard data
* Missing custom-rule changes
* Conflicting endpoint enrollment

After restoration:

1. Confirm system time.
2. Confirm network configuration.
3. Confirm Docker.
4. Confirm all containers.
5. Review logs.
6. Confirm dashboard access.
7. Confirm agents reconnect.
8. Generate a new test event.
9. Confirm index health.
10. Document the restore.

---

## Configuration Backup

Important backup targets include:

* Compose files
* Environment files
* Certificate configuration
* Custom rules
* Custom decoders
* Manager configuration
* Agent deployment notes
* Dashboard objects where export is supported
* Investigation queries
* Change records

Secrets and private keys should be backed up securely but not included in the public repository.

---

## Cold Backup Procedure

1. Stop active testing.
2. Stop the Wazuh stack.
3. Shut down the Ubuntu VM.
4. Copy or export the powered-off VM.
5. Store the backup on separate media.
6. Record the milestone.
7. Verify the backup files.
8. Restart the original VM.
9. Validate Wazuh.
10. Periodically test restoration.

---

## Exporting Configuration

A sanitized configuration export may be stored in GitHub when it contains:

* No passwords
* No private keys
* No real addresses
* No personal usernames
* No active tokens
* No internal domain names
* No sensitive paths

Use `.example` files for templates:

```
docker-compose.example.yml
manager-config.example.xml
agent-config.example.xml
.env.example
```

---

## Git Ignore Requirements

The repository should exclude sensitive or generated files.

Example `.gitignore` entries:

```gitignore
.env
*.key
*.pem
*.p12
*.jks
*.keystore
secrets/
certificates/
config/wazuh_indexer_ssl_certs/
backups/
logs/
data/
volumes/
```

Review the repository before every public push.

---

## Disk Usage Monitoring

Review filesystem usage:

```
df -h
```

Review directory sizes:

```
du -sh <WAZUH_DEPLOYMENT_DIRECTORY>
```

Docker usage:

```
docker system df
```

Detailed Docker usage:

```
docker system df -v
```

Do not delete Docker resources without confirming whether Wazuh depends on them.

---

## Docker Cleanup Warning

Commands such as:

```
docker system prune
```

Can remove unused:

* Containers
* Networks
* Images
* Build cache

More aggressive options may also remove volumes.

Before cleanup:

1. Confirm the Wazuh stack is backed up.
2. Review what Docker considers unused.
3. Confirm required images can be downloaded again.
4. Avoid deleting volumes.
5. Record the change.
6. Validate Wazuh afterward.

---

## Resource Monitoring

Review server resources:

```
free -h
```

```
top
```

Or:

```
htop
```

Review containers:

```
docker stats
```

Monitor:

* CPU
* Memory
* Disk usage
* Container restarts
* Indexing delays
* Dashboard responsiveness

---

## Log Rotation

Logs that grow without limits can consume the virtual disk.

Review:

* Docker logging driver
* Container log size
* Ubuntu system logs
* Wazuh internal logs
* Index retention
* Alert retention

Any retention change should consider the educational need to preserve investigation history.

---

## Troubleshooting: Dashboard Does Not Load

Check:

1. The Ubuntu VM is running.
2. The internal address is correct.
3. The Acer host can reach the server.
4. Docker is running.
5. Wazuh containers are running.
6. The dashboard service is listening.
7. The host firewall permits access.
8. The browser is using the correct protocol and address.
9. The indexer is healthy.
10. Certificate errors are understood.

Commands:

```
docker compose ps
```

```
docker compose logs --tail 100 <DASHBOARD_SERVICE>
```

```
sudo ss -tulpn
```

---

## Troubleshooting: Container Repeatedly Restarts

Review:

```
docker ps
```

```
docker inspect <CONTAINER_NAME>
```

```
docker logs --tail 200 <CONTAINER_NAME>
```

Possible causes include:

* Invalid configuration
* Missing certificate
* Incorrect credentials
* File permission problem
* Low memory
* Low disk space
* Port conflict
* Dependency unavailable
* Corrupted data

Do not repeatedly recreate the stack before identifying the failure.

---

## Troubleshooting: Indexer Is Unhealthy

Possible symptoms include:

* Dashboard login failure
* Alerts not appearing
* Search errors
* Red or unavailable index status
* Container restart loop

Check:

* Indexer logs
* Available memory
* Disk space
* Certificate configuration
* Internal credentials
* File ownership
* System time
* Container network
* Index corruption

The indexer typically has the highest resource demand in the all-in-one stack.

---

## Troubleshooting: Agent Does Not Appear

Check:

* The endpoint installer completed.
* The agent service exists.
* The service is running.
* The manager address is correct.
* Enrollment succeeded.
* The endpoint reaches the server.
* Required firewall rules exist.
* The agent name is unique.
* The agent key is valid.
* The manager is accepting connections.

Endpoint:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Connectivity:

```
Test-NetConnection <WAZUH_SERVER> -Port <AGENT_PORT>
```

---

## Troubleshooting: Agent Shows Disconnected

Possible causes include:

* Endpoint powered off
* Wazuh manager unavailable
* Network adapter disconnected
* Firewall rule changed
* Agent service stopped
* Duplicate identity
* Time problem
* Manager address changed
* Restored snapshot
* Corrupted agent key

Restart the endpoint agent only after reviewing the local logs and network path.

---

## Troubleshooting: Events Do Not Appear

Check:

1. The event exists locally.
2. The relevant channel is enabled.
3. The agent is active.
4. The agent configuration includes the source.
5. The manager receives the event.
6. A decoder recognizes it.
7. A rule matches it.
8. The indexer stores it.
9. The dashboard time filter includes it.
10. System clocks are aligned.

A missing alert does not always mean the raw event was not collected.

---

## Troubleshooting: File Integrity Event Is Missing

Check:

* The monitored path is correct.
* The agent configuration was reloaded.
* The agent service restarted successfully.
* The directory exists.
* The user changed the intended file.
* The scan interval has elapsed.
* Real-time monitoring is enabled when expected.
* The event is not excluded.
* The dashboard time filter is correct.

Use a simple dedicated test file to validate the path.

---

## Troubleshooting: Wazuh Has No Internet Access

Check:

```
ip address
ip route
resolvectl status
```

Confirm:

* The NAT adapter is connected.
* It has a DHCP address.
* It provides the default route.
* VMware NAT is functioning.
* The Acer host has internet access.
* The host VPN is not blocking VMware.
* DNS resolves external names.

Do not add a gateway to the host-only adapter.

---

## Troubleshooting: Internal Systems Cannot Reach Wazuh

Check:

* Host-only adapter state
* Static address
* Subnet
* Ubuntu firewall
* Docker-published ports
* Service binding
* VMware VMnet selection
* Windows endpoint firewall
* Host VPN routing

Test from Windows:

```
Test-NetConnection <WAZUH_SERVER> -Port <PORT>
```

Test locally on Ubuntu:

```
sudo ss -tulpn
```

---

## Troubleshooting: DNS Uses the Wrong Interface

Review:

```
resolvectl status
```

```
ip route
```

Possible causes include:

* NAT DHCP DNS overriding internal DNS
* Incorrect Netplan nameserver configuration
* Multiple default routes
* Interface metric behavior
* Search domain configuration

Configure DNS according to the server’s actual needs and validate internal and external resolution separately.

---

## Troubleshooting: Disk Space Is Low

Review:

```
df -h
```

```
docker system df
```

```
sudo du -xh /var/lib/docker |
    sort -h |
    tail
```

Potential causes include:

* Index growth
* Docker logs
* Old container images
* System logs
* Backups stored inside the VM
* Snapshot growth on the host
* Package cache

Do not delete index data or Docker volumes without a recovery plan.

---

## Troubleshooting: High Memory Use

Review:

```
free -h
```

```
docker stats
```

Potential causes include:

* Indexer workload
* Dashboard searches
* Excessive event volume
* Too many retained indexes
* Too little VM memory
* Another service consuming resources
* Container restart loop

Possible corrective actions include:

* Stop unnecessary VMs
* Increase Wazuh VM memory
* Reduce unnecessary event collection
* Review index retention
* Restart an unhealthy service
* Review host resource pressure

---

## Troubleshooting: Certificate Error

Certificate errors may be caused by:

* Self-signed lab certificate
* Hostname mismatch
* Expired certificate
* Incorrect system time
* Missing certificate
* Wrong permissions
* Certificate generated for another name
* Browser cache

Do not automatically ignore every certificate warning.

Confirm the server identity and expected certificate details.

---

## Troubleshooting: Login Fails

Check:

* Username and password
* Keyboard layout
* Caps Lock
* Dashboard service
* Indexer availability
* Internal credential synchronization
* Password rotation changes
* Browser session state
* Container logs

Avoid repeated guessing that could lock an account or hide the underlying problem.

---

## Troubleshooting: Docker Permission Denied

Symptom:

```
permission denied while trying to connect to the Docker daemon socket
```

Options include:

```
sudo docker ps
```

Or confirm intentional Docker group membership:

```
groups
```

After adding a user to the Docker group, sign out and sign back in.

Remember that Docker group access is effectively administrative.

---

## Change Management

Document significant Wazuh changes, including:

* Version upgrades
* Compose changes
* Credential rotation
* Certificate replacement
* Agent additions
* Agent removal
* Custom rules
* Custom decoders
* File integrity paths
* Firewall changes
* Resource changes
* Retention changes
* Snapshot restoration

Example record:

```
Date:
Change:
Reason:
Components affected:
Backup or snapshot:
Validation:
Rollback plan:
Outcome:
```

---

## Upgrade Planning

Before upgrading Wazuh:

1. Review supported upgrade paths.
2. Record the current version.
3. Back up configuration.
4. Back up custom rules and decoders.
5. Export important dashboard objects.
6. Confirm free disk space.
7. Create a powered-off snapshot.
8. Review release notes privately.
9. Perform the upgrade.
10. Validate every component and agent.

Do not upgrade immediately before an important lab exercise.

---

## Public Sanitization Standards

Remove or replace:

* Real internal IP addresses
* Real domain names
* Administrative usernames
* Dashboard usernames
* Passwords
* Enrollment passwords
* Agent keys
* Registration keys
* API credentials
* Environment files
* Certificate private keys
* Keystore passwords
* Active tokens
* Host-specific paths
* VMware addresses
* Public IP addresses
* MAC addresses
* Home-network information
* Browser profile information
* Raw vulnerability inventories
* Agent identifiers when sensitive

Use placeholders such as:

```
<WAZUH_SERVER>
<WAZUH_SERVER_IP>
<WINDOWS_ENDPOINT>
<ADMIN_USER>
<DASHBOARD_USER>
<AGENT_PORT>
<ENROLLMENT_PORT>
<REDACTED_PASSWORD>
<REDACTED_KEY>
<DEPLOYMENT_DIRECTORY>
```

---

## Screenshot Sanitization

Before publishing dashboard or terminal screenshots:

1. Use a neutral test account.
2. Hide browser bookmarks.
3. Close unrelated tabs.
4. Clear notifications.
5. Remove real addresses.
6. Remove usernames.
7. Remove agent identifiers where needed.
8. Remove enrollment commands.
9. Remove raw credentials.
10. Remove internal domain names.
11. Remove unrelated alerts.
12. Verify all redactions are permanent.

Suitable public screenshots may include:

* Sanitized Wazuh overview
* Active agent status
* Harmless Windows event alert
* File integrity alert
* MITRE ATT&CK mapping
* Compliance tag example
* Container status
* Sanitized service validation
* Controlled exercise results

---

## Validation Checklist

### Ubuntu Server

* Ubuntu starts successfully.
* Hostname is correct.
* Time and timezone are correct.
* Host-only address is correct.
* NAT provides the default route.
* No unintended second default route exists.
* SSH works from the trusted host.

### Docker

* Docker service is running.
* Docker Compose works.
* The administrative user has intended permissions.
* No Docker API is exposed.
* Required images are present.
* Storage usage is acceptable.

### Wazuh Stack

* Manager is running.
* Indexer is running.
* Dashboard is running.
* Containers do not repeatedly restart.
* Dashboard login succeeds.
* Index searches work.
* Recent alerts are visible.

### Endpoint Agent

* WIN11TARGET appears in the dashboard.
* Agent status is active.
* Agent name is correct.
* Last keepalive is recent.
* No duplicate agent exists.
* Windows events are collected.

### Detection Validation

* A successful test event is generated.
* The event exists locally.
* The event reaches Wazuh.
* The timestamp is accurate.
* The correct agent is shown.
* The rule information is understandable.
* The result is documented.

### Recovery

* A clean deployment snapshot exists.
* A post-agent snapshot exists.
* Configuration backups exist.
* Secrets are stored securely.
* The startup procedure is documented.
* The shutdown procedure is documented.
* A restore validation process exists.

---

## Skills Demonstrated

This deployment demonstrates:

* Ubuntu Server administration
* VMware multi-adapter networking
* Static IPv4 configuration
* Linux routing
* DNS troubleshooting
* Docker administration
* Docker Compose
* SIEM deployment
* Wazuh administration
* Windows agent enrollment
* Event collection
* File integrity monitoring
* Alert validation
* MITRE ATT&CK interpretation
* Compliance mapping concepts
* Firewall configuration
* Certificate handling
* Log analysis
* Snapshot management
* Backup planning
* Incident investigation
* Technical documentation
* Public sanitization

---

## Lessons Learned

Key lessons include:

* Wazuh requires deliberate memory and storage planning.
* Only the NAT adapter should normally provide the default route.
* The internal host-only address should be used for agent communication.
* A running container is not always a healthy application.
* Indexer failures can make the entire platform appear unavailable.
* Agent enrollment requires both network access and identity management.
* Time alignment is essential for investigation.
* File integrity monitoring should begin with narrow test paths.
* Alert severity should not replace analyst judgment.
* Compliance tags do not prove formal compliance.
* Custom rules require false-positive testing.
* Docker group access should be treated as administrative access.
* Snapshots do not replace configuration and data backups.
* Public screenshots require aggressive credential and address sanitization.

---

## Planned Improvements

Future improvements may include:

* Sysmon event collection
* Custom Wazuh rules
* Custom decoders
* Windows Event Forwarding integration
* Linux endpoint agent
* File integrity tuning
* Active response testing
* Email or messaging alerts
* Detection-as-code workflow
* Rule version control
* Automated Wazuh health checks
* Dashboard export and backup
* Vulnerability validation workflow
* Security configuration baseline tracking
* Additional Windows endpoints
* Threat intelligence enrichment
* Suricata integration
* Zeek integration
* Incident response runbooks
* Automated snapshot documentation
* Centralized configuration backups

---

## Summary

Wazuh provides the central endpoint monitoring and security analytics capability for the Acer Blue Team CyberLab.

A reliable deployment requires:

* Stable Ubuntu networking
* Controlled internet access
* Adequate memory and storage
* Secure Docker administration
* Healthy manager, indexer, and dashboard services
* Protected credentials and certificates
* Successful endpoint enrollment
* Validated event collection
* Narrow and purposeful monitoring
* Documented snapshots and backups
* Repeatable investigation procedures

By validating each layer independently, the CyberLab gains a dependable platform for endpoint monitoring, detection engineering, alert investigation, and defensive security practice.

