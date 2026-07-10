# User Account Creation

## Exercise Overview

This exercise creates a temporary Active Directory user account and validates the resulting telemetry across:

* DC01
* Windows Event Viewer
* Wazuh
* Splunk
* Active Directory administration tools
* Investigation notes and evidence

The objective is to determine:

* Which account created the new user
* Which user account was created
* Where the account was created
* Whether the account was enabled
* Whether a password was set
* Whether group memberships changed
* Whether the event was collected correctly
* Whether the SIEMs provided enough investigation context
* Whether the temporary account was removed safely

All activity must remain inside the authorized CyberLab.

---

## Exercise ID

```
EX-03
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Active Directory account administration
* Windows account-management auditing
* Event Viewer filtering
* Wazuh event validation
* Splunk SPL development
* Identity-change investigation
* Actor and target correlation
* Timeline reconstruction
* Evidence preservation
* Cleanup and recovery

---

## Authorization Boundary

This exercise must use:

* An authorized CyberLab administrator
* A temporary nonprivileged test account
* A designated test organizational unit
* DC01
* The internal VMware host-only network
* Wazuh and Splunk monitoring
* Synthetic account information

Do not use:

* A personal account
* A school or employer account
* A production identity
* A recovery administrator
* A service account
* An account required for another lab function
* Any system outside the CyberLab

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Domain: cyberlab.example
Authorized administrator: student.admin
Temporary account: creation.test
Test organizational unit: CyberLab-Test-Users
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Create a temporary Active Directory user safely
* Confirm the account’s initial properties
* Identify the account-creation event
* Identify the administrator who performed the action
* Identify the target account
* Review related account-management events
* Locate the activity in Wazuh
* Locate the activity in Splunk
* Build an identity-change timeline
* Evaluate detection quality
* Remove or disable the temporary account
* Confirm the cleanup event

---

## Required Systems

* DC01
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on DC01
* Splunk Universal Forwarder on DC01 where configured

WIN11TARGET is optional for the standard exercise.

KALI-TEST is not required.

---

## Required Accounts

### Student User

Used for observation and routine access.

```
student.user
```

### Authorized Administrator

Used only for account creation, validation, and cleanup.

```
student.admin
```

### Temporary Test Account

Created during the exercise.

```
creation.test
```

The temporary account must not be privileged.

---

## Required Monitoring

Before starting, confirm:

* DC01 Security log is active
* Account-management auditing is enabled
* Wazuh agent is active on DC01
* Wazuh dashboard is accessible
* Splunk is running
* DC01 events are searchable
* System time is synchronized
* The test organizational unit exists
* A stable snapshot exists

---

## Safety Notes

* Create the account only in the designated test organizational unit.
* Do not add the account to privileged groups.
* Do not reuse a real password.
* Do not publish the test password.
* Use synthetic names and descriptions.
* Preserve evidence before deleting the account.
* Confirm no unintended group membership is assigned.
* Remove or disable the account after the exercise.

---

## Expected Event Flow

```
Authorized Administrator
          |
          v
Active Directory User Creation
          |
          v
DC01 Security Event
          |
          v
Wazuh and Splunk Ingestion
          |
          v
Student Investigation
          |
          v
Account Disablement or Deletion
          |
          v
