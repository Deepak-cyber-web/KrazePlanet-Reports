## Title
User Enumeration when adding email to account

## Description
**Affected URL:** https://weblate.pwnedbox.site/
The "Add Email Address" feature in the Account Settings page of Weblate returns different responses depending on whether the submitted email is already associated with another account:

Non-existing email → Returns: "Email address added successfully."
Existing/registered email → Returns: "This email address is already in use."

An authenticated attacker (even with the lowest privilege role) can exploit this response difference to enumerate all registered email addresses in the system by iterating through a wordlist and observing which emails trigger the "already in use" error.

## Steps to Reproduce
1. Log in to `https://weblate.pwnedbox.site/` with any valid account (e.g., `translator` / `translator@weblate.org`).
2. Navigate to **Account Settings** → `https://weblate.pwnedbox.site/index.php?action=account`.
3. Scroll to the **Add Email Address** section.
4. Enter a **non-existing** email (e.g., `hello@gmail.com`) and click **Add Email**.
5. Observe the response: **"Email address added successfully."** — confirms the email is **not** registered.
6. Enter a **registered** email (e.g., `translator@weblate.org`) and click **Add Email**.
7. Observe the response: **"This email address is already in use."** — confirms the email **is** registered.
8. Repeat with a wordlist to enumerate all registered accounts at scale.

## POC (Proof of Concept)
<img width="1918" height="1003" alt="3" src="https://github.com/user-attachments/assets/7cec058f-f52f-45f1-8f9d-27a619275bd2" />
<img width="1918" height="999" alt="4" src="https://github.com/user-attachments/assets/5c5ebe67-b4ad-465c-bb83-682d5e560cb4" />

## Impact
- **Horizontal enumeration** — any authenticated user (even lowest privilege) can enumerate emails of all other users including admins and developers.
- **Privilege escalation precursor** — confirming `admin@weblate.org` exists provides a verified target for credential stuffing or phishing to gain elevated access.
- **Internal reconnaissance** — on a translation/collaboration platform, enumerating all contributor emails reveals organizational structure and personnel.

## Remediation
- **Return a uniform response** regardless of whether the email exists in the system:  
    _"If this email is not already associated with an account, it has been added."_
- **Suppress the "already in use" error** — silently ignore or queue the request without confirming the email's status to the requesting user.
- **Rate-limit the Add Email endpoint** to prevent automated iteration, even for authenticated users.
