# Architecture Documentation Summary & Follow-Up Items

## Document Index

| # | Document | Description |
|---|----------|-------------|
| 01 | [Overall Architecture](01-overall-architecture.md) | High-level system overview |
| 02 | [Amazon Connect Integration](02-amazon-connect-integration.md) | Connect-specific integration details |
| 03 | [Voice Stack Architecture](03-voice-stack-architecture.md) | Audio processing and ASR pipeline |
| 04 | [ML Services Architecture](04-ml-services-architecture.md) | Model serving and inference |
| 05 | [Real-Time Data Flows](05-realtime-dataflow-sequences.md) | Sequence diagrams for key flows |
| 06 | [Data Architecture](06-data-architecture.md) | Data storage, retention, lifecycle |
| 07 | [Security & Compliance](07-security-compliance-architecture.md) | Security controls, compliance framework, PII redaction |
| 08 | [Cresta Feature Documentation Request](08-cresta-feature-documentation-request.md) | Request for feature catalog and capabilities |
| 09 | [Cresta Amazon Connect Integration Request](09-cresta-amazon-connect-integration-request.md) | Request for integration guide and setup |
| 10 | [Cresta Technical Sessions Agenda](10-cresta-technical-sessions-agenda.md) | Agenda and prep for technical sessions |
| 11 | [Business Use Cases](11-business-use-cases.md) | Contact center business use cases |
| 12 | [POC Test Scenarios](12-poc-test-scenarios.md) | Technical test scenarios, success criteria, validation |
| 13 | [Proof of Concept Plan](13-proof-of-concept-plan.md) | Business-focused POC plan aligned with use cases |
| — | [References](references.md) | Central index of all material references and what was referenced |

---

## ⚠️ Critical Verification Notes

**Important**: This documentation is based on publicly available information, Cresta blog posts, AWS documentation, and logical inference. The following items require direct confirmation from Cresta:

1. **Audio Format**: ✅ **Confirmed**: PCM linear16 (16-bit PCM) encoding. Sample rate (8kHz) still requires verification with Cresta or AWS support.

2. **KVS Integration Method**: The exact mechanism by which Cresta consumes KVS streams (Lambda consumer vs direct GetMedia API) is not publicly documented and requires Cresta confirmation.

3. **Authentication Mechanism**: The specific authentication method (API keys, OAuth, IAM) for Connect/Lambda to authenticate with Cresta endpoints is not documented and requires verification.

4. **Failover Behavior**: The behavior when Cresta is unreachable during a call (call continuity, transcript gaps, agent experience) requires Cresta confirmation.

5. **Agent App Deployment**: The exact deployment method (browser extension, desktop app, embedded CCP) for Amazon Connect integration requires verification.

