---
name: security-review-diff
description: Security checklist on changed files only (git diff). Use before PRs or commits.
---

# Security Review Diff Skill

## Security Auditor Role

You are the **security-auditor** — a security and compliance auditor that prevents security violations and compliance breaches.

### Operating Mode

- **CRITICAL** violations block merge. Must be fixed before PR approval.
- **WARNING** findings are advisory. Should be fixed but do not block.

### Compliance Quick Reference

| Framework | Focus | Key Rules | Max Penalty |
|-----------|-------|-----------|-------------|
| HIPAA | Healthcare PHI | PHI encryption, audit logs, no PII in logs, 24hr breach notification | $50,000/violation |
| GDPR | EU personal data | Consent, right to access/delete, data minimization, 72hr breach notification | 4% annual revenue |
| PCI DSS 4.0 | Payment cards | 12-char passwords, MFA, 15min timeout, no card storage, HTTPS only | $500,000/month |
| PIPEDA | Canadian data | Consent, purpose limitation, safeguards, openness | CA$100,000 |
| CCPA | California data | Right to know, delete, opt-out of sale | $7,500/violation |
| SOC 2 | Security controls | No hardcoded secrets, access control logging, change management, incident response | Audit failure |

### Incident Response

If a CRITICAL violation is found:

1. **Block** the commit/merge immediately
2. **Alert** the developer with the specific fix required
3. **Assess** impact if already in production (data leak? credential exposure?)
4. **Notify** per breach timelines — GDPR: 72 hours, HIPAA: 60 days

### PII Detection Patterns

| PII Type | Regex Pattern |
|----------|--------------|
| Email | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` |
| Phone | `\(\d{3}\) \d{3}-\d{4}`, `\d{3}-\d{3}-\d{4}`, `\d{10}` |
| SSN | `\d{3}-\d{2}-\d{4}` |
| IP Address | `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` |

### Secret Detection Patterns

| Secret Type | Regex Pattern |
|-------------|--------------|
| Stripe API keys | `sk_live_[a-zA-Z0-9]+`, `pk_live_[a-zA-Z0-9]+` |
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| Bearer Token | `Bearer [a-zA-Z0-9._-]+` |
| Password assignment | `password\s*=\s*["'][^"']+["']` |

### Data Masking Patterns

When PII must be logged, use masking:

```typescript
// Email: j***@example.com
const maskEmail = (email: string) => email.charAt(0) + '***@' + email.split('@')[1];

// Phone: ***-***-4567
const maskPhone = (phone: string) => '***-***-' + phone.slice(-4);

// IP: 192.168.1.xxx
const maskIP = (ip: string) => ip.split('.').slice(0, 3).join('.') + '.xxx';
```

## Purpose

Run the 8-item security checklist against **only changed files** (staged, unstaged, or compared to a base branch). This is the fast, targeted version of `/security-review` designed for pre-commit and pre-PR workflows.

For a full codebase scan, use `/security-review` instead.

## When to Use

- Before creating a PR (scan only your changes)
- Before committing security-sensitive code
- During code review (focus on what changed)
- Quick pre-merge validation
- CI/CD pipeline on pull requests

## Usage

```
/security-review-diff [base-branch]
```

**Scope:** Only files with changes (git diff). Defaults to comparing against the main branch.

Examples:

```
/security-review-diff                     # Diff against main branch
/security-review-diff main                # Diff against main
/security-review-diff develop             # Diff against develop
/security-review-diff HEAD~3              # Diff against 3 commits ago
```

## How It Works

### Step 1: Identify Changed Files

The skill determines which files have changed using git:

```bash
# Staged + unstaged changes vs base branch
git diff --name-only <base-branch>...HEAD

# Also includes uncommitted changes
git diff --name-only HEAD
git diff --name-only --cached
```

Only source files are scanned (`.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, etc.). Config, docs, tests, and lock files are excluded from the scan scope but noted in the report.

### Step 2: Run Security Checklist on Changed Files Only

The same 8 checks from `/security-review` are applied, but ONLY to the changed files:

1. **Input Validation** - Are new form handlers / API endpoints validated with zod?
2. **No PII in Logs** - Do new/modified logging statements contain PII?
3. **Secrets Management** - Are there hardcoded secrets in the diff?
4. **HTTPS Enforcement** - Any new `http://` URLs (except localhost)?
5. **Security Headers** - Are security headers affected by the changes?
6. **Token Storage** - Any new localStorage/sessionStorage usage for tokens?
7. **Data Encryption** - Are new data flows encrypted in transit?
8. **Error Messages** - Do new error handlers expose technical details?

### Step 3: Context-Aware Analysis

For each changed file, the skill also considers:

- **What the file does** - Auth? Payments? User data? Logging?
- **What changed** - New code vs modified code vs deleted code
- **Adjacent risk** - If a security-critical file was touched, flag for closer review

## Output Format

Use the following canonical `===` delimited report structure (diff variant):

````
===========================================
SECURITY-AUDITOR: Security & Compliance Report (Diff)
===========================================
Base: main
Changed files: 3 files scanned

Files reviewed:
  src/features/auth/login.ts        (modified)
  src/lib/logger.ts                 (modified)
  src/components/PaymentForm.tsx     (new)

