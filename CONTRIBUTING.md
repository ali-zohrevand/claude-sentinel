# Contributing to sentinel

Thanks for helping improve `sentinel` — a comprehensive AI security toolkit for Claude Code. This guide gets you from clone to merged PR.

By participating you agree to the [Code of Conduct](CODE_OF_CONDUCT.md). For reporting a vulnerability **in this plugin**, see [SECURITY.md](SECURITY.md).

> 🛡️ **Ethics first.** sentinel can drive active attacks (the DAST/black-box paths). Contributions must keep the **authorization gate** intact and must never make active scanning easier to run against systems the user hasn't authorized. We don't accept changes that weaken that gate, add stealth/evasion for unauthorized use, or remove the read-only/secret-redaction guarantees.

## Ways to contribute

- **New checks / standards** — extend `skills/security-conventions/` (e.g. a new OWASP/CWE mapping, a stack-specific taint pattern, a new scanner in `scanners.md`).
- **Persona improvements** — sharpen an agent in `agents/` (advisor, whitebox, blackbox-dast, ceh, owasp, threat-modeler).
- **New scope/command** — add or refine a `commands/*.md` orchestrator.
- **Docs** — README / this guide.
- **Bugs & ideas** — open an issue (templates provided).

## Project layout

```text
.claude-plugin/   plugin.json + marketplace.json
agents/           6 persona subagents
commands/         8 scope commands (diff, project, stride, url, site, api, apis, audit)
skills/security-conventions/   SKILL.md (router) + topic files
                  (authorization, dast-browser, site-scan, scanners, owasp, ceh, stride, whitebox, api-testing)
```

## Local development setup

1. Fork and clone your fork.
2. Install from your local checkout:
   ```text
   /plugin marketplace add /path/to/your/claude-sentinel
   /plugin install sentinel@claude-sentinel
   ```
   or run a one-off session: `claude --plugin-dir /path/to/your/claude-sentinel`.
3. Reload Claude Code after edits (Command Palette → "Developer: Reload Window").
4. Optional but recommended: install the scanner CLIs (see the README's "Scanners" section) so you can test scans end-to-end.

## Non-negotiable design rules

- **Authorization gate** (`skills/security-conventions/authorization.md`) is mandatory before any live/active probe; no flag may bypass it. Code-only scopes never touch the network.
- **Read-only personas** — agents report, never edit files. Never print secret *values*; redact tokens in evidence.
- **Signal discipline** — main report = confidence ≥ High with a concrete exploit path; the rest goes in a labeled appendix.
- **Scanner install-guard** — invoke every CLI behind `command -v … || echo "SKIP — install: …"`; never a silent skip, never a hard failure.
- **Model aliases** (`opus`) — never pin a model id.
- **Compose, don't reinvent** — prefer the built-in `/security-review`, `squad:security-owasp`, `postman:security`, and the Playwright MCP over duplicating them.

## House style

- **Agents** — frontmatter (`name`, `description`, `tools`/`disallowedTools`, `model`, `color`) + persona → "When invoked" → method → output contract.
- **Commands** — frontmatter (`description`, `argument-hint`); body scripts the fan-out and enforces the gate where relevant.
- **Skill topics** — focused, cross-linked, with concrete checklists/commands.

## Validate before you push

```bash
claude plugin validate /path/to/your/claude-sentinel   # manifests must pass
claude plugin details sentinel                          # component inventory loads
```

Then smoke-test the change. Safe paths (no network): `/sentinel:diff`, `/sentinel:project`, `/sentinel:stride`. For live paths, test only against a host you own (e.g. a localhost dev server) and confirm the authorization gate fires.

## Submitting a PR

1. Branch from `main`: `git checkout -b feat/<short-name>`.
2. Keep PRs focused.
3. Bump `version` in `.claude-plugin/plugin.json` (semver) if the surface changes; keep `marketplace.json` consistent.
4. Clear commit messages (e.g. `feat: add GraphQL API checks`, `fix: tighten SSRF proof in dast-browser`).
5. Run validation; note what you tested.
6. Open the PR against `ali-zohrevand/claude-sentinel:main` with the PR template.

Reviews check correctness, signal quality, and that the safety rules above are intact. Thank you!
