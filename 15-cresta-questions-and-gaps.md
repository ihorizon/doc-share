# Cresta Integration – What We Know vs. What We Need to Clarify

**Purpose**: Summary of confirmed information and outstanding questions requiring Cresta clarification for the Amazon Connect integration.

**Last Updated**: January 2025

---

## Executive Summary

Based on our technical assessment using public sources (Cresta blog, Trust Center, AWS documentation, SDK guide), we have validated significant portions of the Cresta architecture. However, several critical integration details remain undocumented and require direct clarification from Cresta before POC design can be finalized.

| Category | Status |
|----------|--------|
| **Confirmed Components** | 35+ items verified |
| **Partial Information** | 8 items with gaps |
| **Unknown / Requires Clarification** | 16 questions for Cresta |

---

## ✅ What We Know (Confirmed)

### Platform Infrastructure (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Cloud Provider** | AWS (EKS clusters) | Cresta blog, AWS ML Blog |
| **Customer Isolation** | Separate databases per customer | Cresta blog |
| **DNS Routing** | Customer-specific subdomains (`customer.region.cresta.ai`) | Cresta blog |
| **WAF** | Wallarm (NGINX Ingress-based) | Cresta blog |
| **API Endpoint Pattern** | `https://api-CUSTOMER_ID.cresta.com` | SDK Guide ✅ |
| **Auth Endpoint** | `https://auth.cresta.com` (default) | SDK Guide ✅ |

### Voice Stack (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Audio Handler** | gowalter service | Cresta blog |
| **ASR Provider** | Deepgram Nova-3 | Cresta blog |
| **ASR Latency** | Sub-300ms end-to-end | Deepgram docs, benchmarks |
| **Transcript Store** | PostgreSQL | Cresta blog |
| **Event Streaming** | Redis Streams | Cresta blog |
| **Audio Storage** | AWS S3 (encrypted) | Cresta blog |
| **Real-time Delivery** | ClientSubscription WebSocket service | SDK Guide ✅ |

### ML Services (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Foundation Model** | Ocean-1 (Mistral 7B fine-tuned) | Cresta blog, Fireworks AI |
| **Model Hosting** | Fireworks AI | Cresta blog, Fireworks AI |
| **Customization** | LoRA adapters per customer | Fireworks AI blog |
| **Cost Efficiency** | 100x reduction vs GPT-4 | Fireworks AI blog |
| **ML Framework** | PyTorch with TorchServe | AWS ML Blog |

### Amazon Connect Integration (High Confidence)

| Component | Detail | Source |
|-----------|--------|--------|
| **Audio Streaming** | Kinesis Video Streams (KVS) | AWS Docs |
| **Audio Format** | PCM linear16 (16-bit) | Confirmed |
| **Sample Rate** | 8kHz (telephony standard) | AWS Docs |
| **Track Separation** | `AUDIO_TO_CUSTOMER`, `AUDIO_FROM_CUSTOMER` | AWS Docs |
| **Integration Trigger** | Contact Flow + Lambda | AWS Docs |
| **Contact Attributes** | `streamARN`, `startFragmentNum`, `ContactId` | AWS Docs |

### Security & Compliance (High Confidence – via Trust Center)

| Component | Detail | Source |
|-----------|--------|--------|
| **Certifications** | SOC 2 Type II, ISO 27001/27701/42001, PCI-DSS, HIPAA | Trust Center |
| **Additional Compliance** | CCPA/CPRA, GDPR, TISAX | Trust Center |
| **RTO** | 8 hours | Trust Center |
| **MFA** | Multi-factor authentication supported | Trust Center |
| **Risk Profile** | Data Access: Internal; Impact: Substantial | Trust Center |
| **PII Redaction** | Audio beeps + text masking | Cresta blog |
| **Redaction Verification** | Temporal workflows | Cresta blog |

### SDK & Agent App (Confirmed via SDK Guide)

