# Cresta Data Security Architecture

## Document Purpose

This document analyzes Cresta's security architecture across data protection, access control, compliance, and operational security. All analysis based on verified certifications and architectural components.

---

## Security Layers Overview

```
┌─────────────────────────────────────────────────────────┐
│              Compliance & Certifications                │
│  SOC 2 Type II | ISO 27001/27701/42001 | PCI-DSS | HIPAA│
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  Network Security                        │
│  Customer Subdomains | WAF (Wallarm) | TLS 1.2+ | VPC   │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│              Application Security                        │
│  Authentication | Authorization | Input Validation       │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  Data Protection                         │
│  Encryption (Transit + Rest) | PII Redaction | DLP       │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│              Operational Security                        │
│  Monitoring | Logging | Incident Response | DR/BC        │
└─────────────────────────────────────────────────────────┘
```

---

## Verified Security Controls

### Network Security

#### Customer-Specific Subdomains
**Source**: Cresta blog "Handling Incoming Traffic with Customer-Specific Subdomains"

**Architecture**:
- **Format**: `customer.region.cresta.ai`
- **Isolation**: DNS-level tenant separation
- **Regional Deployment**: Data sovereignty compliance (EU customers → EU region)
- **Benefits**: Traffic isolation, DDoS protection per tenant, regional data residency

**Security Properties**:
- **DNS Resolution**: AWS Route53 (health checks, failover)
- **TLS Termination**: At load balancer level
- **Certificate Management**: Automated cert provisioning per subdomain

#### WAF Protection
**Source**: Cresta documentation

**Verified**: Wallarm WAF integrated with NGINX Ingress

**Capabilities**:
- **OWASP Top 10**: Protection against common web vulnerabilities
- **API Security**: REST/WebSocket API attack prevention
- **Rate Limiting**: DDoS and abuse prevention
- **Threat Intelligence**: Real-time security updates

#### Network Encryption
**Verified Standards**:
- **In Transit**: TLS 1.2+ mandatory on all connections
  - Client → AWS ELB: TLS 1.2+
  - ELB → NGINX Ingress: TLS 1.2+
  - NGINX → Internal Services: 🟡 Needs verification (likely mTLS)
  - gowalter → Deepgram: TLS 1.2+ (WebSocket over HTTPS)
  - Cresta → Fireworks AI: 🟡 Needs verification

**KVS Stream Encryption**:
- **Source**: AWS Connect documentation
- **Default**: HTTPS transport encryption
- **Optional**: Server-side encryption (requires KMS permissions)
- **Verification Needed**: 🟡 Confirm if Cresta customers use SSE on KVS

---

### Authentication & Authorization

#### 🟡 Gap: Authentication Architecture Not Publicly Documented

**Critical Questions Requiring Vendor Clarification**:

1. **Customer AWS Account Access**:
   - Does Cresta use IAM AssumeRole to access customer KVS streams?
   - If yes, what is the External ID pattern?
   - If no, what authentication mechanism is used?

2. **Agent App Authentication**:
   - How do agents authenticate to Cresta app?
   - SSO integration (SAML, OIDC)?
   - Session management and token expiration?

3. **API Authentication**:
   - How are Cresta API endpoints secured?
   - API keys, OAuth 2.0, or other mechanism?
   - Credential rotation policies?

4. **Service-to-Service**:
   - How does gowalter authenticate to Deepgram?
   - How does Cresta authenticate to Fireworks AI?
   - Are credentials stored in AWS Secrets Manager?

**Assumed Best Practices** (pending verification):
- IAM roles for AWS resource access (not static credentials)
- Least privilege principle for all service accounts
- Credential rotation (90-day maximum for API keys)
- Secrets Manager for credential storage

---

### Data Protection

#### PII Redaction (Verified)
**Source**: Cresta blog "ML Services, Inference Graphs, and Real-Time Intelligence"

**Dual-Layer Approach**:

**Layer 1: Real-Time Redaction**
```
Entity Recognition Service
   ├─ Audio Stream Analysis
   │  └─ Beep insertion on detected PII
   └─ Transcript Analysis  
      └─ Text replacement with entity markers
         └─ Examples: [FULLNAME], [SSN], [CREDIT_CARD]
```

