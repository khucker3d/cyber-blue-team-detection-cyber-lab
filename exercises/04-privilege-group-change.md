# Privileged Group Change

## Exercise Overview

This exercise adds a temporary nonprivileged Active Directory account to an approved privileged or elevated lab group, validates the resulting telemetry, and then removes the account safely.

The investigation focuses on:

* Which administrator changed the group
* Which account was added
* Which group was modified
* Whether the change increased privilege
* Whether the activity was authorized
* Whether Wazuh and Splunk captured the event
* Whether the member was removed successfully
* Whether the detection provided enough context for investigation

The preferred version of this exercise uses a dedicated lab-created elevated group rather than a highly sensitive built-in administrative group.

All activity must remain inside the authorized CyberLab.

---

## Exercise ID

```
EX-04
```

---

## Difficulty

```
Intermediate
```

---

## Primary Skills

* Active Directory group administration
* Privilege-change investigation
* Windows account-management auditing
* Event Viewer analysis
* Wazuh event validation
* Splunk SPL development
* Actor, target, and group correlation
* Detection tuning
* Evidence preservation
* Cleanup verification

---

## Authorization Boundary

This exercise must use:

* An authorized CyberLab administrator
* A disposable nonprivileged test account
* An approved lab group
* DC01
* The internal VMware host-only network
* Wazuh and Splunk monitoring
* Synthetic account and group names

Do not use:

* A personal account
* A school or employer account
* A service account
* A primary recovery account
* An account required by another lab service
* Any system outside the CyberLab

Avoid using these groups unless the exercise specifically requires them and a stable snapshot exists:

* Domain Admins
* Enterprise Admins
* Schema Admins
* Administrators
* Account Operators
* Server Operators
* Backup Operators

---

## Preferred Exercise Design

Use a dedicated elevated lab group such as:

```
CyberLab-Security-Operators
```

This allows the exercise to demonstrate privilege-change monitoring without granting unrestricted domain control.

The group should have only the permissions required for the lab scenario.

---

## Public Example Environment

```
Domain controller: DC01
Windows endpoint: WIN11TARGET
Wazuh server: WAZUH-SERVER
Splunk server: SPLUNK-SERVER
Domain: cyberlab.example
Authorized administrator: student.admin
Temporary member: privilege.test
Target group: CyberLab-Security-Operators
Documentation subnet: 192.0.2.0/24
```

These values are sanitized examples.

---

## Learning Objectives

By the end of the exercise, the student should be able to:

* Review the target group’s purpose and scope
* Confirm the test account is initially nonprivileged
* Add the account to an approved group
* Identify the administrator who performed the change
* Identify the account added to the group
* Identify the group that was modified
* Interpret relevant Windows event IDs
* Validate the activity in Wazuh
* Validate the activity in Splunk
* Correlate addition and removal events
* Assess whether the detection severity is appropriate
* Restore the account to its original state

---

## Required Systems

* DC01
* WAZUH-SERVER
* SPLUNK-SERVER
* Wazuh agent on DC01
* Splunk Universal Forwarder on DC01 where configured

WIN11TARGET is optional.

KALI-TEST is not required.

---

## Required Accounts

### Student User

Used for routine observation.

```
student.user
```

### Authorized Administrator

Used only for group administration and cleanup.

```
student.admin
```

### Disposable Test Account

Temporarily added to the target group.

```
privilege.test
```

The test account must not already be privileged.

---

## Required Monitoring

Before starting, confirm:

* DC01 Security log is active
* Security group management auditing is enabled
* Wazuh agent is active on DC01
* Wazuh dashboard is accessible
* Splunk is running
* DC01 events are searchable
* System time is synchronized
* The target group exists
* A stable snapshot exists

---

## Safety Notes

* Use a dedicated lab group whenever possible.
* Confirm the test account is not already a member.
* Record the account’s original group membership.
* Do not grant broader permissions than required.
* Do not leave the test account elevated after the exercise.
* Preserve evidence before removing the account.
* Confirm the removal event.
* Stop immediately if the wrong account or group is modified.

---

## Expected Event Flow

```
Authorized Administrator
          |
          v
Member Added to Security Group
          |
          v
DC01 Security Event
          |
          v
Wazuh and Splunk Ingestion
          |
          v
Privilege-Change Investigation
          |
          v
Member Removed from Group
          |
          v
Cleanup Event Validation
```

