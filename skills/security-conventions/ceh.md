# CEH methodology (defensive framing) + attack trees

The `ceh-reviewer` structures findings into the EC-Council engagement phases. For **code** scopes the phases map to static-recon equivalents; for **live** scopes (post-authorization) they map to real probing via [dast-browser.md](dast-browser.md). Output exactly these five labeled sections, then attack trees.

## The five phases

1. **Reconnaissance** — what an attacker learns before touching the target. Code: exposed config, comments, `.env`/secrets in repo or history, source maps, README/infra hints, dependency fingerprints. Live: headers/banners, tech fingerprint, `robots.txt`/sitemap, TLS/cert, public endpoints. (Passive — pre-auth OK.)
2. **Scanning & Enumeration** — map the attack surface. Code: routes/endpoints, params, auth points, trust boundaries, third-party integrations. Live: crawl, port/service exposure, directory/endpoint enumeration, parameter discovery.
3. **Gaining Access (vectors)** — concrete exploitable vectors: injection, auth bypass, IDOR/BOLA, SSRF, deserialization, file upload, RCE. Each tied to a CWE + OWASP tag and the evidence (`file:line` or request/response).
4. **Maintaining Access / Post-Exploitation risk** — what an attacker reaches *after* one foothold: privilege escalation, lateral movement, persistence, secret/credential access, data exfiltration potential, blast radius of the crown-jewel data.
5. **Covering Tracks / Detection gaps** — logging/monitoring weaknesses that would let an attack go unnoticed: missing audit logs, tamperable logs, no alerting, secrets in logs (A09).

## Attack trees

For each top-severity finding, emit an ASCII attack tree showing the kill chain from initial access to impact:

```
GOAL: exfiltrate user PII
└── AND gain authenticated context
    ├── OR steal session (XSS in /profile bio  → cookie, missing HttpOnly)
    └── OR auth bypass (JWT alg:none accepted at api/auth/verify.php:42)
        └── access other users' records (IDOR: GET /api/v1/users/{id}, no ownership check)
            └── dump PII → no rate limit / no audit log (A09 detection gap)
```

Map each node to its CWE/OWASP tag and evidence so defenders can cut the chain at any node.
