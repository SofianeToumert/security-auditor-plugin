---
name: security-review
description: Validate security checklist before PR. Use when reviewing security posture.
---

# Security Review Skill

## Purpose

Run a manual security checklist against code changes to ensure compliance with security best practices before PR creation.

## When to Use

- Before committing feature that handles PII/PHI
- Before PR creation (supplement to automated Claude PR review)
- After implementing authentication/authorization changes
- Manual security audit
- As final check before deployment

## Usage

```
/security-review [path]
```

Examples:

```
/security-review                          # Check entire codebase
/security-review src/                     # Check specific directory
/security-review src/features/auth/       # Check authentication feature
/security-review src/features/payments/   # Check payment processing
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

### All Checks Passed

```
🔒 Security Review: PASSED

Checked: src/features/auth/

✅ Input validation: Found zod schemas in all form handlers
✅ No PII in logs: Clean (0 violations)
✅ Secrets: No hardcoded secrets detected
✅ HTTPS: All URLs use HTTPS (except localhost)
✅ Security headers: Configured in middleware
✅ Token storage: Using httpOnly cookies
✅ Data encryption: HTTPS enforced, database encrypted
✅ Error messages: Generic messages, detailed logging server-side

All security requirements met. ✅

Compliance: HIPAA ✓ GDPR ✓ PCI DSS ✓ PIPEDA ✓ CCPA ✓ SOC 2 ✓
```

### Violations Found

```
🔒 Security Review Results:

Checked: src/features/auth/

✅ Input validation: Found zod schemas in all form handlers

❌ PII in logs: Found potential PII exposure
   File: src/lib/logger.ts:42
   Code: console.log('User email:', user.email);
   Fix: Remove PII from logs or use sanitization
   Reference: docs/compliance/logging-data-protection.md

✅ Secrets: No hardcoded secrets detected

⚠️  HTTPS: Found 1 http:// URL
   File: src/config.ts:5
   Code: const API_URL = 'http://localhost:3000';
   Note: localhost is acceptable for development

❌ Security headers: Not configured
   Issue: Missing CSP, HSTS, X-Frame-Options
   Fix: Add security headers to your middleware

⚠️  Token storage: Using localStorage (XSS vulnerable)
   File: src/lib/auth.ts:25
   Code: localStorage.setItem('token', authToken);
   Fix: Use httpOnly cookies instead
   Severity: HIGH

✅ Data encryption: HTTPS enforced

❌ Error messages: Technical details exposed
   File: src/api/users.ts:45
   Code: res.status(500).json({ error: error.message });
   Fix: Use generic error messages
   Example:
     res.status(500).json({
       error: 'An error occurred. Please try again.',
       errorId: generateErrorId(),
     });

---

Summary:
- Passed: 4/8
- Failed: 3/8 (Critical)
- Warnings: 1/8

Critical Issues:
1. PII in logs (HIPAA, GDPR violation)
2. Security headers missing (SOC 2 requirement)
3. Error messages expose technical details

Compliance Impact:
- ❌ HIPAA: Violation (PII in logs)
- ❌ GDPR: Violation (PII in logs)
- ⚠️  PCI DSS: Warning (token storage)
- ⚠️  SOC 2: Warning (security headers)

Recommendation:
FIX CRITICAL ISSUES before merging:
1. Remove PII from logs (/pii-scanner for detailed scan)
2. Configure security headers in middleware
3. Use generic error messages

Reference Documentation:
- Security Guidelines: docs/SECURITY_GUIDELINES.md
- Compliance Docs: docs/compliance/
```

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
