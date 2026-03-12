---
name: security-review
description: Full codebase security checklist. Use for comprehensive security audits.
---

# Security Review Skill

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

Run the full 8-item security checklist against the **entire codebase** (or a specified directory) to ensure compliance with security best practices. This is a comprehensive scan — it checks all files, not just changed ones.

For scanning only changed files (e.g., before a PR), use `/security-review-diff` instead.

## When to Use

- Comprehensive security audit of the full codebase
- After implementing authentication/authorization changes
- Monthly or quarterly security reviews
- Before major releases or deployments
- Initial security assessment of a new project

## Usage

```
/security-review [path]
```

**Scope:** Scans ALL files in the target path. Defaults to the entire codebase.

Examples:

```
/security-review                          # Scan entire codebase
/security-review src/                     # Scan all source files
/security-review src/features/auth/       # Scan authentication feature
/security-review src/features/payments/   # Scan payment processing
```

## Security Checklist

### 1. Input Validation

**Requirement**: Input validated with zod schemas (no raw user input accepted)

**Checks:**

- All form inputs validated with zod/yup/joi
- API endpoints validate request bodies
- No direct `req.body` usage without validation
- Type-safe input parsing

**Anti-patterns:**

```typescript
// ❌ VIOLATION: Raw user input
app.post('/api/users', (req, res) => {
  const { email, password } = req.body; // No validation!
  createUser(email, password);
});

// ❌ VIOLATION: Manual validation
if (!email || !password) {
  // Not comprehensive enough
}
```

**Fix:**

```typescript
// ✅ FIXED: Zod validation
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(12), // PCI DSS 4.0 requirement
});

app.post('/api/users', (req, res) => {
  const input = CreateUserSchema.parse(req.body);
  createUser(input.email, input.password);
});
```

---

### 2. No PII in Logs

**Requirement**: No PII in logs, error messages, or console output

**Checks:**

- No email, phone, SSN in console.log
- No PHI (medical records, diagnosis) in logs
- No card data in logs
- Error messages sanitized (no user data)

**See**: `/pii-scanner` for detailed PII detection

---

### 3. Secrets Management

**Requirement**: Secrets via environment variables (never hardcoded)

**Checks:**

- No hardcoded API keys
- No hardcoded passwords
- No database credentials in code
- Environment variables for all secrets
- `.env` files in `.gitignore`

**Anti-patterns:**

```typescript
// ❌ VIOLATION: Hardcoded secret
const API_KEY = 'sk_live_1234567890abcdef';

// ❌ VIOLATION: Credentials in code
const db = new Database({
  host: 'localhost',
  user: 'admin',
  password: 'password123', // NEVER!
});
```

**Fix:**

```typescript
// ✅ FIXED: Environment variables
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');

// ✅ FIXED: Env-based config
const db = new Database({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
});
```

**See**: `/secrets-check` for detailed secret detection

---

### 4. HTTPS Enforcement

