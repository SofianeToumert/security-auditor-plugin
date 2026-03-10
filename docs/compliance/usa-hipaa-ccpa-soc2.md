# Security & Privacy Compliance Standards - USA

## Overview

This document provides a comprehensive listing of all security and privacy compliance standards and certifications relevant for the United States.

---

## 1. Federal Privacy & Security Laws

### HIPAA (Health Insurance Portability and Accountability Act)

**Applicability:** Healthcare providers, health plans, healthcare clearinghouses, and their business associates handling Protected Health Information (PHI)

**Three Categories of Safeguards:**

#### Administrative Safeguards

| Requirement | Description |
| --- | --- |
| Security Management Process | Risk analysis, risk management, sanction policy, information system activity review |
| Assigned Security Responsibility | Designate a security official |
| Workforce Security | Authorization/supervision, clearance procedures, termination procedures |
| Information Access Management | Access authorization, access establishment/modification |
| Security Awareness Training | Security reminders, malware protection, login monitoring, password management |
| Security Incident Procedures | Response and reporting |
| Contingency Plan | Data backup, disaster recovery, emergency mode operation, testing |
| Evaluation | Periodic technical and non-technical evaluation |
| Business Associate Contracts | Written agreements with all business associates |

#### Physical Safeguards

| Requirement | Description |
| --- | --- |
| Facility Access Controls | Contingency operations, facility security plan, access control, maintenance records |
| Workstation Use | Policies for appropriate workstation use |
| Workstation Security | Physical safeguards for workstations |
| Device and Media Controls | Disposal, media re-use, accountability, data backup |

#### Technical Safeguards

| Requirement | Description |
| --- | --- |
| Access Control | Unique user identification, emergency access procedure, automatic logoff, encryption/decryption |
| Audit Controls | Hardware, software, procedural mechanisms to record and examine access |
| Integrity | Mechanism to authenticate ePHI, protect from improper alteration/destruction |
| Person or Entity Authentication | Verify identity of persons seeking access |
| Transmission Security | Integrity controls, encryption for transmission |

#### 2025 HIPAA Security Rule Updates (Proposed)

- Encryption of ePHI at rest AND in transit (mandatory, limited exceptions)
- Business associates must notify covered entities within **24 hours** of contingency plan activation
- Annual compliance audits required
- Vulnerability scans **at least twice annually**
- Annual penetration testing
- Complete technology asset inventory
- Network segmentation required
- Written verification from business associates every **12 months**

**Penalties:** Up to $1.5 million per violation category per year; criminal penalties up to $250,000 and 10 years imprisonment

---

### PCI DSS 4.0 (Payment Card Industry Data Security Standard)

**Applicability:** Any organization that stores, processes, or transmits cardholder data

**The 12 Requirements:**

#### Goal 1: Build and Maintain a Secure Network and Systems

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 1 | Install and Maintain Network Security Controls | Firewall configuration, network segmentation, NSC management, restrict CDE access |
| 2 | Apply Secure Configurations to All System Components | Change vendor defaults, harden all systems, document configuration standards |

#### Goal 2: Protect Account Data

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 3 | Protect Stored Account Data | Minimize data storage, mask PAN display, encrypt stored data, key management |
| 4 | Protect Cardholder Data with Strong Cryptography During Transmission | TLS 1.2+, never send PAN via insecure channels, strong cryptographic protocols |

#### Goal 3: Maintain a Vulnerability Management Program

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 5 | Protect All Systems from Malicious Software | Anti-malware on all systems, regular updates, protection against phishing |
| 6 | Develop and Maintain Secure Systems and Software | Secure development practices, patch management, WAF for public-facing apps, script integrity |

#### Goal 4: Implement Strong Access Control Measures

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 7 | Restrict Access by Business Need to Know | Role-based access, least privilege, access reviews |
| 8 | Identify Users and Authenticate Access | Unique IDs, MFA required for ALL CDE access, 12-character passwords, no hardcoded passwords |
| 9 | Restrict Physical Access to Cardholder Data | Facility controls, visitor management, media protection |