Cleanup Event Validation
```

---

## Key Windows Event IDs

| Event ID | Description                                              |
| -------- | -------------------------------------------------------- |
| 4720     | A user account was created                               |
| 4722     | A user account was enabled                               |
| 4725     | A user account was disabled                              |
| 4726     | A user account was deleted                               |
| 4738     | A user account was changed                               |
| 4723     | An attempt was made to change an account password        |
| 4724     | An attempt was made to reset an account password         |
| 4732     | A member was added to a security-enabled local group     |
| 4728     | A member was added to a security-enabled global group    |
| 4756     | A member was added to a security-enabled universal group |

The exact sequence depends on how the account is created and configured.

---

## Important Event Fields

Review:

* Subject security ID
* Subject account name
* Subject account domain
* Subject logon ID
* Target account name
* Target account domain
* Target security ID
* Display name
* User principal name
* Password settings
* Account control values
* Organizational unit
* Group membership changes
* Timestamp
* Reporting computer

---

## Investigation Questions

The final report should answer:

* Which administrator created the account?
* Which account was created?
* What time was it created?
* On which domain controller was the event recorded?
* Was the account enabled immediately?
* Was a password set or reset?
* Which organizational unit contains the account?
* Was the account added to any groups?
* Did Wazuh collect the event?
* Did Splunk index the event?
* Were the actor and target fields parsed correctly?
* Were timestamps aligned?
* Did any unexpected account changes occur?
* Was the account removed or disabled successfully?
* What legitimate and suspicious scenarios could produce the same event?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* DC01
* WAZUH-SERVER
* SPLUNK-SERVER

A pre-exercise snapshot is recommended because the exercise changes directory state.

Suggested name:

```
PRE-EX03-User-Account-Creation
```

---

## Confirm System Time

On DC01:

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

## Confirm Account-Management Auditing

Run on DC01:

```
auditpol /get /subcategory:"User Account Management"
```

The effective policy should record successful account-management events.

If the setting is controlled through Group Policy, review the applicable GPO rather than making an undocumented local change.

---

## Confirm Wazuh Agent Status

```
Get-Service |
    Where-Object DisplayName -Match "Wazuh"
```

Confirm DC01 is active in the Wazuh dashboard.

---

## Confirm Splunk Forwarder Status

```
Get-Service |
    Where-Object DisplayName -Match "Splunk"
```

Confirm recent DC01 telemetry:

```spl
index=windows host=DC01 earliest=-15m
| head 20
```

---

## Confirm the Test Organizational Unit

Example distinguished name:

```
OU=CyberLab-Test-Users,DC=cyberlab,DC=example
```

Check whether the OU exists:

```
Get-ADOrganizationalUnit `
    -Filter 'Name -eq "CyberLab-Test-Users"'
```

Create it only if the exercise design requires it:

```
New-ADOrganizationalUnit `
    -Name "CyberLab-Test-Users" `
    -Path "DC=cyberlab,DC=example" `
    -ProtectedFromAccidentalDeletion $true
```

Use the actual sanitized lab path privately.

---

## Confirm the Temporary Account Does Not Already Exist

```
Get-ADUser `
    -Filter 'SamAccountName -eq "creation.test"'
```

Expected result:

```
No matching account
```

If the account already exists, use a new disposable name or remove the stale test object after preserving any required evidence.

---

## Start the Exercise Record

```
Exercise ID: EX-03
Authorized administrator: student.admin
Target account: creation.test
Target OU: CyberLab-Test-Users
Domain controller: DC01
Start time:
Expected event IDs:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Create the Temporary Account

Run on DC01 using the authorized administrative account.

Use a secure interactive password prompt:

```
$Password = Read-Host `
    "Enter the temporary test password" `
    -AsSecureString
```

Create the account:

```
New-ADUser `
    -Name "Creation Test" `
    -GivenName "Creation" `
    -Surname "Test" `
    -DisplayName "Creation Test" `
    -SamAccountName "creation.test" `
    -UserPrincipalName "creation.test@cyberlab.example" `
    -Path "OU=CyberLab-Test-Users,DC=cyberlab,DC=example" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $false `
    -Description "Temporary account for authorized CyberLab Exercise EX-03"
```

Do not place the password directly in the command or documentation.

---

## Record the Creation Time

Immediately record:

```
Creation time:
Administrator:
Target account:
Target OU:
Account enabled:
Password configured:
```

---

## Validate the Account Object

```
Get-ADUser `
    -Identity "creation.test" `
    -Properties `
        Enabled,
        DistinguishedName,
        UserPrincipalName,
        Description,
        MemberOf,
        WhenCreated |
Select-Object `
    SamAccountName,
    Enabled,
    UserPrincipalName,
    DistinguishedName,
    Description,
    WhenCreated,
    MemberOf
