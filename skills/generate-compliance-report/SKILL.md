---
name: generate-compliance-report
description: Generate comprehensive compliance status report. Use when auditing compliance status.
---

# Generate Compliance Report Skill

## Purpose

Generate comprehensive compliance status report for stakeholders covering HIPAA, GDPR, PCI DSS, PIPEDA, CCPA, and SOC 2.

## When to Use

- Monthly compliance audit
- Before security review
- For compliance team/auditors
- Board reporting
- Vendor assessment

## Usage

```
/generate-compliance-report [format]
```

Examples:

```
/generate-compliance-report              # Generate markdown report
/generate-compliance-report --format=pdf # Generate PDF report
/generate-compliance-report --format=json # Generate JSON data
```

## What It Generates

### Executive Summary

```markdown
# Compliance Status Report

**Generated:** 2026-03-10 14:30 UTC
**Scope:** Full codebase (all packages and apps)
**Reporter:** Claude AI (Automated Compliance Check)

---

## Executive Summary

**Overall Compliance Score:** 95% ✅

The codebase is substantially compliant with all applicable regulatory frameworks. 3 medium-priority issues identified requiring remediation within 30 days.

**Framework Status:**

- ✅ HIPAA (USA - Healthcare): 95% compliant
- ✅ GDPR (Europe): 100% compliant
- ⚠️ PCI DSS (Payment Security): 90% compliant
- ✅ PIPEDA (Canada): 100% compliant
- ✅ CCPA (California): 100% compliant
- ✅ SOC 2 Type II: 98% compliant

**Risk Level:** LOW
**Action Required:** Remediate 3 medium-priority issues

---

## Summary by Framework

### HIPAA (Health Insurance Portability and Accountability Act)

**Status:** 95% Compliant ✅

**Requirements Met:** 19/20
**Violations:** 1 (Medium Priority)

**Compliance Areas:**

- ✅ PHI Encryption (at rest and in transit)
- ✅ Access Controls (RBAC implemented)
- ✅ Audit Logging (comprehensive audit trail)
- ⚠️ Logging Practices (1 issue found)
- ✅ Business Associate Agreements (all vendors covered)
- ✅ Breach Notification Procedures (documented)
- ✅ Patient Rights (access, amendment, accounting)

**Issues Found:**

1. **Medium Priority**
   - **Issue:** Missing audit log for patient data export
   - **Location:** `src/patients/exportData.ts:25`
   - **Impact:** Incomplete audit trail for PHI access
   - **Remediation:** Add audit log entry for export operations
   - **Timeline:** 30 days

**Reference Documentation:**
docs/compliance/usa-hipaa-ccpa-soc2.md

---

### GDPR (General Data Protection Regulation)

**Status:** 100% Compliant ✅

**Requirements Met:** 15/15
**Violations:** 0

**No Issues Found** ✅

**Reference Documentation:**
docs/compliance/europe-gdpr.md

---

### PCI DSS (Payment Card Industry Data Security Standard)

**Status:** 90% Compliant ⚠️

**Requirements Met:** 11/12
**Violations:** 1 (Medium Priority)

**Issues Found:**

1. **Medium Priority**
   - **Issue:** Password minimum length is 10 characters (PCI DSS 4.0 requires 12)
   - **Location:** `src/auth/validatePassword.ts:5`
   - **Impact:** Non-compliance with PCI DSS 4.0 Requirement 8.3.6
   - **Remediation:** Update password validation schema to require 12+ characters
   - **Timeline:** 30 days

**Reference Documentation:**
docs/compliance/usa-hipaa-ccpa-soc2.md

---

### PIPEDA (Personal Information Protection and Electronic Documents Act)

**Status:** 100% Compliant ✅

**Requirements Met:** 10/10
**Violations:** 0

**No Issues Found** ✅

**Reference Documentation:**
docs/compliance/canada-pipeda.md

---

### CCPA (California Consumer Privacy Act)

**Status:** 100% Compliant ✅

**Requirements Met:** 8/8
**Violations:** 0

**No Issues Found** ✅

**Reference Documentation:**
docs/compliance/usa-hipaa-ccpa-soc2.md

---

### SOC 2 Type II

**Status:** 98% Compliant ✅

**Requirements Met:** 23/24
**Violations:** 1 (Medium Priority)

**Issues Found:**

1. **Medium Priority**
   - **Issue:** Incident response playbook not tested in last 6 months
   - **Impact:** CC9.2 (Risk Mitigation) - untested procedures
   - **Remediation:** Schedule incident response tabletop exercise
   - **Timeline:** 30 days

**Reference Documentation:**
docs/compliance/usa-hipaa-ccpa-soc2.md

---

## Detailed Issues

### All Issues (Priority Order)

#### Medium Priority (3 issues) - Remediate within 30 days

1. **HIPAA: Missing Audit Log**
   - Framework: HIPAA
   - Location: `src/patients/exportData.ts:25`
   - Description: Patient data export not logged in audit trail
   - Remediation: Add audit log entry

2. **PCI DSS: Weak Password Requirements**
   - Framework: PCI DSS
   - Location: `src/auth/validatePassword.ts:5`
   - Description: Password minimum length is 10 (should be 12)
   - Remediation: Update validation schema to require 12+ characters

3. **SOC 2: Untested Incident Response**
   - Framework: SOC 2
   - Description: Incident response playbook not tested recently
   - Remediation: Schedule tabletop exercise

---

## Risk Assessment

### Overall Risk Level: LOW ✅

**Risk Factors:**

- ✅ No critical or high-priority violations
- ✅ All frameworks substantially compliant (>90%)
- ⚠️ 3 medium-priority issues require attention
- ✅ Clear remediation plan with timelines

---

## Recommendations

### Immediate Actions (This Month)

1. ✅ Add audit logging for patient data exports (HIPAA)
2. ✅ Update password requirements to 12+ characters (PCI DSS)
3. ✅ Schedule incident response tabletop exercise (SOC 2)

### Short-Term (Next 3 Months)

1. Review and update privacy policies
2. Conduct quarterly GDPR audit
3. Perform PCI DSS 4.0 gap analysis
4. Test data breach notification procedures

### Long-Term (Next 6-12 Months)

1. Implement automated compliance monitoring
2. Achieve SOC 2 Type II certification
3. Review vendor security agreements
4. Update incident response playbook

---

## Compliance Documentation

**Review these resources for detailed requirements:**

1. **HIPAA, CCPA, SOC 2:** docs/compliance/usa-hipaa-ccpa-soc2.md
2. **PIPEDA:** docs/compliance/canada-pipeda.md
3. **GDPR:** docs/compliance/europe-gdpr.md
4. **Logging & Data Protection:** docs/compliance/logging-data-protection.md
5. **Gap Analysis & Roadmap:** docs/compliance/gap-analysis-roadmap.md

---

## Appendix

### Methodology

1. Static code analysis (AST parsing)
2. PII/PHI detection (regex patterns)
3. Security best practices validation
4. Framework-specific rule checks
5. Dependency graph analysis

### Tools Used

- `/compliance-check` - Framework validation
- `/pii-scanner` - PII/PHI detection
- `/secrets-check` - Secret detection
- `/security-review` - Security checklist

---

**Report Generated by:** Claude AI (Automated Compliance Scanner)
**Questions?** Contact your compliance team
```

