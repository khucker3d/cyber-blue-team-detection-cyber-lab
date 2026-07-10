# Account Lockout Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating Windows and Active Directory account lockouts.

It applies to events involving:

* Windows Event ID `4740`
* Repeated failed authentication preceding a lockout
* Domain user lockouts
* Administrative account lockouts
* Service account lockouts
* Recurring or unexplained lockouts
* Wazuh or Splunk account-lockout alerts
* User reports that an account is repeatedly becoming locked

The runbook helps the analyst determine:

* Which account was locked
* Which domain controller recorded the lockout
* Which system caused the final failed attempt
* Which systems contributed earlier authentication failures
* Whether stale credentials, a service, a scheduled task, a mapped drive, or an application caused the issue
* Whether the activity resembles brute force or password spraying
* Whether the account was used successfully after the failures
* Whether unlocking the account is safe
* Whether containment or escalation is required

---

## Runbook ID

```
RB-02
```

---

## Related Exercise

```
EX-02 – Account Lockout Investigation
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Identity and access administrators
* Windows administrators
* Detection engineers
* Incident responders
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Active Directory domain accounts
* Local Windows accounts where applicable
* Standard users
* Administrative users
* Service accounts
* Scheduled-task accounts
* Application identities
* Remote Desktop users
* Accounts used for mapped drives or network services

This runbook does not authorize:

* Unlocking an account without investigating the cause
* Disabling critical service accounts without impact review
* Requesting or collecting a user’s password
* Testing credentials outside the CyberLab
* Making changes to external, employer, or school systems
* Deleting authentication evidence before collection

---

## Primary Data Sources

Use the available sources in this order:

1. Domain controller Security logs
2. Windows Event ID `4740`
3. Related failed-authentication events
4. Active Directory account state
5. Source endpoint logs
6. Wazuh events and alerts
7. Splunk authentication searches
8. Service and scheduled-task configuration
9. Saved credentials and mapped drives
10. Firewall, VPN, and remote-access logs
11. Optional Sysmon telemetry
12. Change and support records

---

## Key Windows Event IDs

| Event ID | Description                                           |
| -------: | ----------------------------------------------------- |
|     4740 | A user account was locked out                         |
|     4767 | A user account was unlocked                           |
|     4625 | An account failed to log on                           |
|     4624 | An account successfully logged on                     |
|     4771 | Kerberos pre-authentication failed                    |
|     4776 | A domain controller attempted to validate credentials |
|     4648 | A logon was attempted using explicit credentials      |
|     4768 | A Kerberos authentication ticket was requested        |
|     4769 | A Kerberos service ticket was requested               |
|     4738 | A user account was changed                            |
|     4723 | An attempt was made to change an account password     |
|     4724 | An attempt was made to reset an account password      |

Not every authentication path produces every event.

---

# Alert Intake

## Alert Sources

An investigation may begin from:

* Wazuh account-lockout alert
* Splunk correlation search
* Windows Event Viewer
* Help-desk report
* User report
* Administrator report
* Failed service or scheduled task
* Repeated application authentication error
* Remote Desktop issue
* Ingestion-health validation
* Detection exercise

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Locked account:
Account domain:
Reporting domain controller:
Caller computer:
Source address:
Related event IDs:
Alert severity:
Analyst:
```

Do not assume the caller computer is the only source of failed authentication.

Event ID `4740` generally identifies the computer associated with the attempt that triggered the lockout threshold.

---

## Initial Analyst Questions

Determine immediately:

* Is the account privileged?
* Is the account a service account?
* Is the account still locked?
* Is the lockout affecting critical operations?
* Is the caller computer known?
* Are failures still occurring?
* Has the account been locked repeatedly?
* Was the password recently changed?
* Did a successful logon occur after the failures?
* Is there an approved test or maintenance activity?
* Are other accounts experiencing similar lockouts?

---

# Severity Classification

## Informational

Use informational severity when:

* The lockout was part of an approved test
* The account is disposable and nonprivileged
* The source is known
* No suspicious activity occurred
* The account was restored safely

---

## Low

Use low severity when:

* A standard user account is locked
* The source is the user’s normal workstation
* A recent password change likely caused stale credentials
* No other accounts are affected
* No suspicious successful logon occurred
* Business impact is minimal

---

## Medium

Use medium severity when:

* The cause is not immediately known
* The lockout recurs
* The caller computer is unusual
* Several source systems are involved
* A service account is affected
* The event disrupts a noncritical application
* The lockout occurs outside expected hours
* Multiple users report similar behavior

---

## High

Use high severity when:

* A privileged account is locked
* A critical service account is affected
* One source targets multiple accounts
* The source is unknown or unauthorized
* Lockouts occur across several systems
* A successful logon follows repeated failures
* The activity resembles password spraying or brute force
* Operational services are disrupted
* The account is used for recovery or administration

---

## Critical

Use critical severity when:

* A domain or enterprise administrator account is compromised
* Multiple privileged accounts are locked
* Authentication failures are followed by confirmed unauthorized access
* Domain-wide authentication disruption occurs
* An identity attack is actively spreading
* Lockouts are combined with privilege escalation or security-control changes
* A critical infrastructure account cannot be restored safely

---

# Initial Triage

## Step 1: Confirm the Lockout Event

Run on a domain controller as an authorized administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Search for the account:

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

Record the exact event time and domain controller.

---

## Step 2: Record Event ID 4740 Fields

Review:

* Subject security ID
* Subject account name
* Subject account domain
* Subject logon ID
* Account that was locked out
* Caller computer name
* Reporting domain controller
* Timestamp

Record:

```
Event time:
Reporting domain controller:
Locked account:
Account domain:
Caller computer:
Subject account:
Subject logon ID:
```

Do not publish operational SIDs or internal names.

---

## Step 3: Check the Current Account State

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
Enabled:
Locked:
Password expired:
Password last set:
Last bad password attempt:
Bad logon count:
Last successful logon:
Privileged membership:
```

---

## Step 4: Review the Domain Lockout Policy

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
Automatic unlock:
```

Do not change the domain policy during an active investigation without a separate approved change.

---

## Step 5: Determine Whether Activity Is Continuing

Search recent failed logons for the target account:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625, 4771, 4776
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Continuing failures after lockout suggest an unresolved credential source or active attack.

---

# Caller Computer Analysis

## Understand the Caller Computer Field

The caller computer in Event ID `4740` often identifies the system involved in the authentication attempt that caused the threshold to be reached.

It may represent:

* User workstation
* Application server
* Remote Desktop client
* File server
* Domain controller
* Scheduled-task host
* Service host
* VPN or remote-access system
* Device using cached credentials

It does not always identify every source that contributed to the lockout.

---

## Validate the Caller Computer

Confirm:

* Host exists
* Hostname is current
* IP address is known
* System owner is known
* User normally uses the system
* System is powered on
* System is part of the CyberLab
* No snapshot clone is using the same identity

On a Windows caller:

```
hostname
```

```
Get-NetIPConfiguration
```

```
quser
```

```
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, UserName
```

---

## Resolve the Caller Computer Address

Use approved internal records such as:

* DNS
* DHCP
* Active Directory
* VMware configuration
* Asset inventory

Example:

```
Resolve-DnsName "<CALLER_COMPUTER>"
```

```
Get-ADComputer `
    -Identity "<CALLER_COMPUTER>" `
    -Properties IPv4Address, LastLogonDate, OperatingSystem |
Select-Object `
    Name,
    IPv4Address,
    LastLogonDate,
    OperatingSystem
