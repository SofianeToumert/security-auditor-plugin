# Logging & Data Protection Requirements Summary

## Overview

This document provides a summary of logging and data protection requirements across all major compliance standards for USA, Canada, and Europe.

---

## 1. What Must NEVER Be Logged

| Data Type | Regulations Prohibiting | Risk Level |
| --- | --- | --- |
| Passwords / Authentication tokens | ALL (HIPAA, PCI DSS, GDPR, etc.) | Critical |
| Full payment card numbers (PAN) | PCI DSS | Critical |
| CVV/CVC codes | PCI DSS | Critical |
| Full Social Security Numbers | HIPAA, CCPA, GDPR | Critical |
| Government ID numbers | HIPAA, GDPR, PIPEDA | Critical |
| Full email addresses (unmasked) | GDPR, CCPA (if identifiable) | High |
| Full phone numbers (unmasked) | GDPR, CCPA (if identifiable) | High |
| IP addresses (EU contexts) | GDPR (considered PII) | High |
| Biometric data | GDPR (special category), BIPA, Quebec Law 25 | Critical |
| Health information / PHI | HIPAA, PHIPA, GDPR | Critical |
| Genetic data | GDPR (special category) | Critical |
| Racial/ethnic origin | GDPR (special category) | Critical |
| Sexual orientation | GDPR (special category) | Critical |
| Religious beliefs | GDPR (special category) | Critical |
| Political opinions | GDPR (special category) | Critical |
| Trade union membership | GDPR (special category) | Critical |
| Children's data (under 13/16) | COPPA, GDPR | Critical |
| Financial account numbers | GLBA, PCI DSS | Critical |
| Driver's license numbers | State privacy laws | High |
| Geolocation data (precise) | GDPR, CCPA | High |

---

## 2. Data That Should Be Masked/Tokenized in Logs

| Data Type | Masking Approach | Example |
| --- | --- | --- |
| Email addresses | Partial mask | `j***n@example.com` |
| Phone numbers | Last 4 digits only | `***-***-1234` |
| Names | Initials or partial | `J. D***` |
| Addresses | City/country only | `***, ***, New York, US` |
| Account numbers | Last 4 digits | `****1234` |
| IP addresses | Truncate last octet | `192.168.1.xxx` |
| User IDs | Use internal IDs | `user_abc123` instead of PII |
| Session tokens | Hash or truncate | `abc...xyz` |

---

## 3. Logging Requirements by Standard

### HIPAA (Healthcare - USA)

| Requirement | Details |
| --- | --- |
| Audit Controls | Hardware, software, procedural mechanisms to record and examine access |
| Access Logs | Log all access to ePHI |
| Modification Logs | Track all changes to ePHI |
| Activity Review | Regular review of audit logs |
| Retention | Minimum 6 years for policies; logs as needed for compliance |
| Integrity | Protect logs from alteration |
| Authentication Logging | Log authentication attempts |

**Specific 2025 Requirements:**

- Vulnerability scans at least twice annually
- Annual penetration testing
- 24-hour business associate incident notification

---

### PCI DSS 4.0 (Payment Cards - Global)

| Requirement | Details |
| --- | --- |
| User Identification | Log individual user access |
| CDE Access | Log all access to cardholder data environment |
| Time Synchronization | NTP for accurate timestamps |
| Retention | Minimum 1 year; 3 months immediately available |
| SIEM Required | Automated log review tools |
| Log Integrity | Detect unauthorized changes |
| Daily Review | Review logs at least daily |
| Alert Response | Respond to security alerts |

**Requirement 10 Details:**

| 10.x | Requirement |
| --- | --- |
| 10.1 | Audit trail policies documented |
| 10.2 | Automated audit trails for all system components |
| 10.3 | Record specific audit trail entries |
| 10.4 | Time synchronization technology |
| 10.5 | Secure audit trails |
| 10.6 | Review logs and security events |
| 10.7 | Retain audit trail history |

---

### GDPR (Data Protection - Europe)

| Requirement | Details |
| --- | --- |
| Purpose Limitation | Logs only for specified purposes |
| Data Minimization | Log only necessary data |
| Access Controls | Restrict who can view logs |
| Deletion Capability | Ability to delete from logs (right to erasure) |
| Lawful Basis | Must have legal basis for logging |
| Transparency | Inform users about logging |
| Security | Appropriate protection of log data |

