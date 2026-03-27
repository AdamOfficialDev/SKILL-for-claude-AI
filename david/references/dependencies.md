# DAVID — Dependency Auditor Reference

Full pattern library for Scanner H.

---

## Known Vulnerable Package Patterns

### npm / Node.js

| Package | Issue | Action |
|---------|-------|--------|
| `node-serialize` | RCE via deserialization | Remove — use `safe-json-stringify` |
| `lodash < 4.17.21` | Prototype pollution (CVE-2021-23337) | Upgrade to ≥4.17.21 |
| `jsonwebtoken < 9.0.0` | Algorithm confusion attacks | Upgrade to ≥9.0.0 |
| `axios < 1.6.0` | SSRF in some configurations | Upgrade |
| `node-fetch < 3.3.2` | SSRF | Upgrade |
| `express < 4.19.2` | Open redirect | Upgrade |
| `qs < 6.10.3` | Prototype pollution | Upgrade |
| `minimist < 1.2.6` | Prototype pollution | Upgrade |
| `semver < 7.5.2` | ReDoS | Upgrade |
| `tough-cookie < 4.1.3` | Prototype pollution | Upgrade |
| `vm2` (any version) | Sandbox escape — abandoned | Replace with `isolated-vm` |

### Python

| Package | Issue | Action |
|---------|-------|--------|
| `PyYAML < 6.0` without SafeLoader | RCE via yaml.load() | Use `yaml.safe_load()` always |
| `Pillow < 10.0.1` | Multiple CVEs | Upgrade |
| `cryptography < 41.0.3` | Timing attacks | Upgrade |
| `requests < 2.32.0` | Various | Upgrade |
| `urllib3 < 2.2.2` | Header injection | Upgrade |
| `werkzeug < 3.0.3` | Path traversal | Upgrade |
| `setuptools < 70.0.0` | Command injection | Upgrade |
| `pickle` module | RCE when used with untrusted data | Never deserialize untrusted pickle data |

### Java

| Package | Issue | Action |
|---------|-------|--------|
| `log4j 2.x < 2.17.1` | Log4Shell RCE (CVE-2021-44228) | Upgrade immediately |
| `Spring Framework < 5.3.18` | Spring4Shell | Upgrade |
| `jackson-databind` (old) | Deserialization RCE | Use latest + disable default typing |
| `commons-text < 1.10.0` | Text4Shell RCE | Upgrade |

---

## License Audit Rules

### Licenses INCOMPATIBLE with commercial/proprietary projects

| License | Restriction | Risk |
|---------|-------------|------|
| **GPL v2/v3** | All derivative work must be GPL-licensed | 🔴 HIGH — must open-source your product |
| **AGPL v3** | Network use counts as distribution — must open-source | 🔴 CRITICAL — stricter than GPL |
| **EUPL** | Copyleft with European jurisdiction | 🟠 HIGH |
| **CC BY-SA** | Share-alike — derivatives must use same license | 🟠 HIGH |
| **SSPL** | MongoDB license — network service counts as distribution | 🔴 HIGH |

### Licenses COMPATIBLE (permissive)

| License | Commercial Use | Modification | Distribution |
|---------|---------------|-------------|-------------|
| **MIT** | ✅ | ✅ | ✅ (keep attribution) |
| **Apache 2.0** | ✅ | ✅ | ✅ (keep notices, patent grant) |
| **BSD 2/3-clause** | ✅ | ✅ | ✅ (keep attribution) |
| **ISC** | ✅ | ✅ | ✅ |
| **CC0** | ✅ | ✅ | ✅ (no attribution required) |

### LGPL — Special Case
LGPL allows use in proprietary software **only if** the LGPL component is dynamically linked (separate shared library). Static linking = violation.

---

## Package Health Signals

### Abandonment Risk
Flag packages where:
- Last publish date > 2 years ago AND no explicit "stable/complete" statement
- GitHub repository archived
- Open issues/PRs with no maintainer response for >6 months
- No README, no tests, no documentation

### Quality Red Flags
- Package with <100 weekly downloads on npm (potential malicious package)
- Package name that closely resembles a popular package (typosquatting: `lodahs`, `requets`)
- Package published very recently (<1 month) with no stars/forks
- Package with no license field

### Version Pinning Risks
```json
// ❌ Wildcard ranges allow silent breaking changes + vulnerability introduction
"dependencies": {
    "express": "^4.18.0",    // Any 4.x — could get 4.19.0 with new CVE
    "lodash": "~4.17.0"      // Any 4.17.x patch
}

// ✅ For production: exact versions in lockfile, managed updates via Dependabot/Renovate
"dependencies": {
    "express": "4.18.2"      // Exact — lock file provides real pinning
}
```
