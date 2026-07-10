# Troubleshooting

## Overview

This document provides a structured troubleshooting process for the Acer Blue Team CyberLab.

The lab includes several dependent systems:

* Acer Windows virtualization host
* VMware Workstation
* VMware host-only and NAT networks
* DC01 domain controller
* WIN11TARGET Windows endpoint
* WAZUH-SERVER
* SPLUNK-SERVER
* KALI-TEST
* Wazuh agent
* Splunk Universal Forwarder
* Windows Event Logs
* Linux services
* Docker containers
* Security dashboards

A failure in one component can affect several others.

For example:

* A DNS problem can appear to be an Active Directory failure.
* A stopped SIEM service can appear to be an agent failure.
* A broken route can appear to be a firewall problem.
* A time difference can appear to be an authentication problem.
* A browser failure can appear to be an application failure.
* A snapshot restore can create identity, trust, and timestamp problems.

The troubleshooting process should move from the simplest and lowest-level checks toward the application layer.

All hostnames, IP addresses, domains, usernames, ports, file paths, and configuration values in this public document are sanitized placeholders.

---

## Troubleshooting Goals

The troubleshooting process should:

* Identify the affected layer
* Preserve evidence
* Avoid making the problem worse
* Confirm assumptions with tests
* Change one variable at a time
* Use the least disruptive repair
* Validate the repair
* Document the root cause
* Record preventive improvements
* Maintain a rollback path

---

## Public Example Environment

The public documentation uses values such as:

```
Acer host: CYBERLAB-HOST
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Kali system: KALI-TEST
Internal domain: cyberlab.example
Documentation subnet: 192.0.2.0/24
```

The `192.0.2.0/24` range is reserved for documentation.

These values do not represent the operational CyberLab.

---

## Troubleshooting Principle

Begin at the lowest relevant layer.

```
Power and VM State
        |
        v
Virtual Hardware
        |
        v
Network Interface
        |
        v
IP Addressing
        |
        v
Routing
        |
        v
DNS
        |
        v
Firewall
        |
        v
Service
        |
        v
Application
        |
        v
Agent, Forwarder, or Data Pipeline
        |
        v
Dashboard, Search, or Detection
```

Do not begin by reinstalling an application when the VM has no network route.

---

## Troubleshooting Workflow

Use this standard process:

1. Define the symptom.
2. Identify the affected system.
3. Record the time.
4. Preserve important evidence.
5. Confirm the VM is running.
6. Confirm network adapters.
7. Confirm IP configuration.
8. Confirm routes.
9. Confirm DNS.
10. Confirm firewall state.
11. Confirm the required service.
12. Confirm application logs.
13. Test the dependency chain.
14. Apply one controlled change.
15. Validate the result.
16. Document the root cause and repair.

---

## Symptom Definition

Avoid vague descriptions such as:

```
It does not work.
The network is broken.
Splunk is down.
The agent is bad.
```

Use specific descriptions:

```
Splunk Web does not load from the Acer host.

WIN11TARGET resolves DC01 to an unexpected address.

The Wazuh agent service is running but the dashboard shows Disconnected.

The endpoint can ping DC01 but cannot join the domain.

The Kali VM has host-only connectivity but no NAT internet access.
```

A specific symptom makes testing more efficient.

---

## Incident Notes Template

```
Date and time:

Affected system:

Observed symptom:

Last known-good state:

Recent changes:

Expected behavior:

Actual behavior:

Commands run:

Evidence collected:

Root cause:

Repair:

Validation:

Follow-up:
```

---

## Preserve Evidence First

Before restarting or restoring:

* Record the error message.
* Take a screenshot.
* Export relevant logs.
* Save service status.
* Record IP configuration.
* Record routes.
* Record timestamps.
* Preserve configuration files.
* Save packet captures.
* Record the active snapshot.

Restarting may remove useful evidence.

---

# General System Troubleshooting

## Confirm VM Power State

In VMware Workstation, confirm whether the VM is:

* Powered on
* Suspended
* Powered off
* Stuck during startup
* Waiting for user input

A suspended VM may return with stale:

* Time
* Network state
* Authentication sessions
* DHCP lease
* Agent connection
* Browser session

For stable testing, a clean shutdown and startup is often preferable to long suspension.

---

## Confirm Guest Responsiveness

Check:

* Console input
* Mouse and keyboard
* Login screen
* Desktop response
* Terminal response
* Disk activity
* CPU usage
* Memory pressure

If the guest is unresponsive, review host resource usage before forcing power off.

---

## Avoid Immediate Forced Power-Off

Forced power-off can damage:

* Filesystems
* Splunk indexes
* Wazuh data
* Windows updates
* Active Directory state
* Event logs
* Package databases

Use the guest shutdown process when possible.

---

## Host Resource Review

On the Acer host, review:

* CPU
* Memory
* Disk usage
* Disk activity
* VMware processes
* Windows updates
* Antivirus scans
* Number of active VMs

Use Task Manager or Resource Monitor.

A slow guest may be caused by host pressure rather than a guest configuration problem.

---

## Check Host Disk Space

Low host disk space can affect:

* VM startup
* Snapshot creation
* Snapshot consolidation
* Guest disk writes
* Application performance
* VM suspension
* Log ingestion

PowerShell:

