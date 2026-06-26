# OWASP checklists — Web, API, and LLM

Tag every finding to the relevant category. Prefer delegating this lens to `squad:security-owasp` when the `squad` plugin is installed; otherwise use this file. End with a **compliance coverage table** (per category: CLEAN / FINDINGS(n) / NOT-TESTED).

## OWASP Top 10 (2021, with 2025 direction)
- **A01 Broken Access Control** — IDOR, missing function/object authz, path traversal, force-browsing, CORS misconfig. (2025: still #1.)
- **A02 Cryptographic Failures** — weak/missing crypto, hardcoded keys, plaintext secrets/PII, weak hashing, bad TLS.
- **A03 Injection** — SQL/NoSQL/LDAP/OS-command/XSS; any untrusted input reaching an interpreter/sink.
- **A04 Insecure Design** — missing threat modeling, unsafe business-logic flows, no rate limits by design.
- **A05 Security Misconfiguration** — defaults, verbose errors/stack traces, dir listing, missing headers, debug on.
- **A06 Vulnerable & Outdated Components** — known-CVE deps (cross-check the dependency scanners).
- **A07 Identification & Auth Failures** — weak creds, broken session, JWT `alg:none`/weak sig, fixation, missing MFA.
- **A08 Software & Data Integrity Failures** — insecure deserialization, unsigned updates, untrusted CI/CD or plugins.
- **A09 Security Logging & Monitoring Failures** — no/insufficient logging, secrets *in* logs, no alerting.
- **A10 SSRF** — server fetches a user-controlled URL without allowlist.

## OWASP API Security Top 10 (2023)
- **API1 BOLA** (object-level authz) · **API2 Broken Authentication** · **API3 Broken Object Property Level Authz** (mass assignment / excessive data exposure) · **API4 Unrestricted Resource Consumption** · **API5 Broken Function Level Authz** · **API6 Unrestricted Access to Sensitive Business Flows** · **API7 SSRF** · **API8 Security Misconfiguration** · **API9 Improper Inventory Management** (shadow/deprecated/undocumented endpoints, version sprawl) · **API10 Unsafe Consumption of APIs** (trusting upstream/3rd-party responses).

## OWASP LLM / GenAI Top 10 (2025) — for AI-powered apps
- **LLM01 Prompt Injection** (direct + indirect via retrieved/user content) · **LLM02 Sensitive Information Disclosure** · **LLM03 Supply Chain** (models/datasets/plugins) · **LLM04 Data & Model Poisoning** · **LLM05 Improper Output Handling** (LLM output → injection/XSS/SSRF/RCE downstream) · **LLM06 Excessive Agency** (over-broad tools/permissions/autonomy) · **LLM07 System Prompt Leakage** · **LLM08 Vector/Embedding Weaknesses** (RAG) · **LLM09 Misinformation/over-reliance** · **LLM10 Unbounded Consumption** (cost/DoS, model extraction).
- Practical checks: untrusted text concatenated into prompts without delimiting/guarding; LLM output used in SQL/HTML/shell/HTTP without validation; tools/functions exposed to the model with more authority than needed; secrets or system prompt reachable in responses; no output schema/allowlist; no spend/rate caps.
