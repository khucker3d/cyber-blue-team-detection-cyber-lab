# User Account Creation Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating the creation of a Windows or Active Directory user account.

It applies to events involving:

* Windows Event ID `4720`
* Newly created domain accounts
* Newly created local accounts
* Wazuh or Splunk account-creation alerts
* Unexpected administrator activity
* New service or application identities
* Accounts created shortly before privilege assignment
* Accounts created and deleted within a short period
* Accounts created outside approved provisioning workflows

The runbook helps the analyst determine:

* Who created the account
* When and where the account was created
* Whether the account was authorized
* What attributes and group memberships were assigned
* Whether the account was enabled or used
* Whether privilege was granted
* Whether the account was modified, disabled, or deleted
* Whether the activity represents normal administration, persistence, privilege escalation, or another security concern
* Whether containment or escalation is required

---

## Runbook ID

```
RB-03
```

---

## Related Exercise

```
EX-03 – User Account Creation
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
* Local Windows accounts
* Standard user accounts
* Administrative accounts
* Service accounts
* Application identities
* Temporary accounts
* Test accounts
* Accounts created manually or through automation

This runbook does not authorize:

* Deleting an account before evidence is preserved
* Disabling a critical service account without impact review
* Resetting credentials without appropriate approval
* Modifying production, employer, or school environments
* Accessing an account using unknown credentials
* Assuming every new account is malicious

---

## Primary Data Sources

Use the available data sources in this order:

1. Domain controller Security logs
2. Windows Event ID `4720`
3. Active Directory account properties
4. Related account-management events
5. Group-membership events
6. Successful and failed authentication events
7. Wazuh alerts and raw events
8. Splunk account-management searches
9. Administrative workstation telemetry
10. PowerShell and process-creation logs
11. Change-management or provisioning records
12. Service-desk or onboarding records

---

## Key Windows Event IDs

| Event ID | Description                                              |
| -------: | -------------------------------------------------------- |
|     4720 | A user account was created                               |
|     4722 | A user account was enabled                               |
|     4723 | An attempt was made to change an account password        |
|     4724 | An attempt was made to reset an account password         |
|     4725 | A user account was disabled                              |
|     4726 | A user account was deleted                               |
|     4738 | A user account was changed                               |
|     4740 | A user account was locked out                            |
|     4767 | A user account was unlocked                              |
|     4781 | The name of an account was changed                       |
|     4728 | A member was added to a security-enabled global group    |
|     4732 | A member was added to a security-enabled local group     |
|     4756 | A member was added to a security-enabled universal group |
|     4624 | An account successfully logged on                        |
|     4625 | An account failed to log on                              |
|     4648 | A logon was attempted using explicit credentials         |
|     4672 | Special privileges were assigned to a new logon          |
|     4688 | A new process was created                                |

Not every account-creation workflow produces every related event.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh account-creation alert
* Splunk correlation search
* Windows Event Viewer
* Active Directory administrative review
* Identity governance report
* Help-desk ticket
* Administrator report
* User report
* Privileged-group alert
* Authentication alert involving a new account
* Scheduled account inventory review
* Detection exercise

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Reporting host:
New account:
Account domain:
Creator account:
Creator domain:
Creator logon ID:
Creation event ID:
Alert severity:
Analyst:
```

Do not assume that a generic `user` field identifies the newly created account. It may represent the administrator who created it.

---

## Initial Analyst Questions

Determine immediately:

* Is the new account privileged?
* Is the creator authorized to provision accounts?
* Is the account expected?
* Is there an approved onboarding or change record?
* Is the account enabled?
* Has the account authenticated?
* Was the account added to any groups?
* Was the account created outside expected hours?
* Was the account created from an unusual workstation?
* Was the account created and deleted quickly?
* Are several accounts being created?
* Is related suspicious PowerShell or command-line activity present?

---

# Severity Classification

## Informational

Use informational severity when:

* The account was created during an approved CyberLab exercise
* The creator is authorized
* The account is nonprivileged
* The account purpose is documented
* No unexpected authentication or group assignment occurred

---

## Low

Use low severity when:

* An authorized administrator created a standard account
* The account matches an approved provisioning record
* Attributes and group memberships are appropriate
* The source system is expected
* No unusual follow-on activity exists

---

## Medium

Use medium severity when:

