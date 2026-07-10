# Account Lockout Investigation

## Exercise Overview

This exercise generates a controlled Active Directory account lockout and validates the resulting telemetry across:

* Windows Event Viewer
* Domain controller security logs
* Wazuh
* Splunk
* Optional packet capture
* Investigation notes and evidence

The objective is to practice identifying:

* Which account was locked
* Which system generated the failed attempts
* How many failures occurred
* When the failures began
* Whether the activity was expected
* Whether the SIEMs provided enough context
* Whether the account was restored safely

All activity must remain inside the authorized CyberLab.

---

## Exercise ID

```
EX-02
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Active Directory account monitoring
* Windows authentication analysis
* Account lockout investigation
* Event correlation
* Wazuh alert review
* Splunk SPL development
* Timeline reconstruction
* Evidence preservation
* Recovery and cleanup

---

## Authorization Boundary

This exercise must use:

* A disposable non-administrative test account
* The CyberLab domain controller
* The monitored Windows endpoint
* The internal VMware host-only network
* Authorized lab credentials
* A documented lockout policy

Do not use:

* A Domain Administrator account
* The primary endpoint administrator
* A personal account
* A school or employer account
* An account needed for system recovery
* Any system outside the CyberLab

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Domain: cyberlab.example
Test account: lockout.test
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Review the effective domain lockout policy
* Safely generate a controlled lockout
* Identify failed authentication events
* Identify the account lockout event
* Determine the source of the failures
* Correlate events across DC01, Wazuh, and Splunk
* Distinguish an account lockout from a disabled or expired account
* Restore access without weakening the policy
* Document findings and detection gaps

---

## Required Systems

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on the monitored Windows systems
* Splunk Universal Forwarder where configured

KALI-TEST is not required for the standard version of this exercise.

---

## Required Accounts

Use three separate identities.

### Student User

Used for routine access and observation.

```
student.user
```

### Authorized Administrator

Used only for account creation, unlock, and cleanup.

```
student.admin
```

### Disposable Test Account

Used only for the exercise.

```
lockout.test
```

The disposable account must not be a member of privileged groups.

---

## Required Monitoring

Confirm the following before starting:

* DC01 Security log is active
* WIN11TARGET Security log is active
* Wazuh agent is active
* Wazuh dashboard is accessible
* Splunk is running
* The expected Windows index is searchable
* System time is synchronized
* The selected dashboard time range is understood

---

## Safety Notes

* Review the lockout threshold before generating failures.
* Use only the disposable test account.
* Confirm a separate administrator account remains available.
* Do not change the domain lockout policy merely to complete the exercise.
* Stop immediately once the account locks.
* Do not repeatedly test after the lockout event is confirmed.
* Preserve the evidence before unlocking the account.
* Confirm the account is restored during cleanup.

---

## Expected Event Sequence

A typical event sequence is:

```
Failed authentication attempt
        |
        v
Additional failed attempts
        |
        v
Lockout threshold reached
        |
        v
Account locked by Active Directory
        |
        v
Lockout event recorded on DC01
        |
        v
Wazuh and Splunk ingest the event
        |
        v
