---
name: pii-scanner
description: Find PII in logs for HIPAA/GDPR/PCI DSS compliance. Use when auditing for PII exposure.
---

# PII Scanner Skill

## Purpose

Specialized scanner for PII (Personally Identifiable Information) exposure in logs, error messages, and console output to ensure HIPAA/GDPR/PCI DSS/PIPEDA/CCPA compliance.

## When to Use

- Before committing logging changes
- Auditing existing codebase for PII leaks
- After implementing new features with user data
- Before security review
- As pre-commit hook validation

## Usage

```
/pii-scanner [path]
```

Examples:

```
/pii-scanner                          # Scan entire codebase
/pii-scanner src/                     # Scan specific directory
/pii-scanner src/features/            # Scan features directory
/pii-scanner src/lib/logger.ts        # Scan specific file
```

## What It Scans For

### PII Field Patterns

The skill detects common PII field names in logging statements:

**Personal Information:**

- `email`, `emailAddress`, `userEmail`, `contactEmail`
- `phone`, `phoneNumber`, `mobile`, `telephone`
- `firstName`, `lastName`, `fullName`, `name` (when logging user objects)
- `address`, `street`, `city`, `zipCode`, `postalCode`, `country`

**Sensitive Identifiers:**

- `ssn`, `socialSecurityNumber`
- `dob`, `dateOfBirth`, `birthDate`
- `driversLicense`, `passport`
- `ipAddress`, `ip`, `clientIp`

**Healthcare (PHI under HIPAA):**

- `mrn`, `medicalRecordNumber`
- `diagnosis`, `treatment`, `medication`
- `patientId`, `patient` (entire object)
- `healthRecord`, `medicalHistory`

**Financial (PCI DSS):**

- `cardNumber`, `ccNumber`, `creditCard`
- `cvv`, `cvc`, `securityCode`
- `accountNumber`, `bankAccount`
- `routingNumber`

### Logging Patterns Detected

#### Console Logging

```typescript
// ❌ VIOLATION: Direct PII logging
console.log('User email:', user.email);
console.log('Login attempt:', { email: user.email, ip: req.ip });
console.debug({ user }); // Entire user object may contain PII
console.error('Error:', error, { patient }); // PHI exposure
```

#### Logger Libraries

```typescript
// ❌ VIOLATION: PII in structured logging
logger.info('User registered', { email: user.email, phone: user.phone });
logger.error('Failed to load patient', { patientId, mrn });
logger.debug({ user }); // Object logging
```

#### Error Messages

```typescript
// ❌ VIOLATION: PII in error messages
throw new Error(`Invalid email: ${user.email}`);
throw new Error(`User ${user.firstName} ${user.lastName} not found`);
```

## Anti-Patterns Detected

### 1. Email in Logs

```typescript
// ❌ VIOLATION
console.log('Login attempt:', { email: user.email, ip: req.ip });
```

**Why it's a violation:**

- Email is PII under GDPR, PIPEDA, CCPA
- Enables user identification
- Privacy policy violation

**Fix**: Use sanitized logging

```typescript
// ✅ FIXED
logger.info('Login attempt', {
  userId: user.id, // Use non-PII identifier
  timestamp: new Date().toISOString(),
});
```

### 2. Entire User Object

```typescript
// ❌ VIOLATION
logger.error('Update failed', { user });
```

**Why it's a violation:**

- User object likely contains multiple PII fields
- Inadvertent exposure
- Hard to audit what was logged

**Fix**: Log only necessary non-PII fields

```typescript
// ✅ FIXED
logger.error('Update failed', {
  userId: user.id,
  tier: user.tier,
  // Only log non-PII fields
});
```

### 3. PHI in Healthcare Context

```typescript
// ❌ VIOLATION
console.error('Failed to load patient:', error, patient);
logger.info('Patient updated', {
  mrn: patient.medicalRecordNumber,
  diagnosis: patient.diagnosis,
});
```

**Why it's a violation:**

- PHI under HIPAA
- Severe legal consequences
- Audit trail violations