---

## Key Windows Event IDs

The event ID depends on the group scope.

| Event ID | Description                                                  |
| -------- | ------------------------------------------------------------ |
| 4728     | A member was added to a security-enabled global group        |
| 4729     | A member was removed from a security-enabled global group    |
| 4732     | A member was added to a security-enabled local group         |
| 4733     | A member was removed from a security-enabled local group     |
| 4756     | A member was added to a security-enabled universal group     |
| 4757     | A member was removed from a security-enabled universal group |
| 4735     | A security-enabled local group was changed                   |
| 4737     | A security-enabled global group was changed                  |
| 4755     | A security-enabled universal group was changed               |
| 4672     | Special privileges were assigned to a new logon              |
| 4624     | An account successfully logged on                            |

The primary addition and removal event IDs depend on the target group type.

---

## Important Event Fields

Review:

* Subject security ID
* Subject account name
* Subject account domain
* Subject logon ID
* Member name
* Member security ID
* Target group name
* Target group domain
* Target group security ID
* Privilege context
* Timestamp
* Reporting domain controller

---

## Investigation Questions

The final report should answer:

* Which administrator performed the change?
* Which account was added?
* Which group was modified?
* What was the group scope?
* Did the change increase privilege?
* What time did the addition occur?
* Which domain controller recorded the event?
* Was the member name parsed correctly?
* Did Wazuh generate an alert?
* Did Splunk index the event?
* Did the account authenticate while privileged?
* Were any unexpected group changes observed?
* Was the account removed successfully?
* Did Wazuh and Splunk capture the removal?
* What legitimate activity could resemble this event?
* What detection improvements are needed?

---

# Preparation

## Review the Current Snapshot State

Confirm stable snapshots exist for:

* DC01
* WAZUH-SERVER
* SPLUNK-SERVER

A pre-exercise snapshot is recommended.

Suggested name:

```
PRE-EX04-Privileged-Group-Change
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

## Confirm Security Group Management Auditing

Run on DC01:

```
auditpol /get /subcategory:"Security Group Management"
```

The effective policy should record successful group-management events.

When Group Policy controls the setting, review the applicable GPO rather than making an undocumented local change.

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

Confirm recent DC01 events are searchable:

```spl
index=windows host=DC01 earliest=-15m
| head 20
```

---

## Validate the Test Account

```
Get-ADUser `
    -Identity "privilege.test" `
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
* The account is not already in the target group.

---

## Record Original Group Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "privilege.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Export or save the output privately.

This provides the rollback baseline.

---

## Validate the Target Group

```
Get-ADGroup `
    -Identity "CyberLab-Security-Operators" `
    -Properties `
        GroupScope,
        GroupCategory,
        Description,
        Members |
Select-Object `
    Name,
    GroupScope,
    GroupCategory,
    Description
```

Confirm:

* The group exists.
* The purpose is documented.
* The group is security-enabled.
* The group is approved for the exercise.
* The group does not grant unintended permissions.

---

## Confirm the Test Account Is Not Already a Member

```
Get-ADGroupMember `
    -Identity "CyberLab-Security-Operators" |
Where-Object {
    $_.SamAccountName -eq "privilege.test"
}
```

Expected:

```
No matching member
```

---

## Start the Exercise Record

```
Exercise ID: EX-04
Authorized administrator: student.admin
Test account: privilege.test
Target group: CyberLab-Security-Operators
Group scope:
Domain controller: DC01
Start time:
Expected addition event:
Expected removal event:
Expected Wazuh result:
Expected Splunk result:
```

---

# Exercise Procedure

## Add the Test Account to the Group

Run on DC01 using the authorized administrator account:

```
Add-ADGroupMember `
    -Identity "CyberLab-Security-Operators" `
    -Members "privilege.test"
```

Authorized domain permissions are required.

---

## Record the Change Time

Immediately record:

```
Addition time:
Administrator:
Member:
Target group:
Group scope:
Expected privilege increase:
```

---

## Confirm Membership

```
Get-ADGroupMember `
    -Identity "CyberLab-Security-Operators" |
Where-Object {
    $_.SamAccountName -eq "privilege.test"
}
```