```

Confirm:

* The account exists.
* The account is enabled.
* The account is in the intended OU.
* The description identifies it as a temporary lab account.
* The account is not privileged.
* No unexpected group membership exists.

---

## Optional Successful Authentication

A successful authentication may be performed after creation to confirm the account is usable.

This is optional because it creates additional authentication telemetry outside the main account-creation objective.

When performed:

1. Record the authentication time.
2. Use WIN11TARGET.
3. Sign in with the temporary account.
4. Confirm access.
5. Sign out.
6. Record the related successful-logon event.

---

# DC01 Event Validation

## Search for Event ID 4720

Run as an administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

---

## Search for the Specific Test Account

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

---

## Event Viewer Procedure

1. Open **Event Viewer**.
2. Navigate to **Windows Logs**.
3. Select **Security**.
4. Choose **Filter Current Log**.
5. Enter event ID `4720`.
6. Use a narrow time range.
7. Open the matching event.
8. Review the **General** tab.
9. Review the XML in the **Details** tab.
10. Record the actor and target fields.

---

## Review Event ID 4720 Fields

### Subject

Review:

* Security ID
* Account name
* Account domain
* Logon ID

This section identifies the actor.

### New Account

Review:

* Security ID
* Account name
* Account domain

This section identifies the target account.

### Attributes

Review where available:

* SAM account name
* Display name
* User principal name
* Home directory
* Profile path
* Password settings
* Account control
* User parameters
* Allowed workstation values

Some fields may display a dash when they were not configured.

---

## Record Event ID 4720 Findings

```
Event time:
Reporting host:
Actor account:
Actor domain:
Actor logon ID:
Target account:
Target domain:
Target SID:
Display name:
User principal name:
Account control:
```

Do not publish the operational SID.

---

## Search for Event ID 4722

If the account was enabled during creation:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4722
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

Depending on the creation method, enablement may appear separately or be represented through account-control changes.

---

## Search for Event ID 4738

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4738
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

This event may appear when attributes are updated.

---

## Search for Password-Related Events

Password reset attempt:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4724
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

Password change attempt:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4723
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

The exact event sequence depends on the administrative method.

---

## Search for Unexpected Group Membership

Review the new account’s groups:

```
Get-ADPrincipalGroupMembership `
    -Identity "creation.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Expected default membership may include the standard domain user group.

Confirm the account was not added to:

* Domain Admins
* Enterprise Admins
* Schema Admins
* Administrators
* Account Operators
* Server Operators
* Backup Operators
* Other privileged custom groups

---

# Wazuh Validation

## Confirm Agent Health

In the Wazuh dashboard, confirm:

* DC01 is active
* Last keepalive is recent
* The selected time range includes the exercise
* No agent identity changed

---

## Search by Test Account

Search for:

```
creation.test
```

Use a narrow time range around the account creation.

---

## Search by Event ID

Review events associated with:

```
4720
4722
4723
4724
4738
```

The available fields depend on the Wazuh decoder and Windows-event collection configuration.

---

## Review Wazuh Fields

Record:

* Agent
* Windows event ID
* Rule description
* Rule level
* Actor account
* Target account
* Target domain
* Target SID
* Account attributes
* Timestamp
* MITRE ATT&CK mapping
* Compliance mappings

---

## Wazuh Validation Questions

* Did Wazuh collect Event ID `4720`?
* Did it create a visible alert?
* Was the actor account parsed correctly?
* Was the target account parsed correctly?
* Was the domain identified?
* Was the rule level appropriate?
* Were related account events visible?
* Did the alert explain why the event matters?
* Was the event received promptly?

---

## Wazuh Detection Gap Record

```
Telemetry present:
Alert present:
Rule:
Rule level:
Actor parsed:
Target parsed:
Missing fields:
Severity appropriate:
Recommended improvement:
```

A user-creation event may be legitimate or suspicious depending on context.

---

# Splunk Validation

## Search for the Test Account

```spl
index=windows "creation.test" earliest=-30m
| table _time host source sourcetype EventCode user Message
| sort _time
```

---

## Search for Event ID 4720

```spl
index=windows EventCode=4720 earliest=-30m
| search "creation.test"
| table
    _time
    host
    SubjectUserName
    SubjectDomainName
    TargetUserName
    TargetDomainName
    TargetSid
    SamAccountName
    UserPrincipalName
    Message
| sort _time
```

Field names depend on the sourcetype and installed add-ons.

---

## Search for Related Account Events

```spl
index=windows earliest=-30m
(
    EventCode=4720 OR
    EventCode=4722 OR
    EventCode=4723 OR
    EventCode=4724 OR
    EventCode=4738
)
| search "creation.test"
| table
    _time
    host
    EventCode
    SubjectUserName
    TargetUserName
    UserPrincipalName
    Message
