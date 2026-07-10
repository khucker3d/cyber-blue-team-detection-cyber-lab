# Failed Logon Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating failed Windows authentication activity in the CyberLab.

It is intended for events involving:

* Windows Event ID `4625`
* Kerberos pre-authentication failures
* NTLM credential-validation failures
* Repeated failed sign-in attempts
* Failed Remote Desktop authentication
* Failed network logons
* Failed local or domain account authentication
* Wazuh or Splunk alerts related to failed logons

The runbook helps the analyst determine:

* Which account was targeted
* Which system generated the activity
* Why authentication failed
* Whether the activity was isolated or repeated
* Whether the source is expected
* Whether the account later authenticated successfully
* Whether the activity represents user error, stale credentials, password spraying, brute force, or another security concern
* Whether containment or escalation is required

---

## Runbook ID

```
RB-01
```

---

## Related Exercise

```
EX-01 – Failed Logon Investigation
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Windows administrators
* Detection engineers
* Incident responders
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Domain user accounts
* Local Windows accounts
* Administrative accounts
* Service accounts
* Remote Desktop authentication
* Interactive logons
* Network logons
* Scheduled task or service authentication
* Authentication events recorded by endpoints or domain controllers

This runbook does not authorize:

* Password guessing
* Account testing outside the CyberLab
* Unauthorized credential use
* Active response against external systems
* Disabling accounts without appropriate approval
* Changing production security policy

---

## Primary Data Sources

Use the available data sources in this order:

1. Windows Security logs
2. Domain controller authentication logs
3. Wazuh events and alerts
4. Splunk indexed events
5. Active Directory account state
6. Endpoint process and network telemetry
7. Firewall logs
8. Optional Sysmon telemetry
9. Optional packet capture
10. Administrative or change records

---

## Key Windows Event IDs

| Event ID | Description                                           |
| -------: | ----------------------------------------------------- |
|     4625 | An account failed to log on                           |
|     4624 | An account successfully logged on                     |
|     4740 | A user account was locked out                         |
|     4767 | A user account was unlocked                           |
|     4771 | Kerberos pre-authentication failed                    |
|     4776 | A domain controller attempted to validate credentials |
|     4768 | A Kerberos authentication ticket was requested        |
|     4769 | A Kerberos service ticket was requested               |
|     4648 | A logon was attempted using explicit credentials      |
|     4672 | Special privileges were assigned to a new logon       |

Not every event appears during every authentication path.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh failed-logon alert
* Splunk correlation search
* Windows Event Viewer review
* Account lockout report
* User report
* Administrator report
* Remote Desktop failure
* Service failure
* Scheduled task failure
* Firewall or network alert
* Ingestion-health validation

---

## Minimum Alert Information

Record the following before beginning:

```
Alert time:
Alert source:
Alert name:
Reporting host:
Target account:
Target domain:
Source system:
Source address:
Event ID:
Logon type:
Failure reason:
Alert severity:
Analyst:
```

Do not assume the alert contains correct or complete field extraction.

---

## Initial Analyst Questions

Determine immediately:

* Is the target account privileged?
* Is the account currently locked?
* Is the source system known?
* Is the source internal or external?
* Is the activity still occurring?
* How many failures occurred?
* How many accounts were targeted?
* Did a successful logon follow?
* Is there an approved test or maintenance activity?
* Is this an isolated failure or a pattern?

---

# Severity Classification

## Informational

Use informational severity when:

* One isolated failure occurred
* The source is known
* The target account is nonprivileged
* The failure aligns with a user typing error
* No lockout occurred
* No suspicious follow-on activity exists
* The event is part of an approved test

Example:

```
One failed interactive sign-in from the user’s normal workstation, followed by a successful sign-in.
```

---

## Low

Use low severity when:

* Several failures occurred for one nonprivileged account
* The source is expected
* Stale credentials are likely
* A service or scheduled task may be involved
* No privilege or external source is involved
* No successful suspicious authentication followed

---

## Medium

Use medium severity when:

* Failures are repeated
* The source system is unusual
* Several accounts are targeted
* The target account is sensitive
* The event occurs outside expected hours
* The source cannot be attributed quickly
* A lockout occurs
* A new or recently changed account is targeted

---

## High

Use high severity when:

* A privileged account is targeted repeatedly
* One source targets many accounts
* One account is targeted from multiple sources
* Failed attempts are followed by a successful logon
* The source is external or unauthorized
* The activity resembles password spraying or brute force
* The target account is a service, administrator, or recovery account
* Related suspicious process or network activity exists

---

## Critical

Use critical severity when:

* A privileged account is compromised
* Failed attempts are followed by confirmed unauthorized access
* Multiple privileged accounts are affected
* Authentication activity is followed by privilege escalation
* The activity spreads across several systems
* Identity controls are being disabled or modified
* A domain-wide attack is suspected

---

# Initial Triage

## Step 1: Confirm the Alert Time

Record the alert time and the timezone shown by:

* Windows
* Wazuh
* Splunk
* The analyst workstation

Use absolute timestamps.

Example:

```
2026-07-10 14:32:18 PDT
```

Do not rely only on relative terms such as “five minutes ago.”

---

## Step 2: Confirm the Event Exists at the Source

On the reporting Windows system, run as an administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Search for the target account:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

If the event is not present locally, investigate:

* Incorrect event source
* Wrong host
* Wrong time range
* Incorrect parsing
* Stale SIEM event
* Duplicate or forwarded event

---

## Step 3: Validate the Target Account

For a domain account, run on DC01 or from an authorized administrative system:

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
    -Properties `
        Enabled,
        LockedOut,
        PasswordExpired,
        PasswordLastSet,
        LastBadPasswordAttempt,
        BadLogonCount,
        LastLogonDate,
        MemberOf |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    PasswordExpired,
    PasswordLastSet,
    LastBadPasswordAttempt,
    BadLogonCount,
    LastLogonDate,
    MemberOf