Expected result:

```
privilege.test
```

---

## Confirm Effective Group Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "privilege.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Confirm the target group now appears.

---

## Optional Privileged Authentication Validation

This step is optional and should be performed only when the group grants a controlled lab permission.

If used:

1. Sign out of existing sessions.
2. Authenticate with the test account.
3. Access only the approved lab resource.
4. Record the authentication time.
5. Do not perform unrelated administrative actions.
6. Sign out.
7. Preserve related event evidence.

Group membership may not be reflected in an existing access token until the user signs in again.

---

# DC01 Event Validation

## Determine the Correct Addition Event

Use the target group scope:

| Group Scope  | Addition Event | Removal Event |
| ------------ | -------------: | ------------: |
| Global       |           4728 |          4729 |
| Domain Local |           4732 |          4733 |
| Universal    |           4756 |          4757 |

Record the applicable event IDs before searching.

---

## Search for a Global Group Addition

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4728
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

---

## Search for a Domain Local Group Addition

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4732
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

---

## Search for a Universal Group Addition

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4756
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

---

## Event Viewer Procedure

1. Open **Event Viewer**.
2. Navigate to **Windows Logs**.
3. Select **Security**.
4. Choose **Filter Current Log**.
5. Enter the applicable addition event ID.
6. Use a narrow time range.
7. Open the matching event.
8. Review the **General** tab.
9. Review the XML in the **Details** tab.
10. Record the actor, member, and group fields.

---

## Review Group-Addition Event Fields

### Subject

Review:

* Security ID
* Account name
* Account domain
* Logon ID

This identifies the actor.

### Member

Review:

* Member security ID
* Member name

This identifies the account added.

### Group

Review:

* Group security ID
* Group name
* Group domain

This identifies the group that changed.

---

## Record Addition Findings

```
Event ID:
Event time:
Reporting host:
Actor account:
Actor domain:
Actor logon ID:
Member account:
Member SID:
Target group:
Target group domain:
Target group SID:
```

Do not publish operational SIDs.

---

## Search for Related Group-Change Events

Depending on the group scope, review:

```
4735
4737
4755
```

These may provide additional group-change context but do not replace the direct membership event.

---

## Search for Special Privilege Assignment

If the group grants administrative rights and the test account authenticates afterward, review Event ID `4672`:

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4672
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

Event ID `4672` does not appear for every elevated group.

Its absence does not prove the membership change failed.

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
privilege.test
```

Use a narrow time range around the change.

---

## Search by Group Name

Search for:

```
CyberLab-Security-Operators
```

This can help locate events when the member field is not parsed consistently.

---

## Search by Event ID

Review the applicable addition event:

```
4728
4732
4756
```

Also review the corresponding removal event after cleanup:

```
4729
4733
4757
```

---

## Review Wazuh Fields

Record:

* Agent
* Windows event ID
* Rule description
* Rule level
* Actor account
* Member account
* Target group
* Group domain
* Timestamp
* MITRE ATT&CK mapping
* Compliance mappings

---

## Wazuh Validation Questions

* Did Wazuh collect the membership addition?
* Did it generate a visible alert?
* Was the actor parsed correctly?
* Was the member parsed correctly?
* Was the target group parsed correctly?
* Was the rule severity appropriate?
* Did the alert distinguish ordinary and privileged groups?
* Was the event received promptly?
* Did the cleanup removal appear?

---

## Wazuh Detection Gap Record

```
Telemetry present:
Alert present:
Rule:
Rule level:
Actor parsed:
Member parsed:
Group parsed:
Privilege context present:
Missing fields:
Recommended improvement:
```

A generic group-addition alert may need additional context to determine severity.

---

# Splunk Validation

## Search for the Test Account

```spl
index=windows "privilege.test" earliest=-30m
| table _time host source sourcetype EventCode user Message
| sort _time
```

---

## Search for Security Group Additions

```spl
index=windows earliest=-30m
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| search "privilege.test"
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    GroupName
    TargetUserName
    Message
| sort _time
```

Field names depend on the sourcetype and installed Windows add-on.

---

## Search by Target Group

```spl
index=windows earliest=-30m
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| search "CyberLab-Security-Operators"
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    GroupName
    Message
