# Lessons Learned

## Overview

Building the Acer Blue Team CyberLab required more than installing security tools.

The project involved:

* Virtualization
* Network design
* Windows administration
* Linux administration
* Active Directory
* DNS
* Endpoint monitoring
* SIEM deployment
* Log ingestion
* Detection validation
* Snapshot management
* Recovery planning
* Troubleshooting
* Public documentation
* Security sanitization

The most important result was not any single installed platform.

The project demonstrated how infrastructure, identity, logging, monitoring, testing, recovery, and documentation depend on one another.

This document summarizes the major technical and operational lessons learned while designing, building, validating, and documenting the CyberLab.

All public examples remain sanitized.

---

## The Lab Is a System, Not a Collection of Tools

Wazuh, Splunk, Kali, Active Directory, VMware, Windows, and Linux do not operate independently.

A failure in one area can affect several others.

Examples include:

* DNS failure preventing domain authentication
* Time drift breaking Kerberos and event correlation
* Network adapter changes interrupting agent communication
* Low disk space stopping SIEM ingestion
* Snapshot restoration breaking endpoint trust
* Firewall changes blocking dashboards or telemetry
* Incorrect service ownership preventing Splunk startup

The environment must be understood as one connected system.

---

## Architecture Should Be Designed Before Deployment

It is easier to deploy systems when their roles and dependencies are defined first.

Important design questions included:

* Which VM provides identity?
* Which VM provides DNS?
* Which network carries internal traffic?
* Which systems require temporary internet access?
* Which systems need static addresses?
* Which systems depend on one another?
* Which services must start first?
* Which data should remain isolated?
* Which values can appear in public documentation?

The architecture diagram became a troubleshooting and recovery reference, not just a presentation image.

---

## Dependency Mapping Matters

The lab has a clear dependency chain.

```
VMware Networking
        |
        v
DC01 and DNS
        |
        v
WIN11TARGET Domain Connectivity
        |
        v
Wazuh and Splunk Collection
        |
        v
Kali Exercise Activity
        |
        v
Detection and Investigation
```

Starting systems out of order can create misleading failures.

For example:

* WIN11TARGET may show domain errors if DC01 is not ready.
* Wazuh agents may appear disconnected if the manager is still starting.
* Splunk searches may appear empty before the forwarder reconnects.
* Kali testing may produce no useful results if monitoring is offline.

---

## Start with a Stable Foundation

Advanced security testing is only useful when the basic infrastructure is reliable.

The strongest early milestones were:

* Working VMware networking
* Stable host-only communication
* Correct DNS
* Healthy domain controller
* Domain-joined Windows endpoint
* Accurate system time
* Clean snapshots
* Known local recovery accounts

Without these foundations, later troubleshooting becomes difficult because every result is uncertain.

---

# Virtualization Lessons

## VMware Network Type Must Match the Objective

Each VMware network type has a different purpose.

### Host-Only

Best for:

* Isolated internal lab communication
* Domain services
* Endpoint monitoring
* SIEM traffic
* Controlled testing

### NAT

Best for:

* Operating system updates
* Package downloads
* Tool installation
* External documentation access

### Bridged

Carries greater risk because it connects the VM directly to the physical network.

The safest standard configuration was:

```
Host-only for lab traffic
NAT only when internet access is required
No bridged networking by default
```

---

## Dual Adapters Increase Complexity

Using host-only and NAT adapters together is useful, but it creates several potential problems:

* Multiple default routes
* Wrong DNS server
* Wrong source interface
* DNS registration of the NAT address
* Service binding to all interfaces
* Agent traffic using the wrong path
* Confusing troubleshooting results

A clear rule helped:

* Host-only adapter: static address, internal DNS, no gateway
* NAT adapter: DHCP, outbound gateway, temporary use

---

## The Default Route Should Be Intentional

The internal host-only adapter should not normally provide a default route.

The NAT adapter should provide the external route only when needed.

Incorrect routes can cause:

* Internal traffic leaving through NAT
* Intermittent application access
* Broken return traffic
* DNS problems
* Agent disconnections

Route verification became a standard startup and troubleshooting step.

---

## Virtual Machines Need Resource Planning

Running several VMs simultaneously can create host pressure.

The main consumers included:

* Wazuh indexer
* Splunk indexing and searches
* Windows updates
* Defender scans
* Active Directory
* Kali tools
* VMware snapshots

