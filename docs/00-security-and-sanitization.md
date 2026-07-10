# Security and Sanitization

## Overview

The Acer Blue Team CyberLab contains internal infrastructure details, authentication systems, monitoring platforms, endpoint telemetry, test accounts, security logs, network information, and recovery data.

Some of this information is useful for learning and portfolio documentation. Other information should remain private because it could expose:

* Internal network structure
* Administrative access paths
* Usernames
* Authentication data
* Security-tool configuration
* Agent enrollment information
* Backup locations
* Service versions
* System identifiers
* Operational weaknesses
* Personal information

This document defines the security controls and sanitization process used before CyberLab information is published to GitHub, LinkedIn, a portfolio, or another public platform.

The goal is to demonstrate technical capability without exposing operational details that would weaken the lab or reveal personal information.

---

## Security Objectives

The CyberLab security and sanitization process is designed to:

* Keep the lab isolated from untrusted networks
* Limit administrative access
* Protect credentials and secrets
* Prevent accidental public exposure
* Preserve useful technical documentation
* Separate private operational records from public portfolio content
* Reduce information leakage through screenshots
* Protect logs, packet captures, and evidence
* Maintain recoverable backups
* Support ethical and authorized testing
* Review repository content before publication

---

## Security Model

The CyberLab uses a defense-in-depth approach.

```text
Physical Host Security
        |
        v
VMware Isolation
        |
        v
Host-Only Network Boundary
        |
        v
Guest Operating System Security
        |
        v
Identity and Access Controls
        |
        v
Application Security
        |
        v
Logging and Monitoring
        |
        v
Backup and Recovery
        |
        v
Public Sanitization Review
```

No single control is considered sufficient.

---

## Public and Private Documentation

The project should maintain separate documentation layers.

### Private Operational Documentation

May include:

* Real IP addresses
* Actual domain names
* Administrative usernames
* Service ports
* VM names
* Internal URLs
* Backup locations
* Snapshot names
* Agent identifiers
* License details
* Troubleshooting output
* Recovery information

Private documentation may be stored in:

* Confluence
* Encrypted local notes
* Private repositories
* Password manager secure notes
* Protected backup storage

### Public Portfolio Documentation

Should include:

* Architecture concepts
* Sanitized examples
* Technical goals
* Setup methodology
* Validation procedures
* Troubleshooting methods
* Lessons learned
* Skills demonstrated
* Redacted screenshots
* Placeholder configurations

Public documentation should not contain values that provide direct access to the operational environment.

---

## Sanitization Boundary

Before information crosses from the private environment into the public repository, it should pass through a review boundary.

```text
Private CyberLab
      |
      | Raw configuration, logs, screenshots, notes
      v
Sanitization Review
      |
      | Remove, replace, crop, redact, verify
      v
Public Repository
```

Sanitization is a required project stage, not an optional final cleanup.

---

## Data Classification

A simple classification system can help determine what may be published.

### Public

Safe for release after normal review.

Examples:

* General architecture
* Tool purpose
* High-level workflows
* Synthetic examples
* Documentation-only IP ranges
* Sanitized diagrams
* General lessons learned

### Internal

Useful for operation but not intended for public release.

Examples:

* Real hostnames
* Internal addresses
* VM network details
* Private troubleshooting notes
* Snapshot register
* Service startup commands tied to operational paths

### Confidential

Information that could meaningfully increase risk if exposed.

Examples:

* Administrative usernames
* Security event exports
* Agent identifiers
* Internal DNS configuration
* Firewall rules
* Backup locations
* Private repository details

### Secret

Information that must not be published.

Examples:

* Passwords
* API tokens
* Private keys
* Enrollment keys
* Recovery keys
* Session cookies
* Authentication files
* License files
* Password-manager exports

---

## Documentation-Safe Values

Use reserved example values in public documentation.

### Example IPv4 Network

```text
192.0.2.0/24
```

### Example Domain

```text
cyberlab.example
```

### Example Hostnames

```text
DC01
WIN11TARGET
WAZUH-SERVER
SPLUNK-SERVER
KALI-TEST
```