```

Treat the returned address as context, not absolute proof of identity.

---

# Authentication Correlation

## Search Event ID 4625

On the likely target or source endpoint:

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

Review:

* Logon type
* Failure reason
* Status
* Substatus
* Source network address
* Workstation name
* Process name
* Authentication package

---

## Search Event ID 4771

On DC01:

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

* Client address
* Failure code
* Pre-authentication type
* Account name
* Ticket options

Repeated bad-password Kerberos failures commonly appear here.

---

## Search Event ID 4776

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

* Source workstation
* Account
* Status
* Domain controller
* Time

---

## Search for Successful Authentication

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

Determine whether success occurred:

* Before the lockout
* Immediately before the lockout
* After an unlock
* From the caller computer
* From another source
* With a privileged logon type

A suspicious success after repeated failures requires escalation.

---

## Search for Explicit Credential Use

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4648
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

This event may identify:

* `runas`
* Alternate credentials
* Applications using explicit credentials
* Administrative tools
* Scripts

---

# Common Lockout Sources

## Saved Windows Credentials

On the caller computer:

```
cmdkey /list
```

Review entries associated with:

* File servers
* Remote Desktop
* Administrative hosts
* Network applications
* Old domain names
* Previous passwords

Do not publish the full credential list.

Do not remove entries before preserving evidence and confirming ownership.

---

## Mapped Network Drives

```
Get-SmbMapping
```

Alternative:

```
net use
```

Look for:

* Disconnected mappings
* Persistent mappings
* Old server paths
* Mappings using explicit credentials
* Drives reconnecting automatically

---

## Windows Services

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

A service using an old password may create failures at regular intervals.

---

## Scheduled Tasks

```
Get-ScheduledTask |
    Where-Object {
        $_.Principal.UserId -match "<TARGET_ACCOUNT>"
    } |
Select-Object `
    TaskName,
    TaskPath,
    State,
    Author,
    Description
```

Review task history where available.

---

## IIS Application Pools

On systems using IIS:

```
Import-Module WebAdministration

Get-ChildItem IIS:\AppPools |
    Select-Object `
        Name,
        processModel
```

Review custom identities privately.

---

## Remote Desktop

Review:

* Saved Remote Desktop credentials
* Remote Desktop history
* Terminal Services logs
* Event ID `4625` with Logon Type `10`
* Source network address
* Approved administrative hosts

---

## Mobile Devices and Applications

Possible sources include:

* Email client
* VPN client
* Wireless authentication
* Synchronization software
* Mobile device
* Old laptop
* Background application
* Password manager automation

Confirm whether the user changed the password recently.

---

## PowerShell and Scripts

Review recent process events for:

* `powershell.exe`
* `pwsh.exe`
* `cmd.exe`
* `runas.exe`
* Script hosts
* Automation tools

Search Event ID `4688` or Sysmon Event ID `1` where available.

---

# Wazuh Investigation

## Search by Account

Search for:

```
<TARGET_ACCOUNT>
```

Use a time range covering at least:

* The observation window
* The lockout time
* Several minutes after the lockout

---

## Search by Event ID

Review:

```
4740
4625
4771
4776
4624
4767
4648
```

---

## Search by Caller Computer

Search for:

```
<CALLER_COMPUTER>
```

Review whether the same source generated:

* Failed logons
* Process events
* Service events
* Scheduled-task events
* Network events
* Successful logons

---

## Wazuh Fields to Record

```
Agent:
Event ID:
Rule ID:
Rule description:
Rule level:
Locked account:
Caller computer:
Source address:
Logon type:
Failure reason:
Authentication package:
Timestamp:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Was Event ID `4740` ingested?
* Was the locked account parsed correctly?
* Was the caller computer parsed?
* Were preceding failures visible?
* Did Wazuh correlate the failures and lockout?
* Did the platform identify a repeated pattern?
* Did an unlock event appear?
* Did a success follow?
* Was the severity appropriate?
* Were source and target roles clear?

---

# Splunk Investigation

## Search the Lockout Event

```spl
index=windows EventCode=4740 earliest=-24h
| search "<TARGET_ACCOUNT>"
| table
    _time
    host
    SubjectUserName
    TargetUserName
    CallerComputerName
    Message
| sort _time
```