Assigning more virtual CPUs or memory is not always the solution.

Overallocating resources can reduce host stability.

The better approach is to:

* Start only required VMs
* Monitor actual usage
* Increase resources deliberately
* Avoid unnecessary background services
* Maintain sufficient host storage

---

## Snapshots Are Powerful but Easy to Misuse

Snapshots were invaluable before:

* Domain joins
* Agent installation
* SIEM changes
* Group Policy changes
* Detection exercises
* Updates
* Troubleshooting

However, snapshots introduced risks:

* Long snapshot chains
* Increased storage use
* Time rollback
* Broken secure channels
* Duplicate SIEM events
* Agent identity issues
* Restored credentials
* Lost evidence

Snapshots work best as temporary or milestone rollback points, not permanent backups.

---

# Networking Lessons

## IP Connectivity and Service Connectivity Are Different

A successful ping does not prove an application works.

A failed ping does not prove an application is unavailable.

More useful tests included:

```
Test-NetConnection <SERVER> -Port <PORT>
```

and:

```
nc -vz <SERVER> <PORT>
```

Service-specific testing provides more accurate evidence than relying only on ICMP.

---

## Troubleshoot from the Lowest Layer Upward

A reliable troubleshooting order was:

```
VM state
Network adapter
IP address
Subnet
Route
DNS
Firewall
Service listener
Application
Agent or forwarder
Dashboard or search
```

This prevented unnecessary reinstalls and configuration changes.

---

## Multiple Interfaces Require Source Awareness

When a VM has more than one adapter, it is important to confirm:

* Which address is used
* Which interface provides the route
* Which DNS server is selected
* Which interface the service binds to
* Which source address appears in logs

A connection may succeed through the wrong interface and still create future problems.

---

## Internal Addresses Should Be Stable

Infrastructure systems benefit from predictable addresses.

These include:

* Domain controller
* Wazuh server
* Splunk server
* Monitoring endpoints
* Internal services

Stable addresses simplify:

* DNS
* Agent configuration
* Forwarder configuration
* Firewall rules
* Documentation
* Troubleshooting
* Recovery

---

# DNS and Active Directory Lessons

## DNS Is the Foundation of Active Directory

Many apparent domain problems were actually DNS problems.

Symptoms included:

* Domain join failure
* Group Policy failure
* Trust errors
* Incorrect network profile
* Failed domain discovery
* Kerberos errors

The endpoint must use the domain controller for internal DNS.

A public resolver should not replace Active Directory DNS on a domain-joined system.

---

## Active Directory Uses Service Records

Resolving the domain name alone is not enough.

Active Directory relies on SRV records such as those used for:

* LDAP
* Kerberos
* Domain controller discovery

Validation should include service-record queries, not only hostname lookups.

---

## Temporary NAT Adapters Can Pollute DNS

A temporary external adapter may register an unintended address in DNS.

This can cause clients to resolve DC01 or another server to the wrong interface.

Preventive controls include:

* Disable DNS registration on the temporary adapter
* Use the host-only adapter for internal services
* Review DNS records after network changes
* Remove stale records
* Revalidate domain discovery

---

## Time Is an Identity Dependency

Kerberos depends on reasonably synchronized time.

Time problems can also affect:

* Certificate validation
* Event timelines
* Agent status
* SIEM searches
* Authentication investigations

Time should be checked before troubleshooting complex identity failures.

---

## Local Recovery Accounts Are Essential

A domain-joined endpoint should retain a tested local administrator account.

This account becomes critical when:

* DC01 is offline
* DNS is incorrect
* The secure channel fails
* Group Policy causes an issue
* Domain credentials do not work
* The endpoint must leave and rejoin the domain

The local recovery account must be tested before it is needed.

---

## Group Policy Requires Correct OU Placement

A GPO may be configured correctly but still fail to apply if the computer or user is in the wrong location.

Validation should include:

* Object OU
* GPO link
* Security filtering
* User versus computer scope
* Group Policy results
* Operational logs

---

## Do Not Force DomainAuthenticated as a Workaround

Windows determines the DomainAuthenticated network profile based on successful domain detection.

If the profile remains Public, the correct response is to repair:

* DNS
* Secure channel
* Network Location Awareness
* DC availability
* Time

Forcing the profile hides the real problem.

---

# Windows Endpoint Lessons