```
Get-Volume |
    Select-Object DriveLetter, FileSystemLabel, SizeRemaining, Size
```

Maintain free space for:

* Virtual disk growth
* Snapshots
* Temporary files
* Windows updates
* Backup creation

---

## Check Guest Disk Space

Windows:

```
Get-Volume |
    Select-Object DriveLetter, SizeRemaining, Size
```

Linux:

```
df -h
```

Low guest disk space can cause:

* Log loss
* Service failure
* Update failure
* Database failure
* Container restart loops
* Indexing failure

---

## Confirm Time and Timezone

Windows:

```
Get-Date
w32tm /query /status
w32tm /query /source
```

Linux:

```
date
timedatectl
```

Time problems can affect:

* Kerberos
* TLS
* SIEM searches
* Alert correlation
* Agent enrollment
* Certificate validation
* Snapshot recovery

---

# VMware Workstation Troubleshooting

## VM Will Not Start

Possible causes:

* Insufficient memory
* VM file lock
* Missing virtual disk
* Snapshot issue
* Host disk full
* Corrupted configuration
* VMware service problem
* Another process using the VM

Check:

1. Confirm host free space.
2. Close duplicate VMware windows.
3. Confirm the VM is not running elsewhere.
4. Review the VM directory.
5. Review VMware error messages.
6. Restart VMware Workstation.
7. Restart the host only when necessary.
8. Do not delete lock files unless the VM is confirmed fully closed.

---

## Virtual Disk Not Found

Possible causes:

* VM directory moved
* Drive letter changed
* External drive disconnected
* File renamed
* Backup incomplete
* Broken snapshot chain

Do not create a replacement blank disk over the missing disk.

Confirm:

* The `.vmdk` files exist.
* The VM configuration points to the correct path.
* External storage is mounted.
* Snapshot files are present.
* The VM directory was copied completely.

---

## VM Was Moved or Copied Prompt

VMware may ask whether the VM was moved or copied.

Choose based on the intent:

### Moved

Preserves the VM identity where practical.

Use for restoring the same VM.

### Copied

Generates new identity values.

Use for a deliberate clone.

This can affect:

* MAC address
* VMware UUID
* Agent identity
* Forwarder identity
* Licensing
* Domain trust

Record the decision.

---

## VMware Tools Problems

Symptoms:

* Display does not resize
* Mouse integration fails
* Guest shutdown does not work
* Clipboard sharing fails
* Network driver missing

Windows:

* Repair or reinstall VMware Tools.
* Restart the guest.
* Review Device Manager.

Linux:

```
systemctl status open-vm-tools
```

Install when missing:

```
sudo apt install open-vm-tools open-vm-tools-desktop
```

---

## Snapshot Creation Fails

Check:

* Host disk space
* Existing consolidation warnings
* VM state
* File permissions
* Storage health
* Backup software conflicts
* VMware logs

Do not manually delete snapshot files.

---

## Snapshot Restore Creates New Problems

After restoration, check:

* System time
* Network adapters
* Domain secure channel
* Wazuh agent identity
* Splunk forwarder identity
* Service status
* Recent logs
* Duplicate events
* Stale credentials

A restored VM may boot successfully but still be operationally unhealthy.

---

# VMware Network Troubleshooting

## Network Troubleshooting Order

1. Confirm the correct VMware adapter exists.
2. Confirm the adapter is connected.
3. Confirm the correct VMnet is selected.
4. Confirm the guest interface is enabled.
5. Confirm the IP address.
6. Confirm the subnet.
7. Confirm the route table.
8. Confirm DNS.
9. Confirm firewall.
10. Confirm the application port.

---

## Windows Network Commands

```
Get-NetAdapter
Get-NetIPConfiguration
Get-NetIPAddress
Get-NetRoute
Get-DnsClientServerAddress
Get-NetConnectionProfile
ipconfig /all
arp -a
```

---

## Linux Network Commands

```
ip link
ip address
ip route
ip neigh
resolvectl status
nmcli device status
nmcli connection show
```

---

## Adapter Is Disconnected

Symptoms:

* No IP address
* Media disconnected
* Interface DOWN
* No route
* No communication

In VMware:

1. Open VM settings.
2. Select the adapter.
3. Confirm **Connected**.
4. Confirm **Connect at power on**.
5. Confirm the correct VMnet.

Linux:

```
sudo ip link set <INTERFACE> up
```

Windows:

```
Enable-NetAdapter `
    -Name "<ADAPTER_NAME>"
```

Administrator privileges are required.

---

## Automatic Private Address

A Windows address beginning with `169.254` usually indicates DHCP failure.

Check:

* VMware DHCP service
* Correct VMnet
* Adapter state
* DHCP enabled
* Existing static configuration
* Host VMware services

Renew:

```
ipconfig /release
ipconfig /renew
```

For static infrastructure systems, confirm the intended static configuration instead.

---

## Duplicate IP Address

Symptoms:

* Intermittent connectivity
* Wrong MAC address in ARP
* Authentication failures
* Agent disconnects
* Access reaches the wrong VM

Windows:

```
arp -a
```

Linux:

```
ip neigh
```

Confirm static addresses are not inside an unmanaged DHCP pool.

---

## Wrong Subnet

Systems on different subnets cannot communicate directly without routing.

Review:

* IP address
* Prefix length
* Subnet mask
* VMware VMnet
* Static configuration
* DHCP range

Example mismatch:

```
System A: 192.0.2.20/24
System B: 192.0.3.30/24
```

These systems require routing or corrected addressing.

---

## No Default Route

This is expected on an isolated host-only interface.

A NAT-connected system requiring internet access should have a default route through the NAT adapter.

Windows:

```
Get-NetRoute |
    Where-Object DestinationPrefix -eq "0.0.0.0/0"