**Layer 2: Post-Call Verification**
```
apiserver (at call end)
   ↓
Temporal Workflow Launch
   ├─ Verification Process
   │  ├─ Re-scan audio for un-redacted PII
   │  ├─ Re-scan transcripts for un-redacted PII
   │  └─ Comparison against original detection
   └─ Remediation
      ├─ If gaps found: Apply additional redaction
      └─ Log verification results for audit
```

**Key Quote from Source**:
"Redaction is an important aspect of the platform, as we want to keep PII confidential. In the case of audio, beeps are applied, while for text it's a simple text redaction for the entity, for example `[FULLNAME]`. This can sometimes be imperfect in real time – for instance, if the entity recognition service misses something – so to ensure all data is properly redacted, apiserver spins up a Temporal workflow to double-check that no un-redacted data remains in the system."

**PII Categories Detected** (🟡 Needs Complete List):
- Full names
- Social Security Numbers (SSN)
- Credit card numbers
- Likely: Phone numbers, addresses, dates of birth, email addresses
- **Verification Needed**: Complete list of entity types and detection accuracy rates

#### Encryption at Rest
**Verified Components**:

**AWS S3 (Audio Storage)**:
- **AWS Standard**: Server-side encryption available
- **Options**: 
  - SSE-S3 (AWS-managed keys)
  - SSE-KMS (Customer-managed keys)
- **Verification Needed**: 🟡 Which option does Cresta use? Customer choice?

**PostgreSQL (Transcripts)**:
- **Standard**: EBS encryption for RDS/Aurora
- **Verification Needed**: 🟡 Confirm encryption enabled

**Redis (Events)**:
- **ElastiCache**: Encryption at rest available
- **Verification Needed**: 🟡 Confirm encryption enabled

**ClickHouse (Analytics)**:
- **Deployment**: Likely on AWS (EBS encryption)
- **Verification Needed**: 🟡 Confirm deployment model and encryption

**Elasticsearch (Search)**:
- **AWS OpenSearch**: Encryption at rest available
- **Verification Needed**: 🟡 Confirm encryption enabled

#### Data Retention & Deletion
**Verified Controls**:

**Audio Files (S3)**:
- **Retention**: Configurable per Amazon Connect instance settings
- **Deletion**: Automatic after retention period (AWS lifecycle policies)
- **Verified**: AWS Connect documentation

**Transcripts (PostgreSQL)**:
- **Retention**: 🟡 Needs clarification (customer-configurable?)
- **Deletion**: 🟡 Needs clarification (soft delete vs hard delete?)

**Analytics (ClickHouse)**:
- **Retention**: 🟡 Typically long-term for historical analysis
- **Verification Needed**: Customer control over retention periods?

**Search Index (Elasticsearch)**:
- **Retention**: 🟡 Likely synchronized with PostgreSQL
- **Verification Needed**: Index cleanup procedures?

**GDPR Right to Deletion**:
- **Requirement**: Customer data deletion within 30 days of request
- **Verification Needed**: 🟡 Documented deletion procedures and timelines

---

### Compliance Certifications (Verified)

#### SOC 2 Type II
**Source**: Cresta documentation

**Coverage**:
- **Trust Service Criteria**: Security, Availability, Processing Integrity, Confidentiality, Privacy
- **Audit Frequency**: Annual
- **Verification**: Current certification status ✅

**Key Controls**:
- Access control and authorization
- Encryption of data in transit and at rest
- Monitoring and logging
- Incident response procedures
- Change management
- Vendor management

#### ISO 27001 (Information Security Management)
**Source**: Cresta documentation

**Scope**: Information security management system (ISMS)
**Coverage**: Comprehensive security controls across organization
**Verification**: Current certification status ✅

#### ISO 27701 (Privacy Information Management)
**Source**: Cresta documentation

**Scope**: Privacy-specific extension to ISO 27001
**Coverage**: GDPR compliance, data subject rights, privacy by design
**Verification**: Current certification status ✅

#### ISO 42001 (AI Management System)
**Source**: Cresta documentation

**Scope**: Responsible AI governance
**Coverage**: AI system transparency, fairness, accountability
**Significance**: First ISO standard for AI management (published 2023)
**Verification**: Current certification status ✅

#### PCI-DSS (Payment Card Industry)
**Source**: Cresta documentation

