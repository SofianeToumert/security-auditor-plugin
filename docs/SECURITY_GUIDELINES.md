# Security & Compliance Guidelines

## Overview

This document provides security implementation guidelines based on HIPAA, GDPR, SOC 2, PIPEDA, PCI DSS 4.0, and CCPA compliance requirements.

**IMPORTANT:** This document must be followed for all code contributions handling user data, authentication, logging, or external integrations.

---

## 1. Compliance Documentation Reference

**CRITICAL:** Before implementing ANY feature that handles user authentication, PII, logging, external integrations, or payment/health data, review the bundled compliance docs:

1. **Compliance Standards - USA** (`docs/compliance/usa-hipaa-ccpa-soc2.md`) - HIPAA, CCPA, SOC 2
2. **Compliance Standards - Canada** (`docs/compliance/canada-pipeda.md`) - PIPEDA
3. **Compliance Standards - Europe** (`docs/compliance/europe-gdpr.md`) - GDPR
4. **Logging & Data Protection Requirements** (`docs/compliance/logging-data-protection.md`)
5. **Compliance Gap Analysis & Remediation Roadmap** (`docs/compliance/gap-analysis-roadmap.md`)

---

## 2. CRITICAL: What NEVER to Log

Based on "Logging & Data Protection Requirements", **NEVER** log the following:

| Data Type                         | Regulations Violated    | Risk Level   |
| --------------------------------- | ----------------------- | ------------ |
| Passwords / Authentication tokens | ALL                     | **Critical** |
| Email addresses (unmasked)        | GDPR, CCPA              | **High**     |
| Phone numbers (unmasked)          | GDPR, CCPA              | **High**     |
| IP addresses (in EU contexts)     | GDPR                    | **High**     |
| Full names                        | GDPR, CCPA              | **High**     |
| Payment card numbers (PAN)        | PCI DSS                 | **Critical** |
| CVV/CVC codes                     | PCI DSS                 | **Critical** |
| Social Security Numbers           | HIPAA, CCPA             | **Critical** |
| Health information / PHI          | HIPAA, GDPR             | **Critical** |
| Biometric data                    | GDPR (special category) | **Critical** |
| Session tokens                    | ALL                     | **Critical** |
| Authorization headers             | ALL                     | **Critical** |
| Request/response bodies           | ALL (may contain PII)   | **Critical** |
| Addresses (full)                  | GDPR, CCPA              | High         |
| Geolocation data (precise)        | GDPR, CCPA              | High         |

---

## 3. Data Masking Requirements

When you MUST log user-related data, use these masking patterns:

| Data Type          | Masking Method                                  | Input Example                     | Output Example    |
| ------------------ | ----------------------------------------------- | --------------------------------- | ----------------- |
| **Email**          | Partial mask (name + domain)                    | `john.doe@example.com`            | `j***@e***.com`   |
| **Phone**          | Last 4 digits only (any format)                 | `+14155551234`                    | `***1234`         |
| **Name**           | Initials + mask                                 | `John Doe`                        | `J. D***`         |
| **Address**        | City/country only                               | `123 Main St, New York, NY 10001` | `New York, US`    |
| **IP Address**     | Mask last 2 octets (IPv4) or prefix only (IPv6) | `192.168.1.100`                   | `192.168.xxx.xxx` |
| **Account Number** | Last 4 digits                                   | `123456789`                       | `****6789`        |
| **Session Token**  | Hash or truncate                                | `eyJhbGci...xyz123`               | `eyJ...xyz`       |

### Example Masking Functions

