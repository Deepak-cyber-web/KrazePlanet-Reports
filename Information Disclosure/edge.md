## Title
KestrelEdge CDN — Misconfigured Directory Listing Exposing KYC PII, Employee Records, API Keys & Confidential Documents

## Description
**Affected URL:** https://edge.pwnedbox.site/
The KestrelEdge CDN platform has Apache/Nginx directory listing enabled on the /documents path, allowing any unauthenticated user to browse and directly download all files stored in that directory. The exposed files include a KYC verification batch archive, user data backup exports, employee records, an internal quarterly report, a confidential NDA contract, and a staging API keys file. No authentication, authorization, or access control of any kind is enforced on this path. The combination of KYC identity data, employee PII, and API credentials in a single unauthenticated directory represents a severe multi-vector data breach.

## Steps to Reproduce
1. Navigate to `https://edge.pwnedbox.site/documents` in a browser
2. Observe the server returns a full Apache-style directory index with no authentication prompt
3. Click any file to download it directly — no login, token, or session required
4. Note the file `api_keys_staging.txt` — directly accessible and likely contains live or staging API credentials

## POC (Proof of Concept)
<img width="1912" height="997" alt="e" src="https://github.com/user-attachments/assets/96188511-6a8e-4b91-a2d5-449a3b4d8da4" />

## Impact
- KYC batch, employee records, and user data backup exposed → mass PII/identity theft risk and GDPR violation
- Confidential NDA with TechCorp exposed → legal breach of confidentiality
- `api_keys_staging.txt` accessible → potential unauthorized API access if keys overlap with production

## Remediation
- Disable directory listing: `Options -Indexes` (Apache) or `autoindex off` (Nginx)
- Move `/documents` outside the web root; serve files only via authenticated server-side logic
- Rotate all API keys in `api_keys_staging.txt` immediately — treat as compromised
