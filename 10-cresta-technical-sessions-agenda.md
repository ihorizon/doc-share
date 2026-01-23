# Cresta AI – Technical Sessions Agenda & Prep

**Purpose**: Prepare for technical sessions with Cresta to complete our assessment and POC planning. Use this agenda to schedule and run sessions, and to track open questions.

**Context**: Technical assessment of Cresta AI and its Amazon Connect integration. We have documented architecture and flows; we need clarifications, demos, and collaborative POC scoping.

---

## Session 1: Architecture Deep-Dive & Open Questions

**Duration**: 60–90 minutes  
**Attendees**: Our technical/architecture team, Cresta solutions engineering or architecture  
**Goal**: Clarify architecture assumptions, resolve open items from our docs, align on integration model.

### Agenda

| # | Topic | Objective | References |
|---|--------|-----------|------------|
| 1.1 | **Architecture overview** | Confirm high-level flow: Connect → KVS → Cresta (gowalter, ASR, ML, Agent App). Validate our [01-overall-architecture](01-overall-architecture.md) and [05-realtime-dataflow-sequences](05-realtime-dataflow-sequences.md). | 01, 05 |
| 1.2 | **Integration method** | Confirm: Lambda consumer vs direct GetMedia API vs other. Who initiates connection to KVS? | 02, 00-summary |
| 1.3 | **Authentication** | API keys, OAuth, IAM, or other for Connect/Lambda ↔ Cresta. How are credentials managed? | 02, 06 |
| 1.4 | **Failover & resilience** | If Cresta is unreachable, does the call continue? Transcript gaps? Agent experience? | 02, 05 |
| 1.5 | **Agent App** | Deployment (extension, desktop, web). How it connects to CCP. Audio source (KVS vs desktop capture) for Connect. | 01, 02 |
| 1.6 | **External dependencies** | Deepgram, Fireworks, etc. Fallbacks, SLAs, regional constraints. | 03, 04 |

### Questions to Cover

- KVS consumer: Lambda, direct API, or hybrid? Exact flow?
- Required Contact attributes (system + custom) and where they are set.
- Cross-account KVS access: IAM model if Cresta reads from our AWS.
- gowalter recovery: buffer size, replay behavior, impact on transcript continuity.
- Multi-region: how Cresta routes traffic when we have multiple Connect instances/regions.

### Prep

- Review [00-summary-and-followups.md](00-summary-and-followups.md) “Items Requiring Follow-Up” and “Risk Summary.”
- Review [02-amazon-connect-integration.md](02-amazon-connect-integration.md) “Items Requiring Follow-up.”
- Prepare list of our Connect instance config (regions, cross-account if any).

---

## Session 2: Integration Walkthrough & Demo

**Duration**: 60–90 minutes  
**Attendees**: Our integration/ops team, Cresta solutions engineering  
**Goal**: Live walkthrough of Amazon Connect + Cresta integration; understand setup and validation.

### Agenda

| # | Topic | Objective |
|---|--------|-----------|
| 2.1 | **Contact Flow walkthrough** | Step-by-step Contact Flow: blocks, order, attributes, Lambda trigger. |
| 2.2 | **Lambda & AWS** | Lambda role, KVS permissions, DynamoDB (if used). How Lambda passes data to Cresta. |
| 2.3 | **Cresta configuration** | Tenant, endpoints, credentials. What we configure in Cresta for Connect. |
| 2.4 | **End-to-end demo** | Live test call: Connect → KVS → Cresta → transcript + Agent App. |
| 2.5 | **Validation checklist** | How to verify integration (config checks, test calls, Cresta UI). |
| 2.6 | **Troubleshooting** | Common failures (e.g. no transcript, wrong customer), logs, escalation. |

### Questions to Cover

- Exact Contact Flow export or template for Connect + Cresta.
- Validation steps we should run before POC go-live.
- Where to look for errors (CloudWatch, Connect, Cresta).

### Prep

- Ensure we have (or will have) Connect instance + test queue for demo.
- Prepare optional screen-share of our Connect setup if useful.

---

## Session 3: Feature Demonstration & Capabilities

**Duration**: 60–90 minutes  
**Attendees**: Our product/ops + technical teams, Cresta product/solutions  
**Goal**: See key features in action; understand scope for POC test scenarios.

