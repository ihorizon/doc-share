# References – Cresta AI & Amazon Connect Documentation

This document centralizes all material references used across the architecture and assessment documentation. It indicates what was referenced where and helps validate sources.

---

## 1. Official Cresta Sources

### 1.1 Cresta Trust Center

| URL | Description | What Was Referenced |
|-----|-------------|---------------------|
| [https://trust.cresta.com/](https://trust.cresta.com/) | Cresta Trust Center (SafeBase) | Compliance certifications (SOC 2, ISO 27001/27701/42001, PCI-DSS, HIPAA, TISAX, CCPA/CPRA, GDPR); RTO 8 hours; Risk Profile (Data Access: Internal, Impact: Substantial); MFA; subprocessors (Fireworks.ai, Deepgram, OpenAI, Cartesia AI, ElevenLabs, Google Cloud, Datadog, Segment, GUIDEcx); discontinued services (Mixpanel, Linear, Atlassian, FullStory, MosaicML, Optimizely); security (IDS/IPS, network vulnerability scanning, disk encryption, EDR, MDM); data backups |

**Documents referencing Trust Center:**  
00-summary-and-followups, 01-overall-architecture, 06-data-architecture, 07-security-compliance-architecture, Cresta-review

---

### 1.2 Cresta Blog

| Source | Description | What Was Referenced |
|--------|-------------|---------------------|
| Cresta blog (general) | Product and platform posts | Platform hosting (AWS EKS); customer isolation (separate databases); DNS routing (customer.region.cresta.ai); WAF (Wallarm/NGINX); gowalter; Deepgram ASR; PostgreSQL; Redis Streams; Elasticsearch; ClickHouse; S3; Ocean-1; Fireworks AI; LoRA; PII redaction (audio beeps + text); Temporal workflows |
| [Understanding Cresta's Voice Platform - The Voice Stack](https://cresta.com/blog/understanding-crestas-voice-platform-the-voice-stack/) | Voice stack overview | gowalter, ASR, apiservice, orchestration; Agent App audio capture when CCaaS doesn’t provide streams |
| [Introducing Ocean-1](https://cresta.com/blog/introducing-ocean-1-worlds-first-contact-center-foundation-model/) | Ocean-1 foundation model | Mistral 7B base; contact-center fine-tuning; Fireworks hosting; LoRA adapters |
| [How fine-tuned LLMs power knowledge assist, summarization, chat suggestions](https://cresta.com/blog/how-fine-tuned-llms-power-knowledge-assist-summarization-and-chat-suggestions/) | LLM use cases | Ocean-1; Knowledge Assist; summarization; chat suggestions |
| [Building and Deploying Production-Grade AI Agents](https://cresta.com/blog/building-and-deploying-production-grade-ai-agents) | AI agents | Architecture, deployment |
| [Cresta's Three Strategic Pillars of AI Agent Defense](https://cresta.com/blog/crestas-three-strategic-pillars-of-ai-agent-defense) | Guardrails | System-level, supervisory, adversarial testing |
| [Platform Auth, APIs and More](https://cresta.com/blog/platform-auth-apis) | Auth & APIs | Client layer (SDK/APIs); Edge layer; Service layer |

**Documents referencing Cresta blog:**  
00-summary-and-followups, 01-overall-architecture, 02-amazon-connect-integration, 03-voice-stack-architecture, 04-ml-services-architecture, 05-realtime-dataflow-sequences, 06-data-architecture, 07-security-compliance-architecture, Cresta-review

---

### 1.3 Cresta Client SDK Developer Guide (Local)

| File | Description | What Was Referenced | Notes |
|------|-------------|---------------------|-------|
| `Cresta Client SDK Developer Guide.pdf` | Client SDK developer guide | **✅ Analyzed** – Text extracted via pdftotext. See [14-cresta-sdk-developer-guide-analysis.md](14-cresta-sdk-developer-guide-analysis.md) | Exported from Google Docs (Skia/PDF renderer). **Age unknown** – confirm currency with Cresta. **Key findings**: ClientSubscription WebSocket confirmed; API endpoints (`api-CUSTOMER_ID.cresta.com`); BYOID auth (OIDC); voice session detection; Profile ID required; SDK packages/versions. |

**Documents referencing SDK guide:**  
references.md (this file). Recommend adding to 02-amazon-connect-integration, 08-feature-request, 10-technical-sessions when used.

---

## 2. AWS & Amazon Connect

### 2.1 AWS Documentation

| Source | Description | What Was Referenced |
|--------|-------------|---------------------|
| Amazon Connect – live media streaming | [Enable live media streaming](https://docs.aws.amazon.com/connect/latest/adminguide/enable-live-media-streams.html), [Access media stream data](https://docs.aws.amazon.com/connect/latest/adminguide/access-media-stream-data.html) | Kinesis Video Streams (KVS); **PCM linear16 (16-bit PCM)** ✅ confirmed; tracks AUDIO_TO_CUSTOMER, AUDIO_FROM_CUSTOMER; Contact Flow + Lambda |
| Amazon Connect – Contact Flow | Contact Flow blocks | Start Media Streaming; Set Contact Attributes; Invoke Lambda; streamARN, startFragmentNum, ContactId |

**Documents referencing AWS docs:**  
00-summary-and-followups, 02-amazon-connect-integration, 05-realtime-dataflow-sequences

**Note:** ✅ **Audio encoding confirmed**: PCM linear16 (16-bit PCM). Sample rate (8 kHz) still requires verification.

---

### 2.2 AWS Machine Learning Blog

| Source | Description | What Was Referenced |
|--------|-------------|---------------------|
| [Evolution of Cresta's ML Architecture: Migration to AWS and PyTorch](https://aws.amazon.com/blogs/machine-learning/evolution-of-crestas-machine-learning-architecture-migration-to-aws-and-pytorch) | Cresta ML on AWS | AWS consolidation; PyTorch; TorchServe; EKS; EC2/Spot; Aurora; S3; 20+ models; intent, NER, etc. |

**Documents referencing AWS ML blog:**  
00-summary-and-followups, Cresta-review, 04-ml-services-architecture (indirect)

---

## 3. GitHub – Cresta Organization

| URL | Description | What Was Referenced |
|-----|-------------|---------------------|
| [https://github.com/cresta](https://github.com/cresta) | Cresta Intelligence, Inc. (verified cresta.com, cresta.ai) | Org-level validation |
| [cresta/amazon-connect-pstn-transfer](https://github.com/cresta/amazon-connect-pstn-transfer) | AWS resources for PSTN-only transfer with Amazon Connect | **Relevant to Connect integration** – use for Connect + Cresta PSTN transfer scenarios |
| [cresta/CrestaVoice-TwilioFlexPluginIntegration-FollowTheAgent](https://github.com/cresta/CrestaVoice-TwilioFlexPluginIntegration-FollowTheAgent) | Twilio Flex + Cresta Voice integration | Example of CCaaS + Cresta Voice / Agent App–style integration |
| atlantis-drift-detection, helm-autoupdate, gitdb, pipe, gogit, etc. | Internal tooling (Go, Terraform, GitOps) | **Not** used in architecture docs; shows Cresta uses Go, Terraform, GitOps |

**Documents referencing GitHub:**  
references.md (this file). Recommend referencing `amazon-connect-pstn-transfer` in 02-amazon-connect-integration, 09-integration-request.

**Validation:**  
- GitHub org verified. `amazon-connect-pstn-transfer` confirms Cresta publishes Amazon Connect–related assets.  
- No public “Client SDK” repo; SDK likely proprietary, with usage described in `Cresta Client SDK Developer Guide.pdf`.

---

## 4. External Services (Subprocessors)

| Service | Purpose | Source | Documents |
|---------|---------|--------|-----------|
| Fireworks.ai | LLM model inference (Ocean-1) | Trust Center, Cresta blog | 01, 04, 00-summary, 07 |
| Deepgram | ASR (speech-to-text) | Trust Center, Cresta blog | 01, 03, 05, 00-summary, 07 |
| OpenAI | LLM services | Trust Center | 00-summary, 07 |
| Cartesia AI | TTS (text-to-speech) | Trust Center, architecture docs | 01, 00-summary, 07 |
| ElevenLabs | TTS (text-to-speech); added Aug 2025 | Trust Center | 01, 00-summary, 07 |
| Google Cloud, Datadog, Segment, GUIDEcx | Infrastructure, monitoring, analytics, onboarding | Trust Center | 00-summary, 07 |

---

## 5. Client-Side Configuration (Implementation-Specific)

| Configuration | Value | Source | Notes |
|---------------|-------|--------|-------|
| **SSO Provider** | PingFederate | Client implementation | ✅ Confirmed for this implementation. Verify which other SSO providers Cresta supports if needed. |

---

## 6. Other Uploaded / Local Material

| File | Description | What Was Referenced |
|------|-------------|---------------------|
| `cresta_ai_technical_assessment.md.pdf` | Technical assessment (markdown exported to PDF) | Content not cross-checked in this references pass; use for alignment with Cresta-review and architecture docs |
| `01-01-2025 - Applicant 1 - Application (2).pdf` | Other PDF | Not used in architecture or references |

---

## 7. Summary by Document

| Document | Primary References |
|----------|--------------------|
| **00-summary-and-followups** | Trust Center, Cresta blog, AWS docs, Fireworks, Deepgram |
| **01-overall-architecture** | Cresta blog, Trust Center, Deepgram, Fireworks, Cartesia, ElevenLabs |
| **02-amazon-connect-integration** | AWS docs, Cresta blog; recommend adding SDK guide, GitHub `amazon-connect-pstn-transfer` |
| **03-voice-stack-architecture** | Cresta blog, Deepgram |
| **04-ml-services-architecture** | Cresta blog, Fireworks, Ocean-1, web search |
| **05-realtime-dataflow-sequences** | Cresta blog, Deepgram, Fireworks |
| **06-data-architecture** | Cresta blog, Trust Center |
| **07-security-compliance-architecture** | Trust Center (certifications, subprocessors, security, RTO, risk profile) |
| **Cresta-review** | Trust Center, Cresta blog, AWS ML blog, web search |
| **14-cresta-sdk-developer-guide-analysis** | SDK guide (extracted text), architecture docs (for validation) |
| **08–13** (requests, agenda, POC) | Point to architecture docs and Trust Center; feature/integration details from Cresta |

---

## 8. Gaps and Recommended Actions

1. **Cresta Client SDK Developer Guide**  
   - **Status:** ✅ **Analyzed** – Text extracted via pdftotext; analysis in [14-cresta-sdk-developer-guide-analysis.md](14-cresta-sdk-developer-guide-analysis.md).  
   - **Action:** Extract or re-export as text; confirm with Cresta that it’s the current SDK guide; then reference specific sections in 02, 08, 10.

2. **GitHub `amazon-connect-pstn-transfer`**  
   - **Status:** Not yet cited in architecture docs.  
   - **Action:** Add to 02-amazon-connect-integration and 09-integration-request as a relevant Cresta-owned Connect resource.

3. **Audio Format**  
   - **Status:** ✅ **Confirmed**: PCM linear16 (16-bit PCM) encoding. Sample rate (8 kHz) still requires verification.  
   - **Action:** Verify sample rate (8 kHz) with Cresta or AWS; encoding format is now confirmed.

4. **Twilio Flex integration repo**  
   - **Status:** Useful as integration pattern reference.  
   - **Action:** Optionally add to references and “integration examples” in 02 or 09.

---

## 9. Quick Links

- [Cresta Trust Center](https://trust.cresta.com/)
- [Cresta GitHub](https://github.com/cresta)
- [Cresta – Amazon Connect PSTN Transfer](https://github.com/cresta/amazon-connect-pstn-transfer)
- [Cresta Voice – Twilio Flex Integration](https://github.com/cresta/CrestaVoice-TwilioFlexPluginIntegration-FollowTheAgent)
- [Cresta Blog – Voice Stack](https://cresta.com/blog/understanding-crestas-voice-platform-the-voice-stack/)
- [Cresta Blog – Ocean-1](https://cresta.com/blog/introducing-ocean-1-worlds-first-contact-center-foundation-model/)
- [AWS ML Blog – Cresta ML Architecture](https://aws.amazon.com/blogs/machine-learning/evolution-of-crestas-machine-learning-architecture-migration-to-aws-and-pytorch)
- [Amazon Connect – Live Media Streaming](https://docs.aws.amazon.com/connect/latest/adminguide/access-media-stream-data.html)