#### Goal 5: Regularly Monitor and Test Networks

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 10 | Log and Monitor All Access | Audit trails, automated log reviews (SIEM), log integrity, time synchronization |
| 11 | Test Security Systems and Processes | Quarterly vulnerability scans, annual penetration testing, authenticated internal scans |

#### Goal 6: Maintain an Information Security Policy

| Req | Requirement | Key Controls |
| --- | --- | --- |
| 12 | Support Information Security with Policies and Programs | Security policy, risk assessment, awareness training, incident response plan |

#### PCI DSS 4.0 Key 2025 Changes (Mandatory as of March 31, 2025)

- MFA for ALL access to CDE (not just remote)
- 12-character minimum passwords
- Authenticated internal vulnerability scans
- Client-side script management (6.4.3)
- Change detection for payment pages (11.6.1)
- Targeted risk analysis for flexible requirements
- Annual scope documentation and confirmation
- Enhanced incident response for PAN found in unexpected locations

**Compliance Levels:**

| Level | Criteria | Requirements |
| --- | --- | --- |
| 1 | >6M transactions/year | Annual on-site audit by QSA, quarterly network scans |
| 2 | 1-6M transactions/year | Annual SAQ, quarterly network scans |
| 3 | 20K-1M transactions/year | Annual SAQ, quarterly network scans |
| 4 | <20K transactions/year | Annual SAQ, quarterly network scans |

---

### GLBA (Gramm-Leach-Bliley Act)

**Applicability:** Financial institutions (banks, securities firms, insurance companies, loan providers, financial advisors)

**Key Requirements:**

| Component | Requirements |
| --- | --- |
| Financial Privacy Rule | Privacy notices explaining data collection/sharing practices |
| Safeguards Rule | Written information security plan, designate coordinator, risk assessment, oversee service providers, evaluate/adjust program |
| Pretexting Protection | Protection against fraudulent access to personal financial information |

**Penalties:** Up to $100,000 per violation for institutions; up to $10,000 per violation for individuals; criminal penalties up to 5 years imprisonment

---

### FERPA (Family Educational Rights and Privacy Act)

**Applicability:** Educational institutions receiving federal funding

**Key Requirements:**

- Protect student education records
- Parental/student consent for disclosure
- Right to access and amend records
- Annual notification of rights

**Penalties:** Withdrawal of federal funding

---

### COPPA (Children's Online Privacy Protection Act)

**Applicability:** Websites and online services directed at children under 13, or that knowingly collect data from children under 13

**Key Requirements:**

- Clear privacy policy
- Verifiable parental consent before collection
- Parental access to child's information
- Ability to delete child's data
- Limit data collection to what's necessary

**2025 Updates:**

- Disclose identities and categories of third parties receiving children's data
- Enhanced consent mechanisms

**Penalties:** Up to $50,120 per violation (2024 adjustment)

---

## 2. State Privacy Laws (Comprehensive List)

**20 States with Comprehensive Privacy Laws as of 2025:**