```

Linux:

```
ip route |
    grep default
```

---

## Multiple Default Routes

Symptoms:

* Intermittent internet access
* Internal traffic uses NAT
* DNS leaves through the wrong adapter
* Return path fails
* Agent connections are unstable

Only the intended external adapter should normally provide the default route.

Review interface metrics before deleting routes.

---

## Host-Only Works but NAT Does Not

Check:

* NAT adapter is connected.
* VMware DHCP assigned an address.
* A default route exists.
* VMware NAT service is running.
* The host has internet access.
* A VPN is not interfering.
* DNS works separately from IP connectivity.

Test IP:

```
ping -c 4 1.1.1.1
```

Test DNS:

```
dig example.com
```

---

## NAT Works but Host-Only Does Not

Check:

* Correct host-only VMnet
* Static address
* Subnet
* Windows or Linux firewall
* VMware host adapter
* Target VM state
* VPN route overlap

Use service-port tests rather than relying only on ping.

---

## VPN Interference

A VPN on the Acer host may affect:

* VMware NAT
* Host-only routes
* DNS
* Interface metrics
* Browser access
* Split tunneling

Test by:

1. Recording the current state.
2. Disconnecting the VPN temporarily.
3. Retesting the lab path.
4. Reconnecting the VPN.
5. Comparing routes.

Do not permanently disable security software without understanding the impact.

---

# DNS Troubleshooting

## DNS Troubleshooting Principle

If IP connectivity works but hostnames fail, investigate DNS.

Do not replace internal Active Directory DNS with a public resolver as a workaround.

---

## Windows DNS Checks

```
Get-DnsClientServerAddress
ipconfig /all
Resolve-DnsName DC01.cyberlab.example
nslookup cyberlab.example
```

---

## Linux DNS Checks

```
resolvectl status
dig cyberlab.example
dig @<DOMAIN_CONTROLLER> cyberlab.example
nslookup cyberlab.example <DOMAIN_CONTROLLER>
```

---

## Endpoint Uses the Wrong DNS Server

Symptoms:

* Domain join fails
* DC01 cannot be discovered
* Group Policy fails
* Kerberos fails
* Internal names do not resolve

WIN11TARGET should use DC01 as its DNS server.

Correct with administrator PowerShell:

```
Set-DnsClientServerAddress `
    -InterfaceAlias "CyberLab-Internal" `
    -ServerAddresses "<DOMAIN_CONTROLLER_IP>"
```

---

## NAT Adapter Overrides DNS

Check:

```
Get-DnsClientServerAddress
```

Review interface metrics:

```
Get-NetIPInterface |
    Sort-Object InterfaceMetric
```

Disable DNS registration on the NAT adapter:

```
Set-DnsClient `
    -InterfaceAlias "<NAT_ADAPTER>" `
    -RegisterThisConnectionsAddress $false
```

---

## DC01 Resolves to the Wrong Address

This may happen when a temporary NAT adapter registers itself in DNS.

Check:

```
Resolve-DnsName DC01.cyberlab.example
```

Review DNS records on DC01.

Corrective process:

1. Disable DNS registration on the NAT adapter.
2. Remove the incorrect DNS record.
3. Confirm the host-only record.
4. Restart Netlogon.
5. Register DNS again.
6. Clear client DNS caches.
7. Retest.

---

## Clear Windows DNS Cache

```
ipconfig /flushdns
```

Register the local system:

```
ipconfig /registerdns
```

Administrator privileges may be required for complete registration behavior.

---

## Active Directory Service Records Missing

Check:

```
Resolve-DnsName `
    "_ldap._tcp.dc._msdcs.cyberlab.example" `
    -Type SRV
```

Restart Netlogon on DC01:

```
Restart-Service Netlogon
```

Then:

```
ipconfig /registerdns
```

Review DNS and Directory Service logs.

---

# Firewall Troubleshooting

## Do Not Disable the Firewall First

Before modifying firewall rules:

1. Confirm addressing.
2. Confirm routes.
3. Confirm DNS.
4. Confirm the service is listening.
5. Test the required port.
6. Review the active firewall profile.
7. Add a narrow rule if required.

Disabling the entire firewall can hide the real cause and reduce lab safety.

---

## Windows Firewall Review

```
Get-NetFirewallProfile
Get-NetConnectionProfile
Get-NetFirewallRule |
    Where-Object Enabled -eq "True"
