## Title
Enumeration of username on password reset page

## Description
**Affected URL:** https://endless.pwnedbox.site/
The endpoint returns different responses based on whether the submitted username exists:

Existing username → Returns: "Password reset code sent. Your code: ce6a2071"
Non-existing username → Returns: "If a user with that username exists, a reset code will be sent."

## Steps to Reproduce
1. Navigate to `https://endless.pwnedbox.site/` and open the **Request Password Reset** page.
2. Enter a **non-existing** username (e.g., `cyberbro`) and click **Request Password Reset**.
3. Observe the generic response: **"If a user with that username exists, a reset code will be sent."**
4. Enter a **valid** username (e.g., `codermak`) and click **Request Password Reset**.
5. Observe the different response — the account is confirmed **and** the reset code is disclosed.

## POC (Proof of Concept)
<img width="1920" height="998" alt="5" src="https://github.com/user-attachments/assets/f850c692-dacb-446f-a12a-b1b99510302d" />
<img width="1919" height="1001" alt="6" src="https://github.com/user-attachments/assets/796450e9-3740-4a28-bbdc-fb6bc7827032" />

## Impact
- **Username enumeration** — Confirmed usernames can be fed into brute-force or credential-stuffing attacks against the admin panel.
- Full admin panel compromise — DirectAdmin is a server control panel; compromise of the admin account provides server-level access (files, databases, DNS, email).

## Remediation
- **Return a uniform response** for all username inputs:  
	"If a user with that username exists, a reset code will be sent."_ — identical for both valid and invalid usernames.
- **Rate-limit the reset endpoint** to prevent automated username enumeration and token fishing.
- Use short-lived, single-use tokens and invalidate them immediately after use or after a timeout (e.g., 15 minutes).
