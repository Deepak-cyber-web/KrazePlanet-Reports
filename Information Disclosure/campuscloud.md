## Title
CampusCloud LMS — Unauthenticated Debug Interface Exposing Full Production Environment & All Infrastructure Credentials

## Description
**Affected URL:** https://campuscloud.pwnedbox.site/
The CampusCloud Enterprise LMS has a debug interface left enabled in production, publicly accessible with no authentication. The /debug endpoint returns a JSON index linking to sub-endpoints including /debug/environment.php, which dumps the complete production environment configuration. This single endpoint exposes every infrastructure secret in the application — database credentials, AWS IAM keys, Redis password, RabbitMQ credentials, SendGrid API key, SSO OIDC secret, internal service bearer token, and the application encryption key — all in plaintext JSON to any unauthenticated user.

## Steps to Reproduce
1. Navigate to `https://campuscloud.pwnedbox.site/debug` — observe JSON index of debug sub-endpoints returned with no auth
2. Navigate to `https://campuscloud.pwnedbox.site/debug/environment.php`
3. Observe the full production environment dump returned in plaintext JSON
4. Also check `https://campuscloud.pwnedbox.site/debug/info.php` and `/debug/status.php` for additional disclosures
   
## POC (Proof of Concept)
<!-- Add screenshots, HTTP requests/responses, payloads, or videos that demonstrate the vulnerability. -->

## Impact
- **AWS IAM keys exposed** → full access to `campuscloud-student-backups-us-east-1` S3 bucket containing student data; potential for broader AWS account takeover
- **PostgreSQL production DB credentials exposed** → direct access to `campuscloud_prod_db` containing all student/staff records
- **Encryption key exposed** → all data encrypted at rest can be decrypted; encrypted tokens/sessions can be forged
- **Internal service bearer token exposed** → attacker can impersonate backend services on internal APIs
- **Redis & RabbitMQ credentials exposed** → cache poisoning, session hijacking, message queue manipulation possible
- **SSO OIDC secret exposed** → attacker can forge authentication flows and impersonate any user
- **SendGrid API key exposed** → phishing emails can be sent on behalf of the platform

## Remediation
- **Disable all `/debug/*` endpoints immediately in production** — debug interfaces must never be reachable on public-facing servers
- Rotate every exposed secret immediately: DB password, AWS IAM keys, Redis password, RabbitMQ password, SendGrid key, SSO secret, bearer token, and encryption key
- Use environment variables injected at runtime via a secrets manager (AWS Secrets Manager, HashiCorp Vault) — never store secrets in app config files readable by a debug endpoint
- Add environment checks to block debug routes: `if (APP_ENV === 'production') { return 403; }`
- Restrict any remaining internal debug/actuator endpoints to localhost or internal VPN ranges only
