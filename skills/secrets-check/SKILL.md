---
name: secrets-check
description: Enhanced secret detection with gitleaks patterns. Use when scanning for hardcoded secrets.
---

# Secrets Check Skill

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

### Secret Detection Patterns

| Secret Type | Regex Pattern |
|-------------|--------------|
| Stripe API keys | `sk_live_[a-zA-Z0-9]+`, `pk_live_[a-zA-Z0-9]+` |
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| Bearer Token | `Bearer [a-zA-Z0-9._-]+` |
| Password assignment | `password\s*=\s*["'][^"']+["']` |

### Auto-Remediation Guidance

**Can auto-fix** (with approval):

- Replace hardcoded URLs with env vars
- Add basic zod schemas (simple cases)
- Update password length in zod schemas

**Cannot auto-fix** (requires manual review):

- Removing PII from logs (context-dependent)
- Implementing httpOnly cookies (requires server setup)
- Complex input validation logic
- Session timeout implementation

## Purpose

Enhanced secret detection using gitleaks + custom patterns to find hardcoded secrets, API keys, credentials, and sensitive configuration data.

## When to Use

- Before committing environment-sensitive code
- Auditing codebase for hardcoded secrets
- After refactoring configuration
- Before PR creation
- Monthly security audits

## Usage

```
/secrets-check [path]
```

Examples:

```
/secrets-check                          # Check entire codebase
/secrets-check src/                     # Check specific directory
/secrets-check src/infra/               # Check infrastructure code
/secrets-check src/config/              # Check configuration files
```

## What It Checks

### 1. Gitleaks Rules

