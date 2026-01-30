# Cresta Real-Time Data Flow Sequences

## Document Purpose

This document traces end-to-end data flows through the Cresta platform, showing how audio becomes actionable agent guidance. All sequences are based on verified architecture components from previous documents.

---

## Complete Call Flow: Customer Speech to Agent Guidance

### Phase 1: Audio Capture & Streaming (0-100ms)

```
Customer speaks
   ↓ (Telephony network)
Amazon Connect Instance
   ↓ (Contact Flow: Start Media Streaming block)
Kinesis Video Streams
   ├─ Creates dedicated stream (one per call)
   ├─ 8kHz PCM audio
   ├─ AUDIO_FROM_CUSTOMER track
   └─ AUDIO_TO_CUSTOMER track
   ↓ (WebSocket/HTTP2, StreamARN passed via Contact Attributes)
Customer-specific subdomain (customer.region.cresta.ai)
   ↓ (AWS ELB → NGINX Ingress + Wallarm WAF)
gowalter WebSocket Handler
```

**Cumulative Latency**: ~50-100ms
**Verified**: AWS Connect docs, Cresta blog

---

### Phase 2: ASR Transcription (100-400ms)

```
gowalter
   ├─ Audio Buffer (accumulates 20-100ms chunks)
   ├─ WebSocket to Deepgram ASR
   └─ Recovery Manager (replays on disconnect)
   ↓ (20-100ms chunks streamed)
Deepgram Streaming ASR (Nova-3)
   ├─ Partial transcripts (every 0.5-1.5s)
   ├─ Final transcripts (every 3-7s)
   └─ Phonetic corrections as context arrives
   ↓ (Sub-300ms latency)
gowalter Utterance Builder
   ├─ Organizes ASR chunks into conversation units
   ├─ Speaker diarization (customer vs agent)
   └─ Prepares for persistence
   ↓
apiserver Transcript Upsert
   ├─ PostgreSQL insert (~10-50ms)
   ├─ Event publication to Redis (~5-20ms)
   └─ Approximately 1 update/second to agent
```

**Cumulative Latency**: ~150-500ms
**Verified**: Deepgram benchmarks, Cresta blog on voice stack

---

### Phase 3: ML Processing (Variable, 50-600ms)

```
apiserver Event Publisher
   ↓ (Redis Streams, asynchronous)
Orchestrator
   ├─ Loads cached search policies
   │  └─ Customer/team/agent-specific configuration
   ├─ Builds inference graph
   │  └─ Blueprint of which models to call
   └─ Routes to ML services
   ↓ (Routing decision ~5-20ms)
ML Router
   ├─ Selects appropriate model shards
   ├─ Batch mode (if burst traffic): 100-300ms window
   └─ Real-time mode (if live serving): immediate forwarding
   ↓
Model Shards (Kubernetes Pods)
   ├─ Envelope (intra-pod router) <5ms
   ├─ Local models (classifiers: ~50-200ms)
   │  ├─ Intent detection
   │  ├─ Sentiment analysis
   │  └─ Entity extraction
   └─ Ocean-1 + LoRA (via Fireworks AI)
      ├─ Knowledge Assist (RAG)
      ├─ Response suggestions
      └─ Policy compliance checks
   ↓
Annotations Generated
   ├─ Moments (detections, alerts)
   └─ Actions (suggestions, hints)
```

**ML Processing Latency**:
- Lightweight classifiers: 50-200ms
- Ocean-1 RAG: Variable (🟡 needs benchmarks)
- Batching adds: 100-300ms if triggered

**Verified**: Cresta blog on ML services, Fireworks partnership

---

### Phase 4: Real-Time Delivery to Agent (50-150ms)

```
Orchestrator
   ↓ (Annotations aggregated)
clientsubscription Service
   ├─ Redis Streams pub/sub (~5-20ms)
   ├─ WebSocket to Agent App
   └─ Approximately 1 update/second delivery
   ↓
Cresta Agent App (Desktop)
   ├─ Renders guidance in UI (~50-100ms)
   ├─ Displays real-time transcript
   ├─ Shows actions (knowledge articles, suggestions)
   └─ Highlights moments (compliance alerts, sentiment)
   ↓
Agent sees guidance on screen
```

**Cumulative Latency**: ~55-170ms
**Verified**: Cresta blog on voice stack and clientsubscription

---

## End-to-End Latency Budget