---

## Search the Authentication Timeline

```spl
index=windows earliest=-24h
(
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4648 OR
    EventCode=4740 OR
    EventCode=4767 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| eval activity=case(
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4648, "Explicit credentials used",
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
    CallerComputerName
    WorkstationName
    SourceNetworkAddress
    LogonType
    FailureReason
| sort _time
```

---

## Count Failures Before the Lockout

```spl
index=windows earliest=-1h
(
    EventCode=4625 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| eval source_system=coalesce(
    CallerComputerName,
    WorkstationName,
    SourceWorkstation,
    SourceNetworkAddress
)
| stats
    count as failures
    values(EventCode) as event_codes
    values(host) as reporting_hosts
    min(_time) as first_seen
    max(_time) as last_seen
    by source_system
| convert ctime(first_seen) ctime(last_seen)
| sort - failures
```

---

## Identify Recurring Intervals

Regular intervals may indicate a service or scheduled task.

```spl
index=windows earliest=-4h
(
    EventCode=4625 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| sort _time
| streamstats current=f last(_time) as previous_time
| eval seconds_since_previous=_time-previous_time
| table
    _time
    host
    EventCode
    WorkstationName
    SourceNetworkAddress
    seconds_since_previous
```

Patterns such as every minute, five minutes, or hourly may help identify automation.

---

## Search Multiple Lockouts

```spl
index=windows EventCode=4740 earliest=-24h
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as lockouts
    values(CallerComputerName) as caller_computers
    values(host) as domain_controllers
    min(_time) as first_seen
    max(_time) as last_seen
    by target_account
| where lockouts > 1
| convert ctime(first_seen) ctime(last_seen)
| sort - lockouts
```

---

## One Caller Locking Multiple Accounts

```spl
index=windows EventCode=4740 earliest=-1h
| eval caller=coalesce(CallerComputerName, WorkstationName)
| eval target_account=coalesce(TargetUserName, user)
| stats
    count as lockouts
    dc(target_account) as unique_accounts
    values(target_account) as locked_accounts
    by caller
| where unique_accounts >= <ACCOUNT_THRESHOLD>
| sort - unique_accounts
```

This may indicate a broken application, shared stale credentials, or malicious activity.

---

## Failures Followed by Success

```spl
index=windows earliest=-1h
(
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4740
)
| search "<TARGET_ACCOUNT>"
| eval event_type=case(
    EventCode=4624, "success",
    EventCode=4625, "failure",
    EventCode=4740, "lockout"
)
| stats
    count(eval(event_type="failure")) as failures
    count(eval(event_type="success")) as successes
    count(eval(event_type="lockout")) as lockouts
    values(SourceNetworkAddress) as sources
    min(_time) as first_seen
    max(_time) as last_seen
    by TargetUserName
| convert ctime(first_seen) ctime(last_seen)
```

Review raw event order before concluding that success followed the lockout.

---

# Investigation Decision Tree

```
Account Lockout Alert
         |
         v
Confirm Event ID 4740
         |
         v
Is the account privileged or operationally critical?
         |
    +----+----+
    |         |
   Yes        No
    |         |
Raise       Identify caller
severity    computer and sources
                  |
                  v
        Was the password changed recently?
                  |
             +----+----+
             |         |
            Yes        No
             |         |
     Check stale      Are failures
     credentials      recurring?
                           |
                      +----+----+
                      |         |
                     Yes        No
                      |         |
              Check service,   Check user error,
              task, app,       RDP, mapped drive,
              device           unknown source
                      |
                      v
        One source affecting many accounts?
                      |
                 +----+----+
                 |         |
                Yes        No
                 |         |
          Possible spray,  Resolve specific
          broken service,  stale credential
          or shared app
```

---

# Common Investigation Scenarios

## Scenario 1: User Entered an Old Password

Indicators:

* Recent password change
* Caller computer is the user’s normal endpoint
* Interactive or unlock failures
* User confirms the activity
* Failures stop after correct authentication

