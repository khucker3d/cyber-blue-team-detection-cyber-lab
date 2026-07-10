# Firewall Block Investigation Runbook

## Runbook Overview

This runbook provides a structured process for investigating network traffic blocked by Windows Defender Firewall or another endpoint firewall control.

It applies to alerts involving:

* Windows Filtering Platform blocked connections
* Windows Defender Firewall dropped packets
* Repeated inbound connection attempts
* Repeated outbound connection attempts
* Blocked administrative services
* Blocked application traffic
* Wazuh or Splunk firewall alerts
* Unexpected firewall rule changes
* Firewall service interruption
* Controlled CyberLab firewall validation activity

The runbook helps the analyst determine:

* Which system initiated the connection
* Which endpoint blocked it
* Whether the traffic was inbound or outbound
* Which protocol and port were involved
* Which process or application was associated with the traffic
* Which firewall profile, rule, or default policy caused the block
* Whether the traffic was authorized
* Whether the block indicates misconfiguration, reconnaissance, malware, lateral movement, or another security concern
* Whether containment, rule correction, or escalation is required

---

## Runbook ID

```
RB-08
```

---

## Related Exercise

```
EX-08 – Firewall Block Investigation
```

---

## Intended Audience

* Cybersecurity students
* Blue Team analysts
* Security analysts
* Windows administrators
* Network security analysts
* Detection engineers
* Incident responders
* Home CyberLab operators

---

## Scope

This runbook applies to:

* Windows Defender Firewall
* Windows Filtering Platform events
* Endpoint firewall text logs
* Inbound and outbound connection blocks
* Application-specific firewall blocks
* Port-specific firewall blocks
* Firewall profile and rule changes
* Wazuh and Splunk firewall telemetry
* CyberLab host-only network traffic

This runbook does not authorize:

* Disabling the firewall to solve connectivity problems
* Creating broad allow rules without review
* Testing external systems
* Scanning systems outside the authorized CyberLab
* Removing rules before evidence is preserved
* Publishing operational firewall rules or internal addresses
* Assuming every block represents malicious activity

---

## Primary Data Sources

Use the available sources in this order:

1. Original firewall alert
2. Windows Security events
3. Windows Defender Firewall text log
4. Windows Firewall operational logs
5. Current firewall profile and rule configuration
6. Application and process telemetry
7. Sysmon network and process events
8. Wazuh alerts and raw events
9. Splunk indexed telemetry
10. Packet capture where available
11. Network and change records
12. User or administrator confirmation

---

## Common Windows Event IDs

### Windows Filtering Platform

| Event ID | General Meaning                                         |
| -------: | ------------------------------------------------------- |
|     5152 | A packet was blocked                                    |
|     5153 | A restrictive filter blocked a packet                   |
|     5155 | An application or service was blocked from listening    |
|     5157 | A connection was blocked                                |
|     5159 | An application or service was blocked from listening on |