## Windows Edition Matters

Traditional Active Directory domain joining requires a supported Windows edition.

This should be confirmed before spending time troubleshooting domain membership.

---

## Standard Users Produce Better Test Results

Most detection exercises should use standard users.

This improves realism and reduces risk.

Administrative accounts should be reserved for:

* Configuration
* Software installation
* Policy changes
* Recovery
* Approved privilege-change exercises

---

## Audit Policy Must Be Verified

It is not enough to assume that Windows logs a specific action.

Validation should confirm:

* The audit category is enabled
* The correct channel exists
* The event appears locally
* The SIEM collects the channel
* The expected fields are present

A missing SIEM event may begin with missing source telemetry.

---

## PowerShell Visibility Depends on Configuration

Useful PowerShell visibility may require:

* Process creation auditing
* Command-line inclusion
* PowerShell Operational logging
* Script block logging
* Module logging
* Sysmon in later phases

Installing a SIEM agent does not automatically create complete PowerShell telemetry.

---

## Microsoft Defender Should Remain Enabled

Disabling endpoint protection to simplify testing weakens the lab and reduces useful telemetry.

Safer validation methods include:

* Official harmless test artifacts
* Benign scripts
* Controlled file changes
* Synthetic logs
* Detection simulations

---

# Linux Administration Lessons

## Use Normal Accounts and Elevate Only When Needed

Routine administration is safer with a normal user and `sudo`.

Persistent root sessions increase the chance of:

* Wrong-path changes
* Permission mistakes
* Destructive commands
* Unclear audit history

Explicit privilege use also improves documentation.

---

## File Ownership Can Break Applications

The Splunk deployment showed how application behavior can depend on ownership and runtime account consistency.

Alternating between root and a service account can create:

* Root-owned logs
* Inaccessible index files
* Lock-file failures
* Startup errors
* Configuration permission issues

Ownership should be reviewed before applying recursive changes.

---

## Service Status Is Not Application Health

A process may be running while the application is unhealthy.

Examples include:

* Docker container running but indexer unavailable
* Splunk process active but Web not listening
* Agent service running but manager connection broken
* DNS service running with incorrect records

Validation must include application-level checks.

---

## Logs Should Be Reviewed Before Restarting Repeatedly

Repeated restarts can:

* Erase useful timing information
* Create additional errors
* Hide the original condition
* Increase event noise
* Cause file or index problems

The better process is:

1. Record the symptom.
2. Review the logs.
3. Identify the likely layer.
4. Apply one repair.
5. Validate.

---

# Wazuh Lessons

## Wazuh Is a Multi-Component Platform

The all-in-one Wazuh deployment still includes multiple logical components:

* Manager
* Indexer
* Dashboard
* Agent communication
* Certificates
* Container networking
* Storage

A dashboard problem may originate from the indexer rather than the dashboard itself.

---

## The Indexer Requires the Most Attention

The indexer is resource-intensive and affects:

* Alert storage
* Dashboard searches
* Login experience
* Data availability
* Overall platform health

Memory and disk checks should be part of Wazuh troubleshooting.

---

## Agent Active Does Not Mean Every Event Is Collected

An active agent confirms communication, not complete telemetry.

Collection still depends on:

* Event channel configuration
* File paths
* Decoders
* Rules
* Agent group configuration
* Dashboard filters

---

## Event Collection and Alerting Are Different

Wazuh may collect an event without creating a visible alert.

This can occur when:

* No rule matches
* Rule level is low
* Dashboard filters hide the event
* Decoder coverage is incomplete

A missing alert may be a detection-development opportunity.

---

## File Integrity Monitoring Should Start Narrow

Monitoring entire drives or noisy directories creates excessive events.

A dedicated test path made validation easier:

```
C:\CyberLab-Test\Files
```

The monitoring scope should expand only after baseline behavior is understood.

---

## Rule Severity Requires Context

A high rule level is not automatic proof of malicious activity.

An analyst must consider:

* User
* System role
* Frequency
* Source
* Expected exercise
* Related events
* Business or lab context

---

# Splunk Lessons

## Installation Success Does Not Mean Splunk Is Running

The `.deb` package installation only placed the software on the system.

Additional validation was required:

* Start Splunk
* Accept the license
* Create credentials
* Confirm `splunkd`
* Confirm port `8000`
* Open Splunk Web
* Run a search

Each step validated a different layer.

