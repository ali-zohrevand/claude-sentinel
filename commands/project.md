---
description: Full-project security audit — all static personas + the complete scanner suite (SAST, secrets incl. git history, dependency CVEs + SBOM, headers, AI/LLM) across the whole repo, consolidated into one severity-ranked report. No network.
argument-hint: [subdir — defaults to the repo root]
---

Whole-project security audit (scope: **$ARGUMENTS** or repo root). Read-only, no network.

## 1. Map the project

Detect the stack (load `sentinel:security-conventions`, reuse `squad:stack-conventions` if present). Enumerate code with `git ls-files`; read `CLAUDE.md` + any `claude-security-guidance.md` policy. Identify the API surface and any AI/LLM usage.

## 2. Fan out (parallel, read-only — one message)

Dispatch with the stack report + project map:
- **whitebox-checker** → full SAST + data-flow; run the complete scanner suite (semgrep, gitleaks/trufflehog over history, dependency audits + SBOM via syft/grype, IaC/container scanners if present) with install-guard skips.
- **owasp-reviewer** → OWASP Top 10 + API Top 10 + LLM Top 10 audit + compliance coverage table.
- **ceh-reviewer** → the five-phase methodology + attack trees for the top risks (static recon).

## 3. Second gate

Optionally run the built-in **`/security-review`** for an independent diff/working-tree pass and fold it in.

## 4. Consolidate

Dispatch **security-advisor** to dedupe across personas + scanners + `/security-review` and produce ONE report: severity-ranked findings (confidence ≥ High; rest in appendix), the OWASP compliance table, and a top-risks summary. `--json` array if requested. Note which scanners ran vs were skipped — never imply coverage you didn't achieve.