```

Record:

```
Account enabled:
Account locked:
Password expired:
Password last changed:
Last bad password attempt:
Bad logon count:
Last successful logon:
Privileged membership:
```

Do not expose operational group membership publicly.

---

## Step 4: Identify the Source

Use the event fields to identify:

* Workstation name
* Source network address
* Source port
* Caller process
* Reporting host
* Authentication package

Validate the source against:

* DNS
* DHCP records
* Known VM identity
* Asset inventory
* VMware network configuration
* User’s normal workstation
* Approved administrative hosts

Do not assume a hostname and IP address refer to the same system without validation.

---

## Step 5: Determine the Failure Reason

Review:

* Failure reason
* Status
* Substatus
* Event ID
* Account state
* Authentication package

Common causes include:

* Bad password
* Unknown username
* Disabled account
* Expired account
* Locked account
* Logon restriction
* Workstation restriction
* Time restriction
* Unauthorized logon type

Use status codes together with the human-readable failure reason.

---

# Event ID 4625 Analysis

## Important Sections

### Subject

This identifies the security context requesting the authentication.

Review:

* Security ID
* Account name
* Account domain
* Logon ID

---

### Account for Which Logon Failed

This identifies the targeted account.

Review:

* Security ID
* Account name
* Account domain

---

### Failure Information

Review:

* Failure reason
* Status
* Substatus

---

### Process Information

Review:

* Caller process ID
* Caller process name

---

### Network Information

Review:

* Workstation name
* Source network address
* Source port

---

### Detailed Authentication Information

Review:

* Logon process
* Authentication package
* Transited services
* Package name
* Key length

---

## Common Logon Types

| Logon Type | Meaning            | Investigation Context                                                 |
| ---------: | ------------------ | --------------------------------------------------------------------- |
|          2 | Interactive        | Local console sign-in                                                 |
|          3 | Network            | SMB, remote resource, network authentication                          |
|          4 | Batch              | Scheduled task                                                        |
|          5 | Service            | Windows service                                                       |
|          7 | Unlock             | Workstation unlock                                                    |
|          8 | Network cleartext  | Authentication with cleartext credentials inside a protected protocol |
|          9 | New credentials    | Alternate credentials or `runas /netonly`                             |
|         10 | Remote interactive | Remote Desktop                                                        |
|         11 | Cached interactive | Cached domain logon                                                   |

The logon type should match the reported activity.

---

## Common Investigative Interpretations

### Logon Type 2

Possible causes:

* User typed the wrong password
* Wrong username format
* Local versus domain account confusion
* Password recently changed

### Logon Type 3

Possible causes:

* Mapped drive
* SMB access
* Remote administration
* Stale saved credentials
* Password spray
* Service-to-service authentication

### Logon Type 5

Possible causes:

* Service account password changed
* Service configured with stale credentials
* Disabled service account
* Expired password

### Logon Type 10

Possible causes:

* Remote Desktop attempt
* Administrative access
* Unauthorized remote access
* Stale Remote Desktop credentials

---

# Domain Controller Correlation

## Search for Event ID 4771

Run on DC01:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4771
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

Review:

* Account name
* Client address
* Failure code
* Pre-authentication type
* Ticket options

---

## Search for Event ID 4776

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4776
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

Review:

* Account
* Source workstation
* Status
* Domain controller
* Timestamp

---

## Search for Event ID 4740

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

If present, transition to the account lockout runbook.

---

## Search for Successful Authentication

Search for Event ID `4624`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

Determine whether a successful authentication occurred:

* Before the failures
* During the failures
* Immediately afterward
* From the same source
* From a different source

A failure followed by success may require escalation.

---

# Wazuh Investigation

## Search by Account

Search for:

```
<TARGET_ACCOUNT>
```

Use a narrow time window around the event.

---

## Search by Event ID

Review:

```
4625
4771
4776
4740
4624
```

---

## Review Wazuh Fields

Record:

```
Agent:
Rule ID:
Rule description:
Rule level:
Windows event ID:
Target account:
Subject account:
Source workstation:
Source address:
Logon type:
Failure reason:
Authentication package:
Timestamp:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Did the endpoint event appear?
* Did the domain controller event appear?
* Did Wazuh correlate repeated failures?
* Was the account parsed correctly?
* Was the source address parsed correctly?
* Was the logon type visible?
* Was the rule severity appropriate?
* Did a lockout alert appear?
* Did a success event follow?
* Were any fields missing?

