# Failed Login Investigation

## Exercise Overview

This exercise generates a controlled failed authentication attempt and traces the resulting telemetry across:

* WIN11TARGET
* DC01
* Windows Event Viewer
* Wazuh
* Splunk
* Optional packet capture
* Investigation notes and evidence

The objective is to determine:

* Which account was targeted
* Which system initiated the attempt
* Why the authentication failed
* Which authentication path was used
* Whether the event was collected correctly
* Whether the SIEMs provided enough investigation context
* Whether the activity was isolated or part of a broader pattern

The exercise uses one or a very small number of failed attempts and should not intentionally trigger an account lockout.

---

## Exercise ID

```
EX-01
```

---

## Difficulty

```
Foundation
```

---

## Primary Skills

* Windows authentication analysis
* Failed logon investigation
* Event Viewer filtering
* Active Directory event review
* Wazuh alert validation
* Splunk SPL development
* Source-system identification
* Timeline reconstruction
* Evidence preservation
* False-positive analysis

---

## Authorization Boundary

This exercise must use:

* A disposable or designated non-administrative CyberLab account
* The monitored Windows endpoint
* The internal CyberLab domain
* The VMware host-only network
* Authorized lab credentials
* A known lockout policy

Do not use:

* A Domain Administrator account
* A personal account
* A school or employer account
* A recovery account
* A production service account
* Any system outside the CyberLab

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Domain: cyberlab.example
Test account: auth.test
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Generate a safe failed authentication event
* Confirm the event locally
* Identify the target account
* Identify the source workstation
* Interpret the failure reason
* Determine the logon type
* Compare endpoint and domain-controller telemetry
* Locate the event in Wazuh
* Locate the event in Splunk
* Measure ingestion delay
* Distinguish user error from suspicious activity
* Document detection gaps

---

## Required Systems

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on the monitored Windows systems
* Splunk Universal Forwarder where configured

KALI-TEST is not required for the standard exercise.

---

## Required Accounts

### Student User

Used for observation and routine access.

```
student.user
```

### Authorized Administrator

Used only for preparation or cleanup.

```
student.admin
```

### Test Account

Used for the controlled authentication failure.

```
auth.test
```

The test account must not be privileged.

---

## Required Monitoring

Before starting, confirm:

* WIN11TARGET Security log is active
* DC01 Security log is active
* Wazuh agent is active
* Wazuh dashboard is accessible
* Splunk is running
* Relevant Windows events are searchable
* System time is synchronized
* The account lockout policy is known

---

## Safety Notes

* Use only one failed attempt unless the exercise explicitly allows more.
* Do not continue until account lockout.
* Do not use a privileged account.
* Confirm the account is valid before testing.
* Record the exact attempt time.
* Preserve evidence before repeating the test.
* Stop immediately if another account is affected.
* Do not place a password in the documentation.

---

## Expected Event Flow

```
Incorrect Credential Entry
          |
          v
WIN11TARGET Authentication Attempt
          |
          v
Local Windows Security Event
          |
          v
DC01 Credential Validation or Kerberos Event
          |
          v
Wazuh and Splunk Ingestion
          |
          v
Student Investigation
```

The exact path depends on:

* Authentication method
* Logon type
* Domain state
* Cached credentials
* Protocol
* Audit configuration

---

## Key Windows Event IDs

| Event ID | Description                                         |
| -------- | --------------------------------------------------- |
| 4625     | An account failed to log on                         |
| 4771     | Kerberos pre-authentication failed                  |
| 4776     | Domain controller attempted to validate credentials |
| 4768     | Kerberos authentication ticket was requested        |
| 4624     | An account successfully logged on                   |

Not every event will appear in every test.

---

## Important Event Fields

Review:

* Target account
* Subject account
* Domain
* Workstation name
* Source network address
* Source port
* Logon type
* Authentication package
* Failure reason
* Status
* Substatus
* Process name
* Timestamp
* Reporting computer

---

## Investigation Questions

The final report should answer:

* Which account was targeted?
* Which system initiated the attempt?
* Which system recorded the failure?
* What was the failure reason?
* What was the logon type?
* Which authentication protocol was involved?
* Was the source address present?
* Did DC01 record a related event?
* Did Wazuh collect or alert on the event?
* Did Splunk index the event?
* Were timestamps aligned?
* Was the event isolated?
* Did any successful authentication follow?
* What legitimate activity could produce the same event?
* What additional context would improve the detection?