```

Test a port:

```
Test-NetConnection <TARGET> -Port <PORT>
```

---

## Linux Firewall Review

UFW:

```
sudo ufw status verbose
```

nftables:

```
sudo nft list ruleset
```

Listeners:

```
sudo ss -tulpn
```

---

## Service Is Listening Locally but Not Remotely

Possible causes:

* Bound only to loopback
* Firewall rule
* Wrong interface
* Wrong port
* Container port not published
* Application ACL
* Host-only path failure

Local Linux test:

```
curl -I http://127.0.0.1:<PORT>
```

Remote Windows test:

```
Test-NetConnection <SERVER> -Port <PORT>
```

Compare the results.

---

# Active Directory Troubleshooting

## Domain Join Fails

Check:

* Windows edition supports joining
* DC01 is running
* DNS points to DC01
* Domain resolves
* SRV records resolve
* Time is synchronized
* Required ports are reachable
* Computer name is unique
* Credentials have permission
* No pending restart

Commands:

```
Get-ComputerInfo |
    Select-Object WindowsProductName, CsName, CsDomain
```

```
Resolve-DnsName cyberlab.example
```

```
nltest /dsgetdc:cyberlab.example
```

```
w32tm /query /status
```

---

## Trust Relationship Error

Symptom:

```
The trust relationship between this workstation and the primary domain failed.
```

Test:

```
Test-ComputerSecureChannel -Verbose
```

Repair:

```
Test-ComputerSecureChannel `
    -Repair `
    -Credential "CYBERLAB\<AUTHORIZED_ADMIN>"
```

Administrator privileges and domain permissions are required.

If repair fails, a controlled domain leave and rejoin may be required.

---

## Domain User Cannot Sign In

Check:

* DC01 availability
* DNS
* User enabled state
* Account lockout
* Password state
* Time
* Secure channel
* Correct sign-in format
* Network profile

Use the local recovery account when required:

```
.\localadmin
```

---

## Group Policy Does Not Apply

Run:

```
gpupdate /force
```

Review:

```
gpresult /r
```

Generate report:

```
gpresult /h "<REPORT_PATH>"
```

Check:

* OU placement
* GPO link
* Security filtering
* WMI filter
* DNS
* Secure channel
* User versus computer policy
* Event logs

---

## Network Profile Remains Public

Check:

```
Get-NetConnectionProfile
```

Then:

```
Resolve-DnsName DC01.cyberlab.example
Test-ComputerSecureChannel -Verbose
Get-Service NlaSvc, Netlogon
```

Windows determines the DomainAuthenticated profile automatically.

Do not force it manually as a substitute for fixing domain detection.

---

## Account Is Locked

On DC01:

```
Search-ADAccount -LockedOut
```

Unlock the intended test account:

```
Unlock-ADAccount `
    -Identity "<TEST_ACCOUNT>"
```

Investigate repeated failures before unlocking during a detection exercise.

---

## SYSVOL or NETLOGON Missing

Check:

```
net share
```

Review services:

```
Get-Service NTDS, Netlogon, DFSR
```

Run:

```
dcdiag /v
```

Review:

* Directory Service log
* DFS Replication log
* DNS
* Promotion status

Do not manually create the shares.

---

## Kerberos Problems

Common causes:

* Time difference
* DNS failure
* Duplicate name
* Broken secure channel
* Snapshot rollback
* SPN problem

Check:

```
w32tm /query /status
Resolve-DnsName DC01.cyberlab.example
setspn -L DC01
```

---

# Windows Endpoint Troubleshooting

## Endpoint Has No Internal Connectivity

Review:

```
Get-NetAdapter
Get-NetIPConfiguration
Get-NetRoute
Get-DnsClientServerAddress
```

Confirm:

* Host-only adapter connected
* Correct static address
* Correct subnet
* DNS points to DC01
* No unintended gateway
* Target systems running

---

## Endpoint Has No Internet Access

Check the temporary NAT adapter:

```
Get-NetIPConfiguration
Get-NetRoute
```

Test:

```
Test-NetConnection 1.1.1.1 -Port 443
Resolve-DnsName example.com
```

Do not add a gateway to the host-only adapter.

---

## Defender Is Disabled or Unhealthy

Review:

```
Get-MpComputerStatus
```

Check:

* Antivirus enabled
* Real-time protection
* Signature age
* Service state
* Policy
* Third-party antivirus conflict

Update signatures:

```
Update-MpSignature
```

Administrator privileges may be required.

---

## Security Events Are Missing

Check:

```
auditpol /get /category:*
```

```
Get-WinEvent -ListLog Security
```

```
gpresult /r
```

Confirm the test actually generated the expected event.

---

## PowerShell Events Are Missing

Review:

```
Get-WinEvent `
    -ListLog "Microsoft-Windows-PowerShell/Operational"
