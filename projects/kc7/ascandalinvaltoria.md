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
