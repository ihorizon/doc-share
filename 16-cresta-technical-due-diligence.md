# Cresta Technical Due Diligence – What We Know, Assumptions & Questions

**Client Context**: Banking / Financial Services  
**Regulatory Framework**: APRA (CPS 231, CPS 232, CPS 234), Privacy Act 1988, AML/CTF  
**Data Classification**: Restricted Trusted  
**Last Updated**: January 2025

---

## Executive Summary

This document provides a structured technical due diligence summary for the Cresta AI integration with Amazon Connect. Each section covers:
- **What We Know**: Confirmed facts from verified sources
- **Assumptions**: Inferences we are making that require validation
- **Questions for Cresta**: Items requiring clarification

**Client-Specific Considerations**:
- As a regulated bank under APRA, material outsourcing arrangements require compliance with CPS 231 (Outsourcing) and CPS 234 (Information Security)
- Data is classified as **Restricted Trusted**, requiring enhanced controls for handling customer financial data
- Australian data sovereignty and Privacy Act requirements may apply

---

## 1. Overall Architecture

### What We Know ✅

| Component | Detail | Source |
|-----------|--------|--------|
| **Cloud Provider** | AWS (EKS clusters) | Cresta blog, AWS ML Blog |
| **Container Orchestration** | Amazon Elastic Kubernetes Service (EKS) | AWS ML Blog |
| **Infrastructure Migration** | Consolidated from multi-cloud to AWS (2021) | AWS ML Blog |
| **Customer Isolation** | Separate databases per customer | Cresta blog |
| **DNS Routing** | Customer-specific subdomains (`customer.region.cresta.ai`) | Cresta blog |
| **API Endpoint Pattern** | `https://api-CUSTOMER_ID.cresta.com` | SDK Guide |
| **Auth Endpoint** | `https://auth.cresta.com` (default) | SDK Guide |
| **WAF** | Wallarm (NGINX Ingress-based) | Cresta blog |
| **TLS** | TLS 1.2+ enforced | Trust Center |
| **Regions Available** | US (us-west-2), EU (eu-west-1) | Cresta docs |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A1 | AWS region AP-Southeast-2 (Sydney) is available or can be provisioned for Australian data sovereignty | Data may need to leave Australia; APRA/Privacy Act non-compliance | Low |
| A2 | Customer namespace isolation in Kubernetes provides adequate separation for banking data | Cross-tenant data exposure risk | Medium |
| A3 | Cresta infrastructure is single-tenant or can be deployed single-tenant for banking requirements | Shared infrastructure may not meet CPS 234 requirements | Low |
| A4 | Cresta can provide architecture documentation suitable for APRA CPS 231 material outsourcing assessment | May not be able to complete regulatory approval | Unknown |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q1.1 | **Is AP-Southeast-2 (Sydney) available for Australian data sovereignty requirements?** If not, what is the roadmap? | Critical | CPS 234 – Data must remain in Australia |
| Q1.2 | **Can Cresta provide single-tenant deployment** for banking clients requiring isolated infrastructure? | High | CPS 234 – Information security capability |
| Q1.3 | **What is the full architecture documentation** available for APRA CPS 231 due diligence? | High | CPS 231 – Material outsourcing |
| Q1.4 | **How is namespace/tenant isolation implemented** and what controls prevent cross-tenant access? | High | CPS 234 – Control testing |
| Q1.5 | **What third-party penetration testing results** can be shared? | Medium | CPS 234 – Security testing |
| Q1.6 | **What is the geographic location of Cresta management/support staff** with access to customer data? | Medium | Privacy Act – Offshore disclosure |

---

## 2. Amazon Connect Integration

### What We Know ✅