**Documentation Status**: All documents use consistent terminology and architecture patterns. Where information is inferred or not explicitly confirmed, it is marked with 🟡 (requires verification) indicators.

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
| **Foundation Model** | Ocean-1 (Mistral 7B fine-tuned) | Cresta blog | ✅ Confirmed via web search |
| **Model Hosting** | Fireworks AI (with LoRA adapters) | Cresta blog, Fireworks | ✅ Confirmed via web search |
| **ML Framework** | PyTorch with TorchServe | AWS ML Blog | ✅ Confirmed via web search |
| **Infrastructure Migration** | Consolidated from multi-cloud to AWS | AWS ML Blog | ✅ Confirmed via web search |
| **PII Redaction** | Audio beeps + text masking | Cresta blog |
| **Redaction Verification** | Temporal workflows | Cresta blog |
| **Certifications** | SOC 2 Type II, ISO 27001/27701/42001, PCI-DSS, HIPAA, CCPA/CPRA, GDPR, TISAX | [Cresta Trust Center](https://trust.cresta.com/) |
| **Recovery Time Objective (RTO)** | 8 hours | [Cresta Trust Center](https://trust.cresta.com/) |
| **Risk Profile** | Data Access Level: Internal; Impact Level: Substantial | [Cresta Trust Center](https://trust.cresta.com/) |
| **MFA** | Multi-factor authentication | [Cresta Trust Center](https://trust.cresta.com/) |
| **SSO Provider (Client-side)** | PingFederate | Client implementation confirmed |

### From Amazon Connect Documentation (High Confidence)

| Component | Detail | Source | Verification Status |
|-----------|--------|--------|---------------------|
| **Audio Streaming** | Kinesis Video Streams | AWS Docs | ✅ Confirmed |
| **Audio Format** | PCM linear16 (16-bit PCM) | Confirmed | ✅ **Confirmed**: PCM linear16 encoding. Sample rate (8kHz) still requires verification |
| **Track Separation** | AUDIO_TO_CUSTOMER, AUDIO_FROM_CUSTOMER | AWS Docs | ✅ Confirmed |
| **Integration Method** | Contact Flow + Lambda trigger | AWS Docs | ✅ Confirmed |
| **Contact Attributes** | streamARN, startFragmentNum, ContactId | AWS Docs | ✅ Confirmed |

### Subprocessors (from [Cresta Trust Center](https://trust.cresta.com/))

| Subprocessor | Purpose | Notes |
|--------------|---------|-------|
| **Fireworks.ai** | LLM model inference (Ocean-1) | Confirms architecture |
| **Deepgram** | ASR (speech-to-text) | Confirms voice stack |
| **OpenAI** | LLM services | Additional model provider |
| **Cartesia AI** | TTS (text-to-speech) | Voice synthesis |
| **ElevenLabs** | TTS (text-to-speech) | Added Aug 2025 |
| **Google Cloud** | Infrastructure | |
| **Datadog** | Monitoring | |
| **Segment** | Analytics | |

*Cresta has discontinued Mixpanel (Aug 2025), Linear, Atlassian, FullStory, MosaicML, Optimizely. Added GUIDEcx for onboarding.*

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
| 16 | **Backup RPO** - Recovery Point Objective (RTO 8h confirmed via Trust Center; RPO not stated) | 06 | DR planning |

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
   - Identify test scenarios ([12-poc-test-scenarios.md](12-poc-test-scenarios.md))
   - Establish success metrics
   - Review business use cases ([11-business-use-cases.md](11-business-use-cases.md)) and POC plan ([13-proof-of-concept-plan.md](13-proof-of-concept-plan.md))
   - Send [Feature Documentation Request](08-cresta-feature-documentation-request.md) and [Integration Guide Request](09-cresta-amazon-connect-integration-request.md) to Cresta
   - Schedule technical sessions per [10-cresta-technical-sessions-agenda.md](10-cresta-technical-sessions-agenda.md)

---

## Document Summaries by Area

### 01 - Overall Architecture
**Summary**: High-level system architecture showing integration between Amazon Connect, Cresta platform components (voice stack, ML services, data stores), and external AI services. Documents customer-specific subdomain routing, multi-tenant isolation, and end-to-end data flows.

**Key Components Documented**:
- Traffic management (DNS, ELB, NGINX Ingress + Wallarm WAF)
- Voice stack (gowalter, Deepgram ASR, apiserver)
- ML services (Orchestrator, ML Router, Ocean-1 model, LoRA adapters)
- Data layer (PostgreSQL, Redis Streams, Elasticsearch, ClickHouse, S3)
- External dependencies (Fireworks AI, Cartesia)

**Verification Status**: Architecture components align with confirmed Cresta blog and AWS documentation. Service names (gowalter, apiserver, clientsubscription) are consistent across documents but exact implementation details require Cresta confirmation.

### 02 - Amazon Connect Integration
**Summary**: Detailed integration architecture between Amazon Connect and Cresta, including Contact Flow configuration, KVS audio streaming, Lambda triggers, and authentication mechanisms.

**Key Integration Points**:
- Contact Flow blocks and execution order
- KVS audio stream consumption (method requires verification)
- Contact attribute mapping (system + custom)
- Authentication and security (method requires verification)
- Failover and error handling (behavior requires verification)

**Verification Status**: Contact Flow structure and KVS integration approach are documented per AWS patterns, but specific Cresta implementation details (Lambda vs direct API, auth method, failover behavior) require confirmation from Cresta.

**Critical Gap**: Audio format specification - AWS docs confirm PCM but do not explicitly state 8kHz sample rate. This should be verified with Cresta or AWS.

### 03 - Voice Stack Architecture
**Summary**: Audio processing pipeline from ingestion through ASR to transcript persistence, including WebSocket recovery mechanisms, PII redaction, and audio storage.

**Key Processing Stages**:
- Audio ingestion (20-100ms chunks)
- ASR processing (Deepgram, partial/final transcripts)
- Utterance building and speaker diarization
- Transcript persistence and event publishing
- Audio redaction and verification (Temporal workflows)

**Verification Status**: Processing pipeline structure is logical and consistent with real-time requirements. Deepgram as ASR provider is confirmed. Exact buffer sizes, recovery behavior, and multi-language support require Cresta confirmation.

### 04 - ML Services Architecture
**Summary**: ML inference architecture including policy-based model selection, inference graph execution, Ocean-1 foundation model with LoRA adapters, and batching strategies.

**Key ML Components**:
- Policy engine (agent/team/org-level policies)
- ML Router and shard management
- Model shards (GPU/CPU pods)
- Ocean-1 foundation model (Mistral 7B base, Fireworks hosting)
- LoRA adapters for customer-specific fine-tuning

**Verification Status**: Ocean-1 model details (Mistral 7B base, Fireworks hosting) confirmed via web search. Model inventory, latency SLAs, and scaling rules require Cresta confirmation.

### 05 - Real-Time Data Flows
**Summary**: Sequence diagrams for end-to-end real-time guidance, Knowledge Assist (RAG), call summarization, translation, agent app event subscription, and error recovery flows.

**Key Flows Documented**:
- Real-time guidance (target <1.5s latency)
- Knowledge Assist RAG (target <500ms)
- Call summarization (end-of-call)
- Translation flow (requires verification)
- Error recovery mechanisms

**Verification Status**: Flow sequences are logically structured. Latency targets are documented but should be verified against Cresta SLAs. Translation flow includes components (TTS, audio playback) that require verification.

### 06 - Data Architecture
**Summary**: Data storage architecture, customer data isolation, regional data residency, and data lifecycle management for the Cresta platform.

**Key Data Components**:
- Operational stores (PostgreSQL, Redis)
- Analytics stores (Elasticsearch, ClickHouse)
- Object storage (S3 for audio files and model artifacts)
- Customer data isolation (separate databases, S3 paths, ES indices per customer)
- Regional data residency (US, EU confirmed; APAC requires verification)
- Data retention and lifecycle management
- **RTO 8 hours** and data backups confirmed via [Cresta Trust Center](https://trust.cresta.com/)

**Verification Status**: Data isolation consistent with Cresta blog. RTO 8h confirmed via Trust Center. Data retention defaults, APAC availability, archive strategy, and RPO require Cresta confirmation.

### 07 - Security & Compliance Architecture
**Summary**: Security controls, compliance framework, PII redaction pipeline, and subprocessors for the Cresta platform. Key details from [Cresta Trust Center](https://trust.cresta.com/).

**Key Security & Compliance Elements**:
- Security controls: WAF, TLS, encryption at rest/transit, RBAC, IAM, **MFA** ✅
- Network: IDS/IPS, network vulnerability scanning; Endpoint: disk encryption, EDR, MDM
- **Risk Profile**: Data Access Level Internal, Impact Level Substantial, **RTO 8 hours**
- **Subprocessors**: Fireworks.ai, Deepgram, OpenAI, Cartesia AI, **ElevenLabs** (TTS; Aug 2025), Google Cloud, Datadog, Segment, GUIDEcx
- PII redaction (NER, regex, ML detection, audio beeps, text masking; Temporal verification)
- Compliance: SOC 2, ISO 27001/27701/42001, PCI-DSS, HIPAA, **TISAX, CCPA/CPRA, GDPR**

**Verification Status**: Certifications, MFA, RTO, subprocessors, and expanded security confirmed via Trust Center. **SSO**: PingFederate confirmed for client-side implementation. Other SSO providers and SIEM integration require Cresta confirmation.

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
