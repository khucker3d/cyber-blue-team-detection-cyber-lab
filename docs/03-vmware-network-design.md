# VMware Network Design

## Overview

The Acer Blue Team CyberLab uses VMware Workstation virtual networking to isolate lab systems, provide controlled internet access, and support communication between defensive security components.

The network design separates:

* Internal CyberLab traffic
* Internet-bound traffic
* Identity and DNS services
* Security monitoring traffic
* Administrative access
* Controlled adversary simulation

The environment is designed to avoid placing lab virtual machines directly on the physical home network unless a specific exercise requires it.

All IP addresses, subnets, hostnames, domain names, and interface details shown in this document are sanitized examples.

---

## Network Design Goals

The VMware network design was created to support the following goals:

* Isolate lab traffic from household devices
* Provide predictable addressing
* Support Active Directory and DNS
* Allow Wazuh and Splunk to collect telemetry
* Permit controlled testing from Kali Linux
* Provide internet access only when required
* Reduce accidental exposure
* Simplify troubleshooting
* Support repeatable exercises
* Keep the public documentation safe to publish

---

## VMware Network Types Used

The CyberLab uses two primary VMware network types:

* Host-only networking
* NAT networking

Bridged networking is intentionally avoided for normal lab operation.

---

## High-Level Network Layout

```
                               Internet
                                  |
                         Physical Home Router
                                  |
                         Acer Windows Host
                                  |
                         VMware Workstation
                                  |
               ___________________|___________________
              |                                       |
         VMware NAT                              Host-Only Network
         VMnet8                                  VMnet1
              |                                       |
              |                 ______________________|____________________
              |                |              |              |            |
         Wazuh Server         DC01       WIN11TARGET     Splunk Server   Kali Linux
         Splunk Server
         Kali Linux
```

The NAT network provides controlled outbound connectivity.

The host-only network provides isolated communication between lab systems and the Acer host.

---

## Public Example Addressing

The following values are documentation examples only.

```
Host-only subnet: 192.0.2.0/24
NAT subnet: 198.51.100.0/24
Internal domain: cyberlab.example
```

Example host-only assignments:

| System              | Example Address |
| ------------------- | --------------- |
| Domain Controller   | 192.0.2.10      |
| Windows Endpoint    | 192.0.2.20      |
| Wazuh Server        | 192.0.2.30      |
| Splunk Server       | 192.0.2.40      |
| Kali Linux          | 192.0.2.50      |
| VMware Host Adapter | 192.0.2.1       |

The `192.0.2.0/24` and `198.51.100.0/24` ranges are reserved for documentation.

They do not represent the operational CyberLab.

---

## Host-Only Network

## Purpose

The host-only network is the primary internal CyberLab network.

It supports communication between:

* Domain controller
* Windows endpoint
* Wazuh server
* Splunk server
* Kali Linux
* Acer host administration tools

The host-only network is isolated from the physical home network by default.

---

## Host-Only Characteristics

The host-only network provides:

* Layer 2 communication between VMs
* Communication between VMs and the host
* No direct internet route by default
* No direct access from household devices
* Predictable internal addressing
* A safer environment for controlled testing

This network is used for most identity, monitoring, administration, and security exercise traffic.

---

## VMware Host Adapter

VMware creates a virtual network adapter on the Windows host for the host-only network.

The adapter may appear with a name similar to:

```
VMware Network Adapter VMnet1
```

The host adapter allows the Acer system to communicate with the virtual machines.

Example documentation address:

```
192.0.2.1
```

The real adapter address should not be published in public screenshots unless it has been replaced with a placeholder.

---

## Host-Only DHCP

VMware can provide DHCP on the host-only network.

There are two common design choices:

### VMware DHCP Enabled

Advantages:

* Faster initial setup
* Automatic address assignment
* Useful for temporary test systems
* Less manual configuration

Disadvantages:

* Addresses may change
* Infrastructure systems may become harder to locate
* DNS and domain services may be less predictable
* Documentation can become inconsistent

### VMware DHCP Disabled

Advantages:

