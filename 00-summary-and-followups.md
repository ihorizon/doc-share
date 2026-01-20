# Architecture Documentation Summary & Follow-Up Items

## Document Index

| # | Document | Description |
|---|----------|-------------|
| 01 | [Overall Architecture](01-overall-architecture.md) | High-level system overview |
| 02 | [Amazon Connect Integration](02-amazon-connect-integration.md) | Connect-specific integration details |
| 03 | [Voice Stack Architecture](03-voice-stack-architecture.md) | Audio processing and ASR pipeline |
| 04 | [ML Services Architecture](04-ml-services-architecture.md) | Model serving and inference |
| 05 | [Real-Time Data Flows](05-realtime-dataflow-sequences.md) | Sequence diagrams for key flows |
| 06 | [Data & Security](06-data-security-architecture.md) | Storage, compliance, security |

---

## Confirmed Architecture Details

### From Cresta Documentation (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Platform Hosting** | AWS (EKS clusters) | Cresta blog |
| **Customer Isolation** | Separate databases per customer | Cresta blog |
| **DNS Routing** | Customer-specific subdomains (customer.region.cresta.ai) | Cresta blog |
| **WAF** | Wallarm (based on NGINX Ingress) | Cresta blog |
| **Audio Handler** | gowalter service | Cresta blog |
| **ASR Provider** | Deepgram (primary) | Cresta blog |
| **Transcript Store** | PostgreSQL | Cresta blog |
| **Event Streaming** | Redis Streams | Cresta blog |
| **Search** | Elasticsearch | Cresta blog |
| **Analytics** | ClickHouse | Cresta blog |
| **Audio Storage** | AWS S3 (encrypted) | Cresta blog |
| **Foundation Model** | Ocean-1 (Mistral 7B fine-tuned) | Cresta blog |
| **Model Hosting** | Fireworks AI (with LoRA adapters) | Cresta blog, Fireworks |
| **PII Redaction** | Audio beeps + text masking | Cresta blog |
| **Redaction Verification** | Temporal workflows | Cresta blog |
| **Certifications** | SOC 2 Type II, ISO 27001/27701/42001, PCI-DSS, HIPAA | Cresta Trust Center |

### From Amazon Connect Documentation (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Audio Streaming** | Kinesis Video Streams | AWS Docs |
| **Audio Format** | 8kHz PCM | AWS Docs |
| **Track Separation** | AUDIO_TO_CUSTOMER, AUDIO_FROM_CUSTOMER | AWS Docs |
| **Integration Method** | Contact Flow + Lambda trigger | AWS Docs |
| **Contact Attributes** | streamARN, startFragmentNum, ContactId | AWS Docs |

---

## 🟡 Items Requiring Follow-Up

### High Priority (Integration Critical)

| # | Item | Document | Impact |
|---|------|----------|--------|
| 1 | **KVS Consumer Method** - Does Cresta use Lambda consumer or direct GetMedia API? | 02 | Integration design |
| 2 | **Authentication Method** - API Key, OAuth, or IAM-based auth to Cresta? | 02 | Security configuration |
| 3 | **Agent App Deployment** - Browser extension, desktop app, or embedded CCP? | 02 | Agent rollout |
| 4 | **Contact Attribute Mapping** - Required custom attributes for Cresta | 02 | Contact Flow config |
| 5 | **Failover Behavior** - Call continuity if Cresta is unreachable | 02, 05 | SLA/resilience |

### Medium Priority (Operational)

| # | Item | Document | Impact |
|---|------|----------|--------|
| 6 | **ASR Alternatives** - Can Amazon Transcribe be used instead of Deepgram? | 03 | Cost/latency options |
| 7 | **Audio Buffer Size** - How much audio is buffered for recovery? | 03 | Memory planning |
| 8 | **Model Inventory** - Complete list of models and purposes | 04 | Capacity planning |
| 9 | **Latency SLAs** - Published latency guarantees | 04, 05 | SLA definition |
| 10 | **Scaling Rules** - Auto-scaling configuration for ML shards | 04 | Capacity planning |

### Lower Priority (Nice to Have)

| # | Item | Document | Impact |
|---|------|----------|--------|
| 11 | **Multi-Language ASR** - How is language detection handled? | 03 | Global deployment |
| 12 | **TTS for Translation** - How is translated audio played to customer? | 05 | Translation feature |
| 13 | **SSO Providers** - Which SSO platforms are supported? | 06 | Admin setup |
| 14 | **Data Retention Defaults** - Default retention periods | 06 | Compliance |
| 15 | **APAC Region Availability** - Data residency in AP-Southeast-1 | 06 | Regional expansion |
| 16 | **Backup RPO/RTO** - Recovery objectives | 06 | DR planning |

---

## Risk Summary by Category

### ⏱️ Latency Risks (Real-Time Critical Path)

| Component | Risk Level | Notes |
|-----------|------------|-------|
| KVS → Cresta network | Medium | Regional deployment critical |
| ASR processing | **High** | 200-300ms target, Deepgram dependency |
| ML inference | **High** | 500ms target, Fireworks dependency |
| WebSocket delivery | Medium | Client network dependent |

### 🔒 Security Risks

| Component | Risk Level | Notes |
|-----------|------------|-------|
| Cross-account KVS access | Medium | IAM configuration required |
| API authentication | Medium | 🟡 Verify mechanism |
| PII in transit | Low | TLS enforced |
| Cross-tenant isolation | Low | Database separation confirmed |

### 📋 Compliance Risks

| Component | Risk Level | Notes |
|-----------|------------|-------|
| GDPR data residency | Low | EU region available |
| Call recording consent | Medium | Contact Flow must announce |
| PII redaction gaps | Low | Temporal verification in place |
| Data retention | Medium | 🟡 Verify customer controls |

### ⚙️ Operational Risks

| Component | Risk Level | Notes |
|-----------|------------|-------|
| Cresta availability | Medium | 🟡 Verify failover |
| Deepgram availability | Medium | 🟡 Verify fallback |
| Fireworks availability | Medium | 🟡 Verify fallback |
| KVS quota limits | Low | Monitor and request increases |

---

## Recommended Next Steps

1. **Schedule Cresta Technical Deep-Dive**
   - Clarify KVS integration method
   - Confirm authentication approach
   - Review failover architecture

2. **AWS Architecture Review**
   - Validate KVS quota requirements
   - Plan Lambda concurrency
   - Design IAM role structure

3. **Security Assessment**
   - Complete security questionnaire
   - Review Cresta Trust Center documentation
   - Validate PII handling for your data types

4. **POC Planning**
   - Define pilot scope (agent count, call volume)
   - Identify test scenarios
   - Establish success metrics

---

## Diagram Legend Reference

| Icon | Meaning |
|------|---------|
| ⏱️ | Latency risk - performance critical |
| 🔒 | Security risk - data protection concern |
| 📋 | Compliance risk - regulatory concern |
| ⚙️ | Operational risk - availability/monitoring |
| 🟡 | Requires follow-up/verification |
| 🟢 | Confirmed/documented |
| 🔴 | Critical gap/unknown |
