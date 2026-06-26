# White-box / source-review patterns

Source-level review (no network). Combine manual taint reasoning with the SAST/secrets/dep scanners in [scanners.md](scanners.md). Reuse `squad:stack-conventions` for per-language idioms when present. Trace: **source (untrusted input) → propagation → sink**, and confirm a control exists at the boundary.

## Universal data-flow targets

- **Sources:** request params/body/query/headers/cookies, file uploads, env from untrusted callers, message queues, **LLM output**, third-party API responses.
- **Sinks:** SQL/ORM raw, OS/shell exec, file path ops, template/HTML render, HTTP client (SSRF), deserializers, `eval`/dynamic code, redirects.

## Vulnerability classes to check

- **Injection (A03):** untrusted data concatenated into SQL/NoSQL/LDAP/OS-command/HTML/template. Want: parameterized queries, escaping, allowlists. Flag string-built queries and shell calls.
- **Code execution / RCE:** `eval`, dynamic `require/import`, deserialization of untrusted data, unsafe reflection, command exec with user input, unrestricted file upload + include.
- **Broken access control / IDOR (A01):** records fetched by id with no ownership/tenant check; missing role/function authz; client-trusted authorization.
- **Auth / session / JWT (A07):** weak password handling, JWT `alg:none`/unverified signature/weak secret, missing expiry, session fixation, missing MFA, insecure cookie flags (HttpOnly/Secure/SameSite), missing CSRF protection on state-changing requests.
- **Crypto (A02):** hardcoded keys, weak algorithms (MD5/SHA1 for passwords, ECB), static IV/nonce, predictable randomness for security, plaintext secrets/PII at rest.
- **Secrets:** keys/passwords/tokens in code **and git history** — run gitleaks/trufflehog; report locations only.
- **SSRF (A10):** server fetches a user-controlled URL with no allowlist/scheme/host restriction.
- **Security headers / misconfig (A05):** missing or weak CSP (watch `unsafe-eval`/`unsafe-inline`), HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy; verbose errors/stack traces; debug enabled; directory listing; default creds.
- **Dependencies (A06):** known-CVE/outdated packages — cross-check the dependency scanners; flag with CVSS + upgrade path.
- **AI/LLM (LLM Top 10):** untrusted text concatenated into prompts without delimiting/guarding (prompt injection); **LLM output used in a sink** (SQL/HTML/shell/HTTP) without validation (LLM05); tools/functions exposed to the model with excessive authority (LLM06); secrets/system-prompt reachable in responses (LLM02/LLM07); no output schema/allowlist; no spend/rate caps (LLM10).
- **Logging (A09):** insufficient audit logging, secrets in logs, no alerting.

## Per-stack quick notes

- **PHP:** raw `$_GET/$_POST` → query/`exec`/`include`; prefer prepared statements (named params) + a data-access layer; `declare(strict_types=1)`; no closing `?>`; watch CSP set in view/header code.
- **TypeScript/Node/Nest:** validate at the boundary (DTO + `class-validator` + global `ValidationPipe` whitelist); avoid `any`; no `eval`/`child_process` with user input; check JWT guard config.
- **Vue/HTML/JS:** `v-html`/`innerHTML` with untrusted data (XSS/DOM); CSP; avoid client-trusted authz.
- **Go:** `database/sql` with placeholders not `fmt.Sprintf`; `os/exec` with user input; `html/template` (auto-escapes) over `text/template` for HTML; wrap/handle every error.
