# Privileged Group Change Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating additions to, removals from, and modifications involving privileged Windows or Active Directory security groups.

It applies to events involving:

* A user added to a privileged group
* A user removed from a privileged group
* A computer or service account added to an elevated group
* Privileged group properties changed
* Nested group membership that grants elevated access
* Temporary administrative access
* Wazuh or Splunk alerts related to privileged group changes
* Unexpected privilege assignment following account creation
* Privileged access that does not match an approved workflow

The runbook helps the analyst determine:

* Which group changed
* Which account or object was affected
* Who performed the change
* When and where the change occurred
* Whether the change was authorized
* What effective privilege was granted or removed
* Whether the affected account authenticated after the change
* Whether the activity represents routine administration, privilege escalation, persistence, or account compromise
* Whether containment or escalation is required

---

## Runbook ID

```
RB-04
```

---

## Related Exercise

```
EX-04 – Privileged Group Change
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

* Active Directory security groups
* Local Windows security groups
* Administrative groups
* Delegated operational groups
* Remote-management groups
* Backup and recovery groups
* Application administration groups
* Security-tool administration groups
* Nested groups that grant effective privilege
* Temporary or time-limited administrative access

This runbook does not authorize:

* Removing a member before evidence is preserved
* Disabling a critical administrator or service account without impact review
* Modifying production, employer, or school systems
* Treating every group change as malicious
* Testing privileges outside the authorized CyberLab
* Publishing operational group structure without sanitization

---

## Privileged Group Examples

High-risk Active Directory groups may include:

* Domain Admins
* Enterprise Admins
* Schema Admins
* Administrators
* Account Operators
* Server Operators
* Backup Operators
* Group Policy Creator Owners
* DNSAdmins
* Protected Users

Privileged local groups may include:

* Administrators
* Remote Desktop Users
* Remote Management Users
* Backup Operators
* Hyper-V Administrators
* Event Log Readers
* Distributed COM Users

The actual privilege of a group depends on:

* Group scope
* Assigned rights
* Nested membership
* Delegated permissions
* Group Policy
* Application configuration
* Local group mapping
* Resource access

A group name alone does not fully define its risk.

---

## Primary Data Sources

Use the available sources in this order:

1. Domain controller Security logs
2. Local Windows Security logs
3. Active Directory group membership
4. Group-management event records
5. Authentication and privilege events
6. Wazuh events and alerts
7. Splunk group-change searches
8. Creator or administrator workstation telemetry
9. PowerShell and process-creation logs
10. Change-management or access-approval records
11. Privileged access management records where available
12. Asset and identity ownership records

---

## Key Windows Event IDs

### Global Groups

| Event ID | Description                                               |
| -------: | --------------------------------------------------------- |
|     4728 | A member was added to a security-enabled global group     |
|     4729 | A member was removed from a security-enabled global group |
|     4737 | A security-enabled global group was changed               |

### Domain Local Groups

| Event ID | Description                                              |
| -------: | -------------------------------------------------------- |
|     4732 | A member was added to a security-enabled local group     |
|     4733 | A member was removed from a security-enabled local group |
|     4735 | A security-enabled local group was changed               |

### Universal Groups

| Event ID | Description                                                  |
| -------: | ------------------------------------------------------------ |
|     4756 | A member was added to a security-enabled universal group     |
|     4757 | A member was removed from a security-enabled universal group |
|     4755 | A security-enabled universal group was changed               |

### Related Events

| Event ID | Description                                      |
| -------: | ------------------------------------------------ |
|     4720 | A user account was created                       |
|     4726 | A user account was deleted                       |
|     4738 | A user account was changed                       |
|     4741 | A computer account was created                   |
|     4743 | A computer account was deleted                   |
|     4624 | An account successfully logged on                |
|     4625 | An account failed to log on                      |
|     4648 | A logon was attempted using explicit credentials |
|     4672 | Special privileges were assigned to a new logon  |
|     4688 | A new process was created                        |
|     4719 | System audit policy was changed                  |

Not every environment produces every related event.

---

# Alert Intake

## Alert Sources

The investigation may begin from:

* Wazuh privileged-group alert
* Splunk correlation search
* Windows Event Viewer
* Active Directory administrative review
* Identity governance report
* Access certification
* Help-desk or change ticket
* Administrator report
* User account creation alert
* Suspicious authentication alert
* Detection exercise

---

## Minimum Alert Information

Record:

```
Alert time:
Alert source:
Alert name:
Reporting host:
Group name:
Group scope:
Affected member:
Member object type:
Actor account:
Actor domain:
Actor logon ID:
Event ID:
Action:
Alert severity:
Analyst:
```

Do not assume a generic `user` field identifies the actor. It may represent the affected group member.

---

## Initial Analyst Questions

Determine immediately:

* Is the target group privileged?
* What effective access does the group grant?
* Was a member added or removed?
* Is the affected member a user, computer, service account, or group?
* Is the actor authorized to manage the group?
* Is there an approved access request or change record?
* Is the affected account newly created?
* Did the account authenticate after the change?
* Did it receive special privileges?
* Was the change made outside expected hours?
* Was the change reversed quickly?
* Are several privileged groups being modified?
* Is the actor account potentially compromised?

---

# Severity Classification

## Informational

Use informational severity when:

* The activity was part of an approved CyberLab exercise
* A disposable test account was added to an approved lab-only group
* The actor was authorized
* The group does not grant operational domain-wide privilege
* The membership was removed after testing
* No suspicious follow-on activity occurred

---

## Low

Use low severity when:

* The change matches an approved request
* The actor is authorized
* The group grants limited delegated access
* The affected account has a documented need
* The source workstation is expected
* No unusual authentication or privilege use occurred

---

## Medium

Use medium severity when:

* Approval cannot be confirmed immediately
* The group provides meaningful administrative access
* The change occurred outside normal hours
* The actor used an unusual workstation
* The affected account is newly created
* Temporary privilege lacks an expiration
* The membership was added and removed rapidly
* The account authenticated shortly after the change

---

## High

Use high severity when:

* An unauthorized account added a member
* A privileged account was added to a high-risk group
* A newly created account received administrative access
* The affected account authenticated immediately after elevation
* Special privileges were assigned after the change
* Multiple privileged groups were modified
* The actor account is suspicious or compromised
* The membership provides access to domain controllers or security systems
* The change bypassed the approved access process

---

## Critical

Use critical severity when:

* An unauthorized account enters Domain Admins, Enterprise Admins, or an equivalent group
* Multiple domain-wide privileged accounts are created or elevated
* Privilege assignment is followed by confirmed malicious access
* The actor account is a compromised domain administrator
* Group changes are part of an active domain compromise
* Audit, security, or identity controls are disabled
* Privileged access is used for persistence, credential theft, or lateral movement

---

# Initial Triage

## Step 1: Confirm the Group-Change Event

Run on the reporting domain controller as an authorized administrator:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728, 4729, 4732, 4733, 4756, 4757
        StartTime = (Get-Date).AddHours(-1)
    } |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message
```