Runs the patterns defined in the bundled `.gitleaks.toml` (or your project's `.gitleaks.toml` if present):

1. **Generic API Key** - `api[_-]?key`
2. **Generic Secret** - `secret[_-]?key`
3. **Password in Code** - `password\s*=`
4. **Private Key** - `-----BEGIN PRIVATE KEY-----`
5. **AWS Access Key** - `AKIA[0-9A-Z]{16}`
6. **Stripe API Key** - `sk_live_[0-9a-zA-Z]{24}`
7. **GitHub Token** - `ghp_[0-9a-zA-Z]{36}`
8. **Firebase API Key** - Firebase configuration objects
9. **JWT Secret** - `jwt[_-]?secret`
10. **Database URL** - Connection strings with credentials
11. **OAuth Client Secret** - OAuth configuration
12. **Encryption Key** - `encryption[_-]?key`
13. **Auth Token** - `auth[_-]?token`
14. **Bearer Token** - `Bearer [A-Za-z0-9-._~+/]+=*`

**Note:** If your project has its own `.gitleaks.toml`, it takes precedence. Otherwise, the plugin's bundled `config/.gitleaks.toml` is used as the default.

### 2. Additional Custom Patterns

Beyond gitleaks, this skill checks for:

**API Keys in Comments:**

```typescript
// ❌ VIOLATION: API key in comment
// Use this API key: sk_live_1234567890abcdef
const stripe = new Stripe(process.env.STRIPE_KEY);
```

**Database Credentials in Code:**

```typescript
// ❌ VIOLATION: Hardcoded credentials
const db = new Database({
  host: 'localhost',
  user: 'admin',
  password: 'password123', // NEVER!
  database: 'myapp',
});
```

**Encryption Keys/Salts:**

```typescript
// ❌ VIOLATION: Hardcoded encryption key
const ENCRYPTION_KEY = 'my-secret-key-12345';

function encryptData(data: string) {
  return crypto.encrypt(data, ENCRYPTION_KEY);
}
```

**OAuth Client Secrets:**

```typescript
// ❌ VIOLATION: OAuth secret in code
const oauth = {
  clientId: 'abc123',
  clientSecret: 'xyz789secretvalue', // NEVER!
  redirectUri: 'https://example.com/callback',
};
```

**Third-Party Service Tokens:**

```typescript
// ❌ VIOLATION: Hardcoded tokens
const twilioClient = twilio(
  'ACxxxxxxxxxxxxx',
  'auth_token_12345' // NEVER!
);

const sendgridClient = new SendGrid('SG.xxxxxxxxxxxxxx'); // NEVER!
```

**Firebase Config Objects:**

```typescript
// ❌ VIOLATION: Firebase config with API key
const firebaseConfig = {
  apiKey: 'AIzaSyC1234567890abcdef', // Should be env var
  authDomain: 'myapp.firebaseapp.com',
  projectId: 'myapp',
};
```

**AWS/GCP Credentials:**

```typescript
// ❌ VIOLATION: Cloud credentials
const AWS = require('aws-sdk');
AWS.config.update({
  accessKeyId: 'AKIAIOSFODNN7EXAMPLE',
  secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
});
```

### 3. Base64-Encoded Secrets

Detects Base64-encoded strings that look like secrets:

```typescript
// ❌ VIOLATION: Base64-encoded secret
const secret = 'c2VjcmV0LWtleS0xMjM0NTY='; // Decodes to 'secret-key-123456'
```

## Anti-Patterns Detected

### 1. API Keys in Code

```typescript
// ❌ VIOLATION
const STRIPE_API_KEY = 'sk_live_1234567890abcdef';

// ✅ FIXED
const STRIPE_API_KEY = process.env.STRIPE_API_KEY;
if (!STRIPE_API_KEY) {
  throw new Error('STRIPE_API_KEY environment variable is required');
}
```

### 2. Hardcoded Passwords

```typescript
// ❌ VIOLATION
const dbConfig = {
  password: 'admin123',
};

// ✅ FIXED
const dbConfig = {
  password: process.env.DB_PASSWORD,
};
```

### 3. Firebase Config

```typescript
// ❌ VIOLATION: Hardcoded Firebase config
const firebaseConfig = {
  apiKey: 'AIzaSyC1234567890abcdef',
  authDomain: 'myapp.firebaseapp.com',
  projectId: 'myapp',
  storageBucket: 'myapp.appspot.com',
};

// ✅ FIXED: Environment variables
const firebaseConfig = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: process.env.FIREBASE_AUTH_DOMAIN,
  projectId: process.env.FIREBASE_PROJECT_ID,
  storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
};
```

### 4. JWT Secrets

```typescript
// ❌ VIOLATION
const JWT_SECRET = 'my-super-secret-key';

function signToken(payload: any) {
  return jwt.sign(payload, JWT_SECRET);
}

// ✅ FIXED
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET) {
  throw new Error('JWT_SECRET environment variable is required');
}

function signToken(payload: any) {
  return jwt.sign(payload, JWT_SECRET);
}
```

### 5. Third-Party Tokens

```typescript
// ❌ VIOLATION: Twilio credentials
const twilioClient = twilio('ACxxxxx', 'auth_token_12345');

// ✅ FIXED
const twilioClient = twilio(process.env.TWILIO_ACCOUNT_SID, process.env.TWILIO_AUTH_TOKEN);
```

## Output Format

### Clean Result

```
✅ Secrets Check: CLEAN

Scanned:
- src/           (243 files)
- config/        (12 files)

No secrets detected. ✅

Gitleaks rules: 14 patterns checked
Custom patterns: 8 additional patterns checked

Total files scanned: 255
Total secrets found: 0
```

### Violations Found

```
🔐 Secrets Check Results:

❌ Secrets Detected (5 violations):

---

❌ Violation 1: Stripe API Key (Live)

File: src/config/api.ts:5
Rule: Stripe Live API Key
Code: const API_KEY = 'sk_live_1234567890abcdef';

Issue: Live Stripe API key hardcoded in source code
Severity: CRITICAL
Impact: Full access to Stripe account, payment processing

Fix:
1. Revoke this API key immediately in Stripe dashboard
2. Generate new key
3. Move to environment variable:
   - Add to .env.local: STRIPE_API_KEY=sk_live_...
   - Use: process.env.STRIPE_API_KEY
   - Add .env.local to .gitignore (already included)

---

Summary:
- Total violations: 5
- Critical: 4
- High: 1

Compliance Impact:
- SOC 2: Violation (hardcoded secrets)
- Security: Critical risk (credential exposure)

Recommendation:
FIX IMMEDIATELY - These secrets are exposed in version control
```

## Validation Rules

### Critical (Immediate Action Required)

1. **Live API keys** - Stripe, Twilio, SendGrid production keys
2. **Database credentials** - Passwords, connection strings
3. **Cloud credentials** - AWS, GCP, Azure access keys
4. **JWT secrets** - Authentication/authorization secrets
5. **Private keys** - RSA, SSH, SSL private keys

### High Priority

1. **Development API keys** - Stripe test mode, sandbox keys
2. **OAuth secrets** - Client secrets for OAuth flows
3. **Encryption keys** - Data encryption keys
4. **Session secrets** - Session signing keys

### Allowed Patterns

The following are **not** violations (allowlist):

1. **Example files**: `.env.example`, `.env.template`, `config.example.ts`
2. **Test files**: `*.test.ts`, `*.spec.ts`, `__tests__/**`
3. **Mock data**: `/fixtures/**`, `/mocks/**`
4. **Documentation**: `*.md`, `docs/**`

## After Running

**If secrets found:**

### 1. Revoke Secrets Immediately

```bash
# Stripe
# Go to https://dashboard.stripe.com/apikeys
# Revoke the exposed key

# AWS
aws iam delete-access-key --access-key-id AKIA...
aws iam create-access-key --user-name <username>
```

### 2. Move to Environment Variables

```typescript
// Before: Hardcoded secret
const STRIPE_KEY = 'sk_live_1234567890abcdef';

// After: Environment variable
const STRIPE_KEY = process.env.STRIPE_API_KEY;

if (!STRIPE_KEY) {
  throw new Error('STRIPE_API_KEY environment variable is required');
}
```

### 3. Verify .gitignore

```bash
# Ensure these are in .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
```

### 4. Audit Git History

```bash
# Search for leaked secrets in git history
git log -p -S "sk_live_" -- "*.ts" "*.tsx" "*.js"
git log -p -S "AKIA" -- "*.ts" "*.tsx" "*.js"
```

### 5. Re-run Secrets Check

```bash
/secrets-check
```

## Tips

- **Use .env files** - Store all secrets in .env (never commit)
- **Generate strong secrets** - Use `openssl rand -base64 32`
- **Rotate regularly** - Change secrets every 90 days
- **Different per environment** - Dev/staging/prod use different secrets
- **Secret management** - Consider AWS Secrets Manager, HashiCorp Vault
- **CI/CD secrets** - Use GitHub Secrets, GitLab CI/CD variables
- **Audit frequently** - Run secrets check before every commit
- **Enable GitHub scanning** - Use GitHub's secret scanning (if on GitHub)

## Gitleaks Configuration

The skill uses `.gitleaks.toml` with these settings:

- **14+ secret patterns** defined
- **Allowlist** for example files, tests, docs
- **High entropy detection** for random strings
- **Custom regexes** for Firebase, Stripe, AWS, etc.

See the plugin's `config/.gitleaks.toml` for the bundled default configuration.

## Related Skills

- `/security-review` - Full security checklist
- `/pii-scanner` - Detect PII in logs
- `/compliance-check` - HIPAA/GDPR/PCI DSS validation

## CI/CD Integration

Add to your CI/CD pipeline:

```yaml
name: Security Scan

on: [pull_request, push]

jobs:
  secrets:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Secrets Check (Gitleaks)
        run: |
          docker run --rm -v $PWD:/path zricethezav/gitleaks:latest \
            detect --source="/path" -v -c /path/.gitleaks.toml
```

## Security Incident Response

If secrets are exposed in git history:

1. **Revoke immediately** - Assume compromised
2. **Rotate secrets** - Generate new ones
3. **Audit access logs** - Check for unauthorized use
4. **Clean git history** - Use BFG Repo-Cleaner or git-filter-repo
5. **Force push** - Update remote history (if feasible)
6. **Notify team** - Inform everyone of the incident
7. **Document** - Record what happened and remediation steps