| Stage | Component | Latency (Verified/Estimated) |
|-------|-----------|------------------------------|
| 1 | Audio capture (Connect) | ~20-50ms |
| 2 | KVS ingestion | ~20-30ms |
| 3 | Network transit | ~30-100ms |
| 4 | gowalter buffering | 20-100ms |
| 5 | **Deepgram ASR** | **<300ms** ✅ |
| 6 | Utterance building | ~100-200ms |
| 7 | PostgreSQL upsert | ~10-50ms |
| 8 | Redis event pub | ~5-20ms |
| 9 | Orchestrator processing | ~5-20ms |
| 10 | ML routing | ~5-20ms |
| 11 | **ML inference (light)** | **50-200ms** |
| 11a | **ML inference (Ocean-1)** | **🟡 Variable** |
| 11b | **Batching (if triggered)** | **+100-300ms** |
| 12 | Annotation aggregation | ~10-30ms |
| 13 | Redis to clientsub | ~5-20ms |
| 14 | Agent App render | ~50-100ms |

**Best Case Total** (lightweight models, no batching): ~530ms - 820ms ✅
**Typical Case** (Ocean-1 RAG): ~750ms - 1,500ms (assuming ~200-500ms Ocean-1 latency)
**Worst Case** (batching + complex ML): ~1,250ms - 2,120ms

**Target**: <1,500ms (industry standard for voice assistance)
**Confidence**: High for ASR, Medium for ML (needs vendor benchmarks)

---

## Parallel End-of-Call Processing

### Triggered When: Call Disconnects

```
gowalter Detects Call End
   ├─ Final Audio Processing
   │  ├─ Audio Redaction Engine (PII beeps)
   │  ├─ Audio Encoder (compression)
   │  └─ S3 Upload (redacted audio)
   │
   └─ Async processing, no impact on call
       
apiserver Receives CloseConversation Event
   ├─ Elasticsearch Indexing
   │  └─ Fast search capability for historical lookup
   ├─ ClickHouse Storage  
   │  └─ Analytics data warehouse
   ├─ Temporal Workflow Launch
   │  └─ PII Redaction Verification
   │     ├─ Double-check audio redaction complete
   │     ├─ Verify text redaction complete
   │     └─ Ensure no un-redacted PII remains
   └─ End-of-Call Features
      ├─ Conversation Summarization
      ├─ Quality Scoring
      └─ Analytics Computation
```

**Processing Time**: Async (minutes), no impact on real-time performance
**Verified**: Cresta blog on ML services and voice stack

---

## Critical Path Analysis

### Latency-Critical Components (Must Optimize)

1. **Deepgram ASR**: <300ms ✅ (Verified best-in-class)
2. **Ocean-1 Inference**: Variable ⚠️ (Needs benchmarks from Fireworks)
3. **gowalter Buffering**: 20-100ms ✅ (Optimized chunk size)
4. **Batcher Window**: 100-300ms ⚠️ (Trade-off: latency vs throughput)

### Non-Critical Path (Async Acceptable)

1. **S3 Audio Storage**: End-of-call, async
2. **Elasticsearch Indexing**: End-of-call, async
3. **ClickHouse Analytics**: End-of-call, async
4. **Temporal Verification**: End-of-call, async

---

## Data Flow Variations

### Variation 1: Agent App Audio Capture (CCaaS Without Streaming)

```
Agent Desktop
   ↓ (System audio capture: mic + speakers)
Cresta Agent App
   ↓ (Direct upload to Cresta)
gowalter WebSocket Handler
   ↓ (Continue standard flow from Phase 2)
```

**Use Case**: CCaaS platforms that don't provide native audio streaming APIs
**Note**: Amazon Connect supports native streaming, so this is fallback only

---

### Variation 2: Batch ML Processing (High Traffic)

```
Multiple Transcript Updates (burst traffic)
   ↓
ML Router Detects Pattern
   ↓ (Activates Batcher)
Batcher (100-300ms collection window)
   ├─ Aggregates requests by model type
   └─ Sends batched requests to shards
   ↓
GPU Batch Inference
   ├─ Process multiple requests simultaneously
   ├─ Higher throughput (~2-5x)
   └─ Better GPU utilization
   ↓ (Results returned for batch)
Individual Annotations Distributed
   ↓ (To respective conversations)
Standard delivery to Agent Apps
```

**Trade-off**:
- **Latency**: +100-300ms from batching window
- **Throughput**: 2-5x increase in requests/second
- **Cost**: Improved GPU efficiency

**Trigger**: High traffic periods (call center peak hours)

---

## Monitoring Recommended Metrics

### Real-Time Performance (SLA Compliance)