Search for the group or member:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728, 4729, 4732, 4733, 4756, 4757
        StartTime = (Get-Date).AddHours(-1)
    } |
Where-Object {
    $_.Message -match "<TARGET_GROUP>|<AFFECTED_MEMBER>"
}
```

If the event is not present locally, review:

* Wrong domain controller
* Wrong time range
* Local group change on an endpoint
* Field-extraction issue
* Forwarded or duplicated event
* Stale SIEM event
* Group change recorded under a different event ID

---

## Step 2: Record the Event Fields

Review:

### Subject

* Security ID
* Account name
* Account domain
* Logon ID

This identifies the actor.

### Member

* Member name
* Member ID

This identifies the object added or removed.

### Group

* Group name
* Group domain
* Group security ID

This identifies the group.

### Additional Information

* Privileges
* Expiration time where available

Record:

```
Event time:
Reporting system:
Event ID:
Action:
Actor account:
Actor domain:
Actor logon ID:
Affected member:
Member SID:
Target group:
Group domain:
Group SID:
Expiration time:
```

Store operational SIDs privately.

---

## Step 3: Confirm Current Membership

For an Active Directory group:

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>" |
Select-Object `
    Name,
    SamAccountName,
    ObjectClass,
    DistinguishedName
```

Confirm whether the affected member is currently present:

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>" |
Where-Object {
    $_.SamAccountName -eq "<AFFECTED_MEMBER>"
}
```

Record:

```
Member currently present:
Membership direct:
Member object type:
Membership changed again:
```

---

## Step 4: Review the Group

```
Get-ADGroup `
    -Identity "<TARGET_GROUP>" `
    -Properties `
        GroupScope,
        GroupCategory,
        Description,
        ManagedBy,
        MemberOf,
        Members,
        Created,
        Modified |
Select-Object `
    Name,
    GroupScope,
    GroupCategory,
    Description,
    ManagedBy,
    Created,
    Modified,
    MemberOf
```

Determine:

* Group purpose
* Owner
* Scope
* Security category
* Parent groups
* Whether it is nested into a privileged group
* Whether it is governed by an approved process
* Whether membership should be temporary

---

## Step 5: Review the Affected Member

### User Account

```
Get-ADUser `
    -Identity "<AFFECTED_MEMBER>" `
    -Properties `
        Enabled,
        Created,
        Modified,
        LastLogonDate,
        PasswordLastSet,
        LockedOut,
        MemberOf,
        Description |
Select-Object `
    SamAccountName,
    Enabled,
    Created,
    Modified,
    LastLogonDate,
    PasswordLastSet,
    LockedOut,
    Description,
    MemberOf
```

### Computer Account

```
Get-ADComputer `
    -Identity "<AFFECTED_MEMBER>" `
    -Properties `
        Enabled,
        Created,
        Modified,
        LastLogonDate,
        OperatingSystem,
        MemberOf |
Select-Object `
    Name,
    Enabled,
    Created,
    Modified,
    LastLogonDate,
    OperatingSystem,
    MemberOf
```

### Group Object

```
Get-ADGroup `
    -Identity "<AFFECTED_MEMBER>" `
    -Properties `
        GroupScope,
        GroupCategory,
        MemberOf,
        Members |
Select-Object `
    Name,
    GroupScope,
    GroupCategory,
    MemberOf
```

Determine whether the member is:

```
Standard user
Privileged user
Service account
Computer account
Security group
Temporary account
Test account
Unknown
```

---

## Step 6: Validate the Actor

```
Get-ADUser `
    -Identity "<ACTOR_ACCOUNT>" `
    -Properties `
        Enabled,
        LockedOut,
        LastLogonDate,
        PasswordLastSet,
        MemberOf,
        Description |
