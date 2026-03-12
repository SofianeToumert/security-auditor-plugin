---
name: compliance-check
description: Full HIPAA/GDPR/PCI DSS/PIPEDA/CCPA/SOC 2 validation. Use when checking compliance.
---

# Compliance Check Skill

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

### Compliance Rules Detail

**GDPR:**

- No PII in logs (CRITICAL)
- Consent required for data collection
- Right to access/delete implementation
- 72-hour breach notification

**HIPAA:**

- PHI encryption in transit and at rest
- Audit logs for access
- No PHI in error messages (CRITICAL)
- 24-hour breach notification

**PCI DSS 4.0:**

- 12+ character passwords (updated from 8)
- No card data storage (use Stripe tokenization)
- HTTPS only (CRITICAL)
- 15-minute session timeout (CRITICAL)
- MFA for admin access

**SOC 2:**

- No hardcoded secrets (CRITICAL)
- Access control logging
- Change management
- Incident response procedures

**PIPEDA:**

- Consent for collection, use, disclosure
- Purpose limitation
- Safeguards proportional to sensitivity
- Openness about practices

**CCPA:**

- Right to know what data is collected
- Right to delete
- Right to opt-out of sale
- Non-discrimination for exercising rights

## Purpose

Comprehensive compliance validation against HIPAA, GDPR, PCI DSS, PIPEDA, CCPA, and SOC 2 requirements for healthcare and personal data handling.

## When to Use

- Before implementing features that handle:
  - User authentication/authorization
  - Personal Health Information (PHI)
  - Payment data
  - Personal Identifiable Information (PII)
- Before PR for compliance-sensitive features
- Monthly compliance audits
- After implementing data handling logic

## Usage

```
/compliance-check [path]
```

Examples:

```
/compliance-check                          # Check entire codebase
/compliance-check src/                     # Check specific directory
/compliance-check src/users/               # Check user handling code
/compliance-check src/features/patients/   # Check PHI handling
```

## Compliance Frameworks

### HIPAA (Healthcare - USA)

**Applies to:** Personal Health Information (PHI) handling

**Requirements:**

- PHI encrypted at rest and in transit
- Access controls (role-based, minimum necessary)
- Audit logs for PHI access
- No PHI in logs or errors
- Business Associate Agreement (BAA) for third-party services
- Data breach notification procedures
- Patient rights (access, amendment, accounting)

**What It Checks:**

**PHI Detection:**

```typescript
// ❌ VIOLATION: PHI in logs
console.log('Patient record:', {
  mrn: patient.medicalRecordNumber,
  diagnosis: patient.diagnosis,
  treatment: patient.treatment,
});

// ❌ VIOLATION: PHI in error messages
throw new Error(`Patient ${patient.mrn} diagnosis: ${patient.diagnosis}`);

// ❌ VIOLATION: PHI not encrypted
const patient = await fetch('/api/patients/123').then((r) => r.json());
localStorage.setItem('patient', JSON.stringify(patient)); // Not encrypted!
```

**Fixes:**

```typescript
// ✅ FIXED: No PHI in logs
logger.info('Patient record accessed', {
  patientId: patient.id, // Non-PHI identifier
  userId: currentUser.id,
  timestamp: new Date().toISOString(),
});

// ✅ FIXED: Generic error message
throw new PatientAccessError('Unable to access patient record');

// ✅ FIXED: PHI encrypted at rest (via SecureStore)
import * as SecureStore from 'expo-secure-store';

await SecureStore.setItemAsync('patient_data', JSON.stringify(patient), {
  keychainAccessible: SecureStore.WHEN_UNLOCKED,
});
```

### GDPR (Europe)

**Applies to:** EU residents' personal data

**Requirements:**