---

## The Full CLI Path Is More Reliable

Using:

```
sudo /opt/splunk/bin/splunk status
```

avoided ambiguity when the Splunk binary was not included in the shell path.

Explicit commands are also clearer for documentation.

---

## Root Execution Solved the Immediate Problem but Added Risk

The working lab startup command was:

```
sudo /opt/splunk/bin/splunk start --run-as-root
```

This restored functionality, but it also created a future hardening task.

A working configuration is not always the final secure configuration.

The long-term objective is to:

* Identify the ownership issue
* Create or validate a dedicated service account
* Correct permissions carefully
* Test all inputs
* Enable safe boot startup

---

## Web Availability Does Not Prove Data Ingestion

A functioning Splunk Web page confirms:

* The web service is reachable
* The application is running sufficiently to display the interface

It does not confirm:

* Receiver configuration
* Forwarder connectivity
* Correct indexes
* Event parsing
* Timestamp accuracy
* Search permissions
* Detection logic

A known test event is required for end-to-end validation.

---

## Splunk Metadata Is Critical

The following fields determine how events are found and interpreted:

* Index
* Host
* Source
* Sourcetype
* Event time
* Index time

Poor metadata can make valid data appear missing.

---

## Narrow SPL Searches Are Better

Broad searches can be slow and confusing.

A stronger process is to begin with:

* Expected index
* Expected host
* Narrow time range
* Known event identifier
* Known source

Then expand only when troubleshooting.

---

## Event Time and Index Time Reveal Pipeline Health

Comparing `_time` with `_indextime` can identify:

* Forwarder backlog
* Time drift
* Parsing issues
* Network delays
* Service outages
* Replayed data

This became an important ingestion-validation method.

---

# Kali and Testing Lessons

## Kali Is a Validation System

The value of Kali in the Blue Team lab is not the number of tools installed.

Its value comes from generating controlled activity that helps answer:

* Was the action logged?
* Was it collected?
* Was it detected?
* Was it searchable?
* Could the analyst explain it?
* What telemetry was missing?

---

## Narrow Tests Produce Better Telemetry

A small test is easier to correlate than a broad scan.

Examples include:

* One failed sign-in
* One approved connection test
* A short port list
* One DNS query
* One file change
* One harmless PowerShell command

Broad activity produces noise and makes root-cause analysis harder.

---

## Authorization Must Be Technical, Not Assumed

Scope controls included:

* Host-only networking
* No bridged adapter
* Known target list
* Narrow commands
* Pre-exercise checklist
* Defined start and stop times
* Disposable accounts
* Stable snapshots

These controls reduce the chance of accidentally testing an unintended system.

---

## Not Every Scan Produces a Host Alert

A connection or port scan may be visible only through:

* Packet capture
* Firewall logs
* Sysmon
* Suricata
* Zeek
* Custom SIEM logic

The absence of a native endpoint alert is a valid finding.

It identifies a telemetry or detection gap.

---

# Log Ingestion Lessons

## Always Validate the Source Event First

The troubleshooting order should begin with:

1. Did the activity happen?
2. Did the source record it?
3. Is the agent or forwarder running?
4. Is the receiver reachable?
5. Was the event stored?
6. Was it parsed correctly?
7. Did the search use the correct filters?
8. Did a rule exist?

This prevents blaming the SIEM for an event the operating system never generated.

---

## Connectivity Does Not Prove Ingestion

A successful TCP connection proves only that a listener is reachable.

It does not confirm:

* Correct credentials
* Correct input
* Correct index
* Correct event channel
* Correct parsing
* Successful storage

End-to-end testing requires a known event.

---

## A Running Agent Can Still Be Misconfigured

Agent or forwarder health requires more than service status.

Validation should include:

* Correct destination
* Correct source interface
* Correct identity
* Recent keepalive
* Expected event collection
* Searchable test data

---

## Missing Events Should Be Traced by Stage

The pipeline should be checked in order:

```
Action
Source log
Agent or input
Network
Receiver
Parser
Storage
Search
Detection
```

This makes missing-event investigations repeatable.

---

## Data Quality Matters as Much as Event Count

Useful telemetry should be:

* Complete
* Accurate
* Timely
* Consistent
* Unique
* Searchable
* Contextual

A large volume of poorly parsed data does not create strong security visibility.

---

# Detection Engineering Lessons

