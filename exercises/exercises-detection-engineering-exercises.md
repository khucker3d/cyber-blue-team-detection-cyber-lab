# Detection Engineering

## Overview

This exercise library contains controlled Blue Team activities designed to validate telemetry, detections, alerting, investigation workflows, and recovery procedures inside the Acer CyberLab.

The exercises use the lab’s existing systems:

* Domain controller
* Windows endpoint
* Wazuh
* Splunk
* Kali Linux
* Windows Event Viewer
* PowerShell
* Packet-capture tools
* VMware snapshots

Each exercise is designed to answer four questions:

* Did the activity generate the expected telemetry?
* Did the monitoring platform collect and parse it correctly?
* Did an alert or useful search result appear?
* Could a student analyst investigate and explain what happened?

The exercises use harmless, authorized actions performed only against CyberLab systems.

---

## Folder Placement

This document should be stored as:

```
exercises/detection-engineering-exercises.md
```

It serves as the central index for the individual exercise files.

Recommended structure:

```
exercises/
├── detection-engineering-exercises.md
├── 01-failed-logon-investigation.md
├── 02-account-lockout-investigation.md
├── 03-user-account-creation.md
├── 04-privileged-group-change.md
├── 05-suspicious-powershell.md
├── 06-file-integrity-change.md
├── 07-defender-alert-validation.md
├── 08-firewall-block-investigation.md
├── 09-network-reconnaissance-detection.md
└── 10-ingestion-health-validation.md
```

The individual exercise files should contain the detailed procedures, expected telemetry, searches, cleanup steps, and findings.

---

## Exercise Goals

The exercise library is intended to develop practical experience with:

* Security event generation
* Windows event analysis
* Active Directory monitoring
* Wazuh alert investigation
* Splunk search development
* Detection validation
* False-positive analysis
* Timeline reconstruction
* Evidence collection
* Incident documentation
* Detection tuning
* Recovery testing
* Purple Team methodology

---

## Defensive Focus

The exercises are not intended to demonstrate how many offensive techniques can be performed.

Their purpose is to validate the defensive lifecycle:

```
Authorized Activity
        |
        v
Telemetry Generation
        |
        v
Collection and Ingestion
        |
        v
Detection or Search
        |
        v
Analyst Investigation
        |
        v
Tuning and Improvement
```

An exercise is incomplete if it generates activity but does not validate the resulting logs, alerts, searches, and investigation outcome.

---

## Public Example Environment

The public exercise documentation uses sanitized values such as:

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Kali system: KALI-TEST
Domain: cyberlab.example
Documentation subnet: 192.0.2.0/24
```

These values do not represent the operational CyberLab.

---

## Authorization Boundary

All exercises must remain limited to:

* CyberLab virtual machines
* Systems owned by the lab operator
* Dedicated test accounts
* Synthetic data
* Approved test directories
* Services intentionally configured for training
* Explicitly authorized network ranges

The exercises must not target:

* Public internet systems
* Employer systems
* School systems
* Neighboring wireless networks
* Household systems outside the approved scope
* ISP infrastructure
* Third-party cloud services
* Accounts not created for the lab

---

## Safety Requirements

Before beginning an exercise:

1. Confirm the exact source system.
2. Confirm the exact target system.
3. Confirm the host-only VMware network is active.
4. Confirm no unintended bridged adapter is connected.
5. Confirm Wazuh and Splunk are running.
6. Confirm the required agents or forwarders are active.
7. Confirm system time is synchronized.
8. Confirm stable snapshots exist.
9. Confirm the test account and data are disposable.
10. Record the exercise start time.

Stop the exercise immediately if the activity reaches an unintended system or network.

---

## Required Lab State

The baseline exercise environment should include:

* DC01 running
* Active Directory healthy
* Internal DNS working
* WIN11TARGET joined to the domain
* Windows audit policy applied
* PowerShell logging enabled where required
* Wazuh stack running
* Wazuh agent active
* Splunk running
* Splunk forwarder active when required
* Kali connected only to the approved lab network
* Accurate system time
* Adequate disk space
* Known-good snapshots

Not every exercise requires every system, but the exercise file should identify its dependencies.

---

## Recommended Startup Order

1. Start DC01.
2. Confirm Active Directory and DNS.
3. Start WIN11TARGET.
4. Confirm domain connectivity.
5. Start WAZUH-SERVER.
6. Confirm the Wazuh manager, indexer, and dashboard.
7. Confirm the Windows Wazuh agent is active.
8. Start SPLUNK-SERVER.
9. Start Splunk and confirm Splunk Web.
10. Confirm the Universal Forwarder connection.
11. Start KALI-TEST only when the exercise requires it.
12. Generate a harmless validation event.

---

## Standard Exercise Lifecycle

Each exercise should follow the same lifecycle.

### Prepare

Confirm the environment, monitoring, snapshots, accounts, and scope.

### Generate

Perform the smallest harmless action required to produce the expected telemetry.

### Observe

Review the local source event and confirm collection by the monitoring platforms.

### Investigate

Examine the relevant account, process, host, network, file, and timeline information.

### Validate

Determine whether the detection or search produced the expected result.

### Tune

Document missing fields, false positives, excessive noise, and rule improvements.

### Clean Up

Restore accounts, files, group memberships, services, and network state.

### Document

Save evidence, hashes, findings, and lessons learned.

---

## Standard Exercise Template

Each individual exercise should include:

```
Title:

Exercise ID:

Objective:

Difficulty:

Estimated scope:

Required systems:

Required accounts:

Required monitoring:

Authorized source:

Authorized target:

Preconditions:

Safety notes:

Activity:

Expected local telemetry:

Expected Wazuh telemetry:

Expected Splunk telemetry:

Investigation questions:

Validation steps:

Cleanup:

Evidence:

Detection gaps:

Tuning recommendations:

Conclusion:
```

---

## Evidence Requirements

Each completed exercise should preserve enough evidence to demonstrate the full workflow.

Recommended evidence includes:

* Exercise start and stop time
* Source system
* Target system
* Test account
* Command or action
* Local event identifier
* Windows Event Viewer screenshot
* Wazuh alert or event screenshot
* Splunk search result
* Packet capture when applicable
* Relevant query
* Timeline
* Evidence hashes
* Analyst conclusion
* Cleanup confirmation

Evidence should be sanitized before publication.

---

## Evidence Naming Standard

Recommended format:

```
<DATE>-<EXERCISE-ID>-<SOURCE>-<EVIDENCE-TYPE>
```

Examples:

```
YYYY-MM-DD-EX01-WIN11TARGET-event-log.evtx
YYYY-MM-DD-EX01-WAZUH-alert.png
YYYY-MM-DD-EX01-SPLUNK-search.csv
YYYY-MM-DD-EX09-KALI-packet-capture.pcap
YYYY-MM-DD-EX09-investigation-notes.md
```

Do not include passwords, operational IP addresses, or personal usernames in filenames.

---

## Evidence Hashing

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

Recommended record:

```
File:
Exercise:
Source system:
Collection date:
SHA-256:
Student analyst:
```

---

## Exercise Difficulty Levels

### Foundation

Focuses on one system and one clear event.

Examples:

* Successful sign-in
* Failed sign-in
* File creation
* Service start

### Intermediate

Requires correlation across multiple systems or logs.

Examples:

* Account lockout
* User creation
* Group membership change
* PowerShell execution

### Advanced

Requires custom searches, rule tuning, or multiple telemetry sources.

Examples:

* Reconnaissance detection
* Multi-event correlation
* Detection-gap analysis
* Custom Wazuh rule
* Splunk alert development

---

# Exercise 01: Failed Logon Investigation

## Objective

Generate a controlled failed authentication attempt and trace it through the source system, domain controller, Wazuh, and Splunk.

## Primary Skills

* Authentication-event analysis
* Event correlation
* Failure-reason interpretation
* Account and source identification
* SIEM searching

## Authorized Activity

Use a dedicated non-administrative test account and enter an incorrect password once.

Do not use a privileged or personal account.

## Expected Telemetry

Potential evidence includes:

* Failed authentication event
* Source workstation
* Target account
* Logon type
* Failure reason
* Domain controller authentication event
* Wazuh alert or event
* Splunk indexed record

## Investigation Questions

* Which account was targeted?
* Which system initiated the attempt?
* Was the account valid?
* What was the logon type?
* What failure reason was recorded?
* Was the event isolated or repeated?
* Did Wazuh and Splunk show the same source and time?
* Would the activity justify escalation in a real environment?

## Cleanup

* Confirm the account is not locked.
* Sign in successfully when appropriate.
* Record the result.
* Remove any temporary screenshots containing real values.

Detailed file:

```
01-failed-logon-investigation.md
```

---

# Exercise 02: Account Lockout Investigation

## Objective

Generate a controlled account lockout and investigate the preceding authentication failures.

## Primary Skills

* Active Directory account monitoring
* Multi-event correlation
* Root-cause identification
* Lockout source analysis
* Incident timeline creation

## Authorized Activity

Use a disposable test account and generate only the number of failures required by the lab lockout policy.

## Preconditions

* Lockout policy documented
* Separate administrator account available
* DC01 healthy
* Wazuh and Splunk collecting events
* Test account confirmed noncritical

## Expected Telemetry

Potential evidence includes:

* Multiple failed logons
* Account lockout event
* Caller computer
* Target account
* Domain controller record
* Wazuh alert
* Splunk event sequence

## Investigation Questions

* What caused the lockout?
* Which system generated the failures?
* How many failures occurred?
* Over what time period?
* Was the activity interactive, network-based, or service-related?
* Could a stale credential or scheduled task produce the same pattern?
* Were the SIEM timestamps aligned?

## Cleanup

* Unlock the test account.
* Confirm successful sign-in.
* Remove temporary credentials.
* Record the final account state.

Detailed file:

```
02-account-lockout-investigation.md
```

---

# Exercise 03: User Account Creation

## Objective

Create a temporary Active Directory user and validate that the identity change is visible in the monitoring platforms.

## Primary Skills

* Account-management auditing
* Active Directory investigation
* Actor and target identification
* Change validation
* Privileged activity monitoring

## Authorized Activity

Create a temporary nonprivileged account inside the designated test organizational unit.

## Expected Telemetry

Potential evidence includes:

* Account creation event
* Administrator performing the change
* New account name
* Domain controller
* Organizational unit
* Wazuh event
* Splunk event
* Follow-up account activity

## Investigation Questions

* Who created the account?
* Where was the account created?
* Was the creator authorized?
* Was the target account enabled?
* Was a password set or reset?
* Was the account added to any groups?
* Did the account later authenticate?
* Does the detection provide enough context?

## Cleanup

* Disable or delete the temporary account.
* Confirm the cleanup event.
* Remove the account from any test groups.
* Record the final state.

Detailed file:

```
03-user-account-creation.md
```

---

# Exercise 04: Privileged Group Change

## Objective

Add a test account to an approved lab group, investigate the change, and validate detection coverage.

## Primary Skills

* Group-membership monitoring
* Privilege-change analysis
* Actor and target correlation
* Active Directory event review
* Detection tuning

## Authorized Activity

Use a dedicated test group whenever possible.

Avoid adding an account to highly privileged groups unless the exercise specifically requires it and a snapshot exists.

## Expected Telemetry

Potential evidence includes:

* Group membership addition
* Account performing the change
* Target group
* Added account
* Domain controller
* Wazuh alert
* Splunk event
* Removal event during cleanup

## Investigation Questions

* Which account changed the group?
* Which group was modified?
* Which member was added?
* Did the change increase privilege?
* Was the action expected?
* Did the monitoring platform distinguish ordinary and privileged groups?
* Should the rule severity change based on the group?

## Cleanup

* Remove the test account from the group.
* Confirm the removal event.
* Validate the account’s effective permissions.
* Record cleanup.

Detailed file:

```
04-privileged-group-change.md
```

---

# Exercise 05: Suspicious PowerShell

## Objective

Generate a harmless PowerShell event and validate process, command-line, and script-logging visibility.

## Primary Skills

* PowerShell event analysis
* Process-tree review
* Command-line interpretation
* Script block logging
* Detection development

## Safe Test Command

```
Get-Service |
    Sort-Object Status |
    Select-Object -First 10