**Key Considerations:**

- IP addresses are personal data
- Logs containing personal data have retention limits
- Must be able to respond to data subject requests
- Cross-border log transfers require safeguards

---

### SOC 2 (Security Controls - USA/Global)

| Requirement | Details |
| --- | --- |
| Sufficient Logging | Enable detection, monitoring, investigation |
| CC7.2 | Monitor system components for anomalies |
| CC7.3 | Evaluate security events |
| CC7.4 | Respond to identified security incidents |
| Retention | Based on organizational policies |
| Access | Restricted to authorized personnel |

---

### ISO 27001:2022 (Information Security - Global)

| Control | Requirement |
| --- | --- |
| 8.15 | Logging - produce, store, protect, analyze event logs |
| 8.16 | Monitoring activities - detect anomalous behavior |
| 8.17 | Clock synchronization - consistent timestamps |
| 5.25 | Assessment of information security events |
| 5.26 | Response to information security incidents |

---

### NIS2 (Cybersecurity - Europe)

| Requirement | Details |
| --- | --- |
| Detection | Systems to detect incidents |
| Monitoring | Continuous monitoring of networks and systems |
| Incident Records | Document security incidents |
| Evidence Preservation | Retain evidence for investigation |
| Reporting | 24-hour early warning, 72-hour notification |

---

### PIPEDA (Privacy - Canada)

| Requirement | Details |
| --- | --- |
| Safeguard 7 | Security safeguards appropriate to sensitivity |
| Access Records | Track who accesses personal information |
| Breach Records | Maintain records of breaches for 24 months |
| Limited Purpose | Use logs only for security purposes |

---

### Quebec Law 25 (Privacy - Quebec)

| Requirement | Details |
| --- | --- |
| Security Measures | Appropriate safeguards for data |
| Breach Records | Document all breaches |
| Access Logs | Track access to personal information |
| 72-Hour Notification | Rapid breach reporting |

---

## 4. Breach Notification Timelines

| Regulation | Timeline | Notify Whom |
| --- | --- | --- |
| GDPR | 72 hours (proposed: 96 hours) | Supervisory authority, affected individuals (if high risk) |
| HIPAA | 60 days | HHS, affected individuals, media (if 500+) |
| CCPA/CPRA | "Most expedient time possible" | Affected consumers |
| PIPEDA | "As soon as feasible" | Privacy Commissioner, affected individuals |
| Quebec Law 25 | 72 hours | CAI, affected individuals |
| NIS2 | 24 hours (early warning), 72 hours (notification) | Competent authority |
| DORA | 4 hours (initial), 72 hours (intermediate) | Competent authority |
| PCI DSS | Immediately | Card brands, acquiring bank |
| Ontario PHIPA | "At first reasonable opportunity" | IPC, affected individuals |
| Alberta PIPA | Without unreasonable delay | Commissioner |

---

## 5. Log Retention Requirements

| Regulation | Minimum Retention | Notes |
| --- | --- | --- |
| PCI DSS | 1 year (3 months online) | For cardholder data environment |
| HIPAA | 6 years (policies) | Logs as needed for compliance |
| SOC 2 | Per organizational policy | Document and follow consistently |
| GDPR | As short as necessary | Must justify retention period |
| PIPEDA | Only as long as needed | For identified purposes |
| NIS2 | Sufficient for investigation | Not specifically defined |
| DORA | 5 years (incidents) | For financial sector |

---

## 6. Technical Implementation Guidelines

### Logging Infrastructure Requirements

| Component | Requirement | Standards |
| --- | --- | --- |
| Time Synchronization | NTP to authoritative source | PCI DSS 10.4, ISO 8.17 |
| Log Integrity | Tamper-evident storage | PCI DSS 10.5 |
| Centralized Logging | Aggregate from all sources | SOC 2, ISO 8.15 |
| Encryption | Protect logs in transit and at rest | GDPR Art. 32, HIPAA |
| Access Control | Role-based log access | All standards |
| Automated Analysis | SIEM or equivalent | PCI DSS 10.6 |
| Backup | Regular log backups | ISO 8.13, PCI DSS |

### What to Log (Minimum)

