# aurasell-hunt

24/7 multi-model bug-hunting automation for the **Aurasell Bug Bounty** program.

- **Scope**: Aurasell web apps, admin consoles, APIs, other Aurasell-owned production services (`aurasell.ai` + subdomains)
- **Disclosure**: email `security@aurasell.ai` (policy: https://www.aurasell.ai/bug-bounty)
- **Rewards (USD)**: Critical $1000 / High $500 / Medium $200 / Low $5
- **Triage**: within 10 business days; payout within 20 business days after remediation
- 5 opencode models (Big Pickle, Nemotron 3 Ultra, Longcat, Ling 3.0, Laguna) hunt in parallel every 10 minutes
- Subdomain recon pipeline (subfinder + crt.sh + wayback + dnsx + httpx) daily at 02:20 UTC
- JS recon pipeline (endpoint/sourcemap/secret extraction from live app bundles) every 5 minutes
- All testing **read-only / non-destructive** — policy forbids data access/modification, DoS, and any disclosure outside Aurasell; PoC artifacts must be destroyed after the report is closed

## Program rules
- Only test accounts you personally own
- If a vuln provides system-level access: **stop testing immediately** and report
- No status-update requests before 10 business days
- All PoCs (code/screenshots/videos) must be destroyed after the report closes

## Aurasell policy — what is NOT reportable (per their bug-bounty page)
Open redirects **without serious impact**, clickjacking, cache poisoning, content spoofing, missing SPF/DKIM/DMARC, brute force, non-reproducible issues, missing rate limiting, DoS/DDoS, CSRF with minimal impact, stack traces/path disclosures, MITM, theoretical subdomain takeovers, missing HSTS, informational headers/version banners, third-party hosted systems (unless directly impacting Aurasell). See `scope.yml` for the full rejected list.

## What pays
SQLi, RCE, directory traversal, privilege escalation, auth/authz bypass, sensitive data exposure, XSS, LFI/RFI, CSRF, SSRF, **open redirects that lead to token or credential theft**, security control bypasses, administrative interfaces without authentication.

## Stack (verified seed)
- **Auth0** tenant `aurasell.us.auth0.com` (API audience `https://aurasell.us.auth0.com/api/v2/`)
- `app.aurasell.ai` → 302 → `auth.aurasell.ai/authorize` with PKCE S256 + `offline_access`
- OAuth client_id `U7HwecmanN1eQcUiZonVHDLUmXygGSpm`
- Custom session cookie (HMAC-signed, HttpOnly, SameSite=Lax) from a custom backend

| Artifact | Purpose |
|---|---|
| `recon/scope.txt` | Seed subdomain list |
| `inventory/` | Recon + JS inventory results |
| `leads/` | Candidate findings (UNVALIDATED) |
| `reports/` | Ranked hypotheses + valid findings |
| `knowledge/index.md` | Verified learnings + rejected classes (RAG) |
| `findings.md` | JS recon output |
| `scope.yml` | Program scope + rules (edit to adjust) |

## Reporting
Email `security@aurasell.ai` with: vulnerability description, reproduction steps, affected system/endpoint, date of discovery, PoC details. Rewards at Aurasell's sole discretion (severity, complexity, business impact, originality).