| sort _time
```

---

## Build a Group-Membership Timeline

```spl
index=windows earliest=-30m
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| search "privilege.test"
| eval action=case(
    EventCode=4728, "Added to global group",
    EventCode=4729, "Removed from global group",
    EventCode=4732, "Added to domain local group",
    EventCode=4733, "Removed from domain local group",
    EventCode=4756, "Added to universal group",
    EventCode=4757, "Removed from universal group",
    true(), "Other group event"
)
| table
    _time
    host
    EventCode
    action
    SubjectUserName
    MemberName
    GroupName
    Message
| sort _time
```

---

## Normalize Actor, Member, and Group Fields

```spl
index=windows earliest=-30m
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval actor=coalesce(SubjectUserName, user)
| eval member=coalesce(MemberName, TargetUserName)
| eval target_group=coalesce(GroupName, TargetDomainName)
| table _time host EventCode actor member target_group
| sort _time
```

Review the raw event before relying on normalized fields.

---

## Measure Ingestion Delay

```spl
index=windows earliest=-30m
| search "privilege.test"
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

## Check for Unexpected Additional Changes

```spl
index=windows earliest=-30m
(
    EventCode=4728 OR
    EventCode=4729 OR
    EventCode=4732 OR
    EventCode=4733 OR
    EventCode=4756 OR
    EventCode=4757
)
| stats
    count
    values(MemberName) as members
    values(SubjectUserName) as actors
    by GroupName EventCode
| sort - count
```

Confirm only the intended membership change occurred.

---

## Splunk Validation Questions

* Which host recorded the event?
* Which account performed the change?
* Which account was added?
* Which group was modified?
* Was the group scope clear?
* Were actor, member, and group fields distinguished correctly?
* Was ingestion timely?
* Were duplicate events present?
* Was the removal event collected?
* Could a saved search identify changes to privileged groups reliably?

---

# Privilege Context Validation

## Confirm the Group’s Purpose

Document:

```
Group name:
Group scope:
Group category:
Permissions granted:
Systems affected:
Approved administrators:
Expected members:
```

A group name alone does not prove that it is privileged.

---

## Confirm the Account’s New Membership

```
Get-ADPrincipalGroupMembership `
    -Identity "privilege.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Compare with the pre-exercise baseline.

---

## Review Nested Membership

A group may gain privilege through nested membership.

Check direct group membership:

```
Get-ADGroup `
    -Identity "CyberLab-Security-Operators" `
    -Properties MemberOf |
Select-Object `
    Name,
    MemberOf
```

Review the parent groups carefully.

---

## Review Effective Authorization

Document whether the group grants:

* Local administrator rights
* Remote access
* Log-reading permissions
* Service-management permissions
* File-share access
* SIEM administration
* Delegated directory permissions
* Other elevated capabilities

Do not test unrelated permissions.

---

# Investigation

## Build the Timeline

| Time     | System |            Event ID | Actor         | Member         | Group                       | Analyst Note       |
| -------- | ------ | ------------------: | ------------- | -------------- | --------------------------- | ------------------ |
| `<TIME>` | DC01   | 4728, 4732, or 4756 | student.admin | privilege.test | CyberLab-Security-Operators | Member added       |
| `<TIME>` | Wazuh  |      Alert or event | student.admin | privilege.test | CyberLab-Security-Operators | Telemetry received |
| `<TIME>` | Splunk |       Indexed event | student.admin | privilege.test | CyberLab-Security-Operators | Event searchable   |
| `<TIME>` | DC01   | 4729, 4733, or 4757 | student.admin | privilege.test | CyberLab-Security-Operators | Member removed     |

---

## Root Cause

For this controlled exercise:

```
Authorized temporary addition of a disposable CyberLab account to an approved elevated lab group during Exercise EX-04.
```

---

## Common Legitimate Causes

Privileged or elevated group changes may be legitimate for:

* Administrator onboarding
* Temporary maintenance
* Help-desk delegation
* Backup operations
* Security monitoring
* Application administration
* Incident response
* Approved project access
* Emergency access
* Role changes

---

## Common Suspicious Causes

The activity may be suspicious when:

* An unusual administrator performs the change
* A standard user is added to a highly privileged group
* The change occurs outside an approved window
* The member name resembles an existing administrator
* The account was newly created
* The account authenticates immediately afterward
* The change is followed by remote administration
* The member is removed shortly after suspicious activity
* No approved request exists
* Several accounts are elevated together

---

## Distinguish Benign from Suspicious Activity

Consider:

* Identity of the actor
* Actor privilege
* Target group sensitivity
* Member account age
* Member account role
* Change request
* Time of change
* Source workstation
* Duration of membership
* Authentication activity afterward
* Related PowerShell or process events
* Whether cleanup occurred as expected

---

## False-Positive Analysis

Legitimate tools may change group membership through:

* Active Directory Users and Computers
* PowerShell
* Identity governance platforms
* Privileged access management systems
* Help-desk workflows
* Automation
* Group Policy preferences
* Provisioning scripts
* Emergency access procedures

A strong detection should include group sensitivity, actor approval, and change context.

---

# Detection Development

## Basic Splunk Search

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    GroupName
| sort - _time
```