### Example Accounts

```text
student.user
student.admin
validation.user
test.user01
```

These values should not reveal the actual operational environment.

---

## Placeholder Standard

Use consistent placeholders such as:

```text
<DOMAIN_CONTROLLER>
<WINDOWS_ENDPOINT>
<WAZUH_SERVER>
<SPLUNK_SERVER>
<KALI_SYSTEM>
<DOMAIN_NAME>
<INTERNAL_IP>
<ADMIN_USER>
<TEST_ACCOUNT>
<PORT>
<INDEX_NAME>
<BACKUP_PATH>
<REDACTED_VALUE>
```

Consistent placeholders make public documentation easier to understand.

---

## Information That Must Be Removed

Remove or replace:

* Real internal IP addresses
* Real public IP addresses
* Internal domain names
* Personal hostnames
* Personal usernames
* Email addresses
* Passwords
* Password hints
* API tokens
* Session cookies
* SSH private keys
* Certificate private keys
* Enrollment passwords
* Wazuh agent keys
* Splunk credentials
* License identifiers
* Recovery keys
* MAC addresses
* VMware UUIDs
* Device serial numbers
* Backup locations
* NAS addresses
* Personal Windows paths
* Home-directory usernames
* VPN configuration
* Wireless network names
* Browser profile information
* Unrelated notifications
* Personal documents
* Employer data

---

## Information That Requires Review

Some information may be publishable only after context and risk review.

Examples include:

* Software versions
* Open ports
* Firewall rules
* Service names
* Operating system build numbers
* Detection queries
* Directory structure
* Event identifiers
* Vulnerability findings
* Troubleshooting output
* Dashboard screenshots
* Packet metadata
* Installed applications
* Domain structure
* Group Policy names

The decision should consider whether the information is necessary to demonstrate the project.

---

## Software Version Disclosure

Publishing software versions can demonstrate current technical work, but it can also reveal whether a system is outdated.

Before publishing a version:

* Confirm the value is necessary.
* Confirm no active credential or license information appears.
* Avoid pairing an exact version with operational access details.
* Avoid publishing known unpatched weaknesses in the live environment.
* Prefer a general version description when precision adds little value.

Example:

```text
A supported Splunk Enterprise release
```

instead of exposing every build detail.

---

## Port Disclosure

Port numbers may be documented when they are central to the technical explanation.

Examples include:

* Splunk Web
* Wazuh agent communication
* DNS
* SSH
* HTTPS

However, the public repository should not combine:

* Real IP address
* Open port
* Administrative service
* Authentication method

Use placeholders where practical.

---

## Credential Security

Credentials should never be placed directly in:

* GitHub
* Markdown documents
* Screenshots
* Shell history
* PowerShell history
* Configuration examples
* Packet captures
* Browser bookmarks
* Public issue trackers
* Commit messages
* Diagram labels

Use interactive prompts and password-manager storage.

---

## Password Reuse

Do not reuse passwords between:

* Windows local administrator
* Domain administrator
* Standard domain user
* Linux administrator
* Wazuh dashboard
* Splunk administrator
* NAS account
* GitHub
* Personal email
* School systems

Each system should use a unique credential.

---

## Administrative Account Separation

Use separate identities for:

* Normal user activity
* Domain administration
* Linux administration
* Application administration
* Test activity
* Service accounts

Examples:

```text
student.user
student.admin
svc.monitoring
test.user01
```

Administrative accounts should not be used for routine browsing or ordinary testing.

---

## Least Privilege

Grant only the access required for the task.

Examples:

* Standard users should not be Domain Admins.
* Wazuh agents should not use administrative credentials unnecessarily.
* Splunk forwarders should not run with broader privileges than required.
* Docker group membership should be limited.
* SSH should be restricted to trusted systems.
* File shares should use narrow permissions.
* Test accounts should be disposable and nonprivileged.

---

## Root Access

Linux commands requiring elevation should use:

```bash
sudo <COMMAND>
```

Avoid routine use of:

```bash
sudo su
```

