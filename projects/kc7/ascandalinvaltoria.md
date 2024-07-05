## Section 1:
3. How many employees work at the Valdorian Times?
```kql
Employees
| count
```

4. What is the Editorial Director's name?
```kql
Employees
| where role == 'Editorial Director'
```

5. How many emails did Nene Leaks receive?
```kql
Email
| where recipient == 'nene_leaks@valdoriantimes.news'
| count
```

6. How many distinct senders were seen in the email logs from the domain name "weprinturstuff.com"?
```kql
Email
| where sender has 'weprinturstuff.com'
| distinct sender
| count
```

7. How many distinct websites did “Lois Lane” visit?
```kql
Employees
| where name == 'Lois Lane'

OutboundNetworkEvents
| where src_ip == '10.10.0.22'
| distinct url
| count
```

8. How many distinct domains in the PassiveDns records contain the word “hire”?
```kql
PassiveDns
| where domain contains 'hire'
| distinct domain
| count
```

9. What IPs did the domain “jobhire.org” resolve to (enter any one of them)?
```kql
PassiveDns
| where domain == 'jobhire.org'
| distinct ip
```

10. How many distinct websites did employees with the first name "Mary" Visit?
```kql
let mary_ips = Employees
| where name has 'Mary'
| distinct ip_addr;
OutboundNetworkEvents
| where src_ip in (mary_ips)
| distinct url
| count
```

11. How many authentication attempts did we see to the accounts of employees with the first name Mary?
```kql
let marys = Employees
| where name has 'Mary'
| distinct username;
AuthenticationEvents
| where username in (marys)
| count
```
## Section 2
2. What is the Editorial Intern's name?
3. When was the Editorial Intern hired at The Valdorian Times?
```kql
Employees
| where role has 'Intern'
```

4. How many total emails has Clark Kent received?
```kql
Email
| where recipient has 'clark_kent'
| count
```

5. Review the emails sent to Clark Kent for the one sent on January 31, 2024 containing the final edits for the election OpEd. What was the subject line of this email?
6. Who sent this email containing the final edits for the OpEd piece? Enter the sender's email address.
7. What was the name of the .docx file that was sent in this email?
```kql
Email
| where recipient has 'clark_kent'
```

## Section 3
1. What is Sonia's job role?
```kql
Employees
| where name == 'Sonia Gose'
```

2. Sonia shows you a suspicious email she received a few weeks ago. What email address was used to send this email?
3. Let's look for this email in our email logs. When was the email sent to Sonia Gose? Enter the exact timestamp from the logs.
4. What URL was included in the email?
```kql
Email
| where sender == 'newspaper_jobs@gmail.com'
```

5. You ask Sonia if she clicked on the link but she says she doesn't remember. Let's help her remember. 😐 What is Sonia Gose's IP address?
```kql
Employees
| where name == 'Sonia Gose'
| project ip_addr
```

6. Did Sonia click on this link? If so, enter the timestamp when she clicked the link. If not, type "no".
7. What was the name of the docx file in the link that Sonia clicked?
```kql
OutboundNetworkEvents
| where src_ip == '10.10.0.3' and url == 'https://promotionrecruit.com/published/Valdorian_Times_Editorial_Offer_Letter.docx'
```

8. What is Sonia Gose's hostname?
```kql
Employees
| where name == 'Sonia Gose'
| project hostname
```

9. When did the downloaded docx file first show up on Sonia's machine?
10. What was the full path of the docx file that was downloaded to Sonia's machine?
```kql
FileCreationEvents
| where hostname == 'UL0M-MACHINE' and filename == 'Valdorian_Times_Editorial_Offer_Letter.docx'
```

11. What is the sha256 hash of the file that Sonia downloaded?
```kql
ProcessEvents
| where process_commandline has 'ps1'and hostname == 'UL0M-MACHINE'
```

12. What is the name of the file (.ps1) that was written to disk immediately after the docx was downloaded?
13. When was this new file created?
```kql`
FileCreationEvents
| where hostname == 'UL0M-MACHINE'
``
18. How many Process Events are there related to this PowerShell script on Sonia's machine?
```kql
ProcessEvents
| where process_commandline has 'ps1'and hostname == 'UL0M-MACHINE'
``` 


