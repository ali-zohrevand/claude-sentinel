---
description: Security-review one API endpoint — white-box review of its route source (always) plus an optional live probe of the endpoint (authorization-gated), through the OWASP API Top 10 lens.
argument-hint: <endpoint-url | path/to/openapi#operationId | route file:line>
---

Security review of a single API: **$ARGUMENTS**

## 1. Resolve the target

Load `sentinel:security-conventions` → [api-testing.md]. The arg is either a live endpoint URL, a pointer into an OpenAPI/Swagger spec (operation), or a source route. Locate the handler source (Read/Grep) and/or the spec operation.

## 2. White-box (always, no gate)

Dispatch **whitebox-checker** on the route handler + **owasp-reviewer** for the API Top 10 lens: BOLA/object-ownership (API1), auth (API2), mass assignment / excessive data exposure (API3), resource limits (API4), function-level authz (API5), SSRF (API7), misconfig (API8), unsafe upstream consumption (API10). Include the input-validation/authz status for the endpoint.

## 3. Live probe (only if the arg is a live URL AND authorized)

If a running endpoint is targeted, enforce the authorization gate ([authorization.md]) for its host, then dispatch **blackbox-dast** to probe just that endpoint (per-operation tests from [api-testing.md] / [dast-browser.md]), redacting any auth token. Prefer the **`postman:security`** skill if a Postman collection/OpenAPI spec is available.

## 4. Consolidate

Dispatch **security-advisor** → one severity-ranked report with OWASP API tags, evidence (`file:line` and/or request/response), CVSS, and fixes. `--json` if requested.