Select-Object `
    SamAccountName,
    Enabled,
    LockedOut,
    LastLogonDate,
    PasswordLastSet,
    Description,
    MemberOf
```

Determine:

* Is the actor authorized to manage the group?
* Is the actor privileged?
* Was the actor using an approved administrative account?
* Was the actor recently created, reset, or unlocked?
* Is the actor currently active?
* Does the actor recognize the action?
* Was the actor logged in from an approved administrative system?

---

# Group Risk Assessment

## Evaluate the Group’s Effective Privilege

Review whether the group grants:

* Domain administration
* Local administration
* Group Policy control
* Account-management rights
* Backup and restore rights
* Remote Desktop access
* Remote PowerShell or WinRM access
* Security-tool administration
* Hypervisor administration
* Database administration
* File-server administration
* Application-level administration
* Password or credential management
* Domain controller access

Do not classify privilege based only on the group name.

---

## Review Nested Membership

Determine whether the group is nested into another group:

```
Get-ADGroup `
    -Identity "<TARGET_GROUP>" `
    -Properties MemberOf |
Select-Object `
    Name,
    MemberOf
```

Review effective groups for a user:

```
Get-ADPrincipalGroupMembership `
    -Identity "<AFFECTED_MEMBER>" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Nested membership can grant privilege indirectly.

---

## Review Delegated Permissions

A group may be privileged through delegated permissions even when it is not a standard administrative group.

Review:

* Organizational unit delegation
* Group Policy permissions
* File-system permissions
* Local group assignment
* Application roles
* Security-platform roles
* Remote-management permissions

Document the source of effective privilege.

---

## High-Risk Conditions

Increase severity when:

* The group is protected by AdminSDHolder
* The group controls domain policy
* The group manages users or groups
* The group can reset passwords
* The group grants local administrator rights broadly
* The group controls security monitoring
* The group grants backup or restore rights
* The group permits remote access to critical systems
* A group is nested into a domain-wide administrative group
* Membership has no documented owner or expiration

---

# Related Group Events

## Search for Group Property Changes

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4735, 4737, 4755
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<TARGET_GROUP>"
}
```

Review whether:

* Group description changed
* Group scope changed
* Group type changed
* Group name changed
* Membership-management behavior changed

---

## Search for Membership Removal

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4729, 4733, 4757
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<TARGET_GROUP>|<AFFECTED_MEMBER>"
}
```

A rapid add-and-remove sequence may indicate:

* Temporary approved access
* Test activity
* Mistake correction
* Adversary cleanup
* Short-lived privilege escalation

---

## Search for Other Privileged Group Changes

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728, 4729, 4732, 4733, 4756, 4757
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<ACTOR_ACCOUNT>"
}
```

Determine whether the actor modified multiple groups.

---

# Account-Creation Correlation

## Search for Recent Account Creation

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4720
        StartTime = (Get-Date).AddHours(-24)
    } |
Where-Object {
    $_.Message -match "<AFFECTED_MEMBER>"
}
```

A newly created account receiving privilege quickly requires additional scrutiny.

---

## Review Account Age

```
Get-ADUser `
    -Identity "<AFFECTED_MEMBER>" `
    -Properties Created |
Select-Object `
    SamAccountName,
    Created
```

Record:

```
Account age at time of elevation:
Creation actor:
Creation source:
Approved account purpose:
```

Transition to the user account creation runbook when the account itself is unexpected.

---

# Authentication and Privilege-Use Investigation

## Search for Successful Authentication

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<AFFECTED_MEMBER>"
}
```

Determine whether the member authenticated:

* Before the change
* Immediately afterward
* From the actor’s workstation
* From an unusual source
* To a critical system
* Using a remote administrative logon type

---

## Search for Failed Authentication

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4625
        StartTime = (Get-Date).AddHours(-4)
    } |
Where-Object {
    $_.Message -match "<AFFECTED_MEMBER>"
}
```

Repeated failures before elevation may indicate:

* Administrator troubleshooting
* Account provisioning problems
* Password guessing
* Compromised credential testing

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
    $_.Message -match "<AFFECTED_MEMBER>|<ACTOR_ACCOUNT>"
}
```

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
    $_.Message -match "<AFFECTED_MEMBER>"
}
```

Event ID `4672` after elevation may confirm that the account received a privileged logon token.

---

## Consider Authentication Token Timing

Group membership changes do not always affect an existing sign-in session immediately.

A user may need to:

* Sign out and sign back in
* Obtain a new Kerberos ticket
* Restart a service
* Start a new process
* Reconnect to a resource

An account removed from a group may retain effective access through an existing token until the session is renewed or terminated.

---

## Review Kerberos Tickets Where Appropriate

On the affected endpoint:

```
klist
```

To purge the current user’s cached tickets during authorized recovery:

```
klist purge
```

Do not purge tickets before preserving relevant session evidence.

---

# Actor Workstation Investigation

## Correlate the Actor Logon ID

Search the reporting system for the actor’s logon ID:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "<ACTOR_LOGON_ID>"
}
```

Review related:

* Event ID `4624`
* Event ID `4648`
* Event ID `4688`
* Additional group changes
* Account creation
* Password resets
* Policy changes

---

## Review Process Creation

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4688
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "powershell\.exe|pwsh\.exe|net\.exe|net1\.exe|dsmod\.exe|mmc\.exe"
}
```

Common legitimate tools include:

* Active Directory Users and Computers
* Active Directory Administrative Center
* PowerShell
* `net group`
* `net localgroup`
* Identity-management automation

The tool alone does not determine intent.

---

## Review PowerShell Activity

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-PowerShell/Operational"
        Id = 4103, 4104
        StartTime = (Get-Date).AddHours(-2)
    } |
Where-Object {
    $_.Message -match "Add-ADGroupMember|Remove-ADGroupMember|Add-LocalGroupMember|Remove-LocalGroupMember|<TARGET_GROUP>|<AFFECTED_MEMBER>"
}
```

Review:

* Command
* Script block
* User
* Host application
* Source endpoint
* Additional directory actions

---

## Review Administrative Source

Confirm:

* Hostname
* IP address
* Logged-on administrator
* Approved administrative role
* Host health
* Recent suspicious alerts
* Remote session activity
* Time synchronization

If the actor source is suspicious, investigate or isolate it before trusting additional changes from that system.

---

# Local Group Change Investigation

## Review Local Group Membership

On the affected Windows endpoint:

```
Get-LocalGroupMember `
    -Group "<LOCAL_GROUP>"
```

For the local Administrators group:

```
Get-LocalGroupMember `
    -Group "Administrators"
```

---

## Relevant Local Group Events

Local security-enabled group changes commonly use:

* `4732` for addition
* `4733` for removal
* `4735` for group changes

The events are recorded on the affected endpoint rather than only on a domain controller.

---

## Review Group Policy Influence

Local group membership may be controlled through:

* Group Policy Preferences
* Restricted Groups
* Endpoint-management tools
* Provisioning scripts
* Security baselines
* Local administrative actions

A manual correction may be overwritten by policy.

Review the effective policy before treating a recurring membership as a separate incident.

---

# Wazuh Investigation

## Search by Group

Search for:

```
<TARGET_GROUP>
```

Use a time range beginning before the change and extending through any authentication or cleanup.

---

## Search by Affected Member

```
<AFFECTED_MEMBER>
```

---

## Search by Actor

```
<ACTOR_ACCOUNT>
```

---

## Search by Event ID

Review:

```
4728
4729
4732
4733
4735
4737
4755
4756
4757
4624
4625
4648
4672
4720
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
Actor account:
Actor logon ID:
Affected member:
Target group:
Group domain:
Member SID:
Source system:
Source address:
Timestamp:
MITRE mapping:
```

---

## Wazuh Investigation Questions

* Was the membership event collected?
* Were actor and affected member distinguished?
* Was the target group parsed correctly?
* Was the group recognized as privileged?
* Was account-creation context visible?
* Did authentication follow?
* Was Event ID 4672 visible?
* Was removal or rollback visible?
* Did Wazuh correlate repeated privileged changes?
* Was the severity appropriate?
* Were any fields missing?

---

# Splunk Investigation

## Search for Group Additions and Removals

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| search "<TARGET_GROUP>" OR "<AFFECTED_MEMBER>"
| table
    _time
    host
    EventCode
    SubjectUserName
    SubjectDomainName
    SubjectLogonId
    MemberName
    MemberSid
    TargetUserName
    TargetDomainName
    GroupName
    Message
| sort _time
```

Field names depend on the Windows add-on and sourcetype.

---

## Build a Privileged Membership Timeline

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757 OR
    EventCode=4624 OR
    EventCode=4625 OR
    EventCode=4648 OR
    EventCode=4672
)
| search "<AFFECTED_MEMBER>"
| eval activity=case(
    EventCode=4720, "Account created",
    EventCode=4728 OR EventCode=4732 OR EventCode=4756, "Member added to group",
    EventCode=4729 OR EventCode=4733 OR EventCode=4757, "Member removed from group",
    EventCode=4624, "Successful logon",
    EventCode=4625, "Failed logon",
    EventCode=4648, "Explicit credentials used",
    EventCode=4672, "Special privileges assigned",
    true(), "Related event"
)
| table
    _time
    host
    EventCode
    activity
    SubjectUserName
    MemberName
    GroupName
    TargetUserName
    SourceNetworkAddress
    WorkstationName
    LogonType
| sort _time
```

---

## Detect Changes to High-Risk Groups

Maintain a reviewed lookup of privileged groups.

Example lookup fields:

```
group_name
group_domain
risk_level
group_owner
expected_change_process
```

Conceptual search:

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| eval changed_group=coalesce(GroupName, TargetUserName)
| lookup privileged_groups group_name as changed_group
    OUTPUT risk_level group_owner expected_change_process
| where isnotnull(risk_level)
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    changed_group
    risk_level
    group_owner
```

---

## Detect Newly Created Account Elevated Quickly

```spl
index=windows earliest=-24h
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval account=coalesce(TargetUserName, MemberName)
| stats
    min(eval(if(EventCode=4720, _time, null()))) as creation_time
    min(eval(if(EventCode=4728 OR EventCode=4732 OR EventCode=4756, _time, null()))) as elevation_time
    values(SubjectUserName) as actors
    values(GroupName) as groups
    by account
| where isnotnull(creation_time)
    AND isnotnull(elevation_time)
| eval seconds_to_elevation=elevation_time-creation_time
| where seconds_to_elevation >= 0
    AND seconds_to_elevation <= <TIME_THRESHOLD_SECONDS>
| convert ctime(creation_time) ctime(elevation_time)
```

---