```

An additional benign process test may use:

```
Start-Process notepad.exe
```

## Expected Telemetry

Potential evidence includes:

* PowerShell process creation
* Parent process
* User
* Command line
* PowerShell Operational event
* Script block event
* Wazuh event
* Splunk event

## Investigation Questions

* Which user launched PowerShell?
* What was the parent process?
* Was the command line captured?
* Was the script content available?
* Did the activity run elevated?
* Was the behavior expected?
* Which fields would support a stronger detection?
* What normal administrative activity could resemble this event?

## Cleanup

* Close the test application.
* Remove temporary output files.
* Confirm no persistent script or scheduled task was created.
* Record the validation result.

Detailed file:

```
05-suspicious-powershell.md
```

---

# Exercise 06: File Integrity Change

## Objective

Create, modify, and delete a file inside an approved test directory and validate Wazuh file integrity monitoring.

## Primary Skills

* File integrity monitoring
* Hash comparison
* File-event analysis
* User and path validation
* Change classification

## Approved Test Path

```
C:\CyberLab-Test\Files
```

## Safe Test Procedure

Create a file:

```
New-Item `
    -Path "C:\CyberLab-Test\Files\fim-test.txt" `
    -ItemType File `
    -Force
```

Add content:

```
Set-Content `
    -Path "C:\CyberLab-Test\Files\fim-test.txt" `
    -Value "Authorized file integrity exercise."
```