* Predictable addressing
* Better for infrastructure systems
* Easier SIEM and DNS configuration
* Easier troubleshooting
* More stable documentation

Disadvantages:

* Every system must be configured manually
* Incorrect static settings can cause conflicts
* More administrative work

A mixed approach can also be used.

Infrastructure systems may use static addresses while temporary test systems use DHCP.

---

## Recommended Addressing Strategy

Stable infrastructure systems should use static addresses or documented reservations.

Recommended examples:

| System Type       | Addressing Method     |
| ----------------- | --------------------- |
| Domain controller | Static                |
| Wazuh server      | Static                |
| Splunk server     | Static                |
| Windows endpoint  | Static or reservation |
| Kali Linux        | DHCP or static        |
| Temporary test VM | DHCP                  |

Static infrastructure addresses reduce the chance of broken DNS, agent, and dashboard configurations.

---

## NAT Network

## Purpose

The VMware NAT network provides outbound internet access through the Acer host.

It is used for:

* Operating system updates
* Package installation
* Software downloads
* Repository access
* Container image downloads
* Threat intelligence updates
* Time synchronization

The virtual machines do not normally receive direct inbound access from the physical network.

---

## NAT Characteristics

The VMware NAT network:

* Translates guest traffic through the host
* Uses a private VMware-managed subnet
* Provides a default gateway
* Can provide DHCP
* Allows outbound internet access
* Reduces direct exposure compared with bridged networking

A NAT-connected VM is still capable of reaching external systems, so internet access should be limited to legitimate lab needs.

---

## VMware NAT Adapter

The Windows host typically has a virtual adapter associated with the NAT network.

It may appear as:

```
VMware Network Adapter VMnet8
```

The guest default gateway is provided by VMware.

The exact gateway and subnet should be treated as sensitive internal configuration in public documentation.

---

## NAT DHCP

VMware normally provides DHCP on the NAT network.

This is suitable for temporary internet access because NAT addresses usually do not need to remain stable.

A VM connected to both host-only and NAT networks should use:

* A stable internal address on host-only
* A VMware-assigned address on NAT

The NAT adapter should normally provide the default route.

---

## Why Bridged Networking Is Avoided

Bridged networking connects a VM directly to the physical network.

A bridged VM may be able to communicate with:

* Personal computers
* Mobile devices
* Smart home devices
* Network infrastructure
* Other trusted systems
* Internet-facing services depending on the router configuration

This increases the chance of:

* Accidental scanning of non-lab systems
* Unintended service exposure
* DHCP conflicts
* Home-network visibility
* Cross-network infection
* Misconfigured firewall rules

Bridged networking is not required for most Blue Team exercises.

Host-only and NAT networking provide a safer default architecture.

---

## When Bridged Networking May Be Appropriate

Bridged networking may be used only when a documented exercise requires:

* Direct physical-network visibility
* Testing with a dedicated VLAN
* Monitoring traffic from a controlled segment
* Interacting with a dedicated lab appliance
* Integration with physical security hardware

Before enabling bridged networking:

1. Confirm the target network is authorized.
2. Confirm non-lab devices are protected.
3. Document the intended traffic flow.
4. Review guest firewall rules.
5. Use a dedicated VLAN where practical.
6. Disable bridging after the exercise.
7. Validate that no service remains exposed.

---

## Virtual Network Adapter Matrix

| System            | Host-Only |       NAT |       Bridged |
| ----------------- | --------: | --------: | ------------: |
| Domain Controller |       Yes |  Optional |            No |
| Windows Endpoint  |       Yes |  Optional |            No |
| Wazuh Server      |       Yes |       Yes |            No |
| Splunk Server     |       Yes |  Optional |            No |
| Kali Linux        |       Yes |  Optional |            No |
| Temporary Test VM | As needed | As needed | No by default |

Each VM should receive only the adapters required for its role.

---

## Domain Controller Network Design

The domain controller should use the host-only network as its primary interface.

Recommended configuration:

```
Host-only adapter:
- Static internal address
- Internal subnet mask
- No default gateway for isolated operation
- Preferred DNS set to itself
```

A temporary NAT adapter may be added for updates.

