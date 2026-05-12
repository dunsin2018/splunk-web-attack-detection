# Indicators of Compromise (IOCs)

**Project:** Web Application Attack Detection & Threat Hunting  
**Source:** Splunk analysis of `access_log.txt`  
**Events Analyzed:** 248,664

---

## Suspicious IP Addresses

| IP Address        | Classification | Activity                          | Notes                          |
|-------------------|----------------|-----------------------------------|--------------------------------|
| `185.199.110.153` | External       | Command Injection / RFI / LFI    | High-volume, multi-vector      |
| `37.49.226.150`   | External       | SQL Injection / Recon             | sqlmap user-agent confirmed    |
| `192.168.1.101`   | Internal/LAN   | SQLi / XSS / LFI                 | RFC 1918 — internal segment    |
| `10.0.0.5`        | Internal       | Command Injection                 | ⚠️ Potential insider / lateral movement |

---

## Suspicious User Agents

| User Agent               | Tool       | Purpose                                   |
|--------------------------|------------|-------------------------------------------|
| `sqlmap/1.4.5-dev`       | sqlmap     | Automated SQL injection & DB enumeration  |
| `Nmap Scripting Engine`  | Nmap (NSE) | Port scanning, service fingerprinting     |

---

## Attack Payloads

### Directory Traversal / LFI
```
../../../../etc/passwd
../../../etc/shadow
```

### SQL Injection
```
' OR '1'='1
' OR 1=1--
```

### Cross-Site Scripting (XSS)
```
<script>alert('XSS')</script>
```

### Command Injection / RCE
```
cmd=whoami;id;ls -la
cmd=cat /etc/passwd
```
> ⚠️ **Critical:** HTTP 200 response observed for `cmd=whoami;id;ls -la` — potential successful execution.

### Remote File Inclusion (RFI)
```
http://malicious.com/shell.txt
```

---

## Affected Endpoints

| Endpoint          | Attack Type              |
|-------------------|--------------------------|
| `/search`         | SQL Injection            |
| `/profile.php`    | XSS                      |
| `/download`       | Directory Traversal / LFI|
| `/exec.php`       | Command Injection / RCE  |
| `/page.php`       | Remote File Inclusion    |

---

*Last updated: May 2026*
