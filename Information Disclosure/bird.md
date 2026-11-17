# Title
Exposed Source Code Backup File Leaking Hardcoded Database Credentials

# Vulnerability Type
Information Disclosure — Source Code Backup Exposure

# Summary
A backup file `index.php.bak` is publicly accessible on the web root of the Riverview Admissions Management System. The file contains the full PHP source code of the application including hardcoded database credentials, internal hostname, database name, and application configuration. This file was noted internally as requiring deletion post-migration (Reference Ticket: RV-2025-1842) but was never removed, leaving production secrets fully exposed to any unauthenticated user.

# Vulnerable Endpoint
http://bird.pwnedbox.site/index.php.bak

# Steps to Reproduce
1. Navigate to `http://bird.pwnedbox.site/index.php.bak` directly in a browser
2. Observe that the server serves the raw PHP backup file as plain text instead of blocking or returning a 403/404
3. Scroll through the file to find hardcoded database credentials and internal configuration

# Proof of Concept
<img width="1915" height="1000" alt="a2" src="https://github.com/user-attachments/assets/407e316d-6de9-4103-b1f0-bf6cc82070f9" />


# Impact
- Attacker obtains plaintext database password → direct MySQL access to `riverview_admissions` DB
- Internal hostname (`db01.internal.riverview.test`) reveals internal network topology
- Full source code review enables discovery of further vulnerabilities (SQLi, auth bypasses, logic flaws)

# Recommendation for fix
- Immediately delete all `.bak`, `.old`, `.backup`, `.orig` files from the web root and any publicly accessible directories
- Configure the web server to block or return 403 on common backup file extensions (`.bak`, `.old`, `.swp`, `.tmp`)
- Never hardcode credentials in source code — use environment variables or a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault)
- Add backup file patterns to `.htaccess` or Nginx config:
```
	<FilesMatch "\.(bak|old|backup|orig|swp)$">
    Require all denied
	</FilesMatch>
```
