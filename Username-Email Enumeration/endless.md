## Title
Enumeration of username on password reset page
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://endless.pwnedbox.site/
Vulnerability 1 — Username Enumeration (CWE-204):
The endpoint returns different responses based on whether the submitted username exists:

Existing username → Returns: "Password reset code sent. Your code: ce6a2071"
Non-existing username → Returns: "If a user with that username exists, a reset code will be sent."

Vulnerability 2 — Password Reset Token Leaked in Response (CWE-640):
For valid usernames, the password reset code is returned directly in the HTTP response body and rendered on-screen (Your code: ce6a2071). An attacker who knows or guesses a valid username can immediately obtain the reset token without access to the target's email or phone, and use it to take over the account.

Note: These two issues chain together — Vuln 1 confirms valid usernames; Vuln 2 allows immediate account takeover for any confirmed username.

## Steps to Reproduce
**To reproduce Username Enumeration:**

1. Navigate to `https://endless.pwnedbox.site/` and open the **Request Password Reset** page.
2. Enter a **non-existing** username (e.g., `cyberbro`) and click **Request Password Reset**.
3. Observe the generic response: **"If a user with that username exists, a reset code will be sent."**
4. Enter a **valid** username (e.g., `codermak`) and click **Request Password Reset**.
5. Observe the different response — the account is confirmed **and** the reset code is disclosed.

**To reproduce Token Disclosure (Account Takeover):**

1. Enter any known or guessed valid username (e.g., `admin`, `developer`, `codermak`) and click **Request Password Reset**.
2. If the account exists, the response immediately reveals: **"Password reset code sent. Your code: `<token>`"**
3. Use the disclosed token to reset the password and take over the account — no email access needed.

## POC (Proof of Concept)
<img width="1915" height="999" alt="1" src="https://github.com/user-attachments/assets/bac99e21-f075-4387-9eac-6ea1f83806e1" />
<img width="1917" height="998" alt="2" src="https://github.com/user-attachments/assets/183b25c8-b785-4ef7-ad55-2e6264900f66" />


## Impact
- **Account Takeover (High)** — Reset tokens returned in-response bypass email verification entirely; an attacker can take over `admin` or `developer` accounts without any access to the victim's inbox.
- **Username enumeration** — Confirmed usernames can be fed into brute-force or credential-stuffing attacks against the admin panel.
- **Full admin panel compromise** — DirectAdmin is a server control panel; compromise of the `admin` account provides server-level access (files, databases, DNS, email).

## Remediation
- **Never return the reset token in the HTTP response.** Send it exclusively to the user's registered email or phone. The on-screen display of `Your code: <token>` is the root cause of the token disclosure.
- **Return a uniform response** for all username inputs:  
    _"If a user with that username exists, a reset code will be sent."_ — identical for both valid and invalid usernames.
- **Use short-lived, single-use tokens** and invalidate them immediately after use or after a timeout (e.g., 15 minutes).
- **Rate-limit the reset endpoint** to prevent automated username enumeration and token fishing.
