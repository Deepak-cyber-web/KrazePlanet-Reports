## Title
Email Enumeration on the Sign-up Page
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://exchange.pwnedbox.site/
The sign-up page of GO Exchange (a cryptocurrency trading platform) discloses whether an email address is already registered by returning different responses based on the input:

- **Existing email** → Returns: `"Email is invalid."`
- **Non-existing email** → Returns: `"Success! A verification link has been sent to your email address."`

This behavioral difference allows an unauthenticated attacker to enumerate valid user email addresses without any authentication or rate-limiting.


## Steps to Reproduce
1. Navigate to `https://exchange.pwnedbox.site/`
2. Enter a **registered** email address (e.g., `john@example.com`) in the Email Address field and click **Sign Up**.
3. Observe the error response: **"Email is invalid."** — this silently confirms the account exists.
4. Now enter a **non-registered** email address (e.g., `test@gmail.com`) and click **Sign Up**.
5. Observe the success response: **"Success! A verification link has been sent to your email address."** — confirming the email does **not** exist.
6. Repeat with a wordlist to enumerate valid accounts at scale.

## POC (Proof of Concept)
<img width="1920" height="1007" alt="1" src="https://github.com/user-attachments/assets/24114f4a-3ec3-49fb-a1e0-4d7385a1138c" />
<img width="1923" height="1000" alt="2" src="https://github.com/user-attachments/assets/e816c0fa-c2ab-48e2-bdf6-c7863b153d80" />

## Impact
An attacker can silently confirm which email addresses have accounts on the platform, enabling:

- **Credential stuffing** — confirmed emails paired with breached password lists for targeted login attacks.
- **Spear phishing** — verified account holders (especially `admin@exchange.com`) become high-value phishing targets.

## Remediation
- **Return a uniform response** for both existing and non-existing emails:  
    _"If this email is not registered, a verification link has been sent."_
- **Never distinguish** between "already registered" and "invalid email" errors on the sign-up form.
- **Implement rate-limiting** on the registration endpoint to prevent automated enumeration.
