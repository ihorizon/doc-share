# Cresta AI + Amazon Connect – POC Test Scenarios

**Purpose**: Define test scenarios, success criteria, and validation approach for the Cresta AI proof of concept with Amazon Connect.

**Context**: Technical assessment of Cresta AI and its Amazon Connect integration. Use these scenarios to validate integration, features, and performance during POC. Update as feature documentation and integration details are received (see [08-cresta-feature-documentation-request](08-cresta-feature-documentation-request.md), [09-cresta-amazon-connect-integration-request](09-cresta-amazon-connect-integration-request.md)).

---

## 1. POC Scope & Success Criteria

### 1.1 POC Scope (Define with Cresta)

| Element | Proposed | Notes |
|--------|----------|-------|
| **Duration** | _TBD_ | e.g. 4–8 weeks |
| **Agents** | _TBD_ | e.g. 5–15 pilot agents |
| **Queues** | _TBD_ | Specific Connect queues in scope |
| **Channels** | Voice (Connect) | Chat/email if in scope |
| **Call volume** | _TBD_ | e.g. X calls/day |
| **Features in scope** | Integration, Agent Assist, Knowledge Assist, Conversation Intelligence | Refine from feature catalog |

### 1.2 Success Criteria

**Integration**

| # | Criterion | Target | How to Measure |
|---|-----------|--------|----------------|
| I1 | Connect → Cresta audio flow | Stream established for each call | Cresta shows active stream / transcript per contact |
| I2 | Transcript availability | Transcript visible in Agent App during call | Manual check + spot checks |
| I3 | Call lifecycle | Stream start/stop aligned with contact | No orphaned streams; clean stop on disconnect |
| I4 | Integration errors | &lt; 1% of calls with integration failure | Logs, Cresta/Connect reporting |

**Performance**

| # | Criterion | Target | How to Measure |
|---|-----------|--------|----------------|
| P1 | End-to-end latency (speech → guidance) | &lt; 1.5 s | Timestamp customer utterance vs guidance display; sample N calls |
| P2 | Agent assist suggestion latency | &lt; 200 ms (p95) | Per Cresta targets; validate in POC |
| P3 | Knowledge Assist response | &lt; 500 ms | Question detected → suggestion displayed |
| P4 | Transcript partial latency | 200–300 ms (first partial) | Per voice stack docs; sample checks |
| P5 | System availability | 99.5%+ during POC | Uptime / failed calls due to Cresta |

**Feature**

| # | Criterion | Target | How to Measure |
|---|-----------|--------|----------------|
| F1 | Agent Assist | Hints/suggestions displayed when relevant | Manual review of sample calls |
| F2 | Knowledge Assist | Correct KB articles surfaced for common questions | Test script + agent feedback |
| F3 | Conversation intelligence | Sentiment/topics available post-call | Dashboards, exports |
| F4 | Guardrails | Policies fire as configured (e.g. no PCI) | Deliberate test scenarios |
| F5 | Agent adoption | Agents use guidance in &gt; X% of applicable situations | Survey + spot checks |

**Operational**

| # | Criterion | Target | How to Measure |
|---|-----------|--------|----------------|
| O1 | Setup time | Environment ready within _TBD_ days | Timeline tracking |
| O2 | Config changes | Policy/guardrail updates without downtime | Change during POC |
| O3 | Support | Response within _TBD_ for POC issues | Ticket tracking |

---

## 2. Integration Test Scenarios

**Goal**: Validate Amazon Connect ↔ Cresta integration end-to-end.

### 2.1 Connect → KVS → Cresta Audio Flow

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| INT-1 | Inbound call – stream established | 1. Place test call to Connect. 2. Contact Flow runs (Start Media Streaming, etc.). 3. Call routed to agent. | KVS stream active; Cresta receives audio; transcript appears in Agent App. | |
| INT-2 | Dual-track audio | 1. Agent and customer speak. 2. Verify diarization. | Customer vs agent utterances correctly attributed in transcript. | |
| INT-3 | Call disconnect – clean stop | 1. End call from customer or agent. 2. Verify Contact Flow (Stop Media Streaming). | Cresta stream stops; no duplicate or orphaned sessions; call summarization triggers. | |
| INT-4 | Multiple concurrent calls | 1. Place 3–5 simultaneous test calls. 2. All routed to agents with Cresta. | Each contact has distinct transcript; no cross-talk or session mix-up. | |

