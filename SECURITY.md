# Security Policy

`sentinel` is a security-review plugin. Two things belong here: how to report a vulnerability **in the plugin itself**, and the rules for **using it responsibly**.

## Reporting a vulnerability in sentinel

If you find a way the plugin could be abused (e.g. a path that bypasses the authorization gate, leaks secrets in a report, or runs an unsafe scanner command), report it privately:

- **Preferred:** open a [private security advisory](https://github.com/ali-zohrevand/claude-sentinel/security/advisories/new).
- **Or** email **ksramn97@gmail.com** with details and reproduction steps.

Please do **not** open a public issue for a security report. We'll acknowledge within a few days and coordinate a fix and disclosure.

In scope: the plugin's agents/commands/skills and its distribution. Out of scope: Claude Code itself (report to Anthropic) and the third-party scanners sentinel invokes (report upstream).

## Responsible use (important)

sentinel can perform **active** security testing — sending real attack traffic to a running target (the `url`, `site`, and live `api`/`apis` scopes). Active scanning of systems you do not own or lack **written authorization** to test is illegal in most jurisdictions (e.g. CFAA, Computer Misuse Act).

- Every live/active scope is gated behind an explicit per-host confirmation; **no flag bypasses it**.
- Only run active scans against hosts you own or are explicitly authorized to test.
- Code-only scopes (`diff`, `project`, `stride`, and static `api`/`apis`) never touch the network and are always safe to run.

You are solely responsible for ensuring you have authorization for any target you scan.