## Detection Engineering Begins with Telemetry

A detection cannot be reliable when the required data source is missing or inconsistent.

Before writing a detection, confirm:

* The event exists
* The event is collected
* Important fields are available
* Timestamps are accurate
* Normal behavior is understood
* Test activity is repeatable

---

## Alerts Must Be Validated for the Correct Reason

An alert appearing during a test does not prove the intended logic worked.

The rule may have matched:

* A different field
* A different event
* A broad pattern
* Background activity
* An unrelated condition

Validation should inspect the raw event and rule logic.

---

## False-Positive Testing Is Required

A detection should be tested against legitimate behavior that resembles suspicious activity.

Examples include:

* User password mistakes
* Help-desk account changes
* Approved PowerShell administration
* Vulnerability scanning
* Software installation
* Configuration-management changes

Without false-positive testing, a detection is incomplete.

---

## Detection Gaps Are Valuable Findings

A test may produce:

* No alert
* Missing fields
* Delayed events
* Incorrect host identity
* Excessive noise
* Incomplete context

These results are not failed projects.

They are findings that guide improvement.

---

## Wazuh and Splunk Offer Different Strengths

Wazuh provides:

* Agent management
* Prebuilt security rules
* File integrity monitoring
* Configuration assessment
* MITRE and compliance metadata

Splunk provides:

* Flexible searching
* Data exploration
* Custom statistics
* Dashboards
* Correlation
* Detection development

Comparing both platforms improves understanding of SIEM design.

---

# Troubleshooting Lessons

## Define the Symptom Precisely

Specific symptoms lead to better tests.

Better:

```
Splunk Web does not respond on the internal interface.
```

Worse:

```
Splunk is broken.
```

---

## Preserve Evidence Before Restarting

Useful evidence may include:

* Error message
* Service status
* Event log
* Route table
* Listener state
* Application log
* Container log
* Screenshot
* Timestamp

Restarting may remove the original state.

---

## Change One Variable at a Time

Changing DNS, firewall, routing, and service configuration together makes the root cause impossible to identify.

One controlled change followed by validation creates reliable troubleshooting records.

---

## Do Not Reinstall Too Early

Reinstallation can:

* Destroy evidence
* Create new variables
* Hide the root cause
* Reset working configuration
* Consume time
* Reintroduce old errors

Repair and evidence collection should come first.

---

## Always End with a Known Test Event

A service restart is not complete validation.

After repair, generate a harmless event and confirm:

* Local source
* Agent or forwarder
* SIEM
* Search
* Timestamp
* Host identity

---

# Snapshot and Recovery Lessons

## Snapshots Are Not Backups

Snapshots remain dependent on the active VM files and host storage.

Independent protection requires:

* Powered-off VM copy
* VM export
* Configuration archive
* External storage
* Recovery documentation

---

## Recovery Must Be Application-Aware

A VM that boots is not necessarily recovered.

Validation should include:

* DC01 health
* Secure channel
* Wazuh containers
* Wazuh agents
* Splunk service
* Splunk indexes
* Forwarder connectivity
* New event ingestion

---

## Evidence Must Be Exported Before Rollback

Snapshot restoration can remove:

* Logs
* Test files
* Alerts
* Search results
* Account changes
* Packet captures
* Troubleshooting evidence

Important evidence should be stored outside the disposable VM state.

---

## Restore Order Matters

Identity and DNS should be restored before dependent systems.

A useful full recovery order is:

1. VMware networking
2. DC01
3. WIN11TARGET
4. Wazuh
5. Splunk
6. Agents and forwarders
7. Kali
8. End-to-end validation

---

## A Backup Must Be Tested

A copied file is not automatically a usable backup.

Verification should include:

* File readability
* Hash validation
* VM startup
* Network functionality
* Service health
* Known-event ingestion

---

# Documentation Lessons

## Documentation Is an Engineering Control

Documentation supported:

* Repeatability
* Troubleshooting
* Recovery
* Change management
* Security review
* Portfolio presentation
* Knowledge retention

It was not merely a final report.

---

## Private and Public Documents Serve Different Purposes

Private documentation needs operational precision.

Public documentation needs:

* Safe examples
* Clear methodology
* Demonstrated skills
* Sanitized values
* Minimal exposure

A public document should be rewritten rather than copied directly from private notes.

---

## Commands Should Identify Privilege Level

