# Oxford Academy Turkey Bowl Sabotage Investigation: Forensic Analysis Using Kusto Query Language
![image](https://github.com/user-attachments/assets/3ef0bf8b-1ee6-427f-ab83-7bf4cf2298e8)

# Case Overview

**Incident:** Tampering of Oxford Academy's playbook, which led to a loss in the annual Turkey Bowl competition against Sierra Vista High School.  
**Background:** Oxford Academy, historically dominant in both athletics and reputation, suffered an unexpected defeat. Upon investigation, the coaching staff found that their playbook had been tampered with, which may have contributed to their loss.  
**Objective:** To determine how the playbook was compromised, identify the individuals involved, and assess the potential scope of the sabotage.

## Scope of Investigation

- **Timeline:** Post-competition, the issue was identified when coaches noted discrepancies in their playbook.
- **Data Sources:** Logs from internal systems, file access and modification records, network traffic, and potential insider threats.
- **Tools Used:** Kusto Query Language (KQL) for querying logs and analyzing data from the organization's systems.

---

## Section 1: Employee Activity and Network Investigations

### **Code Highlights:**

#### 1. Quick Employee Overview
```kusto
// Display a sample of employee data for rapid assessment
Employees
| take 10
```
**Purpose:** Provides a quick look at employee data for further investigation.

#### 2. Suspicious Network Activity: Tracking Employee 'Patrick Stump'
```kusto
// Investigate any network activity originating from the IP address linked to 'Patrick Stump'
Employees
| where name == 'Patrick Stump'

OutboundNetworkEvents
| where src_ip == '10.10.0.25'
| distinct url
```
**Analysis:** Correlates employee data with outbound network activity to identify URLs accessed by ‘Patrick Stump’ and assess potential malicious actions.

#### 3. Identifying Suspicious Domains
```kusto
// Perform a passive DNS lookup for domains related to 'owls' in their names
PassiveDns
| where domain contains 'owls'
| distinct domain
```
**Insight:** Identifies suspicious domains related to the 'Oxford Owls,' which could indicate phishing or malicious web infrastructure.

---

## Section 2: Tracking Malicious Email Activity

### **Code Highlights:**

#### 1. Investigating Suspicious Emails
```kusto
// Track emails sent to the recipient 'dill_delichick@go-oxford-owls.edu'
Email
| where recipient == 'dill_delichick@go-oxford-owls.edu'
| count
```
**Purpose:** Counts the number of emails sent to a specific recipient, helping identify potential phishing or malicious email interactions.

#### 2. Tracking Email Links for Malicious Content
```kusto
// Find emails containing suspicious links related to 'Help_with_Playbook-Heres_What_I_tried.pdf'
Email
| where links contains 'Help_with_Playbook-Heres_What_I_tried.pdf'
```
**Outcome:** Tracks the propagation of potentially harmful documents and traces user interactions with malicious files.

#### 3. Investigating Suspicious File Creation Events
```kusto
// Track file creation on the host 'WMNQ-LAPTOP' related to a suspicious document
FileCreationEvents
| where hostname == 'WMNQ-LAPTOP' and filename == 'Help_with_Playbook-Heres_What_I_tried.pdf'
```
**Purpose:** Identifies any file creation events tied to suspicious documents that may have been used for sabotage.

---

## Section 3: Investigating User Behavior and Process Execution

### **Code Highlights:**

#### 1. Tracking 'Tac Zaylor' Activity
```kusto
// Locate 'Tac Zaylor' in the employee database and investigate their network activity
Employees
| where name == 'Tac Zaylor'

AuthenticationEvents
| where username == 'tazaylor'
```
**Objective:** Examines login attempts and network activity associated with 'Tac Zaylor' to detect potential signs of insider threat or unauthorized access.

#### 2. Analyzing Process Events: Malicious File Execution
```kusto
// Monitor process execution related to suspicious file 'Advanced_IP_Scanner_2.5.3850.exe'
ProcessEvents
| where username == 'tazaylor' and timestamp >= datetime('2024-11-15T13:04:36Z')
| where filename contains 'Advanced_IP_Scanner_2.5.3850.exe'
```
**Insight:** Tracks the execution of a suspicious scanning tool, which could be associated with information gathering or system compromise.

#### 3. Investigating Document Activity
```kusto
// Track any file creation activities involving documents containing 'doc' in their filenames
FileCreationEvents
| where username == 'tazaylor' and filename contains 'doc'
```
**Outcome:** Examines documents created or modified by 'Tac Zaylor' that may indicate the creation of malicious content.

---

## Section 4: Network and DNS Investigations

### **Code Highlights:**

#### 1. Correlating Network Activity with DNS Queries
```kusto
// Correlate network traffic with suspicious DNS queries
let suspicious_domains = PassiveDns
| where ip == '66.159.94.55'
| distinct domain;

OutboundNetworkEvents
| where url has_any (suspicious_domains)
| distinct src_ip
```
**Application:** Connects DNS queries to network traffic, identifying IP addresses involved in accessing malicious domains.

#### 2. Identifying Potential Exfiltration
```kusto
// Investigate network traffic involving outbound requests to suspicious URLs
OutboundNetworkEvents
| where url has_any ('oxfordacademy-bestatfootball.com','oxfordacademyfo0tball.com','0xfordacademyf00tball.org','oxfordacademyfutball.com')
| distinct src_ip
```
**Insight:** Tracks outbound network traffic to URLs potentially associated with data exfiltration or malicious domains.

---

## Section 5: Advanced Investigative Techniques

### **Code Highlights:**

#### 1. Modular Queries with 'let' Statements
```kusto
// Use 'let' statement to modularize queries and track multiple users' activity
let tac_ips = Employees
| where name contains 'Tac'
| distinct ip_addr;

OutboundNetworkEvents
| where src_ip in (tac_ips)
| distinct url
```
**Benefits:** Demonstrates how to modularize queries for reusability and flexibility, improving query efficiency and scalability.

#### 2. Cross-referencing Email with DNS Activity
```kusto
let threat_domains = PassiveDns
| where ip == '176.167.219.168'
| distinct domain;

Email
| where link has_any (threat_domains)
| summarize count() by sender
```
**Application:** Cross-references email interactions with DNS activity, detecting coordinated malicious actions across different vectors.

---

## Conclusion

This investigation demonstrates advanced proficiency in Kusto Query Language (KQL) through:

- **Correlating multi-source data** for a comprehensive forensic analysis.
- **Leveraging modular queries** with `let` statements for flexibility and reusability.
- **Tracking complex attack patterns**, including phishing, malware distribution, and insider threats.

### **Skills Demonstrated:**
- **Forensic Analysis:** Tracked user activity, email links, and file creation events to pinpoint malicious actions.
- **Network Security:** Analyzed network traffic and DNS queries to uncover potential data exfiltration or compromise.
- **Data Correlation:** Linked disparate datasets (e.g., Emails, Passive DNS, Network Events) to uncover coordinated attack efforts.