or persistent root sessions.

Root access should be limited to:

* Package installation
* Network configuration
* Service management
* Protected file access
* Firewall changes
* Docker administration
* Application configuration

---

## Docker Group Risk

Membership in the Docker group can effectively provide root-level control.

A Docker-enabled user may be able to:

* Mount host files
* Start privileged containers
* Access protected directories
* Modify application data
* Control service availability

Only trusted administrative users should belong to the Docker group.

---

## Splunk Root Execution

The current lab may use:

```bash
sudo /opt/splunk/bin/splunk start --run-as-root
```

This should be documented as a lab-specific operational state rather than a recommended production practice.

Long-term hardening should consider:

* Dedicated Splunk service account
* Correct file ownership
* Narrow service permissions
* Restricted boot configuration
* Review of protected inputs

---

## Network Isolation

The primary CyberLab network should use VMware host-only networking.

Host-only networking reduces exposure to:

* Household devices
* Public networks
* ISP infrastructure
* Untrusted wireless systems
* External scanning
* Accidental testing beyond scope

NAT may be enabled temporarily for:

* Updates
* Package installation
* Repository access
* Time synchronization

---

## Bridged Networking

Bridged networking should not be the default for:

* Kali
* Vulnerable targets
* Domain controller
* Windows test endpoint
* Wazuh
* Splunk

Bridging can expose lab systems directly to the physical network.

Use bridged networking only when:

* The exercise specifically requires it.
* Scope is documented.
* The target is authorized.
* Firewall controls are in place.
* Recovery procedures exist.

---

## No Public Exposure

Do not configure:

* Router port forwarding
* Public tunnels
* Internet-facing Splunk Web
* Internet-facing Wazuh dashboard
* Public SSH access
* Public RDP access
* Public agent enrollment
* Public Docker API access

CyberLab management services should remain internal.

---

## Firewall Controls

Firewalls should remain enabled on:

* Acer Windows host
* DC01
* WIN11TARGET
* WAZUH-SERVER
* SPLUNK-SERVER
* KALI-TEST where appropriate

Use narrow rules based on:

* Source subnet
* Destination host
* Required port
* Protocol
* Service role

Do not disable a firewall permanently to make a service work.

---

## Host Security

The Acer host controls the entire virtual lab.

Host security should include:

* Windows updates
* Microsoft Defender or approved endpoint protection
* Windows Firewall
* Strong account password
* Screen lock
* BitLocker where appropriate
* Limited administrative use
* Secure browser profile
* Protected VMware storage
* Backup strategy
* No unnecessary public shares

A compromised host can affect every VM.

---

## VM Storage Security

VM files may contain:

* Operating systems
* Credentials
* Domain databases
* Logs
* Browser sessions
* Agent keys
* Index data
* Packet captures
* Snapshot memory

Protect VM storage through:

* Host access controls
* Disk encryption
* Secure backups
* Limited sharing
* Safe disposal
* Restricted synchronization

Do not upload VM files to public storage.

---

## Snapshot Security

Snapshots may preserve:

* Credentials in memory
* Active browser sessions
* Command history
* Authentication tokens
* Sensitive files
* Deleted data
* Test account state

Avoid publishing snapshot names or screenshots that expose operational details.

Snapshots should be protected like the active VM.

---

## Backup Security

Backups may contain more sensitive information than the public project.

Protect backups with:

* Encryption
* Access controls
* Separate storage
* Offline copies
* Integrity checks
* Recovery-key management
* Documented retention

Do not store the only decryption key inside the encrypted backup.

---

## Log Security

Security logs may expose:

* Usernames
* Domain names
* Internal addresses
* Process commands
* File paths
* Authentication failures
* Security identifiers
* Software inventory
* Service names
* Account changes

Raw logs should remain private unless specifically sanitized.

---

## Packet Capture Security

Packet captures may contain:

* Internal IP addresses
* MAC addresses
* Hostnames
* DNS queries
* Usernames
* Cleartext protocols
* Session information
* Authentication exchanges
* Application metadata

