---
description: STRIDE architectural threat model — build a data-flow diagram with trust boundaries and enumerate threats by design (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege), each mapped to a mitigation. Design-level, read-only, no network.
argument-hint: [subsystem or feature — defaults to the whole app]
---

STRIDE threat model (scope: **$ARGUMENTS** or the whole application). Read-only, no network — this analyzes the design, not running code.

## 1. Understand the system

Load `sentinel:security-conventions` → [stride.md]. Read `CLAUDE.md`, architecture/docs, and the code structure. Identify external entities, processes, data stores, data flows, third-party integrations (DB, cache, queue, storage, OpenAI/Stripe/Mailgun, auth), and the **trust boundaries** between them.

## 2. Model the threats

Dispatch **threat-modeler** to:
- Build the **data-flow diagram** with trust boundaries marked.
- Apply **STRIDE** to each element/flow (focus on boundary-crossing flows), rate each threat, map it to its OWASP category and the `file:line` where the design decision lives, and mark the mitigation present / partial / missing.

## 3. Prioritize

Dispatch **security-advisor** to risk-rank the threat table and surface the top design risks with the single best next step each. (Optionally cross-reference `/sentinel:project` findings to see which design threats are already realized in code.)

## 4. Output

The DFD + the STRIDE threat table (`element | category | threat | risk | mitigation | status`) + top design risks first. `--json` if requested. Frame missing mitigations as design gaps, not confirmed exploits.
