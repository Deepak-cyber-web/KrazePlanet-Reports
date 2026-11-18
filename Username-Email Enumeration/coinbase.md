## Title
User email enumuration using Gmail

## Description
**Affected URL:** https://coinbase.pwnedbox.site/
The "Invite Friends" feature on Coinbase allows users to submit a list of email addresses (one per line) to check which contacts already have accounts. The response explicitly labels each submitted email with one of two statuses:

Existing account → Status column: "Already a Coinbase user"
Non-existing account → Status column: "Invite sent"

This design is intentional for the invite feature's UX, but it creates a bulk email enumeration oracle — an attacker can submit hundreds of email addresses in a single request and receive a confirmed account-existence result for every one of them. No authentication, no CAPTCHA, and no rate-limit feedback is required.

## Steps to Reproduce
1. Navigate to `https://coinbase.pwnedbox.site/` and open the **Invite Friends** page.
2. Enter a **registered** email address (e.g., `bitcoiner@gmail.com`) in the textarea and click **Import Contacts & Invite**.
3. Observe the results table — the email shows status: **"Already a Coinbase user"** → confirms account **exists**.
4. Enter a **non-registered** email (e.g., `test@gmail.com`) and click **Import Contacts & Invite**.
5. Observe the results table — the email shows status: **"Invite sent"** → confirms account does **not** exist.
6. Submit a large wordlist (one email per line) in a single request to enumerate accounts in bulk.

## POC (Proof of Concept)
<img width="1913" height="998" alt="3" src="https://github.com/user-attachments/assets/fdd1bfbf-1f62-4d7f-aaa6-d726fab6bfe3" />
<img width="1918" height="1002" alt="4" src="https://github.com/user-attachments/assets/420cea20-7745-4592-8045-e34e2e9a17b9" />

## Impact
- **Bulk enumeration in a single request** — unlike single-email endpoints, this feature processes an entire wordlist per request, allowing an attacker to confirm thousands of registered accounts per minute with minimal infrastructure.
- **Financial platform membership disclosure** — confirming that a specific email has a Coinbase account reveals sensitive financial affiliation data, violating user privacy.

## Remediation
- **Remove explicit account-status disclosure** from the invite response. The feature can still work without revealing which specific emails are registered — simply send invites silently and show a generic `"Invite processed"` status for all submitted emails.
- **Do not label individual emails** as "Already a user" in the UI response — this is the sole root cause of the enumeration.
- **Implement per-request rate-limiting** on the number of emails that can be submitted in a single invite request (e.g., max 10 per request, with a daily cap per user/IP).