- User consent for data collection
- Right to access (data export)
- Right to erasure (data deletion)
- Data minimization (only collect what's needed)
- Privacy by design
- Data breach notification (72 hours)
- Data processing agreements

**What It Checks:**

**Consent Flows:**

```typescript
// ❌ VIOLATION: No consent before data collection
export async function createUser(data: CreateUserInput) {
  return await db.users.create({
    email: data.email,
    phone: data.phone,
    marketingPreferences: true, // Auto-opted in!
  });
}

// ✅ FIXED: Explicit consent required
export async function createUser(data: CreateUserInput) {
  if (!data.consentGiven) {
    throw new ConsentRequiredError('User must consent to data collection');
  }

  return await db.users.create({
    email: data.email,
    phone: data.phone,
    marketingPreferences: data.marketingConsent || false, // Opt-in only
    consentedAt: new Date().toISOString(),
  });
}
```

**Right to Access:**

```typescript
// ✅ FIXED: Data export endpoint
export async function exportUserData(userId: string): Promise<UserDataExport> {
  const [user, subscriptions, activity] = await Promise.all([
    db.users.findUnique({ where: { id: userId } }),
    db.subscriptions.findMany({ where: { userId } }),
    db.activity.findMany({ where: { userId } }),
  ]);

  return {
    personalInfo: { email: user.email, name: user.name },
    subscriptions,
    activityLog: activity,
    exportedAt: new Date().toISOString(),
  };
}
```

**Right to Erasure:**

```typescript
// ✅ FIXED: Hard delete with cascade
export async function deleteUser(userId: string) {
  await db.$transaction([
    db.activity.deleteMany({ where: { userId } }),
    db.subscriptions.deleteMany({ where: { userId } }),
    db.users.delete({ where: { id: userId } }),
  ]);

  logger.info('User data deleted (GDPR erasure)', {
    userId,
    deletedAt: new Date().toISOString(),
  });
}
```

### PCI DSS (Payment - Global)

**Applies to:** Credit card data handling

**Requirements:**

- No credit card data stored locally
- Tokenization via payment provider (Stripe, etc.)
- HTTPS enforced
- Password requirements (min 12 chars, complexity)
- Session timeout configured (15 minutes idle)
- Strong cryptography (TLS 1.2+)

**What It Checks:**

**Card Data Storage:**

```typescript
// ❌ VIOLATION: Storing card data
export async function processPayment(data: PaymentInput) {
  const payment = await db.payments.create({
    cardNumber: data.cardNumber, // NEVER STORE!
    cvv: data.cvv, // NEVER STORE!
    expiryDate: data.expiryDate,
  });
}

// ✅ FIXED: Use Stripe tokenization
import Stripe from 'stripe';

export async function processPayment(data: PaymentInput) {
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

  const paymentIntent = await stripe.paymentIntents.create({
    amount: data.amount,
    currency: 'usd',
    payment_method: data.stripeToken, // Stripe token, not raw card
  });

  const payment = await db.payments.create({
    stripePaymentIntentId: paymentIntent.id,
    last4: data.last4, // Only last 4 digits allowed
    amount: data.amount,
  });

  return payment;
}
```

**Password Requirements (PCI DSS 4.0):**

```typescript
// ❌ VIOLATION: Weak password requirements (8 chars)
const PasswordSchema = z.object({
  password: z.string().min(8), // PCI DSS 4.0 requires 12!
});

// ✅ FIXED: PCI DSS 4.0 compliant (12 chars minimum)
const PasswordSchema = z.object({
  password: z
    .string()
    .min(12, 'Password must be at least 12 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain special character'),
});
```

### PIPEDA (Canada)

**Applies to:** Personal information of Canadians

**Requirements:**

- Consent for collection, use, disclosure
- Purpose limitation (collect only for stated purpose)
- Accuracy (keep data current)
- Safeguards (protect with security measures)
- Openness (transparent about practices)
- Individual access (right to access own data)
- Accountability (responsible for compliance)

### CCPA (California - USA)

**Applies to:** California residents' personal information

**Requirements:**

- Right to know (what data is collected)
- Right to delete
- Right to opt-out of sale
- Non-discrimination (equal service regardless of opt-out)
- Privacy policy disclosure

**What It Checks:**

```typescript
// ❌ VIOLATION: No opt-out mechanism
export async function shareDataWithPartners(userId: string) {
  const user = await getUser(userId);
  await sendToPartner(user); // No opt-out check!
}

// ✅ FIXED: Respect "Do Not Sell" preference
export async function shareDataWithPartners(userId: string) {
  const user = await getUser(userId);

  if (user.doNotSell) {
    throw new DataSharingOptOutError('User opted out of data sharing');
  }

  await sendToPartner(user);
}
```

### SOC 2 Type II

**Applies to:** All user data (security controls)

**Requirements:**

- Access controls (authentication, authorization)
- Logging and monitoring
- Change management
- Vendor management
- Business continuity
- Incident response

**What It Checks:**

```typescript
// ❌ VIOLATION: No authorization check
export async function deleteOrganization(orgId: string) {
  return await db.organizations.delete({ where: { id: orgId } });
}

// ✅ FIXED: Role-based access control
export async function deleteOrganization(orgId: string, currentUser: User) {
  if (!currentUser.roles.includes('ADMIN')) {
    throw new UnauthorizedError('Only admins can delete organizations');
  }

  return await db.organizations.delete({ where: { id: orgId } });
}
```

## Output Format

### All Compliant

```
✅ Compliance Check: PASSED

Checked: src/features/auth/

Framework Status:
- ✅ HIPAA: Compliant (no violations)
- ✅ GDPR: Compliant (no violations)
- ✅ PCI DSS: Compliant (no violations)
- ✅ PIPEDA: Compliant (no violations)
- ✅ CCPA: Compliant (no violations)
- ✅ SOC 2: Compliant (no violations)

All compliance requirements met. ✅

Compliance Documentation:
- HIPAA, CCPA, SOC 2: docs/compliance/usa-hipaa-ccpa-soc2.md
- PIPEDA: docs/compliance/canada-pipeda.md
- GDPR: docs/compliance/europe-gdpr.md
- Logging & Data Protection: docs/compliance/logging-data-protection.md
```

### Violations Found

```
🛡️ Compliance Check Results:

Checked: src/features/patients/

---

❌ HIPAA Violation (Critical)

File: src/features/patients/PatientProfile.tsx:25
Code: console.error('Failed to load patient:', error, patient);

Issue: PHI (patient object) exposed in console logs
Impact: HIPAA Privacy Rule 45 CFR § 164.502 violation
Penalty: Up to $50,000 per violation
Severity: CRITICAL

Fix: Remove PHI from logs
  logger.error('Failed to load patient', {
    patientId: patient.id,
    errorCode: error.code,
  });

Reference: docs/compliance/logging-data-protection.md

---

Summary:
- Critical violations: 2 (HIPAA, PCI DSS)
- High violations: 1 (GDPR)
- Medium violations: 1 (SOC 2)
- Warnings: 1 (PCI DSS)

Recommendation:
FIX CRITICAL ISSUES before merging:
1. Remove PHI from logs (/pii-scanner for detailed scan)
2. Remove raw card data storage, use Stripe tokenization
3. Require explicit consent for marketing
4. Add authorization checks for sensitive operations
5. Update password requirements to 12+ characters

Action Items:
1. Review compliance docs: docs/compliance/
2. Implement fixes for critical violations
3. Re-run: /compliance-check src/features/patients/
4. Schedule monthly compliance audits
```

## Validation Rules

### Critical (Blocks Merge)

These violations **must be fixed** before PR approval:

1. **PHI in logs** - HIPAA violation
2. **Card data storage** - PCI DSS violation
3. **Missing consent** - GDPR violation
4. **No encryption at rest** - HIPAA/SOC 2 violation
5. **Missing access controls** - SOC 2 violation

### High Priority

Fix before deploying to production:

1. **Weak passwords** - PCI DSS 4.0 violation
2. **Session timeout too long** - PCI DSS violation
3. **Missing audit logs** - SOC 2 violation
4. **No data export** - GDPR violation
5. **No data deletion** - GDPR/CCPA violation

### Medium Priority

Fix when possible:

1. **Missing privacy policy** - GDPR/CCPA warning
2. **Unclear consent flows** - PIPEDA warning
3. **Over-collection** - GDPR data minimization

## After Running

**If critical violations found:**

```bash
# Fix violations immediately
# Re-run compliance check
/compliance-check src/features/patients/

# Run related security skills
/pii-scanner
/security-review
```

**If compliant:**

```bash
# Document compliance status
# Add to monthly audit schedule
# Continue with PR
```

## Tips

- **Review compliance docs first** - Read bundled documentation before implementing
- **Consult legal team** - For complex compliance questions
- **Use this skill early** - Don't wait until PR time
- **Monthly audits** - Run compliance check monthly
- **Educate team** - Share violations and remediation
- **Document decisions** - Keep audit trail of compliance choices

## Related Skills

- `/pii-scanner` - Detailed PII detection in logs
- `/security-review` - General security checklist
- `/secrets-check` - Detect hardcoded secrets
- `/generate-compliance-report` - Generate stakeholder report

## Compliance Documentation

**Review before implementing security-sensitive features:**

1. **HIPAA, CCPA, SOC 2**: `docs/compliance/usa-hipaa-ccpa-soc2.md`
2. **PIPEDA**: `docs/compliance/canada-pipeda.md`
3. **GDPR**: `docs/compliance/europe-gdpr.md`
4. **Logging & Data Protection**: `docs/compliance/logging-data-protection.md`
5. **Gap Analysis & Roadmap**: `docs/compliance/gap-analysis-roadmap.md`