| Metric | Target | Priority | Where to Measure |
|--------|--------|----------|------------------|
| ASR latency (p50) | <300ms | Critical | gowalter → Deepgram |
| ASR latency (p99) | <500ms | Critical | gowalter → Deepgram |
| ML inference (p50) | <200ms | High | Router → Shard → Response |
| ML inference (p99) | <500ms | High | Router → Shard → Response |
| End-to-end (p50) | <1,000ms | Critical | Speech → Agent screen |
| End-to-end (p99) | <1,500ms | Critical | Speech → Agent screen |
| Agent update frequency | ~1/sec | Medium | clientsubscription delivery |

### Operational Health

| Metric | Target | Priority | Where to Measure |
|--------|--------|----------|------------------|
| gowalter recovery events | <1% | Medium | WebSocket disconnect rate |
| ASR WebSocket uptime | >99.9% | High | Deepgram connection status |
| Fireworks API availability | >99.9% | Critical | Ocean-1 inference success |
| PostgreSQL write latency | <50ms | Medium | apiserver upsert timing |
| Redis pub/sub latency | <20ms | Medium | Event delivery timing |
| Shard autoscaling lag | <60s | Medium | K8s pod startup time |

---

## Bottleneck Identification

### Known Bottlenecks (From Architecture)

1. **ASR Processing**: <300ms latency dominates budget but already optimized ✅
2. **Ocean-1 Inference**: Variable latency, external dependency ⚠️
3. **Batching Window**: 100-300ms added when triggered ⚠️
4. **Network Jitter**: Variable based on regional proximity

### Optimization Opportunities

1. **Regional Colocation**: Deploy Cresta in same AWS region as Connect
2. **Warm Shard Pool**: Pre-scale shards before peak traffic
3. **Adaptive Batching**: Dynamically adjust window based on latency SLA
4. **Edge Caching**: Cache frequent Ocean-1 responses (e.g., common FAQs)

---

## Failure Mode Flows

### Scenario 1: Deepgram ASR Unavailable

```
gowalter ASR Client
   ↓ (WebSocket connection fails)
Recovery Manager Activates
   ├─ Retry with exponential backoff
   └─ If recovery fails after N attempts:
      └─ 🟡 Unknown: Failover to backup ASR or graceful degradation?
         └─ **Requires vendor clarification**
```

**Impact**: No transcription = no ML guidance
**Severity**: Critical

---

### Scenario 2: Fireworks AI Unavailable

```
Model Shard (Ocean-1 + LoRA)
   ↓ (Inference request to Fireworks)
Fireworks API Timeout/Error
   └─ 🟡 Unknown: Local fallback or error to orchestrator?
      ├─ Option A: Return error, skip Ocean-1 annotations
      ├─ Option B: Fallback to simpler model (e.g., GPT-3.5)
      └─ **Requires vendor clarification**
```

**Impact**: No Ocean-1 guidance (RAG, summarization)
**Severity**: High (but other classifiers may continue)

---

### Scenario 3: PostgreSQL Write Failure

```
apiserver Transcript Upsert
   ↓ (Database write fails)
Error Handling
   ├─ Retry with backoff
   ├─ Log to error tracking (Sentry, CloudWatch)
   └─ Continue event publishing to Redis
      └─ ML processing continues (uses in-memory transcript)
      └─ Transcript persistence delayed but eventually consistent
```

**Impact**: Temporary transcript loss, but real-time ML continues
**Severity**: Medium

---

## Data Sovereignty & Regional Routing

### Verified Regional Architecture
**Source**: Cresta documentation on customer subdomains

```
EU Customer Call
   ↓
Amazon Connect (EU region)
   ↓ (Regional KVS)
customer.eu.cresta.ai
   ├─ Cresta infrastructure deployed in EU AWS region
   ├─ Data stays within EU for GDPR compliance
   └─ Regional Deepgram ASR endpoint (🟡 needs confirmation)

Fireworks AI Inference
   └─ 🟡 Critical Question: Are Fireworks endpoints regional?
      ├─ If no: Data transits to US (potential GDPR issue)
      └─ If yes: Confirm regional deployment matches Cresta regions
```

**Follow-Up Required**:
- Fireworks AI regional deployment map
- Data Processing Agreement (DPA) coverage for cross-border ML inference
- Customer ability to specify inference region

---

**Document Status**: Comprehensive data flow sequences based on verified architecture. Critical latency dependencies and failure modes identified. Specific benchmarks and failure handling require vendor clarification.