## Detect Elevation Followed by Privileged Logon

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756 OR
    EventCode=4672
)
| eval account=coalesce(MemberName, TargetUserName, SubjectUserName)
| stats
    min(eval(if(EventCode=4728 OR EventCode=4732 OR EventCode=4756, _time, null()))) as elevation_time
    min(eval(if(EventCode=4672, _time, null()))) as privileged_logon_time
    values(GroupName) as groups
    values(host) as hosts
    by account
| where isnotnull(elevation_time)
    AND isnotnull(privileged_logon_time)
| eval seconds_to_privileged_logon=privileged_logon_time-elevation_time
| where seconds_to_privileged_logon >= 0
    AND seconds_to_privileged_logon <= <TIME_THRESHOLD_SECONDS>
| convert ctime(elevation_time) ctime(privileged_logon_time)
```

---

## Detect One Actor Modifying Multiple Privileged Groups

```spl
index=windows earliest=-1h
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| eval actor=coalesce(SubjectUserName, user)
| eval changed_group=coalesce(GroupName, TargetUserName)
| stats
    count as group_changes
    dc(changed_group) as unique_groups
    dc(MemberName) as unique_members
    values(changed_group) as groups
    values(MemberName) as members
    by actor
| where unique_groups >= <GROUP_THRESHOLD>
| sort - unique_groups
```

---

## Detect Rapid Add and Remove

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| eval action=case(
    EventCode=4728 OR EventCode=4732 OR EventCode=4756, "added",
    EventCode=4729 OR EventCode=4733 OR EventCode=4757, "removed"
)
| eval member=coalesce(MemberName, MemberSid)
| eval group=coalesce(GroupName, TargetUserName)
| stats
    min(eval(if(action="added", _time, null()))) as add_time
    max(eval(if(action="removed", _time, null()))) as remove_time
    values(SubjectUserName) as actors
    by member group
| where isnotnull(add_time)
    AND isnotnull(remove_time)
    AND remove_time >= add_time
| eval membership_duration_seconds=remove_time-add_time
| where membership_duration_seconds <= <DURATION_THRESHOLD_SECONDS>
| convert ctime(add_time) ctime(remove_time)
```

Short duration may represent approved temporary access or adversary cleanup.

---

# Investigation Decision Tree

```
Privileged Group Change Alert
             |
             v
Does the event exist locally?
             |
        +----+----+
        |         |
       No        Yes
        |         |
Check host,     Is the group privileged?
time range,           |
parsing               v
                 +----+----+
                 |         |
                No        Yes
                 |         |
Review normal    Increase severity
group workflow         |
                       v
             Is the actor authorized?
                       |
                  +----+----+
                  |         |
                 No        Yes
                  |         |
           Contain and      Is approval
           escalate         documented?
                                  |
                             +----+----+
                             |         |
                            No        Yes
                             |         |
                    Validate purpose   Review affected
                    and effective      member, duration
                    access             and follow-on use
                             |
                             v
              Did the account authenticate or
                 receive special privileges?
                             |
                        +----+----+
                        |         |
                       Yes        No
                        |         |
                 Investigate      Confirm final
                 post-elevation   membership and
                 activity         expiration
```

---

# Common Investigation Scenarios

## Scenario 1: Approved Administrative Access

Indicators:

* Approved access request
* Authorized actor
* Known administrative account
* Expected source workstation
* Appropriate target group
* Documented business purpose
* Access duration matches policy

Response:

* Validate membership
* Confirm least privilege
* Confirm expiration or review date
* Close as authorized activity

---

## Scenario 2: Approved CyberLab Test

Indicators:

* Dedicated disposable account
* Lab-only delegated group
* Exercise record exists
* No real high-risk administrative group used
* Membership removed after validation
* No unexpected activity followed

Response:

* Preserve exercise evidence
* Confirm cleanup
* Classify as authorized test activity

---

## Scenario 3: Temporary Emergency Access

Indicators:

* Incident or recovery ticket
* Time-limited access
* Authorized emergency administrator
* Critical operational need
* Membership removed after the event

Response:

* Confirm approval
* Record start and end time
* Review actions performed
* Confirm access removal
* Schedule post-event review

---

## Scenario 4: Newly Created Account Elevated

Indicators:

* Event ID `4720` shortly before group addition
* Account receives administrative access quickly
* Account authenticates immediately
* Naming or purpose is unclear
* Creator and elevation actor may be the same

Response:

* Escalate
* Use the account creation runbook
* Disable unauthorized account
* Remove privilege
* Review actor and source endpoint
* Review all follow-on activity

---

## Scenario 5: Unauthorized Domain Admin Addition

Indicators:

* No approved request
* Unknown or suspicious actor
* Member added to Domain Admins or equivalent
* Privileged authentication follows
* Remote access or account changes occur

Response:

* Treat as high or critical
* Preserve evidence
* Remove unauthorized membership
* Disable or restrict affected and actor accounts as authorized
* Revoke sessions
* Investigate domain-wide scope

---

## Scenario 6: Service Account Added to an Administrative Group

Indicators:

* Service identity receives interactive or administrative rights
* No documented requirement
* Password controls are weak
* Account is used across several systems

Response:

* Escalate based on scope
* Review service dependency
* Remove unnecessary privilege
* Rotate credentials where compromise is possible
* Apply least privilege

---

## Scenario 7: Rapid Add and Remove

Indicators:

* Membership lasts only minutes
* Account authenticates while elevated
* Actor removes membership afterward
* No normal temporary-access workflow exists

Response:

* Preserve the entire timeline
* Review authentication tokens and actions performed
* Investigate actor and affected account
* Treat removal as possible cleanup rather than resolution

---

## Scenario 8: Group Policy Reapplies Membership

Indicators:

* Manual removal is reversed
* Membership returns after policy refresh
* Restricted Groups or Group Policy Preferences are configured
* Actor appears as system or policy context

Response:

* Identify the controlling policy
* Correct the approved configuration source
* Avoid repeated manual changes
* Validate effective policy after correction

---

# Containment

## When Containment Is Not Required

Containment may not be required when:

* The change is authorized
* Approval is documented
* Actor and source are expected
* Membership grants appropriate access
* Access duration is controlled
* No suspicious follow-on activity exists
* The event is part of an approved lab exercise

Document why containment was unnecessary.

---

## Remove Unauthorized Active Directory Membership

Run from an authorized administrative session:

```
Remove-ADGroupMember `
    -Identity "<TARGET_GROUP>" `
    -Members "<AFFECTED_MEMBER>" `
    -Confirm
```

Confirm:

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>" |
Where-Object {
    $_.SamAccountName -eq "<AFFECTED_MEMBER>"
}
```

Expected:

```
No matching member
```

---

## Remove Unauthorized Local Membership

On the affected endpoint:

```
Remove-LocalGroupMember `
    -Group "<LOCAL_GROUP>" `
    -Member "<AFFECTED_MEMBER>"
```

Confirm:

```
Get-LocalGroupMember `
    -Group "<LOCAL_GROUP>"
```

---

## Account Containment Options

With appropriate authorization:

* Disable the affected account
* Disable or restrict the actor account
* Force credential reset
* Revoke active sessions
* Remove additional unauthorized memberships
* Restrict remote logon
* Invalidate temporary access
* Require reauthentication

Do not disable critical service or recovery accounts without impact review.

---

## Session and Token Containment

Removing group membership may not invalidate existing tokens.

Depending on risk:

* Sign the user out
* End remote sessions
* Restart the affected service
* Purge Kerberos tickets
* Revoke cloud or application sessions
* Reboot the affected endpoint when justified

Preserve session evidence first.

---

## Source Endpoint Containment

Depending on severity:

* Isolate the actor workstation
* Disconnect its host-only adapter
* End unauthorized administrative sessions
* Stop suspicious PowerShell or automation
* Disable compromised scheduled tasks
* Preserve volatile process and network evidence

---

# Eradication

## Unauthorized Membership

* Remove the membership
* Remove related nested privilege
* Review all groups modified by the actor
* Review local group assignments
* Review delegated permissions
* Review temporary access artifacts
* Confirm no policy reintroduces the membership

---

## Compromised Actor Account

* Disable or restrict the actor
* Reset credentials
* Revoke sessions
* Review all directory changes
* Review authentication history
* Review administrative source endpoint
* Review persistence and lateral movement
* Restore account only after scope is understood

---

## Compromised Affected Account

* Disable or restrict the account
* Remove privilege
* Reset credentials
* Revoke sessions
* Review successful authentication
* Review files, systems, and resources accessed
* Review group and account changes performed

---

## Faulty Automation or Policy

* Stop the affected workflow
* Correct the source configuration
* Review all accounts impacted
* Remove unintended access
* Validate in a limited test scope
* Re-enable only after successful verification

---

# Recovery

## Confirm Final Group Membership

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>" |
Select-Object `
    Name,
    SamAccountName,
    ObjectClass
```

For effective membership:

```
Get-ADPrincipalGroupMembership `
    -Identity "<AFFECTED_MEMBER>" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

---

## Confirm the Removal Event

Search for:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4729, 4733, 4757
        StartTime = (Get-Date).AddMinutes(-30)
    } |
Where-Object {
    $_.Message -match "<TARGET_GROUP>|<AFFECTED_MEMBER>"
}
```

---

## Invalidate Existing Access

When required:

* Sign out affected sessions
* Purge Kerberos tickets
* Restart services using the account
* Reconnect remote sessions only after validation
* Recheck application-specific roles

---

## Validate Account Access

For an authorized membership, perform one controlled access validation.

Record:

```
Validation time:
Source system:
Target resource:
Expected privilege:
Access succeeded:
Unexpected privilege:
Event ID 4624:
Event ID 4672:
```

Do not test unauthorized or unnecessary administrative actions.

---

## Confirm Monitoring Health

Verify:

* Wazuh agent active
* Splunk forwarder active
* Domain controller healthy
* Group-change events searchable
* Authentication events searchable
* Removal event searchable
* No continuing unauthorized group changes
* System time synchronized

---

# Escalation Criteria

Escalate immediately when:

* A domain-wide administrative group changes unexpectedly
* The actor is unauthorized
* The actor account may be compromised
* A newly created account gains privilege
* The affected account authenticates immediately after elevation
* Event ID `4672` follows the group change
* Multiple privileged groups are modified
* Several accounts are elevated
* Privilege is added and removed rapidly without an approved process
* Security-monitoring or identity groups are modified
* Existing tokens cannot be invalidated safely
* The activity is followed by lateral movement or persistence
* The analyst cannot determine effective privilege or scope

---

## Escalation Package

Provide:

```
Incident summary:
Severity:
Change time:
Action:
Target group:
Group risk:
Affected member:
Member type:
Member account age:
Actor account:
Actor privilege:
Actor source system:
Actor source address:
Approval record:
Effective privilege:
Authentication after change:
Special privileges assigned:
Additional groups changed:
Containment completed:
Current membership:
Evidence locations:
Outstanding questions:
Recommended next action:
```

---

# Evidence Collection

## Minimum Evidence

Preserve:

* Original alert
* Group-change event
* Current group membership
* Group properties
* Affected member properties
* Actor account properties
* Related account-creation event
* Related authentication events
* Event ID `4672` where present
* Related addition and removal events
* Wazuh results
* Splunk privilege timeline
* Actor workstation process or PowerShell evidence
* Approval or change records
* Containment and recovery actions
* Analyst notes

---

## Export Security Events

```
wevtutil epl `
    Security `
    "<EVIDENCE_PATH>\Security.evtx"
```

Raw Security logs may contain unrelated sensitive data.

Store them privately.

---

## Export a Narrow Event Summary

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4624, 4625, 4648, 4672, 4688, 4720, 4728, 4729, 4732, 4733, 4735, 4737, 4755, 4756, 4757
        StartTime = <START_TIME>
        EndTime = <END_TIME>
    } |
Where-Object {
    $_.Message -match "<TARGET_GROUP>|<AFFECTED_MEMBER>|<ACTOR_ACCOUNT>"
} |
Select-Object `
    TimeCreated,
    Id,
    MachineName,
    Message |
Export-Csv `
    -Path "<EVIDENCE_PATH>\privileged-group-change-summary.csv" `
    -NoTypeInformation
```

---

## Export Group Membership

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>" |
Select-Object `
    Name,
    SamAccountName,
    ObjectClass,
    DistinguishedName |
Export-Csv `
    -Path "<EVIDENCE_PATH>\target-group-membership.csv" `
    -NoTypeInformation
```

This export may contain sensitive directory information and should remain private.

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
Runbook: RB-04
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

* Approved onboarding
* Role change
* Temporary project access
* Emergency administrative access
* Service deployment
* Application administration
* Help-desk troubleshooting
* Group Policy enforcement
* Identity automation
* Privileged access management
* Disaster-recovery testing
* CyberLab exercise
* Correction of an earlier provisioning mistake

A legitimate change should still be reviewed for:

* Authorization
* Least privilege
* Duration
* Ownership
* Expiration
* Effective access
* Correct removal when no longer needed

---

# Detection Gaps

Document whether the investigation lacked:

* Actor account
* Affected member
* Target group
* Group scope
* Group risk classification
* Member object type
* Actor source workstation
* Source address
* Actor logon ID
* Approval context
* Account age
* Nested privilege
* Delegated privilege
* Authentication after elevation
* Special privilege assignment
* Membership duration
* Removal event
* Existing token invalidation
* Policy or automation source

---

# Detection Improvement Recommendations

Potential improvements include:

* Maintain a privileged-group lookup
* Distinguish actor, member, and group fields
* Correlate additions with account creation
* Correlate additions with authentication
* Correlate additions with Event ID `4672`
* Detect rapid add-and-remove sequences
* Detect one actor modifying several groups
* Detect newly created accounts receiving privilege
* Add approved-actor and approved-source lookups
* Add temporary-access expiration context
* Monitor nested group changes
* Monitor local Administrators membership
* Correlate Group Policy-driven changes
* Create a privileged access dashboard
* Review high-risk groups regularly
* Add session-revocation procedures to access removal

---

# MITRE ATT&CK Context

Unauthorized privileged group membership may relate to:

```
T1098 – Account Manipulation
```

Relevant behavior may include:

* Adding an account to a privileged group
* Modifying account privileges
* Establishing persistent access
* Expanding existing account permissions

The mapping should be applied only when the activity is unauthorized or otherwise suspicious.

A legitimate administrative change does not establish adversary behavior.

---

# Documentation and Closure

## Investigation Summary

```
Alert:
Change time:
Action:
Target group:
Group scope:
Group risk:
Affected member:
Member type:
Member account age:
Actor account:
Actor privilege:
Actor source:
Approval record:
Effective privilege:
Nested privilege:
Authentication after change:
Special privilege assignment:
Membership duration:
Removal event:
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
Approved role assignment
Approved temporary access
Approved emergency access
Approved service access
CyberLab test activity
Provisioning error
Group Policy enforcement
Faulty automation
Unauthorized privilege assignment
Compromised administrator
Compromised user account
Persistence attempt
Privilege escalation
Unable to determine
```

---

## Closure Criteria

The case may be closed when:

* The original event is confirmed.
* The actor is identified.
* The affected member is identified.
* The target group and its risk are understood.
* Authorization is confirmed or disproved.
* Effective and nested privilege are assessed.
* Account age and purpose are reviewed.
* Authentication after the change is reviewed.
* Event ID `4672` is reviewed where relevant.
* Related group changes are reviewed.
* Unauthorized membership is removed.
* Existing sessions or tokens are addressed where required.
* Final group membership is validated.
* Monitoring confirms no continuing unauthorized changes.
* Evidence is preserved.
* Detection gaps are documented.
* Final classification is recorded.

---

## Closure Statement

