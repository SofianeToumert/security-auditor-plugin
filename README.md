# Security Auditor Plugin for Claude Code

A comprehensive security and compliance auditor plugin for Claude Code. Scans codebases for HIPAA, GDPR, PCI DSS, PIPEDA, CCPA, and SOC 2 violations including PII in logs, hardcoded secrets, and input validation issues.

## What's Included

### 1 Agent

- **`security-auditor`** - Orchestrator agent that runs all security skills and generates a comprehensive report

> `/security-review` scans the **full codebase**. `/security-review-diff` scans **only changed files** (git diff) for fast pre-PR checks.

### 6 Skills

| Skill | Command | Description |
| --- | --- | --- |
| Security Review | `/security-review` | Full codebase 8-item security checklist (input validation, PII, secrets, HTTPS, headers, tokens, encryption, errors) |
| Security Review Diff | `/security-review-diff` | Same 8-item checklist, but only on changed files (git diff) — ideal for PRs |
| Secrets Check | `/secrets-check` | Gitleaks + custom patterns for 14+ secret types |
| PII Scanner | `/pii-scanner` | Detects PII in logs, errors, and console output |
| Compliance Check | `/compliance-check` | Full HIPAA/GDPR/PCI DSS/PIPEDA/CCPA/SOC 2 validation |
| Compliance Report | `/generate-compliance-report` | Generates stakeholder-ready compliance reports |

### 5 Compliance Reference Documents

Bundled markdown docs covering all major compliance frameworks:

| Document | Contents |
| --- | --- |
| `docs/compliance/usa-hipaa-ccpa-soc2.md` | HIPAA, CCPA, SOC 2, PCI DSS, NIST, FedRAMP |
| `docs/compliance/canada-pipeda.md` | PIPEDA, Quebec Law 25, provincial laws |
| `docs/compliance/europe-gdpr.md` | GDPR, NIS2, DORA, EU AI Act, ISO 27001 |
| `docs/compliance/logging-data-protection.md` | Logging requirements across all standards |
| `docs/compliance/gap-analysis-roadmap.md` | Common gaps and remediation roadmap |

### Additional Resources

- `config/.gitleaks.toml` - Default gitleaks configuration with 18 secret patterns
- `docs/SECURITY_GUIDELINES.md` - Universal security implementation guidelines

## Installation

### As a marketplace

```bash
claude plugin marketplace add ~/path/to/security-auditor-plugin
claude plugin install security-auditor@security-auditor-marketplace
```

### From Git URL

```bash
claude plugin marketplace add <git-url>
claude plugin install security-auditor@security-auditor-marketplace
```

## Usage

### Run the full security auditor

```
/security-auditor
```

This triggers the orchestrator agent which runs all skills and produces a comprehensive report.

### Run individual skills

```bash
# Full codebase security checklist (all files)
/security-review [path]

# Security checklist on changed files only (git diff)
/security-review-diff [base-branch]

# Scan for hardcoded secrets
/secrets-check [path]

# Find PII in logs
/pii-scanner [path]

# Validate compliance frameworks
/compliance-check [path]

# Generate compliance report for stakeholders
/generate-compliance-report [--format=pdf|json]
```

### Examples

```bash
# Check entire codebase
/security-review

# Check specific directory
/pii-scanner src/features/auth/

# Check after implementing payment features
/compliance-check src/payments/

# Monthly audit report
/generate-compliance-report
```

## What It Detects

### Critical (Blocks Merge)

- PII in logs (emails, phones, SSNs, health data)
- Hardcoded secrets (API keys, tokens, passwords)
- Missing input validation (raw `req.body` usage)
- localStorage for auth tokens (XSS vulnerability)
- Credit card data storage (PCI DSS violation)
- PHI in error messages (HIPAA violation)

### High Priority

- Weak passwords (< 12 characters, PCI DSS 4.0)
- Missing security headers (CSP, HSTS)
- HTTP URLs in production
- Technical details in error messages
- Missing session timeout (PCI DSS: 15 min)

### Compliance Frameworks

| Framework | Focus | Max Penalty |
| --- | --- | --- |
| HIPAA | Healthcare PHI | $50,000/violation |
| GDPR | EU personal data | 4% annual revenue |
| PCI DSS 4.0 | Payment cards | $500,000/month |
| PIPEDA | Canadian data | CA$100,000 |
| CCPA | California data | $7,500/violation |
| SOC 2 | Security controls | Audit failure |

## CI/CD Integration

Add to your GitHub Actions workflow:

```yaml
name: Security Audit

on: [pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Gitleaks
        run: |
          docker run --rm -v $PWD:/path zricethezav/gitleaks:latest \
            detect --source="/path" -v -c /path/.gitleaks.toml

      - name: Claude Security Review
        run: |
          claude /security-auditor
```

## Customization

### Project-level gitleaks config

If your project has its own `.gitleaks.toml`, it takes precedence over the plugin's bundled default.

### Extending detection patterns

The skills use regex-based detection. You can extend patterns by creating project-specific security rules in your CLAUDE.md.

## License

MIT