**Requirement**: HTTPS enforced (no http:// URLs in production code)

**Checks:**

- No `http://` URLs (except localhost)
- API calls use `https://`
- External resources over HTTPS
- Middleware enforces HTTPS

**Anti-patterns:**

```typescript
// ❌ VIOLATION: HTTP in production
const API_URL = 'http://api.example.com';

// ❌ VIOLATION: Mixed content
<img src="http://example.com/image.jpg" />
```

**Fix:**

```typescript
// ✅ FIXED: HTTPS only
const API_URL = process.env.NODE_ENV === 'development'
  ? 'http://localhost:3000'
  : 'https://api.example.com';

// ✅ FIXED: HTTPS resources
<img src="https://example.com/image.jpg" />
```

---

### 5. Security Headers

**Requirement**: Security headers configured (when implementing web middleware)

**Checks:**

- CSP (Content Security Policy) configured
- HSTS (HTTP Strict Transport Security)
- X-Frame-Options (prevent clickjacking)
- X-Content-Type-Options (prevent MIME sniffing)

**Implementation:**

```typescript
// Example middleware
function securityHeaders(req, res, next) {
  res.setHeader('Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';");
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  next();
}
```

---

### 6. Token Storage

**Requirement**: Authentication tokens stored securely (httpOnly cookies or secure storage)

**Checks:**

- No tokens in localStorage (vulnerable to XSS)
- Use httpOnly cookies (server-side only)
- Use Secure storage (React Native)
- Tokens encrypted at rest

**Anti-patterns:**

```typescript
// ❌ VIOLATION: Token in localStorage (XSS vulnerable)
localStorage.setItem('token', authToken);

// ❌ VIOLATION: Token in sessionStorage
sessionStorage.setItem('token', authToken);
```

**Fix:**

```typescript
// ✅ FIXED: httpOnly cookie (web)
// Server-side
res.cookie('token', authToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 3600000, // 1 hour
});

// ✅ FIXED: Secure storage (React Native)
import * as SecureStore from 'expo-secure-store';

await SecureStore.setItemAsync('token', authToken);
```

---

### 7. Data Encryption

**Requirement**: Data encrypted in transit (HTTPS) and at rest (when applicable)

**Checks:**

- HTTPS for all API calls (in transit)
- Database encryption enabled (at rest)
- File storage encryption (S3, etc.)
- Backup encryption

**Implementation:**

```typescript
// ✅ In transit: HTTPS enforced
fetch('https://api.example.com/data');

// ✅ At rest: Database encryption
// PostgreSQL: Enable SSL, encrypt backups
// MongoDB: Enable encryption at rest

// ✅ File storage: S3 encryption
const s3 = new AWS.S3({
  serverSideEncryption: 'AES256',
});
```

---

### 8. Error Messages

**Requirement**: Error messages user-friendly (no technical details exposed)

**Checks:**

- Generic error messages to users
- Detailed errors logged server-side only
- No stack traces in production
- No database errors exposed

**Anti-patterns:**

```typescript
// ❌ VIOLATION: Exposing technical details
try {
  await db.query('SELECT * FROM users WHERE id = $1', [userId]);
} catch (error) {
  res.status(500).json({ error: error.message }); // Exposes SQL!
}

// ❌ VIOLATION: Stack trace to user
catch (error) {
  res.status(500).json({ error: error.stack });
}
```

**Fix:**

```typescript
// ✅ FIXED: Generic message + server-side logging
try {
  await db.query('SELECT * FROM users WHERE id = $1', [userId]);
} catch (error) {
  logger.error('Database query failed', {
    userId,
    error: error.message,
    stack: error.stack,
  });

  res.status(500).json({
    error: 'An error occurred. Please try again later.',
    errorId: generateErrorId(), // For support reference
  });
}
```

---

## Output Format

Use the following canonical `===` delimited report structure:

````
===========================================
SECURITY-AUDITOR: Security & Compliance Report
===========================================

=== CRITICAL VIOLATIONS (BLOCKS MERGE) ===
Count: 2

1. src/auth/login.ts:45
   Violation: Email logged in error message
   Code: console.error('Login failed for user:', email)
   Standards: GDPR, HIPAA, CCPA
   Impact: Personal data leakage in logs
   Fix: Use user ID instead, mask email if logging required
   Example:
     ```typescript
     // Bad
     console.error('Login failed for user:', email);

     // Good
     console.error('Login failed', { userId: user.id });
     ```

2. src/config/api.ts:12
   Violation: Hardcoded API key
   Code: const API_KEY = "sk_live_abc123xyz"
   Standards: SOC 2
   Impact: Secret exposure in git history
   Fix: Use environment variable
   Example:
     ```typescript
     // Bad
     const API_KEY = "sk_live_abc123xyz";

     // Good
     const API_KEY = process.env.STRIPE_API_KEY;
     ```

=== WARNINGS (NON-BLOCKING) ===
Count: 3

1. src/components/Form.tsx:78
   Warning: Missing zod validation
   Code: const handleSubmit = (data) => { ... }
   Standards: Best practice
   Impact: Potential XSS, SQL injection via unvalidated input
   Suggestion: Add zod schema validation

=== COMPLIANCE STATUS ===
✓ HIPAA (Healthcare): PASS
✗ GDPR (Europe): FAIL (email in logs)
✓ PCI DSS (Payment): PASS
✓ PIPEDA (Canada): PASS
✓ CCPA (California): PASS
✗ SOC 2 (Security): FAIL (hardcoded secret)

=== PII DETECTED ===
Locations where PII found (masked in logs):
- src/auth/login.ts:45 - Email
- src/components/Profile.tsx:23 - Phone number
- src/utils/logger.ts:67 - IP address

=== REQUIRED ACTIONS ===
Before merge:
1. Remove email from login error message (line 45)
2. Move API key to environment variable (line 12)

Recommended (not blocking):
3. Add zod validation to Form component

===========================================
SUMMARY
===========================================
Critical: 2 (BLOCKS MERGE)
Warnings: 3 (should fix)
Compliance: 4/6 passing

Status: ❌ CRITICAL ISSUES - CANNOT MERGE
(or: Status: ✅ ALL CHECKS PASSED)

Next Steps:
1. Fix critical violations
2. Run /security-review again
3. Consider implementing warnings
````

## Validation Rules

### Critical (Blocks Merge)

These must be fixed before PR approval:

1. **PII in logs** - HIPAA/GDPR violation
2. **Hardcoded secrets** - Security breach risk
3. **No input validation** - Injection attack risk
4. **localStorage for tokens** - XSS vulnerability

### High Priority

Fix before deploying to production:

1. **Security headers missing** - SOC 2 requirement
2. **HTTP URLs in production** - PCI DSS violation
3. **Error messages expose details** - Information leakage

### Medium Priority

Fix when possible:

1. **HTTPS warning for localhost** - Development only
2. **Weak validation** - Improve validation schemas

## After Running

**If critical issues found:**

```bash
# Fix issues manually
# Re-run security review
/security-review src/features/auth/

# Run related skills for detailed scans
/pii-scanner
/secrets-check
```

**If all passed:**

```bash
# Continue with PR creation
```

## CI/CD Integration

Add to your CI/CD pipeline:

```yaml
name: Security Review

on: [pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Security Review
        run: |
          # Run your security review script
          npm run security-review || exit 1

      - name: PII Scanner
        run: |
          npm run pii-scan || exit 1

      - name: Secrets Check
        run: |
          npm run secrets-check || exit 1
```

## Tips

- **Run before PR** - catch issues early
- **Use with other skills** - `/pii-scanner`, `/secrets-check`, `/compliance-check`
- **Review compliance docs** before implementing features
- **Educate team** - share security requirements
- **Test security** - include security tests in test suite

## Compliance Documentation

Review before implementing security-sensitive features:

1. **Security Guidelines**: `docs/SECURITY_GUIDELINES.md`
2. **HIPAA, CCPA, SOC 2**: `docs/compliance/usa-hipaa-ccpa-soc2.md`
3. **PIPEDA**: `docs/compliance/canada-pipeda.md`
4. **GDPR**: `docs/compliance/europe-gdpr.md`
5. **Logging & Data Protection**: `docs/compliance/logging-data-protection.md`
6. **Gap Analysis**: `docs/compliance/gap-analysis-roadmap.md`

## Related Skills

- `/security-review-diff` - Same checklist, but only on changed files (git diff)
- `/pii-scanner` - Detailed PII detection
- `/secrets-check` - Enhanced secret detection
- `/compliance-check` - Full HIPAA/GDPR/PCI DSS validation

## PCI DSS 4.0 Requirements

Special attention for payment processing:

- Password minimum 12 characters (not 8!)
- No card data stored locally
- Tokenization via Stripe/payment provider
- HTTPS enforced
- Session timeout configured (15 minutes idle)
- Strong cryptography (TLS 1.2+)