---

## Monitor Named Privileged Groups

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval target_group=coalesce(GroupName, TargetUserName)
| where target_group IN (
    "Domain Admins",
    "Enterprise Admins",
    "Administrators",
    "CyberLab-Security-Operators"
)
| table
    _time
    host
    EventCode
    SubjectUserName
    MemberName
    target_group
```

Maintain privileged group names through a lookup in a mature implementation.

---

## Detect Unapproved Actors

```spl
index=windows earliest=-24h
(
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval actor=coalesce(SubjectUserName, user)
| eval member=coalesce(MemberName, TargetUserName)
| eval target_group=coalesce(GroupName, TargetDomainName)
| where NOT actor IN (
    "<APPROVED_ADMIN_1>",
    "<APPROVED_ADMIN_2>"
)
| table _time host actor member target_group EventCode
```

---

## Detect Newly Created Account Added to a Privileged Group

```spl
index=windows earliest=-30m
(
    EventCode=4720 OR
    EventCode=4728 OR
    EventCode=4732 OR
    EventCode=4756
)
| eval account=coalesce(TargetUserName, MemberName, SamAccountName)
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

Field normalization may require adjustment.

---

## Detect Short-Lived Privileged Membership

A member added and removed shortly afterward may represent:

* Approved temporary elevation
* Automated access management
* Adversary cleanup
* Test activity

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
| eval member=coalesce(MemberName, TargetUserName)
| eval target_group=coalesce(GroupName, TargetDomainName)
| stats
    values(EventCode) as event_codes
    min(_time) as first_seen
    max(_time) as last_seen
    by member target_group
| eval membership_duration=last_seen-first_seen
| where membership_duration <= <DURATION_SECONDS>
```

Short duration is context, not proof of malicious intent.

---

## Example Wazuh Rule Concept

```xml
<group name="windows,account_management,group_membership,">
  <rule id="<CUSTOM_RULE_ID>" level="9">
    <field name="win.system.eventID">4728|4732|4756</field>
    <description>Account added to a security-enabled group</description>
    <mitre>
      <id>T1098</id>
    </mitre>
  </rule>
</group>
```

This conceptual example should be refined to match:

* Actual decoded field names
* Approved custom rule range
* Privileged group names
* Actor context
* Expected administrative behavior

A generic security-group addition should not automatically be treated as critical.

---

## MITRE ATT&CK Context

This exercise may relate to:

```
T1098 – Account Manipulation
```

Depending on the scenario, privileged group membership may support persistence or privilege escalation.

The mapping provides analytical context.

The authorized exercise does not establish malicious intent.

---

## Detection Severity Guidance

| Pattern                                               | Suggested Context           |
| ----------------------------------------------------- | --------------------------- |
| Approved admin adds user to ordinary group            | Informational or Low        |
| Approved admin adds user to elevated lab group        | Medium                      |
| Unapproved actor modifies elevated group              | High                        |
| New account added to Domain Admins                    | High or Critical            |
| Privileged membership followed by remote access       | High                        |
| Membership added and quickly removed without approval | High investigation priority |
| Several users elevated at once                        | High                        |
| Built-in administrator group modified unexpectedly    | Critical review             |

Severity should reflect group sensitivity, actor legitimacy, and subsequent activity.

---

# Evidence Collection

## Evidence

Preserve:

* Original account group membership
* Target group properties
* Membership-addition command or administrative action
* Addition event
* Wazuh result
* Splunk timeline
* Effective membership after addition
* Optional privileged authentication evidence
* Removal event
* Final membership state
* Investigation notes

---

## Export Relevant Windows Events

Operational example:

```
wevtutil epl Security "<EVIDENCE_PATH>\DC01-Security.evtx"
```

Raw Security logs may contain unrelated sensitive events.

Store privately.

---

## Export Splunk Results

Export the narrow membership timeline as:

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
Exercise: EX-04
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

## Preserve Evidence Before Removal

Before removing the account from the group:

1. Capture the addition event.
2. Capture current group membership.
3. Save Wazuh evidence.
4. Save the Splunk timeline.
5. Record the current access state.
6. Confirm no related investigation is still active.

---

## Remove the Test Account from the Group

Run on DC01:

```
Remove-ADGroupMember `
    -Identity "CyberLab-Security-Operators" `
    -Members "privilege.test" `
    -Confirm
```

Authorized domain permissions are required.

---

## Record the Removal Time

```
Removal time:
Administrator:
Member:
Target group:
Reason:
```

---

## Confirm the Account Is No Longer a Member

```
Get-ADGroupMember `
    -Identity "CyberLab-Security-Operators" |
Where-Object {
    $_.SamAccountName -eq "privilege.test"
}
```

Expected:

```
No matching member
```

---

## Confirm Effective Membership Returned to Baseline

```
Get-ADPrincipalGroupMembership `
    -Identity "privilege.test" |
Select-Object `
    Name,
    GroupScope,
    GroupCategory
```

Compare with the pre-exercise record.

---

## Confirm the Removal Event

### Global Group

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4729
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

### Domain Local Group

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4733
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

### Universal Group

```
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Security"
        Id = 4757
        StartTime = (Get-Date).AddMinutes(-15)
    } |
