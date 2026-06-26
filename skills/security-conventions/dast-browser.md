# Browser-driven AI DAST (Acunetix-style, with AI)

How `blackbox-dast` (and `ceh-reviewer` for live targets) tests a running app like a DAST scanner — but with a real browser and AI reasoning over each response, instead of fixed signatures. **Requires the authorization gate ([authorization.md](authorization.md)) to have passed.**

## Engine: the Playwright MCP (the "browser emulator")

Use the Playwright MCP tools when connected (`mcp__plugin_playwright_playwright__browser_*`): `browser_navigate`, `browser_snapshot` (accessibility tree — the primary "what's on the page"), `browser_click`, `browser_type`, `browser_fill_form`, `browser_select_option`, `browser_press_key`, `browser_evaluate` (read DOM / check for sink execution), `browser_network_requests` (capture XHR/fetch + status/headers), `browser_console_messages`, `browser_take_screenshot` (evidence).

If the Playwright MCP is **not** connected in this session, degrade gracefully to `curl`/`WebFetch` + the CLI DAST tools in [scanners.md](scanners.md) (`nuclei`, `zap`, `sqlmap`, `nikto`, `testssl`) and say the browser engine was unavailable.

## Phase 1 — Crawl & map the attack surface

1. `browser_navigate` to the authorized target; `browser_snapshot` to read the page.
2. Discover surface: links (same-host only — never leave the authorized host), forms + their fields/types, buttons, file uploads, and every XHR/fetch endpoint via `browser_network_requests`.
3. Authenticated crawl: if the user supplied credentials/token, log in through the UI (or set the header) and re-crawl to reach authenticated surface. Redact creds in all output.
4. Build a target list: `{url, method, params, content-type, auth-required}` per input. Pace the crawl; respect robots where the user asks.

## Phase 2 — Active tests per input (the catalog)

For each discovered input, test the relevant classes; **reason over the response/DOM/network** to decide real vs not. Use the smallest proof. Map each hit to OWASP/CWE ([owasp.md](owasp.md)).

| Class | How to test (safe proof) | Confirm by |
|---|---|---|
| **Reflected/Stored XSS** | inject a unique marker payload (`"><svg onload=…marker>`); for stored, re-visit the render page | `browser_evaluate` shows the marker executed / DOM node created; not just echoed-escaped |
| **DOM XSS** | drive `location.hash`/inputs that feed `innerHTML`/`eval` sinks | sink fires in `browser_evaluate`/console |
| **SQL/NoSQL injection** | error-based (`'`), boolean/time-based (safe `SLEEP`-style only with consent) | response diff / timing / DB error in body; optionally `sqlmap` to confirm |
| **Command injection** | benign separators (`;id`, `|whoami`) where input reaches OS | command output reflected (no destructive payloads) |
| **SSRF** | point URL params at a controlled benign collaborator / `localhost` metadata only as proof | callback / differential response |
| **Path traversal / LFI** | `../` to a known-benign file | file content disclosed |
| **Auth bypass / IDOR / BOLA** | request another object id / drop the token / swap role | 200 + data you shouldn't see |
| **CSRF** | check state-changing POSTs for anti-CSRF token / SameSite | missing token accepted cross-origin |
| **Open redirect** | `?next=//evil.example` (don't actually follow off-host) | 3xx Location to attacker-controlled host |
| **Auth/session** | cookie flags (HttpOnly/Secure/SameSite), JWT `alg:none`/weak sig, fixation | inspect Set-Cookie / decode JWT |
| **Security misconfig** | headers, verbose errors/stack traces, dir listing, default creds, debug endpoints | response evidence |
| **Sensitive data exposure** | secrets/PII/keys in responses, source maps, `.git/`, backups | matched content (redact in report) |

`nuclei -u <target>` is a good breadth pass alongside the browser tests; ZAP CLI for a fuller automated crawl when present.

## Phase 3 — Report

Per finding: title, **severity + CVSS v3.1 vector**, OWASP/API/CWE tag, the **exact request** (method/url/headers/body, secrets redacted), the **response evidence** (or screenshot ref), a **copy-pasteable `curl` repro**, and the fix. Note coverage: which inputs/endpoints were tested vs skipped (rate-limit/scope) — never imply full coverage you didn't achieve.