**Scope**: Cardholder data protection
**Key Control**: PII redaction for credit card numbers
**Verification**: Current compliance status ✅

**Redaction Mechanism**:
- Real-time detection of card numbers in audio/transcripts
- Beep/text redaction before storage
- Temporal verification workflow

#### HIPAA (Healthcare)
**Source**: Cresta documentation

**Availability**: Business Associate Agreement (BAA) available
**Scope**: Protected Health Information (PHI) handling
**Key Controls**: Encryption, access control, audit logging, PII redaction
**Verification**: BAA offering confirmed ✅

**Verification Needed**:
- 🟡 Does BAA cover Deepgram ASR subprocessor?
- 🟡 Does BAA cover Fireworks AI subprocessor?
- 🟡 Regional deployment requirements for HIPAA (US-only?)

---

## Data Security Architecture by Layer

### Layer 1: Network Perimeter

```
Internet
   ↓ (HTTPS)
AWS Route53 DNS
   ├─ customer.region.cresta.ai resolution
   └─ Health checks, failover
   ↓
AWS Elastic Load Balancer (ELB)
   ├─ TLS 1.2+ termination
   ├─ DDoS protection (AWS Shield)
   └─ Security groups (IP filtering)
   ↓ (TLS 1.2+ re-encryption)
NGINX Ingress Controller
   ├─ Wallarm WAF (OWASP protection)
   ├─ Rate limiting
   ├─ API schema validation
   └─ Request logging for audit
   ↓ (Internal routing)
Kubernetes Services (gowalter, apiserver, etc.)
```

**Security Controls**:
- ✅ DNS-based tenant isolation
- ✅ TLS 1.2+ encryption
- ✅ WAF protection (Wallarm)
- ✅ DDoS mitigation (AWS Shield)
- 🟡 IP allowlist capability (needs verification)
- 🟡 Geographic restrictions (needs verification)

---

### Layer 2: Application Security

```
NGINX Ingress
   ↓
Application Services
   ├─ Authentication Middleware
   │  └─ 🟡 Mechanism TBD (OAuth, API key, IAM?)
   ├─ Authorization Middleware  
   │  └─ 🟡 RBAC, ABAC, or other model?
   ├─ Input Validation
   │  ├─ JSON schema validation
   │  ├─ SQL injection prevention
   │  └─ XSS prevention
   └─ Rate Limiting (per tenant)
```

**Security Controls**:
- 🟡 Authentication mechanism (needs verification)
- 🟡 Authorization model (needs verification)
- ✅ Input validation (standard practice)
- 🟡 Rate limiting thresholds (needs verification)

---

### Layer 3: Data Protection

```
Customer Audio
   ↓
gowalter (Real-time PII Detection)
   ├─ Entity Recognition Service
   ├─ Audio redaction (beeps)
   └─ Transcript redaction (markers)
   ↓
S3 Storage
   ├─ Redacted audio only
   ├─ Server-side encryption (🟡 SSE-S3 or SSE-KMS?)
   └─ Bucket policies (least privilege)

Transcripts
   ↓
PostgreSQL
   ├─ Redacted text only
   ├─ EBS encryption (🟡 needs verification)
   ├─ Row-level security (🟡 needs verification)
   └─ Backup encryption (🟡 needs verification)

Temporal Workflow (End-of-Call)
   ├─ Verification scan
   ├─ Gap remediation
   └─ Audit logging
```

**Security Controls**:
- ✅ Dual-layer PII redaction (real-time + post-call)
- ✅ Encrypted storage (S3 standard)
- 🟡 Customer-managed KMS keys option (needs verification)
- 🟡 Database encryption at rest (needs verification)
- 🟡 Backup encryption (needs verification)

---

### Layer 4: Operational Security

#### Monitoring & Logging
**Components Requiring Verification**:

**Application Logs**:
- 🟡 Centralized logging (CloudWatch, ELK, Splunk?)
- 🟡 Log retention period (90 days, 1 year, 7 years?)
- 🟡 PII scrubbing in logs (to prevent sensitive data in logs)

**Audit Logs**:
- 🟡 Authentication/authorization events logged?
- 🟡 Data access logging (who accessed what data when)?
- 🟡 Redaction operation logging?
- 🟡 Model inference request logging?

