## Title
VaultTech - Hardcoded Admin Credentials in JavaScript & Credential Reuse
<!-- Write a short, clear title describing the vulnerability and where it was found. Example: Stored XSS in User Profile -->

## Description
**Affected URL:** https://vault.pwnedbox.site/
The VaultTech Enterprise Payment Processing platform exposes plaintext admin credentials and internal route paths directly inside a publicly accessible JavaScript file (/static/js/runtime.js). The file is referenced via a <script> tag in the HTML page source and requires no authentication to access. Using the exposed credentials, an attacker can authenticate to both the Administration Console (/admin/) and the Operations Center (/manage/) as Super Admin, gaining full control over all operator accounts, production API/signing keys, and live financial settlement data for 1,847 merchants processing $48.2M/month.

## Steps to Reproduce
1. Navigate to `https://vault.pwnedbox.site` and view the page source (`Ctrl+U`)
2. Locate the script tag on line 11:

html

```html
<script src="/static/js/runtime.js" defer></script>
```

3. Navigate directly to `https://vault.pwnedbox.site/static/js/runtime.js`
4. Search for `pass` — observe the `getControlPlaneCredentials()` function containing plaintext credentials:

js

```js
username: "vt_admin",
password: "V@ultT3ch!Ops2026"
```

5. Also note the exposed internal routes in `VaultConfig`:

js

```js
internalRoutes: {
    admin: "/admin/",
    operations: "/manage/"
}
```

6. Navigate to `https://vault.pwnedbox.site/admin/` and login with `vt_admin` / `V@ultT3ch!Ops2026`
7. Observe full **Super Admin** access to Administration Console — all operator accounts and API keys visible
8. Navigate to `https://vault.pwnedbox.site/manage/` — same credentials grant access to Operations Center with full merchant settlement data

## POC (Proof of Concept)
<img width="1918" height="1000" alt="a" src="https://github.com/user-attachments/assets/ae0ef213-1b63-4adb-b11a-3cc414500891" />
<img width="1917" height="994" alt="b" src="https://github.com/user-attachments/assets/4f551409-4622-4c07-a94e-71a8cb322215" />
<img width="1916" height="1006" alt="c" src="https://github.com/user-attachments/assets/4c94eb1c-7eee-4303-838c-6e13e09c4e21" />
<img width="1919" height="1000" alt="d" src="https://github.com/user-attachments/assets/99d052fa-de1c-404a-b09b-13204a28b27f" />


## Impact
- **Full Super Admin takeover** via plaintext credentials embedded in publicly accessible JavaScript — no cracking, no brute force required
- **Production API signing keys exposed** (`vt_live_84c1...`) with `charges:write, payouts:write` scope → attacker can initiate fraudulent charges and payouts against $48.2M/month in settlement volume
- **Webhook HMAC key exposed** (`whsec_vt_918c...`) → attacker can forge webhook events and manipulate payment event processing
- **All 5 operator accounts exposed** with emails and roles → enables targeted phishing, account takeover of remaining staff
- **Super Admin account has MFA disabled** — combined with exposed plaintext password, represents the highest possible account compromise risk

## Remediation
- **Immediately remove all credentials from JavaScript files** — client-side code is fully visible to any user; it must never contain secrets of any kind
- **Rotate all exposed secrets immediately:**
    - Admin password `V@ultT3ch!Ops2026`
    - Production Settlement Key (`vt_live_84c1...`)
    - Webhook HMAC Dispatcher key (`whsec_vt_918c...`)
    - Internal Telemetry Key (`vtk_diag_910a...`)
- **Enable MFA on `vt_admin`** — the highest-privilege account must not have MFA disabled under any circumstances
- Use **server-side secrets management** for all credentials and API keys (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)
- **Remove `getControlPlaneCredentials()`** function entirely — credentials must never be returned to the client