Where-Object {
    $_.Message -match "privilege\.test"
}
```

---

## End Existing User Sessions

If the test account authenticated while elevated, remove stale privileged access tokens by signing out the account.

A membership removal may not affect an already issued token immediately.

Where required:

* Sign out the test account
* Close remote sessions
* Reauthenticate only after cleanup
* Confirm the elevated access is no longer available

Do not rely only on directory membership output.

---

## Disable or Delete the Test Account

If the account exists only for exercises, disable or delete it after all evidence is preserved.

Disable:

```
Disable-ADAccount `
    -Identity "privilege.test"
```

Delete:

```
Remove-ADUser `
    -Identity "privilege.test" `
    -Confirm
```

Deletion is permanent unless recovered through backup or directory recovery features.

---

## Final State Record

```
Original membership recorded:
Member added:
Addition event confirmed:
Wazuh reviewed:
Splunk reviewed:
Member removed:
Removal event confirmed:
Effective membership restored:
Existing sessions ended:
Test account disabled or deleted:
Evidence preserved:
Cleanup complete:
```

---

# Detection Validation

## Validation Criteria

The exercise is successfully validated when:

* A dedicated nonprivileged test account was used.
* The target group was approved and documented.
* Original membership was recorded.
* The account was added successfully.
* The correct addition event appeared on DC01.
* The actor, member, and group were identified.
* Wazuh was reviewed.
* Splunk was reviewed.
* Privilege context was documented.
* The account was removed from the group.
* The correct removal event appeared.
* Existing privileged sessions were addressed.
* Final membership matched the baseline.
* Evidence was preserved.
* Detection gaps were documented.

---

## Detection Quality Assessment

| Category                | Result                     |
| ----------------------- | -------------------------- |
| Addition event present  | Pass / Fail                |
| Actor identified        | Pass / Partial / Fail      |
| Member identified       | Pass / Partial / Fail      |
| Target group identified | Pass / Partial / Fail      |
| Privilege context       | Pass / Partial / Fail      |
| Wazuh ingestion         | Pass / Fail                |
| Wazuh alert quality     | Pass / Needs Tuning / Fail |
| Splunk ingestion        | Pass / Fail                |
| Splunk field quality    | Pass / Needs Tuning / Fail |
| Removal event present   | Pass / Fail                |
| Timestamp accuracy      | Pass / Needs Review / Fail |
| Cleanup                 | Complete / Incomplete      |