* Documentation is incomplete
* The account was created outside expected hours
* The source workstation is unusual
* The account naming pattern is inconsistent
* The account was enabled immediately
* The account authenticated before approval could be confirmed
* Several accounts were created together
* The creator has account-management rights but the business purpose is unclear

---

## High

Use high severity when:

* An unauthorized account created the user
* The new account was added to an elevated group
* The account authenticated immediately after creation
* The account was created by a compromised or unusual administrator
* The account resembles an existing administrator
* The account was hidden, renamed, or deleted shortly afterward
* Creation was followed by remote access or privilege escalation
* A new account was created from an unusual process or source host

---

## Critical

Use critical severity when:

* A new account gained domain-wide administrative privilege
* An unauthorized account was created and used successfully
* Several privileged accounts were created
* Account creation is part of confirmed domain compromise
* Identity or audit controls were disabled
* The activity is followed by persistence, credential access, or lateral movement
* The creator account is confirmed compromised

---

# Initial Triage

## Step 1: Confirm the Creation Event

Run on the reporting domain controller as an authorized administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Search for the new account:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

If the event is not present locally, review:

* Incorrect reporting host
* Wrong time range
* Field-extraction error
* Forwarded or duplicated event
* Local-account creation on another endpoint
* Stale SIEM data

---

## Step 2: Record Event ID 4720 Fields

Review:

### Subject

* Security ID
* Account name
* Account domain
* Logon ID

This identifies the creator.

### New Account

* Security ID
* Account name
* Account domain

This identifies the account created.

### Attributes

Review:

* SAM account name
* Display name
* User principal name
* Home directory
* Home drive
* Script path
* Profile path
* User workstations
* Password-last-set value
* Account expiration
* Primary group ID
* Delegation values
* User account control
* Other account parameters

Record:

```
Creation time:
Reporting domain controller:
Creator account:
Creator domain:
Creator logon ID:
New account:
New account domain:
New account SID:
Account attributes:
```

Do not publish operational SIDs or sensitive directory attributes.

---

## Step 3: Confirm the Account Exists

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties * |
Select-Object `
    SamAccountName,
    UserPrincipalName,
    DisplayName,
    Enabled,
    Created,
    Modified,
    PasswordLastSet,
    PasswordNeverExpires,
    ChangePasswordAtLogon,
    AccountExpirationDate,
    LastLogonDate,
    LockedOut,
    Description,
    DistinguishedName
```

Record:

```
Account exists:
Enabled:
Creation time:
Last modified:
Password last set:
Password never expires:
Password change required:
Expiration date:
Last logon:
Locked:
Description:
Organizational unit:
```

---

## Step 4: Determine Account Type and Purpose

Classify the account as:

```
Standard user
Administrator
Service account
Application account
Temporary account
Test account
Guest or shared account
Unknown
```

Validate its purpose through:

* Approved change request
* Onboarding record
* Service owner
* Application owner
* Administrative notes
* Directory description
* Naming standard
* CyberLab exercise record

Do not rely only on the account name.

---

## Step 5: Validate the Creator

Review the creator account:

```
Get-ADUser `
    -Identity "<CREATOR_ACCOUNT>" `
    -Properties `
        Enabled,
        LockedOut,
        LastLogonDate,
        MemberOf,
        PasswordLastSet |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    LastLogonDate,
    PasswordLastSet,
    MemberOf
```

Determine:

* Is the creator enabled?
* Is the creator privileged?
* Is account creation part of the creator’s role?
* Was the creator logged in from an approved workstation?
* Does the creator recognize the action?
* Was the creator account recently changed or reset?

---

# Account Attribute Review

## Review Standard Properties

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties `
        Enabled,
        Description,
        Department,
        Title,
        Manager,
        Office,
        EmployeeID,
        AccountExpirationDate,
        PasswordNeverExpires,
        ChangePasswordAtLogon,
        CannotChangePassword,
        SmartcardLogonRequired,
        TrustedForDelegation,
        ServicePrincipalName |
Format-List
```

Review whether the values match the account’s intended purpose.

---

## High-Risk Attribute Indicators

Increase concern when:

* `PasswordNeverExpires` is enabled without justification
* The account cannot change its password unexpectedly
* Delegation is enabled
* Service principal names are added unexpectedly
* No expiration exists for a temporary account
* The account description is blank or misleading
* The account resembles a privileged user
* The account is placed in an unusual organizational unit
* Logon restrictions are absent where expected

---

## Review User Account Control

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties UserAccountControl |
Select-Object `
    SamAccountName,
    UserAccountControl
