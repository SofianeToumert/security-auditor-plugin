---
name: security-review-diff
description: Security checklist on changed files only (git diff). Use before PRs or commits.
---

# Security Review Diff Skill

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

### Clean Result

```
🔒 Security Review (Diff): PASSED

Base: main
Changed files: 5 files scanned

Files reviewed:
  src/features/auth/login.ts        (modified)
  src/features/auth/register.ts     (modified)
  src/lib/api-client.ts             (modified)
  src/components/LoginForm.tsx       (modified)
  src/hooks/useAuth.ts              (new)

✅ Input validation: Zod schemas found in new handlers
✅ No PII in logs: No PII detected in changes
✅ Secrets: No hardcoded secrets in diff
✅ HTTPS: All new URLs use HTTPS
✅ Security headers: No changes to headers
✅ Token storage: Using httpOnly cookies
✅ Data encryption: HTTPS enforced for new API calls
✅ Error messages: Generic messages in new error handlers

All security checks passed for changed files. ✅

Compliance: HIPAA ✓ GDPR ✓ PCI DSS ✓ PIPEDA ✓ CCPA ✓ SOC 2 ✓
```

### Violations Found

```
🔒 Security Review (Diff) Results:

Base: main
Changed files: 3 files scanned

Files reviewed:
  src/features/auth/login.ts        (modified)  ← 2 issues
  src/lib/logger.ts                 (modified)  ← 1 issue
  src/components/PaymentForm.tsx     (new)       ← clean

---

❌ PII in logs (CHANGED LINE)

File: src/features/auth/login.ts:42
Diff: + console.log('Login attempt:', { email: user.email });

Issue: Email address logged in new code
Standards: GDPR, HIPAA, CCPA
Severity: CRITICAL

Fix:
  logger.info('Login attempt', { userId: user.id });

---

❌ Hardcoded secret (CHANGED LINE)

File: src/features/auth/login.ts:15
Diff: + const API_KEY = 'sk_live_abc123xyz';

Issue: Live API key hardcoded in new code
Standards: SOC 2
Severity: CRITICAL

Fix:
  const API_KEY = process.env.API_KEY;

---

⚠️  Missing input validation (CHANGED LINE)

File: src/lib/logger.ts:30
Diff: + function handleWebhook(req) { const data = req.body; ... }

Issue: Raw req.body usage without zod validation
Standards: Best practice
Severity: WARNING

Fix:
  const data = WebhookSchema.parse(req.body);

---

Summary:
- Changed files scanned: 3
- Clean files: 1
- Files with issues: 2
- Critical: 2 (BLOCKS MERGE)
- Warnings: 1

Compliance Impact:
- ❌ GDPR: Violation (email in logs - new code)
- ❌ SOC 2: Violation (hardcoded secret - new code)

Recommendation:
FIX CRITICAL ISSUES before merging:
1. Remove email from login log (line 42)
2. Move API key to environment variable (line 15)

For a full codebase audit, run: /security-review
```

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
