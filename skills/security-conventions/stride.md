# STRIDE threat modeling

Architectural, *find-threats-by-design* analysis — the proactive complement to the find-the-bug personas. Read-only; no network. Works from code + docs + `CLAUDE.md`.

## Step 1 — Build the data-flow diagram (DFD)

Identify and list the system's elements (infer from code/architecture):

- **External entities** — users, browsers, third-party services (e.g. OpenAI, Stripe, Mailgun).
- **Processes** — your app/services/workers/endpoints.
- **Data stores** — DBs, caches, queues, object storage, file system, secrets store.
- **Data flows** — each request/response/message between the above.
- **Trust boundaries** — where data crosses a privilege/trust level (browser→API, API→DB, API→3rd-party, user→admin, internet→internal). Threats concentrate on flows that cross a boundary.

Render the DFD as an indented/ASCII diagram with trust boundaries marked.

## Step 2 — Apply STRIDE to each element

For every element/flow (especially boundary-crossing ones), ask the six questions:

| Category | Question | Property at risk | Typical mitigation |
|---|---|---|---|
| **S**poofing | Can someone impersonate this actor/service? | Authentication | strong auth, MFA, mutual TLS, signed tokens |
| **T**ampering | Can data/code be modified in transit or at rest? | Integrity | TLS, signing/HMAC, access control, checksums |
| **R**epudiation | Can an actor deny an action with no proof? | Non-repudiation | audit logs, signed/timestamped events |
| **I**nformation disclosure | Can data leak to the wrong party? | Confidentiality | encryption at rest/in transit, least privilege, redaction |
| **D**enial of service | Can it be made unavailable/expensive? | Availability | rate limits, quotas, timeouts, autoscaling, cost caps |
| **E**levation of privilege | Can someone gain rights they shouldn't? | Authorization | authz checks, RBAC, sandboxing, deny-by-default |

Element→relevant categories shortcut: external entity → S, R · process → S T R I D E · data store → T I D (and R if it should be auditable) · data flow → T I D.

## Step 3 — Rate & map

For each threat give a risk rating (Critical/High/Medium/Low — optionally a DREAD-style note: Damage, Reproducibility, Exploitability, Affected users, Discoverability) and map it to the matching OWASP category ([owasp.md]) and `file:line` where the design choice lives.

## Output

1. The DFD (elements + trust boundaries).
2. A **threat table**: `element/flow | STRIDE category | concrete threat | risk | mitigation | status (present / partial / missing)`.
3. Top design risks first; concrete next step per missing/partial mitigation. Signal discipline: focus on real, design-level threats — don't pad with generic checklist items that don't apply to this architecture.