```typescript
// Email: "john@example.com" -> "j***@e***.com"
function maskEmail(email: string): string {
  const [name, domain] = email.split('@');
  if (!name || !domain) return '[REDACTED_EMAIL]';
  const domainParts = domain.split('.');
  if (domainParts.length < 2) return `${name[0]}***@[REDACTED]`;
  const maskedDomain = domainParts[0][0] + '***.' + domainParts.slice(-1)[0];
  return `${name[0]}***@${maskedDomain}`;
}

// Phone: "+14155551234" -> "***1234"
function maskPhone(phone: string): string {
  const digits = phone.replace(/\D/g, '');
  if (digits.length < 4) return '****';
  return `***${digits.slice(-4)}`;
}

// Name: "John Doe" -> "J. D***"
function maskName(name: string): string {
  const parts = name.trim().split(/\s+/);
  if (parts.length === 0 || !parts[0]) return '[REDACTED_NAME]';
  if (parts.length === 1) return `${parts[0][0]}***`;
  return `${parts[0][0]}. ${parts[parts.length - 1][0]}***`;
}

// IP: "192.168.1.100" -> "192.168.xxx.xxx"
function maskIP(ip: string): string {
  if (ip.includes(':')) {
    const firstSegment = ip.split(':')[0];
    return `${firstSegment}:****`;
  }
  const parts = ip.split('.');
  if (parts.length !== 4) return '[REDACTED_IP]';
  return `${parts[0]}.${parts[1]}.xxx.xxx`;
}

// Account/Card: "123456789" -> "****6789"
function maskAccountNumber(account: string): string {
  const digits = account.replace(/\D/g, '');
  if (digits.length <= 4) return '****';
  return `${'*'.repeat(digits.length - 4)}${digits.slice(-4)}`;
}
```

---

## 4. Code Review Checklist for Pull Requests

**MANDATORY Security Checks Before Merge:**

- [ ] **Input Validation**: All user input validated with zod schemas
- [ ] **PII Protection**: NO PII in logs, errors, or console output
- [ ] **Secrets Management**: All secrets in environment variables (NEVER hardcoded)
- [ ] **HTTPS Enforcement**: Production URLs use HTTPS only (no `http://`)
- [ ] **Error Handling**: User-friendly messages only (no stack traces or technical details exposed)
- [ ] **Password Policy**: 12-character minimum with complexity (PCI DSS 4.0 requirement)
- [ ] **Session Security**: Tokens in httpOnly cookies or secure storage
- [ ] **Security Headers**: CSP, HSTS, X-Frame-Options configured (when applicable)
- [ ] **No Secrets in Git**: Run `gitleaks` before committing

---

## 5. Implementation Requirements by Feature

### Input Validation

**When to Implement:** Any feature accepting user input

**Requirements:**

- Use **zod** for ALL user input validation
- Create schemas alongside your models

**Validation Patterns:**

```typescript
import { z } from 'zod';

// Email validation
const emailSchema = z.string().email().toLowerCase().trim();

// Phone validation (E.164 format)
const phoneSchema = z.string().regex(/^\+[1-9]\d{1,14}$/);

// Safe string (XSS prevention)
const safeStringSchema = z.string().trim().max(1000);

// Password (12+ chars, complexity)
const passwordSchema = z
  .string()
  .min(12, 'Password must be at least 12 characters')
  .regex(/[a-z]/, 'Password must contain lowercase')
  .regex(/[A-Z]/, 'Password must contain uppercase')
  .regex(/[0-9]/, 'Password must contain number')
  .regex(/[^a-zA-Z0-9]/, 'Password must contain special character');

// HTTPS-only URLs (production)
const secureUrlSchema = z
  .string()
  .url()
  .refine(
    (url) => process.env.NODE_ENV !== 'production' || url.startsWith('https://'),
    'Production URLs must use HTTPS'
  );
```

### Environment Configuration

**When to Implement:** Any feature using environment variables

**Requirements:**

- Type-safe env vars with zod validation
- Fail fast at startup if required vars missing
- **NEVER** use `process.env` directly in business logic

### Logging

**When to Implement:** Any feature that logs events

**Requirements:**

- Use structured JSON logging
- PII redaction (see "What NEVER to Log")
- Audit trail: user ID (NOT email), action, timestamp

**Audit Trail Format:**