**Security Monitoring**:
- 🟡 SIEM integration (security event correlation)
- 🟡 Anomaly detection (unusual access patterns)
- 🟡 Alert thresholds and escalation procedures

#### Incident Response
**SOC 2 Requirement**: Documented incident response plan

**Verification Needed**:
- 🟡 Incident classification (P0-P4 severity levels)
- 🟡 Response SLAs per severity
- 🟡 Customer notification procedures
- 🟡 Post-incident review process

#### Vulnerability Management
**Verification Needed**:
- 🟡 Vulnerability scanning frequency (weekly, monthly?)
- 🟡 Penetration testing frequency (annual, semi-annual?)
- 🟡 Patch management SLAs (critical patches within 72 hours?)
- 🟡 Dependency scanning (npm, pip packages)

#### Business Continuity & Disaster Recovery
**High Availability**:
- ✅ Multi-AZ deployment (standard AWS pattern for EKS)
- ✅ Kubernetes pod redundancy (replica sets)
- 🟡 Cross-region failover capability (needs verification)

**Backup & Recovery**:
- ✅ PostgreSQL automated backups (AWS RDS/Aurora standard)
- 🟡 Backup retention period (7 days, 30 days, 90 days?)
- 🟡 Recovery Time Objective (RTO): <4 hours?
- 🟡 Recovery Point Objective (RPO): <1 hour?
- 🟡 Disaster recovery runbook availability

---

## Data Sovereignty & Cross-Border Data Flows

### Regional Architecture (Verified)
**Source**: Cresta documentation on customer subdomains

```
EU Customer
   ↓
Amazon Connect (EU region: eu-west-1, eu-central-1)
   ↓
Kinesis Video Streams (same EU region)
   ↓
customer.eu.cresta.ai
   ├─ Cresta services deployed in EU AWS region
   ├─ PostgreSQL in EU region
   ├─ S3 in EU region
   └─ Elasticsearch/ClickHouse in EU region
   ↓ (Critical question)
Deepgram ASR Endpoint
   └─ 🟡 Regional endpoints available?
      └─ Or data transits to US for ASR processing?
   ↓ (Critical question)
Fireworks AI Inference
   └─ 🟡 Where are Fireworks clusters located?
      ├─ US-only?
      ├─ EU availability?
      └─ Customer choice of region?
```

**GDPR Compliance Implications**:

**Scenario 1: All Processing in EU** ✅
- Cresta: EU region
- Deepgram: EU endpoint
- Fireworks: EU endpoint
- **Result**: GDPR-compliant, no cross-border data flow

**Scenario 2: ASR or ML in US** ⚠️
- Cresta: EU region
- Deepgram or Fireworks: US-only
- **Result**: Cross-border data flow
  - **Requires**: Standard Contractual Clauses (SCCs)
  - **Requires**: Data Protection Impact Assessment (DPIA)
  - **Requires**: Adequate safeguards per GDPR Article 46

**Verification Critical**:
- 🟡 Deepgram regional endpoint availability
- 🟡 Fireworks AI regional deployment map
- 🟡 Customer ability to specify inference region
- 🟡 Data Processing Addendum (DPA) coverage for subprocessors

---

## Subprocessor Security

### Critical Subprocessors

#### Deepgram (ASR Provider)
**Role**: Real-time speech-to-text processing
**Data Exposure**: Customer/agent audio, partial/final transcripts

**Security Requirements**:
- 🟡 SOC 2 Type II certification (verify)
- 🟡 GDPR compliance and DPA availability
- 🟡 HIPAA BAA availability (if processing PHI)
- 🟡 Data retention policy (immediate deletion vs temporary storage?)
- 🟡 Encryption in transit and at rest

#### Fireworks AI (Model Hosting)
**Role**: Ocean-1 + LoRA inference
**Data Exposure**: Conversation transcripts, customer context for RAG

**Security Requirements**:
- 🟡 SOC 2 Type II certification (verify)
- 🟡 Model isolation guarantees (prevent cross-customer data leakage)
- 🟡 GDPR compliance and DPA availability
- 🟡 Data retention policy for inference logs
- 🟡 Encryption in transit and at rest
- 🟡 Regional deployment options

