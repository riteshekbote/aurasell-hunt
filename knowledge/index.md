# Aurasell hunt KB — verified learnings (RAG for all models)
> Rules: never propose a class on the AURASELL REJECT LIST (see scope.yml); never
> duplicate KNOWN-DUP; use ALIVE surface facts. Aurasell policy explicitly rejects
> open-redirect-without-impact/clickjacking/cache-poisoning/content-spoofing/
> SPF-DKIM-DMARC/brute-force/non-reproducible/rate-limit/DoS/low-impact-CSRF/
> stack-traces/MITM/theoretical-subdomain-takeover/HSTS/version-banners.
> What pays: SQLi/RCE/LFI/PrivEsc/AuthBypass/DataExposure/XSS/CSRF/SSRF/
> open-redirect-to-token-theft/security-control-bypass/unauth-admin-interface.

## REJECTED CLASSES (Aurasell policy — do not propose)
- REJECTED open redirect without serious impact @ *: only token/credential-theft chains pay.
- REJECTED clickjacking / cache poisoning / content spoofing @ *: explicitly excluded.
- REJECTED SPF/DKIM/DMARC, HSTS, version banners, stack traces @ *: explicitly excluded.
- REJECTED brute-force / rate limit / DoS / DDoS @ *: explicitly excluded.
- REJECTED CSRF with minimal impact, MITM, theoretical subdomain takeover @ *: explicitly excluded.
- REJECTED third-party hosted systems @ *: unless they directly impact Aurasell.

## ALIVE SURFACE FACTS (verified)
- 2026-08-18 aurasell.ai + www.aurasell.ai: HTTP 200 (marketing site).
- 2026-08-18 app.aurasell.ai: 302 -> https://auth.aurasell.ai/authorize?state=...&audience=https%3A%2F%2Faurasell.us.auth0.com%2Fapi%2Fv2%2F&redirect_uri=https%3A%2F%2Fapp.aurasell.ai%2Fauth%2Fcallback&scope=openid%20profile%20email%20offline_access&response_type=code&code_challenge_method=S256&client_id=U7HwecmanN1eQcUiZonVHDLUmXygGSpm&code_challenge=...
  - Auth0 tenant: aurasell.us.auth0.com (US region). PKCE S256 + offline_access (refresh tokens).
  - Custom session cookie: session=<hmac-ish token>; Path=/; SameSite=Lax; HttpOnly
  - permissions-policy: camera=(), microphone=(self), geolocation=()
- 2026-08-18 auth.aurasell.ai: HTTP 200 (Auth0 authorize endpoint).
- DNS: aurasell.ai -> 64:ff9b::c6ca:d301 (IPv6-mapped IPv4 198.202.211.1).

## OPEN QUESTIONS
- Full subdomain inventory (app/auth confirmed; api/admin/dashboard/staging to verify).
- Auth0 tenant configuration: user enumeration on /dbconnections/signup? password reset flow?
- App API surface behind app.aurasell.ai/auth/callback (what APIs does the SPA call?)
- Admin console location (admin.aurasell.ai? subdomain or path on app?)
- CRM data models: prospects, accounts, sequences, templates — IDOR candidates.

## FINDING INBOX (validated = move to reports/)
- (empty)
## Session 3 intel (2026-08-18)
- **app1.aurasell.ai** — undocumented host discovered via APISIX redirect (Location: https://app1.aurasell.ai/auth/me). Next.js 14/15 app (data-color-mode light/dark), NextAuth-style routes:
  - /auth/login → 307 → auth.aurasell.ai/authorize (Auth0 OIDC, PKCE S256, audience https://aurasell.us.auth0.com/api/v2/)
  - /auth/logout → 307 → auth.aurasell.ai/oidc/logout?post_logout_redirect_uri=https://app1.aurasell.ai
  - /auth/callback → 500 "The state parameter is missing." (NextAuth callback)
  - /api/auth/* blocked by gateway (307 → /auth/login); /auth/register → 404
- **UNVALIDATED callbackUrl**: /auth/login?callbackUrl=<anything> embeds it raw into the Auth0 authorize redirect (tested: //evil.com, javascript:alert(1), https://app1.aurasell.ai.evil.com, relative paths). Post-login redirect behavior NOT yet verified (no account). If post-login redirect is unvalidated → open redirect; per program policy payable only if token/credential theft is proven (PKCE+code flow is server-side, so likely phishing-only → likely rejected per policy). TODO: verify with an account.
- Gateway: Apache APISIX 3.9.0 (server header); admin API not exposed (ports 9080/9443 closed, /apisix/admin 404/426).
- Session cookie: `session=<16B b64|exp_ts|<235B opaque|20B digest>` — custom HMAC/encrypted format (A=16 random bytes, B=expiry, C=opaque blob (not zlib, not Fernet), D=20B SHA1-sized MAC). Set pre-auth on app.aurasell.ai. Forging requires server secret — do NOT attempt offline brute force.
- Auth0 tenant: aurasell.us.auth0.com; client U7HwecmanN1eQcUiZonVHDLUmXygGSpm; signup via dbconnections closed ("email_verified needs to be true"). change_password: no user enumeration (uniform response).
- Marketing: Webflow site, GTM-N2GCB6QB, LinkedIn insight, reb2b; community at aurasellai.discourse.group (Discourse-hosted, not in scope). No self-serve signup; demo via app.cal.com.
- Program: security@aurasell.ai; page https://www.aurasell.ai/bug-bounty confirms $1000/$500/$200/$5 (Critical/High/Medium/Low).