```

Check:

* Channel enabled
* Group Policy
* Script block logging
* Module logging
* Process creation auditing
* Test time range

---

## Endpoint Performance Is Poor

Review:

```
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10
```

```
Get-Counter '\Memory\Available MBytes'
```

Potential causes:

* Windows updates
* Defender scan
* Too little memory
* Too many virtual CPUs
* Wazuh activity
* Splunk forwarder activity
* Sysmon volume
* Long snapshot chain
* Host storage pressure

---

# Wazuh Troubleshooting

## Wazuh Dashboard Does Not Load

Check the VM:

```
ip address
ip route
```

Check Docker:

```
sudo systemctl status docker
```

Check containers:

```
docker compose ps
```

Check listeners:

```
sudo ss -tulpn
```

Review logs:

```
docker compose logs --tail 100
```

---

## Wazuh Container Restarts Repeatedly

Review:

```
docker ps
docker inspect <CONTAINER_NAME>
docker logs --tail 200 <CONTAINER_NAME>
```

Possible causes:

* Invalid configuration
* Missing certificate
* Wrong credentials
* Permission problem
* Low memory
* Low disk space
* Port conflict
* Dependency failure
* Corrupted index data

---

## Wazuh Indexer Unhealthy

Symptoms:

* Dashboard errors
* Login failure
* Missing alerts
* Search failure
* Restart loops

Check:

* Indexer logs
* Memory
* Disk space
* Certificates
* Credentials
* File ownership
* Time
* Container network
* Index health

The indexer is often the most resource-intensive Wazuh component.

---

## Wazuh Agent Missing

On Windows:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Test connectivity:

```
Test-NetConnection <WAZUH_SERVER> -Port <WAZUH_AGENT_PORT>
```

Check:

* Manager address
* Enrollment
* Firewall
* Agent name
* Agent key
* Duplicate record
* Endpoint time
* Host-only route

---

## Wazuh Agent Shows Disconnected

Possible causes:

* Endpoint powered off
* Agent service stopped
* Manager down
* Firewall change
* Adapter disconnected
* Duplicate identity
* Snapshot restore
* Agent key mismatch
* Time problem

Review the endpoint agent logs before reinstalling.

---

## Wazuh Event Missing

Confirm:

1. The event exists locally.
2. The event channel is collected.
3. The agent is active.
4. The decoder recognizes the event.
5. A rule exists.
6. The dashboard time range is correct.
7. The agent filter is correct.

A missing alert may be a rule-coverage issue rather than ingestion failure.

---

## File Integrity Event Missing

Check:

* Path exists
* Path is configured
* Agent restarted
* Scan interval
* Real-time mode
* Exclusions
* File actually changed
* Time range

Use a simple dedicated test file.

---

## Wazuh Disk Space Low

```
df -h
docker system df
```

Inspect large directories:

```
sudo du -xh /var/lib/docker |
    sort -h |
    tail
```

Do not delete Docker volumes or Wazuh index data without a recovery plan.

---

# Splunk Troubleshooting

## Splunk Command Not Found

Use the full path:

```
sudo /opt/splunk/bin/splunk status
```

The Splunk binary may not be in the shell `PATH`.

---

## Splunk Will Not Start

Check:

```
sudo /opt/splunk/bin/splunk status
```

Start using the documented current lab command:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

Review:

```
sudo tail -n 200 \
    /opt/splunk/var/log/splunk/splunkd.log
```

Possible causes:

* Ownership problem
* Disk full
* Invalid configuration
* Port conflict
* License problem
* Memory pressure
* Corrupted index
* Stale lock file

---

## Splunk Requires `--run-as-root`

The current lab installation successfully starts with:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

Investigate:

* `/opt/splunk` ownership
* Log ownership
* Index ownership
* Boot service
* Runtime account
* Protected inputs
* Previous root-created files

Do not recursively change ownership without a backup and documented target account.

---

## Splunk Web Does Not Load

Check:

```
sudo /opt/splunk/bin/splunk status
```

Check port:

```
sudo ss -tulpn |
    grep 8000
```

Test locally:

```
curl -I http://127.0.0.1:8000
```

Test from Windows:

```
Test-NetConnection <SPLUNK_SERVER> -Port 8000
```

If local access works but remote access fails, investigate:

* Firewall
* Interface binding
* VMware networking
* Wrong address
* Host VPN

---

## Port 8000 Is Not Listening

Review:

```
sudo tail -n 200 \
    /opt/splunk/var/log/splunk/splunkd.log
```

Review effective web configuration:

```
sudo /opt/splunk/bin/splunk btool web list --debug
```

Possible causes:

* Splunk stopped
* Web disabled
* Port changed
* Port conflict
* Configuration error
* Permission issue

---

## Splunk Permission Denied

Review:

```
sudo ls -ld /opt/splunk
```

```
sudo ls -l /opt/splunk/var/log/splunk |
    head
```

Review running account:

```
ps aux |
    grep -i splunk
```

A common cause is switching between root and a dedicated service account.

---

## Splunk Search Returns No Results

Start broad within the expected index:

```spl
index=<INDEX_NAME> earliest=-24h
```

Then:

```spl
index=<INDEX_NAME> earliest=-24h
| stats count by host source sourcetype
```

Check:

* Time range
* Index
* Host
* Source
* Sourcetype
* Event field
* Permissions
* Forwarder
* Receiver
* Timestamp parsing

---

## Splunk Forwarder Not Sending Data

On Windows:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Test receiver:

```
Test-NetConnection `
    <SPLUNK_SERVER> `
    -Port <SPLUNK_RECEIVER_PORT>