---

## Possible Detection Gaps

Document whether the platforms lacked:

* Actor account
* Member account
* Target group
* Group scope
* Group sensitivity
* Source workstation
* Member account age
* Privilege context
* Approved-change context
* Subsequent authentication
* Removal correlation
* Clear severity

---

## Improvements

Potential improvements include:

* Maintain a privileged-group lookup
* Maintain an approved-administrator lookup
* Correlate account creation with elevation
* Correlate elevation with successful authentication
* Track short-lived privileged membership
* Track out-of-hours changes
* Track changes from unusual source systems
* Improve actor, member, and group field extraction
* Create a privileged-group dashboard
* Create a group-change response runbook

---

# Troubleshooting

## Addition Event Does Not Appear

Check:

* Correct event ID for the group scope
* Security Group Management auditing
* Correct domain controller
* Security log time range
* Membership actually changed
* Event log retention
* Group is security-enabled

Review:

```
auditpol /get /subcategory:"Security Group Management"
```

---

## Account Is Already a Member

Check:

```
Get-ADGroupMember `
    -Identity "CyberLab-Security-Operators" |
Where-Object {
    $_.SamAccountName -eq "privilege.test"
}
```

Do not perform an additional add operation.

Either:

* Remove the stale membership after investigation
* Use another test account
* Restore the original baseline

---

## Wrong Group Was Modified

Stop the exercise immediately.

Then:

1. Preserve the event.
2. Remove the member from the unintended group.
3. Confirm the removal event.
4. Review the command and group name.
5. Validate the account’s effective permissions.
6. Document the error privately.
7. Improve command-review safeguards.

---

## Wrong Account Was Added

Stop immediately.

Then:

1. Preserve evidence.
2. Remove the unintended member.
3. Confirm removal.
4. Review whether the account authenticated.
5. Review effective privileges.
6. Validate the intended account remains unchanged.
7. Document and remediate.

---

## Member Name Appears as a SID

Windows events may record the member using a distinguished name or SID.

Correlate with:

```
Get-ADUser `
    -Identity "privilege.test"
```

Splunk and Wazuh field extraction may require additional normalization.

Do not publish the operational SID.

---

## Account Does Not Gain Expected Access

Possible causes include:

* Existing access token
* Sign-out required
* Nested group behavior
* Group replication
* Permission not actually assigned
* Resource ACL mismatch
* Deny permission
* Group scope issue

Confirm the group’s actual permissions before changing directory membership again.

---

## Access Remains After Removal

Possible causes include:

* Existing logon token
* Cached session
* Open remote session
* Nested membership
* Separate direct permission
* Group Policy delay

Sign out the account and reauthenticate before retesting.

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

Confirm the event exists locally before changing Wazuh.

---

## Splunk Does Not Show the Event

Check:

* DC01 forwarder
* Security event input
* Receiver
* Target index
* Host
* Sourcetype
* Time range
* Field names
* Forwarder logs

Begin with:

```spl
index=windows host=DC01 earliest=-30m
| stats count by EventCode
```

---

## Actor, Member, and Group Fields Are Confused

Possible causes include:

* Generic `user` field
* Incorrect sourcetype
* Add-on issue
* Decoder limitation
* Search normalization error

Review the raw event XML before creating permanent mappings.

---

## Removal Event Does Not Appear

Check:

* Member was actually removed
* Correct event ID for the group scope
* Security log time range
* Auditing
* Correct domain controller
* Forwarder or agent health

Do not mark cleanup complete until both membership and telemetry are validated.

---

# Public Sanitization

Remove or replace:

* Operational domain name
* Real administrator identities
* Real test account names
* Internal IP addresses
* Distinguished names
* Security identifiers
* Real privileged group names when sensitive
* Exact permission assignments
* Wazuh agent identifiers
* Splunk internal URLs
* Evidence paths
* Raw event XML

Use placeholders such as:

```
<AUTHORIZED_ADMIN>
<TEST_ACCOUNT>
<TARGET_GROUP>
<DOMAIN_CONTROLLER>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<DOMAIN_NAME>
<INDEX_NAME>
<EVIDENCE_PATH>
<REDACTED_VALUE>
```

---

## Screenshot Guidance