| sort _time
```

---

## Build an Identity-Change Timeline

```spl
index=windows earliest=-30m
| search "creation.test"
| eval action=case(
    EventCode=4720, "Account created",
    EventCode=4722, "Account enabled",
    EventCode=4723, "Password change attempted",
    EventCode=4724, "Password reset attempted",
    EventCode=4738, "Account changed",
    EventCode=4725, "Account disabled",
    EventCode=4726, "Account deleted",
    true(), "Other related event"
)
| table
    _time
    host
    EventCode
    action
    SubjectUserName
    TargetUserName
    Message
| sort _time
```

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
| search "creation.test"
| eval ingestion_delay_seconds=_indextime-_time
| table
    _time
    _indextime
    ingestion_delay_seconds
    host
    EventCode
| sort _time
```

---

## Check Actor and Target Fields

```spl
index=windows EventCode=4720 earliest=-30m
| search "creation.test"
| stats
    values(SubjectUserName) as actors
    values(TargetUserName) as created_accounts
    values(host) as reporting_hosts
    count
```

---

## Splunk Validation Questions

* Which host recorded the event?
* Which account performed the creation?
* Which account was created?
* Were actor and target fields distinguished correctly?
* Was the UPN visible?
* Was the target SID present?
* Were related enablement or change events present?
* Was ingestion timely?
* Were duplicate events present?
* Could the event be used in a reliable saved search?

---

# Group Membership Validation

## Confirm Current Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "creation.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Record:

```
Expected default groups:
Unexpected groups:
Privileged membership:
```

---

## Search for Group-Addition Events

Global group addition:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

Local group addition:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4732
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

Universal group addition:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4756
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

No unexpected privileged-group event should appear.

---

# Investigation

## Build the Timeline

| Time     | System |       Event ID | Actor         | Target        | Analyst Note                               |
| -------- | ------ | -------------: | ------------- | ------------- | ------------------------------------------ |
| `<TIME>` | DC01   |           4720 | student.admin | creation.test | Temporary account created                  |
| `<TIME>` | DC01   |   4722 or 4738 | student.admin | creation.test | Account enabled or attributes updated      |
| `<TIME>` | Wazuh  | Alert or event | student.admin | creation.test | Account-management telemetry received      |
| `<TIME>` | Splunk |  Indexed event | student.admin | creation.test | Event searchable                           |
| `<TIME>` | DC01   |   4725 or 4726 | student.admin | creation.test | Account disabled or deleted during cleanup |

---

## Root Cause

For this controlled exercise:

```
Authorized creation of a temporary nonprivileged Active Directory account during CyberLab Exercise EX-03.
```

---

## Common Legitimate Causes

User creation may be legitimate when performed for:

* New employee onboarding
* Contractor access
* Temporary project access
* Test account creation
* Service-account provisioning
* Training environment setup
* Application integration
* Help-desk workflow

---

## Common Suspicious Causes

User creation may be suspicious when:

* An unusual administrator performs it
* The account is created outside business hours
* The account is placed in a privileged group
* The account name mimics an existing user
* The account has unusual password settings
* The account is hidden in an unexpected OU
* The account authenticates immediately from an unusual system
* Security controls are changed afterward
* The account is created following suspicious PowerShell activity
* No approved request exists

---

## Distinguish Benign from Suspicious Activity

Consider:

* Identity of the creator
* Privilege of the creator
* Target OU
* Account naming standard
* Description
* Request or change record
* Group membership
* Password policy
* Time of creation
* Initial authentication
* Related changes
* Source workstation
* Whether the account is temporary

---

## False-Positive Analysis

Legitimate administrative tools may create accounts through:

* Active Directory Users and Computers
* PowerShell
* Identity-management platforms
* HR onboarding automation
* Infrastructure-as-code
* Service-management workflows
* Delegated help-desk tools

A strong detection should preserve the actor, target, source, and privilege context.

---

# Detection Development

## Basic Splunk Search

```spl
index=windows EventCode=4720 earliest=-24h
| table
    _time
    host
    SubjectUserName
    TargetUserName
    UserPrincipalName
| sort - _time
```

