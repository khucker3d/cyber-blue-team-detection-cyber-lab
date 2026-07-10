# Splunk Enterprise Setup

## Overview

Splunk Enterprise provides a second security analytics and log investigation platform within the Acer Blue Team CyberLab.

The deployment is used to practice:

* Centralized log collection
* Search Processing Language
* Windows event analysis
* Linux log analysis
* Data ingestion
* Source and sourcetype management
* Dashboard creation
* Detection development
* Event correlation
* Timeline reconstruction
* Security investigation
* SIEM platform administration

Splunk runs on an Ubuntu-based virtual machine hosted through VMware Workstation.

This document describes the sanitized public deployment process. Operational IP addresses, credentials, host-specific paths, license information, usernames, and other sensitive configuration details have been removed or replaced.

---

## Splunk Role in the CyberLab

Splunk is used as a complementary platform to Wazuh.

Wazuh focuses heavily on:

* Endpoint agents
* Security alerts
* File integrity monitoring
* Configuration assessment
* Vulnerability visibility
* Prebuilt security rules

Splunk provides additional practice with:

* Flexible event searches
* Field extraction
* Data normalization
* Dashboard development
* Statistical analysis
* Custom detection logic
* Cross-source correlation
* Investigation workflows

Using both platforms provides experience with different SIEM architectures and analyst workflows.

---

## Public Example Configuration

The public documentation uses placeholder values such as:

```
Server hostname: SPLUNK-SERVER
Internal address: 192.0.2.40
Internal network: 192.0.2.0/24
Web interface: http://<SPLUNK_SERVER>:8000
Administrative user: <SPLUNK_ADMIN>
```

The `192.0.2.0/24` network is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Architecture Placement

Splunk communicates with the Acer host and monitored systems through the VMware host-only network.

A VMware NAT adapter may provide controlled outbound access for installation, updates, or add-on downloads.

```
                              Internet
                                 |
                          VMware NAT Network
                                 |
                          SPLUNK-SERVER
                                 |
                       Host-Only CyberLab
                                 |
             ____________________|____________________
            |                    |                    |
          DC01              WIN11TARGET           Acer Host
     Windows Events       Endpoint Logs       Splunk Web Access
```

The host-only interface supports:

* Splunk Web access
* Forwarder communication
* Log ingestion
* Administrative access
* Internal CyberLab searches

The NAT interface supports:

* Package downloads
* Splunk installation
* Operating system updates
* Add-on downloads
* Time synchronization

---

## Deployment Scope

The CyberLab deployment is a single-instance Splunk Enterprise server.

The same server may perform several roles:

* Search head
* Indexer
* Data input receiver
* Web interface
* Local configuration manager

This is appropriate for a personal learning environment.

It does not provide:

* Indexer clustering
* Search-head clustering
* High availability
* Distributed management
* Enterprise-scale ingestion
* Dedicated deployment server infrastructure
* Production disaster recovery

---

## Splunk Components

| Component        | Purpose                                                  |
| ---------------- | -------------------------------------------------------- |
| Splunk Web       | Browser-based administration, searching, and dashboards  |
| Search processor | Executes Search Processing Language queries              |
| Indexer          | Parses, indexes, and stores incoming events              |
| Data inputs      | Defines how logs enter Splunk                            |
| Splunkd          | Main Splunk service and management process               |
| Forwarder        | Sends logs from monitored systems to Splunk              |
| Apps and add-ons | Extend inputs, fields, dashboards, and knowledge objects |

The initial deployment focuses on the core Splunk Enterprise installation.

---

## Prerequisites

Before installing Splunk, confirm that the following are available:

* VMware Workstation
* An Ubuntu-based virtual machine
* A configured host-only VMware network
* Controlled internet access or an offline installer-transfer method
* Sufficient memory and storage
* Accurate system time
* A non-root Linux administrative user
* `sudo` access
* The official Splunk Linux `.deb` installer
* A secure Splunk administrator password
* A snapshot and recovery plan

---

## Virtual Machine Planning

A dedicated Splunk virtual machine is recommended when host resources permit.

Suggested public VM name:

```
SPLUNK-SERVER
```

General resource considerations:

| Resource           | Guidance                                              |
| ------------------ | ----------------------------------------------------- |
| Virtual processors | Moderate allocation                                   |
| Memory             | Enough for indexing, searching, and Splunk Web        |
| Storage            | Sufficient for indexes, logs, packages, and snapshots |
| Adapter 1          | Host-only                                             |
| Adapter 2          | NAT when required                                     |
| Operating system   | Supported 64-bit Ubuntu release                       |

Splunk can consume substantial disk space as data is indexed.

---

## Storage Considerations

Splunk storage consumption is influenced by:

* Ingestion volume
* Event retention
* Number of indexes
* Search activity
* Internal Splunk logs
* Application installations
* Forwarder data
* Uploaded test datasets
* Snapshot growth

Low disk space can cause:

* Failed indexing
* Search errors
* Service instability
* Data loss
* Slow dashboards
* Failed upgrades
* Virtual machine performance problems

The host and guest storage should both be monitored.

---

## Network Design

A typical Splunk VM may use two adapters.

### Host-Only Adapter

```
Purpose: Internal CyberLab communication
Addressing: Static
Default gateway: none
```

### NAT Adapter

```
Purpose: Software installation and updates
Addressing: VMware DHCP
Default gateway: VMware NAT gateway
```

Only the NAT adapter should normally provide the default route.

---

## Internal Traffic Flow

```
WIN11TARGET
     |
     | Windows events or forwarder traffic
     v
SPLUNK-SERVER
     |
     | Parsing and indexing
     v
Splunk Index
     |
     | SPL searches and dashboards
     v
Student Analyst
```

Management and ingestion services should remain internal to the CyberLab.

---

## Ubuntu Server Preparation

Before installing Splunk:

1. Confirm the Ubuntu VM starts successfully.
2. Confirm the hostname.
3. Confirm host-only connectivity.
4. Confirm NAT connectivity when required.
5. Confirm DNS resolution.
6. Confirm system time.
7. Install available operating system updates.
8. Confirm sufficient disk space.
9. Create a stable pre-installation snapshot.
10. Download or transfer the Splunk installer.

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
* Distribution
* Release
* Architecture
* Kernel

Command output may contain operational identifiers and should be sanitized before publication.

---

## Set the Hostname

Example:

```
sudo hostnamectl set-hostname SPLUNK-SERVER
```

Review `/etc/hosts`:

```
sudo nano /etc/hosts
```

Example:

```
127.0.0.1       localhost
127.0.1.1       SPLUNK-SERVER
```

Restart or open a new login session after changing the hostname.

---

## Verify Network Configuration

```
ip address
```

```
ip route
```

```
resolvectl status
```

Confirm:

* The host-only interface has the intended internal address.
* The NAT interface has an address when enabled.
* Only the NAT interface provides the default route.
* Internal systems are reachable.
* External resolution works when required.