### 2.2 Authentication & Connectivity

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| INT-5 | Auth validation | 1. Confirm auth method (API key, IAM, etc.). 2. Make test call. | Cresta accepts connection; transcript flows. | |
| INT-6 | Invalid/missing credentials | 1. Simulate misconfigured auth (per Cresta guidance). 2. Place call. | Call continues in Connect; Cresta fails gracefully; clear error visibility. | |
| INT-7 | Cresta unreachable | 1. Simulate Cresta outage (or disconnect). 2. Place call. | Call continues; agent can work without Cresta; behaviour matches failover docs. | |

### 2.3 Contact Attributes & Configuration

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| INT-8 | Required attributes | 1. Set all required Contact attributes per integration guide. 2. Place call. | Cresta receives attributes; correct tenant/profile; transcript tied to contact. | |
| INT-9 | Missing custom attribute | 1. Omit one custom attribute (e.g. CrestaProfileId). 2. Place call. | Defined behaviour (e.g. default profile or clear error); no silent failure. | |
| INT-10 | Multi-queue / multi-profile | 1. Route calls from different queues with different Cresta profiles. 2. Verify. | Correct profile per queue; policies/guardrails align. | |

---

## 3. Feature Test Scenarios

**Goal**: Validate Cresta capabilities in scope for POC. Adjust based on [08-cresta-feature-documentation-request](08-cresta-feature-documentation-request.md) and actual features.

### 3.1 Agent Assist (Real-Time Guidance)

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| FEAT-1 | Intent-based hint | 1. Customer states clear intent (e.g. “I want to cancel”). 2. Agent continues conversation. | Relevant hint/suggestion appears in Agent App within latency target. | |
| FEAT-2 | Knowledge Assist trigger | 1. Customer asks a question covered in KB. 2. Agent waits for suggestion. | KB-based suggestion appears; agent can use it in response. | |
| FEAT-3 | Next-best-action | 1. Handle typical call flow (e.g. inquiry → resolution → wrap-up). | Contextual next-best-action or wrap-up prompts appear. | |
| FEAT-4 | Compliance / warning | 1. Configure guardrail (e.g. no PCI). 2. Customer or agent mentions sensitive data. | Warning or block as configured; no inappropriate suggestion. | |
| FEAT-5 | No irrelevant guidance | 1. Conduct routine call with no clear trigger. | No misleading or irrelevant hints; minimal noise. | |

### 3.2 Knowledge Assist (RAG)

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| FEAT-6 | KB article retrieval | 1. Add known articles to KB. 2. Customer asks matching question. | Relevant article(s) surfaced; suggestion usable by agent. | |
| FEAT-7 | Paraphrased question | 1. Ask same question with different wording. | Semantic match; correct article still retrieved. | |
| FEAT-8 | Out-of-scope question | 1. Ask question not in KB. | Clear “no match” or fallback; no hallucinated answer. | |
| FEAT-9 | Multiple articles | 1. Question matches 2+ articles. | Top relevant articles shown; ranking sensible. | |
| FEAT-10 | Latency | 1. Ask KB question. 2. Measure time to suggestion. | &lt; 500 ms from question detection to display. | |

### 3.3 Conversation Intelligence & Analytics

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| FEAT-11 | Sentiment | 1. Conduct calls with clearly positive/negative/neutral tone. 2. Check post-call. | Sentiment reflects conversation; available in UI/reports. | |
| FEAT-12 | Topic extraction | 1. Handle calls with distinct topics. 2. Check post-call. | Topics tagged correctly; usable for reporting. | |
| FEAT-13 | Moments / compliance | 1. Trigger configured moments (e.g. pricing, escalation). 2. Check analytics. | Moments detected and attributed; available in dashboards. | |
| FEAT-14 | Dashboards | 1. Run POC calls. 2. Use Cresta dashboards. | Metrics (e.g. handle time, resolution, containment) populated and accurate. | |

### 3.4 Guardrails & Policies

| ID | Scenario | Steps | Expected Result | Pass/Fail |
|----|----------|-------|-----------------|-----------|
| FEAT-15 | System-level guardrail | 1. Configure rule (e.g. “Do not output account numbers”). 2. Simulate trigger. | Output blocked or sanitized as configured. | |
| FEAT-16 | Supervisory guardrail | 1. Use scenario that should trigger supervisory check. 2. Verify. | Violation detected; action per policy (e.g. block, alert). | |
| FEAT-17 | Policy scope | 1. Apply policy to subset of agents/queues. 2. Test in/out of scope. | Policy only applies where configured. | |

---

## 4. Performance Validation Tests