---

# Splunk Investigation

## Search for the Target Account

```spl
index=windows earliest=-24h
"<TARGET_ACCOUNT>"
| table
    _time
    host
    source
    sourcetype
    EventCode
    user
    Message
| sort _time
```

Adjust the index for the actual deployment.

---

## Search Failed Logons

```spl
index=windows EventCode=4625 earliest=-24h
| search "<TARGET_ACCOUNT>"
| table
    _time
    host
    SubjectUserName
    TargetUserName
    LogonType
    FailureReason
    Status
    SubStatus
    SourceNetworkAddress
    WorkstationName
    ProcessName
    AuthenticationPackageName
| sort _time
```

---

## Search Related Domain Events

```spl
index=windows earliest=-24h
(
    EventCode=4771 OR
    EventCode=4776 OR
    EventCode=4740
)
| search "<TARGET_ACCOUNT>"
| table
    _time
    host
    EventCode
    TargetUserName
    Workstation
    IpAddress
    Status
    FailureCode
    Message
| sort _time
```

---

## Build an Authentication Timeline

```spl
index=windows earliest=-24h
(
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4740 OR
    EventCode=4767 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| eval activity=case(
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4740, "Account locked",
    EventCode=4767, "Account unlocked",
    EventCode=4771, "Kerberos pre-authentication failure",
    EventCode=4776, "Credential validation",
    true(), "Related authentication event"
)
| table
    _time
    host
    EventCode
    activity
    TargetUserName
    SubjectUserName
    SourceNetworkAddress
    WorkstationName
    LogonType
    FailureReason
| sort _time
```

---

## Count Failures by Account

```spl
index=windows EventCode=4625 earliest=-15m
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as failed_attempts
    dc(SourceNetworkAddress) as unique_sources
    values(SourceNetworkAddress) as source_addresses
    values(host) as reporting_hosts
    values(LogonType) as logon_types
    min(_time) as first_seen
    max(_time) as last_seen
    by target_account
| sort - failed_attempts
```

---

## One Source Targeting Multiple Accounts

