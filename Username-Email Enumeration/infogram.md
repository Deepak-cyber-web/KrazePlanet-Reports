## Title
User Enumeration on Infogram

## Description
**Affected URL:** https://infogram.pwnedbox.site/
The forgot-password page of Infogram (Infographics & Data Visualization platform) returns different responses depending on whether the submitted email address is registered:

Non-existing email → Returns: "No account with that email address found."
Existing email → Returns: "If that email address exists in our system, we will send a password reset link to your inbox."

The success message is intentionally generic, but the explicit error message for non-existing accounts directly reveals account non-existence — making the contrast between the two responses sufficient to enumerate valid accounts with certainty.

## Steps to Reproduce
1. Navigate to `https://infogram.pwnedbox.site/` and open the **Forgot Password / Send Reset Link** page.
2. Enter a **non-registered** email (e.g., `hello@gmail.com`) and click **Send Reset Link**.
3. Observe: **"No account with that email address found."** — explicitly confirms account does **not** exist.
4. Enter a **registered** email (e.g., `designer@infogram.com`) and click **Send Reset Link**.
5. Observe: **"If that email address exists in our system, we will send a password reset link to your inbox."** — by contrast, confirms account **exists**.
6. Any email that does **not** return the explicit error is a confirmed valid account. Automate with a wordlist to enumerate at scale.

## POC (Proof of Concept)
<img width="1917" height="993" alt="3" src="https://github.com/user-attachments/assets/2899a0e2-1f26-49e5-b79f-0f1b34b97872" />
<img width="1917" height="999" alt="4" src="https://github.com/user-attachments/assets/6516fd8e-f334-4aee-a1f5-7dd4bcaa2f5b" />

## Impact
- **Account enumeration at scale** — confirmed email list enables targeted credential stuffing or password spraying against the platform.
- **User privacy** — membership on a data visualization/infographics platform may reveal business or organizational affiliations.

## Remediation
- **Remove the explicit error message.** Always return the same response regardless of whether the email exists:  
    _"If that email address exists in our system, we will send a password reset link to your inbox."_
- **Never** expose `"No account with that email address found."` or any variant — this single message is the entire root cause.
- **Rate-limit** the reset endpoint to prevent automated enumeration.
