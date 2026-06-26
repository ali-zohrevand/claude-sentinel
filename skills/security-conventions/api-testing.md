# API security review

Covers one endpoint (`api`) or every endpoint (`apis`), from a spec and/or live (live = post-authorization). Lens = OWASP API Top 10 ([owasp.md](owasp.md)).

## Discover the API surface

- **From a spec:** parse OpenAPI/Swagger (`openapi.*`, `swagger.*`) or a Postman collection → list `{method, path, params, auth, request/response schema}` per operation.
- **From source (no spec):** grep route definitions by framework — NestJS `@Get/@Post/@Controller`, Express `app.get/router.*`, PHP front-controllers / `api/v1/*.php`, Go `mux`/`chi`/`gin` handlers, Vue/SPA `fetch`/axios calls. Record `file:line` per route.
- **API9 (inventory):** flag shadow/undocumented/deprecated endpoints and version sprawl (`/v1` vs `/v2`) — endpoints in code but not the spec, or vice-versa.

## Per-operation checks (API Top 10)

- **API1 BOLA:** does the handler verify the caller owns/ may access the object id? (the #1 API bug — every `GET/PUT/DELETE /thing/{id}` needs an ownership/tenant check).
- **API2 Broken Auth:** is auth required and correctly enforced? token validation, expiry, `alg`.
- **API3 Object Property authz:** mass assignment (binding unfiltered input to models) and excessive data exposure (returning more fields than needed — passwords, internal flags).
- **API4 Resource consumption:** pagination caps, rate limits, payload size limits, expensive queries.
- **API5 Function-level authz:** can a low-priv user hit admin/privileged functions?
- **API6 Sensitive business flows:** can automation abuse a sensitive flow (signup, purchase, send)?
- **API7 SSRF** · **API8 Misconfig** (CORS `*` + credentials, verbose errors, headers) · **API10 Unsafe upstream consumption.**

## Live probing (after the gate)

Use [dast-browser.md](dast-browser.md) / `curl` to send the per-operation tests. For `apis`, sample + rate-limit and **report coverage** (which ops tested vs skipped). Prefer the **`postman:security`** skill when a collection/spec exists — it runs an OWASP API audit you can fold into the report.

## Secret hygiene (mandatory)

Auth tokens/keys come from the user/env — never hardcode, never log, **redact in every request shown** in the report. Summarize responses; don't dump bodies containing PII/secrets (report the field + location instead).