| State | Law | Effective Date | Key Thresholds |
| --- | --- | --- | --- |
| California | CCPA/CPRA | Jan 1, 2020 / Jan 1, 2023 | $26.6M revenue OR 100K+ consumers OR 50%+ revenue from selling data |
| Virginia | VCDPA | Jan 1, 2023 | 100K consumers OR 25K consumers + 50% revenue from data sales |
| Colorado | CPA | Jul 1, 2023 | 100K consumers OR 25K consumers + revenue from data sales |
| Connecticut | CTDPA | Jul 1, 2023 | 100K consumers OR 25K consumers + 25%+ revenue from data sales |
| Utah | UCPA | Dec 31, 2023 | $25M revenue AND 100K consumers OR 50% revenue from data sales |
| Iowa | ICDPA | Jan 1, 2025 | 100K consumers OR 25K consumers + 50%+ revenue from data sales |
| Delaware | DPDPA | Jan 1, 2025 | 35K consumers OR 10K consumers + 20%+ revenue from data sales |
| Nebraska | NDPA | Jan 1, 2025 | No revenue threshold; activity-based triggers |
| New Hampshire | NHPA | Jan 1, 2025 | 35K consumers OR 10K consumers + 25%+ revenue from data sales |
| New Jersey | NJDPA | Jan 15, 2025 | 100K consumers OR 25K consumers + revenue from data sales |
| Tennessee | TIPA | Jul 1, 2025 | $25M revenue AND 175K consumers OR 25K consumers + 50%+ revenue |
| Minnesota | MCDPA | Jul 31, 2025 | 100K consumers OR 25K consumers + 25%+ revenue from data sales |
| Maryland | MODPA | Oct 1, 2025 | 35K consumers OR 10K consumers + 20%+ revenue from data sales |
| Indiana | INCDPA | Jan 1, 2026 | 100K consumers OR 25K consumers + 50%+ revenue from data sales |
| Kentucky | KCDPA | Jan 1, 2026 | 100K consumers OR 25K consumers + 50%+ revenue from data sales |
| Rhode Island | RIDPA | Jan 1, 2026 | 35K consumers OR 10K consumers + 20%+ revenue from data sales |
| Montana | MCDPA | Oct 1, 2024 | 50K consumers (excludes payment transactions) |
| Oregon | OCPA | Jul 1, 2024 | 100K consumers OR 25K consumers + 25%+ revenue from data sales |
| Texas | TDPSA | Jul 1, 2024 | Any business conducting in Texas (no threshold) |
| Florida | FDBR | Jul 1, 2024 | $1B revenue AND specific data-related activities |

### Common State Privacy Law Requirements

| Right/Requirement | Description |
| --- | --- |
| Right to Know | Consumers can request what personal data is collected |
| Right to Delete | Consumers can request deletion of their data |
| Right to Correct | Consumers can correct inaccurate data |
| Right to Portability | Obtain data in portable format |
| Right to Opt-Out | Opt out of sale/sharing of personal data |
| Opt-Out of Profiling | Opt out of automated decision-making |
| Non-Discrimination | Cannot discriminate against consumers exercising rights |
| Privacy Notice | Clear disclosure of data practices |
| Data Protection Assessments | Required for high-risk processing |

### California CCPA/CPRA Specific Requirements

| Requirement | Details |
| --- | --- |
| Privacy Policy | Comprehensive, updated annually |
| "Do Not Sell/Share" Link | Prominent on website |
| Opt-Out Preference Signals | Honor Global Privacy Control |
| Data Minimization | Collect only necessary data |
| Purpose Limitation | Use only for disclosed purposes |
| Contracts with Service Providers | Written agreements required |
| Employee Data Rights | Full CCPA rights for employees |
| Risk Assessments | For high-risk processing |
| Cybersecurity Audits | For high-risk businesses |

**CCPA Penalties (2025 Adjusted):**

- Unintentional violations: Up to $2,663 per violation
- Intentional violations: Up to $7,988 per violation
- Violations involving minors: Up to $7,988 per violation
- Data breach damages: $107 to $799 per affected consumer

---

## 3. Security Certifications (USA/Global)

### SOC 2 (Service Organization Control 2)

**Applicability:** Service organizations that store, process, or transmit customer data

**Types:**

| Type | Description | Duration |
| --- | --- | --- |
| Type I | Point-in-time evaluation of control design | Single date |
| Type II | Operational effectiveness over time | 3-12 months (minimum 3) |

**Five Trust Services Criteria:**

#### 1. Security (Common Criteria - REQUIRED)

| Category | Controls |
| --- | --- |
| CC1 | Control Environment |
| CC2 | Communication and Information |
| CC3 | Risk Assessment |
| CC4 | Monitoring Activities |
| CC5 | Control Activities |
| CC6 | Logical and Physical Access Controls |
| CC7 | System Operations |
| CC8 | Change Management |
| CC9 | Risk Mitigation |

#### 2. Availability (Optional)