```
The privileged group change was investigated using domain-controller, Active Directory, endpoint, Wazuh, and Splunk telemetry. The actor, affected member, target group, effective privilege, approval context, authentication activity, and related membership changes were reviewed. The activity was classified as <CLASSIFICATION>. Required containment and corrective actions were completed, final membership and active access were validated, monitoring confirmed the intended state, and the case was closed.
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
* Group SIDs
* Member SIDs
* Sensitive group names
* Delegated permission details
* Actor logon IDs
* Raw event XML
* Wazuh agent IDs
* Splunk internal URLs
* Evidence paths

Use placeholders such as:

```
<TARGET_GROUP>
<AFFECTED_MEMBER>
<ACTOR_ACCOUNT>
<ACTOR_SOURCE>
<SOURCE_IP>
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<DOMAIN_NAME>
<LOCAL_GROUP>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Privileged group addition event
* Privileged group removal event
* Current group membership
* Affected member properties
* Wazuh privileged-group alert
* Splunk privilege timeline
* Account-creation-to-elevation correlation
* Event ID `4672`
* Final membership validation

Before publishing:

1. Crop unrelated information.
2. Remove account names.
3. Remove operational addresses.
4. Remove domain and organizational-unit values.
5. Remove SIDs.
6. Remove sensitive group names where required.
7. Remove unrelated directory information.
8. Hide browser tabs and bookmarks.
9. Permanently redact sensitive values.
10. Reopen and inspect the final image.

---

# Analyst Checklist

## Alert Intake

* Alert recorded
* Time and timezone confirmed
* Reporting system identified
* Event ID identified
* Action identified
* Target group identified
* Affected member identified
* Actor identified
* Initial severity assigned

## Event Validation

* Original event confirmed locally
* Actor and member fields distinguished
* Actor logon ID recorded
* Group SID recorded privately
* Member SID recorded privately
* Addition or removal confirmed
* Current membership reviewed

## Group Review

* Group scope identified
* Group category identified
* Group purpose identified
* Group owner identified
* Group risk classified
* Parent groups reviewed
* Nested privilege assessed
* Delegated permissions considered
* Temporary-access expectations reviewed

## Member Review

* Member object type identified
* Member account exists
* Member enabled state reviewed
* Member account age reviewed
* Member purpose reviewed
* Member current groups reviewed
* Member effective privilege reviewed
* Recent account creation checked

## Actor Review

* Actor enabled state confirmed
* Actor lockout state confirmed
* Actor privilege reviewed
* Actor authorization reviewed
* Actor source workstation reviewed
* Actor authentication reviewed
* Actor process activity reviewed
* Actor PowerShell activity reviewed where available
* Other group changes by actor reviewed

## Authentication Review

* Event ID 4624 reviewed
* Event ID 4625 reviewed
* Event ID 4648 reviewed
* Event ID 4672 reviewed
* Authentication after change assessed
* Source system identified
* Logon type reviewed
* Existing token risk assessed

## SIEM Review

* Wazuh event reviewed
* Wazuh field quality assessed
* Splunk membership timeline created
* Creation-to-elevation pattern checked
* Elevation-to-privileged-logon pattern checked
* Multi-group modification pattern checked
* Rapid add-and-remove pattern checked

## Response

* Severity updated
* Containment decision recorded
* Unauthorized membership removed
* Additional unauthorized groups reviewed
* Actor account contained where required
* Affected account contained where required
* Existing sessions or tickets invalidated where required
* Policy or automation corrected
* Final membership validated

## Closure

* Timeline completed
* Evidence preserved
* Evidence hashes recorded
* Root cause documented
* Authorized or incident classification recorded
* Detection gaps documented
* Recommendations recorded
* Closure statement completed

---

# Quick Reference

## Primary Group-Change Search

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728, 4729, 4732, 4733, 4756, 4757
        StartTime = (Get-Date).AddHours(-1)
    }
```

## Primary Membership Check

```
Get-ADGroupMember `
    -Identity "<TARGET_GROUP>"
```

## Effective Membership Check

```
Get-ADPrincipalGroupMembership `
    -Identity "<AFFECTED_MEMBER>"
```

## Primary Splunk Search

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| search "<TARGET_GROUP>" OR "<AFFECTED_MEMBER>"
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    GroupName
    SubjectLogonId
| sort _time
```

## Remove Active Directory Membership

```
Remove-ADGroupMember `
    -Identity "<TARGET_GROUP>" `
    -Members "<AFFECTED_MEMBER>" `
    -Confirm
```

## Immediate Escalation Indicators

```
Domain-wide privileged group
Unauthorized actor
New account elevated rapidly
Immediate privileged authentication
Event ID 4672 after elevation
Multiple privileged groups changed
Rapid add and remove
Compromised administrator
Security-tool administration granted
Continuing unauthorized changes
```

---

# Summary

This runbook provides a repeatable process for investigating privileged Windows and Active Directory group changes.

A complete investigation should determine:

* Which group changed
* Which member was affected
* Who performed the change
* Where the action originated
* Whether the actor was authorized
* What effective privilege was granted or removed
* Whether nested or delegated access exists
* Whether the affected account was newly created
* Whether the account authenticated after elevation
* Whether special privileges were assigned
* Whether the membership was temporary or persistent
* Whether active sessions retained access after removal
* Whether containment was required
* Whether final membership and access are correct

The investigation is complete only after the group change is classified, unauthorized privilege is removed or approved access is validated, active sessions are addressed, evidence is preserved, and detection improvements are documented.