```

The raw value should be interpreted with account properties and directory tools rather than used alone.

---

# Related Account-Management Events

## Search for Account Enablement

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4722
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

An account may be created disabled and enabled later.

---

## Search for Account Modification

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4738
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Review which attributes changed and who changed them.

---

## Search for Password Activity

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4723, 4724
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Determine whether:

* A password was set during provisioning
* A password was reset by another administrator
* Unexpected password changes occurred after creation

---

## Search for Disablement or Deletion

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4725, 4726
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

A quickly deleted account may indicate:

* Test activity
* Provisioning mistake
* Adversary cleanup
* Temporary persistence
* Automated account lifecycle

---

## Search for Account Rename

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4781
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Record both old and new names.

---

# Group-Membership Investigation

## Review Current Group Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "<NEW_ACCOUNT>" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Record:

```
Direct groups:
Privileged groups:
Remote-access groups:
Application groups:
Unexpected groups:
```

---

## Search for Global Group Additions

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

---

## Search for Domain Local Group Additions

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4732
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

---

## Search for Universal Group Additions

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4756
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

If privileged group membership is identified, transition to the privileged group change runbook.

---

## Review Nested Privilege

Direct membership may not show all effective privilege.

Review parent groups:

```
Get-ADPrincipalGroupMembership `
    -Identity "<NEW_ACCOUNT>" |
ForEach-Object {
    Get-ADGroup `
        -Identity $_ `
        -Properties MemberOf |
    Select-Object Name, MemberOf
}
```

Interpret nested groups carefully and avoid publishing operational directory structure.

---

# Authentication Investigation

## Search for Successful Logons

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Determine:

* First successful authentication
* Source system
* Source address
* Logon type
* Authentication package
* Target system
* Whether special privileges were assigned

---

## Search for Failed Logons

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Failures may indicate:

* Provisioning test
* Incorrect initial password
* Automated access attempt
* Unauthorized password guessing

---

## Search for Kerberos Activity

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4768, 4769, 4771
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

Review ticket requests and pre-authentication failures.

---

## Search for Explicit Credential Use

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4648
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

This may reveal the account being used through:

* `runas`
* Scripts
* Administrative utilities
* Remote-management tools
* Applications using explicit credentials

---

## Search for Special Privilege Assignment

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4672
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>"
}
```

A `4672` event may indicate an administrative or otherwise privileged logon.

---

# Creator Workstation Investigation

## Identify the Administrative Source

Event ID `4720` does not always provide the source workstation directly.

Correlate the creator’s logon ID with:

* Event ID `4624`
* Event ID `4648`
* Event ID `4688`
* PowerShell Operational logs
* Sysmon process events
* Administrative tool logs

---

## Correlate the Creator Logon ID

Search for the creator’s logon ID in the Security log:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<CREATOR_LOGON_ID>"
}
```

Review related:

* Successful logon
* Explicit credential use
* Process creation
* Account-management events
* Group changes

---

## Review Process Creation

Search Event ID `4688`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe|dsadd\.exe|net\.exe|net1\.exe|mmc\.exe"
}
```

Possible legitimate creation methods include:

* Active Directory Users and Computers
* Active Directory Administrative Center
* PowerShell
* `net user`
* `dsadd`
* Identity automation
* Provisioning platforms

The tool alone does not determine intent.

---

## Review PowerShell Activity

When PowerShell logging is enabled:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "New-ADUser|Enable-ADAccount|Add-ADGroupMember|<NEW_ACCOUNT>"
}
```

Review:

* Command
* Script block
* User
* Host application
* Source endpoint
* Related account actions

---

# Wazuh Investigation

## Search by New Account

Search for:

```
<NEW_ACCOUNT>
```

Use a time range beginning before account creation and extending through any related authentication or group changes.

---

## Search by Event ID

Review:

```
4720
4722
4723
4724
4725
4726
4738
4781
4728
4732
4756
4624
4625
4648
4672
```

---

## Review Wazuh Fields

Record:

```
Agent:
Event ID:
Rule ID:
Rule description:
Rule level:
Creator account:
New account:
Account domain:
Creator logon ID:
Group name:
Source system:
Source address:
Logon type:
Timestamp:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Was Event ID `4720` collected?
* Were the creator and new account distinguished?
* Did Wazuh identify the account as enabled or disabled?
* Were follow-on modifications visible?
* Were group additions visible?
* Did the account authenticate?
* Was privilege context available?
* Was the alert severity appropriate?
* Were related events correlated?
* Did any field extraction confuse the actor and target?

---

# Splunk Investigation

## Search for Account Creation

```spl
index=windows EventCode=4720 earliest=-24h
| search "<NEW_ACCOUNT>"
| table
    _time
    host
    SubjectUserName
    SubjectDomainName
    SubjectLogonId
    TargetUserName
    TargetDomainName
    SamAccountName
    UserPrincipalName
    DisplayName
    Message
| sort _time
```

Field names depend on the sourcetype and installed Windows add-on.

---

## Build an Account Lifecycle Timeline

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4722 OR
    EventCode=4723 OR
    EventCode=4724 OR
    EventCode=4725 OR
    EventCode=4726 OR
    EventCode=4738 OR
    EventCode=4781
)
| search "<NEW_ACCOUNT>"
| eval activity=case(
    EventCode=4720, "Account created",
    EventCode=4722, "Account enabled",
    EventCode=4723, "Password change attempted",
    EventCode=4724, "Password reset attempted",
    EventCode=4725, "Account disabled",
    EventCode=4726, "Account deleted",
    EventCode=4738, "Account changed",
    EventCode=4781, "Account renamed",
    true(), "Related account event"
)
| table
    _time
    host
    EventCode
    activity
    SubjectUserName
    TargetUserName
    TargetDomainName
    Message
| sort _time
```

---

## Correlate Creation and Group Membership

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| search "<NEW_ACCOUNT>"
| eval activity=case(
    EventCode=4720, "Account created",
    EventCode=4728, "Added to global group",
    EventCode=4732, "Added to domain local group",
    EventCode=4756, "Added to universal group",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    SubjectUserName
    TargetUserName
    MemberName
    GroupName
    Message
| sort _time
```

---

## Correlate Creation and Authentication

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4648 OR
    EventCode=4672
)
| search "<NEW_ACCOUNT>"
| eval activity=case(
    EventCode=4720, "Account created",
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4648, "Explicit credential use",
    EventCode=4672, "Special privileges assigned",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    SubjectUserName
    TargetUserName
    SourceNetworkAddress
    WorkstationName
    LogonType
    ProcessName
| sort _time
```

---

## Detect Creation Followed by Rapid Authentication

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4624
)
| eval account=coalesce(TargetUserName, user)
| stats
    min(eval(if(EventCode=4720, _time, null()))) as creation_time
    min(eval(if(EventCode=4624, _time, null()))) as first_logon_time
    values(SubjectUserName) as creators
    values(SourceNetworkAddress) as source_addresses
    values(LogonType) as logon_types
    by account
| where isnotnull(creation_time)
    AND isnotnull(first_logon_time)
| eval seconds_to_first_logon=first_logon_time-creation_time
| where seconds_to_first_logon >= 0
    AND seconds_to_first_logon <= <TIME_THRESHOLD_SECONDS>
| convert ctime(creation_time) ctime(first_logon_time)
```

Rapid use may be legitimate provisioning or suspicious persistence.

---

## Detect Creation Followed by Privilege Assignment

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval account=coalesce(TargetUserName, MemberName, user)
| stats
    values(EventCode) as event_codes
    values(SubjectUserName) as actors
    values(GroupName) as groups
    min(_time) as first_seen
    max(_time) as last_seen
    by account
| where mvfind(event_codes, "4720") >= 0
    AND (
        mvfind(event_codes, "4728") >= 0
        OR mvfind(event_codes, "4732") >= 0
        OR mvfind(event_codes, "4756") >= 0
    )
| convert ctime(first_seen) ctime(last_seen)
```

Review field extraction and raw events before relying on this correlation.

---

## Detect Multiple Accounts Created by One Actor

```spl
index=windows EventCode=4720 earliest=-1h
| eval creator=coalesce(SubjectUserName, user)
| eval new_account=coalesce(TargetUserName, SamAccountName)
| stats
    count as accounts_created
    dc(new_account) as unique_accounts
    values(new_account) as created_accounts
    values(host) as reporting_hosts
    by creator
| where unique_accounts >= <ACCOUNT_THRESHOLD>
| sort - unique_accounts
```