| Field       | Required By | Purpose               | Example                    |
| ----------- | ----------- | --------------------- | -------------------------- |
| **Who**     | ALL         | User ID (NEVER email) | `user_abc123`              |
| **What**    | ALL         | Action type           | `profile_update`           |
| **When**    | ALL         | ISO 8601 timestamp    | `2025-01-07T12:00:00.000Z` |
| **Where**   | SOC 2, PCI  | Source system         | `web-app`                  |
| **Outcome** | ALL         | Success/failure       | `SUCCESS`                  |

### HTTP Client

**When to Implement:** Any feature making API calls

**Requirements:**

1. **HTTPS Enforcement** - Production URLs MUST use HTTPS
2. **Request/Response Logging** - Log method, URL, status code. **NEVER** log headers or bodies.
3. **Token Storage** - Web: httpOnly cookies. Mobile: Keychain/Keystore. **NEVER**: localStorage.

### Authentication

**Requirements:**

| Requirement         | Specification                                                   |
| ------------------- | --------------------------------------------------------------- |
| **Password Policy** | 12+ characters, uppercase, lowercase, number, special character |
| **Token Storage**   | httpOnly cookies (web) or secure keychain/keystore (mobile)     |
| **Session Timeout** | 15 minutes idle (PCI DSS requirement)                           |
| **MFA**             | Required for admin users (HIPAA, PCI DSS, ISO 27001)            |
| **Account Lockout** | After 5 failed attempts                                         |
| **Token Refresh**   | Automatic before expiration                                     |

---

## 6. Breach Notification Timelines

If a security incident occurs, these are the **legal deadlines**:

| Regulation        | Timeline                       | Notify Whom                                                 |
| ----------------- | ------------------------------ | ----------------------------------------------------------- |
| **GDPR**          | 72 hours                       | Supervisory authority + affected individuals (if high risk) |
| **CCPA/CPRA**     | "Most expedient time possible" | Affected consumers                                          |
| **PIPEDA**        | "As soon as feasible"          | Privacy Commissioner + affected individuals                 |
| **Quebec Law 25** | 72 hours                       | CAI + affected individuals                                  |
| **HIPAA**         | 60 days                        | HHS + affected individuals + media (if 500+)                |
| **PCI DSS**       | Immediately                    | Card brands + acquiring bank                                |

---

## 7. Incident Response: Secrets Committed to Git

If you accidentally commit secrets:

### Immediate Actions

1. **IMMEDIATELY** rotate all exposed credentials
2. Remove secrets from git history:

   ```bash
   # Use git-filter-repo (recommended)
   git filter-repo --path .env --invert-paths

   # Or use BFG Repo-Cleaner
   bfg --delete-files .env
   ```

3. Force push to remove from remote
4. Audit all access logs for exposed credentials
5. Document the incident (compliance requirement)
6. Review if breach notification is required (see table above)

### Prevention

- Pre-commit hooks (gitleaks) will catch most secrets
- Always use `.env` files (git-ignored)
- Never hardcode credentials
- Use secrets managers (e.g., Vault, AWS Secrets Manager)

---

## 8. Compliance Quick Reference

### Key Requirements by Standard

| Standard        | Key Frontend Requirements                                                        |
| --------------- | -------------------------------------------------------------------------------- |
| **HIPAA**       | Encryption, audit logging, access controls, 24-hour incident notification        |
| **PCI DSS 4.0** | 12-char passwords, MFA, 15-min timeout, no PAN in logs                           |
| **GDPR**        | 72-hour breach notification, data minimization, right to erasure, PII protection |
| **PIPEDA**      | Consent, safeguards, openness, individual access                                 |
| **SOC 2**       | Security controls, monitoring, incident response                                 |

### Common Violations to Avoid

1. Logging PII (emails, phones, names)
2. Weak passwords (less than 12 characters)
3. No session timeout
4. No MFA for admin users
5. Hardcoded secrets
6. HTTP instead of HTTPS in production
7. Exposing technical errors to users
8. No input validation (XSS/injection vulnerabilities)

---

## Summary

This document is the **authoritative security checklist** for codebases using this plugin. Every PR must be reviewed against these guidelines before merge.

**Remember:**

- Security is NOT optional - it's a legal requirement
- When in doubt, ask for a compliance review
- Better to over-protect than under-protect user data