A screenshot of a few sanitized packets is generally safer than publishing a raw `.pcap`.

Do not upload operational packet captures to GitHub.

---

## Event Export Security

Files such as `.evtx`, CSV exports, JSON logs, and raw SIEM results may contain sensitive details.

Before publication:

1. Open the export.
2. Review all fields.
3. Remove unrelated events.
4. Replace operational values.
5. Confirm no hidden metadata remains.
6. Create a sanitized copy.
7. Preserve the original privately.

---

## Git Repository Security

Before every public push, review:

* Modified files
* New files
* Deleted files
* Commit message
* Image directory
* Configuration templates
* Log exports
* Backup archives
* Hidden files
* Environment files

Commands:

```bash
git status
```

```bash
git diff
```

```bash
git diff --staged
```

---

## Git Ignore Requirements

A `.gitignore` file should exclude sensitive or generated content.

Example:

```gitignore
# Credentials and environment files
.env
.env.*
*.secret
*.token

# Keys and certificates
*.key
*.pem
*.p12
*.pfx
*.jks
*.keystore

# Logs and evidence
*.log
*.evtx
*.pcap
*.pcapng
evidence/
logs/
captures/

# Backups and archives
backups/
*.bak
*.backup
*.tar
*.tar.gz
*.zip

# Wazuh and Splunk data
certificates/
secrets/
volumes/
data/
indexes/

# VMware
*.vmem
*.vmsn
*.vmsd
*.lck
*.nvram
*.vmdk
*.vmx
*.vmxf

# System and editor files
.DS_Store
Thumbs.db
*.swp
*.tmp
```

A `.gitignore` rule does not remove files that were already committed.

---

## Already-Committed Secrets

If sensitive information was committed:

1. Remove the file from the current repository state.
2. Rotate the exposed secret.
3. Assume the secret was copied.
4. Remove it from Git history when appropriate.
5. Review forks, caches, and releases.
6. Document the incident privately.
7. Add preventive controls.
8. Verify the repository again.

Deleting the latest copy does not erase earlier commits.

---

## Secret Scanning

Before publication, search for likely secrets.

Examples:

```bash
grep -RniE \
    'password|passwd|secret|token|api[_-]?key|private[_-]?key' \
    .
```

Search for private-key markers:

```bash
grep -Rni \
    "BEGIN.*PRIVATE KEY" \
    .
```

Search for internal address patterns privately before replacing them.

Automated scanning should supplement, not replace, manual review.

---

## Repository Review Checklist

Before pushing:

* No operational IP addresses
* No passwords
* No tokens
* No private keys
* No agent keys
* No license files
* No personal usernames
* No email addresses
* No packet captures
* No raw event logs
* No VM files
* No backups
* No personal file paths
* No unrelated screenshots
* No hidden sensitive metadata

---

## Commit Message Security

Commit messages are public in a public repository.

Do not include:

* Passwords
* Tokens
* Internal URLs
* Incident details
* Personal names
* Private IP addresses
* Security weaknesses tied to live systems

Use neutral messages such as:

```text
Add sanitized Wazuh deployment documentation
```

---

## Branch and Pull Request Review

A branch or pull request may also expose content before it reaches the main branch.

Review:

* Draft pull requests
* CI logs
* Automated previews
* Build artifacts
* Comments
* Attachments
* Screenshots
* Review discussions

Do not assume unpublished branches are private in a public repository.

---

## Screenshot Sanitization Workflow

Before taking a screenshot:

1. Close unrelated applications.
2. Close personal browser tabs.
3. Hide bookmarks.
4. Clear notifications.
5. Use a neutral test account.
6. Remove personal desktop files.
7. Position only the relevant window.
8. Use a narrow time range.
9. Avoid showing raw credentials.
10. Confirm no unrelated terminal output appears.

After taking the screenshot:

1. Crop unnecessary areas.
2. Permanently redact sensitive values.
3. Remove metadata when appropriate.
4. Reopen the final image.
5. Zoom in and inspect every corner.
6. Confirm the redaction cannot be reversed.
7. Store the original privately.
8. Publish only the sanitized copy.