---

## Account Creation by Unusual Actor

```spl
index=windows EventCode=4720 earliest=-24h
| eval actor=coalesce(SubjectUserName, user)
| eval target=coalesce(TargetUserName, SamAccountName)
| where NOT actor IN (
    "<APPROVED_ADMIN_1>",
    "<APPROVED_ADMIN_2>"
)
| table _time host actor target UserPrincipalName
```

Replace the approved values with a maintained lookup in a more mature implementation.

---

## Account Creation Followed by Group Addition

```spl
index=windows earliest=-30m
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval target_account=coalesce(TargetUserName, MemberName, SamAccountName)
| transaction target_account maxspan=15m
| search EventCode=4720
| table
    _time
    host
    target_account
    EventCode
    SubjectUserName
    GroupName
```

`transaction` can be expensive. A production search may use `stats` or data-model correlation instead.

---

## Account Creation Outside Expected Hours

```spl
index=windows EventCode=4720 earliest=-7d
| eval hour=strftime(_time, "%H")
| where hour < <START_HOUR> OR hour >= <END_HOUR>
| table
    _time
    host
    SubjectUserName
    TargetUserName
    UserPrincipalName
```

Time-based logic should account for approved schedules and timezones.

---

## Example Wazuh Rule Concept

```xml
<group name="windows,account_management,user_creation,">
  <rule id="<CUSTOM_RULE_ID>" level="7">
    <field name="win.system.eventID">4720</field>
    <description>Windows user account created</description>
    <mitre>
      <id>T1136.002</id>
    </mitre>
  </rule>
</group>
```

Validate actual field names against the decoded Wazuh event.

A higher severity may be justified when the new user is added to a privileged group.

---

## MITRE ATT&CK Context

This exercise may relate to:

```
T1136.002 – Create Account: Domain Account
```

The mapping provides analytical context.

The authorized exercise does not establish malicious intent.

---

## Detection Severity Guidance

| Pattern                                                 | Suggested Context    |
| ------------------------------------------------------- | -------------------- |
| Approved admin creates documented standard user         | Informational or Low |
| Unknown admin creates standard user                     | Medium               |
| Account created outside expected process                | Medium               |
| New account added to privileged group                   | High                 |
| New account authenticates from unusual host immediately | High                 |
| Account created after suspicious script execution       | High                 |
| Multiple hidden or similarly named accounts created     | High or Critical     |

Severity should reflect confidence, privilege, and impact.

---

# Evidence Collection

## Recommended Evidence

Preserve:

* Pre-exercise account search
* Account-creation command or administrative action
* Event ID `4720`
* Related account-change events
* Active Directory account properties
* Group membership output
* Wazuh result
* Splunk timeline
* Cleanup event
* Final account state
* Investigation notes

---

## Export Relevant Windows Events

Example:

```
wevtutil epl Security "<EVIDENCE_PATH>\DC01-Security.evtx"
```

Raw Security logs may contain unrelated sensitive events.

Store privately.

---

## Export Splunk Results

Export the narrow account-creation timeline as:

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
Exercise: EX-03
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

## Preserve Evidence Before Cleanup

Before disabling or deleting the account:

1. Capture the account properties.
2. Capture group membership.
3. Export Event ID `4720`.
4. Save Wazuh evidence.
5. Save the Splunk timeline.
6. Record the current account state.
7. Confirm no other exercise depends on the account.

---

## Disable the Temporary Account

A safer first cleanup step is to disable the account:

```
Disable-ADAccount `
    -Identity "creation.test"
```

---

## Confirm the Disabled State

```
Get-ADUser `
    -Identity "creation.test" `
    -Properties Enabled |
Select-Object `
    SamAccountName,
    Enabled
```

Expected:

```
Enabled: False
```

---

## Confirm Event ID 4725

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4725
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

---

## Delete the Temporary Account

After all evidence is preserved:

```
Remove-ADUser `
    -Identity "creation.test" `
    -Confirm
```

Deletion is permanent unless the object is recoverable through Active Directory recovery features or backup.

---

## Confirm Event ID 4726

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4726
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "creation\.test"
}
```

---

## Confirm the Account No Longer Exists

```
Get-ADUser `
    -Filter 'SamAccountName -eq "creation.test"'
```