Documentation is clearer when it distinguishes:

* Normal user commands
* Administrator PowerShell
* `sudo` commands
* Root-level operations
* Domain-administrator tasks

This reduces mistakes and teaches access-control awareness.

---

## Procedures and Explanations Need Different Structures

Numbered lists work best for:

* Installation
* Recovery
* Validation
* Exercises
* Troubleshooting steps

Bullets and sections work better for:

* Architecture
* Concepts
* Risks
* Lessons
* Skills

---

## Consistent Names Improve the Repository

Consistent naming helps readers understand the lab.

Examples include:

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* KALI-TEST

Consistent placeholders also make sanitization easier.

---

## A Project Index Improves Navigation

A long repository benefits from:

* Main README
* Documentation sequence
* Exercise index
* Runbook index
* Image gallery
* Architecture diagram
* Cross-links between related documents

The reader should not need to guess which document comes next.

---

# Security and Sanitization Lessons

## Public Documentation Is an Exposure Surface

A repository may reveal more than expected through:

* Screenshots
* Command output
* Commit history
* File metadata
* Configuration files
* Image filenames
* Browser tabs
* Shell prompts
* Raw logs
* Packet captures

Security review must include every artifact, not just Markdown text.

---

## Internal IP Addresses Should Still Be Sanitized

Private addressing is not directly routable from the internet, but it can still reveal:

* Network structure
* Service location
* Addressing conventions
* Relationships between systems
* Management patterns

Documentation-safe address ranges provide the same educational value with less disclosure.

---

## Screenshots Are High-Risk Artifacts

Screenshots may expose:

* Usernames
* IP addresses
* Domain names
* Browser bookmarks
* Notifications
* File paths
* License information
* Session data
* Other VMs

Every screenshot requires a separate review.

---

## Redaction Must Be Permanent

Covering text with an editable shape is not secure.

A safer process is:

* Edit a copy
* Permanently remove or replace the content
* Flatten the image
* Reopen the final file
* Confirm the original value cannot be recovered

---

## `.gitignore` Is Preventive, Not Corrective

A `.gitignore` file prevents untracked files from being added.

It does not remove:

* Previously committed secrets
* Old log files
* Earlier screenshots
* Deleted credentials from Git history

An exposed secret must be rotated.

---

## Webhooks and Tokens Are Credentials

Notification URLs, API tokens, agent keys, and enrollment secrets should be treated like passwords.

If published, they must be revoked or rotated immediately.

---

## Metadata Can Reveal Hidden Information

Images, documents, and PDFs may contain:

* Author name
* Device name
* Organization
* File path
* GPS location
* Editing history
* Hidden text

Public review must include metadata where applicable.

---

# Operational Improvements Identified

## Improve Splunk Service Hardening

Current state:

* Splunk starts successfully using an elevated command.

Improvement:

* Create or validate a dedicated Splunk service account.
* Review ownership.
* Test all inputs.
* Configure safe boot startup.
* Confirm that root execution is no longer required.

---

## Add Sysmon

Sysmon would improve visibility into:

* Process creation
* Network connections
* DNS queries
* File creation
* Registry activity
* Image loading

Deployment should use a deliberate configuration to avoid excessive noise.

---

## Add Windows Event Forwarding

Windows Event Forwarding could centralize:

* Domain controller events
* Endpoint events
* PowerShell telemetry
* Defender logs
* Group Policy logs

This would introduce an additional enterprise-relevant logging architecture.

---

## Add Network Telemetry

Network reconnaissance testing showed the value of:

* Windows Firewall logs
* Sysmon network events
* Suricata
* Zeek
* Packet capture
* Network flow data

Endpoint SIEM agents alone do not provide complete network visibility.

---

## Automate Health Validation

Future scripts could check:

* VM reachability
* DNS resolution
* DC01 services
* Domain secure channel
* Wazuh containers
* Wazuh agent status
* Splunk service
* Receiver port
* Forwarder status
* Disk space
* Time synchronization

Automation would reduce startup validation time and configuration drift.

---

## Expand Runbooks

Focused runbooks should cover:

* CyberLab startup
* CyberLab shutdown
* VM unreachable
* DNS failure
* Domain join failure
* Secure-channel repair
* Wazuh dashboard down
* Wazuh agent disconnected
* Splunk Web down
* Splunk forwarder failure
* Missing event investigation
* Snapshot restore
* Low disk space