---

## Screenshot Redaction

Do not redact by:

* Placing movable shapes in an editable document
* Using transparent highlights
* Blurring text that remains readable
* Cropping only the visible preview while retaining the original
* Covering text in a layered file without flattening

Use permanent raster redaction or replace the screenshot with a sanitized recreation.

---

## Common Screenshot Leaks

Review for:

* Browser tabs
* Bookmarks
* Address bar
* Notifications
* Taskbar
* Clock and date
* Personal username
* Home directory
* Command prompt
* IP address
* Domain name
* MAC address
* Email
* File path
* License status
* Agent identifier
* Session ID
* QR code
* Background application
* VM inventory list

---

## Terminal Screenshot Security

Before publishing terminal output:

* Use a neutral shell prompt.
* Clear previous sensitive commands.
* Avoid showing home-directory usernames.
* Replace operational addresses.
* Avoid showing tokens in environment variables.
* Crop unrelated output.
* Confirm command history is not visible.

A public example prompt may use:

```text
student@LAB-SERVER:~$
```

---

## PowerShell Screenshot Security

Review for:

* Windows username
* Domain name
* Computer name
* Current path
* Profile path
* Security identifier
* Credential prompt
* Internal address
* Command history
* Administrative group membership

Use placeholder output or a sanitized recreation when possible.

---

## Browser Screenshot Security

Review:

* URL
* Hostname
* Port
* Session cookie visibility
* User account
* Browser profile
* Bookmarks
* Open tabs
* Download bar
* Notifications
* License information

A cropped dashboard panel is often safer than a full-browser screenshot.

---

## Diagram Sanitization

Architecture diagrams should avoid:

* Operational IP addresses
* Public IP addresses
* Exact firewall rules
* Real Wi-Fi names
* Device serial numbers
* Personal usernames
* Home address or floor plan
* Backup locations
* Unnecessary management ports

Use logical roles rather than operational identifiers.

---

## Configuration Template Sanitization

Before publishing a configuration file:

1. Copy the operational file.
2. Rename it with `.example`.
3. Remove secrets.
4. Replace addresses.
5. Replace usernames.
6. Replace domain names.
7. Remove private keys.
8. Remove tokens.
9. Add comments indicating required values.
10. Validate that the template is not accidentally operational.

Example:

```text
outputs.conf.example
docker-compose.example.yml
agent-config.example.xml
.env.example
```

---

## Environment File Safety

An `.env.example` may contain variable names but should not contain real values.

Example:

```dotenv
WAZUH_SERVER=<WAZUH_SERVER>
SPLUNK_SERVER=<SPLUNK_SERVER>
ADMIN_USER=<ADMIN_USER>
ADMIN_PASSWORD=<REDACTED_PASSWORD>
```

The real `.env` file should be ignored by Git.

---

## YAML and JSON Safety

Review YAML and JSON files for:

* Inline passwords
* Tokens
* Hostnames
* IP addresses
* Paths
* Certificates
* Base64-encoded secrets
* Cloud identifiers
* Webhook URLs

Encoding is not encryption.

A Base64 value may still contain a secret.

---

## Webhook Security

Webhook URLs often function as credentials.

Do not publish:

* Discord webhook URLs
* Slack webhook URLs
* Teams webhook URLs
* Alerting service tokens
* Notification API keys

If exposed:

* Revoke or regenerate the webhook.
* Update the application.
* Remove it from repository history.
* Review for misuse.

---

## API Key Security

API keys should be:

* Stored in a password manager
* Provided through environment variables
* Excluded from Git
* Rotated after exposure
* Scoped narrowly
* Disabled when no longer needed

Do not place keys directly in scripts.

---

## SSH Key Security

Never publish:

* Private key
* Unencrypted key archive
* Key passphrase
* SSH agent output
* Full authorized key inventory tied to personal systems

Public keys may sometimes be shared, but only when there is a clear need.

---

## Certificate Security

Public certificates may be shareable in some contexts.

Private keys must remain private.

Review for:

* Private-key files
* Keystore passwords
* Certificate subject names
* Internal hostnames
* Expiration dates
* SAN entries
* Certificate chain
* Truststore content

A lab certificate can still reveal infrastructure details.

---

## License Security

Do not publish:

* License files
* License keys
* Entitlement identifiers
* Customer IDs
* Download tokens
* Activation information
* Vendor account details

Public documentation may state that a trial, free, or lab license was used without exposing the details.

---

## Vulnerability Disclosure

The project may identify vulnerabilities or misconfigurations inside the lab.

Public documentation should focus on:

* The learning objective
* The remediation process
* The validation method
* The lesson learned

Avoid publishing a live weakness together with:

* Operational address
* Accessible service
* Exact version
* Administrative path
* Missing control

Remediate or isolate the issue before publishing.

---

## Detection Rule Security

Detection queries are generally safe to publish when sanitized.

Review for:

* Internal index names
* Hostnames
* Usernames
* Network ranges
* Private lookup tables
* Proprietary data sources
* Credential fields

Use placeholders in SPL, Wazuh rules, and Sigma examples.

---

## Log Query Sanitization

Example Splunk query:

```spl
index=<INDEX_NAME> host=<WINDOWS_ENDPOINT>
| search EventCode=<EVENT_ID>
| table _time host user source
```

Avoid embedding operational hostnames or usernames.

---

## Wazuh Rule Sanitization

A public rule template should replace:

* Operational rule IDs when sensitive
* Real usernames
* Real file paths
* Internal addresses
* Private groups
* Service accounts

The rule should still demonstrate the detection logic.

---

## Exercise Security

Detection exercises should use:

* Dedicated test accounts
* Synthetic data
* Approved test paths
* Narrow target lists
* Defined start and stop times
* Stable snapshots
* Cleanup procedures

Exercises should not use personal accounts or uncontrolled targets.

---

## Kali Safety

Before active testing:

1. Confirm the host-only adapter.
2. Confirm no unintended bridged adapter.
3. Confirm the authorized target.
4. Confirm monitoring is active.
5. Confirm the test command.
6. Confirm the expected impact.
7. Record the start time.
8. Stop after the objective is met.

Kali should not be treated as authorization to scan external systems.

---

## Malware Safety

The standard CyberLab should not be used for unrestricted live-malware execution.

Safer validation methods include:

* Official harmless antivirus test files
* Synthetic logs
* Benign scripts
* Detection simulation frameworks
* Public sample datasets
* Manually generated event patterns

A malware-analysis environment requires separate isolation and documentation.

---

## Personal Data Protection

Do not use personal data as test content.

Avoid:

* Personal documents
* Tax records
* Health records
* Private photos
* Personal email
* Real contact lists
* Banking information
* Employer information
* School records
* Real passwords

Use synthetic data instead.

---

## Synthetic Test Data

Suitable test content includes:

* Sample text
* Dummy usernames
* Generated CSV files
* Fake log entries
* Placeholder domains
* Documentation IP ranges
* Public sample datasets
* Benign scripts

Synthetic data should be clearly labeled as test data.

---

## Metadata Removal

Files may contain metadata such as:

* Author name
* Organization
* Device model
* GPS data
* Creation date
* Software version
* Editing history
* Comments
* Hidden layers

Review metadata before publication, especially for:

* Images
* Office documents
* PDFs
* Screenshots
* Exported reports
* Packet captures

---

## Image Metadata

Before publishing images:

* Export a flattened copy.
* Remove unnecessary EXIF data.
* Confirm no GPS information exists.
* Confirm the filename is neutral.
* Avoid preserving personal device names.

---

## Document Metadata

Before publishing a PDF or Office document, review:

* Author
* Company
* Last saved by
* Comments
* Revision history
* Hidden text
* Tracked changes
* Embedded files
* File path

Markdown is often preferable for public technical documentation because it contains less hidden metadata.

---

## PDF Sanitization

A visual black box may not remove the underlying text in a PDF.

For sensitive PDFs:

* Create a new sanitized document.
* Remove sensitive source text.
* Export a flattened version.
* Test text selection.
* Search the PDF for removed values.
* Inspect metadata.

---

## File Naming

Public filenames should be descriptive but neutral.

Good examples:

```text
wazuh-dashboard-sanitized.png
splunk-authentication-search.png
vmware-host-only-topology.png
file-integrity-alert.png
```

Avoid:

```text
my-home-network-192-168-x-x.png
admin-password-reset.png
personal-name-domain-controller.png
```

---

## Public Repository Structure

Recommended structure:

```text
CyberLab/
├── README.md
├── docs/
├── exercises/
├── runbooks/
├── images/
├── templates/
├── scripts/
├── .gitignore
├── LICENSE
└── SECURITY.md
```

Sensitive operational files should not exist inside the public working tree.

---

## Security Policy File

A `SECURITY.md` file may explain:

* The project is a personal lab.
* All targets are owned or authorized.
* No credentials or sensitive infrastructure details are intentionally published.
* Security issues in the repository can be reported privately.
* The repository should not be interpreted as permission to test any public system.

---

## Example Security Statement

```text
This repository documents a privately owned cybersecurity training lab.

All testing described in this project is limited to systems owned by the operator or systems for which explicit authorization has been granted.

Operational credentials, internal addressing, private keys, and sensitive configuration values are intentionally excluded or replaced with documentation-safe examples.
```

---

## Pre-Publication Workflow

1. Copy private notes into a separate public draft.
2. Replace operational values.
3. Remove secrets.
4. Remove personal information.
5. Review commands.
6. Review code blocks.
7. Review screenshots.
8. Review filenames.
9. Review metadata.
10. Run secret searches.
11. Review `git diff`.
12. Ask whether every detail is necessary.
13. Commit the sanitized files.
14. Review the GitHub-rendered version.
15. Correct any remaining exposure immediately.

---

## Two-Pass Review

Use two separate reviews.

### Technical Review

Confirm:

* Commands are correct.
* Architecture is understandable.
* Procedures are complete.
* Placeholders are consistent.
* Screenshots support the explanation.

### Security Review

Confirm:

* No credentials
* No operational addresses
* No personal identifiers
* No sensitive paths
* No secrets
* No raw evidence
* No reversible redactions
* No unnecessary service exposure

Separating these reviews reduces oversight.

---

## Search for Operational Values

Maintain a private checklist of operational values to search for before publication.

Examples:

* Real domain
* Real VM subnet
* Real administrator username
* Hostname
* NAS address
* Email address
* Home directory name
* Wazuh agent name
* Splunk server address

Search the public project for each value before pushing.

---

## PowerShell Repository Search

Example:

```powershell
Get-ChildItem `
    -Path . `
    -Recurse `
    -File |
Select-String `
    -Pattern "<PRIVATE_VALUE>"
```

Use the actual private values only during the local review.

Do not commit the search list.

---

## Linux Repository Search

Example:

```bash
grep -Rni "<PRIVATE_VALUE>" .
```

Search images manually because text-search tools may not detect visible content inside screenshots.

---

## Post-Publication Review

After publishing:

1. Open the repository in a private browser session.
2. Review the README.
3. Open every new image.
4. Open raw file views.
5. Review commit history.
6. Review downloadable artifacts.
7. Search the repository for sensitive values.
8. Confirm `.gitignore` behavior.
9. Correct any issue immediately.
10. Rotate exposed credentials when necessary.

---

## Incident Response for Accidental Exposure

If sensitive information is published:

1. Identify the exposed value.
2. Remove the public file.
3. Rotate or revoke the credential.
4. Assume the value was copied.
5. Remove it from Git history where appropriate.
6. Review logs for misuse.
7. Check forks and releases.
8. Update affected systems.
9. Document the incident privately.
10. Add a preventive control.

Do not wait to rotate a secret until repository history cleanup is complete.

---

## Sanitization Record

For major public releases, maintain a private record.

```text
Release:

Files reviewed:

Images reviewed:

Values replaced:

Secret scan completed:

Metadata reviewed:

Git diff reviewed:

Reviewer:

Publication date:

Issues found:

Corrections:
```