**Goal**: Validate latency and throughput against documented targets. Ref: [05-realtime-dataflow-sequences](05-realtime-dataflow-sequences.md), [Cresta-review](Cresta-review.md).

### 4.1 Latency Tests

| ID | Test | Method | Target | Pass/Fail |
|----|------|--------|--------|-----------|
| PERF-1 | Speech → guidance (end-to-end) | Timestamp customer utterance; timestamp first relevant guidance in Agent App. Sample 20+ calls. | p95 &lt; 1.5 s | |
| PERF-2 | Agent assist suggestion | Measure from trigger (e.g. intent detected) to suggestion display. | p95 &lt; 200 ms | |
| PERF-3 | Knowledge Assist | Question detection → suggestion displayed. | &lt; 500 ms | |
| PERF-4 | First partial transcript | Customer speaks → first partial in Agent App. | 200–300 ms | |
| PERF-5 | Call summary | Call end → summary available. | Within seconds of disconnect | |

### 4.2 Throughput & Stability

| ID | Test | Method | Target | Pass/Fail |
|----|------|--------|--------|-----------|
| PERF-6 | Concurrent calls | Ramp to N concurrent calls (N = POC max). Maintain for 15–30 min. | No errors; latency within targets; no transcript loss. | |
| PERF-7 | Sustained load | Typical POC volume for 2–4 hours. | No degradation; no spike in errors. | |
| PERF-8 | Recovery | Restart Cresta/Connect component per runbook; place calls. | Stream and transcript recover; no data loss for in-flight calls per design. | |

---

## 5. Feature Test Matrix

Use this matrix to track which scenarios apply to which POC features. Update “In POC scope” based on final scope.

| Feature | In POC scope | Integration | Agent Assist | Knowledge Assist | Intelligence | Guardrails | Performance |
|---------|--------------|-------------|--------------|------------------|--------------|------------|-------------|
| Amazon Connect audio | Yes | INT-1–INT-10 | – | – | – | – | PERF-1, PERF-4 |
| Agent Assist | _TBD_ | – | FEAT-1–FEAT-5 | – | – | – | PERF-2 |
| Knowledge Assist | _TBD_ | – | FEAT-2 | FEAT-6–FEAT-10 | – | – | PERF-3 |
| Conversation intelligence | _TBD_ | – | – | – | FEAT-11–FEAT-14 | – | – |
| Guardrails | _TBD_ | – | FEAT-4 | – | – | FEAT-15–FEAT-17 | – |
| Summarization | _TBD_ | – | – | – | – | – | PERF-5 |

---

## 6. Baseline Metrics (Pre-POC)

**Goal**: Capture Connect metrics before Cresta for comparison.

| Metric | Baseline (pre-Cresta) | Source |
|--------|------------------------|--------|
| Average handle time | _TBD_ | Connect / CCD |
| First-contact resolution rate | _TBD_ | Connect / survey |
| Agent satisfaction (optional) | _TBD_ | Survey |
| Customer satisfaction (e.g. NPS) | _TBD_ | Survey / post-call |
| Transcript availability | N/A | – |
| Knowledge assist usage | N/A | – |

---

## 7. Test Execution Notes

- **Environment**: Use dedicated Connect instance + Cresta tenant for POC; avoid production.
- **Data**: Use synthetic or anonymized call scenarios; comply with data policies.
- **Agents**: Pilot agents trained on Agent App and POC scenarios.
- **Logging**: Retain Contact Flow logs, Cresta logs, and timestamps for latency tests.
- **Defects**: Log failures with scenario ID, steps, expected vs actual, and environment details.
- **Iteration**: Update scenarios as Cresta provides feature and integration docs.

---

## 8. References

- [05-realtime-dataflow-sequences.md](05-realtime-dataflow-sequences.md) – Real-time flows, latency budget  
- [02-amazon-connect-integration.md](02-amazon-connect-integration.md) – Integration architecture  
- [Cresta-review.md](Cresta-review.md) – Assessment, latency targets, features  
- [08-cresta-feature-documentation-request.md](08-cresta-feature-documentation-request.md) – Feature request  
- [09-cresta-amazon-connect-integration-request.md](09-cresta-amazon-connect-integration-request.md) – Integration request  
- [10-cresta-technical-sessions-agenda.md](10-cresta-technical-sessions-agenda.md) – Technical sessions, incl. POC planning
- [13-proof-of-concept-plan.md](13-proof-of-concept-plan.md) – Business-focused POC plan aligned with use cases
