# sentinel — comprehensive AI security toolkit for Claude Code

`sentinel` is a security-review plugin: **6 expert personas** across **8 scopes**, on the strongest model (opus). It reviews code statically, models threats architecturally (STRIDE), and tests running apps dynamically with an **AI-driven, browser-powered DAST** (Acunetix-style whole-site crawl, via the Playwright MCP) — all behind a **hard authorization gate**. One install serves every project, and it composes with the tools you already have rather than reinventing them.

> ⚠️ **Authorization required for live testing.** Active scanning of any host you don't own or lack written permission to test is illegal (CFAA / Computer Misuse Act). Every live/active probe blocks until you type an explicit per-host confirmation. Code-only scopes never touch the network.

## Quick install

```text
/plugin marketplace add ali-zohrevand/claude-sentinel
/plugin install sentinel@claude-sentinel
```

Then reload Claude Code (Command Palette → "Developer: Reload Window"). Optionally install the [scanners](#scanners) for full coverage. Full details in [Install](#install).

> ⏳ **Coming to the Claude community directory.** Once approved, you'll also be able to install it directly: `/plugin marketplace add anthropics/claude-plugins-community` then `/plugin install sentinel@claude-community`.

---

## Table of contents

- [Install](#install)
- [Commands](#commands)
- [Examples](#examples)
- [The personas](#the-personas)
- [Coverage](#coverage)
- [Scanners](#scanners) — what to install
- [The authorization gate](#the-authorization-gate)
- [Whole-site DAST](#whole-site-dast-sentinelsite)
- [STRIDE threat model](#stride-threat-model-sentinelstride)
- [Principles](#principles)
- [Composes with](#composes-with)
- [Layout](#layout)
- [Updating / developing](#updating--developing)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Install

```text
/plugin marketplace add ali-zohrevand/claude-sentinel
/plugin install sentinel@claude-sentinel
```

Then **reload Claude Code** so the commands/agents load (Command Palette → "Developer: Reload Window", or relaunch the CLI).

Local development:

```text
/plugin marketplace add ~/projects/claude-sentinel
/plugin install sentinel@claude-sentinel
# or one-off:  claude --plugin-dir ~/projects/claude-sentinel
```

Install the scanner CLIs too (see [Scanners](#scanners)) so scans run fully — sentinel still works without them, reporting each missing tool with its install command.

---

## Commands

| Command | Scope | Network |
|---|---|---|
| `/sentinel:diff [base]` | Uncommitted + unpushed changes (pre-push review) | no |
| `/sentinel:project [subdir]` | Whole repo: all static personas + full scanner suite | no |
| `/sentinel:stride [subsystem]` | STRIDE architectural threat model (DFD + trust boundaries) | no |
| `/sentinel:url <url>` | Live URL — AI browser DAST + CLI DAST | **gated** |
| `/sentinel:site <url>` | **Whole website** — deep-crawl + active-test every page (Acunetix-style) | **gated** |
| `/sentinel:api <endpoint\|spec#op>` | One API: source review + optional live probe | gated if live |
| `/sentinel:apis <openapi\|routes>` | All APIs: enumerate from source + optional live crawl | gated if live |
| `/sentinel:audit <plain English>` | Natural-language router → picks the right scope | depends |

Add `--json` to any command for machine-readable output (CI / ticketing).

---

## Examples

```text
/sentinel:diff                                   # review what you're about to push
/sentinel:project                                # full static audit of the repo
/sentinel:stride                                 # threat-model the architecture
/sentinel:audit check my latest changes          # NL router → diff
/sentinel:url http://localhost:8000              # single-target browser DAST (gated)
/sentinel:site http://localhost:8000 --max-pages 100   # whole-site DAST (gated)
/sentinel:api api/v1/cartoons.php                # one endpoint, source + optional probe
/sentinel:apis openapi.yaml                      # every endpoint, OWASP API Top 10
```

---

## The personas

Six subagents (in `agents/`), all `model: opus`, all **read-only** (they report, never edit; secret values are never printed). Only the live-probing personas get network/browser access.

| Agent | Tools | Responsibility |
|---|---|---|
| `security-advisor` | Read, Grep, Glob, Bash, WebFetch, Skill | Triage brain: scope, detect stack, **consolidate + dedupe** all findings into one severity-ranked report |
| `whitebox-checker` | Read, Grep, Glob, Bash, Skill | SAST + manual data-flow: injection, RCE, IDOR, auth/JWT, crypto, secrets, dep CVEs, headers, AI prompt-injection. No network |
| `blackbox-dast` | inherits all except Edit/Write (incl. Playwright MCP) | Browser-driven AI DAST + nuclei/ZAP/sqlmap/nikto. Powers `url` and `site`. Gated |
| `ceh-reviewer` | Read, Grep, Glob, Bash, WebFetch, Skill | EC-Council 5-phase methodology + attack-tree kill-chains |
| `owasp-reviewer` | Read, Grep, Glob, Bash, Skill | OWASP Top 10 + API Top 10 + LLM Top 10 tagging + compliance coverage table |
| `threat-modeler` | Read, Grep, Glob, Bash, WebFetch, Skill | STRIDE: data-flow diagram + trust boundaries + threat table. Design-level, no network |

Models use the `opus` alias (never a pinned id). Override every subagent with `CLAUDE_CODE_SUBAGENT_MODEL=opus` if desired.

---

## Coverage

OWASP Top 10 (2021/2025), OWASP API Top 10 (2023), OWASP LLM/GenAI Top 10 (2025); full CEH methodology; **STRIDE** architectural threat modeling; injection / RCE / command execution / SSRF / deserialization; broken access control & IDOR/BOLA; auth, session & JWT; crypto failures; security misconfiguration & headers; secrets (code + git history); dependency CVEs + SBOM; and AI/LLM prompt-injection & excessive agency.

---

## Scanners

Run when installed; otherwise reported with their install command — **never silently skipped**. macOS install commands:

```bash
# SAST
pipx install semgrep
go install github.com/securego/gosec/v2/cmd/gosec@latest
pipx install bandit
# Secrets
brew install gitleaks trufflehog
# Dependencies / SBOM
brew install osv-scanner syft grype
go install golang.org/x/vuln/cmd/govulncheck@latest
pipx install pip-audit
# DAST (live)
brew install nuclei nikto testssl
pipx install sqlmap
# IaC / containers
brew install trivy tfsec hadolint
pipx install checkov
```

`npm audit` / `composer audit` ship with npm / composer. The Playwright MCP (the browser engine) is a separate Claude Code plugin. OWASP ZAP and CodeQL are optional, heavier installs.

---

## The authorization gate

Before the first active probe of any live target, sentinel prints:

```text
⛔ AUTHORIZATION REQUIRED
About to send ACTIVE security probes to: <host>
Type exactly:  I confirm I have authorization to test <host>
(or Ctrl-C to abort)
```

No flag bypasses it. Until confirmed, only passive checks run (headers, TLS, public content, static spec review). Authorized probing stays scope-locked to the host, uses non-destructive payloads, and is rate-limited. You own the legal/ethical responsibility per run. Run live scans against your own hosts (e.g. a localhost dev server) or targets you have written permission to test.

---

## Whole-site DAST (`/sentinel:site`)

The Acunetix-with-AI command. After the gate: deep-crawl the whole site (sitemap/robots seed → BFS over same-host links to `--max-pages`/`--depth` caps, authenticated crawl with creds, captures forms + XHR endpoints, dedupes templated URLs), then active-test every unique input with the full attack catalog via the Playwright browser + `nuclei` breadth pass. Ends with a site-wide report **and a coverage map** (pages crawled vs skipped, inputs tested vs skipped). Falls back to `curl`/CLI DAST if the Playwright MCP isn't connected.

---

## STRIDE threat model (`/sentinel:stride`)

Architectural, find-threats-by-design analysis (no network). The `threat-modeler` builds a data-flow diagram with **trust boundaries**, then walks each element through **S**poofing → auth, **T**ampering → integrity, **R**epudiation → audit, **I**nformation disclosure → confidentiality, **D**enial of service → availability, **E**levation of privilege → authorization — emitting a `element | category | threat | risk | mitigation | status` table. The proactive complement to the find-the-bug personas.

---

## Principles

- **Strongest model**, via the `opus` alias (never pinned, so it never ages).
- **Deterministic commands** are the contract; `/sentinel:audit` is the natural-language front door.
- **High signal**: only findings with a concrete exploit path + confidence ≥ High in the main report; the rest in a labeled appendix.
- **Read-only & secret-safe**: never edits files; never prints secret values; redacts tokens in API probing.
- **No silent gaps**: a missing scanner is reported with its install command; coverage is always stated.

---

## Composes with

- Built-in **`/security-review`** — deterministic second gate on diff/project.
- **security-guidance** plugin — passive, always-on layer; sentinel is the on-demand deep layer and avoids re-flagging it.
- **squad** — prefers `squad:security-owasp` and reuses `squad:stack-conventions` when installed; fully self-contained without it.
- **postman:security** — OWASP API audit for the api/apis scopes.
- **Playwright MCP** — the browser engine for the AI DAST.

---

## Layout

```text
.claude-plugin/   plugin.json + marketplace.json
agents/           6 persona subagents
commands/         8 scope commands
skills/security-conventions/   SKILL.md + authorization, dast-browser, site-scan,
                               scanners, owasp, ceh, stride, whitebox, api-testing
```

---

## Updating / developing

```text
# edit files in ~/projects/claude-sentinel, then:
git commit -am "…" && git push
/plugin update sentinel        # in Claude Code; restart to apply
```

Validate before pushing: `claude plugin validate ~/projects/claude-sentinel`. Inspect components + token cost: `claude plugin details sentinel`.

> If `/plugin update` reports "Plugin not found" after a `marketplace update`, do a clean cycle: `claude plugin uninstall sentinel && claude plugin install sentinel@claude-sentinel`.

---

## Troubleshooting

- **Commands don't appear:** reload the window — a freshly installed/updated plugin loads at the next session start.
- **Browser DAST does nothing:** the Playwright MCP isn't connected; sentinel falls back to curl/CLI DAST. Install/enable the Playwright plugin for the full browser engine.
- **A scanner is "skipped":** install it (see [Scanners](#scanners)); sentinel never fails the run for a missing tool, it just reports it.
- **The gate won't let me probe:** that's by design — type the exact confirmation phrase naming the host, and only for hosts you're authorized to test.

---

## Contributing

Contributions welcome — new checks, scanner integrations, sharper personas, docs. See **[CONTRIBUTING.md](CONTRIBUTING.md)** for setup, the non-negotiable safety rules, and the PR flow, plus the **[Code of Conduct](CODE_OF_CONDUCT.md)**. Report a vulnerability in the plugin privately via **[SECURITY.md](SECURITY.md)** — never as a public issue. Bugs and ideas → open an issue (templates provided).

## License

[MIT](LICENSE) © ali zohrevand. Use only against systems you own or are authorized to test.