**Fix**: Never log PHI

```typescript
// ✅ FIXED
logger.error('Failed to load patient', {
  patientId: patient.id, // Use non-PHI identifier
  errorCode: error.code,
});

// Use audit logging (separate secure system)
auditLog.record({
  action: 'PATIENT_UPDATE',
  patientId: patient.id,
  userId: currentUser.id,
  timestamp: new Date().toISOString(),
});
```

### 4. Credit Card Data

```typescript
// ❌ VIOLATION
console.log('Payment processed', {
  cardNumber: payment.cardNumber,
  cvv: payment.cvv,
});
```

**Why it's a violation:**

- PCI DSS violation
- Card data must never be logged
- Severe compliance penalties

**Fix**: Never log card data

```typescript
// ✅ FIXED
logger.info('Payment processed', {
  transactionId: payment.transactionId,
  amount: payment.amount,
  last4: payment.cardLast4, // Only last 4 digits allowed
});
```

## Output Format

### Clean Result

```
✅ PII Scanner: CLEAN

Scanned:
- src/           (243 files)

No PII exposure detected. ✅

Compliance: HIPAA ✓ GDPR ✓ PCI DSS ✓ PIPEDA ✓ CCPA ✓
```

### Violations Found

```
🔍 PII Scanner Results:

❌ PII Exposure Found (4 violations):

---

❌ Violation 1: Email + IP Address in Logs

File: src/lib/auth.ts:42
Code:
  console.log('Login attempt:', {
    email: user.email,
    ip: req.ip
  });

Issue: Email and IP address are PII (GDPR, PIPEDA, CCPA)
Severity: CRITICAL

Fix: Use sanitized logging
  logger.info('Login attempt', {
    userId: user.id,
    timestamp: new Date().toISOString(),
  });

Compliance Impact:
- GDPR: Article 5 (Purpose Limitation)
- PIPEDA: Principle 4.3 (Consent)
- CCPA: Right to Know

Reference: docs/compliance/logging-data-protection.md

---

Summary:
- Total violations: 4
- Critical: 3 (HIPAA, GDPR, PCI DSS violations)
- Warning: 1

Compliance Status:
- ❌ HIPAA: Violation (PHI in logs)
- ❌ GDPR: Violation (email in logs)
- ✅ PCI DSS: Clean (no card data)
- ❌ PIPEDA: Violation (personal info in logs)
- ❌ CCPA: Violation (personal info without consent)

Recommendation:
FIX IMMEDIATELY - These violations expose you to:
- HIPAA fines: up to $50,000 per violation
- GDPR fines: up to 4% of annual revenue
- Reputational damage
- Data breach liability

Action Items:
1. Remove all PII from logs
2. Implement sanitization helpers
3. Use audit logging for compliance
4. Review logging & data protection docs: docs/compliance/logging-data-protection.md
```

## Detection Algorithm

### 1. Find Logging Calls

Scan for:

- `console.log()`, `console.error()`, `console.debug()`, `console.info()`, `console.warn()`
- `logger.log()`, `logger.error()`, `logger.debug()`, `logger.info()`, `logger.warn()`
- `throw new Error(...)` with dynamic messages

### 2. Analyze Arguments

Check if arguments contain:

- Direct PII field access (e.g., `user.email`)
- Entire objects that may contain PII (e.g., `{ user }`, `{ patient }`)
- String interpolation with PII (e.g., `` `User ${user.email}` ``)

### 3. PII Field Detection

Match field names against PII patterns:

- Exact match: `email`, `ssn`, `phoneNumber`
- Partial match: `*Email`, `*Phone`, `*Address`
- Object detection: `user`, `patient`, `member` (entire objects)

### 4. Context Analysis

Determine severity based on:

- Field type (PHI > PII > IP address)
- Compliance framework (HIPAA most strict)
- Exposure risk (console.error vs logger.debug)

## Validation Rules

### Compliance Frameworks