This may indicate:

* Bulk onboarding
* Automation
* Lab testing
* Unauthorized persistence

---

## Detect Short-Lived Accounts

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4726
)
| eval account=coalesce(TargetUserName, SamAccountName)
| stats
    min(eval(if(EventCode=4720, _time, null()))) as creation_time
    max(eval(if(EventCode=4726, _time, null()))) as deletion_time
    values(SubjectUserName) as actors
    by account
| where isnotnull(creation_time)
    AND isnotnull(deletion_time)
| eval lifetime_seconds=deletion_time-creation_time
| where lifetime_seconds >= 0
    AND lifetime_seconds <= <LIFETIME_THRESHOLD_SECONDS>
| convert ctime(creation_time) ctime(deletion_time)
```

Short-lived accounts require context and are not automatically malicious.

---

# Investigation Decision Tree

```
User Account Creation Alert
          |
          v
Does Event ID 4720 exist locally?
          |
     +----+----+
     |         |
    No        Yes
     |         |
Check host,   Is the creator authorized?
time range,         |
parsing             v
               +----+----+
               |         |
              No        Yes
               |         |
        Raise severity   Is an approved
        and contain      record available?
                              |
                         +----+----+
                         |         |
                        No        Yes
                         |         |
                  Validate account  Review attributes,
                  purpose and use   groups and usage
                         |
                         v
              Was privilege assigned?
                         |
                    +----+----+
                    |         |
                   Yes        No
                    |         |
             Escalate or use  Did the account
             privileged-group authenticate?
             runbook               |
                              +----+----+
                              |         |
                             Yes        No
                              |         |
                       Review source,   Validate lifecycle,
                       logon type and   ownership and
                       subsequent use   intended state
```

---

# Common Investigation Scenarios

## Scenario 1: Approved Employee or Student Account

Indicators:

* Approved provisioning record
* Authorized creator
* Standard naming convention
* Expected organizational unit
* Appropriate group membership
* No unexpected privilege
* First logon aligns with onboarding

Response:

* Validate attributes and access
* Record as authorized
* Close after evidence and field quality are reviewed

---

## Scenario 2: Approved CyberLab Test Account

Indicators:

* Exercise record exists
* Synthetic account name
* Dedicated test organizational unit
* Authorized student administrator
* Limited privilege
* Account disabled or deleted after testing

Response:

* Confirm cleanup
* Preserve exercise evidence
* Classify as authorized test activity

---

## Scenario 3: New Service Account

Indicators:

* Application or service owner identified
* Account purpose documented
* Restricted logon rights
* Appropriate password-management controls
* No interactive use
* Limited group membership

Response:

* Confirm dependency and owner
* Review password-expiration policy
* Review service principal names
* Confirm least privilege
* Monitor initial authentication

---

## Scenario 4: Unauthorized Standard Account

Indicators:

* No provisioning record
* Creator not authorized
* Account naming is unusual
* Account enabled immediately
* Authentication follows creation
* Source workstation is unexpected

Response:

* Escalate
* Preserve evidence
* Disable the account
* Investigate the creator
* Review successful authentication
* Review related account and group changes

---

## Scenario 5: Unauthorized Privileged Account

Indicators:

* Added to administrative group
* Creator account is unusual or compromised
* Immediate privileged logon
* Remote access follows
* Account resembles an existing administrator

Response:

* Treat as high or critical
* Disable or contain the account
* Revoke sessions
* Remove unauthorized privilege
* Investigate the creator and source endpoint
* Review domain-wide scope

---

## Scenario 6: Account Created and Deleted Quickly

Indicators:

* Event IDs `4720` and `4726`
* Short account lifetime
* Authentication or group changes occurred in between
* Creator attempts to remove evidence

Response:

* Preserve all lifecycle events
* Review successful logons and resource access
* Investigate the creator account
* Escalate when authorization is absent

---

## Scenario 7: Bulk Account Creation

Indicators:

* One creator creates several accounts
* Accounts share naming pattern
* Activity occurs in a short window
* Automation or onboarding may be involved

Response:

* Validate provisioning batch
* Confirm approved source and script
* Compare count with expected records
* Escalate unexpected or privileged accounts

---

# Containment

## When Containment Is Not Required

Containment may not be required when:

* The account is authorized
* The creator is authorized
* Account purpose is documented
* Attributes and group memberships are correct
* No suspicious use occurred
* The account was created during an approved exercise

Document why no containment was necessary.

---

## Account Containment Options

With appropriate authorization:

* Disable the new account
* Remove unauthorized group membership
* Reset or invalidate credentials
* Set an account expiration
* Restrict logon rights
* Revoke active sessions
* Remove service principal names
* Delete the account after evidence and dependency review

Do not delete the account immediately when compromise is suspected. Deletion can remove useful directory context and disrupt evidence collection.

---

## Disable the Account

```
Disable-ADAccount `
    -Identity "<NEW_ACCOUNT>"
```