```spl
index=windows EventCode=4625 earliest=-15m
| eval source_address=coalesce(SourceNetworkAddress, src_ip)
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as failed_attempts
    dc(target_account) as unique_accounts
    values(target_account) as targeted_accounts
    values(host) as destination_hosts
    by source_address
| where unique_accounts >= <ACCOUNT_THRESHOLD>
| sort - unique_accounts
```

This pattern may indicate password spraying.

---

## One Account Targeted from Multiple Sources

```spl
index=windows EventCode=4625 earliest=-15m
| eval source_address=coalesce(SourceNetworkAddress, src_ip)
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as failed_attempts
    dc(source_address) as unique_sources
    values(source_address) as source_addresses
    by target_account
| where unique_sources >= <SOURCE_THRESHOLD>
| sort - unique_sources
```

---

## Failures Followed by Success

```spl
index=windows earliest=-30m
(
    EventCode=4624 OR
    EventCode=4625
)
| eval target_account=coalesce(TargetUserName, user)
| eval result=if(EventCode=4624, "success", "failure")
| stats
    count(eval(result="failure")) as failures
    count(eval(result="success")) as successes
    min(_time) as first_seen
    max(_time) as last_seen
    values(SourceNetworkAddress) as source_addresses
    values(host) as hosts
    by target_account
| where failures > 0 AND successes > 0
| convert ctime(first_seen) ctime(last_seen)
```

This search identifies candidates for further investigation but does not prove compromise.

---

# Source-System Investigation

## Validate the Source Host

Confirm:

* Hostname
* IP address
* Logged-on user
* Asset owner
* Operating system
* Network adapter
* Expected role
* Current power state

For Windows:

```
hostname
```

```
Get-NetIPConfiguration
```

```
quser
```

For Linux:

```
hostname
```

```
ip address
```

```
who
```

---

## Review Running Processes

On a Windows source:

```
Get-Process |
    Sort-Object ProcessName
```

Review recent process creation if Sysmon or Event ID `4688` is available.

Search for:

* Remote Desktop clients
* PowerShell
* Command Prompt
* Scheduled task engines
* Service processes
* Mapping scripts
* Unknown executables

---

## Review Saved Credentials

On a Windows source:

```
cmdkey /list
```

Do not publish the full output.

Saved credentials may explain repeated authentication failures.

Do not delete credentials until evidence is preserved and ownership is confirmed.

---

## Review Mapped Drives

```
Get-SmbMapping
```

Alternative:

```
net use
```

A stale mapped drive may repeatedly authenticate with an old password.

---

## Review Scheduled Tasks

```
Get-ScheduledTask |
    Where-Object {
        $_.Principal.UserId -match "<TARGET_ACCOUNT>"
    } |
Select-Object `
    TaskName,
    TaskPath,
    State,
    Author
```

---

## Review Services

```
Get-CimInstance Win32_Service |
    Where-Object {
        $_.StartName -match "<TARGET_ACCOUNT>"
    } |
Select-Object `
    Name,
    DisplayName,
    State,
    StartMode,
    StartName
```

A stale service password is a common cause of repeated failures.

---

# Investigation Decision Tree

```
Failed Logon Alert
        |
        v
Does the event exist locally?
        |
   +----+----+
   |         |
  No        Yes
   |         |
Check      Is the account
parsing,   privileged or locked?
host,           |
time range      v
          +-----+-----+
          |           |
         Yes          No
          |           |
   Increase       Determine
   severity       failure count
                      |
                      v
            One isolated failure?
                 |
          +------+------+
          |             |
         Yes            No
          |             |
 Check user error,   Check repetition,
 stale credential,  targeted accounts,
 approved testing   source systems
                        |
                        v
             One source, many accounts?
                        |
                 +------+------+
                 |             |
                Yes            No
                 |             |
          Suspected spray   One account,
          or enumeration   many sources?
                                  |
                           +------+------+
                           |             |
                          Yes            No
                           |             |
                    Possible attack   Review service,
                    or distributed   task, cached or
                    activity        application credentials
```

---

# Common Investigation Scenarios

## Scenario 1: User Typing Error

Indicators:

* One or two failures
* User’s normal workstation
* Interactive logon
* Successful logon shortly afterward
* No lockout
* No additional suspicious activity

Response:

* Document as benign
* Confirm user recognizes activity where needed
* Close with no containment

---

## Scenario 2: Recently Changed Password

Indicators:

* Failures begin after a password change
* Several systems or applications use old credentials
* Mapped drive, service, task, or mobile client involved
* Activity stops after credentials are updated

Response:

* Identify all stale credential sources
* Update credentials securely
* Confirm failures stop
* Do not request the user’s password

---

## Scenario 3: Service Account Failure

Indicators:

* Logon Type `5`
* Repeated failures at regular intervals
* Same source host
* Service or scheduled task configured with target account
* Password recently changed or expired

Response:

* Coordinate credential update
* Avoid disabling a critical service account without impact review
* Confirm service recovery
* Validate no unauthorized use occurred

---

## Scenario 4: Password Spraying

Indicators:

* One source targets many accounts
* Small number of attempts per account
* Similar timing
* Same destination or domain controller
* No immediate account lockout for each user

Response:

* Escalate
* Identify all targeted accounts
* Block or isolate unauthorized source where appropriate
* Review successful logons
* Reset affected credentials based on evidence and policy
* Review privileged accounts first

---

## Scenario 5: Brute Force

Indicators:

* Many attempts against one account
* Short time interval
* Repeated source address
* Lockout may occur
* Remote or network logon type

Response:

* Escalate
* Contain source
* Protect target account
* Review successful logons
* Review endpoint and network telemetry
* Preserve evidence

---

## Scenario 6: Failed RDP Authentication

Indicators:

* Logon Type `10`
* Remote source address
* Target endpoint accepts RDP
* Repeated username attempts
* Event may coincide with firewall or terminal-services logs

Response:

* Confirm RDP is authorized and required
* Validate source system
* Review successful RDP logons
* Restrict or disable unnecessary RDP exposure
* Escalate unknown-source activity

---

# Containment

## When Containment Is Not Required

Containment may not be necessary when:

* The activity is clearly authorized
* One isolated failure occurred
* User error is confirmed
* No account lockout or compromise occurred
* No suspicious follow-on activity exists

Document why containment was not required.

---

## Account Containment Options

Use only with appropriate authorization:

* Temporarily disable the account
* Force password reset
* Revoke active sessions
* Remove temporary privilege
* Unlock after root cause is addressed
* Require additional authentication review

Do not disable:

* Domain service accounts
* Recovery accounts
* Infrastructure accounts

without reviewing operational impact.

---

## Endpoint Containment Options

Depending on severity:

* Disconnect the source VM from the host-only adapter
* Disable the affected network adapter
* Stop the responsible service
* Disable the scheduled task
* Remove stale saved credentials
* End an unauthorized remote session
* Isolate the endpoint through approved controls

Preserve evidence before stopping processes where possible.

---

## Network Containment Options

* Block the source address
* Block the destination service temporarily
* Restrict RDP
* Restrict SMB
* Disable an unnecessary firewall rule
* Limit access to approved management hosts

Do not make broad firewall changes without rollback planning.

---

# Eradication

Eradication depends on the root cause.

## User Error

* No eradication required
* Confirm correct sign-in format
* Validate account health

## Stale Credential

* Update saved credential
* Update mapped drive
* Update service password
* Update scheduled task
* Remove obsolete credential

## Unauthorized Source

* Remove unauthorized access
* Review the source endpoint
* Reset affected credentials
* Remove persistence
* Review privilege changes
* Scan for malicious activity

## Compromised Account

* Disable or restrict account
* Reset credentials
* Revoke sessions
* Review MFA or additional controls
* Review successful authentication
* Review account changes
* Review data and resource access

---

# Recovery

## Account Recovery

Confirm:

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
    -Properties `
        Enabled,
        LockedOut,
        PasswordExpired,
        LastBadPasswordAttempt,
        BadLogonCount |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    PasswordExpired,
    LastBadPasswordAttempt,
    BadLogonCount
```

---

## Validate Successful Authentication

Perform a controlled successful sign-in only after:

* Root cause is addressed
* Account state is known
* Credential use is authorized
* No active attack is continuing

Record:

```
Validation time:
Source system:
Authentication method:
Successful:
Related Event ID 4624:
```

---

## Confirm Failures Stop

Splunk example:

```spl
index=windows EventCode=4625 earliest=-30m
| search "<TARGET_ACCOUNT>"
| timechart span=5m count
```

Wazuh:

* Search the target account
* Review the latest event time
* Confirm no continuing failure pattern

---

## Confirm Monitoring Health

Verify:

* Wazuh agent active
* Splunk forwarder active
* DC01 healthy
* Source endpoint healthy
* Time synchronization healthy
* New validation events are ingested

---

# Escalation Criteria

Escalate immediately when:

* A privileged account is targeted
* The account is confirmed compromised
* A successful logon follows repeated failures
* One source targets many accounts
* One account is targeted from many sources
* The source is external or unknown
* Account lockouts affect operations
* Activity crosses several systems
* Authentication is followed by privilege escalation
* Security controls are disabled
* Evidence suggests persistence or lateral movement
* The analyst cannot determine scope

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
First observed:
Last observed:
Target accounts:
Privileged accounts:
Source systems:
Source addresses:
Destination systems:
Event IDs:
Failure count:
Successful logons:
Lockout status:
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
* Event ID `4625`
* Related DC01 event
* Account state
* Source-system identity
* Wazuh alert
* Splunk timeline
* Failure-count search
* Successful-logon search
* Lockout event if present
* Containment actions
* Recovery validation
* Analyst notes

---

## Export Windows Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

Store raw logs privately.

---

## Export a Narrow Event Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624, 4625, 4740, 4767, 4771, 4776
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\failed-logon-summary.csv" `
    -NoTypeInformation
```

---

## Hash Evidence

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

---

## Evidence Record

```
Runbook: RB-01
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

# False Positives

Common benign causes include:

* Mistyped password
* Caps Lock or keyboard layout issue
* Wrong username format
* Local versus domain account confusion
* Recently changed password
* Cached credentials
* Mapped network drive
* Service account password mismatch
* Scheduled task
* Mobile device
* VPN client
* Remote Desktop saved credential
* Application reconnect
* Approved lab exercise
* Security scanner
* Help-desk testing

A false positive should still be documented when it triggered an alert.

---

# Detection Gaps

Document whether the event lacked:

* Target account
* Source address
* Workstation
* Logon type
* Failure reason
* Status
* Substatus
* Authentication package
* Process
* Account privilege
* Account state
* Correlation with domain-controller events
* Correlation with successful logon
* Threshold or frequency context
* Approved-source context

---

# Detection Improvement Recommendations

Potential improvements include:

* Normalize target-account fields
* Normalize source-address fields
* Correlate `4625`, `4771`, and `4776`
* Correlate failures followed by success
* Add privileged-account context
* Add approved-scanner lookup
* Detect one source targeting many accounts
* Detect one account targeted from many sources
* Add lockout correlation
* Add service-account context
* Add business-hours context
* Measure ingestion delay
* Build failed-logon dashboards
* Create account lockout and password-spray runbooks

---

# MITRE ATT&CK Context

Repeated failed authentication may relate to:

```
T1110 – Brute Force
```

Potential sub-techniques include:

* Password Guessing
* Password Cracking
* Password Spraying
* Credential Stuffing

A single failed logon does not establish ATT&CK behavior.

The mapping should be based on the observed pattern.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Target account:
Account privilege:
Account state:
Source system:
Source address:
Failure count:
Logon type:
Failure reason:
Related DC event:
Successful logon afterward:
Lockout:
Root cause:
Severity:
Containment:
Recovery:
Detection gaps:
```

---

## Root Cause Categories

Select one:

```
User typing error
Stale credential
Service account misconfiguration
Scheduled task
Mapped drive
Remote Desktop
Application authentication
Password spraying
Brute force
Unauthorized credential use
Approved test activity
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The target account is identified.
* The source is identified or documented as unknown.
* The failure reason is understood.
* The failure count is known.
* Related domain-controller events are reviewed.
* Successful logons are reviewed.
* Lockout state is known.
* Suspicious follow-on activity is excluded or escalated.
* Containment is completed where required.
* Authentication is restored where appropriate.
* Monitoring confirms the activity stopped.
* Evidence is preserved.
* Detection gaps are documented.
* The final classification is recorded.