Student correlates source, account, and timeline
```

---

## Key Windows Event IDs

The exact events depend on the authentication path and audit configuration.

| Event ID | Description                                         |
| -------- | --------------------------------------------------- |
| 4625     | An account failed to log on                         |
| 4740     | A user account was locked out                       |
| 4771     | Kerberos pre-authentication failed                  |
| 4776     | Domain controller attempted to validate credentials |
| 4768     | Kerberos authentication ticket was requested        |
| 4624     | An account successfully logged on                   |
| 4767     | A user account was unlocked                         |

Not every environment will record every event for a single test.

---

## Important Event Fields

Review the following fields where available:

* Target account
* Subject account
* Caller computer
* Workstation name
* Source network address
* Logon type
* Authentication package
* Failure reason
* Status
* Substatus
* Domain controller
* Event timestamp

---

## Investigation Questions

The final report should answer:

* Which account was locked?
* What time did the failures begin?
* What time was the lockout recorded?
* Which system generated the attempts?
* How many failed attempts occurred?
* Which authentication protocol was involved?
* What failure reason was recorded?
* Did WIN11TARGET and DC01 show related events?
* Did Wazuh alert on the activity?
* Did Splunk index the events?
* Were all timestamps aligned?
* Was the account unlocked successfully?
* What legitimate activity could create a similar pattern?
* What detection or logging improvements are needed?

---

# Preparation

## Review the Current Snapshot State

Confirm that stable snapshots exist for:

* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER

A new snapshot is not always required for this exercise, but one should exist before changing identity configuration.

---

## Confirm System Time

On DC01 and WIN11TARGET:

```
Get-Date
```

Review time synchronization:

```
w32tm /query /status
```

Review the configured source:

```
w32tm /query /source
```

On Wazuh and Splunk:

```
timedatectl
```

Record any offset before proceeding.

---

## Confirm DC01 Health

Run on DC01 as an administrator:

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

Run on WIN11TARGET:

```
Test-ComputerSecureChannel -Verbose
```

Expected result:

```
True
```

Confirm the domain controller can be discovered:

```
nltest /dsgetdc:cyberlab.example
```

---

## Confirm Wazuh Agent Status

On WIN11TARGET:

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm the agent is active in the Wazuh dashboard.

---

## Confirm Splunk Forwarder Status

On WIN11TARGET:

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm that recent endpoint events are searchable.

Example:

```spl
index=windows host=WIN11TARGET earliest=-15m
| head 20
```

---

## Review the Domain Lockout Policy

Run on DC01:

```
Get-ADDefaultDomainPasswordPolicy |
    Select-Object `
        LockoutThreshold,
        LockoutDuration,
        LockoutObservationWindow
```

Record:

```
Lockout threshold:
Lockout duration:
Observation window:
```

Do not proceed until the threshold is known.

---

## Alternative Policy Review

The policy may also be reviewed with:

```
net accounts /domain
```

Confirm the result matches the Active Directory PowerShell output.

---

## Create the Disposable Test Account

Run on DC01 using an authorized administrator account.

Create the account through Active Directory Users and Computers or approved PowerShell administration.

Example:

```
$Password = Read-Host `
    "Enter the temporary test password" `
    -AsSecureString

New-ADUser `
    -Name "Lockout Test" `
    -SamAccountName "lockout.test" `
    -UserPrincipalName "lockout.test@cyberlab.example" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $false
```

Do not place the password directly in the script or documentation.

---

## Verify the Test Account

```
Get-ADUser `
    -Identity "lockout.test" `
    -Properties Enabled, LockedOut, MemberOf |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    MemberOf
```

Confirm:

* The account is enabled.
* The account is not locked.
* The account is not privileged.
* The account belongs only to expected groups.

---

## Confirm a Successful Authentication First

Before generating failures, confirm that the credentials are valid.

Use the test account for one approved successful sign-in or authentication.

Record the time.

This proves that later failures are not caused by:

* Wrong initial password
* Disabled account
* Expired account
* Incorrect username format
* Domain connectivity failure

Sign out after validation.

---

## Confirm the Account Is Not Locked

On DC01:

```
Get-ADUser `
    -Identity "lockout.test" `
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

Record:

```
Exercise ID: EX-02
Test account: lockout.test
Source system: WIN11TARGET
Domain controller: DC01
Start time:
Lockout threshold:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Generate Controlled Failures

Use the WIN11TARGET sign-in screen or another approved Windows authentication prompt.

Enter:

```
CYBERLAB\lockout.test
```

Use an intentionally incorrect password.

Repeat only until the documented threshold is reached.

Do not exceed the threshold unnecessarily.

---

## Stop When Lockout Is Confirmed

Once Windows reports that the account is locked or the threshold has been reached:

1. Stop entering credentials.
2. Record the exact time.
3. Do not unlock the account yet.
4. Begin evidence collection.
5. Confirm that the test did not affect any other account.

---

# Local Windows Validation

## Review WIN11TARGET Security Events

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

Review:

* Target account
* Failure reason
* Logon type
* Source system
* Timestamp

---

## Review Failed Logons in Event Viewer

Open:

```
Event Viewer
└── Windows Logs
    └── Security
```

Filter the current log for:

```
4625
```

Use a narrow time window around the exercise.

---

## Record WIN11TARGET Findings

```
First failed attempt:
Last failed attempt:
Number of local failures:
Logon type:
Failure reason:
Source address:
Workstation:
```

---

# Domain Controller Validation

## Find the Account Lockout Event

Run on DC01:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Review:

* Account name
* Caller computer
* Domain controller
* Timestamp

---

## Search for the Specific Test Account

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "lockout\.test"
}
```

---

## Review Credential Validation Events

Search for Event ID `4776`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4776
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "lockout\.test"
}
```

---

## Review Kerberos Failures

Search for Event ID `4771`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4771
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "lockout\.test"
}
```

The relevant event depends on how the authentication attempt was performed.

---

## Confirm the Locked State

```
Get-ADUser `
    -Identity "lockout.test" `
    -Properties LockedOut, Enabled |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut
```

Expected:

```
Enabled: True
LockedOut: True
```

---

## Find All Locked Accounts

```
Search-ADAccount -LockedOut |
    Select-Object `
        Name,
        SamAccountName,
        DistinguishedName
