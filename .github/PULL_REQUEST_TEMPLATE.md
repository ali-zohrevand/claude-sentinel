<!-- Thanks for contributing to sentinel! Keep PRs focused on one change. -->

## What & why

<!-- What does this change and what problem does it solve? -->

## Type

- [ ] New check / standards mapping (security-conventions)
- [ ] Scanner integration
- [ ] Persona / agent change
- [ ] Scope / command change
- [ ] Docs
- [ ] Other

## Safety checklist (required)

- [ ] The **authorization gate** is intact; no flag bypasses live/active probing
- [ ] Personas stay **read-only**; no secret values are printed (tokens redacted)
- [ ] New scanner calls use the `command -v … || echo "SKIP — install: …"` guard
- [ ] Does not add evasion/stealth for unauthorized use
- [ ] Model **aliases** used (no pinned model ids)

## General checklist

- [ ] Follows [CONTRIBUTING.md](../CONTRIBUTING.md) house style
- [ ] `claude plugin validate .` passes
- [ ] Smoke-tested (describe below) — live paths only against a host you own
- [ ] Bumped `version` in `.claude-plugin/plugin.json` if the surface changed

## How I tested

<!-- Which command/persona, on what target, and what you observed. -->