| Focus | Requirements |
| --- | --- |
| A1 | System availability commitments and performance monitoring |
| Uptime | Disaster recovery, failover, redundancy |
| Monitoring | Continuous availability monitoring |

#### 3. Processing Integrity (Optional)

| Focus | Requirements |
| --- | --- |
| PI1 | System processing is complete, valid, accurate, timely, and authorized |
| Transaction | Processing meets specifications |
| Error handling | Detection and correction mechanisms |

#### 4. Confidentiality (Optional)

| Focus | Requirements |
| --- | --- |
| C1 | Confidential information protection |
| Classification | Data classification policies |
| Access | Restricted to authorized personnel |
| Encryption | Confidential data encrypted |

#### 5. Privacy (Optional)

| Focus | Requirements |
| --- | --- |
| P1-P8 | Notice, choice, collection, use, access, disclosure, quality, monitoring |
| Consent | Privacy consent mechanisms |
| Data Subject Rights | Access, correction, deletion |

**SOC 2+ (Combined Reports):**
Can combine SOC 2 with: HIPAA, GDPR, NIST CSF, ISO 27001, PCI DSS

---

### NIST Cybersecurity Framework (CSF)

**Applicability:** Mandatory for US federal agencies and contractors; voluntary for private sector

**Five Core Functions:**

| Function | Categories |
| --- | --- |
| IDENTIFY | Asset Management, Business Environment, Governance, Risk Assessment, Risk Management Strategy, Supply Chain Risk Management |
| PROTECT | Access Control, Awareness and Training, Data Security, Information Protection, Maintenance, Protective Technology |
| DETECT | Anomalies and Events, Security Continuous Monitoring, Detection Processes |
| RESPOND | Response Planning, Communications, Analysis, Mitigation, Improvements |
| RECOVER | Recovery Planning, Improvements, Communications |

**Implementation Tiers:**

| Tier | Description |
| --- | --- |
| Tier 1 | Partial - Ad hoc, reactive |
| Tier 2 | Risk Informed - Approved but not organization-wide |
| Tier 3 | Repeatable - Formal, organization-wide policies |
| Tier 4 | Adaptive - Continuous improvement, real-time response |

---

### NIST 800-53 (Security and Privacy Controls)

**Applicability:** Federal information systems; reference for private sector

**20 Control Families:**

| ID | Family | ID | Family |
| --- | --- | --- | --- |
| AC | Access Control | PE | Physical and Environmental Protection |
| AT | Awareness and Training | PL | Planning |
| AU | Audit and Accountability | PM | Program Management |
| CA | Assessment, Authorization, and Monitoring | PS | Personnel Security |
| CM | Configuration Management | PT | PII Processing and Transparency |
| CP | Contingency Planning | RA | Risk Assessment |
| IA | Identification and Authentication | SA | System and Services Acquisition |
| IR | Incident Response | SC | System and Communications Protection |
| MA | Maintenance | SI | System and Information Integrity |
| MP | Media Protection | SR | Supply Chain Risk Management |

---

### FedRAMP (Federal Risk and Authorization Management Program)

**Applicability:** Cloud service providers serving federal agencies

**Impact Levels:**

| Level | Data Types | Controls |
| --- | --- | --- |
| Low | Publicly releasable | ~156 controls |
| Moderate | Controlled unclassified | ~325 controls |
| High | High-value assets, law enforcement, emergency services | ~421 controls |

---

### CMMC (Cybersecurity Maturity Model Certification)

**Applicability:** Department of Defense contractors

**Levels:**

| Level | Description | Controls |
| --- | --- | --- |
| Level 1 | Foundational | 17 practices (basic safeguarding) |
| Level 2 | Advanced | 110 practices (NIST 800-171) |
| Level 3 | Expert | 110+ practices (NIST 800-172) |

---

## Sources

- [IAPP US State Privacy Tracker](https://iapp.org/resources/article/us-state-privacy-legislation-tracker/)
- [HHS HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html)
- [PCI Security Standards Council](https://www.pcisecuritystandards.org/document_library/)
- [AICPA SOC 2](https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
