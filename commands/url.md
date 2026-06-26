---
description: Black-box DAST of a running URL — AI-driven browser crawl + active attack testing (Acunetix-style) via the Playwright MCP, plus CLI DAST tools. REQUIRES explicit per-host authorization before any active probe.
argument-hint: <url> (you will be asked to confirm authorization)
---

Black-box security test of a live target: **$ARGUMENTS**

## 1. Authorization gate (MANDATORY — do this first)

Load `sentinel:security-conventions` → [authorization.md]. Extract the host from `$ARGUMENTS` and print the **AUTHORIZATION REQUIRED** block naming that exact host. **Do not send any active probe** until the user types the exact confirmation phrase. Until then, only passive checks (headers, TLS, public content) are allowed. No flag bypasses this gate.

## 2. Passive recon (always allowed)

Inspect security headers, TLS/cert (`testssl.sh`), public content/`robots.txt`. Confirm the Playwright MCP is connected (else degrade to `curl`/CLI DAST and say so).

## 3. Active DAST (only after authorization)

Dispatch in one message, passing the authorized host + any provided credentials (to be redacted):
- **blackbox-dast** → browser crawl + the active-test catalog from [dast-browser.md] (XSS, SQLi, command injection, SSRF, traversal, auth bypass, IDOR, CSRF, open redirect, misconfig, sensitive-data); `nuclei` breadth pass.
- **ceh-reviewer** → the five-phase live engagement framing + attack trees.
- **owasp-reviewer** → tag findings to OWASP/API categories.

Stay on the authorized host; non-destructive payloads; rate-limited.

## 4. Consolidate

Dispatch **security-advisor** to dedupe and produce ONE severity-ranked report — each finding with the exact request, response/screenshot evidence, a `curl` repro, CVSS, and fix. Report coverage (what was tested vs skipped). `--json` if requested.
