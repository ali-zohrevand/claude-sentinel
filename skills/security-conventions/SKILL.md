---
name: security-conventions
description: Security review conventions, standards checklists, scanner commands, and the live-probe authorization gate for the sentinel personas. Use when performing any security review, vulnerability assessment, OWASP/CEH audit, secrets/dependency scan, or browser-driven DAST.
---

# Security conventions

The shared brain for the sentinel security personas. Don't review from memory — detect the target and stack, then read the topic file(s) you need.

## 1. Detect the target type

| Target | Signal | Network? |
|---|---|---|
| **diff** | reviewing `git diff` / unpushed commits | no |
| **project** | whole repo / a subtree | no |
| **url** | an `http(s)://` target | **yes — gated** |
| **api** | one endpoint URL or one OpenAPI operation | yes if live — gated |
| **apis** | a whole OpenAPI doc or the route table | yes if live — gated |

Any live/active probing requires the authorization gate — read [authorization.md](authorization.md) and enforce it **before the first byte leaves the machine.** Code-only scopes (diff/project, and the static half of api/apis) never touch the network.

## 2. Detect the stack

Reuse the signals from `squad:stack-conventions` when that skill is installed (load it). Otherwise: `package.json` (TS/Vue/Nest/Node), `go.mod` (Go), `composer.json` (PHP), `requirements.txt`/`pyproject.toml` (Python), plus framework route markers. The stack picks which scanners and which language taint patterns apply.

## 3. Signal discipline (applies to every persona)

- Report only findings with a **concrete exploit path** and **confidence ≥ High** in the main report. Park lower-confidence items in a clearly-labeled "Informational / low-confidence" appendix, and only if useful.
- Each finding: **severity** (Critical/High/Medium/Low) + **CVSS v3.1** where it makes sense, **`file:line`** or the **HTTP request/response evidence**, the **attack path**, a **concrete fix**, and the **standard tag** (OWASP/API/LLM/CWE).
- **Read-only always.** Never edit files. Never print secret *values* — report locations and redact tokens.
- **Dedupe** across personas, the built-in `/security-review`, and the passive `security-guidance` plugin so the user sees one ranked list.
- **Never silently skip a gate:** if a scanner isn't installed, say so and print the install command ([scanners.md](scanners.md)).

## 4. Topic files

- [authorization.md](authorization.md) — the hard gate for any live/active probe (read before url/api/apis).
- [dast-browser.md](dast-browser.md) — Acunetix-style browser-driven AI DAST via the Playwright MCP + the active-test catalog.
- [scanners.md](scanners.md) — every CLI scanner: purpose, invocation, install guard.
- [owasp.md](owasp.md) — OWASP Top 10 (2021/2025) + API Top 10 (2023) + LLM Top 10 (2025) checklists.
- [ceh.md](ceh.md) — EC-Council methodology phases + attack-tree format.
- [whitebox.md](whitebox.md) — source-review / SAST patterns per stack (injection, RCE, authz/IDOR, secrets, headers, JWT, AI prompt-injection, misconfig).
- [api-testing.md](api-testing.md) — OpenAPI parsing, per-operation authz, token redaction, `postman:security` handoff.

## 5. Compose, don't reinvent

- Built-in **`/security-review`** — run as a deterministic second gate on diff/project.
- **`security-guidance`** plugin — passive, always-on; don't re-flag what it already surfaced this session; honor a repo's `claude-security-guidance.md` policy file if present.
- **`squad:security-owasp`** — prefer it for the OWASP lens when the `squad` plugin is installed; the local `owasp-reviewer` is the self-contained fallback.
- **`postman:security`** — OWASP API Top 10 audit when a Postman collection / OpenAPI spec exists.