| Component | Detail | Source |
|-----------|--------|--------|
| **Audio Streaming** | Kinesis Video Streams (KVS) | AWS Docs |
| **Audio Format** | PCM linear16 (16-bit PCM) | Confirmed |
| **Sample Rate** | 8kHz (telephony standard) | AWS Docs |
| **Track Separation** | `AUDIO_TO_CUSTOMER`, `AUDIO_FROM_CUSTOMER` (separate tracks) | AWS Docs |
| **Integration Trigger** | Contact Flow + Lambda | AWS Docs |
| **Contact Attributes** | `streamARN`, `startFragmentNum`, `ContactId` available | AWS Docs |
| **SDK Voice Session** | Detected via `voiceManager.onVoiceSession()` callback | SDK Guide |
| **Profile ID** | Required for voice manager initialization | SDK Guide |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A5 | Cresta uses Lambda consumer or GetMedia API to consume KVS streams (not embedded agent) | Integration architecture may be different | Medium |
| A6 | StreamARN can be correlated to ContactId in real-time (despite AWS limitation showing it only in CTR) | Real-time processing may not work as expected | Low |
| A7 | AWS IAM cross-account AssumeRole with external ID is used for KVS access | Different auth pattern may have security implications | Medium |
| A8 | Agent App is SDK-based embedded in CCP or browser-based (not desktop app) | Agent deployment approach may differ | Low |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q2.1 | **How does Cresta consume KVS streams?** Lambda consumer, direct GetMedia API, or other? | Critical | Integration design |
| Q2.2 | **How is KVS StreamARN correlated to ContactId in real-time?** AWS docs indicate StreamARN only available in CTR after call ends. | Critical | Real-time capability |
| Q2.3 | **What is the Agent App deployment model** for Amazon Connect? Browser extension, desktop app, embedded in CCP, or SDK-based custom? | High | Agent rollout |
| Q2.4 | **What IAM permissions/roles are required** for Cresta to access customer's KVS streams? | High | CPS 234 – Access control |
| Q2.5 | **What custom Contact Attributes are required** in the Contact Flow for Cresta? | High | Integration design |
| Q2.6 | **What is the authentication method** between Connect/Lambda and Cresta endpoints? (API key, OAuth, IAM) | High | CPS 234 – Authentication |
| Q2.7 | **Is there an Amazon Connect-specific integration guide** with step-by-step deployment instructions? | Medium | Implementation |

---

## 3. Voice Stack

### What We Know ✅

| Component | Detail | Source |
|-----------|--------|--------|
| **Audio Handler** | gowalter service | Cresta blog |
| **Audio Chunk Size** | 20-100ms chunks | Cresta blog |
| **ASR Provider** | Deepgram Nova-3 | Cresta blog, Deepgram docs |
| **ASR Latency** | Sub-300ms end-to-end | Deepgram benchmarks |
| **ASR Accuracy** | 6.84% WER median; 54% better than AWS Transcribe | Independent benchmarks |
| **Partial Transcripts** | Every 0.5-1.5 seconds | Cresta blog |
| **Final Transcripts** | Every 3-7 seconds | Cresta blog |
| **Transcript Store** | PostgreSQL | Cresta blog |
| **WebSocket Recovery** | gowalter handles buffering and reconnection | Cresta blog |
| **Real-time Delivery** | ClientSubscription WebSocket service | SDK Guide |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A9 | Deepgram processes audio in Cresta's infrastructure, not sent to Deepgram's cloud separately | Data may leave controlled environment | Medium |
| A10 | Audio is encrypted in transit to Deepgram (if external) | Privacy/security gap | High |
| A11 | gowalter buffer is sufficient to handle short network interruptions without data loss | Transcript gaps during network issues | Medium |
| A12 | ASR language can be configured for Australian English | Accuracy may be impacted | Medium |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q3.1 | **Where does Deepgram ASR processing occur?** Within Cresta infrastructure, Deepgram cloud, or customer VPC? | Critical | CPS 234 – Data location; Privacy Act |
| Q3.2 | **What is the audio buffer size** in gowalter for handling network interruptions? | Medium | Resilience |
| Q3.3 | **How is multi-language support handled?** Can Australian English be configured? | Medium | Accuracy |
| Q3.4 | **What happens to audio data if ASR processing fails?** Is it retried, queued, or lost? | High | Data completeness |
| Q3.5 | **Can an alternative ASR provider (e.g., AWS Transcribe) be used** if required for data sovereignty? | Medium | CPS 234 – Vendor flexibility |
| Q3.6 | **What is the data retention for raw audio vs. transcripts** at each processing stage? | High | Privacy Act – Data retention |

---

## 4. ML Services

### What We Know ✅