---

## Create a Detection Coverage Matrix

A coverage matrix should track:

* Exercise
* Data source
* Local event
* Wazuh visibility
* Splunk visibility
* Packet visibility
* Detection status
* Tuning status
* Evidence link

This would show progress more clearly than a list of installed tools.

---

## Add Configuration Version Control

Sanitized configuration templates can be stored for:

* Wazuh rules
* Wazuh decoders
* Splunk inputs
* Splunk outputs
* Splunk indexes
* Sysmon
* Firewall logging
* Validation scripts

Secrets and operational values must remain outside the public repository.

---

## Improve Backup Automation

Future improvements include:

* Scheduled configuration archives
* External VM backups
* Backup hashes
* Snapshot inventory
* Restore testing
* NAS storage
* Offline backup rotation
* Disk-space alerts

---

# Skills Developed

The project strengthened practical skills in:

* VMware Workstation
* Virtual networking
* Windows Server
* Windows 11 administration
* Active Directory
* DNS
* Group Policy
* Linux administration
* Docker
* Wazuh
* Splunk
* Kali Linux
* PowerShell
* Bash
* Windows event logs
* Log ingestion
* SIEM analysis
* Detection engineering
* Packet capture
* Network troubleshooting
* Identity troubleshooting
* Backup and recovery
* Change management
* Technical documentation
* Public sanitization

---

## Professional Skills Developed

The project also reinforced:

* Systems thinking
* Root-cause analysis
* Change planning
* Risk assessment
* Evidence preservation
* Procedure development
* Technical writing
* Documentation architecture
* Defensive testing
* Security communication
* Continuous improvement

---

# Portfolio Value

The CyberLab demonstrates more than tool familiarity.

It shows the ability to:

* Design an environment
* Define security boundaries
* Build infrastructure
* Integrate systems
* Troubleshoot dependencies
* Validate telemetry
* Develop exercises
* Preserve evidence
* Plan recovery
* Document decisions
* Sanitize public materials

These are relevant to roles involving:

* Security engineering
* Detection engineering
* Network security
* Systems administration
* Vulnerability management
* Technical GRC
* IAM
* Security operations engineering
* Blue Team infrastructure

---

# Final Project Assessment

## What Worked Well

* Host-only network isolation
* Clear VM roles
* Active Directory and DNS foundation
* Windows endpoint integration
* Wazuh all-in-one deployment
* Splunk Web deployment
* Controlled Kali testing
* Snapshot milestones
* Public/private documentation separation
* Sanitized GitHub structure
* Detection exercise planning

---

## What Required the Most Troubleshooting

* VMware network behavior
* DNS and multi-adapter interactions
* Domain connectivity dependencies
* Wazuh component health
* Splunk startup permissions
* Browser access versus service status
* Log ingestion validation
* Snapshot-related identity concerns
* Public sanitization

---

## What Should Be Improved Next

Priority improvements include:

1. Validate Splunk Universal Forwarder ingestion.
2. Migrate Splunk away from root execution.
3. Deploy Sysmon.
4. Complete the first detection exercises.
5. Build a detection coverage matrix.
6. Create short operational runbooks.
7. Automate health checks.
8. Implement external VM backups.
9. Test a full recovery workflow.
10. Add network telemetry.

---

# Lessons Summary

The most important lessons from the CyberLab are:

* Build stable infrastructure before advanced testing.
* Treat the lab as one connected system.
* DNS and time are core security dependencies.
* Use host-only networking as the default safety boundary.
* Validate services at the application layer.
* Confirm source logs before blaming the SIEM.
* A running agent does not guarantee complete telemetry.
* A visible alert does not prove correct detection logic.
* Missing detections are useful engineering findings.
* Use narrow, authorized exercises.
* Preserve evidence before restarting or restoring.
* Snapshots are not backups.
* Recovery must be tested.
* Documentation is part of the engineering process.
* Public documentation requires a separate security review.

---

## Conclusion

The Acer Blue Team CyberLab evolved from a group of virtual machines into a structured defensive security environment.

Its value comes from the integration of:

* Infrastructure
* Identity
* Monitoring
* Testing
* Detection
* Investigation
* Recovery
* Documentation
* Security review

The project created a repeatable platform for continued learning while demonstrating the practical engineering discipline required to build, operate, troubleshoot, secure, and document a cybersecurity lab.