Confirm:

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties Enabled |
Select-Object `
    SamAccountName,
    Enabled
```

---

## Remove Unauthorized Group Membership

```
Remove-ADGroupMember `
    -Identity "<TARGET_GROUP>" `
    -Members "<NEW_ACCOUNT>" `
    -Confirm
```

Record and validate the corresponding removal event.

---

## Source Endpoint Containment

Depending on severity:

* Isolate the creator workstation
* End unauthorized administrative sessions
* Stop suspicious scripts
* Disable scheduled automation
* Preserve PowerShell and process logs
* Restrict the creator account
* Disconnect the VM from the host-only adapter where required

Preserve volatile evidence before stopping processes when possible.

---

# Eradication

## Unauthorized Account

* Disable the account
* Remove unauthorized group membership
* Revoke active sessions
* Review created files and resources
* Review directory changes
* Review mailbox, application, or service access where applicable
* Delete the account only after evidence and dependency review

---

## Compromised Creator Account

* Disable or restrict the creator
* Reset credentials
* Revoke sessions
* Review successful authentication
* Review all account-management actions
* Review group changes
* Review endpoint process activity
* Review persistence and lateral movement

---

## Faulty Automation

* Stop or disable the provisioning workflow
* Correct the account template
* Remove unauthorized accounts
* Validate intended accounts
* Review credentials and permissions used by automation
* Test safely before re-enabling

---

## Provisioning Error

* Correct attributes
* Move the account to the correct organizational unit
* Remove unintended groups
* Apply expiration where needed
* Confirm owner and purpose
* Document the correction

---

# Recovery

## Authorized Account Recovery

When the account is valid:

* Correct attributes
* Apply required group membership
* Confirm password controls
* Confirm least privilege
* Enable only when appropriate
* Validate one controlled authentication
* Confirm monitoring visibility

---

## Enable the Account

Run only after authorization and validation:

```
Enable-ADAccount `
    -Identity "<NEW_ACCOUNT>"
```

Confirm:

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties Enabled |
Select-Object `
    SamAccountName,
    Enabled
```

---

## Validate Authentication

Perform one approved authentication.

Record:

```
Validation time:
Source system:
Logon type:
Authentication successful:
Event ID 4624:
Special privileges assigned:
Unexpected failures:
```

---

## Validate Final Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "<NEW_ACCOUNT>" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Confirm only approved memberships remain.

---

## Confirm Monitoring Health

Verify:

* Wazuh agent active
* Splunk forwarder active
* Domain controller healthy
* Account-management events searchable
* Authentication events searchable
* System time synchronized
* No continued unauthorized account creation

---

# Escalation Criteria

Escalate immediately when:

* The creator is unauthorized
* The creator account may be compromised
* The new account is privileged
* The account authenticated unexpectedly
* The account was created from an unusual workstation
* Several accounts were created unexpectedly
* The account resembles an existing administrator
* The account was renamed or deleted rapidly
* Account creation is followed by group changes
* Account creation is followed by remote access
* Audit or identity controls were modified
* The analyst cannot determine scope or purpose

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
Creation time:
New account:
Account type:
Account enabled:
Creator account:
Creator privilege:
Creator source system:
Creator source address:
Approved record:
Current group membership:
Privileged membership:
First authentication:
Successful logons:
Failed logons:
Related account changes:
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
* Event ID `4720`
* Account properties
* Creator account properties
* Related `4722`, `4738`, `4725`, `4726`, and `4781` events
* Group-membership events
* Authentication events
* Wazuh results
* Splunk lifecycle timeline
* Creator source-system evidence
* PowerShell or process evidence
* Containment and recovery actions
* Analyst notes