---

## Connectivity Validation

Test the Acer host or another internal lab system:

```
ping -c 4 <INTERNAL_LAB_SYSTEM>
```

Test external HTTPS connectivity:

```
curl -I https://example.com
```

A failed ping may indicate ICMP filtering rather than complete loss of connectivity.

---

## Confirm System Time

```
timedatectl
```

Accurate time is required for:

* Event correlation
* Search timelines
* Authentication
* Forwarder communication
* Incident reconstruction
* Certificate validation

Splunk events from multiple systems become difficult to analyze when timestamps are not aligned.

---

## Update Ubuntu

```
sudo apt update
```

```
sudo apt upgrade
```

Restart when required:

```
sudo reboot
```

After restart, verify networking and time again.

---

## Obtain the Splunk Installer

Download the supported Splunk Enterprise Linux `.deb` package from the official Splunk distribution source.

Example sanitized filename:

```
splunk-<VERSION>-linux-amd64.deb
```

Record privately:

```
Splunk version:
Package filename:
Download date:
Source:
SHA-256:
Installation date:
```

Do not upload the commercial installer package to GitHub.

---

## Installer Transfer Methods

The package may be transferred to the Splunk VM through:

* Direct browser download
* SCP
* SFTP
* VMware shared folder
* Temporary SMB share
* Mounted ISO
* USB passthrough
* Controlled local web server

In the operational CyberLab, the package was downloaded to a Windows downloads directory and then made available for installation on the Linux system.

Public documentation should replace personal Windows paths with a placeholder such as:

```
<WINDOWS_DOWNLOAD_DIRECTORY>
```

---

## File Transfer Security

Before transferring the installer:

* Confirm it came from the official source.
* Confirm the expected filename.
* Verify the checksum when available.
* Scan the package.
* Avoid exposing unrelated host directories.
* Disable temporary shared folders after use.
* Remove unnecessary installer copies.

Do not publish a path containing a personal Windows username.

---

## Verify the Installer Package

From the Linux VM:

```
ls -lh <SPLUNK_PACKAGE>.deb
```

Calculate a SHA-256 hash:

```
sha256sum <SPLUNK_PACKAGE>.deb
```

Compare it with the vendor-published checksum when one is available.

---

## Inspect the Debian Package

Review package metadata:

```
dpkg-deb --info <SPLUNK_PACKAGE>.deb
```

List package contents:

```
dpkg-deb --contents <SPLUNK_PACKAGE>.deb
```

These commands do not install the package.

---

## Install Splunk Enterprise

Move into the directory containing the package:

```
cd <INSTALLER_DIRECTORY>
```

Install the package:

```
sudo dpkg -i <SPLUNK_PACKAGE>.deb
```

Administrator privileges are required.

The standard package installs Splunk under:

```
/opt/splunk
```

---

## Resolve Package Dependency Issues

If `dpkg` reports missing dependencies:

```
sudo apt --fix-broken install
```

Then repeat the installation if required:

```
sudo dpkg -i <SPLUNK_PACKAGE>.deb
```

Review all package-manager output before proceeding.

---

## Verify the Installation Directory

```
ls -la /opt/splunk
```

Typical directories include:

```
/opt/splunk/bin
/opt/splunk/etc
/opt/splunk/var
```

Do not manually modify core installation files without understanding their function.

---

## Important Splunk Paths

| Path                           | Purpose                                       |
| ------------------------------ | --------------------------------------------- |
| `/opt/splunk/bin`              | Splunk commands and executables               |
| `/opt/splunk/etc/apps`         | Installed applications                        |
| `/opt/splunk/etc/system/local` | Local system configuration                    |
| `/opt/splunk/etc/auth`         | Certificates and authentication-related files |
| `/opt/splunk/var/log/splunk`   | Splunk internal logs                          |
| `/opt/splunk/var/lib/splunk`   | Indexed data and application state            |

Files under authentication and configuration directories may contain sensitive information.

---

## First Start

Start Splunk from its installation directory:

```
sudo /opt/splunk/bin/splunk start
```

On the first launch:

1. Review the software license.
2. Accept the license when appropriate.
3. Create the Splunk administrator account.
4. Enter a strong unique password.
5. Wait for the Splunk services to initialize.
6. Review the reported web interface address.
7. Confirm that no startup error is displayed.

---

## Noninteractive License Acceptance

A lab installation may use:

```
sudo /opt/splunk/bin/splunk start --accept-license
```

Additional options may be required depending on the version and deployment process.

Do not store an administrator password in a shell command committed to GitHub.

---

## Splunk Administrative Credentials

The Splunk administrator account should use:

* A unique username where supported
* A strong unique password
* Password-manager storage
* No reuse with Linux or Windows accounts
* No inclusion in screenshots
* No inclusion in scripts
* No inclusion in shell history
* No inclusion in GitHub

Default or weak credentials should not remain active.

---

## Current Lab Startup Behavior

During installation validation, Splunk required an elevated startup command in the CyberLab.

The working command was:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

This starts Splunk with root privileges.

The `--run-as-root` option should be treated as a documented lab workaround or temporary configuration, not the preferred long-term production model.

---

## Root Versus Normal User Guidance

### Normal User

A normal Linux user can perform tasks such as:

* Reviewing unprotected files
* Testing network connectivity
* Accessing Splunk Web
* Reviewing user-readable logs
* Running searches in the dashboard

### `sudo` or Root Required

Elevated privileges may be required for:

* Installing the `.deb` package
* Changing ownership under `/opt/splunk`
* Binding to protected ports
* Configuring boot startup
* Editing protected configuration
* Managing system services
* Accessing restricted logs
* Running Splunk under the root account

Use elevated privileges only when required.

---

## Risks of Running Splunk as Root

Running Splunk as root increases the impact of:

* A Splunk vulnerability
* A malicious application
* An unsafe scripted input
* Incorrect file permissions
* Compromised administrative credentials
* Misconfigured commands
* App installation risks

A root-run Splunk process may have broad access to the Linux host.

For a long-term hardened deployment, Splunk should normally run under a dedicated service account unless a specific requirement prevents it.

---

## Preferred Dedicated Service Account Model

A dedicated account may be created:

```
sudo useradd \
    --system \
    --create-home \
    --home-dir /opt/splunk \
    --shell /bin/bash \
    splunk
```

The exact account-management method depends on the installation package and current ownership.

Before changing ownership:

1. Stop Splunk.
2. Back up configuration.
3. Record current ownership.
4. Confirm the intended service account.
5. Update ownership carefully.
6. Start Splunk under the account.
7. Validate every input and service.

---

## Review Current Ownership

```
sudo ls -ld /opt/splunk
```

```
sudo find /opt/splunk \
    -maxdepth 2 \
    -printf '%u:%g %p\n' |
    head -50
```

Do not recursively change ownership without confirming how Splunk was installed and configured.

---

## Example Ownership Change

A typical dedicated-user installation may require:

```
sudo chown -R splunk:splunk /opt/splunk
```

This command affects the entire Splunk installation.

Use it only after:

* Confirming the `splunk` account exists
* Stopping Splunk
* Creating a backup
* Reviewing current ownership
* Confirming no root-owned input requires special handling

---

## Start Splunk as the Service User

Example:

```
sudo -u splunk /opt/splunk/bin/splunk start
```

If this fails after previously running Splunk as root, inspect:

* File ownership
* Lock files
* Log permissions
* Index permissions
* Application permissions
* Port bindings

Do not repeatedly alternate between root and service-user startup without correcting ownership.

---

## Verify Splunk Status

```
sudo /opt/splunk/bin/splunk status
```

When using a dedicated service account:

```
sudo -u splunk /opt/splunk/bin/splunk status
```

Expected output should indicate that `splunkd` is running.

---

## Review Splunk Processes

```
ps aux |
    grep -i splunk
```

Or:

```
pgrep -a splunk
```

Review the account under which Splunk processes are running.

---

## Verify Listening Ports

Splunk Web commonly listens on TCP port `8000`.

Review listeners:

```
sudo ss -tulpn |
    grep 8000
```

Review all Splunk-related listeners:

```
sudo ss -tulpn |
    grep -i splunk
```

The output may not display process names without elevated privileges.

---

## Splunk Web Access

From the Acer host, access the internal web interface:

```
http://<SPLUNK_SERVER>:8000
```

The operational CyberLab successfully validated the Splunk Web interface through the server’s host-only address.

The exact address is intentionally omitted from the public repository.

---

## Web Interface Validation

Confirm:

* The login page loads.
* The expected Splunk branding appears.
* The administrator account can sign in.
* The home page loads.
* Settings are accessible.
* Search and Reporting opens.
* No license or service error blocks normal use.

A loading web page alone does not prove that indexing and searching are fully functional.

---

## Service Binding

Splunk Web may listen on:

* All interfaces
* The host-only interface
* The NAT interface
* Loopback only

Review:

```
sudo ss -tulpn |
    grep 8000
```

A listener such as:

```
0.0.0.0:8000
```

means that Splunk Web is listening on all IPv4 interfaces.

A local firewall should restrict access to the trusted CyberLab network.

---

## Host Firewall

Review Ubuntu firewall status:

```
sudo ufw status verbose
```

A narrow rule can permit Splunk Web from only the host-only subnet:

```
sudo ufw allow \
    from 192.0.2.0/24 \
    to any port 8000 \
    proto tcp
```

The documentation subnet is only an example.

Do not copy it into the operational environment.

---

## SSH Firewall Rule

Before enabling a default-deny firewall, allow SSH from the trusted internal network:

```
sudo ufw allow \
    from 192.0.2.0/24 \
    to any port 22 \
    proto tcp
```

Keep VMware console access available in case a firewall rule blocks remote administration.

---

## Default Firewall Policy

A controlled configuration may use:

```
sudo ufw default deny incoming
```

```
sudo ufw default allow outgoing
```

Then enable only required internal services.

Enable UFW:

```
sudo ufw enable
```

Administrator privileges are required.

---

## Do Not Expose Splunk Web Publicly

Splunk Web should not be:

* Port-forwarded through the home router
* Published through the ISP gateway
* Exposed directly to the public internet
* Bound to a public cloud address without protection
* Accessible from untrusted wireless networks
* Shared through a public tunnel

Administrative access should remain inside the CyberLab.

---

## Enable Splunk at Boot

Splunk can configure automatic startup.

When Splunk is currently operating as root:

```
sudo /opt/splunk/bin/splunk enable boot-start
```

The exact generated service behavior depends on the Splunk version and options.

Review the command output carefully.

---

## Boot Start with a Dedicated User

A hardened configuration may use:

```
sudo /opt/splunk/bin/splunk enable boot-start \
    -user splunk
```

Some versions support systemd-specific options.

Always review the command supported by the installed release:

```
sudo /opt/splunk/bin/splunk help enable
```

---

## Review the Boot Service

```
systemctl list-unit-files |
    grep -i splunk
```

Review status:

```
sudo systemctl status Splunkd
```

The service name may vary according to the generated boot configuration.

---

## Start and Stop Through Splunk CLI

Start:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

Stop:

```
sudo /opt/splunk/bin/splunk stop
```

Restart:

```
sudo /opt/splunk/bin/splunk restart --run-as-root
```

Status:

```
sudo /opt/splunk/bin/splunk status
```

If Splunk is migrated to a dedicated account, use the same account consistently.

---

## Routine Startup Procedure

When Splunk is not configured for automatic boot:

1. Start the Ubuntu VM.
2. Confirm the host-only address.
3. Confirm system time.
4. Confirm disk space.
5. Start Splunk using the documented command.
6. Confirm `splunkd` is running.
7. Confirm TCP port `8000` is listening.
8. Open Splunk Web.
9. Run a test search.
10. Confirm expected data is available.

Current lab command:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

---

## Routine Shutdown Procedure

1. End active searches.
2. Confirm required events have been indexed.
3. Export important evidence.
4. Record the current service state.
5. Stop Splunk cleanly.
6. Confirm the process has stopped.
7. Shut down Ubuntu.
8. Confirm the VM is powered off.
9. Create a snapshot when appropriate.
10. Shut down dependent infrastructure according to the lab sequence.

Stop Splunk:

```
sudo /opt/splunk/bin/splunk stop
```

---

## Do Not Force Power Off During Indexing

A forced VM shutdown can interrupt:

* Index writes
* Internal databases
* Search artifacts
* Configuration changes
* App installation
* Log ingestion

Always stop Splunk and shut down the operating system cleanly when possible.

---

## Splunk Internal Logs

Splunk stores internal logs under:

```
/opt/splunk/var/log/splunk
```

Important examples may include:

```
splunkd.log
web_service.log
scheduler.log
metrics.log
mongod.log
```

The exact logs depend on installed features and version.

---

## Review `splunkd.log`

```
sudo tail -n 100 \
    /opt/splunk/var/log/splunk/splunkd.log
```

Follow in real time:

```
sudo tail -f \
    /opt/splunk/var/log/splunk/splunkd.log
```

Look for:

* Startup errors
* Permission errors
* Port conflicts
* Index failures
* Input failures
* License warnings
* Authentication events
* Configuration parsing errors

---

## Search Internal Splunk Logs

From Splunk Web:

```spl
index=_internal
| sort - _time
```

Limit to Splunk daemon events:

```spl
index=_internal source=*splunkd.log
| sort - _time
```

Show recent errors:

```spl
index=_internal log_level=ERROR
| table _time host component message
| sort - _time
```

The exact fields available depend on the events.

---

## Splunk Configuration Layers

Splunk configuration files commonly use `.conf` files.

Configuration may exist under:

```
/opt/splunk/etc/system/default
/opt/splunk/etc/system/local
/opt/splunk/etc/apps/<APP>/default
/opt/splunk/etc/apps/<APP>/local
```

Do not modify files under `default` unless explicitly required.

Place local overrides under an appropriate `local` directory.

---

## Common Configuration Files

| File                  | Purpose                         |
| --------------------- | ------------------------------- |
| `inputs.conf`         | Defines data inputs             |
| `outputs.conf`        | Defines forwarding destinations |
| `server.conf`         | Server-level configuration      |
| `web.conf`            | Splunk Web settings             |
| `indexes.conf`        | Index definitions and retention |
| `props.conf`          | Parsing and field behavior      |
| `transforms.conf`     | Data transformations            |
| `limits.conf`         | Search and platform limits      |
| `authentication.conf` | Authentication configuration    |

Operational configuration files may contain sensitive values.

---

## Back Up Configuration Before Editing

Example:

```
sudo cp \
    /opt/splunk/etc/system/local/<FILE>.conf \
    /opt/splunk/etc/system/local/<FILE>.conf.bak
```

Record:

```
Date:
Configuration file:
Reason:
Original checksum:
Change:
Validation:
Rollback plan:
```

---

## Validate Effective Configuration

Splunk includes `btool` for reviewing merged configuration.

Example:

```
sudo /opt/splunk/bin/splunk btool inputs list --debug
```

Review web settings:

```
sudo /opt/splunk/bin/splunk btool web list --debug
```

Review index settings:

```
sudo /opt/splunk/bin/splunk btool indexes list --debug
```

The `--debug` output includes file paths and should be sanitized before publication.

---

## Data Ingestion Methods

The CyberLab may ingest data through:

* Manual file upload
* Monitor inputs
* Splunk Universal Forwarder
* Windows Event Log input
* Scripted input
* TCP input
* Syslog
* Test datasets
* Imported CSV or JSON data

The safest initial validation method is a small, known test file.

---

## Ingestion Pipeline

```
Data Source
    |
    | Collection or upload
    v
Splunk Input
    |
    | Parsing and timestamp recognition
    v
Sourcetype
    |
    | Indexed into target index
    v
Search and Reporting
```

Each layer should be validated separately.

---

## Indexes

An index is a logical storage location for data.

Splunk includes internal indexes such as:

```
_internal
_audit
_introspection
```

User-created lab data should be stored in a dedicated index rather than mixed unnecessarily with default indexes.

---

## Create a Lab Index

A public example index name:

```
cyberlab
```

It may be created through:

* Splunk Web
* `indexes.conf`
* Splunk REST API
* Splunk CLI where supported

Use a name that communicates the data purpose.

---

## Index Naming Practices

Good examples:

```
windows
linux
cyberlab
security
network
```

Avoid vague names such as:

```
test
newindex
data2
final
```

Document:

* Data source
* Owner
* Retention
* Expected volume
* Sensitivity
* Validation search

---

## Manual File Upload

A small test log may be uploaded through Splunk Web.

General workflow:

1. Open **Settings**.
2. Select **Add Data**.
3. Choose **Upload**.
4. Select a sanitized test file.
5. Review source type detection.
6. Confirm timestamp parsing.
7. Select or create the target index.
8. Review the input settings.
9. Submit the upload.
10. Search the resulting events.

Do not upload files containing real credentials or personal data.

---

## Example Test Log

A harmless test file may contain:

```
2026-01-01T10:00:00Z host=WIN11TARGET action=login status=success user=test.user
2026-01-01T10:01:00Z host=WIN11TARGET action=login status=failure user=test.user
2026-01-01T10:02:00Z host=WIN11TARGET action=file_change status=authorized file=sample.txt
```

These are synthetic values.

---

## Validate Uploaded Data

Search the lab index:

```spl
index=cyberlab
```

Review basic metadata:

```spl
index=cyberlab
| table _time host source sourcetype _raw
```

Count events:

```spl
index=cyberlab
| stats count
```

Count by source type:

```spl
index=cyberlab
| stats count by sourcetype
```

---

## Source, Host, and Sourcetype

Splunk event metadata commonly includes:

* `source`
* `host`
* `sourcetype`
* `index`

### Source

Identifies where the data originated, such as a file path or input.

### Host

Represents the event-producing system.

### Sourcetype

Describes the data format and parsing behavior.

### Index

Specifies the logical storage destination.

Accurate metadata improves searches and detections.

---

## Splunk Universal Forwarder

The Splunk Universal Forwarder can collect and send data from `WIN11TARGET`.

It is designed for lightweight forwarding rather than local searching.

Potential sources include:

* Windows Security log
* Windows System log
* Windows Application log
* PowerShell Operational log
* Sysmon Operational log
* Custom text logs
* File integrity tool output

---

## Forwarder Architecture

```
WIN11TARGET
     |
     | Splunk Universal Forwarder
     v
SPLUNK-SERVER
     |
     | Parsing, indexing, searching
     v
Student Analyst
```

The destination should be the server’s internal host-only address.

---

## Forwarder Prerequisites

Before deployment:

* Splunk Enterprise is running.
* A receiving port is intentionally enabled.
* The endpoint can reach the server.
* The endpoint hostname is correct.
* Time is synchronized.
* The Universal Forwarder installer is from an official source.
* Administrative credentials are protected.
* A snapshot exists.

---

## Enable a Receiving Port

Splunk commonly receives forwarder traffic through a configurable TCP port.

The exact operational port should be documented privately.

Through Splunk Web:

1. Open **Settings**.
2. Select **Forwarding and receiving**.
3. Select **Configure receiving**.
4. Add the intended port.
5. Save the configuration.
6. Validate that the listener exists.
7. Restrict it with the host firewall.

Do not expose the receiver to untrusted networks.

---

## Verify the Receiver Listener

On Linux:

```
sudo ss -tulpn |
    grep <RECEIVER_PORT>
```

From Windows:

```
Test-NetConnection <SPLUNK_SERVER> -Port <RECEIVER_PORT>
```

A successful TCP test confirms the listener is reachable, not that events are being indexed correctly.

---

## Install the Universal Forwarder

Run the official installer on the Windows endpoint using an administrative account.

The installation may request:

* Forwarder service account
* Deployment server
* Receiving indexer
* Management port
* Administrative credentials

Do not publish installer commands containing passwords.

---

## Verify the Forwarder Service

On `WIN11TARGET`:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm the Universal Forwarder service exists and is running.

Administrator privileges are required to start or stop the service.

---

## Forwarder Configuration Paths

A Windows Universal Forwarder commonly stores configuration under its installation directory.

Relevant files may include:

```
inputs.conf
outputs.conf
server.conf
```

Do not publish an operational `outputs.conf` if it reveals internal addresses or credentials.

---

## Example Sanitized `outputs.conf`

```ini
[tcpout]
defaultGroup = cyberlab_indexers

[tcpout:cyberlab_indexers]
server = <SPLUNK_SERVER>:<RECEIVER_PORT>
```

