# sentinel — comprehensive AI security toolkit for Claude Code

`sentinel` is a security-review plugin: **5 expert personas** across **6 scopes**, on the strongest model (opus). It reviews code statically and tests running apps dynamically with an **AI-driven, browser-powered DAST** (Acunetix-style, via the Playwright MCP) — all behind a **hard authorization gate**. It works across every project from one install and composes with the tools you already have rather than reinventing them.

> ⚠️ **Authorization required for live testing.** Active scanning of any host you don't own or lack written permission to test is illegal. Every live/active probe blocks until you type an explicit per-host confirmation. Code-only scopes never touch the network.

## Install

```text
/plugin marketplace add ali-zohrevand/claude-sentinel
/plugin install sentinel@claude-sentinel
```

Local dev: `/plugin marketplace add ~/projects/claude-sentinel` then install, or `claude --plugin-dir ~/projects/claude-sentinel`.

## Commands (scopes)

| Command | Scope | Network |
|---|---|---|
| `/sentinel:diff [base]` | Uncommitted + unpushed changes (pre-push review) | no |
| `/sentinel:project [subdir]` | Whole repo: all static personas + full scanner suite | no |
| `/sentinel:url <url>` | Live URL — AI browser DAST + CLI DAST | **gated** |
| `/sentinel:api <endpoint\|spec#op>` | One API: source review + optional live probe | gated if live |
| `/sentinel:apis <openapi\|routes>` | All APIs: enumerate from source + optional live crawl | gated if live |
| `/sentinel:audit <plain English>` | Natural-language router → picks the right scope | depends |

"Just say it": `/sentinel:audit "check my latest changes"`, `"scan the whole project"`, `"check https://staging.example.com"`, `"audit all my APIs"`.

## Personas (agents, all opus, read-only)

- **security-advisor** — triage + consolidate all findings into one deduped, severity-ranked report.
- **whitebox-checker** — SAST + manual data-flow (injection, RCE, IDOR, auth/JWT, crypto, secrets, dep CVEs, headers, AI prompt-injection). No network.
- **blackbox-dast** — browser-driven AI DAST (Playwright) + nuclei/ZAP/sqlmap/nikto. Gated.
- **ceh-reviewer** — EC-Council 5-phase methodology + attack-tree kill-chains.
- **owasp-reviewer** — OWASP Top 10 + API Top 10 + LLM Top 10 tagging + compliance table.

## Coverage

OWASP Top 10 (2021/2025), OWASP API Top 10 (2023), OWASP LLM/GenAI Top 10 (2025); full CEH methodology; injection / RCE / command execution / SSRF / deserialization; broken access control & IDOR/BOLA; auth, session & JWT; crypto failures; security misconfiguration & headers; secrets (code + git history); dependency CVEs + SBOM; and AI/LLM prompt-injection & excessive agency.

**Scanners** (run when installed, otherwise reported with their install command — never silently skipped): semgrep, gosec, bandit · gitleaks, trufflehog · npm/composer/pip-audit, govulncheck, osv-scanner, syft, grype · nuclei, OWASP ZAP, sqlmap, nikto, testssl.sh · trivy, checkov, tfsec, hadolint.

## Principles

- **Strongest model**, via the `opus` alias (never a pinned id, so it never ages). Override every subagent with `CLAUDE_CODE_SUBAGENT_MODEL=opus` if desired.
- **Deterministic commands** are the contract; `/sentinel:audit` is the natural-language front door.
- **High signal**: only findings with a concrete exploit path + confidence ≥ High in the main report; rest in a labeled appendix.
- **Read-only & secret-safe**: never edits files; never prints secret values; redacts tokens in API probing.
- **`--json`** output on any command for CI/ticketing.

## Composes with (doesn't reinvent)

- Built-in **`/security-review`** — deterministic second gate on diff/project.
- **`security-guidance`** plugin — passive, always-on layer; sentinel is the on-demand deep layer and avoids re-flagging it.
- **`squad`** — `sentinel` prefers `squad:security-owasp` and reuses `squad:stack-conventions` when installed; fully self-contained without it.
- **`postman:security`** — OWASP API audit for the api/apis scopes.
- **Playwright MCP** — the browser engine for the AI DAST.

## Layout

```
.claude-plugin/   plugin.json + marketplace.json
agents/           5 persona subagents
commands/         6 scope commands
skills/security-conventions/   SKILL.md + authorization, dast-browser, scanners,
                               owasp, ceh, whitebox, api-testing
```
