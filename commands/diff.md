---
description: Security-review the latest changes — uncommitted (staged+unstaged) plus unpushed commits. White-box + OWASP + secrets + dependency scan on the changed code, consolidated into one severity-ranked report. No network.
argument-hint: [base ref — defaults to the upstream/origin HEAD]
---

Security review of the **latest changes** (scope: **$ARGUMENTS** or auto-detected). Read-only, no network — this is a code review of what you're about to push.

## 1. Build the diff scope (deterministic)

Run, read-only:
- Uncommitted: `git diff HEAD` (staged + unstaged) and `git status --porcelain`.
- Unpushed commits: `git diff @{u}..HEAD` if an upstream exists, else `git diff $ARGUMENTS..HEAD`, else fall back to `git diff origin/main...HEAD`.
- List the changed files; the review is scoped to these (and their immediate context), not the whole repo.

## 2. Fan out (parallel, read-only — one message)

Dispatch with the diff + changed-file list + detected stack:
- **whitebox-checker** → SAST + data-flow on the changed code; run secrets (gitleaks/trufflehog) and dependency audits if deps changed.
- **owasp-reviewer** → tag findings to OWASP/API/LLM categories.

## 3. Second gate

Also run the built-in **`/security-review`** (its native scope is the branch diff) and fold its findings in.

## 4. Consolidate

Dispatch **security-advisor** with all of the above to dedupe and produce ONE severity-ranked report (confidence ≥ High in the main list; rest in an appendix). End with **GO / NO-GO** for pushing. If `--json` was requested, also emit the JSON array.
