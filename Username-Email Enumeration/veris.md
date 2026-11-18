## Title
User enumeration via error message

## Description
**Affected URL:** https://veris.pwnedbox.site/
The login page of Veris (Secure Authentication Platform) returns two distinct error messages depending on whether the submitted username exists in the system:

Non-existing username → Returns: "User not exist"
Existing username + wrong password → Returns: "Password does not match"

An unauthenticated attacker can exploit this difference to silently confirm which usernames are registered, without needing to know any passwords. This is the most common and direct form of username enumeration — exposed through the application's own login error handling.

## Steps to Reproduce
1. Navigate to `https://veris.pwnedbox.site/`.
2. Enter a **non-existing** username (e.g., `cyberbro`) with any password and click **Login**.
3. Observe the error: **"User not exist"** — confirms the username is **not** registered.
4. Enter a **valid** username (e.g., `user1`) with a wrong password and click **Login**.
5. Observe the error: **"Password does not match"** — confirms the username **exists**.
6. Automate with a username wordlist to enumerate all valid accounts.

## POC (Proof of Concept)
<img width="1918" height="995" alt="1" src="https://github.com/user-attachments/assets/58ccff09-c63b-4d35-bec1-98b384c413f7" />
<img width="1916" height="1000" alt="2" src="https://github.com/user-attachments/assets/77d910a9-760e-4f13-ae9a-5dbcf46a41ca" />

## Impact
- Impact- **Username enumeration at scale** — an attacker can build a confirmed list of valid usernames using only the login error response, with no lockout risk if rate-limiting is absent.
- **Brute-force amplification** — confirmed usernames narrow the attack surface for password spraying (e.g., spraying `admin123`, `password1` only against confirmed accounts).
- **Admin account confirmed** — `admin` username being enumerable makes it a verified target for takeover, which on a "Secure Authentication Platform" is especially critical.

## Remediation
- **Never distinguish** between "user not found" and "wrong password" — both failures must produce identical messages, identical HTTP status codes, and ideally identical response times.
- **Implement rate-limiting and account lockout** after repeated failed attempts to prevent automated enumeration and brute-force attacks.