Response:

* Confirm no other device retains the old password
* Unlock only after failures stop
* Document as benign user error

---

## Scenario 2: Stale Mapped Drive

Indicators:

* Logon Type `3`
* Repeated failures from one workstation
* Persistent network mapping
* Password recently changed
* Activity begins at sign-in or network reconnect

Response:

* Preserve mapping details
* Remove or reconnect the mapping using approved credentials
* Confirm failures stop
* Unlock the account if appropriate

---

## Scenario 3: Service Account Password Mismatch

Indicators:

* Repeated failures at regular intervals
* Same server or service host
* Logon Type `5`
* Service configured with the locked account
* Password recently changed
* Operational service failures

Response:

* Coordinate with the service owner
* Update the service credential securely
* Restart only the affected service where required
* Confirm authentication and service health
* Review whether the account should be configured to avoid interactive use

---

## Scenario 4: Scheduled Task

Indicators:

* Failures occur according to a schedule
* Same caller computer
* Task configured with target account
* Task History aligns with failures

Response:

* Update or disable the affected task
* Validate task ownership
* Confirm successful execution after correction
* Monitor for continuing failures

---

## Scenario 5: Saved Remote Desktop Credential

Indicators:

* Logon Type `10`
* Saved credential exists
* Source is an administrative workstation
* RDP attempts recur when the user opens the client

Response:

* Remove stale saved credential
* Reauthenticate through the approved process
* Confirm no unauthorized RDP success occurred

---

## Scenario 6: Password Spraying

Indicators:

* One source causes failures for many accounts
* A small number of failures per account
* Lockouts may affect some accounts
* Similar timing and logon method
* Source is unknown or unauthorized

Response:

* Escalate
* Contain the source
* Identify all targeted accounts
* Review successful logons
* Prioritize privileged accounts
* Consider credential reset and session revocation based on scope

---

## Scenario 7: Brute Force Against One Account

Indicators:

* Many failures against one account
* Short time window
* Same or related source
* Lockout threshold reached
* Remote interactive or network logon

Response:

* Escalate
* Block or isolate the source
* Protect the target account
* Review successful logons
* Preserve authentication and network evidence

---

## Scenario 8: Shared Application Credential

Indicators:

* Multiple systems use one account
* Several application servers report failures
* Password changed without updating all dependent systems
* Lockout recurs after unlock

Response:

* Inventory every dependency
* Update credentials securely
* Consider managed service accounts or improved secret management
* Do not repeatedly unlock until dependencies are corrected

---

# Unlock Decision

## Do Not Unlock Yet When

* Failures are still occurring
* The source is unknown
* The account may be compromised
* A privileged account is targeted
* A service dependency is unresolved
* A successful suspicious logon occurred
* The same account has been repeatedly unlocked without root-cause correction

Unlocking before resolving the source often causes immediate relockout.

---

## Unlock May Be Appropriate When

* The source is identified
* The root cause is corrected
* Failures have stopped
* The account owner is verified
* No compromise indicators are present
* Operational impact requires restoration
* Approval exists where required

---

## Unlock the Account

Run from an authorized administrative session:

```
Unlock-ADAccount `
    -Identity "<TARGET_ACCOUNT>"
```

---

## Confirm the Account State

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
    -Properties LockedOut, Enabled, LastBadPasswordAttempt, BadLogonCount |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    LastBadPasswordAttempt,
    BadLogonCount
```

---