Modify the file:

```
Add-Content `
    -Path "C:\CyberLab-Test\Files\fim-test.txt" `
    -Value "Controlled modification."
```

Delete it after validation:

```
Remove-Item `
    -Path "C:\CyberLab-Test\Files\fim-test.txt"
```

## Expected Telemetry

Potential evidence includes:

* File creation
* File modification
* File deletion
* Previous hash
* Current hash
* File path
* Endpoint
* User where available
* Wazuh rule and severity

## Investigation Questions

* Which file changed?
* What type of change occurred?
* Did the hash change?
* Was the path expected?
* Was the user known?
* Was the change authorized?
* Did each stage appear separately?
* Was the alert timely?

## Cleanup

* Confirm the test file is deleted.
* Confirm no unnecessary monitoring path remains.
* Export the Wazuh event if required.
* Record the result.

Detailed file:

```
06-file-integrity-change.md
```

---

# Exercise 07: Defender Alert Validation

## Objective

Use an officially documented harmless antivirus test mechanism to verify Microsoft Defender and SIEM visibility.

## Primary Skills

* Endpoint protection validation
* Antivirus-event analysis
* Alert correlation
* Remediation verification
* Security-control testing

## Safety Requirements

* Use only an official harmless test method.
* Keep the test inside the CyberLab.
* Do not email the artifact.
* Do not upload it to GitHub.
* Do not disable Defender.
* Do not introduce real malware.

## Expected Telemetry

Potential evidence includes:

* Local Defender alert
* Detection name
* Affected path
* Remediation status
* Defender Operational event
* Wazuh alert
* Splunk event when collected

## Investigation Questions

* What triggered the alert?
* Which file or process was involved?
* Did Defender block or quarantine it?
* Was remediation successful?
* Did Wazuh receive the event?
* Did Splunk receive the Defender channel?
* Were the timestamps aligned?
* Did the platforms assign appropriate severity?

## Cleanup

* Confirm the artifact is removed or quarantined.
* Confirm Defender reports a healthy state.
* Run an update or scan when appropriate.
* Record the remediation result.

Detailed file:

```
07-defender-alert-validation.md
```

---

# Exercise 08: Firewall Block Investigation

## Objective

Generate a controlled connection to an approved blocked port and validate firewall telemetry.

## Primary Skills

* Host firewall analysis
* Network connection validation
* Source and destination correlation
* Packet-level troubleshooting
* Telemetry-gap analysis

## Preconditions

* Windows Firewall enabled
* Blocked-connection logging enabled
* Target and port authorized
* No required application depends on the test port
* Kali connected only to the host-only network

## Authorized Test

From Kali:

```
nc -vz <AUTHORIZED_TARGET> <APPROVED_BLOCKED_PORT>
```

## Expected Telemetry

Potential evidence includes:

* Kali connection result
* Windows Firewall log
* Packet capture
* Sysmon network event when configured
* Splunk event when the firewall log is collected
* Wazuh event when configured

## Investigation Questions

* Was the connection blocked?
* Which firewall profile applied?
* What were the source and destination?
* Was the event visible in Windows logs?
* Was a packet capture required?
* Did Wazuh or Splunk collect the firewall data?
* What telemetry was missing?
* Would Sysmon or a network IDS improve coverage?

## Cleanup

* Stop packet capture.
* Confirm no firewall rule was changed unnecessarily.
* Remove temporary test rules.
* Save the sanitized evidence.
* Record any telemetry gap.

Detailed file:

```
08-firewall-block-investigation.md
```

---

# Exercise 09: Network Reconnaissance Detection

## Objective

Generate a limited authorized port scan and determine which defensive controls observe the activity.

## Primary Skills

* Network reconnaissance analysis
* Nmap output interpretation
* Packet capture
* Firewall-log analysis
* SIEM correlation
* Detection-gap identification

## Authorized Test

```
nmap -sT \
    -p <APPROVED_PORT_LIST> \
    <AUTHORIZED_TARGET>