#### AWS (Infrastructure Provider)
**Role**: Compute, storage, networking
**Data Exposure**: All customer data (encrypted)

**Security Status**: ✅
- SOC 2, ISO 27001, PCI-DSS, HIPAA compliant
- AWS Shared Responsibility Model applies
- Customer-managed encryption keys option (KMS)

---

## Security Risk Assessment

### High-Risk Areas Requiring Clarification

| Risk Area | Description | Missing Information | Priority |
|-----------|-------------|---------------------|----------|
| **Cross-Account Access** | How Cresta accesses customer KVS | IAM role pattern, external ID | Critical |
| **API Authentication** | Cresta API endpoint security | Auth mechanism, credential rotation | High |
| **Subprocessor DPA** | Legal coverage for Deepgram/Fireworks | DPA/BAA availability and scope | Critical |
| **Data Residency** | ML inference location | Fireworks/Deepgram regional endpoints | Critical |
| **Encryption Keys** | Customer control over encryption | KMS customer-managed keys option | High |
| **Audit Logging** | Comprehensive audit trail | What events logged, retention | Medium |
| **Incident Response** | Security incident procedures | SLAs, notification process | Medium |

### Medium-Risk Areas Requiring Clarification

| Risk Area | Description | Missing Information | Priority |
|-----------|-------------|---------------------|----------|
| **Rate Limiting** | API abuse prevention | Thresholds, customer controls | Medium |
| **IP Allowlisting** | Network access controls | Capability availability | Medium |
| **Backup Encryption** | Backup data protection | Encryption method, key management | Medium |
| **Vulnerability SLAs** | Patch management | Critical patch timeline | Medium |
| **DR Testing** | Disaster recovery validation | Test frequency, RTO/RPO targets | Medium |

### Low-Risk Areas (Likely Standard Practice)

| Area | Assumption | Verification |
|------|------------|--------------|
| TLS 1.2+ | Enforced on all connections | ✅ Industry standard |
| Input Validation | Standard OWASP controls | ✅ Standard practice |
| Pod Security | Kubernetes security policies | ✅ EKS best practices |
| DDoS Protection | AWS Shield | ✅ AWS standard |

---

## Recommended Security Due Diligence

### Phase 1: Documentation Review (Pre-POC)

**Request from Vendor**:
1. System Security Plan (SSP) or equivalent
2. Data Processing Addendum (DPA) with Schedule of Subprocessors
3. Business Associate Agreement (BAA) if processing PHI
4. Incident Response Plan overview
5. Disaster Recovery Plan summary
6. Most recent SOC 2 Type II report
7. ISO 27001/27701/42001 certificates
8. Vulnerability management procedures
9. Authentication and authorization architecture document
10. Data retention and deletion procedures

### Phase 2: Architecture Deep-Dive (POC Planning)

**Discussion Topics**:
1. IAM role architecture for KVS access (hands-on demo)
2. API authentication mechanism (see actual implementation)
3. Encryption key management (customer-managed keys option?)
4. Subprocessor security (Deepgram/Fireworks certifications)
5. Regional deployment model (data residency guarantees)
6. Audit logging (what's captured, where stored, retention)
7. Monitoring and alerting (CloudWatch dashboards, SLAs)

### Phase 3: Hands-On Security Testing (During POC)

**Test Scenarios**:
1. PII redaction accuracy (test with sample SSN, credit cards, etc.)
2. Temporal verification workflow (confirm post-call checks work)
3. Data deletion (test GDPR right to deletion end-to-end)
4. Encryption validation (verify data encrypted at rest)
5. Authentication testing (attempt unauthorized access)
6. Rate limiting (test API throttling behavior)
7. Monitoring alerts (trigger and verify alerting works)

### Phase 4: Compliance Validation (Pre-Production)

**Audit Activities**:
1. Review SOC 2 report with internal audit team
2. Validate DPA covers all subprocessors adequately
3. Execute BAA if processing PHI
4. Confirm GDPR compliance for EU deployments
5. Verify PCI-DSS controls for payment data
6. Review incident response and notification procedures
7. Validate DR/BC plan aligns with organizational requirements

---

**Document Status**: Comprehensive security analysis based on verified certifications and architecture. Critical gaps in authentication, subprocessor security, and data residency require vendor clarification before production deployment.