Expected:

```
No matching account
```

---

## Final State Record

```
Account created:
Event ID 4720 confirmed:
Wazuh reviewed:
Splunk reviewed:
Unexpected group membership:
Account disabled:
Event ID 4725 confirmed:
Account deleted:
Event ID 4726 confirmed:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* The designated test OU was used.
* An authorized administrator created the account.
* The account was nonprivileged.
* Event ID `4720` appeared on DC01.
* The actor and target were identified.
* Related account events were reviewed.
* Group membership was validated.
* Wazuh was reviewed.
* Splunk was reviewed.
* A timeline was created.
* Ingestion delay was reviewed.
* The account was disabled or deleted.
* Cleanup events were confirmed.
* Evidence was preserved.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                 | Result                     |
| ------------------------ | -------------------------- |
| Event ID 4720 present    | Pass / Fail                |
| Actor identified         | Pass / Partial / Fail      |
| Target identified        | Pass / Partial / Fail      |
| Wazuh ingestion          | Pass / Fail                |
| Wazuh alert quality      | Pass / Needs Tuning / Fail |
| Splunk ingestion         | Pass / Fail                |
| Splunk field quality     | Pass / Needs Tuning / Fail |
| Group membership context | Pass / Partial / Fail      |
| Timestamp accuracy       | Pass / Needs Review / Fail |
| Cleanup events present   | Pass / Partial / Fail      |
| Cleanup                  | Complete / Incomplete      |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Actor account
* Target account
* Target SID
* Organizational unit
* Account enabled state
* Password-setting context
* Group membership context
* Source workstation
* Administrative privilege context
* Clear severity
* Correlation with later authentication
* Correlation with account deletion

---

## Recommended Improvements

Potential improvements include:

* Create a saved account-creation search
* Add privileged-group correlation
* Add approved-administrator lookup
* Add approved-OU context
* Track accounts created outside change windows
* Track new accounts followed by authentication
* Track accounts deleted shortly after creation
* Improve Wazuh field extraction
* Create an identity-change dashboard
* Create a new-account investigation runbook

---

# Troubleshooting

## Event ID 4720 Does Not Appear

Check:

* Account-management auditing
* Correct domain controller
* Security log time range
* Account actually created
* Event log retention
* Administrative access to the log
* Group Policy application

Review:

```
auditpol /get /subcategory:"User Account Management"
```

---

## Account Already Exists

Check:

```
Get-ADUser `
    -Filter 'SamAccountName -eq "creation.test"'
```

Use a new unique test name or remove the stale account after confirming it is not needed.

Do not overwrite an existing identity.

---

## Account Is Created in the Wrong OU

Review:

```
Get-ADUser `
    -Identity "creation.test" `
    -Properties DistinguishedName |
Select-Object `
    SamAccountName,
    DistinguishedName
```

Move the account only after documenting the error:

```
Get-ADUser `
    -Identity "creation.test" |
Move-ADObject `
    -TargetPath "OU=CyberLab-Test-Users,DC=cyberlab,DC=example"