```

The target must be a CyberLab system, and the port list should remain narrow.

## Expected Telemetry

Potential evidence includes:

* Kali scan output
* TCP connection attempts
* Target firewall logs
* Sysmon network events when configured
* Packet capture
* Wazuh telemetry
* Splunk telemetry
* Future Suricata or Zeek events

## Investigation Questions

* Which ports were tested?
* Which ports responded?
* How quickly did the scan occur?
* Did the endpoint record each connection?
* Did Wazuh generate an alert?
* Did Splunk receive sufficient data?
* Was packet capture the only reliable evidence?
* What additional data source would improve detection?

## Detection Development Opportunity

A lack of alerting may indicate the need for:

* Windows Firewall log collection
* Sysmon network telemetry
* Suricata
* Zeek
* Custom Wazuh rules
* Splunk threshold search
* Connection-rate correlation

## Cleanup

* Stop the scan.
* Stop the packet capture.
* Confirm the target remains healthy.
* Export the required evidence.
* Record detection gaps and recommendations.

Detailed file:

```
09-network-reconnaissance-detection.md
```

---

# Exercise 10: Ingestion Health Validation

## Objective

Generate a harmless known event and verify the complete collection path from source to Wazuh and Splunk.

## Primary Skills

* Pipeline validation
* Agent and forwarder troubleshooting
* Timestamp analysis
* Metadata validation
* Data-quality assessment

## Test Activity

Use a simple benign event, such as:

```
Get-Date
```

or:

```
Start-Process notepad.exe
```

The selected event must correspond to an enabled audit or collection source.

## Validation Path

```
Action
  |
  v
Local Windows Event
  |
  v
Wazuh Agent and Splunk Forwarder
  |
  v
Network Transport
  |
  v
Wazuh and Splunk
  |
  v
Search and Comparison
```

## Investigation Questions

* Did the local event exist?
* Was the Wazuh agent active?
* Was the Splunk forwarder active?
* Did both platforms receive the event?
* Were host, source, and timestamp values correct?
* Was ingestion delayed?
* Were fields missing?
* Were duplicate events present?

## Cleanup

* Close any test process.
* Export evidence when required.
* Record ingestion delay.
* Document configuration issues.

Detailed file:

```
10-ingestion-health-validation.md
```

---

## Detection Validation Standard

A detection should not be marked complete solely because an alert appears.

The exercise should confirm:

* The activity was authorized and repeatable.
* The source event exists.
* The correct system generated the event.
* The required agent or forwarder was active.
* The event reached the platform.
* The event was parsed correctly.
* The detection logic matched for the intended reason.
* Normal activity was tested.
* False-positive conditions were considered.
* Cleanup succeeded.

---

## Detection Documentation Template

```
Detection name:

Detection objective:

Data source:

Required telemetry:

Platform:

Query or rule:

Expected match:

Expected non-match:

Severity:

MITRE ATT&CK mapping:

Known false positives:

Known limitations:

Validation exercise:

Evidence:

Tuning history:

Response guidance:
```

---

## Splunk Detection Development

A Splunk detection may be developed as:

* Ad hoc SPL
* Saved search
* Scheduled alert
* Dashboard panel
* Report
* Correlation search in an appropriate product

A generic search structure:

```spl
index=<INDEX_NAME> host=<HOST>
| search <CONDITION>
| table _time host user source EventCode
| sort - _time
```

Begin with known events and narrow filters.

---

## Splunk Validation Questions

* Is the correct index used?
* Is the host field accurate?
* Is the sourcetype correct?
* Are important fields extracted?
* Is the time range appropriate?
* Does the search detect the known test?
* Does it avoid unrelated normal events?
* Is the threshold justified?
* Is the query efficient?
* Is the result understandable to another analyst?

---

## Wazuh Detection Development

A Wazuh detection may use:

* Existing rules
* Custom local rules
* Custom decoders
* File integrity monitoring
* Windows event channels
* Agent groups
* Active response in later controlled exercises

A custom rule should include:

* Clear description
* Appropriate rule identifier
* Useful severity
* Required fields
* Expected matching event
* Expected nonmatching event
* False-positive notes
* Validation procedure
* Rollback procedure

---

## Wazuh Validation Questions

* Was the event decoded?
* Which rule matched?
* Was the rule level appropriate?
* Did the rule match the intended fields?
* Was the agent identified correctly?
* Was MITRE mapping useful?
* Were compliance tags relevant?
* Did unrelated events match?
* Is a custom rule required?
* Does the rule need tuning?

---

## MITRE ATT&CK Mapping

Exercises may be mapped to MITRE ATT&CK where the behavior reasonably relates to a documented technique.

Potential learning examples include:

* Valid Accounts
* Account Discovery
* Network Service Scanning
* Command and Scripting Interpreter
* PowerShell
* Create Account
* Account Manipulation
* File and Directory Discovery

A mapping should be used as analytical context.

It does not prove malicious intent.

The exercise documentation should state that the activity was authorized.

---

## False-Positive Analysis

A detection should identify legitimate behavior that may resemble the test.

Examples include:

* User mistyping a password
* Help-desk password reset
* Administrator creating an approved account
* Software installer launching PowerShell
* Configuration-management tool changing a file
* Vulnerability scanner probing ports
* Monitoring service connecting repeatedly
* Scheduled task using stale credentials

The exercise should document how an analyst could distinguish normal and suspicious behavior.

---

## Detection Tuning

Potential tuning methods include:

* Narrowing the event source
* Requiring specific fields
* Adding a threshold
* Adding a time window
* Excluding approved service accounts
* Excluding known management systems
* Increasing severity for privileged targets
* Correlating multiple related events
* Adding process-parent context
* Adding source-network context

Avoid suppressing an event solely because it is inconvenient or noisy.

---

## Severity Assignment

Severity should consider:

* Impact
* Confidence
* Account privilege
* System criticality
* Frequency
* Source trust
* Persistence
* Related activity
* Detection quality

Example conceptual scale:

| Severity      | General Meaning                                         |
| ------------- | ------------------------------------------------------- |
| Informational | Useful context with no immediate concern                |
| Low           | Minor anomaly or weak signal                            |
| Medium        | Suspicious activity requiring review                    |
| High          | Strong indication of unauthorized or high-risk behavior |
| Critical      | Immediate threat to critical systems or identities      |

The scale should be adapted to the platform.

---

## Investigation Workflow

A standard investigation should include:

1. Confirm the alert time.
2. Confirm the affected system.
3. Confirm the user or account.
4. Review the raw event.
5. Review related events before and after it.
6. Identify the source system.
7. Identify the process or protocol.
8. Compare with approved exercise activity.
9. Determine whether the event is expected.
10. Record the conclusion and recommended action.

---

## Investigation Classification

Suggested outcomes:

```
Authorized test activity
Benign administrative activity
Expected user activity
Misconfiguration
False positive
Suspicious activity
Confirmed unauthorized activity
Insufficient evidence
Telemetry gap
```

For this exercise library, most events should conclude as authorized test activity while still demonstrating how the investigation would work in a real environment.

---

## Timeline Template

```
Time | System | Source | Event | Account | Analyst Note
```

Example:

```
10:00:00 | KALI-TEST | Nmap | Scan started | student.user | Authorized exercise
10:00:02 | WIN11TARGET | Firewall | Connection blocked | N/A | Expected target telemetry
10:00:05 | WAZUH-SERVER | Wazuh | Alert indexed | N/A | Rule reviewed
10:00:08 | SPLUNK-SERVER | Splunk | Event searchable | N/A | Ingestion delay measured
```

Use sanitized values publicly.

---

## Detection Coverage Matrix

A coverage matrix can show which platform detects each exercise.

| Exercise              |        Local Event |              Wazuh |             Splunk | Packet Capture | Detection Status |
| --------------------- | -----------------: | -----------------: | -----------------: | -------------: | ---------------- |
| Failed logon          |                Yes |                Yes |                Yes |       Optional | Validated        |
| Account lockout       |                Yes |                Yes |                Yes |       Optional | Planned          |
| User creation         |                Yes |                Yes |                Yes |             No | Planned          |
| Group change          |                Yes |                Yes |                Yes |             No | Planned          |
| PowerShell            |                Yes |  Depends on policy |                Yes |             No | Planned          |
| File integrity change |      File evidence |                Yes |           Optional |             No | Planned          |
| Defender test         |                Yes |                Yes |   Depends on input |             No | Planned          |
| Firewall block        | Depends on logging |   Depends on input |   Depends on input |            Yes | Planned          |
| Reconnaissance        |            Limited | May require tuning | May require tuning |            Yes | Planned          |
| Ingestion health      |                Yes |                Yes |                Yes |             No | Planned          |

This table should be updated as each exercise is completed.

---

## Exercise Status Values

Recommended values:

```
Planned
Prepared
In Progress
Validated
Needs Tuning
Blocked
Retest Required
Complete
```

A validated exercise may still require detection tuning.

---

## Exercise Completion Criteria

An exercise is complete when:

* Scope was documented.
* Required systems were healthy.
* The activity was performed safely.
* The source event was confirmed.
* Wazuh was reviewed.
* Splunk was reviewed.
* Relevant evidence was saved.
* The investigation questions were answered.
* Cleanup was completed.
* Detection gaps were documented.
* Public evidence was sanitized.
* The conclusion was recorded.

---

## Exercise Failure Criteria

An exercise should be stopped or marked incomplete when:

* The target is uncertain.
* The activity reached an unauthorized system.
* A required snapshot is missing.
* Monitoring was unavailable.
* The event was not generated locally.
* System time was incorrect.
* The action caused unexpected instability.
* Evidence cannot be trusted.
* Cleanup cannot be completed safely.
* The scope changed during the exercise.

---

## Cleanup Checklist

After every exercise:

1. Stop active commands and scans.
2. Stop packet captures.
3. Close test applications.
4. Remove temporary files.
5. Restore group memberships.
6. Unlock, disable, or delete test accounts as planned.
7. Remove temporary firewall rules.
8. Confirm agents and forwarders are healthy.
9. Export evidence.
10. Record hashes.
11. Confirm target-system health.
12. Create a stable snapshot when appropriate.

---

## Snapshot Guidance

Snapshots should be created before exercises that change:

* Active Directory accounts
* Group memberships
* Audit policy
* Firewall rules
* Wazuh configuration
* Splunk inputs
* Sysmon configuration
* Network adapters
* System services

Recommended snapshot names:

```
PRE-EX01-Failed-Logon
PRE-EX02-Account-Lockout
PRE-EX04-Group-Change
PRE-EX07-Defender-Test
PRE-EX09-Network-Recon
```

Snapshots are not a replacement for independent backups.

---

## Post-Restore Validation

After restoring a snapshot:

1. Confirm system time.
2. Confirm DNS.
3. Confirm the domain secure channel.
4. Confirm test-account state.
5. Confirm Wazuh agent connectivity.
6. Confirm Splunk forwarder connectivity.
7. Confirm indexes and alerts.
8. Generate a harmless known event.
9. Check for duplicates or missing events.
10. Document the restore.

---

## Troubleshooting: No Local Event

Check:

* Audit policy
* Group Policy
* Event channel
* Test procedure
* Event-log service
* Account used
* Operating system role
* Required privilege
* Event-log capacity

The SIEM cannot collect an event that the source never generated.

---

## Troubleshooting: Wazuh Does Not Show the Event

Check:

* Agent status
* Manager connectivity
* Event-channel configuration
* Decoder
* Rule
* Rule level
* Agent filter
* Dashboard time range
* Indexer health
* System time

A raw event may be collected without creating a high-level alert.

---

## Troubleshooting: Splunk Does Not Show the Event

Check:

* Forwarder service
* Receiver listener
* `inputs.conf`
* `outputs.conf`
* Index name
* Time range
* Host field
* Source
* Sourcetype
* Event time
* Forwarder logs

Begin with a broad search limited to the expected index and timeframe, then add filters gradually.

---

## Troubleshooting: Timestamps Do Not Align

Check:

* DC01 time
* WIN11TARGET time
* Wazuh time
* Splunk time
* Kali time
* Browser timezone
* `_time`
* `_indextime`
* Snapshot history
* VMware Tools synchronization

Do not build a timeline until the time difference is understood.

---

## Troubleshooting: Too Much Noise

Reduce noise by adjusting:

* Test duration
* Attempt count
* Target count
* Port count
* Audit scope
* FIM path
* Search time range
* Rule threshold
* Approved exclusions

Do not disable an entire log source solely to eliminate a small number of unwanted events.

---

## Troubleshooting: No Alert for Reconnaissance

A basic port scan may not generate a native Windows alert.

Possible improvements include:

* Enable firewall logging.
* Deploy Sysmon.
* Collect firewall logs in Splunk.
* Add Suricata.
* Add Zeek.
* Develop a connection-threshold search.
* Create a custom Wazuh rule.
* Correlate repeated destination ports from one source.

Document the absence of detection as a valid finding.

---

## Public Sanitization Standards

Remove or replace:

* Operational IP addresses
* Real domain names
* Personal usernames
* Email addresses
* Passwords
* Security identifiers
* Agent identifiers
* Enrollment keys
* Splunk credentials
* Session cookies
* Internal URLs
* MAC addresses
* VMware identifiers
* Home-network details
* Raw packet captures
* Real account names
* Real command history
* Personal file paths
* Unrelated alerts

Use placeholders such as:

```
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<KALI_SYSTEM>
<TEST_ACCOUNT>
<AUTHORIZED_TARGET>
<APPROVED_PORT>
<EVENT_ID>
<INDEX_NAME>
<REDACTED_VALUE>
```

---

## Screenshot Sanitization

Before publishing exercise screenshots:

1. Use a neutral test account.
2. Select a narrow time range.
3. Close unrelated applications.
4. Hide browser bookmarks.
5. Clear notifications.
6. Remove internal addresses.
7. Remove usernames where required.
8. Remove domain names.
9. Remove security identifiers.
10. Remove tokens and session details.
11. Crop unrelated interface areas.
12. Permanently redact sensitive values.
13. Reopen the image.
14. Confirm the image supports the exercise objective.

---

## Suitable Public Evidence

Good public examples include:

* Sanitized failed-logon event
* Sanitized Wazuh alert
* Sanitized Splunk search
* Synthetic PowerShell telemetry
* File integrity alert
* Defender test notification
* Sanitized firewall event
* Limited scan output using placeholders
* Coverage matrix
* Investigation timeline
* Detection tuning notes

---

## Skills Demonstrated

This exercise library demonstrates:

* Detection engineering
* Blue Team validation
* Windows event analysis
* Active Directory monitoring
* PowerShell telemetry
* Wazuh administration
* Splunk SPL
* Alert investigation
* File integrity monitoring
* Firewall analysis
* Network reconnaissance detection
* Packet capture
* Evidence handling
* Timeline reconstruction
* False-positive analysis
* Detection tuning
* MITRE ATT&CK interpretation
* Change management
* Snapshot recovery
* Public sanitization

---

## Lessons Learned

Key lessons include:

* Detection engineering begins with reliable telemetry.
* A visible alert does not prove the rule matched for the correct reason.
* The source event should be reviewed before the SIEM.
* Wazuh and Splunk may present the same event differently.
* Missing alerts often expose telemetry or detection gaps.
* Narrow exercises are easier to validate than broad attack simulations.
* Account and group changes require careful cleanup.
* PowerShell visibility depends on audit and logging configuration.
* Network reconnaissance may require firewall, Sysmon, or network-sensor data.
* Accurate timestamps are essential for correlation.
* Evidence should be exported before snapshot restoration.
* False-positive testing is part of detection validation.
* Authorized activity should still be investigated as though its intent were initially unknown.
* Public documentation requires a separate sanitization review.

---

## Planned Expansion

Future exercises may include:

* Suspicious service creation
* Scheduled task creation
* Remote Desktop authentication
* Windows Remote Management
* SMB access
* DNS anomaly investigation
* New computer account creation
* Local administrator group change
* Defender configuration change
* Security-log clearing
* Sysmon process injection telemetry
* Encoded PowerShell simulation using safe frameworks
* Linux authentication failures
* Linux privilege escalation telemetry
* Wazuh custom-rule development
* Sigma-to-SPL conversion
* Sigma-to-Wazuh conversion
* Suricata alert investigation
* Zeek connection analysis
* Multi-stage Purple Team scenario
* Incident report development

---

## Summary

The detection engineering exercise library converts the Acer CyberLab from a collection of installed tools into a repeatable defensive training environment.

Each exercise is designed to validate:

* Event generation
* Local logging
* Agent or forwarder collection
* Network transport
* Wazuh visibility
* Splunk visibility
* Detection logic
* Analyst investigation
* Evidence handling
* Cleanup and recovery

The value of the exercises comes from documenting not only what was detected, but also what was missing, why it was missing, and how the defensive environment could be improved.

