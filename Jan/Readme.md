📑 Finding Report: User Access (Jan – HackMyVM)

Executive Summary
Severity: Low / Informational
Brief description: Access to the SSH user account on the Jan machine from HackMyVM was achieved by discovering exposed credentials in a publicly accessible path via the HTTP panel running on port 8080.

Scope and Environment
Platform: HackMyVM

Machine: Jan (Easy – Linux)

Assigned IP: 192.168.1.130

Test Date: 26/04/2025

Tools Used
nmap

gobuster

ssh

Finding Description
While enumerating the HTTP service on port 8080, several interesting paths were discovered, including /redirect and /robots.txt. Manual testing revealed that when two requests were sent in a single line, the server only processed the first one, effectively bypassing certain validations. By exploiting this behavior, the path /credz was accessed, exposing credentials in clear text. These credentials allowed SSH login to the machine, where the user.txt flag was obtained.

Proof of Concept (PoC)
4.1. Port Scanning

nmap -sV -sC -oN jan_nmap.txt 192.168.1.130
Open ports:

22/tcp (SSH)

8080/tcp (HTTP)

4.2. Web Enumeration

gobuster dir -u http://192.168.1.130:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt -t 100 -k -r --exclude-length 45
Discovered directories: /redirect, /robots.txt

4.3. Credential Discovery

The /redirect path displayed the message: "Only accessible internally."

Exploring other directories showed the message: "Parameter 'url' needed."

During manual testing, it was observed that when sending two HTTP requests on the same line, the second one was ignored by the server.

This behavior allowed access to /credz, where the following credential was found:

ssh/EazyLOL
An SSH session was successfully established using this password, and the user.txt flag was retrieved.

4.4. Account Access

Successful authentication via SSH using the exposed credentials.

Full access to the user environment.

Impact
Low impact in a CTF or controlled environment.
In a real-world scenario, an attacker could:

Gain unauthorized access to internal infrastructure.

Attempt privilege escalation or lateral movement.

Perform social engineering attacks using collected user-level information.

Recommendations / Remediation
Harden the /redirect endpoint:

Strict parameter parsing: Accept only one url parameter. Reject requests with multiple url parameters.

Allow-list valid targets: Only allow internal routes or explicitly approved domains.

Normalize and sanitize inputs: Remove or percent-encode characters like /, //, and URL schemes before processing.

Reject unknown schemes: Do not allow redirects to relative paths unless they match known internal routes exactly.

Detailed logging and monitoring: Log all redirect requests and trigger alerts for any attempt to redirect outside the allow-list.