| Component | Detail | Source |
|-----------|--------|--------|
| **Foundation Model** | Ocean-1 (Mistral 7B fine-tuned) | Cresta blog, Fireworks AI |
| **Model Hosting** | Fireworks AI | Cresta blog, Fireworks AI |
| **Customization** | LoRA adapters per customer | Fireworks AI blog |
| **Cost Efficiency** | 100x reduction vs GPT-4 | Fireworks AI blog |
| **ML Framework** | PyTorch with TorchServe | AWS ML Blog |
| **Inference Target** | <500ms for Knowledge Assist | Cresta blog |
| **End-to-End Target** | <1.5 seconds speech-to-guidance | Cresta blog |
| **Model Router** | Routes requests to appropriate model shards | Cresta blog |
| **Policy Engine** | Agent/team/org-level policy configuration | Cresta blog |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A13 | Fireworks AI processes inference within controlled environment (not public cloud) | Banking data sent to third-party cloud | Medium |
| A14 | Customer-specific LoRA adapters are trained only on that customer's data | Cross-customer data exposure in training | High |
| A15 | Model outputs do not persistently store customer conversation data | Data may be retained longer than expected | Medium |
| A16 | Models do not use customer data for general model improvement without consent | Privacy violation; CPS 234 | Medium |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q4.1 | **Where does Fireworks AI inference occur?** Dedicated infrastructure, shared cloud, or customer VPC option? | Critical | CPS 234 – Data location |
| Q4.2 | **Is customer conversation data used to train or improve models?** If so, how can this be opted out? | Critical | Privacy Act – Consent |
| Q4.3 | **What data is sent to Fireworks AI** for inference? Full transcript, summary, or embeddings only? | High | Data minimization |
| Q4.4 | **How are LoRA adapters trained?** Where does training occur and who has access to training data? | High | CPS 234 – Data handling |
| Q4.5 | **What latency SLAs does Cresta provide** for ML inference? | Medium | Performance expectations |
| Q4.6 | **What is the model inventory** and which models require external API calls? | Medium | Third-party risk |
| Q4.7 | **Can models be deployed on-premises or in customer AWS account** for banking requirements? | Medium | CPS 234 – Control |

---

## 5. Data

### What We Know ✅

#### 5.1 Transient Data (In-Flight)

| Data Type | Location | Encryption | Retention |
|-----------|----------|------------|-----------|
| Audio chunks | gowalter buffer | TLS in transit | Buffered until processed |
| ASR partial transcripts | Memory | TLS in transit | Until final transcript |
| ML inference requests | API calls | TLS in transit | Not persisted |
| WebSocket messages | ClientSubscription | TLS in transit | Real-time only |

#### 5.2 Persisted Data (At-Rest)

| Data Type | Storage | Encryption | Isolation |
|-----------|---------|------------|-----------|
| Transcripts | PostgreSQL | AES-256 at rest | Separate database per customer |
| Audio files | AWS S3 | AES-256 (SSE-S3 or KMS) | Separate S3 paths per customer |
| Annotations/moments | PostgreSQL | AES-256 at rest | Separate database per customer |
| Analytics data | ClickHouse | AES-256 at rest | Logical separation |
| Search indices | Elasticsearch | AES-256 at rest | Separate index per customer |
| Configuration | PostgreSQL | AES-256 at rest | Per customer |

#### 5.3 Protected Data (Security Controls)

| Control | Implementation | Status |
|---------|---------------|--------|
| Encryption at rest | AES-256 | ✅ Confirmed |
| Encryption in transit | TLS 1.2+ | ✅ Confirmed |
| Key management | AWS KMS | ✅ Confirmed |
| PII redaction | Audio beeps + text masking | ✅ Confirmed |
| Access control | RBAC + IAM | ✅ Confirmed |
| MFA | Multi-factor authentication | ✅ Trust Center |

#### 5.4 Audited Data (Logging & Compliance)

| Audit Type | Implementation | Status |
|------------|---------------|--------|
| Access logs | All data access logged | ✅ Trust Center |
| Action audit trail | User actions logged | ✅ Trust Center |
| Compliance reporting | SOC 2 Type II | ✅ Confirmed |

#### 5.5 Data Flows