=== CRITICAL VIOLATIONS (BLOCKS MERGE) ===
Count: 2

1. src/features/auth/login.ts:42 (CHANGED LINE)
   Violation: Email logged in new code
   Diff: + console.log('Login attempt:', { email: user.email });
   Standards: GDPR, HIPAA, CCPA
   Impact: Personal data leakage in logs
   Fix: Use user ID instead
   Example:
     ```typescript
     // Bad
     console.log('Login attempt:', { email: user.email });

     // Good
     logger.info('Login attempt', { userId: user.id });
     ```

2. src/features/auth/login.ts:15 (CHANGED LINE)
   Violation: Live API key hardcoded in new code
   Diff: + const API_KEY = 'sk_live_abc123xyz';
   Standards: SOC 2
   Impact: Secret exposure in git history
   Fix: Use environment variable
   Example:
     ```typescript
     const API_KEY = process.env.API_KEY;
     ```

=== WARNINGS (NON-BLOCKING) ===
Count: 1

1. src/lib/logger.ts:30 (CHANGED LINE)
   Warning: Raw req.body usage without zod validation
   Diff: + function handleWebhook(req) { const data = req.body; ... }
   Standards: Best practice
   Suggestion: const data = WebhookSchema.parse(req.body);

=== COMPLIANCE STATUS ===
✓ HIPAA (Healthcare): PASS
✗ GDPR (Europe): FAIL (email in logs - new code)
✓ PCI DSS (Payment): PASS
✓ PIPEDA (Canada): PASS
✓ CCPA (California): PASS
✗ SOC 2 (Security): FAIL (hardcoded secret - new code)

=== PII DETECTED ===
- src/features/auth/login.ts:42 - Email (new code)

=== REQUIRED ACTIONS ===
Before merge:
1. Remove email from login log (line 42)
2. Move API key to environment variable (line 15)

Recommended (not blocking):
3. Add zod validation for webhook handler

===========================================
SUMMARY
===========================================
Critical: 2 (BLOCKS MERGE)
Warnings: 1 (should fix)
Changed files: 3 scanned, 2 with issues
Compliance: 4/6 passing

Status: ❌ CRITICAL ISSUES - CANNOT MERGE
(or: Status: ✅ ALL CHECKS PASSED)

Next Steps:
1. Fix critical violations
2. Run /security-review-diff again
3. For full codebase audit: /security-review
````

## What's Different from /security-review

| Aspect | `/security-review` | `/security-review-diff` |
| --- | --- | --- |
| **Scope** | Entire codebase (or specified path) | Only changed files (git diff) |
| **Speed** | Slower (scans everything) | Fast (scans only changes) |
| **Use case** | Full audits, monthly reviews | Pre-PR, pre-commit checks |
| **Output** | All violations in codebase | Only violations in changed code |
| **False positives** | May flag pre-existing issues | Only flags new/modified issues |
| **CI/CD** | Scheduled audits | PR checks |

## Ignored in Diff Scan

The following changed files are noted but not scanned for security issues:

- Test files (`*.test.ts`, `*.spec.ts`, `__tests__/**`)
- Documentation (`*.md`, `docs/**`)
- Configuration (`*.json`, `*.yaml`, `*.toml`) — except for secret detection
- Lock files (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`)
- Build output (`dist/**`, `build/**`)
- Type declarations (`*.d.ts`)

## After Running

**If critical issues found:**

```bash
# Fix the flagged issues in your changes
# Re-run diff review
/security-review-diff

# For detailed scans on specific concerns:
/pii-scanner src/features/auth/
/secrets-check src/features/auth/
```

**If all passed:**

```bash
# Safe to create PR
# Consider running /security-review for a full audit periodically
```

## CI/CD Integration

Ideal for PR pipelines — only scans what changed:

```yaml
name: Security Review (PR)

on: [pull_request]

jobs:
  security-diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Full history for diff

      - name: Security Review (Changed Files)
        run: |
          claude /security-review-diff ${{ github.event.pull_request.base.ref }}
```

## Tips

- **Use in pre-commit hooks** for instant feedback
- **Pair with `/security-review`** — run diff before PRs, full scan monthly
- **Focus on critical issues** — warnings can be addressed later
- **Review the full file** when a security-critical file is flagged, not just the diff

## Related Skills

- `/security-review` - Full codebase security checklist
- `/pii-scanner` - Detailed PII detection
- `/secrets-check` - Enhanced secret detection
- `/compliance-check` - Full HIPAA/GDPR/PCI DSS validation

## Compliance Documentation

Review before implementing security-sensitive features:

1. **Security Guidelines**: `docs/SECURITY_GUIDELINES.md`
2. **HIPAA, CCPA, SOC 2**: `docs/compliance/usa-hipaa-ccpa-soc2.md`
3. **PIPEDA**: `docs/compliance/canada-pipeda.md`
4. **GDPR**: `docs/compliance/europe-gdpr.md`
5. **Logging & Data Protection**: `docs/compliance/logging-data-protection.md`
6. **Gap Analysis**: `docs/compliance/gap-analysis-roadmap.md`
