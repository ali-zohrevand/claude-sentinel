# Scanners — invocation + install guards

Run the scanners that fit the stack/scope. **Always guard** so a missing tool is reported with its install command, never a silent gap and never a hard failure:

```bash
command -v <tool> >/dev/null 2>&1 && <tool> <args> || echo "SKIP <tool> — install: <install-cmd>"
```

Summarize scanner output into the report (locations + severity); never paste secret values. Prefer the repo's own configured scripts/configs (`semgrep.yml`, `.gitleaks.toml`, `phpstan.neon`) when present.

## SAST (static code analysis)
| Tool | Invocation | Install |
|---|---|---|
| semgrep | `semgrep --config auto --error <path>` (add `--sarif` for machine output) | `pipx install semgrep` |
| gosec (Go) | `gosec ./...` | `go install github.com/securego/gosec/v2/cmd/gosec@latest` |
| bandit (Python) | `bandit -r <path>` | `pipx install bandit` |
| CodeQL (optional, deep) | `codeql database analyze …` | CodeQL CLI |
| PHPStan (PHP, security-adjacent) | `phpstan analyse --level=max` | `composer require --dev phpstan/phpstan` |

## Secrets (code + git history)
| Tool | Invocation | Install |
|---|---|---|
| gitleaks | `gitleaks detect --no-banner --redact` (history) / `gitleaks dir <path>` | `brew install gitleaks` |
| trufflehog | `trufflehog git file://. --only-verified` / `trufflehog filesystem <path>` | `brew install trufflehog` |

## Dependencies / CVEs / SBOM
| Tool | Invocation | Install |
|---|---|---|
| npm | `npm audit --omit=dev` | bundled with npm |
| pnpm/yarn | `pnpm audit` / `yarn npm audit` | bundled |
| composer (PHP) | `composer audit` | bundled with composer |
| pip-audit (Python) | `pip-audit` | `pipx install pip-audit` |
| govulncheck (Go) | `govulncheck ./...` | `go install golang.org/x/vuln/cmd/govulncheck@latest` |
| osv-scanner (any) | `osv-scanner scan -r .` | `brew install osv-scanner` |
| syft (SBOM, CycloneDX) | `syft dir:. -o cyclonedx-json` | `brew install syft` |
| grype (scan SBOM/image) | `grype dir:.` / `grype <image>` | `brew install grype` |

## DAST / live (only after the authorization gate)
| Tool | Invocation | Install |
|---|---|---|
| Playwright MCP | primary engine — see [dast-browser.md](dast-browser.md) | already a Claude Code plugin |
| nuclei | `nuclei -u <target> -severity medium,high,critical` | `brew install nuclei` |
| OWASP ZAP | `zap.sh -cmd -quickurl <target> -quickout report.html` | ZAP CLI |
| sqlmap | `sqlmap -u <target> --batch --level 1` (confirm SQLi only) | `pipx install sqlmap` |
| nikto | `nikto -h <target>` | `brew install nikto` |
| testssl.sh | `testssl.sh <host>` (passive TLS — pre-auth OK) | `brew install testssl` |

## IaC / containers (best-effort when present)
| Tool | Invocation | Install |
|---|---|---|
| trivy (fs/image/IaC) | `trivy fs .` / `trivy image <img>` / `trivy config .` | `brew install trivy` |
| checkov (IaC) | `checkov -d .` | `pipx install checkov` |
| tfsec (Terraform) | `tfsec .` | `brew install tfsec` |
| hadolint (Dockerfile) | `hadolint Dockerfile` | `brew install hadolint` |