---

# Preparation

## Review the Lockout Policy

Run on DC01:

```
Get-ADDefaultDomainPasswordPolicy |
    Select-Object `
        LockoutThreshold,
        LockoutDuration,
        LockoutObservationWindow
```

Alternative:

```
net accounts /domain
```

Record:

```
Lockout threshold:
Lockout duration:
Observation window:
Maximum permitted exercise failures:
```

For this exercise, the permitted failure count should remain below the lockout threshold.

---

## Confirm System Time

On DC01 and WIN11TARGET:

```
Get-Date
```

```
w32tm /query /status
```

```
w32tm /query /source
```

On Wazuh and Splunk:

```
timedatectl
```

Record any time difference before proceeding.

---

## Confirm DC01 Health

Run as an administrator:

```
dcdiag
```

Review core services:

```
Get-Service `
    NTDS,
    DNS,
    Netlogon,
    Kdc
```

Expected state:

```
Running
```

---

## Confirm WIN11TARGET Domain Connectivity

```
Test-ComputerSecureChannel -Verbose
```

Expected result:

```
True
```

Confirm domain discovery:

```
nltest /dsgetdc:cyberlab.example
```

---

## Confirm Wazuh Agent Status

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm WIN11TARGET and DC01 are active in the dashboard where both are monitored.

---

## Confirm Splunk Forwarder Status

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm recent endpoint telemetry:

```spl
index=windows host=WIN11TARGET earliest=-15m
| head 20
```

Confirm recent domain-controller telemetry when configured:

```spl
index=windows host=DC01 earliest=-15m
| head 20
```

---

## Create or Validate the Test Account

Create the account through the approved Active Directory procedure when one does not already exist.

Example:

```
$Password = Read-Host `
    "Enter the temporary test password" `
    -AsSecureString

New-ADUser `
    -Name "Authentication Test" `
    -SamAccountName "auth.test" `
    -UserPrincipalName "auth.test@cyberlab.example" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $false
```

Do not place the password directly in the script.

---

## Validate the Test Account

```
Get-ADUser `
    -Identity "auth.test" `
    -Properties Enabled, LockedOut, PasswordExpired, MemberOf |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    PasswordExpired,
    MemberOf
```

Confirm:

* Account enabled
* Account not locked
* Password not expired
* Account not privileged

---

## Confirm a Successful Authentication

Before generating the failed attempt, authenticate once successfully with the test account.

Record the time.

This confirms:

* The username is correct
* The password is known
* The account is enabled
* Domain communication works
* Later failure is intentional

Sign out after validation.

---

## Confirm the Account Remains Unlocked

```
Get-ADUser `
    -Identity "auth.test" `
    -Properties LockedOut |
Select-Object `
    SamAccountName,
    LockedOut
```

Expected:

```
LockedOut: False
```

---

## Start the Exercise Record

```
Exercise ID: EX-01
Test account: auth.test
Source system: WIN11TARGET
Domain controller: DC01
Start time:
Lockout threshold:
Permitted failures:
Expected local event:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Generate One Failed Sign-In

Use an approved authentication prompt on WIN11TARGET.

Examples include:

* Windows sign-in screen
* Lock-screen sign-in
* Approved `runas` prompt
* Approved remote authentication prompt
* Access to a lab resource requiring domain credentials

Enter:

```
CYBERLAB\auth.test
```

Use an intentionally incorrect password once.

---

## Record the Attempt

Immediately record:

```
Attempt time:
Username format:
Source system:
Authentication method:
Observed message:
```

Do not repeat the attempt until the event is confirmed.

---

## Optional `runas` Method

An approved local test may use:

```
runas /user:CYBERLAB\auth.test notepad.exe
```

Enter an incorrect password at the prompt.

Do not place the password on the command line.

The resulting event details may differ from a console sign-in.

---

## Confirm the Account Is Not Locked

On DC01:

```
Get-ADUser `
    -Identity "auth.test" `
    -Properties LockedOut |
Select-Object `
    SamAccountName,
    LockedOut
```

Expected:

```
LockedOut: False
```

If the account is locked, stop and follow the account recovery procedure.

---

# WIN11TARGET Validation

## Search for Event ID 4625