| Flow | Source → Destination | Data Type | Classification |
|------|---------------------|-----------|----------------|
| Audio ingestion | KVS → gowalter | Raw audio | Restricted Trusted |
| ASR processing | gowalter → Deepgram | Audio chunks | Restricted Trusted |
| Transcript persistence | apiserver → PostgreSQL | Transcripts | Restricted Trusted |
| ML inference | Orchestrator → Fireworks AI | Transcript context | Restricted Trusted |
| Agent delivery | clientsubscription → Agent App | Guidance, hints | Restricted Trusted |
| Audio storage | gowalter → S3 | Redacted audio | Restricted Trusted |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A17 | All data at rest is encrypted with customer-managed keys (CMK) or Cresta-managed keys | May not meet banking key management requirements | Medium |
| A18 | Audit logs are retained for sufficient period for regulatory requirements (7+ years) | Compliance gap | Low |
| A19 | Data can be deleted on request within regulatory timeframes | GDPR/Privacy Act non-compliance | Medium |
| A20 | Backup data is encrypted to same standard as primary data | Backup exposure risk | High |
| A21 | Data residency controls prevent data from leaving designated region | APRA/Privacy Act non-compliance | Low |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q5.1 | **Can customers use their own KMS keys (CMK)** for encryption, or is it Cresta-managed only? | High | CPS 234 – Key management |
| Q5.2 | **What is the default data retention period** for audio, transcripts, and analytics? | High | Privacy Act – Retention limits |
| Q5.3 | **Can retention periods be customized** per data type? What is the minimum/maximum? | High | Compliance flexibility |
| Q5.4 | **How long are audit logs retained?** Can this be extended for banking requirements (7+ years)? | High | CPS 234 – Audit trail |
| Q5.5 | **What is the process for data deletion requests?** Timeframe and verification? | High | Privacy Act – Deletion rights |
| Q5.6 | **What is the RPO (Recovery Point Objective)?** RTO is 8 hours per Trust Center. | High | CPS 232 – BCM |
| Q5.7 | **Is backup data encrypted to the same standard?** Where are backups stored geographically? | High | CPS 234 – Backup security |
| Q5.8 | **What data leaves the primary region** (e.g., for subprocessors like Deepgram, Fireworks)? | Critical | Privacy Act – Cross-border |
| Q5.9 | **Can data residency be guaranteed for Australia** (no data leaves AP region)? | Critical | Privacy Act, CPS 234 |
| Q5.10 | **What metadata about calls is logged/retained** even after audio/transcript deletion? | Medium | Privacy Act |

---

## 6. Security

### What We Know ✅

#### 6.1 Certifications (Confirmed via Trust Center)

| Certification | Status | Notes |
|---------------|--------|-------|
| SOC 2 Type II | ✅ Confirmed | Security, availability, processing integrity |
| ISO 27001 | ✅ Confirmed | Information security management |
| ISO 27701 | ✅ Confirmed | Privacy information management |
| ISO 42001 | ✅ Confirmed | AI management systems |
| PCI-DSS | ✅ Confirmed | Payment card data security |
| HIPAA | ✅ Confirmed | Healthcare data (BAA available) |
| GDPR | ✅ Confirmed | EU privacy |
| CCPA/CPRA | ✅ Confirmed | California privacy |
| TISAX | ✅ Confirmed | Automotive industry |

#### 6.2 Security Controls

| Control Category | Implementation | Status |
|------------------|---------------|--------|
| WAF | Wallarm (NGINX-based) | ✅ Confirmed |
| DDoS Protection | AWS Shield / CloudFront | ✅ Assumed |
| TLS | TLS 1.2+ termination | ✅ Confirmed |
| VPC | AWS VPC with private subnets | ✅ Confirmed |
| IDS/IPS | Intrusion detection/prevention | ✅ Trust Center |
| Network Vulnerability Scanning | Regular assessments | ✅ Trust Center |
| Disk Encryption | Full-disk on endpoints | ✅ Trust Center |
| EDR | Endpoint detection & response | ✅ Trust Center |
| MDM | Mobile device management | ✅ Trust Center |
| MFA | Multi-factor authentication | ✅ Trust Center |
| RBAC | Role-based access control | ✅ Confirmed |

#### 6.3 Subprocessors (Trust Center)

