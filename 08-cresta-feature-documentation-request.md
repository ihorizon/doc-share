# Cresta AI – Feature Documentation Request

**Purpose**: Request to support technical assessment and POC planning. We need a complete understanding of Cresta's capabilities to design test scenarios and assess platform value.

**Context**: Technical assessment of Cresta AI and its Amazon Connect integration. Architecture is documented; specific feature details are needed to determine what to test in proof of concept.

---

## 1. Feature Catalog & Capabilities

Please provide:

| # | Request | Purpose |
|---|---------|---------|
| 1.1 | **Complete feature list** – All capabilities available in the platform | Design POC test scenarios |
| 1.2 | **Agent assistance features** – Real-time hints, knowledge assist, script guidance, next-best-action | Prioritize POC feature tests |
| 1.3 | **Virtual agent capabilities** – If applicable, scope and limitations | Assess full platform scope |
| 1.4 | **Conversation intelligence** – Sentiment analysis, topic extraction, compliance monitoring, moment detection | Define analytics tests |
| 1.5 | **Analytics and reporting** – Dashboards, metrics, exports, custom reports | Assess monitoring for POC |
| 1.6 | **Customization options** – Guardrails, policies, workflows, no-code orchestration | Understand configuration scope |
| 1.7 | **Integration capabilities** – Beyond Amazon Connect (CRM, telephony, etc.) | Assess integration footprint |
| 1.8 | **Multi-channel support** – Voice, chat, email, SMS | Define channel coverage for POC |
| 1.9 | **Agent coaching and performance** – Coaching, insights, scorecards | Assess post-call capabilities |

---

## 2. Model & AI Capabilities

| # | Request | Purpose |
|---|---------|---------|
| 2.1 | **Model inventory** – List of ML models and their purposes (intent, sentiment, NER, etc.) | Map capabilities to test scenarios |
| 2.2 | **Model accuracy and performance** – Published or typical metrics per model type | Set POC success criteria |
| 2.3 | **Customization** – Fine-tuning, custom models, LoRA adapters, customer data usage | Assess fit for our use cases |
| 2.4 | **Knowledge base integration** – RAG, indexing, semantic search, article management | Design knowledge-assist tests |
| 2.5 | **Language support** – Languages, dialect handling, language detection | Plan multi-language POC if needed |
| 2.6 | **Model update process** – Frequency, rollout, versioning, rollback | Assess operational impact |

---

## 3. Agent Application

| # | Request | Purpose |
|---|---------|---------|
| 3.1 | **Deployment method** – Browser extension, desktop app, web-based, embedded CCP | Plan agent rollout for POC |
| 3.2 | **Installation and configuration** – Step-by-step, prerequisites, system requirements | Estimate POC setup effort |
| 3.3 | **Agent interface features** – Real-time guidance, transcript view, suggestions, warnings | Design agent-side test cases |
| 3.4 | **Agent interaction capabilities** – What agents can do within the app (e.g., accept/reject hints) | Define user workflows |

---

## 4. Testing & Simulation

| # | Request | Purpose |
|---|---------|---------|
| 4.1 | **Testing and simulation tools** – What is available (e.g., simulator visitors, LLM judges) | Plan validation approach for POC |
| 4.2 | **Creating test scenarios** – How to define and run test cases | Design POC test scenarios |
| 4.3 | **Synthetic call generation** – Capabilities, supported scenarios | Plan load and regression testing |
| 4.4 | **Evaluation and scoring** – Turn-level, conversation-level, regression/drift checks | Define POC success metrics |
| 4.5 | **A/B testing** – How traffic splitting and evaluation work | Assess experimentation capability |
| 4.6 | **Test data management** – Use of production-like data, anonymization | Ensure compliant POC testing |

---

## 5. Configuration & Setup

| # | Request | Purpose |
|---|---------|---------|
| 5.1 | **Initial setup and provisioning** – Steps, roles, environments | Estimate POC timeline |
| 5.2 | **Agent onboarding** – Process to add agents, assign profiles/teams | Plan pilot agent cohort |
| 5.3 | **Policy and guardrail configuration** – How to configure policies, guardrails, rules | Design configuration tests |
| 5.4 | **Model configuration** – Thresholds, triggers, enabled features per agent/team | Map to POC scenarios |
| 5.5 | **Configuration templates or examples** – Sample configs for common use cases | Accelerate POC setup |
| 5.6 | **Testing and validation procedures** – How to validate config before production | Define POC go-live checks |

---

## 6. Use Case Examples

| # | Request | Purpose |
|---|---------|---------|
| 6.1 | **Use case documentation** – Documented scenarios (e.g., customer service, sales, compliance) | Align POC with our use cases |
| 6.2 | **Feature-to-use-case mapping** – Which features support which scenarios | Prioritize POC scope |
| 6.3 | **Sample workflows** – End-to-end examples (e.g., knowledge assist, compliance monitoring) | Design realistic POC tests |

---

## 7. Optional: Performance & Monitoring

| # | Request | Purpose |
|---|---------|---------|
| 7.1 | **Available dashboards and metrics** – Real-time and historical | Define POC monitoring |
| 7.2 | **Business metrics** – Handle time, resolution rate, containment, etc. | Align POC success criteria |
| 7.3 | **Alerting** – What can be alerted on, how to configure | Plan POC incident handling |

---

## Deliverables Requested

- **Feature catalog** (or equivalent) covering sections 1–3.
- **Model and AI capabilities** overview (section 2).
- **Agent application** deployment and usage guide (section 3).
- **Testing guide** describing tools and procedures (section 4).
- **Configuration guide** for setup, policies, and guardrails (section 5).
- **Use case examples** and, if available, feature-to-use-case mapping (section 6).

**Format**: PDF or web documentation links are acceptable.

---

## Reference

This request supports the technical assessment and POC planning described in:

- [00-summary-and-followups.md](00-summary-and-followups.md) – Architecture summary and follow-ups
- [Cresta-review.md](Cresta-review.md) – Executive assessment
- [references.md](references.md) – Central index of all material references
- **Cresta Client SDK Developer Guide** – We have `Cresta Client SDK Developer Guide.pdf` locally (age unknown). Request latest SDK/API docs if different; confirm currency with Cresta.
- Gap Analysis: Cresta AI Technical Assessment for POC Planning
