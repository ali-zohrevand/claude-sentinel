# Authorization gate (live/active probing)

Active scanning of a host you do not own — or lack written authorization to test — is illegal in most jurisdictions (e.g. CFAA/Computer Misuse Act) and unethical. CEH is *ethical* hacking: authorization first, always. This gate is mandatory and **cannot be bypassed by any flag, config, or prompt.**

## When the gate fires

Before the **first active/network probe** of any live target (the `url` scope, and the live half of `api` / `apis`). It does NOT fire for code-only review (diff, project, static OpenAPI/route review) — those never touch the network.

## The gate

Print exactly this and **stop**; do not send any request until the user types the confirmation verbatim:

```
⛔ AUTHORIZATION REQUIRED
About to send ACTIVE security probes to: <host>
This will send real attack traffic (injection, auth-bypass, SSRF, etc.).
Only proceed if you OWN this host or hold WRITTEN authorization to test it.

Type exactly:  I confirm I have authorization to test <host>
(or press Ctrl-C / say "abort" to cancel)
```

- Substitute `<host>` with the exact hostname (and port) of the target.
- Accept only the exact phrase with the matching host. Anything else → do not proceed; offer the passive-only review instead.
- One confirmation authorizes **one host** for **this run**. A different host re-triggers the gate. Do not persist authorizations to disk.

## Until authorized — passive only

If the user declines or hasn't yet confirmed, you may still perform **passive, non-intrusive** checks (no attack payloads):

- HTTP response headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, cookie flags).
- TLS/cert inspection (version, cipher, chain, expiry) via `testssl.sh` or an equivalent read-only check.
- Publicly served content / robots.txt / sitemap, and **static** review of any OpenAPI/route source you can read.

## Rules for authorized active probing (safe by default)

- **Scope lock:** stay on the authorized host. Never follow links/redirects to other hosts; never probe a host the user didn't name.
- **Non-destructive:** prefer safe/idempotent verbs; do not run payloads that delete/modify data, exfiltrate, or DoS. No `DROP`, no mass writes, no fork-bombs, no brute force without explicit per-run consent.
- **Rate-limit:** throttle requests; back off on 429/5xx. For `apis` (many endpoints) sample and pace; report what was and wasn't covered (no silent truncation).
- **Secret hygiene:** if probing requires an auth token, take it from the user/env, never log it, and redact it in all evidence/reports.
- **Evidence, not exploitation:** demonstrate a finding with the minimum proof (one reflected payload, one 200 on an object you shouldn't see). Don't pivot, persist, or escalate further than needed to prove impact.