### Agenda

| # | Topic | Objective |
|---|--------|-----------|
| 3.1 | **Agent assistance** | Real-time hints, knowledge assist, next-best-action. When they fire, how they appear in Agent App. |
| 3.2 | **Conversation intelligence** | Sentiment, topics, compliance moments. Dashboards and reports. |
| 3.3 | **Guardrails & policies** | Configuring policies, guardrails, compliance rules. Examples (e.g. no PCI, no diagnoses). |
| 3.4 | **Knowledge base & RAG** | How knowledge is ingested, updated, and used for assist. |
| 3.5 | **Analytics & reporting** | Key metrics, dashboards, exports. How we’d measure POC success. |
| 3.6 | **Testing & simulation** | Simulator visitors, test scenarios, evaluation. How we can test before production. |

### Questions to Cover

- Which capabilities are included in standard vs add-on?
- Limitations we should know for POC (e.g. channels, languages, custom models).
- Typical time to value for setup and first useful results.

### Prep

- Review [08-cresta-feature-documentation-request.md](08-cresta-feature-documentation-request.md).
- Draft initial list of POC use cases (e.g. support, sales, compliance).

---

## Session 4: POC Planning & Test Scenarios

**Duration**: 60 minutes  
**Attendees**: Our POC owner, technical lead, Cresta CSM or solutions  
**Goal**: Align on POC scope, success criteria, timeline, and test scenarios.

### Agenda

| # | Topic | Objective |
|---|--------|-----------|
| 4.1 | **POC scope** | Agents, queues, call volume, duration. In-scope vs out-of-scope. |
| 4.2 | **Success criteria** | Metrics (e.g. latency, containment, resolution, adoption). Qualitative goals. |
| 4.3 | **Test scenarios** | Specific scenarios to run (e.g. knowledge assist, compliance, multi-turn). See [12-poc-test-scenarios.md](12-poc-test-scenarios.md) and [13-proof-of-concept-plan.md](13-proof-of-concept-plan.md). |
| 4.4 | **Timeline** | Setup, config, pilot, evaluation. Milestones and dependencies. |
| 4.5 | **Support** | Who we contact during POC. Response expectations. |
| 4.6 | **Next steps** | Documentation access, environment provisioning, kickoff. |

### Questions to Cover

- Recommended POC duration and agent count.
- Cresta-led vs customer-led setup and configuration.
- How we get feature docs, integration guide, and API docs (if needed).

### Prep

- Review [12-poc-test-scenarios.md](12-poc-test-scenarios.md) and [13-proof-of-concept-plan.md](13-proof-of-concept-plan.md).
- Draft success criteria and POC timeline.
- Confirm attendees and roles on our side.

---

## Session Scheduling Checklist

- [ ] Session 1: Architecture deep-dive – scheduled  
- [ ] Session 2: Integration walkthrough & demo – scheduled  
- [ ] Session 3: Feature demonstration – scheduled  
- [ ] Session 4: POC planning – scheduled  
- [ ] Cresta contacts (SE, CSM) confirmed  
- [ ] Our attendees and roles confirmed  
- [ ] Prep completed for each session  
- [ ] Request sent: [08-cresta-feature-documentation-request](08-cresta-feature-documentation-request.md)  
- [ ] Request sent: [09-cresta-amazon-connect-integration-request](09-cresta-amazon-connect-integration-request.md)  

---

## Reference

- [00-summary-and-followups.md](00-summary-and-followups.md) – Follow-ups and risks  
- [01-overall-architecture.md](01-overall-architecture.md) – Overall architecture  
- [02-amazon-connect-integration.md](02-amazon-connect-integration.md) – Connect integration  
- [05-realtime-dataflow-sequences.md](05-realtime-dataflow-sequences.md) – Real-time flows  
- [08-cresta-feature-documentation-request.md](08-cresta-feature-documentation-request.md) – Feature request  
- [09-cresta-amazon-connect-integration-request.md](09-cresta-amazon-connect-integration-request.md) – Integration request  
- [11-business-use-cases.md](11-business-use-cases.md) – Business use cases
- [12-poc-test-scenarios.md](12-poc-test-scenarios.md) – Technical test scenarios
- [13-proof-of-concept-plan.md](13-proof-of-concept-plan.md) – Business-focused POC plan
