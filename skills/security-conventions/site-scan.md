# Whole-site deep scan (Acunetix-style)

Extends [dast-browser.md] from a single target to an **exhaustive site sweep**: crawl the entire app, then active-test the whole attack surface. **Requires the authorization gate ([authorization.md]) for the host before any active probe.** Engine = Playwright MCP (fall back to `curl`/`nuclei`/ZAP if not connected).

## Phase 0 — Seed & limits

- Confirm authorization for the exact host. Stay scope-locked to that host (and any explicitly allowed subdomains) — never wander off-host.
- Set caps so the scan terminates and stays polite: `--max-pages` (default ~200), `--depth` (default ~5), per-request timeout, global rate limit / concurrency. Honor `robots.txt` when the user asks; otherwise note you're ignoring it.

## Phase 1 — Deep crawl (BFS)

1. Seed from the URL + `sitemap.xml` + `robots.txt` + any user-supplied URL list.
2. Breadth-first follow same-host links via Playwright (`browser_navigate` → `browser_snapshot`), enqueue new same-host URLs up to the caps.
3. On each page capture: forms (fields/types/method), buttons, file uploads, and every XHR/fetch endpoint (`browser_network_requests`).
4. **Authenticated crawl:** if creds/token were supplied, log in through the UI (or set the header) and re-crawl to reach authenticated areas. Redact creds everywhere.
5. **Dedupe templated URLs:** collapse `/post/1`, `/post/2`, `/user/42` … into one canonical template so you test the *pattern* once, not thousands of clones. Keep one representative per template.

Output of this phase = a **site map**: unique pages + a deduped list of testable inputs `{url-template, method, params, content-type, auth-required}`.

## Phase 2 — Active test the whole surface

Run the full attack catalog from [dast-browser.md] against each unique input/template (XSS reflected/stored/DOM, SQLi, command injection, SSRF, path traversal, auth bypass, IDOR/BOLA, CSRF, open redirect, auth/session, misconfig, sensitive-data). Add a `nuclei -u <host>` breadth pass; ZAP full-crawl when present. Rate-limited, non-destructive, AI-reasoning over each response to confirm real vs not.

## Phase 3 — Site report + coverage map

- Findings ranked by severity (dedupe identical issues seen on many pages → one finding with the affected templates listed), each with request, evidence/screenshot, CVSS, `curl` repro, fix, OWASP/CWE tag.
- **Coverage map (mandatory):** pages discovered vs crawled, inputs tested vs skipped, why anything was skipped (cap/rate-limit/auth-wall), and what was NOT reached. Never imply full coverage you didn't achieve.
