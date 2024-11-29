# French Socksess Story: A Forensic Investigation Using Kusto Query Language
![https://nm0g0vqj.tinifycdn.com/photos/sockspic.png]

### Author: Jim Rossiter

---

## Overview

In this investigation, Kusto Query Language (KQL) is utilized to analyze a cyberattack involving a combination of phishing, credential dumping, data exfiltration, and extortion. The threat actor used sophisticated tactics, such as masquerading as a legitimate customer service email and exploiting vulnerabilities within the organization. The investigation traces the actor's activity from the initial attack vector to the final stages of extortion.

### Objectives:
1. Investigate the email communication used in the attack.
2. Identify compromised machines and user actions.
3. Trace the malicious files, including their creation and usage.
4. Highlight key IP addresses and domains involved in the attack.
5. Showcase proficiency in KQL to perform complex correlation and analysis.

---

## Section 1: Email and Communication Tracking

### Code Highlights:

#### 1. **Tracking the Extortion Email**
```kql
Email
| where subject == 'Pay up or the world will know your lies'
```
- *Purpose*: Filters out emails with the subject line associated with the extortion attempt.

#### 2. **Identifying the Sender of the Extortion Email**
```kql
Email
| where subject == 'Pay up or the world will know your lies'
| distinct sender
```
- *Outcome*: Identifies the threat actor's email address: `theempireownsyou@proton.me`.

#### 3. **Recipient Information for Extortion Email**
```kql
Email
| where subject == 'Pay up or the world will know your lies'
| distinct recipient
| lookup Employees on $left.recipient == $right.email_addr
```
- *Analysis*: Cross-references the extortion email recipients with employee records, identifying **Cho Cetkipu** and **Brooke Entoe**.

---

## Section 2: Process and File Activity

### Code Highlights:

#### 1. **Reconnaissance on Internal Machines**
```kql
ProcessEvents
| where hostname in ('SCHR-MACHINE', '9RGD-MACHINE')
| where timestamp between (datetime(2024-08-30T00:00:00Z) .. datetime(2024-08-31T00:00:00Z))
```
- *Purpose*: Identifies potential reconnaissance activity by the threat actor on two internal machines.

#### 2. **Detecting Use of Bitsadmin for Exfiltration**
```kql
ProcessEvents
| where not(hostname in ('SCHR-MACHINE', '9RGD-MACHINE'))
| where process_commandline has 'bitsadmin'
| distinct username
| lookup Employees on $left.username == $right.username
```
- *Outcome*: Tracks the use of `bitsadmin` for exfiltration and links the activity to a specific user.

#### 3. **Malicious File Usage by Mike Oz**
```kql
ProcessEvents
| where username == 'mioz'
| where process_commandline has 'feetlover.rar'
```
- *Objective*: Tracks Mike Oz's use of a suspicious `.rar` archive, which may contain a malicious payload.

---

## Section 3: DNS and Network Activity

### Code Highlights:

#### 1. **Tracking Malicious DNS Requests**
```kql
PassiveDns
| where ip == '193.233.125.78'
| distinct domain
```
- *Analysis*: Identifies domains associated with the IP address `193.233.125.78`, linked to the threat actor's infrastructure.

#### 2. **Identifying Suspicious Domains**
```kql
PassiveDns
| where domain in ('thesockwhisperer.com', 'liarliarsocksonfire.net')
| distinct ip
```
- *Outcome*: Traces malicious domains, identifying key IP addresses involved in the attack.

#### 3. **Inbound Network Traffic from Malicious IPs**
```kql
let hacker_ips = PassiveDns
| where domain in ('thesockwhisperer.com', 'liarliarsocksonfire.net')
| distinct ip;

InboundNetworkEvents
| where src_ip in (hacker_ips)
```
- *Insight*: Correlates inbound network events with the identified malicious IPs to track communication between the threat actor and the compromised systems.

---

## Section 4: Email and Network Correlation

### Code Highlights:

#### 1. **Phishing Email Sent by Mike Oz**
```kql
Email
| where sender == 'mike_oz@jusdechaussette.fr'
| where recipient in ('cho_cetkipu@jusdechaussette.fr','brooke_entoe@jusdechaussette.fr')
```
- *Analysis*: Tracks phishing emails sent by Mike Oz to internal recipients, Cho Cetkipu and Brooke Entoe.

#### 2. **Phishing Email Links and Network Correlation**
```kql
Email
| where sender == 'mike_oz@jusdechaussette.fr'
| where recipient in ('cho_cetkipu@jusdechaussette.fr','brooke_entoe@jusdechaussette.fr')
| distinct link
| lookup OutboundNetworkEvents on $left.link == $right.url
```
- *Outcome*: Correlates the links in Mike Oz's phishing emails with outbound network events, identifying the URLs visited.

---

## Section 5: File Creation and Exfiltration

### Code Highlights:

#### 1. **Tracking File Creation by Mike Oz**
```kql
FileCreationEvents
| where username == 'mioz'
| where timestamp between (datetime(2024-08-18T16:10:29Z) .. datetime(2024-08-18T16:58:29Z))
```
- *Purpose*: Detects the creation of suspicious files by Mike Oz, potentially containing exfiltrated data.

#### 2. **Tracking Network Traffic to Exfiltrate Data**
```kql
InboundNetworkEvents
| where url contains 'holesinmysocks.jpeg'
```
- *Analysis*: Tracks the exfiltration of the malicious file "holesinmysocks.jpeg" to a threat actor-controlled server.

---

## Advanced Techniques

### **Correlation of Multiple Data Sources**

- **Combining Email Data with Network Logs**:
    - By correlating email content and network traffic, we can identify the exact sequence of malicious actions and pinpoint the source of the attack.
  
### **Use of `let` Statements for Modular Queries**:
```kql
let hacker_ips = PassiveDns
| where domain in ('thesockwhisperer.com', 'liarliarsocksonfire.net')
| distinct ip;
InboundNetworkEvents
| where src_ip in (hacker_ips)
```
- *Benefits*: Modular queries help to organize and reuse common data (e.g., malicious IPs) across different parts of the investigation.

---

## Conclusion

This investigation demonstrates the ability to use **Kusto Query Language (KQL)** for comprehensive forensic analysis. The queries presented:
1. **Track malicious emails** and **phishing attempts**.
2. **Identify compromised machines** and **exfiltrated files**.
3. Correlate network traffic and **DNS activity** to uncover the infrastructure used by the threat actor.
4. **Use advanced techniques** such as modular queries, data correlation, and event tracking to build a detailed timeline of the attack.

### Skills Demonstrated:
- **Forensic Analysis**: Using email, network events, and file logs to piece together a timeline of an attack.
- **Data Correlation**: Cross-referencing data from multiple sources (emails, DNS, network events).
- **Advanced KQL Queries**: Leveraging `let` statements, joins, and filtering techniques to optimize query performance and results.

---