---

## Security Validation Checklist

### Host

* Windows updated
* Defender enabled
* Firewall enabled
* Storage protected
* Strong account security
* VMware files not publicly shared
* Backups protected

### Network

* Host-only used by default
* NAT enabled only as required
* No unintended bridged adapters
* No port forwarding
* No public tunnels
* Management services internal

### Identity

* Unique passwords
* Standard and admin accounts separated
* Test accounts disposable
* Service accounts limited
* Root access minimized
* Docker group restricted

### Applications

* Wazuh credentials protected
* Splunk credentials protected
* Default credentials changed
* Services patched
* Web interfaces internal
* Logs reviewed

### Repository

* `.gitignore` present
* No `.env`
* No keys
* No logs
* No packet captures
* No backups
* No VM files
* No personal paths
* No operational IP addresses
* No credentials

### Screenshots

* Cropped
* Permanently redacted
* Metadata reviewed
* Personal tabs hidden
* Notifications cleared
* Usernames removed
* Addresses removed
* Final copy reopened and checked

---

## Public Sanitization Checklist

Before publishing a document:

* Replace real IP addresses
* Replace real domain names
* Replace personal usernames
* Remove email addresses
* Remove passwords and tokens
* Remove MAC addresses
* Remove security identifiers
* Remove operational paths
* Replace test-account names where needed
* Remove private keys
* Remove backup locations
* Review command output
* Review tables
* Review code blocks
* Review links
* Review image filenames
* Review screenshots
* Review metadata
* Search the repository
* Review the rendered page

---

## Skills Demonstrated

This security and sanitization process demonstrates:

* Defense-in-depth planning
* Network isolation
* Identity and access management
* Credential protection
* Least privilege
* Secure administration
* Git repository security
* Secret management
* Security documentation
* Evidence handling
* Screenshot sanitization
* Metadata awareness
* Risk assessment
* Incident response
* Backup protection
* Ethical testing boundaries
* Public portfolio preparation
* Operational security

---

## Lessons Learned

Key lessons include:

* Public documentation should never be a direct copy of private operational notes.
* Internal IP addresses should be replaced with documentation-safe values.
* Screenshots often expose more information than the intended subject.
* A `.gitignore` file does not protect already committed files.
* Credentials must be rotated after exposure.
* Encoding does not make a secret safe.
* Webhook URLs should be treated as credentials.
* Docker access should be treated as administrative access.
* Host-only networking reduces accidental testing scope.
* Packet captures and event logs require stronger protection than ordinary screenshots.
* Snapshots and backups may preserve deleted credentials.
* Technical review and security review should be separate steps.
* Public diagrams should emphasize architecture rather than operational details.
* Metadata can expose information that is not visible in the document.
* Sanitization should be validated after publication, not only before it.

---

## Planned Improvements

Future improvements may include:

* Automated secret scanning
* Pre-commit security hooks
* GitHub secret scanning
* Repository security workflow
* Standardized screenshot template
* Automated metadata removal
* Dedicated sanitized documentation workspace
* Private operational repository
* Encrypted backup inventory
* Formal data-classification labels
* Security review checklist automation
* Periodic public-repository audits
* Credential rotation schedule
* Dedicated service accounts
* Splunk non-root migration
* Firewall rule review
* Automated bridged-adapter detection
* Expanded `SECURITY.md`
* Reusable `.example` configuration templates
* Public release approval checklist

---

## Summary

The Acer Blue Team CyberLab security strategy protects both the operational environment and the public portfolio.

A safe publication process requires:

* Host and VM security
* Network isolation
* Least privilege
* Unique credentials
* Protected logs and evidence
* Secure backups
* Separate public and private documentation
* Consistent placeholders
* Permanent screenshot redaction
* Repository secret scanning
* Metadata review
* Post-publication verification
* Immediate response to accidental exposure

The goal is to demonstrate real security engineering work without publishing the information an unauthorized person would need to access, map, or weaken the environment.

