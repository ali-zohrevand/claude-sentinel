---
description: Whole-website DAST — deep-crawl an entire running site (Acunetix-style) with the Playwright AI browser, then active-test the full attack surface across every page and endpoint. REQUIRES explicit per-host authorization. Rate-limited, scope-locked, read-only on the filesystem.
argument-hint: <url> [--max-pages N] [--depth D] [--auth creds] (asks for authorization)
---

Whole-site security scan of: **$ARGUMENTS**

## 1. Authorization gate (MANDATORY — first)

Load `sentinel:security-conventions` → [authorization.md]. Extract the host from `$ARGUMENTS`, print the **AUTHORIZATION REQUIRED** block naming it, and **send no active probe** until the user types the exact confirmation. A full-site active scan is high-traffic — only run it against a host the user owns or is authorized to test. No flag bypasses this.

## 2. Passive recon (allowed pre-auth)

Headers, TLS (`testssl.sh`), `robots.txt`/`sitemap.xml`, public content. Confirm the Playwright MCP is connected (else degrade to `curl`/`nuclei`/ZAP and say so).

## 3. Deep crawl + active test (after authorization)

Read [site-scan.md] and dispatch **blackbox-dast** to run it:
- **Crawl (BFS)** the whole site from the seed + sitemap/robots, follow same-host links to the `--max-pages`/`--depth` caps, authenticated crawl if creds were given (redacted), capture all forms + XHR endpoints, and **dedupe templated URLs**.
- **Active-test** every unique input/template with the full catalog ([dast-browser.md]) + a `nuclei` breadth pass. Rate-limited, non-destructive, scope-locked.

Also dispatch **ceh-reviewer** (five-phase live framing + attack trees) and **owasp-reviewer** (OWASP/API tagging) over the crawl results.

## 4. Consolidate

Dispatch **security-advisor** → ONE site-wide report: severity-ranked findings (identical issues across pages collapsed to one finding listing affected templates), each with request, evidence/screenshot, CVSS, `curl` repro, fix; plus the **coverage map** (pages discovered vs crawled, inputs tested vs skipped and why). `--json` if requested. Never imply coverage you didn't achieve.
