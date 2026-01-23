# Cresta AI – Amazon Connect Integration Guide Request

**Purpose**: Request integration documentation to support technical assessment and POC setup. We need step-by-step integration instructions and implementation details to configure our test environment and validate the Amazon Connect integration.

**Context**: Technical assessment of Cresta AI and its Amazon Connect integration. High-level flows are documented in [02-amazon-connect-integration.md](02-amazon-connect-integration.md); implementation details are required to set up and test the integration.

---

## 1. Integration Method & Architecture

| # | Request | Purpose |
|---|---------|---------|
| 1.1 | **Exact integration method** – Does Cresta use a Lambda consumer, direct GetMedia API, webhook, or other mechanism to receive Kinesis Video Streams (KVS) audio? | Design POC architecture |
| 1.2 | **Audio flow** – End-to-end path from Connect → KVS → Cresta (including any Lambda, DynamoDB, or other AWS components) | Validate our architecture assumptions |
| 1.3 | **Option A vs Option B** – When is direct KVS stream used vs Agent App desktop capture? Confirm which applies to Amazon Connect. | Plan POC configuration |
| 1.4 | **Network connectivity** – Does Cresta pull from KVS (outbound from Cresta) or does Connect/KVS push to Cresta? Regional requirements? | Plan networking and region placement |

---

## 2. Contact Flow Configuration

| # | Request | Purpose |
|---|---------|---------|
| 2.1 | **Step-by-step Contact Flow** – Recommended blocks and order (e.g., Set Voice Language → Start Media Streaming → Set Contact Attributes → Invoke Lambda → Play Prompt → Transfer to Queue) | Configure our Contact Flow for POC |
| 2.2 | **Required Contact Flow blocks** – Minimum required vs optional (e.g., call recording consent, language) | Ensure compliant, minimal config |
| 2.3 | **Contact attributes** – Complete list of attributes Cresta expects: system-provided (e.g. `streamARN`, `startFragmentNum`, `ContactId`, `InstanceId`) and any custom (e.g. `CrestaCustomerId`, `CrestaProfileId`) | Configure attributes correctly |
| 2.4 | **Attribute mapping** – Source of each attribute (Connect system, Lambda, Static), format, and validation rules | Avoid integration errors |
| 2.5 | **Lambda trigger** – Lambda name/signature, when it is invoked, what it receives, what it passes to Cresta | Implement or validate Lambda |
| 2.6 | **Stop Media Streaming** – When and how streaming is stopped (e.g., on contact end), and impact on Cresta | Ensure clean call lifecycle |

---

## 3. Authentication & Security

| # | Request | Purpose |
|---|---------|---------|
| 3.1 | **Authentication method** – API keys, OAuth, IAM roles, or other mechanism for Connect/Lambda to authenticate with Cresta | Configure secure POC |
| 3.2 | **Cresta endpoint** – URL pattern (e.g. `customer.region.cresta.ai`), WebSocket vs REST, and how it is selected per contact | Configure DNS and routing |
| 3.3 | **KVS access** – If Cresta consumes KVS directly: IAM roles, cross-account access, least-privilege permissions | Configure AWS IAM |
| 3.4 | **Secrets management** – How API keys or credentials are stored and rotated (e.g. AWS Secrets Manager, Lambda env vars) | Meet security standards |
| 3.5 | **TLS** – Confirmation of TLS 1.2+ for all Cresta endpoints | Validate compliance |

---

## 4. AWS Prerequisites & Setup

| # | Request | Purpose |
|---|---------|---------|
| 4.1 | **AWS prerequisites** – Amazon Connect instance, KVS, IAM roles, Lambda (if used), DynamoDB (if used), etc. | Checklist for POC |
| 4.2 | **Cresta-side provisioning** – What Cresta provides (e.g. tenant, credentials, endpoints) and what we must configure in Cresta console | End-to-end setup |
| 4.3 | **Cross-account** – If Connect/KVS are in a different AWS account than Cresta, required trust and permissions | Plan multi-account setup |
| 4.4 | **Region alignment** – Supported regions for Cresta and KVS; requirements for same-region vs cross-region | Choose region for POC |

---

## 5. Error Handling & Resilience

| # | Request | Purpose |
|---|---------|---------|
| 5.1 | **Failover behavior** – If Cresta is unreachable, does the call continue? Is audio still recorded? What does the agent experience? | Define SLAs and runbooks |
| 5.2 | **Retry logic** – Retries for Lambda, KVS, or Cresta connections; backoff and limits | Understand resilience |
| 5.3 | **Partial failure** – Behavior when stream starts but drops mid-call (e.g. gowalter recovery, transcript gaps) | Set expectations for POC |
| 5.4 | **Error reporting** – How errors are surfaced (logs, Connect, Cresta UI) and recommended monitoring | Operational readiness |

---

## 6. Testing & Validation

| # | Request | Purpose |
|---|---------|---------|
| 6.1 | **Integration testing steps** – Recommended sequence to validate Connect → KVS → Cresta (e.g. test call, verify transcript in Cresta) | POC validation plan |
| 6.2 | **Validation checklist** – Config checks before going live (Contact Flow, attributes, Lambda, Cresta, IAM) | Pre–go-live verification |
| 6.3 | **Test Contact Flow** – Sample or reference Contact Flow (export or diagram) for Amazon Connect + Cresta | Accelerate POC setup |
| 6.4 | **Troubleshooting guide** – Common integration failures, diagnostics, and resolutions | Support POC debugging |

---

## 7. Agent Application & CCP

| # | Request | Purpose |
|---|---------|---------|
| 7.1 | **Agent app deployment** – Browser extension, desktop app, embedded in CCP, or other? | Plan agent rollout for POC |
| 7.2 | **CCP integration** – How Cresta Agent App connects to Amazon Connect CCP (if applicable) | Configure agent desktop |
| 7.3 | **Audio source** – When using Connect: is audio always from KVS, or can it fall back to Agent App capture? Under what conditions? | Clarify architecture |

---

## 8. Operational Considerations

| # | Request | Purpose |
|---|---------|---------|
| 8.1 | **KVS quotas** – Typical concurrency limits, scaling, and how to request increases | Plan for call volume |
| 8.2 | **Lambda** – Concurrency, cold start, timeout; use of provisioned concurrency if recommended | Capacity planning |
| 8.3 | **Multi-region** – If we use multiple Connect instances or regions, how is traffic routed to Cresta? | Design for scale |
| 8.4 | **Maintenance** – Impact of Cresta or AWS maintenance on live calls; notifications | Plan change management |

---

## Deliverables Requested

- **Amazon Connect integration guide** – Step-by-step setup (Contact Flow, Lambda, IAM, Cresta).
- **Contact Flow reference** – Recommended blocks, order, and attribute mapping.
- **Contact attributes specification** – Complete list, sources, and formats.
- **Authentication and security** – How to authenticate and secure the integration.
- **Testing and validation** – Procedures and checklist to validate the integration.
- **Troubleshooting guide** – Common issues and resolutions for the Connect–Cresta integration.

**Format**: PDF, Confluence, or equivalent documentation links are acceptable.

---

## Reference

- [02-amazon-connect-integration.md](02-amazon-connect-integration.md) – Current integration architecture and diagrams  
- [00-summary-and-followups.md](00-summary-and-followups.md) – Follow-up items (e.g. KVS consumer, auth, failover)  
- [05-realtime-dataflow-sequences.md](05-realtime-dataflow-sequences.md) – End-to-end real-time flows  
- Gap Analysis: Cresta AI Technical Assessment for POC Planning
