## Title
PulseMetrics Analytics — Hardcoded Secrets & Internal API Routes in Minified JS, Source Map Exposure & Infrastructure Topology Disclosure in HTML Source

## Description
**Affected URL:** https://analytics.pwnedbox.site/
The PulseMetrics Analytics platform exposes sensitive secrets and internal infrastructure details through two vectors. First, the publicly accessible minified JavaScript file (/static/js/app.min.js) contains hardcoded production tokens, an internal diagnostic bearer token, an HMAC telemetry secret, a Sentry DSN, and a full map of internal API routes including dangerous admin endpoints. The file also references a source map (app.min.js.map) which, if accessible, would expose the complete unminified source code. Second, the HTML page source contains an infrastructure health widget revealing internal service names, component health, and internal pod IPs.

## Steps to Reproduce
1. Navigate to `https://analytics.pwnedbox.site` and view page source (`Ctrl+U`)
2. Locate `<script src="/static/js/app.min.js">` and navigate to `https://analytics.pwnedbox.site/static/js/app.min.js`
3. Search for `ENVIRONMENT` — observe the full config object with hardcoded secrets and internal routes
4. Note the last line: `// # sourceMappingURL=app.min.js.map`
5. Navigate to `https://analytics.pwnedbox.site/static/js/app.min.js.map` — if accessible, full unminified source code is exposed
6. Return to page source and search `js` — observe the infrastructure health widget in raw HTML leaking internal service names, metrics, and IPs

## POC (Proof of Concept)
<img width="1916" height="985" alt="b" src="https://github.com/user-attachments/assets/a7ef9e3e-8b72-4eae-bdce-724057f6ddbe" />
<img width="1915" height="998" alt="a" src="https://github.com/user-attachments/assets/24e83a40-4d1f-4772-9ef8-ba89d5d43ad6" />

## Impact
- `INTERNAL_DIAGNOSTIC_BEARER` exposed → attacker can call `/api/v2/internal/cluster/health`, `/api/v2/internal/tenants/debug-export`, and `/api/v2/internal/system/cache/flush` directly — cache flush causes service disruption; debug-export leaks all tenant data
- `HMAC_TELEMETRY_SECRET` exposed → attacker can forge signed telemetry events, poisoning analytics data
- `PUBLIC_INGESTION_TOKEN` exposed → unauthorized data ingestion into production pipeline
- `SENTRY_DSN` exposed → attacker can read all application error logs and stack traces from Sentry, revealing further vulnerabilities
- Source map (`app.min.js.map`) → full unminified source code recoverable, exposing all business logic, auth flows, and hidden endpoints

## Remediation
- **Remove all secrets from client-side JavaScript** — `INTERNAL_DIAGNOSTIC_BEARER`, `HMAC_TELEMETRY_SECRET`, and `SENTRY_DSN` must only exist server-side; use a build-time env injection that strips internal keys before bundling
- **Block source map files on production** — serve `.map` files only in development; add server rule to return 403 on `*.map` requests in production:

```nginx
location ~* \.map$ { deny all; }
```

- **Rotate all exposed tokens immediately**: ingestion token, diagnostic bearer, HMAC secret, and regenerate the Sentry DSN
- **Restrict internal API routes** (`/internal/*`) to server-side only — they must never be callable from the browser; enforce at the API gateway with IP allowlisting or mTLS
- **Remove the infrastructure health widget from client-rendered HTML** — internal service names, IPs, and health metrics must never appear in page source; serve them only to authenticated internal dashboards
