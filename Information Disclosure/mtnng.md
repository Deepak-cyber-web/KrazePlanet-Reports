## Title
MTN NC Captive Portal — Admin Credential Exposure via Hardcoded Credentials in Client-Side JavaScript

## Description
**Affected URL:** https://mtnng.pwnedbox.site/
The MTN NC Captive Portal & Mobucks Partner Console exposes the admin username and an MD5-hashed password directly inside client-side JavaScript source code. By viewing the page source, an attacker can extract the credentials, crack the weak unsalted MD5 hash using a public lookup tool, and authenticate to the admin panel at /admin/ — gaining full access to live subscriber telemetry data including phone numbers, real names, NIN/KYC references, IMSI numbers, account balances, and IP addresses.

## Steps to Reproduce
1. Navigate to `https://mtnng.pwnedbox.site/nc/?time_stamp=<timestamp>` and view the page source (`Ctrl+U`)
2. Search for `passwd` in the source — observe the following hardcoded credentials embedded in JavaScript:
```js
uid: "mtnng",
passwd: "5858ea228cc2edf88721699b2c8638e5",
plid: 73
```

3. Take the MD5 hash `5858ea228cc2edf88721699b2c8638e5` and submit it to `https://crackstation.net`
4. CrackStation returns the plaintext password: `welcome123`
5. Navigate to `https://mtnng.pwnedbox.site/admin/` and log in with:
    - **Username:** `mtnng`
    - **Password:** `welcome123`
6. Observe full access to the admin panel showing live subscriber records

## POC (Proof of Concept)
<img width="1917" height="989" alt="a" src="https://github.com/user-attachments/assets/d796ccba-f56c-4dac-a9b2-147d797f331c" />
<img width="1917" height="1003" alt="b" src="https://github.com/user-attachments/assets/835ad17d-7e28-46a9-8a33-97e6c8661993" />
<img width="1915" height="994" alt="c" src="https://github.com/user-attachments/assets/07a4b101-3d79-457d-a8b0-0363f9dc5a95" />


## Impact
- Admin panel fully compromised using credentials exposed in public JavaScript
- Weak, unsalted MD5 hash cracked instantly using a free public tool — no brute force required
- Live subscriber PII exposed: full names, phone numbers (MSISDN), NIN/KYC reference numbers, IMSI numbers, account balances, and cell IP addresses

## Remediation
- **Never embed credentials in client-side JavaScript** — authentication tokens/keys must only exist server-side
- Replace MD5 with a modern password hashing algorithm: **bcrypt**, **Argon2**, or **scrypt** with a unique salt per user
- Rotate the exposed credentials immediately — assume `mtnng:welcome123` is compromised
- Implement server-side authentication for the Mobucks API — use short-lived signed tokens (JWT/OAuth2) instead of static username/password pairs