| Event Type | Details to Capture |
| --- | --- |
| Authentication | User ID, timestamp, success/failure, source IP (masked for GDPR) |
| Authorization | Resource accessed, action attempted, result |
| Data Access | Who accessed what data, when |
| System Changes | Configuration changes, who, when |
| Security Events | Alerts, incidents, responses |
| Administrative Actions | Privileged operations |
| Errors | System errors, failures |

### Log Format Standards

| Field | Description | Example |
| --- | --- | --- |
| Timestamp | ISO 8601 format | `2025-01-01T12:00:00.000Z` |
| Event Type | Categorized event | `AUTH_SUCCESS`, `DATA_ACCESS` |
| User ID | Internal identifier | `user_abc123` |
| Source | System/service | `api-gateway` |
| Action | What was done | `READ`, `UPDATE`, `DELETE` |
| Resource | What was accessed | `user_profile` |
| Result | Outcome | `SUCCESS`, `FAILURE`, `DENIED` |
| Context | Additional info | `{ "request_id": "..." }` |

---

## 7. PII Handling in Logs - Code Examples

### Data Classification

| Classification | Examples | Log Treatment |
| --- | --- | --- |
| Critical | Passwords, payment cards, SSN | NEVER log |
| High | Email, phone, full name | Mask/tokenize |
| Medium | User preferences, settings | Log with caution |
| Low | Public information | Can log |

### Masking Functions (Pseudocode)

```
maskEmail(email):
  parts = email.split('@')
  name = parts[0]
  domain = parts[1]
  masked = name[0] + '***' + '@' + domain
  return masked
  // "john@example.com" -> "j***@example.com"

maskPhone(phone):
  digits = extractDigits(phone)
  return "***-***-" + digits[-4:]
  // "555-123-4567" -> "***-***-4567"

maskName(name):
  parts = name.split(' ')
  return parts[0][0] + '. ' + parts[-1][0] + '***'
  // "John Doe" -> "J. D***"

maskPAN(cardNumber):
  return "****-****-****-" + cardNumber[-4:]
  // "4111111111111111" -> "****-****-****-1111"
```

### Audit Trail Requirements

| Field | Required By | Purpose |
| --- | --- | --- |
| Who | All | Identify the actor |
| What | All | Describe the action |
| When | All | Timestamp of action |
| Where | SOC 2, PCI | Source system/location |
| Why | GDPR, HIPAA | Legal basis/purpose |
| Outcome | All | Result of action |

---

## 8. Compliance Checklist for Logging

### Pre-Implementation

- [ ] Identify all regulations applicable to your organization
- [ ] Map data elements to classification levels
- [ ] Define retention periods per regulation
- [ ] Document logging purposes and legal bases
- [ ] Design log architecture with security in mind

### Implementation

- [ ] Implement time synchronization (NTP)
- [ ] Deploy centralized logging (SIEM)
- [ ] Configure log integrity protection
- [ ] Implement access controls for logs
- [ ] Create PII masking/tokenization utilities
- [ ] Set up automated alerting
- [ ] Configure encryption for log data

### Operations

- [ ] Daily log review process
- [ ] Regular security event analysis
- [ ] Incident response procedures using logs
- [ ] Log retention and archival procedures
- [ ] Regular access reviews for log systems

### Compliance

- [ ] Document logging policies
- [ ] Train staff on logging requirements
- [ ] Regular audits of logging compliance
- [ ] Update procedures for regulation changes
- [ ] Maintain evidence for audits

---

## Sources

### USA

- [HHS HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html)
- [PCI Security Standards Council](https://www.pcisecuritystandards.org/document_library/)
- [AICPA SOC 2](https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2)

### Canada

- [OPC PIPEDA](https://www.priv.gc.ca/en/privacy-topics/privacy-laws-in-canada/the-personal-information-protection-and-electronic-documents-act-pipeda/)
- [Quebec Law 25](https://www.cai.gouv.qc.ca/citoyens/loi-25/)

### Europe

- [GDPR Official Text](https://gdpr-info.eu/)
- [NIS2 Directive](https://eur-lex.europa.eu/eli/dir/2022/2555)
- [DORA Regulation](https://eur-lex.europa.eu/eli/reg/2022/2554)
- [ISO 27001 Annex A](https://www.isms.online/iso-27001/annex-a-2022/)