## Confirm Event ID 4767

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4767
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "<TARGET_ACCOUNT>"
}
```

Record:

```
Unlock time:
Administrator:
Domain controller:
Unlock event confirmed:
```

---

# Containment

## When Containment Is Not Required

Containment may not be necessary when:

* The cause is confirmed as user error
* The source is known and trusted
* No suspicious activity occurred
* Failures stopped after correcting stale credentials
* The account is nonprivileged
* The event was part of an approved test

Document why containment was unnecessary.

---

## Account Containment Options

With appropriate authorization:

* Keep the account locked
* Disable the account
* Force a password reset
* Revoke active sessions
* Remove temporary privileges
* Restrict remote logon
* Rotate service credentials
* Replace shared service credentials

Do not disable operational accounts without impact review.

---

## Source Endpoint Containment

Depending on severity:

* Disconnect the VM’s network adapter
* Isolate the source host
* Stop the responsible service
* Disable the scheduled task
* Remove stale mapped drives
* Remove saved credentials
* End unauthorized sessions
* Preserve volatile evidence

---

## Network Containment

* Block the unauthorized source address
* Restrict Remote Desktop
* Restrict SMB
* Limit access to approved management hosts
* Disable an unnecessary firewall rule
* Segment the source system

Use narrow, documented changes with rollback steps.

---

# Eradication

## Stale User Credential

* Remove obsolete saved credentials
* Reconnect mapped drives
* Update application authentication
* Remove obsolete sessions
* Confirm all user devices use the current password

---

## Service or Task Credential

* Update the service or task securely
* Validate account permissions
* Restart only the affected component
* Confirm successful operation
* Review whether a managed service account is appropriate

---

## Unauthorized Access Attempt

* Remove the unauthorized source
* Reset affected credentials
* Revoke sessions
* Review account and group changes
* Review endpoint processes
* Review remote access
* Review persistence
* Review successful logons

---

## Compromised Account

* Disable or restrict account
* Reset credentials
* Revoke sessions
* Review privilege and group membership
* Review mailbox or application access where relevant
* Review lateral movement
* Restore only after scope is understood

---

# Recovery

## Confirm Failures Have Stopped

Splunk:

```spl
index=windows earliest=-30m
(
    EventCode=4625 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| timechart span=5m count
```

Wazuh:

* Search the target account
* Review the latest event time
* Confirm no continued failure pattern

---

## Confirm the Account Is Unlocked

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
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

## Validate Authentication

Perform one controlled authentication after the cause is resolved.

Record:

```
Validation time:
Source system:
Authentication type:
Successful:
Event ID 4624 confirmed:
Unexpected failures:
```

Do not repeatedly test a sensitive account.

---

## Validate the Corrected Service or Task

Confirm:

* Service is running
* Scheduled task completes
* Mapped drive reconnects
* Application authenticates
* No additional lockout occurs
* Monitoring receives the success event

---

## Confirm Monitoring Health

Verify:

* Wazuh agent active
* Splunk forwarder active
* Domain controller healthy
* Caller computer healthy
* Time synchronized
* New events ingested correctly

---

# Escalation Criteria

Escalate immediately when:

* A privileged account is locked
* A critical service account is affected
* The source is unknown
* Multiple accounts are locked
* Multiple source systems are involved
* A successful logon follows suspicious failures
* Lockouts recur after corrective action
* Authentication activity crosses several hosts
* The account is used for domain recovery
* Operational systems are disrupted
* The activity resembles spraying or brute force
* Identity controls are changed
* The analyst cannot determine scope

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
First observed:
Last observed:
Locked account:
Account privilege:
Account purpose:
Caller computer:
Additional source systems:
Source addresses:
Failure count:
Lockout count:
Successful logons:
Related services or tasks:
Containment completed:
Unlock status:
Evidence locations:
Outstanding questions:
Recommended next action:
```

---

# Evidence Collection

## Minimum Evidence

Preserve:

* Original alert
* Event ID `4740`
* Relevant `4625`, `4771`, and `4776` events
* Account state
* Lockout policy
* Caller-computer identity
* Wazuh results
* Splunk timeline
* Service, task, mapping, or credential evidence
* Event ID `4767` if unlocked
* Successful authentication validation
* Containment and recovery actions
* Analyst notes

---

## Export Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

Raw Security logs may contain unrelated sensitive activity.

Store them privately.

---

## Export a Narrow Event Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624, 4625, 4648, 4740, 4767, 4771, 4776
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\account-lockout-summary.csv" `
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
Runbook: RB-02
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

# False Positives and Benign Causes

Common benign causes include:

* Mistyped password
* Recently changed password
* Saved Windows credential
* Persistent mapped drive
* Remote Desktop credential
* Scheduled task
* Windows service
* Mobile device
* VPN client
* Email client
* Application pool
* Backup software
* Monitoring application
* Shared application credential
* Approved account-lockout exercise

A benign root cause should still be corrected when it can cause repeated lockouts or operational disruption.

---

# Detection Gaps

Document whether the investigation lacked:

* Locked account
* Caller computer
* Source address
* Logon type
* Failure reason
* Authentication package
* Account privilege
* Account purpose
* Password-change context
* Lockout-policy context
* Service or task attribution
* Correlation with successful logon
* Repeated-lockout detection
* Multi-account correlation
* Multi-source correlation
* Unlock-event visibility

---

# Detection Improvement Recommendations

Potential improvements include:

* Normalize locked-account fields
* Normalize caller-computer fields
* Correlate `4740` with `4625`, `4771`, and `4776`
* Alert on repeated lockouts
* Add privileged-account context
* Add service-account context
* Add recent-password-change context
* Detect one source locking multiple accounts
* Detect one account locked from multiple sources
* Correlate lockout with success
* Correlate lockout with unlock
* Track lockout duration
* Add approved-test context
* Create an account-lockout dashboard
* Monitor authentication dependencies
* Reduce shared service credentials

---

# MITRE ATT&CK Context

Account lockouts caused by repeated authentication attempts may relate to:

```
T1110 – Brute Force
```

Potential sub-techniques include:

* Password Guessing
* Password Spraying
* Credential Stuffing

Lockout-related disruption may also be considered in the context of account-access impact, but the ATT&CK mapping must reflect the observed behavior.

A lockout alone does not establish malicious intent.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Locked account:
Account type:
Account privilege:
Account purpose:
Lockout time:
Reporting domain controller:
Caller computer:
Source addresses:
Failure count:
Lockout count:
Password recently changed:
Related service:
Related scheduled task:
Related saved credential:
Successful logon afterward:
Root cause:
Severity:
Containment:
Unlock time:
Recovery:
Detection gaps:
```

---

## Root Cause Categories

Select one:

```
User typing error
Recently changed password
Saved credential
Mapped drive
Remote Desktop
Windows service
Scheduled task
Application pool
Mobile device
VPN client
Shared application credential
Password spraying
Brute force
Unauthorized credential use
Approved test activity
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* Event ID `4740` is confirmed.
* The locked account is identified.
* Account privilege and purpose are known.
* The caller computer is identified.
* Preceding failures are reviewed.
* The lockout policy is understood.
* Related successful logons are reviewed.
* The root cause is corrected or escalated.
* Failures have stopped.
* The account is safely unlocked or intentionally remains restricted.
* Unlock telemetry is reviewed where applicable.
* Authentication or service function is validated.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The account lockout was investigated using domain-controller, endpoint, Wazuh, and Splunk telemetry. The locked account, caller computer, preceding authentication failures, account state, lockout policy, and related successful logons were reviewed. The root cause was classified as <CLASSIFICATION>. Required containment and corrective actions were completed, the account was restored or intentionally restricted, monitoring confirmed that the failures stopped, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Real usernames
* Administrative identities
* Service-account names
* Internal IP addresses
* Domain names
* Hostnames where sensitive
* SIDs
* Account group memberships
* Service names where sensitive
* Scheduled-task details
* Saved credential targets
* Raw event XML
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths

Use placeholders such as:

```
<TARGET_ACCOUNT>
<CALLER_COMPUTER>
<SOURCE_IP>
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
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

* Event ID `4740`
* Related Event ID `4771` or `4776`
* Account lockout state
* Wazuh lockout alert
* Splunk lockout timeline
* Failure-source summary
* Service or scheduled-task evidence
* Event ID `4767`
* Recovery validation

Before publishing:

1. Crop unrelated information.
2. Remove usernames.
3. Remove internal addresses.
4. Remove operational domain names.
5. Remove SIDs.
6. Remove service secrets.
7. Remove unrelated events.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Locked account identified
* Reporting domain controller identified
* Caller computer identified
* Initial severity assigned
* Business impact recorded

## Account Review

* Account enabled state confirmed
* Lockout state confirmed
* Account type identified
* Account privilege reviewed
* Account purpose reviewed
* Password-expiration state reviewed
* Password-last-set time reviewed
* Last bad password attempt reviewed
* Last successful logon reviewed

## Policy Review

* Lockout threshold recorded
* Lockout duration recorded
* Observation window recorded
* Automatic unlock behavior understood

## Authentication Correlation

* Event ID 4740 reviewed
* Event ID 4625 reviewed
* Event ID 4771 reviewed
* Event ID 4776 reviewed
* Event ID 4648 reviewed where relevant
* Event ID 4624 reviewed
* Failure count calculated
* Lockout count calculated
* Source systems identified
* Recurring timing reviewed

## Caller Computer Review

* Caller hostname validated
* Caller address validated
* Logged-on user reviewed
* Saved credentials reviewed where relevant
* Mapped drives reviewed where relevant
* Services reviewed where relevant
* Scheduled tasks reviewed where relevant
* Remote Desktop reviewed where relevant
* Applications reviewed where relevant

## SIEM Review

* Wazuh alert reviewed
* Wazuh field quality assessed
* Splunk timeline created
* Repeated lockouts searched
* One-source/multiple-account pattern checked
* Success-after-failure pattern checked
* Ingestion delay reviewed where needed

## Response

* Severity updated
* Containment decision recorded
* Root cause corrected
* Continuing failures stopped
* Unlock decision documented
* Account unlocked or intentionally restricted
* Event ID 4767 reviewed
* Authentication or service validated

## Closure

* Timeline completed
* Evidence preserved
* Evidence hashes recorded
* Root cause documented
* False positive or incident classification recorded
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Primary Lockout Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4740
        StartTime = (Get-Date).AddHours(-1)
    }
```

## Primary Account Check

```
Get-ADUser `
    -Identity "<TARGET_ACCOUNT>" `
    -Properties LockedOut, Enabled, PasswordLastSet, LastBadPasswordAttempt, BadLogonCount
```

## Primary Splunk Search

```spl
index=windows earliest=-1h
(
    EventCode=4625 OR
    EventCode=4740 OR
    EventCode=4771 OR
    EventCode=4776
)
| search "<TARGET_ACCOUNT>"
| table
    _time
    host
    EventCode
    TargetUserName
    CallerComputerName
    WorkstationName
    SourceNetworkAddress
    FailureReason
| sort _time
```

## Unlock Command

```
Unlock-ADAccount `
    -Identity "<TARGET_ACCOUNT>"
```

## Immediate Escalation Indicators

```
Privileged account
Critical service account
Unknown caller computer
Multiple accounts locked
Repeated lockouts
Successful logon after failures
Password-spray pattern
Brute-force pattern
Operational disruption
Continuing activity
```

---

# Summary

This runbook provides a repeatable process for investigating Active Directory account lockouts.

A complete investigation should determine:

* Which account was locked
* When the lockout occurred
* Which domain controller recorded it
* Which caller computer triggered the threshold
* Which systems generated preceding failures
* Whether the account is privileged or operationally critical
* Whether stale credentials, a service, a task, or malicious activity caused the lockout
* Whether successful authentication followed
* Whether the account can be unlocked safely
* Whether the root cause was corrected
* Whether failures stopped
* Whether monitoring and account access returned to a healthy state

The investigation is complete only after the lockout is classified, the source is corrected or contained, the account is safely restored or intentionally restricted, evidence is preserved, and detection improvements are documented.
