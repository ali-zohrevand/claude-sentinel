---
description: Security-review ALL APIs — enumerate every endpoint from the OpenAPI spec or the route source (white-box, always), with an optional authorization-gated live DAST crawl across all endpoints. Full OWASP API Top 10 coverage incl. shadow/deprecated endpoint detection.
argument-hint: <path/to/openapi | routes-dir> (live crawl asks for authorization)
---

Security review of **all APIs** (scope: **$ARGUMENTS** or auto-discovered). 

## 1. Enumerate the API surface (always, no gate)

Load `sentinel:security-conventions` → [api-testing.md]. Build the complete endpoint inventory `{method, path, params, auth, file:line}` from the OpenAPI/Swagger doc and/or by grepping route definitions in source. Cross-check spec vs code to flag **API9** shadow/undocumented/deprecated endpoints and version sprawl. No route is silently skipped — list any whose analysis is inconclusive.

## 2. White-box every endpoint (parallel, read-only)

Dispatch **owasp-reviewer** (API Top 10 across all operations + compliance table) and **whitebox-checker** (handler source review: BOLA/IDOR, authz, mass assignment, validation) over the inventory. Produce a severity-ranked **per-endpoint table**.

## 3. Optional live crawl (only if authorized)

If the user wants live testing, enforce the authorization gate ([authorization.md]) for the host, then dispatch **blackbox-dast** to crawl + probe each endpoint per [dast-browser.md] — **sampled and rate-limited**; report coverage (tested vs skipped). Tokens redacted. Prefer **`postman:security`** when a collection/spec exists.

## 4. Consolidate

Dispatch **security-advisor** → one report: the endpoint inventory table, ranked findings with OWASP API tags + evidence + CVSS + fixes, the compliance coverage table, and an explicit coverage statement. `--json` if requested.