Run as an administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Search for the Test Account

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "auth\.test"
}
```

---

## Event Viewer Procedure

1. Open **Event Viewer**.
2. Navigate to **Windows Logs**.
3. Select **Security**.
4. Choose **Filter Current Log**.
5. Enter event ID `4625`.
6. Set a narrow time range around the attempt.
7. Open the matching event.
8. Review the **General** and **Details** tabs.
9. Record the relevant fields.
10. Save a sanitized screenshot if required.

---

## Review Event ID 4625 Fields

Important sections commonly include:

### Subject

* Security ID
* Account name
* Account domain
* Logon ID

### Logon Type

* Numeric logon type
* Authentication context

### Account for Which Logon Failed

* Security ID
* Account name
* Account domain

### Failure Information

* Failure reason
* Status
* Substatus

### Process Information

* Caller process ID
* Caller process name

### Network Information

* Workstation name
* Source network address
* Source port

### Detailed Authentication Information

* Logon process
* Authentication package
* Transited services
* Package name

---

## Common Logon Types

| Logon Type | General Meaning    |
| ---------: | ------------------ |
|          2 | Interactive        |
|          3 | Network            |
|          4 | Batch              |
|          5 | Service            |
|          7 | Unlock             |
|          8 | Network cleartext  |
|          9 | New credentials    |
|         10 | Remote interactive |
|         11 | Cached interactive |

Interpret the logon type in the context of the exercise method.

---

## Record WIN11TARGET Findings

```
Event ID:
Event time:
Reporting host:
Target account:
Target domain:
Logon type:
Failure reason:
Status:
Substatus:
Workstation:
Source address:
Source port:
Process:
Authentication package:
```

---

# DC01 Validation

## Search for Credential Validation Event 4776

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4776
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "auth\.test"
}
```

Review:

* Account
* Source workstation
* Status code
* Timestamp
* Reporting domain controller

---

## Search for Kerberos Pre-Authentication Failure 4771

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4771
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "auth\.test"
}
```

Review:

* Account
* Client address
* Failure code
* Ticket options
* Pre-authentication type
* Timestamp

---

## Search for Ticket Request Event 4768

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4768
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "auth\.test"
}
```

This event may provide additional Kerberos context.

---

## Record DC01 Findings

```
Event IDs present:
Event time:
Target account:
Source workstation:
Source address:
Status or failure code:
Authentication protocol:
Reporting domain controller:
```

---

## Confirm No Lockout Event Occurred

Search for Event ID `4740`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "auth\.test"
}
```

Expected result:

```
No matching lockout event
```

---

# Wazuh Validation

## Confirm Agent Health

In the Wazuh dashboard, confirm:

* WIN11TARGET is active
* DC01 is active when monitored
* Last keepalive is recent
* No agent identity changed
* The selected time range includes the exercise

---

## Search by Test Account

Search for:

```
auth.test
```

Use a narrow time range around the attempt.

---

## Search by Event ID

Review events associated with:

```
4625
4771
4776
```

The available fields depend on Wazuh decoding and collection configuration.

---

## Review Wazuh Fields

Record:

* Agent name
* Windows event ID
* Rule description
* Rule level
* Target account
* Source workstation
* Source address
* Logon type
* Failure reason
* Authentication package
* Timestamp
* MITRE mapping
* Compliance tags

---

## Wazuh Validation Questions

* Was the endpoint failure collected?
* Was the domain-controller event collected?
* Did Wazuh create an alert?
* Was the target account parsed correctly?
* Was the source workstation visible?
* Was the failure reason understandable?
* Was the rule severity appropriate?
* Did the event appear promptly?
* Did Wazuh identify the authentication method?

---

## Wazuh Detection Gap Record

When the event is collected but context is incomplete:

```
Telemetry present:
Alert present:
Rule:
Rule level:
Missing account context:
Missing source context:
Missing failure context:
Recommended improvement:
```

---

# Splunk Validation

## Search for the Test Account

```spl
index=windows "auth.test" earliest=-30m
| table _time host source sourcetype EventCode user Message
| sort _time
```

---

## Search for Event ID 4625

```spl
index=windows EventCode=4625 earliest=-30m
| search "auth.test"
| table
    _time
    host
    user
    src_ip
    src_port
    LogonType
    FailureReason
    Status
    SubStatus
    AuthenticationPackageName