This is a template, not an operational configuration.

---

## Example Sanitized Windows Event Input

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

The exact syntax and supported channels should be validated against the installed forwarder version.

---

## Restart the Universal Forwarder

After configuration changes:

```
Restart-Service <SPLUNK_FORWARDER_SERVICE>
```

Administrator privileges are required.

Review the forwarder logs if the service does not restart.

---

## Validate Forwarder Data

Search for the endpoint:

```spl
index=windows host=WIN11TARGET
```

Review event codes:

```spl
index=windows host=WIN11TARGET
| stats count by EventCode
| sort - count
```

Review recent events:

```spl
index=windows host=WIN11TARGET
| table _time host source sourcetype EventCode user _raw
| sort - _time
```

Field names depend on the installed add-ons and sourcetypes.

---

## Windows Event Log Sources

Useful Windows channels include:

* Security
* System
* Application
* Windows PowerShell
* PowerShell Operational
* Windows Defender Operational
* Group Policy Operational
* Sysmon Operational when installed

Collect only the sources needed for the lab exercise.

Broad collection can increase noise and storage use.

---

## Splunk Add-ons

Add-ons may provide:

* Event field extraction
* Sourcetype definitions
* Data normalization
* Windows field mappings
* Technology-specific knowledge
* Common Information Model support

Install add-ons only from trusted sources.

Review:

* Compatibility
* Required permissions
* Configuration changes
* Included scripts
* Update history
* Data model impact

---

## Splunk Common Information Model

The Splunk Common Information Model provides standardized field names and data models.

It can help correlate different data sources around concepts such as:

* Authentication
* Endpoint activity
* Network traffic
* Intrusion detection
* Change analysis
* Malware
* Vulnerabilities

CIM compliance requires correct field extraction and tagging.

Installing an add-on does not automatically guarantee correct data normalization.

---

## Search Processing Language

Search Processing Language, or SPL, is Splunk’s query language.

Basic search:

```spl
index=windows
```

Filter by endpoint:

```spl
index=windows host=WIN11TARGET
```

Filter by event identifier:

```spl
index=windows EventCode=<EVENT_ID>
```

Sort newest first:

```spl
index=windows
| sort - _time
```

---

## Common SPL Commands

| Command       | Purpose                                          |
| ------------- | ------------------------------------------------ |
| `search`      | Filters events                                   |
| `table`       | Displays selected fields                         |
| `stats`       | Calculates aggregations                          |
| `timechart`   | Creates time-based statistics                    |
| `sort`        | Sorts results                                    |
| `dedup`       | Removes duplicate field values                   |
| `eval`        | Creates or transforms fields                     |
| `where`       | Filters using expressions                        |
| `rex`         | Extracts fields using regular expressions        |
| `transaction` | Groups related events, with performance cautions |

Searches should begin narrowly whenever possible.

---

## Failed Logon Search

A generic Windows failed-logon search may resemble:

```spl
index=windows EventCode=<FAILED_LOGON_EVENT>
| table _time host user src_ip LogonType FailureReason
| sort - _time
```

The available fields depend on event parsing and add-ons.

---

## Successful Logon Search

```spl
index=windows EventCode=<SUCCESSFUL_LOGON_EVENT>
| table _time host user src_ip LogonType
| sort - _time
```

Analysts should distinguish:

* Interactive logons
* Network logons
* Service logons
* Remote interactive logons
* System-generated activity

---

## Account Lockout Search

```spl
index=windows EventCode=<ACCOUNT_LOCKOUT_EVENT>
| table _time host user CallerComputerName
| sort - _time
```

Correlate the lockout with earlier failed-logon events.

---

## User Creation Search

```spl
index=windows EventCode=<USER_CREATED_EVENT>
| table _time host SubjectUserName TargetUserName
| sort - _time
```

Then determine:

* Who created the account
* Where the change occurred
* Whether the account was authorized
* Whether privileged groups were modified

---

## Group Membership Change Search

```spl
index=windows EventCode=<GROUP_MEMBERSHIP_CHANGE_EVENT>
| table _time host SubjectUserName MemberName TargetUserName
| sort - _time
```

Event fields vary according to the event type and parsing configuration.

---

## PowerShell Search

```spl
index=windows source=*PowerShell*
| table _time host user EventCode Message
| sort - _time
```

Search for process execution:

```spl
index=windows process_name="powershell.exe"
| table _time host user parent_process process_name process_command_line
| sort - _time
```

These fields may require Sysmon or appropriate event parsing.

---

## Timeline Reconstruction

A simple endpoint timeline:

```spl
index=windows host=WIN11TARGET
| table _time EventCode user process_name process_command_line Message
| sort _time
```

Timeline analysis should consider:

* Timestamp source
* Timezone
* Clock drift
* Duplicate events
* Ingestion delay
* Event generation time versus index time

---

## Event Time Versus Index Time

Splunk commonly tracks:

* `_time`: interpreted event time
* `_indextime`: time the event entered the index

A comparison search:

```spl
index=windows
| eval ingestion_delay=_indextime-_time
| stats avg(ingestion_delay) max(ingestion_delay) by host
```

Large delays may indicate:

* Forwarder outage
* Network interruption
* Parsing problem
* Backlog
* Incorrect timestamps
* System time differences

---

## Dashboard Creation

Dashboards can summarize:

* Authentication failures
* Account lockouts
* User creation
* Privilege changes
* PowerShell activity
* Endpoint event volume
* Agent or forwarder health
* Events by host
* Events over time

A dashboard should support an investigation objective rather than displaying data without context.

---

## Basic Dashboard Workflow

1. Build and validate the search.
2. Choose an appropriate visualization.
3. Save the search as a panel.
4. Create or select a dashboard.
5. Add a clear panel title.
6. Add time controls.
7. Validate field names.
8. Test with known events.
9. Document limitations.
10. Sanitize screenshots before publication.

---

## Detection Search Development

A detection search should include:

* Objective
* Data source
* Search logic
* Required fields
* Time range
* Threshold
* Expected result
* False positives
* Validation procedure
* Response guidance

Example documentation:

```
Detection:
Data source:
SPL:
Expected behavior:
Suspicious behavior:
Known false positives:
Validation:
Investigation steps:
Cleanup:
```

---

## Avoid Overly Broad Searches

A search such as:

```spl
index=*
```

can be expensive and noisy.

Prefer:

```spl
index=windows host=WIN11TARGET earliest=-15m
```

Narrow searches using:

* Index
* Host
* Source
* Sourcetype
* Event identifier
* Time range

This improves speed and accuracy.

---

## Saved Searches

Saved searches support repeatable analysis.

Examples include:

* Recent failed logons
* Account lockouts
* New user accounts
* Administrator group changes
* Suspicious PowerShell
* Forwarder inactivity
* Wazuh-to-Splunk comparison
* Endpoint event health

Saved-search names should communicate intent clearly.

---

## Alerting Considerations