```

Confirm that only the intended test account is affected.

---

# Wazuh Validation

## Confirm the Agent Is Active

In the Wazuh dashboard, confirm that:

* DC01 is active if monitored
* WIN11TARGET is active
* The last keepalive is recent
* No agent identity changed during the exercise

---

## Search by Account

Use the Wazuh dashboard search and filters to locate:

```
lockout.test
```

Use a narrow time range around the exercise.

---

## Search by Event ID

Review events associated with:

```
4625
4740
4771
4776
```

The available fields depend on the Wazuh decoder and Windows-event configuration.

---

## Review Wazuh Fields

Record:

* Agent
* Event ID
* Rule description
* Rule level
* Account
* Source system
* Caller computer
* Failure reason
* Timestamp
* MITRE ATT&CK mapping
* Compliance mappings

---

## Wazuh Validation Questions

* Did Wazuh collect the failed logons?
* Did Wazuh collect the lockout event?
* Did it create an alert?
* Was the account field parsed correctly?
* Was the source workstation visible?
* Was the rule severity appropriate?
* Did the event appear within the expected time range?
* Was additional custom logic required?

---

## Wazuh Detection Gap

If the event is collected but no useful alert appears, document:

```
Telemetry present: Yes
Alert present: No
Decoder result:
Matching rule:
Missing context:
ed custom rule:
```

Do not treat missing default alerting as proof of ingestion failure.

---

# Splunk Validation

## Search for the Test Account

```spl
index=windows "lockout.test" earliest=-30m
| table _time host source sourcetype EventCode user Message
| sort _time
```

Field names may differ depending on the installed Windows add-on.

---

## Search for Failed Logons

```spl
index=windows EventCode=4625 earliest=-30m
| search "lockout.test"
| table _time host user src_ip LogonType FailureReason
| sort _time
```

---

## Search for Account Lockout

```spl
index=windows EventCode=4740 earliest=-30m
| search "lockout.test"
| table _time host SubjectUserName TargetUserName CallerComputerName
| sort _time
```

---

## Search for Credential Validation

```spl
index=windows EventCode=4776 earliest=-30m
| search "lockout.test"
| table _time host user Workstation Status
| sort _time
```

---

## Search for Kerberos Pre-Authentication Failures

```spl
index=windows EventCode=4771 earliest=-30m
| search "lockout.test"
| table _time host user src_ip Status FailureCode
| sort _time
```

---

## Build an Event Timeline

```spl
index=windows earliest=-30m
(
    EventCode=4625 OR
    EventCode=4740 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "lockout.test"
| table _time host EventCode user src_ip CallerComputerName FailureReason Message
| sort _time
```

---

## Count the Failed Attempts

```spl
index=windows EventCode=4625 earliest=-30m
| search "lockout.test"
| stats count as failed_attempts by host user
```

The observed count may differ depending on:

* Authentication protocol
* Duplicate collection
* Endpoint versus domain-controller events
* Event filtering
* User field extraction
* Search scope

---

## Compare Event and Index Time

```spl
index=windows earliest=-30m
| search "lockout.test"
| eval ingestion_delay_seconds=_indextime-_time
| table _time _indextime ingestion_delay_seconds host EventCode
| sort _time
```

Record the largest observed delay.

---

## Splunk Validation Questions

* Which index contained the events?
* Which host recorded Event ID `4740`?
* Was the source workstation parsed?
* Were the failed attempts and lockout correlated?
* Did the host field accurately identify DC01 and WIN11TARGET?
* Was ingestion timely?
* Were duplicate events present?
* Were important fields missing?
* Could a saved search detect the pattern reliably?

---

# Optional Packet Capture

Packet capture is optional because Windows logs usually provide stronger evidence for this exercise.

It may be used to confirm:

* Communication between WIN11TARGET and DC01
* Kerberos or authentication-related traffic
* Source and destination timing

Do not use packet capture as a substitute for Windows security logs.

---

## Capture Filter

From an authorized capture system:

```
sudo tcpdump \
    -i <HOST_ONLY_INTERFACE> \
    host <WINDOWS_ENDPOINT> \
    and host <DOMAIN_CONTROLLER> \
    -w <CAPTURE_FILE>.pcap
```

Stop the capture immediately after the lockout.

---

## Packet Capture Security

The capture may contain:

* Internal addresses
* Domain names
* Account identifiers
* Authentication metadata
* Service information

Do not upload the raw capture to GitHub.

---

# Investigation

## Build the Timeline

Use a table such as:

| Time     | System      |      Event ID | Account      | Source      | Analyst Note                 |
| -------- | ----------- | ------------: | ------------ | ----------- | ---------------------------- |
| `<TIME>` | WIN11TARGET |          4625 | lockout.test | WIN11TARGET | First failed attempt         |
| `<TIME>` | DC01        |          4776 | lockout.test | WIN11TARGET | Credential validation failed |
| `<TIME>` | DC01        |          4740 | lockout.test | WIN11TARGET | Account locked               |
| `<TIME>` | Wazuh       |         Alert | lockout.test | WIN11TARGET | Lockout telemetry received   |
| `<TIME>` | Splunk      | Indexed event | lockout.test | DC01        | Event searchable             |

Use sanitized times and values in public examples.

---

## Determine the Root Cause

For this controlled exercise, the root cause should be:

```
Authorized repeated authentication failures against a disposable CyberLab test account.
```

In a real environment, possible causes could include:

* User typing the wrong password
* Stale stored credentials
* Mapped drive
* Scheduled task
* Windows service
* Mobile email client
* VPN client
* Script
* Password spray
* Brute-force activity
* Compromised endpoint

---

## Distinguish User Error from Suspicious Activity

Consider:

* Number of attempts
* Frequency
* Source systems
* Time of day
* Target account privilege
* Whether multiple users were targeted
* Whether the source is known
* Whether successful authentication followed
* Whether the source continued after lockout
* Whether other suspicious events occurred nearby

---

## False-Positive Analysis

Legitimate causes may include:

* Recently changed password
* Cached credential
* User typo
* Service account password not updated
* Scheduled task using an old password
* Disconnected mapped drive
* Mobile device reconnecting
* Application pool identity
* Remote desktop manager with saved credentials

A strong detection should provide enough context to separate these from malicious activity.

---

## Detection Logic Considerations

A useful detection may correlate:

* Multiple failures
* Same account
* Same source
* Defined time window
* Followed by Event ID `4740`

Additional severity may be justified when:

* The account is privileged
* Several accounts are targeted
* The source is unusual
* The attempts occur rapidly
* The source continues after lockout
* The source has other suspicious activity

---

## Example Splunk Detection Search

```spl
index=windows earliest=-15m
(
    EventCode=4625 OR
    EventCode=4740
)
| eval target_account=coalesce(TargetUserName, user)
| stats
    count(eval(EventCode=4625)) as failed_attempts
    count(eval(EventCode=4740)) as lockout_events
    values(host) as reporting_hosts
    values(src_ip) as source_addresses
    values(CallerComputerName) as caller_computers
    min(_time) as first_seen
    max(_time) as last_seen
    by target_account
| where failed_attempts > 0 AND lockout_events > 0
| convert ctime(first_seen) ctime(last_seen)
```

Field names must be adjusted for the actual sourcetype.

---

## Example Wazuh Rule Concept

A custom rule may look for the Windows lockout event and assign appropriate severity.

Public conceptual example:

```xml
<group name="windows,authentication,account_lockout,">
  <rule id="<CUSTOM_RULE_ID>" level="8">
    <field name="win.system.eventID">4740</field>
    <description>Windows account lockout detected</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

Use a rule ID from the approved custom range for the deployment.

Validate the field names against the actual decoded event.

A lockout does not automatically prove brute force.

---

## MITRE ATT&CK Context

This exercise may relate to:

```
T1110 – Brute Force
```

Potential sub-techniques depend on the observed behavior.

The mapping should be treated as analytical context.

The exercise activity is authorized and does not establish malicious intent.

---

# Evidence Collection

## Evidence

Preserve:

* Domain lockout policy
* Successful pre-test authentication
* Failed-logon events
* Event ID `4740`
* Wazuh search result
* Splunk timeline
* Locked account state
* Unlock event
* Post-unlock successful authentication
* Investigation notes
* Cleanup confirmation

---

## Export Relevant Windows Events

Example:

```
wevtutil epl Security "<EVIDENCE_PATH>\DC01-Security.evtx"
```

Raw event logs may contain unrelated sensitive events.

Store privately.

---

## Export Splunk Results

Export the narrow exercise search as:

* CSV
* JSON
* Raw events

Review and sanitize before publication.

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
Exercise: EX-02
File:
Source:
Collection time:
Description:
SHA-256:
Student analyst:
Storage location:
```

---

# Cleanup and Recovery

## Preserve Evidence Before Unlocking

Before cleanup:

1. Capture the locked account state.
2. Export the relevant logs.
3. Save Wazuh results.
4. Save the Splunk timeline.
5. Record timestamps.
6. Confirm no other account is locked.

---

## Unlock the Test Account

Run on DC01:

```
Unlock-ADAccount `
    -Identity "lockout.test"
```

Authorized domain permissions are required.

---

## Confirm the Account Is Unlocked

```
Get-ADUser `
    -Identity "lockout.test" `
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

## Confirm the Unlock Event

Search DC01 for Event ID `4767`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4767
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "lockout\.test"
}
```

---

## Confirm Successful Authentication

Sign in or authenticate once using the correct test credentials.

Record the time.

Confirm that:

* The account works
* The secure channel remains healthy
* A successful event appears
* Wazuh remains connected
* Splunk receives the event

---

## Remove the Test Account

After all evidence is collected, either disable or delete the disposable account according to the exercise plan.

Disable:

```
Disable-ADAccount `
    -Identity "lockout.test"
```

Delete:

```
Remove-ADUser `
    -Identity "lockout.test" `
    -Confirm
```

Deletion is permanent unless restored from backup.

---

## Confirm Final State

Record:

```
Account unlocked:
Successful authentication confirmed:
Account disabled or deleted:
No other accounts locked:
Wazuh healthy:
Splunk healthy:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* The lockout policy was documented.
* A disposable account was used.
* The account was confirmed valid before testing.
* The expected failed attempts occurred.
* Event ID `4740` appeared on DC01.
* The caller computer or source was identified.
* Wazuh was reviewed.
* Splunk was reviewed.
* A complete timeline was created.
* The account was restored.
* Evidence was preserved.
* Detection gaps were documented.

---

## Detection Quality Assessment

Rate the detection:

| Category                 | Result                              |
| ------------------------ | ----------------------------------- |
| Source event present     | Pass / Fail                         |
| Wazuh ingestion          | Pass / Fail                         |
| Wazuh alert quality      | Pass / Needs Tuning / Fail          |
| Splunk ingestion         | Pass / Fail                         |
| Splunk field quality     | Pass / Needs Tuning / Fail          |
| Source-system visibility | Pass / Partial / Fail               |
| Timestamp accuracy       | Pass / Needs Review / Fail          |
| False-positive context   | Sufficient / Partial / Insufficient |
| Cleanup                  | Complete / Incomplete               |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Caller computer
* Source address
* Failure reason
* Authentication protocol
* Privilege context
* Failure count
* Correlation between failures and lockout
* Clear severity
* User-friendly alert description
* Related-event timeline

---

## Improvements

Potential improvements include:

* Add a saved Splunk correlation search
* Add a Wazuh custom lockout rule
* Add privileged-account context
* Create a lockout dashboard
* Track repeated lockouts by account
* Track one source locking multiple accounts
* Add service-account exclusions carefully
* Improve Windows field extraction
* Measure ingestion delay
* Create a lockout response runbook

---

# Troubleshooting

## Account Does Not Lock

Check:

* Lockout threshold
* Observation window
* Correct domain account
* Correct domain
* Account already locked
* Authentication method
* Cached credentials
* DC availability
* Fine-grained password policy

Review:

```
Get-ADDefaultDomainPasswordPolicy
```

Check whether a fine-grained policy applies:

```
Get-ADUserResultantPasswordPolicy `
    -Identity "lockout.test"
```

---

## No Event ID 4740 Appears

Check:

* The account actually locked
* Correct domain controller
* Security auditing
* Event time range
* Event log retention
* Another DC processed the lockout
* Dashboard or PowerShell filter

In a multi-domain-controller environment, search all domain controllers.

---

## Caller Computer Is Missing

Possible causes include:

* Authentication path
* Protocol
* Event format
* Field extraction
* Application behavior

Correlate with:

* Event ID `4625`
* Event ID `4771`
* Event ID `4776`
* Endpoint logs
* Network evidence

---

## Wazuh Does Not Show the Lockout

Check:

* DC01 Wazuh agent status
* Security log collection
* Event decoder
* Rule level
* Dashboard time range
* Agent filter
* Indexer health

Confirm the event exists locally before changing the Wazuh configuration.

---

## Splunk Does Not Show the Lockout

Check:

* DC01 forwarder
* Security input
* Receiver
* Index
* Host
* Sourcetype
* Event time
* Search field
* Time range

Begin with:

```spl
index=windows earliest=-30m
| stats count by host EventCode
```

---

## Too Many Events Appear

Possible causes include:

* Both endpoint and DC events
* Duplicate inputs
* Multiple failed attempts per authentication
* Credential validation plus Kerberos failures
* Forwarder replay
* Snapshot rollback

Do not assume all similar records are duplicates.

---

## Test Account Cannot Authenticate After Unlock

Check:

* Correct password
* Account enabled state
* Password expiration
* Fine-grained policy
* Domain connectivity
* Time
* Secure channel
* Replication in multi-DC environments

Review:

```
Get-ADUser `
    -Identity "lockout.test" `
    -Properties *
```

Limit output before publishing because it may contain sensitive attributes.

---

# Public Sanitization

Remove or replace:

* Operational domain name
* Real account names
* Internal addresses
* Administrator usernames
* Security identifiers
* Exact workstation names where sensitive
* Raw event XML
* Password policy values when considered sensitive
* Splunk internal URLs
* Wazuh agent identifiers
* Session details
* Evidence storage paths

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

* Domain lockout policy
* Event ID `4740`
* Wazuh lockout event
* Splunk timeline
* Locked account state
* Successful cleanup validation

Before publishing:

1. Crop unrelated information.
2. Remove operational addresses.
3. Remove real account names.
4. Remove security identifiers.
5. Remove browser tabs and bookmarks.
6. Permanently redact sensitive values.
7. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
A controlled lockout was generated against a disposable Active Directory test account. The failed authentication events and resulting account lockout were reviewed on the Windows systems and correlated through Wazuh and Splunk. The source, account, timestamps, and recovery actions were documented.
```

## Findings

```
Test account:

Source system:

Lockout time:

Failed attempts observed:

DC event IDs:

Wazuh result:

Splunk result:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized repeated authentication failures performed as part of CyberLab Exercise EX-02.
```

## Resolution

```
The account was unlocked by an authorized administrator, successful authentication was validated, and the disposable account was disabled or removed after evidence collection.
```

## Recommendations

```
- Improve lockout-event field extraction.
- Correlate failed logons with Event ID 4740.
- Add privileged-account context.
- Create a lockout investigation runbook.
- Track repeated account lockouts over time.
```

---

# Completion Checklist

## Preparation

* Lockout policy recorded
* Test account created
* Test account nonprivileged
* Test account validated
* Separate administrator available
* DC01 healthy
* WIN11TARGET healthy
* Wazuh healthy
* Splunk healthy
* Start time recorded

## Exercise

* Controlled failures generated
* Threshold not exceeded unnecessarily
* Lockout confirmed
* Activity stopped immediately
* No other account affected

## Investigation

* WIN11TARGET events reviewed
* DC01 events reviewed
* Event ID `4740` confirmed
* Caller computer identified
* Wazuh reviewed
* Splunk reviewed
* Timeline created
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Account unlocked
* Unlock event confirmed
* Successful authentication confirmed
* Test account disabled or deleted
* Monitoring systems healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Active Directory administration
* Windows account-lockout policy review
* Windows Event Log analysis
* Authentication investigation
* Wazuh event validation
* Splunk SPL
* Event correlation
* Timeline reconstruction
* False-positive analysis
* Evidence handling
* Account recovery
* Detection tuning
* Security documentation
* Public sanitization

---

# Lessons Learned

* The lockout threshold must be known before testing.
* Disposable test accounts reduce operational risk.
* A successful authentication should be validated before generating failures.
* Failed logons may produce events on both the endpoint and domain controller.
* Event ID `4740` is central to a domain account lockout investigation.
* The caller computer is often more useful than the lockout event alone.
* Wazuh may collect the event without providing complete correlation.
* Splunk can correlate failures and lockout events into one timeline.
* A lockout does not automatically prove brute-force activity.
* Stale credentials and service accounts can create legitimate repeated lockouts.
* Evidence should be preserved before unlocking the account.
* Cleanup is part of the exercise, not a separate optional task.

---

# Summary

This exercise validates the CyberLab’s ability to detect and investigate an Active Directory account lockout.

A complete investigation requires more than confirming that the account is locked.

The student must identify:

* The affected account
* The source system
* The failed authentication sequence
* The domain controller event
* The Wazuh result
* The Splunk result
* The timing and ingestion quality
* The likely cause
* The safe recovery action

The exercise is complete only after the account is restored, the evidence is preserved, and any monitoring or detection gaps are documented.

