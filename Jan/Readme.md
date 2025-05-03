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

Through a brute-force attack on the login form located at http://192.168.1.130:8080/admin, valid credentials were obtained, allowing authentication as user.
This demonstrates that the panel does not limit or delay failed login attempts.

4. Proof of Concept (PoC)
4.1. Port Scanning
nmap -sV -sC -oN jan_nmap.txt 192.168.1.130
Open ports: 22/tcp (SSH), 8080/tcp (HTTP)

4.2. Web Enumeration
gobuster dir -u http://192.168.1.130:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --exclude-length 45
Discovered directory: /admin

4.3. Credentials

The directory was sending a message "Only accessible internally." checking some directorys i found the message "Parameter 'url' needed."
Doing checks i discovered that if you send 2 requests to the domain on the same line the server did not check the 2nd request. 
After going into /credz i found "ssh/EazyLOL"

After conecting via ssh with the password EazyLOL, i found the user.txt with the flag

4.4. Account Access
Successful authentication using the retrieved credentials.

5. Impact
Low impact in a CTF or testing environment.

In a real-world scenario, an attacker could:

Enumerate valid users.

Attempt privilege escalation from that account (if vulnerable code is found).

Launch social engineering attacks against internal users.

6. Recommendations / Remediation
Harden the /redirect endpoint

Strict single‐param parsing: Only accept one url parameter. Reject requests containing multiple url parameters.

Allow-list valid targets: Only permit internal paths or explicitly approved domains.

Normalize & sanitize inputs: Remove or percent-encode any leading /, // or protocol schemes before processing.

Reject unknown schemes: Disallow redirects to relative paths unless they exactly match internal routes.

Detailed logging & monitoring: Record all redirect requests and trigger alerts on any attempt to redirect outside the allow-list.