If the domain controller is multi-homed, routing and DNS behavior must be reviewed carefully.

---

## Domain Controller DNS

The domain controller should use its own DNS service for the internal domain.

Example:

```
Preferred DNS: 192.0.2.10
Alternate DNS: blank or another approved internal DNS server
```

Do not configure a domain controller to use a public resolver directly as its primary DNS server.

External DNS requests should be handled through DNS forwarders.

---

## Windows Endpoint Network Design

The Windows endpoint should use the host-only network for domain communication.

Recommended internal settings:

```
IP address: 192.0.2.20
Subnet mask: 255.255.255.0
Default gateway: blank for isolated operation
Preferred DNS: 192.0.2.10
```

A NAT adapter may be added temporarily for updates or software downloads.

---

## Windows Endpoint DNS Requirement

The endpoint must use the domain controller for DNS.

Using a public resolver can cause:

* Domain join failures
* Missing domain controller records
* Kerberos failures
* Group Policy failures
* Slow authentication
* Incorrect host resolution

The endpoint should not use the NAT adapter’s DNS server as its primary resolver while performing domain tasks.

---

## Wazuh Server Network Design

The Wazuh server typically uses two virtual adapters.

### Internal Adapter

Used for:

* Agent enrollment
* Security telemetry
* Dashboard access
* Administrative access
* Communication with monitored endpoints

Example:

```
Interface: host-only
Address: 192.0.2.30
Gateway: none
```

### NAT Adapter

Used for:

* Linux package updates
* Container downloads
* Repository access
* Threat intelligence updates
* Time synchronization

Example:

```
Interface: NAT
Address: assigned by VMware DHCP
Gateway: assigned by VMware
```

---

## Splunk Server Network Design

The Splunk server should use the host-only network for:

* Forwarder communication
* Log ingestion
* Web administration
* Search access
* Dashboard access

A NAT adapter may be used temporarily for:

* Package installation
* License retrieval
* Updates
* Add-on downloads

The Splunk web interface should not be accessible from the physical home network unless deliberately configured and protected.

---

## Kali Linux Network Design

Kali Linux should use the host-only network for security testing.

This allows it to interact with:

* Windows endpoint
* Domain controller
* Wazuh server
* Splunk server
* Other intentionally deployed lab services

A NAT adapter may be added for:

* Package updates
* Tool installation
* Repository access

The NAT adapter should be disconnected during exercises that do not require external connectivity.

---

## Multi-Homed Virtual Machines

A multi-homed VM has more than one active network adapter.

Examples include:

* Host-only plus NAT
* Two host-only networks
* NAT plus bridged
* Multiple segmented lab networks

Multi-homing is useful but introduces routing and DNS complexity.

---

## Common Multi-Homing Problems

Potential problems include:

* Multiple default gateways
* Incorrect interface priority
* DNS requests leaving through the wrong adapter
* Services binding to the wrong address
* Agent enrollment using the wrong interface
* Dashboard access failures
* Return traffic using a different route
* Hostname resolving to the NAT address
* External traffic entering an internal service

Only one interface should normally provide the default route.

---

## Linux Multi-Homing Example

A Linux SIEM server may use:

```
Host-only interface:
- Static address
- No default gateway
- Internal service traffic

NAT interface:
- DHCP address
- Default gateway
- Internet access
```

Review Linux addresses:

```
ip address
```

Review routes:

```
ip route
```

Expected design:

```
default via <NAT_GATEWAY> dev <NAT_INTERFACE>
192.0.2.0/24 dev <HOST_ONLY_INTERFACE>
```

---

## Windows Multi-Homing Example

A Windows system may use:

```
Host-only adapter:
- Static internal address
- Internal DNS
- No default gateway

NAT adapter:
- DHCP address
- Default gateway
- External connectivity
```

Review configuration:

```
ipconfig /all
Get-NetIPConfiguration
Get-NetRoute
```

Changes to interface metrics or routes normally require administrator privileges.

---

## Interface Metrics

Interface metrics determine route preference.

A lower metric generally indicates a more preferred route.

Review metrics:

```
Get-NetIPInterface |
    Sort-Object InterfaceMetric |
    Select-Object InterfaceAlias, AddressFamily, InterfaceMetric, ConnectionState
```

Linux route metrics can be reviewed with:

```
ip route
```

Incorrect metrics can cause internal traffic to use the NAT adapter.

---

## Default Gateway Design

A default gateway should be configured only on the interface that provides access beyond the local subnet.

For a dual-adapter VM:

```
Host-only adapter:
Default gateway: none

NAT adapter:
Default gateway: VMware NAT gateway
```

Multiple default gateways may create intermittent connectivity.

---

## DNS Design for Multi-Homed Systems

DNS should match the role of the system.

### Domain-Joined Windows System

Use the domain controller as primary DNS.

### Domain Controller

Use itself for internal DNS and configure forwarders for external queries.

### Linux Security Server

Use internal DNS when it must resolve the lab domain.

Use an approved external resolver only through controlled configuration.

### Kali Linux

Use internal DNS when interacting with domain names.

Use NAT-provided DNS for general package access when isolated from domain testing.

---

## DNS Forwarding Flow

```
Windows Endpoint
      |
      | DNS request
      v
Domain Controller DNS
      |
      | Internal zone lookup
      | or external forward
      v
Approved Upstream Resolver
```

This preserves Active Directory functionality while allowing external name resolution.

---

## VMware Virtual Network Editor

The Virtual Network Editor is used to review and modify VMware networks.

It can be used to:

* Review VMnet assignments
* Configure host-only networks
* Configure NAT networks
* Enable or disable DHCP
* Change subnets
* Review NAT settings
* Restore defaults

Administrator privileges are usually required to modify virtual network settings.

---

## Review the Existing Networks

Before making changes:

1. Open VMware Workstation.
2. Open the Virtual Network Editor.
3. Select **Change Settings** if elevation is required.
4. Record the existing VMnet configuration.
5. Identify the host-only network.
6. Identify the NAT network.
7. Record DHCP status.
8. Record subnet values privately.
9. Confirm which host adapters exist.
10. Export or screenshot the configuration for private records.

Do not publish unsanitized screenshots of the Virtual Network Editor.

---

## Example VMware Network Configuration

```
VMnet1
Type: Host-only
Host adapter: Enabled
DHCP: Optional
Purpose: Internal CyberLab

VMnet8
Type: NAT
Host adapter: Enabled
DHCP: Enabled
Purpose: Controlled outbound access
```

Exact subnet and gateway values are omitted from the public document.

---

## Creating a Host-Only Network

General workflow:

1. Open the Virtual Network Editor.
2. Select an unused VMnet.
3. Set the network type to **Host-only**.
4. Enable the host virtual adapter.
5. Choose a private subnet.
6. Decide whether VMware DHCP will be enabled.
7. Apply the configuration.
8. Verify that the host adapter appears in Windows.
9. Connect one test VM.
10. Validate host-to-guest communication.

Administrator privileges are typically required.

---

## Creating a NAT Network

General workflow:

1. Open the Virtual Network Editor.
2. Select an unused VMnet.
3. Set the network type to **NAT**.
4. Enable VMware DHCP.
5. Review the NAT gateway configuration.
6. Apply the settings.
7. Connect a test VM.
8. Confirm that it receives an address.
9. Confirm that it has a default route.
10. Test outbound internet access.

Do not configure inbound port forwarding unless there is a documented need.

---

## VMware NAT Port Forwarding

VMware NAT can forward a host port to a guest service.

Example uses may include:

* Temporary SSH access
* Temporary web application access
* Service testing

Port forwarding increases exposure and should not be enabled by default.

Before creating a port forward:

1. Confirm the service is required.
2. Restrict the listening address where possible.
3. Use a non-sensitive test service.
4. Review the host firewall.
5. Document the mapping.
6. Remove the rule after testing.
7. Validate that the port is closed afterward.

Public documentation should not reveal active port-forwarding rules.

---

## VM Adapter Configuration

Each VM adapter should be labeled and documented.

Example:

```
Network Adapter 1
Type: Host-only
Purpose: Internal CyberLab

Network Adapter 2
Type: NAT
Purpose: Updates and downloads
```

Consistent adapter order reduces troubleshooting confusion.

---

## Connect a VM to Host-Only Networking

1. Shut down the VM if required.
2. Open **Virtual Machine Settings**.
3. Select **Network Adapter**.
4. Choose **Custom**.
5. Select the designated host-only VMnet.
6. Confirm **Connected**.
7. Confirm **Connect at power on**.
8. Start the VM.
9. Review the guest IP configuration.
10. Test communication with another lab system.

---

## Connect a VM to NAT Networking

1. Open **Virtual Machine Settings**.
2. Add or select a network adapter.
3. Choose **NAT** or the designated NAT VMnet.
4. Confirm **Connected**.
5. Start the VM.
6. Confirm that the guest receives an address.
7. Confirm that a default gateway exists.
8. Test DNS resolution.
9. Test outbound connectivity.
10. Disconnect the adapter when no longer required.

---

## Disconnecting an Adapter

A virtual adapter can be disconnected without removing it.

This is useful when:

* An update has completed
* An exercise should remain isolated
* Troubleshooting route conflicts
* Preventing outbound traffic
* Reducing attack surface

In VMware:

1. Open VM settings.
2. Select the adapter.
3. Clear **Connected** or **Connect at power on**.
4. Apply the change.
5. Validate the remaining interface configuration.

---

## Windows Static IP Configuration

Static addressing may be configured through the Windows interface or .

Example  workflow:

```
New-NetIPAddress `
    -InterfaceAlias "<HOST_ONLY_ADAPTER>" `
    -IPAddress "192.0.2.20" `
    -PrefixLength 24
```

Configure DNS:

```
Set-DnsClientServerAddress `
    -InterfaceAlias "<HOST_ONLY_ADAPTER>" `
    -ServerAddresses "192.0.2.10"
```

Do not configure a default gateway on the isolated adapter unless routing is intentionally required.

Administrator privileges are required.

---

## Review Windows IP Settings

```
ipconfig /all
```

```
Get-NetIPConfiguration
```

```
Get-DnsClientServerAddress
```

```
Get-NetRoute
```

Read-only review usually does not require administrator privileges.

---

## Linux Static IP Configuration

Linux network configuration varies by distribution.

Common tools include:

* Netplan
* NetworkManager
* systemd-networkd
* `/etc/network/interfaces`

Example Netplan structure:

```yaml
network:
  version: 2
  ethernets:
    <HOST_ONLY_INTERFACE>:
      addresses:
        - 192.0.2.30/24
      nameservers:
        addresses:
          - 192.0.2.10
    <NAT_INTERFACE>:
      dhcp4: true
```

The exact interface names vary.

Root privileges are required to change Linux network configuration.

---

## Apply Netplan

```
sudo netplan try
```

Use `netplan try` when possible because it provides a rollback window.

After validation:

```
sudo netplan apply
```

Verify:

```
ip address
ip route
resolvectl status
```

---

## Linux NetworkManager Example

List connections:

```
nmcli connection show
```

Review devices:

```
nmcli device status
```

Configure a static internal connection:

```
sudo nmcli connection modify "<CONNECTION_NAME>" \
    ipv4.method manual \
    ipv4.addresses "192.0.2.30/24" \
    ipv4.gateway "" \
    ipv4.dns "192.0.2.10"