Splunk searches can generate alerts based on:

* Event count
* Threshold
* Scheduled search
* Specific field values
* Absence of expected data
* Statistical anomalies

For a personal lab:

* Start with manual validation.
* Confirm false positives.
* Use low-frequency scheduling.
* Avoid excessive notifications.
* Document cleanup.
* Confirm alerts do not expose sensitive data.

---

## Licensing Considerations

Splunk Enterprise licensing and available features may vary.

The CyberLab should track:

* Installed edition
* License state
* Daily ingestion allowance
* Trial expiration
* Search limitations
* Feature availability

Do not publish:

* License files
* License identifiers
* Entitlement information
* Customer account data
* Activation details

---

## Check License Status

License information can be reviewed through Splunk Web under licensing settings.

CLI commands vary by Splunk version and authentication state.

Public documentation should describe the license state generically rather than exposing identifiers.

---

## Data Retention

Retention should balance:

* Learning objectives
* Available storage
* Event volume
* Snapshot size
* Investigation history
* Privacy
* Reset frequency

A personal lab does not normally need production-scale retention.

Document retention by index.

---

## Index Retention Warning

Deleting or changing an index can remove data.

Before changing retention:

1. Identify the index.
2. Record current settings.
3. Export important evidence.
4. Back up configuration.
5. Confirm available storage.
6. Apply one change.
7. Restart only when required.
8. Validate the index.
9. Record the outcome.
10. Maintain a rollback plan.

---

## Monitor Disk Usage

Linux filesystem:

```
df -h
```

Splunk directory:

```
sudo du -sh /opt/splunk
```

Index storage:

```
sudo du -sh /opt/splunk/var/lib/splunk/*
```

Use caution because recursive size checks may be resource intensive.

---

## Review Splunk Storage Through SPL

Internal metrics may provide visibility into indexing and storage activity.

Example:

```spl
index=_internal source=*metrics.log group=per_index_thruput
| stats sum(kb) as total_kb by series
| sort - total_kb
```

The exact internal event structure may vary by release.

---

## Backup Targets

Important Splunk backup targets include:

* `/opt/splunk/etc`
* Custom applications
* Local configuration
* Saved searches
* Dashboards
* Field extractions
* Lookup files
* Index definitions
* Detection searches
* Documentation
* License information stored separately and securely

Indexed data requires a separate strategy if preservation is necessary.

---

## Configuration Backup

Stop Splunk before a consistent full configuration backup when practical.

Example:

```
sudo tar -czf \
    <BACKUP_PATH>/splunk-config-<DATE>.tar.gz \
    /opt/splunk/etc
```

The archive may contain:

* Password hashes
* Authentication data
* Internal addresses
* Certificates
* Application configuration
* License information

Store it securely and do not upload it directly to GitHub.

---

## Sanitized Configuration Templates

Public templates may include:

```
inputs.conf.example
outputs.conf.example
indexes.conf.example
web.conf.example
```

Replace sensitive values with placeholders:

```
<SPLUNK_SERVER>
<RECEIVER_PORT>
<INDEX_NAME>
<WINDOWS_ENDPOINT>
<REDACTED_VALUE>
```

---

## Snapshot Strategy

Recommended milestones:

```
01-SPLUNK-Ubuntu-Clean-Install
02-SPLUNK-Ubuntu-Patched
03-SPLUNK-Network-Configured
04-SPLUNK-Package-Installed
05-SPLUNK-Web-Validated
06-SPLUNK-Test-Data-Ingested
07-SPLUNK-Receiver-Enabled
08-SPLUNK-Forwarder-Connected
09-SPLUNK-Windows-Events-Validated
10-SPLUNK-Exercise-Ready
```

Create important stable snapshots while the VM is powered off.

---

## Safe Snapshot Procedure

1. End active searches.
2. Confirm required data is indexed.
3. Export important investigation evidence.
4. Stop Splunk cleanly.
5. Shut down Ubuntu.
6. Confirm the VM is powered off.
7. Create the snapshot.
8. Add a descriptive note.
9. Start Ubuntu.
10. Start and validate Splunk.

---

## Snapshot Restore Risks

Restoring a Splunk snapshot may cause:

* Loss of recently indexed data
* Time rollback
* Forwarder reconnection issues
* Duplicate events
* Stale credentials
* License inconsistencies
* Index corruption
* Missing saved searches
* Changed server identity

After restoration:

1. Confirm system time.
2. Confirm network configuration.
3. Confirm disk space.
4. Start Splunk.
5. Review internal logs.
6. Open Splunk Web.
7. Confirm indexes.
8. Confirm forwarders reconnect.
9. Generate a new test event.
10. Document the restoration.

---

## Cold Backup Procedure

1. Stop Splunk.
2. Shut down Ubuntu.
3. Confirm the VM is powered off.
4. Copy or export the VM directory.
5. Store the backup separately.
6. Record the backup milestone.
7. Verify the copied files.
8. Restart the original VM.
9. Validate Splunk.
10. Periodically test restoration.

A snapshot stored with the active VM is not an independent backup.

---

## Startup Validation Checklist

### Ubuntu

* Ubuntu starts successfully.
* Hostname is correct.
* Internal address is correct.
* NAT route exists only when required.
* Time and timezone are correct.
* Adequate free space exists.

### Splunk Service

* Splunk starts using the documented account.
* `splunkd` is running.
* No recurring critical startup error appears.
* Splunk Web listens on the intended port.
* Firewall rules restrict access appropriately.

### Splunk Web

* Login succeeds.
* Search and Reporting loads.
* Internal logs are searchable.
* User-created indexes are available.
* No blocking license warning appears.

### Data

* A recent test event can be found.
* Event timestamps are correct.
* Host metadata is correct.
* Sourcetypes are reasonable.
* Forwarded endpoint data is current.

---

## Controlled Validation Exercise

A simple end-to-end test should verify:

1. A harmless event is generated on `WIN11TARGET`.
2. The event exists locally.
3. The Universal Forwarder is running.
4. The event is sent to Splunk.
5. Splunk indexes the event.
6. The correct host is assigned.
7. The timestamp is correct.
8. The event can be found with SPL.
9. The event is added to an investigation timeline.
10. The result is documented.

---

## Example Authentication Exercise

1. Sign in to the endpoint with a designated test account.
2. Record the time.
3. Perform one authorized failed sign-in test.
4. Sign in successfully if appropriate.
5. Search Splunk for the relevant timeframe.
6. Compare successful and failed events.
7. Identify the endpoint and account.
8. Review logon type.
9. Document the reason for the activity.
10. Clean up or unlock the account if needed.

---

## Example PowerShell Exercise

Run a harmless command on the endpoint:

```
Get-Service |
    Sort-Object Status |
    Select-Object -First 10
```

Search Splunk for related process or PowerShell events.

Record:

* Timestamp
* User
* Endpoint
* Parent process
* Process name
* Command line
* Event identifier
* Data source
* Detection result