Suitable sanitized screenshots include:

* Target group properties
* Addition event
* Wazuh group-change alert
* Splunk membership timeline
* Test account membership
* Removal event
* Final cleanup validation

Before publishing:

1. Crop unrelated information.
2. Remove real usernames.
3. Remove operational group names where required.
4. Remove domain values.
5. Remove SIDs.
6. Remove internal addresses.
7. Hide browser tabs and bookmarks.
8. Permanently redact sensitive values.
9. Reopen and inspect the final image.

---

# Exercise Report Template

## Executive Summary

```
A disposable nonprivileged Active Directory account was temporarily added to an approved elevated CyberLab group during Exercise EX-04. The membership-addition event was reviewed on DC01 and correlated through Wazuh and Splunk. The actor, member, target group, privilege context, and cleanup activity were documented.
```

## Findings

```
Authorized administrator:

Test account:

Target group:

Group scope:

Addition time:

Addition event ID:

Wazuh result:

Splunk result:

Privilege granted:

Authentication while elevated:

Removal time:

Removal event ID:

Ingestion delay:

Detection gaps:

False-positive considerations:
```

## Root Cause

```
Authorized temporary group-membership change performed as part of CyberLab Exercise EX-04.
```

## Resolution

```
The test account was removed from the elevated group, existing sessions were ended where required, the removal event was confirmed, and the account’s effective membership was returned to its original state.
```

## Recommendations

```
- Maintain a lookup of privileged groups.
- Correlate account creation with privilege assignment.
- Add approved-administrator context.
- Track short-lived privileged memberships.
- Correlate elevation with subsequent authentication.
- Create a privileged-group-change response runbook.
```

---

# Completion Checklist

## Preparation

* Stable snapshot exists
* Test account validated
* Original membership recorded
* Target group approved
* Group scope recorded
* Test account not already a member
* Security Group Management auditing enabled
* DC01 healthy
* Wazuh healthy
* Splunk healthy
* Start time recorded

## Exercise

* Test account added
* Addition time recorded
* Membership confirmed
* Effective privilege documented
* No unintended group modified
* Optional authentication controlled

## Investigation

* Correct addition event reviewed
* Actor identified
* Member identified
* Group identified
* Privilege context documented
* Wazuh reviewed
* Splunk reviewed
* Timeline created
* Ingestion delay reviewed
* False positives considered
* Detection gaps documented

## Cleanup

* Evidence preserved
* Test account removed from group
* Correct removal event reviewed
* Membership returned to baseline
* Existing sessions ended
* Test account disabled or deleted when appropriate
* Monitoring systems healthy
* Final state documented

---

# Skills Demonstrated

This exercise demonstrates:

* Active Directory group administration
* Security group management auditing
* Privilege-change investigation
* Windows Event Log analysis
* Wazuh validation
* Splunk SPL
* Actor, member, and group correlation
* Nested-group analysis
* Effective-access review
* Timeline reconstruction
* False-positive analysis
* Evidence handling
* Cleanup validation
* Detection tuning
* Public sanitization

---

# Lessons Learned

* Group scope determines the primary Windows addition and removal event IDs.
* The actor, member, and target group must be distinguished clearly.
* A security-group change does not automatically mean the change was privileged.
* Group sensitivity must be documented separately from the event itself.
* Existing user tokens may retain permissions after group removal.
* Nested groups can create privilege that is not obvious from direct membership.
* Wazuh may alert on a group change without understanding the group’s sensitivity.
* Splunk can correlate account creation, privilege assignment, authentication, and removal.
* Short-lived elevation may be legitimate or suspicious depending on context.
* Cleanup must include both directory membership and active sessions.
* Evidence should be preserved before removing the temporary privilege.

---

# Summary

This exercise validates the CyberLab’s ability to detect and investigate an Active Directory group-membership change that increases account privilege.

A complete investigation should identify:

* The administrator who performed the action
* The account that was added
* The group that was modified
* The group scope
* The privilege gained
* The related Windows event
* The Wazuh result
* The Splunk result
* Any authentication performed while elevated
* The removal event
* The final account state
* Any missing detection context

The exercise is complete only after the account is removed from the group, existing elevated sessions are addressed, cleanup telemetry is confirmed, evidence is preserved, and detection improvements are documented.
