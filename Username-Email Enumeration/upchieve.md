## Title
User enumeration through forget password

## Description
**Affected URL:** https://upchieve.pwnedbox.site/
The forgot-password endpoint of UPchieve (Free Online Tutoring platform) returns different responses depending on whether the submitted email address is registered in the system:

- **Non-existing email** → Returns: `"No account with that id found."`
- **Existing email** → Returns: `"If that email address exists in our system, we will send a password reset link to your inbox."`

The success message attempts to be ambiguous, but the **explicit error message for non-existing accounts** directly reveals account existence, allowing an unauthenticated attacker to enumerate valid user emails with certainty.

## Steps to Reproduce
1. Navigate to `https://upchieve.pwnedbox.site/` and open the **Forgot Password** page.
2. Enter a **non-registered** email (e.g., `hello@gmail.com`) and click **Send Reset Link**.
3. Observe the response: **"No account with that id found."** — explicitly confirms the account does **not** exist.
4. Enter a **registered** email (e.g., `tutor@upchieve.org`) and click **Send Reset Link**.
5. Observe the response: **"If that email address exists in our system, we will send a password reset link to your inbox."** — by contrast, confirms the account **does** exist.
6. Any email that does **not** return the error message is a confirmed valid account. Automate with a wordlist to enumerate at scale.

## POC (Proof of Concept)
<img width="1920" height="1003" alt="1" src="https://github.com/user-attachments/assets/a9b9c240-c86c-468d-a17c-032f980ac062" />
<img width="1917" height="999" alt="2" src="https://github.com/user-attachments/assets/cb2bf772-aec4-4e59-b880-4a5d9fa38d27" />

## Impact
- **Account enumeration at scale** — attacker can build a confirmed list of registered emails using only the error/success response difference.
- **Targeted attacks** — confirmed accounts (especially `admin@upchieve.org` and `tutor@upchieve.org`) become direct targets for credential stuffing or phishing.

## Remediation
- **Remove the explicit error message.** Always return the same generic response regardless of whether the email exists:  
    _"If that email address exists in our system, we will send a password reset link to your inbox."_
- **Never** return messages like `"No account with that id found."` — this is the sole root cause of the vulnerability.
- **Add rate-limiting** on the forgot-password endpoint to prevent automated enumeration.