---

## Example User-Creation Exercise

Create a temporary test account through the approved lab procedure.

Then search for:

* Account-creation event
* Administrator who performed the action
* Target account
* Source system
* Related group-membership changes
* Follow-up authentication

Delete or disable the temporary account after the exercise.

---

## Troubleshooting: Splunk Command Not Found

Symptom:

```
splunk: command not found
```

Use the full path:

```
sudo /opt/splunk/bin/splunk status
```

The Splunk binary is not necessarily added to the system `PATH`.

Optional shell configuration can add the path, but public instructions should continue using the explicit path for clarity.

---

## Troubleshooting: Permission Denied

Symptoms may include:

* Splunk cannot write logs
* Splunk cannot start
* Index files are inaccessible
* Lock files cannot be created
* Configuration cannot be read

Review ownership:

```
sudo ls -ld /opt/splunk
```

```
sudo ls -l /opt/splunk/var/log/splunk |
    head
```

Review the running account:

```
ps aux |
    grep -i splunk
```

A common cause is switching between root and a dedicated Splunk account without correcting ownership.

---

## Troubleshooting: Splunk Requires `--run-as-root`

The lab installation successfully started with:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

Possible reasons include:

* Installation ownership
* Files previously created by root
* Existing boot configuration
* Restricted file permissions
* Service-account mismatch
* Protected input requirements

Recommended investigation:

1. Stop Splunk.
2. Review installation ownership.
3. Review log ownership.
4. Review the current boot service.
5. Identify the intended runtime account.
6. Back up configuration.
7. Correct ownership only when appropriate.
8. Test startup under the intended account.
9. Validate data inputs.
10. Document the result.

Do not change ownership blindly.

---

## Troubleshooting: Splunk Web Does Not Load

Check:

1. The VM is running.
2. Splunk is running.
3. The internal address is correct.
4. Port `8000` is listening.
5. The Acer host can reach the server.
6. UFW permits internal access.
7. The browser uses the correct protocol.
8. No VPN route is interfering.
9. Splunk Web logs contain no critical error.
10. Another process is not using the port.

Commands:

```
sudo /opt/splunk/bin/splunk status
```

```
sudo ss -tulpn |
    grep 8000
```

```
sudo tail -n 100 \
    /opt/splunk/var/log/splunk/web_service.log
```

---

## Troubleshooting: Nothing Returns in the Browser

If the browser appears to load indefinitely or displays no page:

1. Confirm the address is the Splunk VM’s host-only address.
2. Confirm the URL includes port `8000`.
3. Confirm Splunk started successfully.
4. Confirm `splunkd` remains running.
5. Confirm the server and Acer host share the host-only subnet.
6. Confirm the firewall rule.
7. Test the port from Windows.
8. Review Linux listening ports.
9. Review Splunk logs.
10. Test locally from the Linux VM.

From Windows:

```
Test-NetConnection <SPLUNK_SERVER> -Port 8000
```

From Linux:

```
curl -I http://127.0.0.1:8000
```

---

## Troubleshooting: Port 8000 Is Not Listening

Check status:

```
sudo /opt/splunk/bin/splunk status
```

Review logs:

```
sudo tail -n 200 \
    /opt/splunk/var/log/splunk/splunkd.log
```

Review web configuration:

```
sudo /opt/splunk/bin/splunk btool web list --debug
```

Possible causes include:

* Splunk is stopped
* Web service disabled
* Configuration error
* Port changed
* Port conflict
* Permission issue
* Incomplete first-run setup

---

## Troubleshooting: Port Conflict

Identify the process using port `8000`:

```
sudo ss -tulpn |
    grep 8000
```

Or:

```
sudo lsof -iTCP:8000 -sTCP:LISTEN
```

Do not terminate an unknown process without identifying it.

If the Splunk Web port is changed, update:

* Firewall rules
* Documentation
* Browser bookmarks
* Monitoring checks
* Architecture diagrams

---

## Troubleshooting: Splunk Will Not Start

Review:

```
sudo /opt/splunk/bin/splunk status
```

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

```
sudo tail -n 200 \
    /opt/splunk/var/log/splunk/splunkd.log
```

Possible causes include:

* Incorrect ownership
* Low disk space
* Invalid configuration
* License issue
* Port conflict
* Stale lock file
* Incomplete upgrade
* Corrupted index
* Insufficient memory

Do not repeatedly reinstall before identifying the cause.

---

## Troubleshooting: Data Does Not Appear

Check:

1. The source event exists.
2. The input is enabled.
3. The forwarder is running.
4. The receiver is listening.
5. The network path works.
6. The target index exists.
7. The time picker includes the event.
8. The host and sourcetype are correct.
9. The event was not routed elsewhere.
10. Internal logs show no ingestion error.

Start with:

```spl
index=* earliest=-15m
| stats count by index host sourcetype
```

Use broad searches only temporarily for troubleshooting.

---

## Troubleshooting: Forwarder Cannot Connect

On Windows:

```
Test-NetConnection <SPLUNK_SERVER> -Port <RECEIVER_PORT>
```

Review the forwarder service:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Check:

* Destination address
* Destination port
* Server listener
* Host firewall
* Endpoint firewall
* VMware adapter
* Time synchronization
* Forwarder logs
* Duplicate or stale configuration

---

## Troubleshooting: Events Have the Wrong Time

Possible causes include:

* Incorrect endpoint time
* Incorrect Splunk server time
* Timezone mismatch
* Timestamp parsing failure
* Wrong sourcetype
* Missing year
* Incorrect date format
* Snapshot restoration

Compare:

```spl
index=<INDEX_NAME>
| eval delay=_indextime-_time
| table _time _indextime delay host source _raw
```

Validate both the source system and Splunk server clocks.

---

## Troubleshooting: Search Returns No Results

Check:

* Time range
* Index name
* Host value
* Sourcetype
* Field spelling
* User permissions
* Event ingestion
* Search syntax

Begin with:

```spl
index=<INDEX_NAME> earliest=-24h
```

Then add filters one at a time.

---

## Troubleshooting: High Disk Usage

Review Linux storage:

```
df -h
```

Review Splunk storage:

```
sudo du -sh /opt/splunk/var/lib/splunk/*
```

Possible causes include:

* High ingestion
* Long retention
* Duplicate inputs
* Test datasets
* Internal log growth
* Large lookup files
* Old application packages
* Snapshot growth on the VMware host

Do not delete index directories manually.

---

## Troubleshooting: High Memory or CPU Use

Review:

```
free -h
```

```
top
```

```
ps aux \
    --sort=-%mem |
    head
```

Potential causes include:

* Expensive searches
* Broad wildcard searches
* High ingestion
* Large dashboards
* Excessive field extraction
* Insufficient VM resources
* Multiple concurrent searches
* Index maintenance
* Host resource pressure

Stop or revise inefficient searches before increasing resources.

---

## Troubleshooting: Login Fails