```

Restart the connection:

```
sudo nmcli connection down "<CONNECTION_NAME>"
sudo nmcli connection up "<CONNECTION_NAME>"
```

Root privileges are required for configuration changes.

---

## Connectivity Validation

Validation should progress from the local interface outward.

### Step 1: Verify the Interface

Windows:

```
Get-NetAdapter
```

Linux:

```
ip link
```

### Step 2: Verify the Address

Windows:

```
ipconfig /all
```

Linux:

```
ip address
```

### Step 3: Verify the Route

Windows:

```
Get-NetRoute
```

Linux:

```
ip route
```

### Step 4: Test Local Subnet Communication

```
ping <LAB_SYSTEM>
```

```
ping -c 4 <LAB_SYSTEM>
```

### Step 5: Test DNS

```
nslookup <LAB_DOMAIN>
```

```
dig <LAB_DOMAIN>
```

### Step 6: Test a Service Port

```
Test-NetConnection <LAB_SYSTEM> -Port <PORT>
```

```
nc -vz <LAB_SYSTEM> <PORT>
```

---

## Service Validation Examples

Examples include:

```
Test-NetConnection <DOMAIN_CONTROLLER> -Port 53
Test-NetConnection <WAZUH_SERVER> -Port <MANAGEMENT_PORT>
Test-NetConnection <SPLUNK_SERVER> -Port 8000
```

Linux:

```
curl -I http://<SPLUNK_SERVER>:8000
```

```
ss -tulpn
```

Do not publish exact active management ports when doing so would expose sensitive implementation details beyond common defaults.

---

## ICMP Considerations

A failed `ping` does not always mean a system is unreachable.

Possible causes include:

* ICMP blocked by firewall
* Network profile set incorrectly
* Guest firewall rule missing
* Service available even though ping fails

Always test the required application port in addition to ICMP.

---

## Windows Network Profiles

Windows classifies networks as:

* Domain
* Private
* Public

The profile affects firewall behavior.

Review profiles:

```
Get-NetConnectionProfile
```

A host-only adapter may be classified as Public until domain detection succeeds.

This may block expected traffic.

Do not disable the firewall broadly.

Create or adjust specific rules after confirming the correct profile.

---

## VMware DHCP Troubleshooting

Symptoms:

* Guest has an address beginning with `169.254`
* No default gateway
* NAT internet access fails
* New VMs receive no address

Possible causes:

* VMware DHCP service stopped
* Adapter connected to the wrong VMnet
* DHCP disabled
* Guest interface disabled
* Static configuration still present

Review host services:

```
Get-Service | Where-Object DisplayName -Match "VMware"
```

Restarting VMware services may require administrator privileges.

---

## VMware NAT Troubleshooting

Symptoms:

* Guest has a NAT address but no internet
* DNS fails
* Default gateway is present but unreachable
* Internet fails only when a VPN is active

Checks:

1. Confirm the guest NAT adapter is connected.
2. Confirm the guest has a default route.
3. Confirm VMware NAT services are running.
4. Review host internet access.
5. Review host VPN status.
6. Review host firewall.
7. Test an IP address and a hostname separately.
8. Renew the guest DHCP lease.
9. Restart VMware networking services if required.

Windows guest:

```
ipconfig /release
ipconfig /renew
ipconfig /flushdns
```

Linux guest:

```
sudo dhclient -r
sudo dhclient
```

---

## DNS Troubleshooting

Symptoms:

* IP connectivity works but hostnames fail
* Domain join fails
* Internal names do not resolve
* External sites fail by name only

Checks:

```
nslookup <LAB_DOMAIN>
nslookup <HOSTNAME>
Get-DnsClientServerAddress
```

Linux:

```
resolvectl status
dig <LAB_DOMAIN>
```

Confirm the endpoint is querying the domain controller rather than the NAT DNS server.

---

## Duplicate IP Addresses

Duplicate addresses can cause:

* Intermittent connectivity
* Incorrect ARP mappings
* Failed authentication
* Agent disconnects
* Dashboard access to the wrong system

Windows:

```
arp -a
```

Linux:

```
ip neigh
```

Avoid assigning static addresses inside an active VMware DHCP pool unless those addresses are excluded or reserved.

---

## ARP Validation

ARP maps IP addresses to MAC addresses on the local network.

Review Windows ARP cache:

```
arp -a
```

Review Linux neighbor table:

```
ip neigh
```

ARP entries should be treated as potentially sensitive in public screenshots because they reveal internal addresses and virtual MAC addresses.

---

## Firewall Troubleshooting

Before changing firewall settings:

1. Confirm basic addressing.
2. Confirm routing.
3. Confirm DNS.
4. Confirm the target service is listening.
5. Test the port.
6. Review the relevant firewall profile.
7. Add a narrow rule if required.

Avoid disabling the firewall as a permanent fix.

---

## Service Binding Problems

A service may be running but listening only on:

* `127.0.0.1`
* The NAT address
* The wrong host-only address
* IPv6 only
* A specific interface

Linux:

```
ss -tulpn
```

Windows:

```
Get-NetTCPConnection -State Listen
```

Confirm the service listens on the interface reachable by the intended lab systems.

---

## Packet Capture

Packet capture can help validate traffic flow.

Potential tools include:

* Wireshark
* tcpdump
* pktmon
* tshark

Linux example:

```
sudo tcpdump -i <HOST_ONLY_INTERFACE> host <LAB_SYSTEM>
```

Windows built-in example:

```
pktmon start --capture
```

Packet captures may contain credentials, hostnames, addresses, and session data.

They should be handled as sensitive evidence.

---

## Wireshark Capture Guidance

Useful capture points include:

* Windows endpoint host-only adapter
* Kali host-only adapter
* Wazuh internal adapter
* Acer VMware host adapter

Suggested display filters:

```
arp
dns
icmp
tcp
kerberos
ldap
```

Capture only the traffic required for the exercise.

---

## Network Validation Sequence

A reliable validation sequence is:

1. Confirm the VM adapter is connected.
2. Confirm the correct VMnet is selected.
3. Confirm the guest interface is enabled.
4. Confirm the address and subnet.
5. Confirm the route table.
6. Confirm the DNS server.
7. Ping a same-subnet system.
8. Test the required service port.
9. Confirm domain resolution.
10. Confirm application-level communication.
11. Confirm SIEM telemetry.
12. Record the results.

---

## Startup Validation

After starting the CyberLab:

1. Start the domain controller.
2. Confirm internal DNS.
3. Start the Windows endpoint.
4. Confirm domain connectivity.
5. Start Wazuh.
6. Confirm agent connectivity.
7. Start Splunk.
8. Confirm web access.
9. Start Kali only when needed.
10. Confirm no bridged adapters are active unintentionally.

---

## Shutdown Sequence

Recommended order:

1. End active security tests.
2. Save investigation evidence.
3. Stop packet captures.
4. Shut down Kali.
5. Shut down the Windows endpoint.
6. Shut down Splunk.
7. Shut down Wazuh.
8. Shut down the domain controller last.
9. Confirm all VMs are powered off.
10. Create snapshots only after stable shutdown when appropriate.

---

## Network Change Management

Changes should be documented before or immediately after implementation.

Examples include:

* Subnet changes
* DHCP changes
* New adapters
* Route changes
* DNS changes
* NAT port forwarding
* Firewall changes
* VMnet changes
* Bridged networking
* New trust zones

Change record example:

```
Date:
Network change:
Reason:
Systems affected:
Previous configuration:
New configuration:
Validation:
Rollback plan:
Outcome:
```

---

## Snapshot Before Network Changes

Create a snapshot before major network changes involving:

* Domain controller addressing
* DNS configuration
* Wazuh interface changes
* Splunk interface changes
* VMware subnet changes
* DHCP scope changes
* Route changes
* Multi-homing

A network change can affect several dependent systems simultaneously.

---

## Public Sanitization Standards

Remove or replace:

* Real VMnet subnets
* Host-only addresses
* NAT addresses
* Default gateways
* MAC addresses
* DHCP pool values
* DNS server addresses
* Internal domain names
* Home network addresses
* Public IP addresses
* Router names
* Wireless network names
* VPN details
* Port-forwarding rules
* Screenshots containing unrelated adapters

Use placeholders such as:

```
<HOST_ONLY_SUBNET>
<NAT_SUBNET>
<DOMAIN_CONTROLLER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<LAB_DOMAIN>
<VMWARE_GATEWAY>
<REDACTED_ADDRESS>
```

---

## Screenshot Sanitization

Before publishing VMware network screenshots:

1. Crop unrelated adapters.
2. Hide the Windows username.
3. Remove physical network names.
4. Remove public and private addresses.
5. Remove MAC addresses.
6. Remove VPN information.
7. Remove gateway values.
8. Remove active port-forwarding rules.
9. Remove personal file paths.
10. Verify redactions cannot be reversed.

Use permanent image editing rather than semi-transparent overlays.

---

## Security Boundaries

The host-only network forms the main lab isolation boundary.

The NAT network forms the controlled internet boundary.

The Acer host sits between:

* The physical network
* The VMware virtual networks
* Administrative access
* VM storage
* Network translation

A compromise of the host could affect all virtual lab systems.

The host should therefore be secured and maintained as critical infrastructure.

---

## Architectural Limitations

VMware host-only networking does not provide the same isolation guarantees as dedicated physical network segmentation.

Limitations include:

* The host can access guest systems
* Guests may access shared host services if permitted
* Misconfiguration can expose services
* Multi-homing can bypass intended paths
* Clipboard and shared folders cross boundaries
* The host remains a single point of failure
* NAT traffic relies on the host network

These limitations are acceptable for a controlled personal CyberLab but should be documented.

---

## Planned Network Improvements

Future improvements may include:

* Separate attacker and defender networks
* Dedicated management network
* Dedicated monitoring network
* Internal router or firewall VM
* Additional domain subnet
* Centralized DHCP
* Dedicated DNS forwarder
* Packet monitoring sensor
* Suricata network IDS
* Zeek network telemetry
* Virtual jump host
* Automated network validation
* Network configuration backup
* VLAN-backed physical lab integration
* Dedicated internet-restricted malware analysis segment

---

## Skills Demonstrated

This network design demonstrates:

* VMware Workstation networking
* Host-only network design
* NAT network design
* Static IP planning
* DHCP considerations
* DNS design
* Active Directory dependencies
* Multi-homed server configuration
* Routing analysis
* Firewall troubleshooting
* Packet capture
* Network isolation
* Change management
* Security sanitization
* Defensive lab architecture

---

## Lessons Learned

Key lessons include:

* Active Directory depends heavily on correct DNS.
* A second adapter can create unexpected routing behavior.
* Only one interface should normally provide the default route.
* NAT access should be treated as temporary when practical.
* Bridged networking is unnecessary for most exercises.
* A successful ping does not prove the required service works.
* A failed ping does not prove the service is unavailable.
* Static infrastructure addressing improves reliability.
* VMware DHCP pools must be considered before assigning static addresses.
* Service binding must be checked in addition to firewall rules.
* VPN software on the host can affect VMware NAT and DNS.
* Network changes should be validated one layer at a time.
* Public network documentation requires aggressive sanitization.

---

## Validation Checklist

### VMware Configuration

* Host-only VMnet exists.
* NAT VMnet exists.
* VMware host adapters are present.
* DHCP status is documented.
* NAT services are running.
* No unexpected bridged adapters are enabled.

### Domain Controller

* Static internal address is correct.
* Preferred DNS points to itself.
* Host-only adapter is connected.
* No unintended default gateway exists.
* DNS service responds.

### Windows Endpoint

* Host-only address is correct.
* DNS points to the domain controller.
* Domain controller is reachable.
* Domain resolution works.
* Required monitoring services are reachable.

### Wazuh

* Internal adapter is reachable.
* NAT adapter provides the default route.
* Only the NAT adapter has a gateway.
* Agents can connect.
* Dashboard access works.

### Splunk

* Internal address is reachable.
* Web interface listens on the intended interface.
* Forwarders or data sources can connect.
* NAT is enabled only when required.

### Kali

* Host-only adapter is connected during tests.
* NAT is disconnected when not required.
* Authorized targets are documented.
* No physical-network scanning occurs.

---

## Summary

The Acer Blue Team CyberLab uses VMware host-only networking for internal isolation and VMware NAT for controlled internet access.

The design supports:

* Active Directory
* Windows endpoint monitoring
* Wazuh telemetry
* Splunk log analysis
* Kali-based controlled testing
* Host-based administration
* Repeatable network troubleshooting

By avoiding bridged networking, limiting default gateways, using stable infrastructure addresses, and documenting traffic flows, the environment remains safer, more predictable, and easier to troubleshoot.