```

Record the corrective action.

---

## Account Is Unexpectedly Privileged

Stop the exercise.

Then:

1. Preserve the membership evidence.
2. Remove the account from the privileged group.
3. Confirm the removal event.
4. Review the creation command or template.
5. Confirm no inherited automation caused the membership.
6. Document the issue.

---

## Wazuh Does Not Show the Event

Check:

* DC01 agent status
* Security log collection
* Decoder
* Rule
* Rule level
* Dashboard time range
* Agent filter
* Indexer health

Confirm Event ID `4720` exists locally before changing Wazuh.

---

## Splunk Does Not Show the Event

Check:

* DC01 forwarder service
* Security event input
* Receiver
* Target index
* Host
* Sourcetype
* Time range
* Event field name
* Forwarder logs

Begin with:

```spl
index=windows host=DC01 earliest=-30m
| stats count by EventCode
```

---

## Actor and Target Are Reversed or Missing

Possible causes include:

* Incorrect field extraction
* Sourcetype mismatch
* Add-on issue
* Wazuh decoder limitation
* Search using generic `user` field only

Review the raw XML before building permanent field mappings.

---

## No Disable or Delete Event Appears

Check:

* Cleanup action actually completed
* Correct event ID
* Security log time range
* Auditing
* Correct domain controller
* Account object still exists

Do not mark cleanup complete until the final state is confirmed.

---

# Public Sanitization

Remove or replace:

* Operational domain name
* Real usernames
* Administrator identities
* Internal IP addresses
* Distinguished names tied to the live environment
* Security identifiers
* Password policy details where sensitive
* Raw event XML
* Wazuh agent identifiers
* Splunk internal URLs
* Evidence paths
* Account descriptions containing personal information

Use placeholders such as:

```
<AUTHORIZED_ADMIN>
<TEST_ACCOUNT>
<DOMAIN_CONTROLLER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<DOMAIN_NAME>
<TEST_OU>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4720`
* Temporary user properties
* Wazuh account-creation alert
* Splunk identity-change timeline
* Group membership validation
* Event ID `4725` or `4726`
* Final cleanup validation

Before publishing:

1. Crop unrelated information.
2. Remove real usernames.
3. Remove operational domain values.
4. Remove SIDs.
5. Remove internal addresses.
6. Hide browser tabs and bookmarks.
7. Permanently redact sensitive values.
8. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
A temporary nonprivileged Active Directory account was created by an authorized administrator as part of CyberLab Exercise EX-03. The account-creation event was reviewed on DC01 and correlated through Wazuh and Splunk. The actor, target account, account properties, group membership, and cleanup events were documented.
```

## Findings

```
Authorized administrator:

Target account:

Target OU:

Creation time:

DC event IDs:

Wazuh result:

Splunk result:

Account enabled:

Group membership:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized creation of a temporary Active Directory user for CyberLab Exercise EX-03.
```

## Resolution

```
The temporary account was reviewed, disabled, and deleted after evidence collection. The related cleanup events were confirmed, and the directory was returned to its intended state.
```

## Recommendations

```
- Correlate account creation with privileged-group membership.
- Add approved-administrator and approved-OU context.
- Track newly created accounts that authenticate immediately.
- Improve actor and target field extraction.
- Create a new-account investigation runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshot exists
* Test OU confirmed
* Temporary account does not exist
* Account-management auditing enabled
* DC01 healthy
* Wazuh healthy
* Splunk healthy
* System time validated
* Start time recorded

## Exercise

* Temporary account created
* Account enabled state confirmed
* Correct OU confirmed
* Account nonprivileged
* Group membership reviewed
* Creation time recorded

## Investigation

* Event ID 4720 reviewed
* Actor identified
* Target identified
* Related events reviewed
* Wazuh reviewed
* Splunk reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Account disabled
* Event ID 4725 reviewed
* Account deleted
* Event ID 4726 reviewed
* Account absence confirmed
* Monitoring systems healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Active Directory administration
* Account-management auditing
* Event ID `4720` interpretation
* Actor and target analysis
* Wazuh validation
* Splunk SPL
* Identity-change correlation
* Group membership review
* Timeline reconstruction
* False-positive analysis
* Evidence handling
* Cleanup validation
* Detection tuning
* Public sanitization

---

# Lessons Learned

* Account creation should be treated as an identity-security event even when authorized.
* The actor and target account must be distinguished clearly.
* Event ID `4720` is the primary Windows account-creation event.
* Related enablement, password, and attribute-change events may provide additional context.
* Group membership should be reviewed immediately after account creation.
* A nonprivileged new account can still create risk if it is unauthorized.
* Wazuh may alert on account creation without correlating later group changes.
* Splunk can combine creation, enablement, authentication, and cleanup into one timeline.
* Approved administrators and OUs provide useful tuning context.
* Evidence should be preserved before deleting the test account.
* Cleanup events are part of the validation, not merely administrative housekeeping.

---

# Summary

This exercise validates the CyberLab’s ability to detect and investigate Active Directory user creation.

A complete investigation should identify:

* The administrator who performed the action
* The account that was created
* The organizational unit
* The account state
* The group memberships
* The related Windows events
* The Wazuh result
* The Splunk result
* The event timeline
* The likely reason for the change
* Any missing telemetry or detection context

The exercise is complete only after the temporary account is removed or disabled, cleanup events are confirmed, evidence is preserved, and detection improvements are documented.

