## Title
Riverview Admissions - Source Code Backup & Sensitive PII Disclosure

## Description
**Affected URL:** https://bird.pwnedbox.site/
A backup file `index.php.bak` is publicly accessible on the web root of the Riverview Admissions Management System. The file contains the full PHP source code of the application including hardcoded database credentials, internal hostname, database name, and application configuration. This file was noted internally as requiring deletion post-migration (Reference Ticket: RV-2025-1842) but was never removed, leaving production secrets fully exposed to any unauthenticated user.

## Steps to Reproduce
1. Navigate to `http://bird.pwnedbox.site/index.php.bak` directly in a browser
2. Observe that the server serves the raw PHP backup file as plain text instead of blocking or returning a 403/404
3. Scroll through the file to find hardcoded database credentials and internal configuration

## POC (Proof of Concept)
<img width="1915" height="1000" alt="a2" src="https://github.com/user-attachments/assets/cd977ed3-5ca2-491c-8429-08470bd9c920" />

## Impact
- Attacker obtains plaintext database password → direct MySQL access to `riverview_admissions` DB
- Internal hostname (`db01.internal.riverview.test`) reveals internal network topology
- Full source code review enables discovery of further vulnerabilities (SQLi, auth bypasses, logic flaws)

## Remediation
- Immediately delete all `.bak`, `.old`, `.backup`, `.orig` files from the web root and any publicly accessible directories
- Configure the web server to block or return 403 on common backup file extensions (`.bak`, `.old`, `.swp`, `.tmp`)
- Never hardcode credentials in source code — use environment variables or a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault)
- Add backup file patterns to `.htaccess` or Nginx config:
```
	<FilesMatch "\.(bak|old|backup|orig|swp)$">
    Require all denied
	</FilesMatch>
```