```

Check:

* `inputs.conf`
* `outputs.conf`
* Receiver enabled
* Target index
* Firewall
* Forwarder logs
* Endpoint time
* Hostname identity

---

## Splunk Duplicate Events

Possible causes:

* Duplicate inputs
* File replay
* Forwarder checkpoint rollback
* Snapshot restoration
* Re-uploaded test data
* Multiple forwarders

Search:

```spl
index=<INDEX_NAME>
| stats count by _time host source EventCode _raw
| where count > 1
```

Confirm the events are truly duplicates before changing collection.

---

## Splunk Events Have Wrong Time

Compare:

```spl
index=<INDEX_NAME>
| eval delay=_indextime-_time
| table _time _indextime delay host source _raw
```

Check:

* Source system time
* Splunk server time
* Timezone
* Sourcetype
* Timestamp extraction
* Snapshot restoration
* Missing year or date

---

## Splunk Disk Usage High

```
df -h
```

```
sudo du -sh /opt/splunk/var/lib/splunk/*
```

Possible causes:

* High ingestion
* Long retention
* Duplicate input
* Test datasets
* Internal logs
* Large lookups
* Snapshot growth

Do not manually delete index directories.

---

# Kali Troubleshooting

## Kali Cannot Reach the Target

Check:

```
ip address
ip route
nmcli device status
ip neigh
```

Confirm:

* Host-only adapter connected
* Correct VMnet
* Correct target
* Target VM running
* Same subnet
* Target firewall
* No accidental NAT address used

---

## Kali Has No Internet

Check:

```
ip address
ip route
resolvectl status
```

Test IP:

```
ping -c 4 1.1.1.1
```

Test DNS:

```
dig example.com
```

Confirm NAT adapter and default route.

---

## Nmap Finds No Ports

Possible causes:

* Wrong target
* Target offline
* Firewall
* Service not listening
* Wrong network
* Scan type requires privilege
* NAT address used
* Port list incorrect

Validate one known port:

```
nc -vz <AUTHORIZED_TARGET> <KNOWN_PORT>
```

---

## Packet Capture Shows Nothing

List interfaces:

```
sudo tcpdump -D
```

Capture a small sample:

```
sudo tcpdump `
    -i <HOST_ONLY_INTERFACE> `
    -c 20
```

On Linux shells, use backslashes rather than PowerShell backticks:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    -c 20
```

Confirm:

* Correct interface
* Traffic generated
* Filter not too narrow
* Capture permissions
* Correct source and target

---

## Tool Requires Root

Use:

```
sudo <COMMAND>
```

only after confirming why elevated privileges are required.

Do not run the full desktop as root.

---

## Wrong Network Was Scanned

Stop immediately.

Then:

1. Stop the scan process.
2. Disconnect Kali if required.
3. Record the command.
4. Record the unintended target range.
5. Review whether non-lab systems were contacted.
6. Correct the adapter or scope.
7. Add a preventive control.
8. Do not repeat the exercise until validated.

---

# Log Ingestion Troubleshooting

## Missing Event Decision Process

```
Expected Event Missing
        |
        v
Did the event occur locally?
        |
   No --+--> Fix audit policy or test method
        |
       Yes
        |
        v
Is the collection service running?
        |
   No --+--> Start or repair agent/forwarder
        |
       Yes
        |
        v
Is the receiver reachable?
        |
   No --+--> Fix network or firewall
        |
       Yes
        |
        v
Was the event indexed or stored?
        |
   No --+--> Fix input, parsing, or receiver
        |
       Yes
        |
        v
Fix search, filter, rule, or dashboard
```

---

## Local Event Missing

Check:

* Audit policy
* Group Policy
* Event channel
* Test method
* Event-log service
* Event retention
* User context
* Required privilege

The SIEM cannot ingest an event that does not exist locally.

---

## Agent Running but No Data

A running service does not guarantee:

* Correct destination
* Successful authentication
* Required input
* Network connectivity
* Correct identity
* Parser compatibility

Validate each stage separately.

---

## Receiver Reachable but No Events

Possible causes:

* Input not configured
* Wrong event channel
* Wrong index
* Wrong manager address
* Wrong agent key
* Parsing failure
* Time range
* Event below alert threshold
* Data sent to another host identity

---

## Data Appears Under the Wrong Host

Check:

* Cloned VM identity
* Forwarder hostname
* Agent name
* Host override
* Snapshot restore
* Incorrect configuration
* Duplicate endpoint

Correct future data while preserving historical evidence.

---

## Missing Fields

Review the raw event first.

Possible causes:

* Incorrect sourcetype
* Missing add-on
* Decoder mismatch
* Different event format
* Field extraction failure
* Source collected as raw text

Do not build a field extraction from only one sample.

---

## High Ingestion Delay

Check:

* Source time
* Agent backlog
* Forwarder queues
* Network interruptions
* Server CPU
* Server memory
* Disk space
* Indexer health
* Service restart history

Measure delay rather than guessing.

---

# Browser and Dashboard Troubleshooting

## Browser Page Does Not Load

Check:

* Correct protocol
* Correct host
* Correct port
* Service running
* Port listening
* Firewall
* VPN
* Browser proxy
* Certificate warning
* Cached redirect

Test with:

```
Test-NetConnection <SERVER> -Port <PORT>
```

Or:

```
curl -I http://<SERVER>:<PORT>
```

---

## Browser Works Locally but Not from the Host

This usually indicates:

* Firewall
* Wrong binding
* Wrong interface
* Host-only route
* VPN
* Browser proxy

Compare local and remote tests.

---

## Certificate Warning

Possible causes:

* Self-signed certificate
* Hostname mismatch
* Expired certificate
* Wrong time
* Untrusted authority
* Connecting by IP instead of certificate hostname

Verify the target before bypassing a warning.

---

## Login Fails

Check:

* Username
* Password
* Keyboard layout
* Caps Lock
* Account lockout
* Service status
* Backend availability
* Password rotation
* Snapshot restore
* Browser session

Do not repeatedly guess administrative credentials.

---

# Docker Troubleshooting

## Docker Service Is Stopped

```
sudo systemctl status docker
```

Start:

```
sudo systemctl start docker
```

Enable at boot when appropriate:

```
sudo systemctl enable docker
```

---

## Docker Permission Denied

Symptom:

```
permission denied while trying to connect to the Docker daemon socket
```

Use:

```
sudo docker ps
```

Or confirm intentional Docker group membership:

```
groups
```

Docker group access is effectively administrative.

---

## Container Port Not Accessible

Check:

```
docker ps
```

```
docker inspect <CONTAINER_NAME>
```

```
sudo ss -tulpn
```

Confirm:

* Port published
* Correct host interface
* Container healthy
* Host firewall
* Application listening inside container

---

## Container Logs

```
docker logs --tail 200 <CONTAINER_NAME>
```

Compose:

```
docker compose logs --tail 200 <SERVICE_NAME>
```

Follow:

```
docker compose logs -f <SERVICE_NAME>
```

Logs may contain secrets and internal details.

---

## Do Not Delete Volumes During Routine Repair

Avoid:

```
docker compose down -v
```

unless data deletion is intentional.

The `-v` option can remove application data.

---

# Service Troubleshooting

## Windows Service Review

```
Get-Service
```

Specific services:

```
Get-Service `
    NTDS,
    DNS,
    Netlogon,
    W32Time
```

Start:

```
Start-Service <SERVICE_NAME>
```

Restart:

```
Restart-Service <SERVICE_NAME>
```

Administrator privileges may be required.

---

## Linux Service Review

```
systemctl status <SERVICE>
```

Start:

```
sudo systemctl start <SERVICE>
```

Restart:

```
sudo systemctl restart <SERVICE>
```

Recent logs:

```
journalctl -u <SERVICE> --since "15 minutes ago"
```

---

## Service Starts and Stops Immediately

Possible causes:

* Invalid configuration
* Port conflict
* Permission problem
* Missing dependency
* Disk full
* Missing certificate
* Bad credential
* Corrupted state

Review service logs before restarting repeatedly.

---

# Performance Troubleshooting

## High CPU

Windows:

```
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10
```

Linux:

```
top
```

Or:

```
ps aux \
    --sort=-%cpu |
    head
```

Potential causes:

* Expensive Splunk search
* Wazuh indexing
* Defender scan
* Windows update
* Large Nmap operation
* Container restart loop
* Host overcommitment

---

## High Memory Use

Windows:

```
Get-Counter '\Memory\Available MBytes'
```

Linux:

```
free -h
```

Docker:

```
docker stats
```

Potential repairs:

* Stop unused VMs.
* Close expensive searches.
* Reduce unnecessary event collection.
* Review container health.
* Increase VM memory only after identifying demand.
* Avoid assigning excessive memory to every VM.

---

## High Disk Activity

Check:

* Windows updates
* Defender
* Wazuh indexer
* Splunk indexing
* Snapshot consolidation
* Backup copy
* Large packet capture
* Guest swap
* Host paging

Do not force shutdown during consolidation or active index writes.

---

## Slow SIEM Searches

Possible causes:

* Broad index wildcard
* Large time range
* No index filter
* Too many fields
* Expensive regular expression
* Resource pressure
* Disk latency
* Excessive event volume

Narrow by:

* Index
* Host
* Sourcetype
* Event code
* Time range

---

# Recovery and Rollback

## When to Repair

Prefer direct repair when:

* Root cause is known
* Change is small
* Evidence must be preserved
* Recent data is important
* Snapshot rollback would affect dependencies

---

## When to Restore a Snapshot

Consider restore when:

* Configuration damage is extensive
* A known-good snapshot exists
* Evidence has been preserved
* Recent state can be lost
* Dependency risks are understood
* Repair would take longer or be less reliable

---

## When to Restore a Cold Backup

Use an independent backup when:

* VM files are corrupted
* Host storage failed
* Snapshot chain is broken
* VM directory was deleted
* Host compromise is suspected
* A clean independent state is required

---

## Post-Repair Validation

After every repair:

1. Confirm system time.
2. Confirm networking.
3. Confirm DNS.
4. Confirm required services.
5. Confirm application access.
6. Confirm agent or forwarder.
7. Generate a known test event.
8. Confirm SIEM ingestion.
9. Record the result.
10. Update the runbook.

---

# Troubleshooting Runbooks

Recommended focused runbooks include:

```
runbooks/
├── vm-will-not-start.md
├── vm-unreachable.md
├── domain-join-failure.md
├── repair-domain-secure-channel.md
├── dns-resolution-failure.md
├── wazuh-dashboard-down.md
├── wazuh-agent-disconnected.md
├── splunk-web-down.md
├── splunk-forwarder-not-sending.md
├── missing-security-events.md
├── restore-vm-snapshot.md
└── low-disk-space.md
```

This document provides the broad decision framework.

Runbooks should provide shorter operational procedures for specific incidents.

---

## Runbook Template

```
Title:

Purpose:

Symptoms:

Affected systems:

Prerequisites:

Evidence to preserve:

Diagnostic steps:

Recovery steps:

Validation:

Rollback:

Escalation conditions:

Outcome:
```

---

# Change Management

## Document Troubleshooting Changes

Record changes to:

* Network configuration
* DNS
* Firewall
* Services
* Agent configuration
* Forwarder configuration
* Docker Compose
* Wazuh rules
* Splunk inputs
* Group Policy
* Audit policy
* System time
* VM snapshots

---

## Change Record Template

```
Date:

System:

Symptom:

Change:

Reason:

Previous state:

New state:

Backup or snapshot:

Validation:

Rollback:

Outcome:
```

---

## One Change at a Time

Avoid changing:

* DNS
* Firewall
* Routes
* Service configuration
* Agent settings

all at once.

Multiple simultaneous changes make it difficult to identify the real cause.

---

# Public Sanitization Standards

Remove or replace:

* Real internal IP addresses
* Real domain names
* Personal usernames
* Email addresses
* Passwords
* API tokens
* Session identifiers
* Agent keys
* Forwarder credentials
* MAC addresses
* VMware UUIDs
* Personal file paths
* Home-network details
* Public IP addresses
* VPN information
* Raw event exports
* Packet captures
* License information
* Certificate private keys
* Recovery keys

Use placeholders such as:

```
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<KALI_SYSTEM>
<HOST_ONLY_INTERFACE>
<NAT_INTERFACE>
<PORT>
<INDEX_NAME>
<EVENT_ID>
<AUTHORIZED_ADMIN>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing troubleshooting screenshots:

1. Close unrelated applications.
2. Hide browser tabs.
3. Hide bookmarks.
4. Clear notifications.
5. Remove usernames.
6. Remove domain names.
7. Remove IP addresses.
8. Remove MAC addresses.
9. Remove session data.
10. Remove tokens and passwords.
11. Crop unrelated systems.
12. Permanently redact sensitive values.
13. Reopen the final image.
14. Confirm the screenshot supports the troubleshooting finding.

---

# Troubleshooting Checklist

## Initial Checks

* Symptom clearly defined
* Affected system identified
* Time recorded
* Recent changes reviewed
* Evidence preserved
* VM power state confirmed
* Host resources checked

## Network

* Adapter connected
* Correct VMnet
* Interface enabled
* IP address correct
* Subnet correct
* Route correct
* DNS correct
* Port tested

## Service

* Service installed
* Service running
* Listener present
* Firewall permits access
* Logs reviewed
* Dependencies available

## Identity

* DC01 available
* DNS resolves
* Time synchronized
* Secure channel healthy
* Account enabled
* Account not locked
* Group Policy applied

## Monitoring

* Wazuh agent active
* Wazuh stack healthy
* Splunk service active
* Forwarder active
* Receiver listening
* Test event generated
* Event searchable

## Recovery

* Snapshot identified
* Evidence exported
* Backup available
* Rollback understood
* Repair validated
* Root cause documented

---

# Skills Demonstrated

This troubleshooting framework demonstrates:

* Layered network troubleshooting
* VMware administration
* Windows administration
* Linux administration
* Active Directory troubleshooting
* DNS troubleshooting
* Routing analysis
* Firewall analysis
* Service management
* Wazuh troubleshooting
* Splunk troubleshooting
* Docker troubleshooting
* SIEM ingestion validation
* Log analysis
* Performance analysis
* Snapshot recovery
* Root-cause analysis
* Change management
* Technical runbook development
* Public sanitization

---

# Lessons Learned

Key lessons include:

* Troubleshooting should begin with a clearly defined symptom.
* The lowest failing layer should be corrected first.
* DNS problems often appear as Active Directory problems.
* A running service may still be unhealthy or unreachable.
* A successful port test does not prove data is being ingested.
* A failed ping does not prove the application is unavailable.
* Time differences can break authentication and event correlation.
* Multiple default routes can create intermittent behavior.
* Root and service-account ownership should remain consistent.
* Wazuh indexer health affects the entire dashboard experience.
* Splunk Web availability does not prove indexing works.
* Snapshot restoration can introduce trust, identity, and timestamp issues.
* Restarting too early may destroy useful evidence.
* One controlled change at a time improves root-cause accuracy.
* Every repair should end with a known-event validation.
* Public troubleshooting evidence requires aggressive sanitization.

---

# Planned Improvements

Future improvements may include:

* Individual incident runbooks
* Automated network validation scripts
* Automated DNS health checks
* Automated DC01 health report
* Wazuh container-health script
* Splunk health-check script
* Agent and forwarder monitoring
* Disk-space alerts
* Snapshot inventory
* Centralized troubleshooting checklist
* Baseline configuration exports
* Automated known-event generation
* Log-ingestion health dashboard
* Packet-capture sensor
* Suricata integration
* Zeek integration
* Configuration drift detection
* Formal incident log
* Recovery test schedule
* Troubleshooting decision diagrams

---

# Summary

Effective CyberLab troubleshooting requires a structured, layered process.

The recommended method is to:

* Define the symptom
* Preserve evidence
* Confirm power and VM state
* Validate networking
* Validate DNS
* Validate firewall behavior
* Validate services
* Review application logs
* Test agents and forwarders
* Generate a known event
* Confirm end-to-end ingestion
* Document the root cause and repair

By troubleshooting from the infrastructure layer upward, the Acer Blue Team CyberLab can be repaired more safely, efficiently, and consistently without relying on unnecessary reinstalls or destructive resets.