## Output Format

### Markdown Report (Default)

```
✅ Compliance Report Generated

Created: docs/compliance-report-YYYY-MM-DD.md

Report Sections:
- Executive Summary
- Framework-by-Framework Analysis (6 frameworks)
- Detailed Issues
- Compliance Metrics (trend analysis)
- Risk Assessment
- Recommendations
- Appendix

Overall Compliance: 95% ✅
Critical Issues: 0
High Issues: 0
Medium Issues: 3
Low Issues: 0

Next Steps:
1. Review report
2. Share with stakeholders
3. Track remediation in issue tracker
4. Schedule follow-up scan
```

### JSON Export

```json
{
  "generatedAt": "2026-03-10T14:30:00Z",
  "overallScore": 95,
  "riskLevel": "LOW",
  "frameworks": {
    "hipaa": {
      "status": "COMPLIANT",
      "score": 95,
      "requirementsMet": 19,
      "requirementsTotal": 20,
      "issues": [
        {
          "severity": "MEDIUM",
          "title": "Missing Audit Log",
          "location": "src/patients/exportData.ts:25",
          "remediation": "Add audit log entry"
        }
      ]
    }
  }
}
```

## After Running

### Share with Stakeholders

```bash
# Email to compliance team
open docs/compliance-report-YYYY-MM-DD.pdf
```

### Track Remediation

```bash
# Create GitHub issues for each finding
gh issue create \
  --title "HIPAA: Add audit log for patient data export" \
  --body "See compliance report" \
  --label "compliance,hipaa,medium-priority"
```

## Tips

- **Run monthly** - Regular compliance audits
- **Track trends** - Monitor score over time
- **Remediate quickly** - Fix issues within timelines
- **Document everything** - Keep audit trail
- **Share with team** - Ensure awareness
- **Board reporting** - Use for governance

## Related Skills

- `/compliance-check` - Run compliance validation
- `/pii-scanner` - Detect PII in logs
- `/security-review` - Security checklist
- `/secrets-check` - Secret detection

## Integration with CI/CD

Generate reports automatically:

```yaml
# .github/workflows/compliance-monthly.yml
name: Monthly Compliance Report

on:
  schedule:
    - cron: '0 0 1 * *' # 1st of each month

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Generate Report
        run: |
          npm run generate-compliance-report --format=pdf

      - name: Notify Team
        run: |
          curl -X POST $SLACK_WEBHOOK \
            -d '{"text":"Monthly compliance report generated"}'
```