| Framework | Applies To           | Severity | Max Fine          |
| --------- | -------------------- | -------- | ----------------- |
| HIPAA     | PHI (health data)    | CRITICAL | $50,000/violation |
| GDPR      | EU residents' data   | CRITICAL | 4% annual revenue |
| PCI DSS   | Payment card data    | CRITICAL | $500,000/month    |
| PIPEDA    | Canadian data        | HIGH     | $100,000          |
| CCPA      | California residents | HIGH     | $7,500/violation  |
| SOC 2     | All user data        | MEDIUM   | Audit failure     |

### Allowed Logging

**Non-PII identifiers:**

- User ID (UUID, not email)
- Session ID
- Transaction ID
- Timestamps
- Error codes
- Non-personal metrics

**Sanitized data:**

- Email domain (not address): `@example.com`
- Last 4 digits of phone: `***-****-1234`
- Last 4 of card: `**** **** **** 1234`
- IP address prefix: `192.168.*.*`

## Sanitization Helpers

The skill suggests creating sanitization helpers in your project:

```typescript
// src/lib/logger/sanitize.ts (suggested location)

export function sanitizeUserData(user: User) {
  return {
    userId: user.id,
    tier: user.tier,
    active: user.active,
    createdAt: user.createdAt,
    // Remove PII fields
  };
}

export function sanitizeError(error: Error, context?: Record<string, any>) {
  return {
    message: error.message,
    code: (error as any).code,
    stack: process.env.NODE_ENV === 'development' ? error.stack : undefined,
    // Remove sensitive context
    context: context ? sanitizeContext(context) : undefined,
  };
}

function sanitizeContext(context: Record<string, any>) {
  const PII_FIELDS = ['email', 'phone', 'address', 'ssn', 'cardNumber'];
  const sanitized: Record<string, any> = {};

  for (const [key, value] of Object.entries(context)) {
    if (!PII_FIELDS.some((field) => key.toLowerCase().includes(field.toLowerCase()))) {
      sanitized[key] = value;
    }
  }

  return sanitized;
}
```

## After Running

**If violations found:**

```bash
# Fix violations manually
# Replace PII logging with sanitized versions

# Re-run scanner
/pii-scanner

# Review compliance docs
# docs/compliance/logging-data-protection.md
```

**If clean:**

```bash
# Add to CI/CD pipeline
# Add to pre-commit hooks
# Continue with commit
```

## CI/CD Integration

Add to your CI/CD pipeline:

```yaml
- name: PII Scanner
  run: |
    npm run pii-scan || exit 1
```

Or as pre-commit hook:

```bash
#!/bin/sh
# .husky/pre-commit
npm run pii-scan || exit 1
```

## Tips

- **Sanitize early** - create helpers from day one
- **Use audit logging** - separate system for compliance
- **Never log PHI** - no exceptions for healthcare data
- **Never log card data** - use tokenization (Stripe, etc.)
- **Review regularly** - run scanner monthly
- **Educate team** - share violations and compliance requirements

## Common False Positives

### Generic field names

```typescript
// May be flagged but safe
const name = 'feature-name';
logger.info('Feature enabled', { name });
```

Skill checks if `name` field is user-related or generic.

### Test data

```typescript
// Test files are allowed to have PII
// fixtures.ts
const testUser = { email: 'test@example.com' };
```

Test files (`.test.ts`, `.spec.ts`) are ignored.

## Related Skills

- `/security-review` - Full security checklist
- `/compliance-check` - HIPAA/GDPR/PCI DSS validation
- `/secrets-check` - Detect hardcoded secrets

## Compliance Documentation

Review before implementing features with PII:

1. **HIPAA, CCPA, SOC 2**: `docs/compliance/usa-hipaa-ccpa-soc2.md`
2. **PIPEDA**: `docs/compliance/canada-pipeda.md`
3. **GDPR**: `docs/compliance/europe-gdpr.md`
4. **Logging & Data Protection**: `docs/compliance/logging-data-protection.md`
5. **Gap Analysis**: `docs/compliance/gap-analysis-roadmap.md`