| sort _time
```

Field names depend on the sourcetype and installed add-ons.

---

## Search for Event ID 4776

```spl
index=windows EventCode=4776 earliest=-30m
| search "auth.test"
| table
    _time
    host
    user
    Workstation
    Status
| sort _time
```

---

## Search for Event ID 4771

```spl
index=windows EventCode=4771 earliest=-30m
| search "auth.test"
| table
    _time
    host
    user
    src_ip
    Status
    FailureCode
| sort _time
```

---

## Build a Unified Timeline

```spl
index=windows earliest=-30m
(
    EventCode=4625 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "auth.test"
| table
    _time
    host
    EventCode
    user
    src_ip
    Workstation
    LogonType
    FailureReason
    Status
    Message
| sort _time
```

---

## Compare Failed and Successful Authentication

```spl
index=windows earliest=-30m
(
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "auth.test"
| eval result=case(
    EventCode=4624, "Success",
    EventCode=4625, "Failure",
    EventCode=4771, "Kerberos failure",
    EventCode=4776, "Credential validation"
)
| table _time host EventCode result user src_ip LogonType
| sort _time
```

This can confirm the successful pre-test authentication and failed test attempt.

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
| search "auth.test"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    EventCode
| sort _time
```

Record:

```
Minimum delay:
Average delay:
Maximum delay:
```

---

## Check for Duplicate Events

```spl
index=windows earliest=-30m
| search "auth.test"
| stats count by _time host source EventCode _raw
| where count > 1
```

Similar endpoint and DC01 events should not automatically be classified as duplicates.

---

## Splunk Validation Questions

* Which host recorded the failed logon?
* Which host recorded credential validation?
* Was the target account extracted?
* Was the source address extracted?
* Was the logon type available?
* Was the failure reason available?
* Did the event arrive promptly?
* Did the successful baseline event also appear?
* Were any duplicate inputs present?
* Could the search distinguish one failure from repeated failures?

---

# Optional Packet Capture

Packet capture is optional.

It may help confirm:

* Communication between WIN11TARGET and DC01
* Source and destination addresses
* DNS lookup
* Kerberos traffic
* Authentication timing

It should not replace Windows event analysis.

---

## Capture Example

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <WINDOWS_ENDPOINT> \
    and host <DOMAIN_CONTROLLER> \
    -w <CAPTURE_FILE>.pcap
```

Stop the capture immediately after the test.

---

## Useful Wireshark Filters

```
kerberos
```

```
dns
```

```
ip.addr == <WINDOWS_ENDPOINT> && ip.addr == <DOMAIN_CONTROLLER>
```

Authentication contents may be encrypted even when the network exchange is visible.

---

# Investigation

## Build the Timeline

| Time     | System      |       Event ID | Account   | Source              | Analyst Note                       |
| -------- | ----------- | -------------: | --------- | ------------------- | ---------------------------------- |
| `<TIME>` | WIN11TARGET |           4624 | auth.test | WIN11TARGET         | Successful baseline authentication |
| `<TIME>` | WIN11TARGET |           4625 | auth.test | WIN11TARGET         | Controlled failed authentication   |
| `<TIME>` | DC01        |   4771 or 4776 | auth.test | WIN11TARGET         | Domain validation failure          |
| `<TIME>` | Wazuh       | Alert or event | auth.test | WIN11TARGET         | Telemetry collected                |
| `<TIME>` | Splunk      |  Indexed event | auth.test | WIN11TARGET or DC01 | Event searchable                   |

---

## Root Cause

For the controlled exercise:

```
Authorized single failed authentication attempt using an intentionally incorrect password for a nonprivileged CyberLab test account.
```

---

## Common Real-World Causes

A failed authentication may result from:

* Mistyped password
* Wrong username format
* Expired password
* Disabled account
* Locked account
* Stale saved credential
* Scheduled task
* Windows service
* Mapped drive
* Mobile application
* VPN client
* Remote Desktop attempt
* Password spray
* Brute force
* Compromised endpoint

---

## Distinguish Benign from Suspicious Activity

Consider:

* Number of failures
* Time between failures
* Number of targeted accounts
* Source-system reputation
* Target-account privilege
* Time of day
* Geographic or network context
* Successful authentication afterward
* Related process activity
* Related account changes
* Whether the user recognizes the activity

One isolated failure is usually weaker evidence than a repeated pattern.

---

## Failure Reason Analysis

The failure reason should be compared with:

* Status code
* Substatus code
* Account state
* Authentication protocol
* Test action

Possible categories include:

* Unknown username
* Bad password
* Disabled account
* Expired account
* Locked account
* Logon restriction
* Time restriction
* Workstation restriction

Do not rely on a single human-readable field when status codes are available.

---

## False-Positive Analysis

Legitimate causes include:

* User typing error
* Old saved password
* Service retry
* Scheduled task
* Mapped network drive
* Remote administration
* Application reconnect
* Password recently changed
* Cached credential
* Test or support activity

A detection should include repetition, context, or account sensitivity when stronger confidence is required.

---

# Detection Development

## Basic Splunk Search

```spl
index=windows EventCode=4625 earliest=-15m
| stats count by user host src_ip LogonType FailureReason
| sort - count
```

---

## Repeated Failure Detection

```spl
index=windows EventCode=4625 earliest=-15m
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as failed_attempts
    values(host) as reporting_hosts
    values(src_ip) as source_addresses
    values(LogonType) as logon_types
    values(FailureReason) as failure_reasons
    min(_time) as first_seen
    max(_time) as last_seen
    by target_account
| where failed_attempts >= <THRESHOLD>
| convert ctime(first_seen) ctime(last_seen)
```

The threshold should be based on the environment and lockout policy.

---

## One Source Targeting Multiple Accounts

```spl
index=windows EventCode=4625 earliest=-15m
| eval target_account=coalesce(TargetUserName, user)
| stats
    dc(target_account) as unique_accounts
    values(target_account) as targeted_accounts
    count as failed_attempts
    by src_ip
| where unique_accounts >= <ACCOUNT_THRESHOLD>
```

This may provide password-spray context.

---

## One Account Targeted from Multiple Sources

```spl
index=windows EventCode=4625 earliest=-15m
| eval target_account=coalesce(TargetUserName, user)
| stats
    dc(src_ip) as unique_sources
    values(src_ip) as source_addresses
    count as failed_attempts
    by target_account
| where unique_sources >= <SOURCE_THRESHOLD>
```

---

## Wazuh Rule Concept

A conceptual custom rule may identify a failed Windows logon:

```xml
<group name="windows,authentication,failed_logon,">
  <rule id="<CUSTOM_RULE_ID>" level="5">
    <field name="win.system.eventID">4625</field>
    <description>Windows failed authentication detected</description>
  </rule>
</group>
```

A higher-confidence rule should use correlation or frequency rather than treating one ordinary failure as critical.

---

## MITRE ATT&CK Context

Repeated failed authentication may relate to:

```
T1110 – Brute Force
```

Possible sub-techniques depend on the pattern:

* Password Guessing
* Password Cracking
* Password Spraying
* Credential Stuffing

A single failed logon does not establish brute-force behavior.

---

## Detection Severity Guidance

| Pattern                                | Suggested Context           |
| -------------------------------------- | --------------------------- |
| One failure from known endpoint        | Informational or Low        |
| Several failures for one account       | Low or Medium               |
| One source targeting many accounts     | Medium or High              |
| Privileged account targeted repeatedly | High                        |
| Failures followed by successful access | High investigation priority |
| External or unexpected source          | Increased severity          |
| Multiple systems involved              | Increased severity          |

Severity should be based on context and confidence.

---

# Evidence Collection

## Evidence

Preserve:

* Domain lockout policy
* Successful baseline authentication
* Event ID `4625`
* Relevant DC01 event
* Wazuh event or alert
* Splunk timeline
* Account unlocked state
* Successful post-test validation
* Investigation notes
* Detection-gap notes

---

## Export a Narrow Event Log

A filtered export is preferable to publishing the full Security log.

Operational example:

```
wevtutil epl Security "<EVIDENCE_PATH>\WIN11TARGET-Security.evtx"
```

Store raw exports privately.

---

## Export Splunk Results

Export the narrow search as:

* CSV
* JSON
* Raw events

Review every field before publication.

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
Exercise: EX-01
File:
Source:
Collection time:
Description:
SHA-256:
Student analyst:
Storage location:
```

---

# Cleanup and Validation

## Confirm the Account Is Not Locked

```
Get-ADUser `
    -Identity "auth.test" `
    -Properties LockedOut, Enabled |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut
```

Expected:

```
Enabled: True
LockedOut: False
```

---

## Confirm Successful Authentication

Authenticate once using the correct credentials.

Record the time.

Confirm:

* Account access works
* Event ID `4624` appears where expected
* Wazuh remains connected
* Splunk receives the successful event
* No account lockout occurred

---

## Disable or Remove the Test Account

When the account was created only for this exercise, disable or delete it after evidence collection.

Disable:

```
Disable-ADAccount `
    -Identity "auth.test"
```

Delete:

```
Remove-ADUser `
    -Identity "auth.test" `
    -Confirm
```

Deletion is permanent unless recovered from backup.

---

## Final State Record

```
Failed attempt completed:
Account remained unlocked:
Successful authentication confirmed:
Wazuh healthy:
Splunk healthy:
Evidence preserved:
Test account disabled or deleted:
Cleanup complete:
```

---

# Validation Criteria

The exercise is successfully validated when:

* The lockout policy was reviewed.
* A nonprivileged test account was used.
* Successful authentication was confirmed first.
* One controlled failed attempt was generated.
* Event ID `4625` was confirmed.
* A related DC01 event was reviewed where present.
* The account remained unlocked.
* Wazuh was reviewed.
* Splunk was reviewed.
* A timeline was created.
* Ingestion delay was measured where possible.
* False positives were considered.
* Cleanup was completed.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                  | Result                     |
| ------------------------- | -------------------------- |
| Local event present       | Pass / Fail                |
| DC01 event present        | Pass / Not Expected / Fail |
| Wazuh ingestion           | Pass / Fail                |
| Wazuh alert quality       | Pass / Needs Tuning / Fail |
| Splunk ingestion          | Pass / Fail                |
| Splunk field quality      | Pass / Needs Tuning / Fail |
| Source visibility         | Pass / Partial / Fail      |
| Failure reason visibility | Pass / Partial / Fail      |
| Timestamp accuracy        | Pass / Needs Review / Fail |
| Account remained unlocked | Pass / Fail                |
| Cleanup                   | Complete / Incomplete      |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Source address
* Workstation name
* Logon type
* Failure reason
* Status or substatus
* Authentication package
* Process name
* User-friendly account field
* Endpoint and DC correlation
* Privileged-account context
* Baseline comparison

---

## Improvements

Potential improvements include:

* Add a saved failed-logon search
* Correlate endpoint and DC events
* Add repeated-failure thresholds
* Add privileged-account context
* Track one source targeting many users
* Track one user targeted from many sources
* Improve Windows field extraction
* Add ingestion-delay monitoring
* Create a failed-logon dashboard
* Create a failed-authentication response runbook

---

# Troubleshooting

## No Event ID 4625 Appears

Check:

* Correct event channel
* Audit policy
* Test method
* Local versus domain authentication
* Event time range
* Security log retention
* Administrative access to the log

Review:

```
auditpol /get /subcategory:"Logon"
```

```
Get-WinEvent -ListLog Security
```

---

## DC01 Shows No Related Event

Possible explanations include:

* Cached authentication
* Local account used accidentally
* Authentication path did not reach DC01
* Event ID differs by protocol
* DC01 auditing is incomplete
* Wrong domain-controller search window

Confirm the username format and domain state.

---

## Account Locks Unexpectedly

Stop the exercise.

Then:

1. Preserve the lockout evidence.
2. Confirm the lockout policy.
3. Unlock only the test account.
4. Confirm no other account is locked.
5. Reduce future attempt count.
6. Review whether background services used the same credentials.

---

## Failure Reason Is Missing

Check:

* Raw event XML
* Event Details view
* Splunk sourcetype
* Wazuh decoder
* Add-on support
* Field extraction

Do not create a permanent extraction from one event sample without broader testing.

---

## Wazuh Does Not Show the Event

Check:

* Agent status
* Security log collection
* Agent configuration
* Decoder
* Rule level
* Dashboard time range
* Agent filter
* Indexer health

Confirm the source event exists locally first.

---

## Splunk Does Not Show the Event

Check:

* Universal Forwarder service
* Security input
* Receiver listener
* Target index
* Host field
* Sourcetype
* Time range
* Event field name
* Forwarder logs

Begin with:

```spl
index=windows earliest=-30m
| stats count by host EventCode
```

---

## Source Address Is Blank or Local

This can occur depending on:

* Logon type
* Authentication method
* Local sign-in
* Cached sign-in
* Event structure
* Field extraction

Use workstation, process, and DC events for additional context.

---

## Too Many Matching Events

Possible causes include:

* Normal background failures
* Both endpoint and DC records
* Duplicate collection
* Service accounts
* Scheduled tasks
* Wrong time range
* Broad account search

Narrow by:

* Exact time
* Test account
* Host
* Event ID
* Workstation
* Source address

---

# Public Sanitization

Remove or replace:

* Operational domain name
* Real usernames
* Internal IP addresses
* Administrator identities
* Security identifiers
* Exact workstation names where sensitive
* Status details tied to live systems
* Raw event XML
* Splunk internal URLs
* Wazuh agent identifiers
* Evidence paths
* Packet captures

Use placeholders such as:

```
<TEST_ACCOUNT>
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<DOMAIN_NAME>
<EVENT_ID>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4625`
* Related DC01 authentication event
* Wazuh failed-logon alert
* Splunk failed-logon timeline
* Successful cleanup validation
* Detection-quality table

Before publishing:

1. Crop unrelated content.
2. Remove operational addresses.
3. Replace real usernames.
4. Remove security identifiers.
5. Hide browser tabs and bookmarks.
6. Permanently redact sensitive values.
7. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
A controlled failed authentication attempt was generated against a nonprivileged CyberLab test account. The resulting endpoint and domain authentication telemetry was reviewed locally and correlated through Wazuh and Splunk. The account remained unlocked, the event was classified as authorized test activity, and detection gaps were documented.
```

## Findings

```
Test account:

Source system:

Attempt time:

Authentication method:

Endpoint event:

Domain-controller event:

Logon type:

Failure reason:

Wazuh result:

Splunk result:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized use of an intentionally incorrect password during CyberLab Exercise EX-01.
```

## Resolution

```
The account was confirmed unlocked, correct authentication was validated after evidence collection, and the temporary test account was disabled or removed according to the exercise plan.
```

## Recommendations

```
- Improve failed-logon field extraction.
- Add repeated-failure correlation.
- Add source-system and account-privilege context.
- Create a failed-authentication dashboard.
- Develop a failed-logon investigation runbook.
```

---

# Completion Checklist

## Preparation

* Lockout policy reviewed
* Maximum attempt count defined
* Test account nonprivileged
* Test account valid
* Test account unlocked
* Successful baseline authentication confirmed
* DC01 healthy
* WIN11TARGET healthy
* Wazuh healthy
* Splunk healthy
* Start time recorded

## Exercise

* One controlled failure generated
* Attempt time recorded
* No unnecessary retries
* Account remained unlocked

## Investigation

* Event ID `4625` reviewed
* DC01 events reviewed
* Logon type identified
* Failure reason identified
* Source system identified
* Wazuh reviewed
* Splunk reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Successful authentication confirmed
* No lockout event occurred
* Test account disabled or deleted when required
* Monitoring systems healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Windows authentication analysis
* Event ID `4625` interpretation
* Active Directory event review
* Event Viewer filtering
* Wazuh validation
* Splunk SPL
* Authentication timeline reconstruction
* Failure-reason analysis
* False-positive assessment
* Evidence handling
* Detection tuning
* Account-state validation
* Public sanitization

---

# Lessons Learned

* A lockout policy should be reviewed before testing authentication failures.
* A successful baseline authentication proves the test account is valid.
* One failed logon can produce events on both the endpoint and domain controller.
* Event ID `4625` provides valuable failure, process, network, and logon-type context.
* DC01 events can clarify the authentication protocol and source workstation.
* A single failed logon does not establish brute-force behavior.
* Wazuh may collect the event without correlating it with related domain events.
* Splunk can combine endpoint and domain-controller events into one timeline.
* Missing source fields may require correlation rather than immediate parser changes.
* Cleanup should confirm both account access and monitoring health.
* Detection engineering should distinguish ordinary user mistakes from repeated suspicious behavior.

---

# Summary

This exercise validates the CyberLab’s ability to observe and investigate a controlled failed authentication attempt.

A complete investigation should identify:

* The target account
* The source system
* The authentication method
* The failure reason
* The logon type
* The related domain event
* The Wazuh result
* The Splunk result
* The event timeline
* The likely cause
* Any missing telemetry

The exercise is complete only after the account remains operational, evidence is preserved, and detection improvements are documented.