| Component | Detail | Source |
|-----------|--------|--------|
| **SDK Packages** | `@cresta/client` (0.21.0), `@cresta/react-client` (0.10.0) | SDK Guide |
| **NPM Registry** | `https://npm.cresta.com` | SDK Guide |
| **Voice Session Detection** | `voiceManager.onVoiceSession()` callback | SDK Guide |
| **Profile ID** | Required for chat and voice managers | SDK Guide |
| **Authentication Methods** | BYOID (OIDC), SAML, Twilio Flex token exchange | SDK Guide |
| **WebSocket Fallback** | SSE or polling when WebSocket disabled | SDK Guide |
| **Network Config** | Configurable timeout (default: browser), retry delay (default: 2s) | SDK Guide |

### Subprocessors (Confirmed via Trust Center)

| Subprocessor | Purpose |
|--------------|---------|
| Fireworks.ai | LLM model inference (Ocean-1) |
| Deepgram | ASR (speech-to-text) |
| OpenAI | LLM services |
| Cartesia AI | TTS (text-to-speech) |
| ElevenLabs | TTS (added Aug 2025) |
| Google Cloud | Infrastructure |
| Datadog | Monitoring |
| Segment | Analytics |

---

## ⚠️ Partial Information (Gaps Identified)

### 1. gowalter Audio Handler
**What we know**: Exists, processes 20-100ms audio chunks, handles WebSocket, performs buffering  
**Gap**: Exact WebSocket protocol, recovery buffer size, integration with KVS

### 2. PII Redaction
**What we know**: Auto-redaction for audio (beeps) and text, Temporal workflows for verification  
**Gap**: Detection model type (rule-based vs ML), entity types detected, false positive handling

### 3. Real-time Agent Guidance
**What we know**: Delivered via clientsubscription, ~1 update/second, uses Redis Streams  
**Gap**: Exact protocol (WebSocket vs SSE for agent app), offline/degraded behavior

### 4. Multi-Region Deployment
**What we know**: US and EU regions available  
**Gap**: APAC availability, cross-region data sync, disaster recovery architecture

### 5. SDK Guide Currency
**What we know**: Guide exists and was analyzed  
**Gap**: Age unknown – may not reflect current implementation

### 6. Amazon Connect-Specific Patterns
**What we know**: General SDK supports voice integrations  
**Gap**: Connect-specific integration examples, whether Connect uses different patterns than Twilio Flex

---

## ❓ Questions for Cresta (Prioritized)

### Priority 1: Blocking for POC Design

| # | Question | Impact | Risk if Unresolved |
|---|----------|--------|-------------------|
| **Q1** | **How does Cresta consume KVS streams?** Does Cresta use a Lambda consumer, direct GetMedia API, or another mechanism? | Integration architecture | Cannot design Contact Flow or Lambda setup |
| **Q2** | **How is KVS StreamARN correlated to ContactId in real-time?** AWS documentation indicates StreamARN is not available in contact attributes during flow execution – only in CTR after call ends. How does Cresta work around this? | Real-time processing | Critical timing issue for audio ingestion |
| **Q3** | **What is the Agent App deployment model for Amazon Connect?** Is it a browser extension, standalone desktop app, embedded in CCP, or SDK-based custom integration? | Agent rollout planning | Cannot plan agent deployment |
| **Q4** | **What authentication method is used between Connect/Lambda and Cresta endpoints?** API keys, OAuth, IAM cross-account AssumeRole, or other? | Security configuration | Cannot configure IAM/secrets |
| **Q5** | **What custom Contact Attributes are required?** What specific attributes must be set in the Contact Flow for Cresta to function? | Contact Flow design | Cannot build Contact Flow |

### Priority 2: Important for Security Review