---

## Closure Statement

```
The failed authentication activity was investigated using endpoint, domain-controller, Wazuh, and Splunk telemetry. The target account, source system, failure reason, event frequency, account state, and related successful authentication activity were reviewed. The activity was classified as <CLASSIFICATION>. Required containment and recovery actions were completed, monitoring confirmed that the activity stopped, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Real usernames
* Administrative identities
* Internal IP addresses
* Domain names
* Hostnames where sensitive
* SIDs
* Account group memberships
* Event XML
* Wazuh agent IDs
* Splunk URLs
* Evidence paths
* Authentication package details where sensitive
* User-reported personal information

Use placeholders such as:

```
<TARGET_ACCOUNT>
<SOURCE_SYSTEM>
<SOURCE_IP>
<WINDOWS_ENDPOINT>
<DOMAIN_CONTROLLER>
<DOMAIN_NAME>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4625`
* Related Event ID `4771` or `4776`
* Account state
* Wazuh failed-logon alert
* Splunk authentication timeline
* Failure-count search
* Account lockout state
* Recovery validation

Before publishing:

1. Crop unrelated information.
2. Remove usernames.
3. Remove internal addresses.
4. Remove operational domain names.
5. Remove SIDs.
6. Remove unrelated events.
7. Hide browser tabs and bookmarks.
8. Permanently redact sensitive values.
9. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Reporting host identified
* Target account identified
* Source identified
* Initial severity assigned

## Source Validation

* Event confirmed locally
* Event ID reviewed
* Logon type reviewed
* Failure reason reviewed
* Status and substatus reviewed
* Authentication package reviewed
* Process reviewed
* Source address reviewed

## Account Review

* Account enabled state confirmed
* Lockout state confirmed
* Password state reviewed
* Privilege reviewed
* Last bad password attempt reviewed
* Last successful logon reviewed

## Correlation

* Event ID 4771 reviewed
* Event ID 4776 reviewed
* Event ID 4740 reviewed
* Event ID 4624 reviewed
* Wazuh reviewed
* Splunk reviewed
* Failure count calculated
* Unique accounts calculated
* Unique sources calculated
* Success-after-failure reviewed

## Source Investigation

* Source hostname validated
* Source address validated
* Logged-on user reviewed
* Services reviewed where relevant
* Scheduled tasks reviewed where relevant
* Saved credentials reviewed where relevant
* Mapped drives reviewed where relevant

## Response

* Severity updated
* Containment decision recorded
* Account action completed where required
* Source containment completed where required
* Root cause addressed
* Authentication validated
* Failure activity stopped

## Closure

* Timeline completed
* Evidence preserved
* Hashes recorded
* Root cause documented
* False positive or incident classification recorded
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Primary Windows Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-1)
    }
```

## Primary Splunk Search

```spl
index=windows EventCode=4625 earliest=-1h
| stats
    count
    values(SourceNetworkAddress) as sources
    values(LogonType) as logon_types
    values(FailureReason) as reasons
    by TargetUserName
| sort - count
```

## Primary Account Check

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
    -Properties Enabled, LockedOut, LastBadPasswordAttempt, BadLogonCount
```

## Immediate Escalation Indicators

```
Privileged target
Unknown source
Many accounts targeted
Many source systems
Lockout
Success after failures
Remote administrative access
Privilege escalation
Continued activity
```

---

# Summary

This runbook provides a repeatable process for investigating failed Windows authentication.

A complete investigation should determine:

* Who or what attempted authentication
* Which account was targeted
* Which system generated the attempt
* Why the authentication failed
* How frequently the failures occurred
* Whether the account was locked
* Whether a successful logon followed
* Whether the activity was legitimate or suspicious
* Whether containment was required
* Whether authentication and monitoring returned to a healthy state

The investigation is complete only after the event is classified, required response actions are completed, evidence is preserved, monitoring confirms the activity stopped, and detection improvements are documented.