| Subprocessor | Purpose | Data Shared | Location |
|--------------|---------|-------------|----------|
| Fireworks.ai | LLM inference | Transcript context | Unknown |
| Deepgram | ASR | Audio | Unknown |
| OpenAI | LLM services | Unknown | Unknown |
| Cartesia AI | TTS | Unknown | Unknown |
| ElevenLabs | TTS | Unknown | Unknown |
| Google Cloud | Infrastructure | Unknown | Unknown |
| Datadog | Monitoring | Logs/metrics | Unknown |
| Segment | Analytics | Usage data | Unknown |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A22 | Annual penetration testing is conducted by independent third party | Security gaps may exist | High |
| A23 | Incident response plan includes customer notification within regulatory timeframes | APRA notification breach | Medium |
| A24 | Subprocessor data flows comply with Australian Privacy Act requirements | Privacy violation | Low |
| A25 | Security certifications cover the Amazon Connect integration specifically | Certification gaps | Medium |
| A26 | Staff with data access undergo background checks | Insider threat risk | High |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q6.1 | **What is the incident response notification timeframe** for security breaches? | Critical | CPS 234 – Incident notification |
| Q6.2 | **Can penetration test reports be shared** with banking client? | High | CPS 234 – Security assurance |
| Q6.3 | **Where are subprocessors (Deepgram, Fireworks) located geographically?** | Critical | Privacy Act – Cross-border |
| Q6.4 | **What data is shared with each subprocessor?** Can this be limited? | High | Data minimization |
| Q6.5 | **Do staff with data access undergo background checks?** What level? | High | CPS 234 – Personnel security |
| Q6.6 | **Can SIEM integration be provided** for customer security monitoring? | Medium | CPS 234 – Monitoring |
| Q6.7 | **What is the vulnerability management process?** Patching SLAs? | Medium | CPS 234 – Vulnerability management |
| Q6.8 | **Is there an option to opt out of specific subprocessors** (e.g., use AWS Transcribe instead of Deepgram)? | Medium | Vendor control |
| Q6.9 | **What security certifications cover the Amazon Connect integration** specifically? | Medium | Certification scope |
| Q6.10 | **Can Cresta provide a CPS 234 mapping document** showing how controls meet APRA requirements? | High | APRA compliance |

---

## 7. Proof of Concept (POC)

### What We Know ✅

| Aspect | Detail | Source |
|--------|--------|--------|
| **SDK Availability** | @cresta/client (0.21.0), @cresta/react-client (0.10.0) | SDK Guide |
| **NPM Registry** | `https://npm.cresta.com` | SDK Guide |
| **Emulator Available** | @cresta/client-sdk-emulator for testing | SDK Guide |
| **Authentication Options** | BYOID (OIDC), SAML, popup, Twilio Flex | SDK Guide |
| **Voice Session API** | `voiceManager.onVoiceSession()` | SDK Guide |
| **UI Components** | Smart Compose, Hints, Suggestions, Omni Search | SDK Guide |

### POC Requirements for Banking Client

| Requirement | Description | APRA Relevance |
|-------------|-------------|----------------|
| **Australian Data Residency** | All data must remain in Australia during POC | CPS 234, Privacy Act |
| **Restricted Data Handling** | POC must use synthetic/anonymized data or production controls | Data classification |
| **Security Assessment** | Security review must be completed before production data | CPS 234 |
| **Audit Trail** | All POC activity must be logged | CPS 234 |
| **Third-Party Risk Assessment** | TPRM assessment of Cresta required | CPS 231 |

### Assumptions ⚠️

| # | Assumption | Risk if Wrong | Confidence |
|---|------------|---------------|------------|
| A27 | POC can be conducted with synthetic data before production approval | May need to wait for full security approval | High |
| A28 | Cresta can provide a non-production environment for POC | May need to use production infrastructure | Medium |
| A29 | POC environment meets same security standards as production | Security gaps in POC | Medium |
| A30 | Agent App can be tested without full production deployment | May need production deployment | Medium |

### Questions for Cresta ❓

| # | Question | Priority | APRA Relevance |
|---|----------|----------|----------------|
| Q7.1 | **Can a POC be conducted in AP-Southeast-2** with Australian data residency? | Critical | Privacy Act, CPS 234 |
| Q7.2 | **Is there a sandbox/non-production environment** for POC? | High | Security isolation |
| Q7.3 | **Can POC use synthetic/anonymized data** instead of real customer calls? | High | Data protection |
| Q7.4 | **What is the minimum scope for a POC?** (Agents, calls, duration) | Medium | Planning |
| Q7.5 | **What support is provided during POC?** Technical resources, SLAs? | Medium | Planning |
| Q7.6 | **What success metrics should be measured** during POC? | Medium | Evaluation |
| Q7.7 | **What is the timeline from POC completion to production** deployment? | Medium | Planning |
| Q7.8 | **Can POC data be completely deleted** after POC completion? | High | Privacy Act |
| Q7.9 | **What third-party risk assessment documentation** can be provided for CPS 231? | High | APRA compliance |
| Q7.10 | **Is there an existing banking/financial services reference customer** we can speak with? | Medium | Due diligence |

