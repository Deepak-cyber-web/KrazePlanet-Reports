## Title
SQL Injection – GET Parameter (`id`) on `courses.pwnedbox.site`

## Description
**Affected URL:** https://courses.pwnedbox.site/
A SQL Injection vulnerability exists in the `id` GET parameter of the application hosted at `https://courses.pwnedbox.site/`. The parameter is passed directly into a SQL query without sanitization or parameterization, allowing an attacker to manipulate the backend MariaDB database.

Two injection techniques were confirmed:

- **Error-Based:** The application returns verbose SQL error messages including internal server paths and database details, directly exposing backend architecture.
- **Time-Based Blind:** Injected `SLEEP()` payloads caused measurable, controlled response delays, confirming that arbitrary SQL expressions are being executed by the server.

**Affected URL:** `https://courses.pwnedbox.site/?id=`  
**Affected Parameter:** `id` (GET)  
**Backend Database:** MariaDB  
**Server Stack:** LAMPP (Linux + Apache + MySQL/MariaDB + PHP)  
**Vulnerable File:** `/opt/lampp/htdocs/subdomains/courses/index.php`

## Steps to Reproduce
1. Navigate to `https://courses.pwnedbox.site/?id=1`.
2. Append a single quote to trigger a syntax error:
   ```
   https://courses.pwnedbox.site/?id=1'
   ```
   → Application returns a fatal SQL error leaking the DB type, server path, and PHP file location.

3. Inject a time-based blind payload to confirm code execution:
   ```
   GET /?id=1'XOR(94102*if(now()=sysdate(),sleep(5),0))XOR'Z
   ```
   → Response is delayed by ~5 seconds, confirming the `SLEEP(5)` executed.

4. Confirm with a longer sleep using Burp Suite Intruder:
   ```
   GET /?id=1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
   ```
   → Response delayed by ~10 seconds (10,293 ms observed).

5. Test a multiplied sleep to confirm arithmetic execution:
   ```
   GET /?id='XOR(if(now()=sysdate(),sleep(5*5),0))OR'
   ```
   → Request timed out (25-second sleep confirmed).

## POC (Proof of Concept)
<img width="1920" height="1030" alt="3" src="https://github.com/user-attachments/assets/d320f134-8910-42b8-8ef2-8381740b8c60" />
<img width="1918" height="1023" alt="2" src="https://github.com/user-attachments/assets/bbdc9d4f-63fe-414d-96ac-90be78bf3f00" />
<img width="1918" height="1027" alt="1" src="https://github.com/user-attachments/assets/00f4365e-7162-4f48-aa61-aff7a3b1cdc9" />

## Impact
- **Data Exfiltration** – Attacker can dump all database contents including credentials, user PII, and application data.
- **Authentication Bypass** – Login queries can be manipulated to gain unauthorized admin/user access.
- **Remote Code Execution** – Via `INTO OUTFILE` (write web shell) or `LOAD_FILE()` (read server files) if DB user has `FILE` privilege.
- **Full Server Path Disclosed** – Verbose errors expose internal stack, aiding further attacks.

## Remediation

1. **Use Prepared Statements** – Never concatenate user input into queries directly.
   ```php
   $stmt = $pdo->prepare("SELECT * FROM courses WHERE id = ?");
   $stmt->execute([$_GET['id']]);
   ```

2. **Validate Input** – Cast `id` to integer; reject anything else.
   ```php
   $id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);
   if (!$id) { http_response_code(400); exit; }
   ```

3. **Suppress Error Messages** – Set `display_errors = Off` in `php.ini` to prevent stack trace leakage.

4. **Least Privilege DB User** – Application DB account should only have `SELECT/INSERT/UPDATE/DELETE` — no `FILE` or admin privileges.