| # | Question | Impact | Risk if Unresolved |
|---|----------|--------|-------------------|
| **Q6** | **What IAM permissions are required for Cresta to access customer KVS streams?** Is it cross-account AssumeRole with external ID, or customer-deployed components? | Security architecture | Security review blocked |
| **Q7** | **How are API credentials managed and rotated?** What is the credential lifecycle? | Security operations | Compliance gap |
| **Q8** | **What PII entity types are detected?** SSN, credit card, phone, address, health info, etc.? | Compliance validation | Cannot verify PCI/HIPAA compliance |
| **Q9** | **What is the encryption status?** Is data encrypted at rest in all Cresta infrastructure components? | Data protection | Security assessment incomplete |

### Priority 3: Important for Operational Readiness

| # | Question | Impact | Risk if Unresolved |
|---|----------|--------|-------------------|
| **Q10** | **What happens if Cresta is unreachable during a call?** Does the call continue without AI assistance (graceful degradation)? Does audio buffer locally? What is the agent experience? | SLA design, resilience | Cannot define SLAs or failover procedures |
| **Q11** | **What latency SLAs does Cresta provide?** Published guarantees for ASR, ML inference, end-to-end guidance? | Performance expectations | Cannot set customer expectations |
| **Q12** | **What CloudWatch metrics should be monitored?** Are there Cresta-specific metrics exposed? Pre-built dashboards? Recommended alert thresholds? | Operations readiness | Monitoring gaps |
| **Q13** | **What is the RPO (Recovery Point Objective)?** RTO is 8 hours per Trust Center, but RPO is not stated. | DR planning | Cannot complete DR plan |

### Priority 4: Nice to Have

| # | Question | Impact | Risk if Unresolved |
|---|----------|--------|-------------------|
| **Q14** | **Is APAC (AP-Southeast-1) region available?** For future regional expansion. | Global deployment | Not blocking for initial POC |
| **Q15** | **Can Amazon Transcribe be used instead of Deepgram?** Alternative ASR for cost/latency comparison. | Vendor flexibility | Not blocking |
| **Q16** | **What SSO providers are supported beyond PingFederate?** Okta, Azure AD, etc.? | Admin configuration | Not blocking |

---

## Recommended Next Steps

### 1. Send Questions to Cresta
- Prioritize Q1-Q5 for immediate response (blocking for POC design)
- Request technical documentation for integration guide and IAM policy templates
- Schedule technical deep-dive session

### 2. Request Documentation
- [ ] Amazon Connect integration guide (step-by-step)
- [ ] IAM policy templates for KVS access
- [ ] Contact Flow design patterns
- [ ] CloudWatch dashboard templates
- [ ] Troubleshooting guide

### 3. Schedule Technical Sessions
- **Session 1**: Integration Architecture (Q1-Q5)
- **Session 2**: Security & Compliance (Q6-Q9)
- **Session 3**: Operations & Monitoring (Q10-Q13)

---

## Summary Matrix

| Area | Confirmed | Partially Known | Unknown |
|------|-----------|-----------------|---------|
| **Platform Infrastructure** | ✅ 6 items | — | — |
| **Voice Stack** | ✅ 7 items | gowalter details | KVS consumption method |
| **ML Services** | ✅ 5 items | — | Model inventory, scaling rules |
| **Amazon Connect Integration** | ✅ 6 items | — | StreamARN correlation, auth |
| **Security & Compliance** | ✅ 8 items | PII detection details | IAM architecture |
| **Agent App / SDK** | ✅ 8 items | SDK currency | Connect deployment model |
| **Operations** | — | Monitoring approach | Failover, SLAs, RPO |

---

## References

- [00-summary-and-followups.md](00-summary-and-followups.md) – Full follow-up items list
- [00-research-findings-summary.md](00-research-findings-summary.md) – Research verification details
- [14-cresta-sdk-developer-guide-analysis.md](14-cresta-sdk-developer-guide-analysis.md) – SDK guide analysis
- [references.md](references.md) – All source materials
- [08-cresta-feature-documentation-request.md](08-cresta-feature-documentation-request.md) – Feature documentation request
- [09-cresta-amazon-connect-integration-request.md](09-cresta-amazon-connect-integration-request.md) – Integration guide request
- [Cresta Trust Center](https://trust.cresta.com/) – Compliance and security details