---

## Summary: Priority Questions for Cresta

### Critical (Blocking for Engagement)

| # | Question | Section |
|---|----------|---------|
| Q1.1 | Is AP-Southeast-2 (Sydney) available for Australian data sovereignty? | Architecture |
| Q2.1 | How does Cresta consume KVS streams? | Connect Integration |
| Q2.2 | How is StreamARN correlated to ContactId in real-time? | Connect Integration |
| Q3.1 | Where does Deepgram ASR processing occur? | Voice Stack |
| Q4.1 | Where does Fireworks AI inference occur? | ML Services |
| Q4.2 | Is customer data used to train models? | ML Services |
| Q5.8 | What data leaves the primary region for subprocessors? | Data |
| Q5.9 | Can data residency be guaranteed for Australia? | Data |
| Q6.1 | What is the incident response notification timeframe? | Security |
| Q6.3 | Where are subprocessors located geographically? | Security |
| Q7.1 | Can POC be conducted in AP-Southeast-2? | POC |

### High Priority (Required for Security Review)

| # | Question | Section |
|---|----------|---------|
| Q1.2 | Can Cresta provide single-tenant deployment? | Architecture |
| Q1.3 | Full architecture documentation for CPS 231? | Architecture |
| Q2.4 | IAM permissions required for KVS access? | Connect Integration |
| Q2.6 | Authentication method to Cresta endpoints? | Connect Integration |
| Q5.1 | Can customers use their own KMS keys? | Data |
| Q5.4 | How long are audit logs retained? | Data |
| Q5.6 | What is the RPO? | Data |
| Q6.2 | Can penetration test reports be shared? | Security |
| Q6.5 | Do staff undergo background checks? | Security |
| Q6.10 | Can Cresta provide CPS 234 mapping document? | Security |

---

## Regulatory Considerations (APRA)

### CPS 231 – Outsourcing

| Requirement | Question for Cresta | Status |
|-------------|---------------------|--------|
| Material outsourcing due diligence | Q1.3 – Architecture documentation | Pending |
| Risk assessment | Q6.10 – CPS 234 mapping | Pending |
| Business continuity | Q5.6 – RPO/RTO | Partial (RTO 8h confirmed) |
| Exit strategy | Not yet addressed | Gap |

### CPS 232 – Business Continuity Management

| Requirement | Question for Cresta | Status |
|-------------|---------------------|--------|
| Recovery objectives | Q5.6 – RPO (RTO 8h confirmed) | Partial |
| Testing | Q6.2 – Penetration test reports | Pending |
| Scenarios | Not yet addressed | Gap |

### CPS 234 – Information Security

| Requirement | Question for Cresta | Status |
|-------------|---------------------|--------|
| Information security capability | Q1.2 – Single-tenant option | Pending |
| Security controls | Q6.10 – CPS 234 mapping | Pending |
| Incident management | Q6.1 – Notification timeframe | Pending |
| Testing | Q6.2 – Penetration test reports | Pending |

### Privacy Act 1988

| Requirement | Question for Cresta | Status |
|-------------|---------------------|--------|
| Australian data residency | Q1.1, Q5.9 – AP-Southeast-2 | Pending |
| Cross-border disclosure | Q5.8, Q6.3 – Subprocessor locations | Pending |
| Access and deletion rights | Q5.5 – Deletion process | Pending |

---

## References

- [00-summary-and-followups.md](00-summary-and-followups.md) – Main summary
- [00-research-findings-summary.md](00-research-findings-summary.md) – Research verification
- [14-cresta-sdk-developer-guide-analysis.md](14-cresta-sdk-developer-guide-analysis.md) – SDK analysis
- [15-cresta-questions-and-gaps.md](15-cresta-questions-and-gaps.md) – Questions summary
- [Cresta Trust Center](https://trust.cresta.com/) – Compliance and security
- [APRA CPS 231](https://www.apra.gov.au/outsourcing) – Outsourcing requirements
- [APRA CPS 234](https://www.apra.gov.au/information-security) – Information security requirements