Check:

* Username
* Password
* Keyboard layout
* Caps Lock
* Splunk service status
* Authentication logs
* Account lockout
* Browser session
* Password expiration
* Restored snapshot state

Do not repeatedly guess the administrator password.

Use the documented Splunk credential-recovery procedure for the installed version when required.

---

## Troubleshooting: Browser Displays a Security Warning

Splunk Web may initially use HTTP or locally generated certificates depending on configuration.

When HTTPS is enabled, warnings may occur because of:

* Self-signed certificate
* Hostname mismatch
* Expired certificate
* Incorrect system time
* Untrusted certificate authority

Confirm the server identity before proceeding.

Do not assume every certificate warning is harmless.

---

## Change Management

Document significant Splunk changes, including:

* Splunk upgrades
* Runtime-user changes
* Boot-start changes
* New indexes
* New inputs
* Forwarder additions
* App installations
* Port changes
* Firewall changes
* Retention changes
* Dashboard creation
* Detection searches
* Authentication changes
* Snapshot restoration

Example:

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

Before upgrading Splunk:

1. Record the current version.
2. Review the supported upgrade path.
3. Back up `/opt/splunk/etc`.
4. Export important dashboards and searches.
5. Confirm disk space.
6. Confirm the current runtime user.
7. Stop Splunk.
8. Create a powered-off VM snapshot.
9. Install the supported package.
10. Validate startup, searches, inputs, and forwarders.

Do not perform a major upgrade immediately before a planned lab exercise.

---

## Public Sanitization Standards

Remove or replace:

* Real internal IP addresses
* Real hostnames when sensitive
* Personal Linux usernames
* Splunk administrator username
* Passwords
* Authentication tokens
* Session cookies
* License details
* Customer identifiers
* Internal domain names
* Forwarder credentials
* Active receiving configuration
* Personal Windows paths
* Linux home-directory usernames
* MAC addresses
* Public IP addresses
* VMware identifiers
* Browser bookmarks
* Private search results
* Raw event data containing personal names

Use placeholders such as:

```
<SPLUNK_SERVER>
<SPLUNK_SERVER_IP>
<SPLUNK_ADMIN>
<WINDOWS_ENDPOINT>
<RECEIVER_PORT>
<INDEX_NAME>
<SOURCETYPE>
<INSTALLER_DIRECTORY>
<REDACTED_PASSWORD>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing Splunk screenshots:

1. Close unrelated browser tabs.
2. Hide bookmarks.
3. Clear notifications.
4. Use a neutral lab account.
5. Remove internal addresses.
6. Remove administrator usernames.
7. Remove license details.
8. Remove session identifiers.
9. Remove personal event data.
10. Review raw event fields.
11. Crop unrelated interface areas.
12. Verify redactions are permanent.

Suitable public screenshots may include:

* Sanitized Splunk login page
* Search and Reporting interface
* Synthetic test events
* Sanitized Windows log search
* Dashboard panel
* Forwarder connection validation
* Internal service status
* Controlled detection exercise

---

## Validation Checklist

### Ubuntu Server

* Ubuntu starts successfully.
* Hostname is correct.
* Host-only networking works.
* NAT works when enabled.
* Only the intended default route exists.
* Time and timezone are correct.
* Adequate storage remains.

### Splunk Installation

* The `.deb` package installed successfully.
* `/opt/splunk` exists.
* Splunk accepts the license.
* Administrative credentials are stored securely.
* The runtime account is documented.
* The current root-run requirement is documented.

### Splunk Service

* Splunk starts successfully.
* `splunkd` is running.
* Port `8000` is listening.
* Splunk Web loads from the Acer host.
* Firewall rules limit access.
* No recurring critical startup errors appear.

### Data Ingestion

* A test index exists.
* A test file can be ingested.
* Events are searchable.
* Timestamps are correct.
* Host and sourcetype values are understood.
* Endpoint data reaches Splunk when configured.

### Recovery

* A pre-installation snapshot exists.
* A post-validation snapshot exists.
* Configuration backups exist.
* Startup and shutdown procedures are documented.
* A restore-validation process exists.
* Important evidence is stored outside disposable snapshots.

---

## Skills Demonstrated

This deployment demonstrates:

* Ubuntu administration
* Debian package installation
* Linux permissions
* Service startup and troubleshooting
* Splunk Enterprise administration
* Splunk Web validation
* Search Processing Language
* Index creation
* Data ingestion
* Source and sourcetype management
* Windows event analysis
* Universal Forwarder concepts
* Firewall configuration
* Multi-adapter VMware networking
* SIEM troubleshooting
* Detection development
* Dashboard planning
* Snapshot management
* Backup planning
* Public documentation sanitization

---

## Lessons Learned

Key lessons include:

* A successful package installation does not prove the service is running.
* The Splunk CLI may need to be called through its full path.
* Splunk Web commonly uses port `8000`.
* A listening port should be validated independently from browser access.
* Linux ownership matters when changing the Splunk runtime account.
* Alternating between root and non-root startup can create permission problems.
* `--run-as-root` can restore functionality but increases risk.
* The host-only address should be used for internal management.
* Splunk services should not be exposed to the public internet.
* Event time and index time should be compared during ingestion troubleshooting.
* Indexes, hosts, sources, and sourcetypes must be understood before writing detections.
* Narrow SPL searches are faster and easier to validate.
* Snapshots are useful but do not replace configuration backups.
* Public search screenshots require careful event-data sanitization.

---

## Planned Improvements

Future improvements may include:

* Migrate Splunk to a dedicated service account
* Configure systemd boot startup
* Enable HTTPS with a lab certificate
* Install Splunk Universal Forwarder on `WIN11TARGET`
* Collect Windows Security events
* Collect PowerShell Operational events
* Collect Sysmon events
* Create dedicated Windows and Linux indexes
* Build authentication dashboards
* Create account-lockout detections
* Create suspicious PowerShell detections
* Develop incident timelines
* Compare Splunk and Wazuh alert coverage
* Add Sigma-to-SPL exercises
* Export dashboards and saved searches
* Automate configuration backups
* Monitor forwarder health
* Develop data-quality checks
* Document index retention
* Create Splunk investigation runbooks
* Add controlled network-security data

---

## Summary

Splunk Enterprise provides flexible log search, indexing, dashboard, and detection-development capabilities for the Acer Blue Team CyberLab.

A reliable deployment requires:

* Correct Ubuntu networking
* Verified installation media
* Secure administrative credentials
* Consistent file ownership
* A documented runtime account
* Successful Splunk service startup
* Restricted internal web access
* Validated data ingestion
* Accurate timestamps
* Deliberate index and sourcetype management
* Repeatable SPL searches
* Snapshots and configuration backups

The current CyberLab installation successfully validated Splunk Web through the internal VMware network using the documented elevated startup command. Future hardening will focus on migrating the service from root execution to a dedicated Splunk account while preserving stable data ingestion and administrative access.

