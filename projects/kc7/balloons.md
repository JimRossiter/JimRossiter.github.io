# Balloons Over Iowa: A Forensic Investigation Using Kusto Query Language

### Author: Jim Rossiter

---

## Overview

This investigation leverages the power of **Kusto Query Language (KQL)** to uncover suspicious activity within a corporate environment. Through detailed queries, the investigation analyzes employee data, email interactions, network events, DNS records, and process execution logs. 

### Objectives:
1. Identify suspicious employee behavior and connections.
2. Trace the flow of potential phishing campaigns and malicious files.
3. Pinpoint compromised systems and associated activities.
4. Showcase advanced KQL skills for forensic analysis.

---

## Section 1: Employee Activity and Network Investigations

### Code Highlights:

#### 1. **Quick Employee Overview**
```kql
Employees
| take 10
```
- *Purpose*: Display a sample of employee data for rapid assessment.

#### 2. **Suspicious Network Activity**
```kql
Employees
| where name contains 'Jorge'
// 192.168.0.142

OutboundNetworkEvents
| where src_ip == '192.168.0.142'
| distinct url
| count
```
- *Analysis*: Correlates employee data with outbound network activity to identify URLs accessed by "Jorge."

#### 3. **Compromised Domain Indicators**
```kql
PassiveDns
| where domain contains 'infiltrate'
| distinct domain
| count
```
- *Insight*: Identifies the prevalence of domains containing "infiltrate," which might signal malicious behavior.

---

## Section 2: Phishing Campaign Analysis

### Code Highlights:

#### 1. **Tracking Phishing Links in Emails**
```kql
Email
| where link contains 'invasion.xyz'
| distinct recipient
```
- *Objective*: Detect recipients of phishing emails containing malicious links.

#### 2. **Tracing Malicious File Activity**
```kql
OutboundNetworkEvents
| where url == 'invasion.xyz/online/public/share/public/search/search/Flight-Crew-Information.xls'
// clicked by user ip 192.168.0.123
```
- *Outcome*: Identifies the IP address of a user who accessed the malicious file.

#### 3. **File Creation Events**
```kql
FileCreationEvents
| where hostname == 'HNOA-LAPTOP' and filename contains 'Flight'
```
- *Purpose*: Tracks the download of suspicious files to a specific machine.

---

## Section 3: Domain and IP Investigations

### Code Highlights:

#### 1. **Analyzing Malicious DNS Records**
```kql
let domains_sus = PassiveDns
| where ip == '131.102.77.156'
| distinct domain;

OutboundNetworkEvents
| where url has_any (domains_sus)
```
- *Application*: Links DNS queries to outbound network activity for compromised domains.

#### 2. **Redirect URL Investigation**
```kql
let sus_redirects = OutboundNetworkEvents
| where url contains "redirect"
| distinct url;

OutboundNetworkEvents
| where url in (sus_redirects)
| distinct src_ip
```
- *Insight*: Pinpoints source IPs affected by redirects, often indicative of phishing or malware campaigns.

---

## Section 4: Process and Ransomware Analysis

### Code Highlights:

#### 1. **Credential Dump Detection**
```kql
ProcessEvents
| where process_commandline contains 'mimikatz.exe'
| summarize count() by hostname
```
- *Objective*: Detect the execution of the well-known credential-dumping tool `mimikatz`.

#### 2. **Detecting Backups Deletion**
```kql
ProcessEvents
| where process_commandline contains 'Shadowcopy Delete'
```
- *Insight*: Indicates ransomware activity by identifying attempts to delete backup files.

#### 3. **Network Exfiltration via Specific Ports**
```kql
ProcessEvents
| where process_commandline contains '443'
| distinct hostname
```
- *Outcome*: Lists all machines connecting to an external server over port 443, potentially indicating data exfiltration.

---

## Advanced Techniques

### Utilizing `let` Statements for Modular Queries:
```kql
let karen_ips = Employees
| where name contains 'Karen'
| distinct ip_addr;

OutboundNetworkEvents
| where src_ip in (karen_ips)
| distinct url
```
- *Benefits*: Enhances modularity and reusability of query components.

### Combining Multiple Sources:
```kql
let threat_domains = PassiveDns
| where ip =='176.167.219.168'
| distinct domain;

Email
| where link has_any (threat_domains)
| summarize count() by sender
```
- *Application*: Cross-references DNS activity with email records to detect coordinated malicious actions.

---

## Conclusion

This investigation highlights advanced proficiency in **Kusto Query Language** through:
1. Correlating multi-source data for comprehensive analysis.
2. Leveraging modular queries with `let` statements.
3. Detecting and tracing complex attack patterns including phishing, malware distribution, and ransomware.

### Skills Demonstrated:
- **Forensic Analysis**: Tracking user activity, email links, and malicious processes.
- **Network Security**: Identifying DNS queries, IP connections, and data exfiltration.
- **Data Correlation**: Using multiple datasets (e.g., Email, PassiveDns, OutboundNetworkEvents) to establish causation.

---