---

## Export Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

Raw Security logs may contain unrelated sensitive information.

Store them privately.

---

## Export a Narrow Event Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624, 4625, 4648, 4672, 4720, 4722, 4723, 4724, 4725, 4726, 4728, 4732, 4738, 4756, 4781
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Where-Object {
    $_.Message -match "<NEW_ACCOUNT>|<CREATOR_ACCOUNT>"
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\user-account-creation-summary.csv" `
    -NoTypeInformation
```

---

## Export Account Properties

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties * |
Select-Object * |
Export-Clixml `
    -Path "<EVIDENCE_PATH>\new-account-properties.xml"
```

This export may contain sensitive directory information and must remain private.

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
Runbook: RB-03
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

Common legitimate causes include:

* New employee or student onboarding
* Approved service-account provisioning
* Application deployment
* Temporary project account
* Help-desk activity
* Identity automation
* Migration
* Disaster-recovery preparation
* Lab exercise
* Account recreated after a provisioning error
* Bulk import
* Approved administrative testing

A legitimate account should still be reviewed for appropriate ownership, attributes, expiration, and least privilege.

---

# Detection Gaps

Document whether the investigation lacked:

* Creator account
* New account
* Creator logon ID
* Source workstation
* Source address
* Account enabled state
* Organizational unit
* Account purpose
* Password controls
* Group memberships
* Privilege context
* First authentication
* Account lifetime
* Related process
* Related PowerShell command
* Approved-provisioning context
* Correlation across lifecycle events

---

# Detection Improvement Recommendations

Potential improvements include:

* Distinguish actor and target fields
* Correlate creation with enablement
* Correlate creation with group assignment
* Correlate creation with first authentication
* Correlate creation with special privileges
* Alert on creation outside approved workflows
* Add privileged-group lookup
* Add approved-creator lookup
* Add account-age fields
* Detect lookalike administrator names
* Detect short-lived accounts
* Detect bulk creation
* Detect creation followed by remote access
* Detect creation followed by rapid deletion
* Create an account lifecycle dashboard
* Create a privileged group change runbook

---

# MITRE ATT&CK Context

Unauthorized account creation may relate to:

```
T1136 – Create Account
```

For Active Directory accounts, the relevant sub-technique may be:

```
T1136.002 – Domain Account
```

Local account creation may relate to:

```
T1136.001 – Local Account
```

The mapping should be applied only when the activity is unauthorized or otherwise suspicious.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Creation time:
New account:
Account type:
Account enabled:
Account purpose:
Creator account:
Creator privilege:
Creator source:
Approved record:
Organizational unit:
Password controls:
Current groups:
Privileged groups:
First authentication:
Successful logons:
Failed logons:
Related changes:
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
Approved onboarding
Approved service account
Approved application account
Approved temporary account
CyberLab test activity
Provisioning error
Faulty automation
Unauthorized account creation
Compromised administrator
Persistence attempt
Privilege escalation
Bulk provisioning
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* Event ID `4720` is confirmed.
* The new account is identified.
* The creator is identified.
* The creator’s authorization is validated.
* Account purpose and owner are known.
* Account attributes are reviewed.
* Current group membership is reviewed.
* Privilege is assessed.
* Authentication activity is reviewed.
* Related lifecycle events are reviewed.
* The creator source is investigated where required.
* Unauthorized access is contained.
* The account is retained, corrected, disabled, or deleted appropriately.
* Monitoring confirms no continuing suspicious activity.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The user account creation was investigated using domain-controller, Active Directory, endpoint, Wazuh, and Splunk telemetry. The new account, creator, account attributes, group memberships, authentication activity, and related lifecycle events were reviewed. The activity was classified as <CLASSIFICATION>. Required containment and corrective actions were completed, the account was retained, corrected, disabled, or removed as appropriate, monitoring confirmed the final state, and the case was closed.
```

---

# Public Sanitization

Remove or replace:

* Real account names
* Administrative identities
* Service-account names
* Internal IP addresses
* Domain names
* Hostnames where sensitive
* Distinguished names
* Organizational unit structure
* SIDs
* Group memberships
* Service principal names
* Employee identifiers
* Creator logon IDs
* Raw event XML
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths

Use placeholders such as:

```
<NEW_ACCOUNT>
<CREATOR_ACCOUNT>
<CREATOR_SOURCE>
<SOURCE_IP>
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<DOMAIN_NAME>
<TARGET_GROUP>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Event ID `4720`
* New account properties
* Creator account context
* Account lifecycle timeline
* Group-membership event
* Wazuh account-creation alert
* Splunk creation and authentication correlation
* Disabled-account validation
* Final group membership

Before publishing:

1. Crop unrelated information.
2. Remove account names.
3. Remove internal addresses.
4. Remove domain and organizational-unit values.
5. Remove SIDs.
6. Remove sensitive group names.
7. Remove employee or service identifiers.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Reporting domain controller identified
* New account identified
* Creator account identified
* Initial severity assigned

## Creation Validation

* Event ID 4720 confirmed locally
* Actor and target fields distinguished
* Creator logon ID recorded
* Account SID recorded privately
* Account creation time validated

## Account Review

* Account existence confirmed
* Account type identified
* Account purpose identified
* Account owner identified
* Enabled state reviewed
* Password controls reviewed
* Expiration reviewed
* Description reviewed
* Organizational unit reviewed
* Delegation reviewed
* Service principal names reviewed

## Creator Review

* Creator enabled state confirmed
* Creator lockout state confirmed
* Creator privilege reviewed
* Creator authorization reviewed
* Creator source workstation investigated
* Creator authentication reviewed
* Creator process activity reviewed where available

## Lifecycle Correlation

* Event ID 4722 reviewed
* Event ID 4723 reviewed where relevant
* Event ID 4724 reviewed where relevant
* Event ID 4738 reviewed
* Event ID 4725 reviewed
* Event ID 4726 reviewed
* Event ID 4781 reviewed
* Account lifetime assessed

## Privilege Review

* Current groups reviewed
* Event ID 4728 reviewed
* Event ID 4732 reviewed
* Event ID 4756 reviewed
* Nested privilege assessed
* Unauthorized membership removed where required

## Authentication Review

* Event ID 4624 reviewed
* Event ID 4625 reviewed
* Event ID 4648 reviewed
* Event ID 4672 reviewed
* First successful logon identified
* Source system identified
* Logon type reviewed
* Rapid-use pattern assessed

## SIEM Review

* Wazuh event reviewed
* Wazuh field quality assessed
* Splunk lifecycle timeline created
* Creation-to-authentication correlation reviewed
* Creation-to-privilege correlation reviewed
* Bulk account creation checked
* Short-lived account pattern checked

## Response

* Severity updated
* Containment decision recorded
* Account disabled where required
* Unauthorized groups removed
* Creator contained where required
* Account corrected or removed appropriately
* Monitoring health confirmed

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

## Primary Creation Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddHours(-1)
    }
```

## Primary Account Check

```
Get-ADUser `
    -Identity "<NEW_ACCOUNT>" `
    -Properties Enabled, Created, Modified, PasswordLastSet, LastLogonDate, MemberOf
```

## Primary Splunk Search

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4722 OR
    EventCode=4738 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756 OR
    EventCode=4624
)
| search "<NEW_ACCOUNT>"
| table
    _time
    host
    EventCode
    SubjectUserName
    TargetUserName
    MemberName
    GroupName
    SourceNetworkAddress
    LogonType
| sort _time
```

## Disable Command

```
Disable-ADAccount `
    -Identity "<NEW_ACCOUNT>"
```

## Immediate Escalation Indicators

```
Unauthorized creator
Privileged new account
Rapid privileged group assignment
Immediate successful authentication
Unknown source workstation
Multiple unexpected accounts
Lookalike administrator name
Rapid rename or deletion
Remote access after creation
Compromised creator account
```

---

# Summary

This runbook provides a repeatable process for investigating Windows and Active Directory user account creation.

A complete investigation should determine:

* Which account was created
* Who created it
* Where the action originated
* Whether the creator was authorized
* Why the account was created
* Which attributes were assigned
* Which groups were assigned
* Whether the account gained privilege
* Whether the account authenticated
* Whether the account was changed, disabled, renamed, or deleted
* Whether the activity was legitimate or suspicious
* Whether containment was required
* Whether the final account state is appropriate

The investigation is complete only after the account lifecycle is understood, unauthorized access is contained, the account is retained, corrected, disabled, or removed appropriately, evidence is preserved, and detection improvements are documented.

