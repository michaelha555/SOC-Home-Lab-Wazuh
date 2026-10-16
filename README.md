# SOC Home Lab — Wazuh SIEM & File Integrity Monitoring

## Overview
A hands-on SOC home lab built with VirtualBox to practice endpoint monitoring, Windows event analysis, and file integrity monitoring (FIM) using Wazuh. This is a controlled learning environment, not a production deployment.

## Lab Architecture

```text
Windows host (VirtualBox)
   |
   +-- Ubuntu Server 24.04 VM — Wazuh Manager, Indexer, Dashboard
   |       Host-only IP: 192.168.56.101
   |
   +-- Windows 10 VM — Wazuh Agent
           Host-only IP: 192.168.56.102
```

Both VMs used a VirtualBox host-only network to communicate. NAT networking was also configured for internet access. The Wazuh dashboard was accessed from the host through a local port-forwarded connection.

## Tools and Technologies
- Oracle VirtualBox
- Ubuntu Server 24.04 LTS
- Windows 10 Enterprise (test VM)
- Wazuh 4.14.9 (Manager, Indexer, Dashboard, Windows Agent)
- Windows Event Viewer
- PowerShell

## Implementation
1. Deployed the Wazuh stack on an Ubuntu Server VM.
2. Configured a VirtualBox host-only network between Ubuntu and Windows VMs.
3. Installed and started the Wazuh Agent on Windows using PowerShell.
4. Verified that the Windows endpoint appeared as **Active** in Wazuh.
5. Used Wazuh Threat Hunting to review endpoint alerts and event details.

## Scenario 1 — Failed Windows Logons
**Objective:** Verify collection and detection of failed Windows authentication events.

**Test:** Intentionally entered an incorrect Windows password three times in the lab VM, then inspected the Security log in Event Viewer.

**Evidence and findings:**
- Windows Security Event ID **4625** recorded failed logon attempts.
- Wazuh generated alerts including rule **60122** (*Logon Failure — Unknown user or bad password*, level **5**).
- The event details included the Windows endpoint, a local source address (`127.0.0.1`), logon type **2**, and status/substatus fields indicating a failed logon due to an incorrect password.

**Assessment:** These were intentional local failed logons. They demonstrate successful event collection and alerting, but do **not** establish a real brute-force attack.

## Scenario 2 — File Integrity Monitoring (FIM)
**Objective:** Detect file creation, modification, and deletion on the Windows endpoint.

**Configuration:** Enabled real-time monitoring for a dedicated test directory in the Windows Agent `ossec.conf`:

```xml
<directories realtime="yes">C:\SOC-Lab</directories>
```

Restarted the Wazuh Agent service after applying the configuration.

**Test:** Created a text file in `C:\SOC-Lab`, wrote test content, saved it, and then deleted it.

**Observed alerts:**

| Action | Wazuh alert | Rule ID | Level |
|---|---|---:|---:|
| File created | File added to the system | 554 | 5 |
| File modified | Integrity checksum changed | 550 | 7 |
| File deleted | File deleted | 553 | 7 |

The modification alert confirmed real-time monitoring and showed a size change from **0 to 15 bytes**, as well as changes in modification time and file hashes. The event path displayed `C:\soc-lab\test.txt.txt` because the text-file extension was duplicated during file creation.

**Assessment:** Wazuh successfully alerted on file lifecycle events in the test directory. A file change is not, by itself, proof of malicious activity.

## Skills Practiced
- Virtual machine and host-only network configuration
- Linux server setup and Windows endpoint agent deployment
- SIEM event collection and alert review
- Windows Security log analysis (Event ID 4625)
- Wazuh rule IDs, alert levels, and event fields
- File integrity monitoring and basic incident triage

## Screenshots
Add your own screenshots in an `images/` folder, such as:
- `01-wazuh-agent-active.png`
- `02-windows-event-4625.png`
- `03-wazuh-failed-logon-alerts.png`
- `04-fim-file-added-and-modified.png`
- `05-fim-modification-details.png`
- `06-fim-file-deleted.png`

> Before publishing, review screenshots for passwords, tokens, personal information, or other sensitive data. Never upload Wazuh installation password archives or configuration files containing credentials.

## Conclusion
This lab demonstrated end-to-end monitoring from a Windows VM through a Wazuh Agent to a Wazuh server, with successful alerting for failed logons and file integrity changes. Future improvements could include a more detailed investigation workflow and additional telemetry sources.
