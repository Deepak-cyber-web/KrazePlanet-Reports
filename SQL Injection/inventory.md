## Title
Time-Based Blind SQL Injection – Multiple POST Parameters in CBSE Result Lookup Form at `inventory.pwnedbox.site`

## Description
**Affected URL:** https://inventory.pwnedbox.site/
A Time-Based Blind SQL Injection vulnerability exists in multiple POST parameters of the CBSE Class XII Examination Results lookup form at `https://inventory.pwnedbox.site` (DigiLocker). The application passes user-supplied form fields directly into a MariaDB query without sanitization. Injected `SLEEP()` payloads cause controlled, measurable response delays confirming arbitrary SQL execution. The application also returns verbose error messages disclosing the internal server path and query structure.

**Affected Parameters:** `roll_no`, `admit_card_id`, `school_no`, `mother_name` (POST body)  
**Method:** POST  
**Injection Point:** CBSE Result Lookup Form (Roll Number, Admit Card ID, School Number, Mother's Name fields)  
**Backend Database:** MariaDB  
**Server Stack:** LAMPP  
**Vulnerable File:** `/opt/lampp/htdocs/subdomains/inventory/index.php` 

## Steps to Reproduce
1. Navigate to `https://inventory.pwnedbox.site` — the DigiLocker CBSE Class XII Results page.
2. Enter a single quote in any field (e.g., Roll Number) and submit:
   ```
   Roll Number: 1662616'
   Admit Card ID: AA232323'
   School Number: 51116161
   Mother's Name: ASFDAFA'
   ```
   → Application throws a fatal SQL error leaking the DB engine, server path, query type, and line number.

3. Inject a time-based blind payload in `roll_no` via Burp Repeater:
   ```
   roll_no=1'XOR(94102*if(now()=sysdate(),sleep(5),0))XOR'Z&class=XII&admit_card_id=AA232323%27&school_no=51116161&mother_name=ASFDAFA%27&terms=on
   ```
   → Response delayed by ~5 seconds confirming blind SQL injection.

4. Confirm with 10-second sleep via Burp Intruder:
   ```
   roll_no=1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
   ```
   → Response time: **10,217 ms** (10-second delay confirmed).

5. Confirm arithmetic execution with multiplied sleep:
   ```
   roll_no='XOR(if(now()=sysdate(),sleep(5*5),0))OR'
   ```
   → Request timed out (25-second sleep confirmed).


## POC (Proof of Concept)
<img width="1918" height="1028" alt="1" src="https://github.com/user-attachments/assets/bb4604e5-248d-44c0-90f2-0c51419694f7" />
<img width="1918" height="1025" alt="2" src="https://github.com/user-attachments/assets/46d5cf92-f638-46a4-9a98-3c90e62148cc" />
<img width="1918" height="1022" alt="3" src="https://github.com/user-attachments/assets/e4dc8aa8-290d-4a28-8827-bbdddbc39042" />
<img width="1918" height="1030" alt="4" src="https://github.com/user-attachments/assets/af934e3e-573c-4418-b3f7-d72c0bda77a8" />

## Impact
- **Sensitive Student PII Exfiltration** – The form handles CBSE student records (roll numbers, admit card IDs, school codes, mother's names, exam results). All data can be extracted via blind techniques.
- **Mass Data Breach Risk** – A government/education portal storing student academic records is a high-value target; full database dump is feasible with automated tools (e.g., sqlmap).
- **Multiple Injectable Parameters** – At least four fields (`roll_no`, `admit_card_id`, `school_no`, `mother_name`) are vulnerable, widening the attack surface.
- **Verbose Error Disclosure** – Stack trace exposes internal server paths and query structure, reducing attack complexity.

## Remediation
1. **Use Prepared Statements** – Parameterize every field in the lookup query.
   ```php
   $stmt = $pdo->prepare(
       "SELECT * FROM results WHERE roll_no = ? AND admit_card_id = ? AND school_no = ? AND mother_name = ?"
   );
   $stmt->execute([$roll_no, $admit_card_id, $school_no, $mother_name]);
   ```

2. **Validate Input** – Enforce strict type/format validation: roll numbers and school codes should be integers only; admit card IDs should match a known alphanumeric pattern.

3. **Suppress Error Messages** – Set `display_errors = Off` in `php.ini` to prevent stack trace and path leakage in production.

4. **Least Privilege DB User** – Application DB account should only hold `SELECT` on the results table — no write, `FILE`, or admin privileges given the read-only nature of this lookup.
