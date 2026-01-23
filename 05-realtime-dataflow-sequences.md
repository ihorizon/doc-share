# Real-Time Data Flow - Sequence Diagrams

## Legend
- 🔒 **Security Risk** - Data protection, authentication, encryption concerns
- ⏱️ **Latency Risk** - Real-time performance critical path
- 📋 **Compliance Risk** - GDPR, PCI-DSS, HIPAA considerations
- ⚙️ **Operational Risk** - Availability, scaling, monitoring concerns
- 🟡 **Yellow/Orange** - Requires follow-up/verification

---

## End-to-End Real-Time Guidance Flow

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff','primaryTextColor':'#1f2937','primaryBorderColor':'#d1d5db','lineColor':'#1f2937','secondaryColor':'#fcfcfd','tertiaryColor':'#f9fafb'}}}%%
sequenceDiagram
    autonumber
    participant C as Customer
    participant AC as Amazon Connect
    participant KVS as Kinesis Video<br/>Streams
    participant GW as gowalter
    participant ASR as Deepgram ASR
    participant API as apiserver
    participant O as Orchestrator
    participant ML as ML Services
    participant CS as clientsubscription
    participant App as Agent App
    participant A as Agent

    rect rgb(255, 255, 255)
        Note over C,A: 📞 Call Initiation (t=0)
        C->>AC: Inbound Call
        AC->>AC: Contact Flow Execution
        AC->>KVS: Start Media Streaming
        AC->>A: Route to Agent
    end

    Note over C,A: ⚡ Real-Time Processing Loop (t=1s onwards)
    
    rect rgb(254, 243, 199)
        Note right of C: 🎤 Audio Capture<br/>⏱️ Latency Critical
        C->>AC: Speech
        AC->>KVS: Audio Stream (PCM linear16)
        KVS->>GW: WebSocket Stream
    end

    rect rgb(209, 250, 229)
        Note right of GW: 📝 Transcription<br/>⏱️ Latency Critical
        GW->>ASR: Audio Chunks<br/>(20-100ms)
        ASR-->>GW: Partial Transcript
        GW->>API: Upsert Transcript
        API->>API: Persist to PostgreSQL
    end

    rect rgb(254, 243, 199)
        Note right of API: 🤖 ML Inference<br/>⏱️ Latency Critical
        API->>O: Transcript Event
        O->>O: Build Inference Graph
        O->>ML: Execute Models
        ML-->>O: Annotations<br/>(Intent, Sentiment, Actions)
    end

    rect rgb(209, 250, 229)
        Note right of O: 📱 Agent Notification
        O->>CS: Publish Actions
        CS->>App: WebSocket Push
        App->>A: Display Guidance
    end

    Note over C,A: ✅ Total Target Latency: <1.5s
```

---

## Knowledge Assist Flow (RAG)

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff','primaryTextColor':'#1f2937','primaryBorderColor':'#d1d5db','lineColor':'#1f2937','secondaryColor':'#fcfcfd','tertiaryColor':'#f9fafb'}}}%%
sequenceDiagram
    autonumber
    participant Agent as Agent
    participant App as Agent App
    participant Orch as Orchestrator
    participant RAG as RAG Pipeline
    participant KB as Knowledge Base
    participant Ocean as Ocean-1 (Fireworks)
    participant CS as clientsubscription

    Note over Agent,CS: Customer asks a question

    Agent->>App: Sees customer question in transcript
    
    App->>Orch: Transcript with question detected
    
    Orch->>RAG: Trigger Knowledge Assist
    
    rect rgb(254, 243, 199)
        Note right of RAG: Query Processing
        RAG->>RAG: Extract query from conversation
        RAG->>KB: Semantic search
        KB-->>RAG: Retrieved articles (top K)
    end

    rect rgb(254, 243, 199)
        Note right of RAG: Response Generation ⏱️
        RAG->>Ocean: Context + Query + Articles
        Ocean-->>RAG: Generated response
    end

    RAG->>RAG: Post-process & format
    RAG-->>Orch: Knowledge Assist result
    
    Orch->>CS: Publish action
    CS->>App: Display suggestion
    
    App->>Agent: Show answer card
    Agent->>Agent: Use response with customer

    Note over Agent,CS: Target: <500ms for suggestion
```

---

## Call Summarization Flow (End of Call)

```mermaid
sequenceDiagram
    autonumber
    participant Agent as Agent
    participant Connect as Amazon Connect
    participant GW as gowalter
    participant API as apiserver
    participant Temporal as Temporal Workflow
    participant Ocean as Ocean-1
    participant S3 as S3 Storage 🔒
    participant ES as Elasticsearch
    participant App as Agent App

    Note over Agent,App: Call Termination

    Agent->>Connect: End Call
    Connect->>GW: Stop Media Streaming
    Connect->>API: CloseConversation Event

    par Audio Processing
        GW->>GW: Encode & compress audio
        GW->>GW: Apply PII redaction (beeps) 📋
        GW->>S3: Upload redacted audio
    and Transcript Processing
        API->>ES: Index conversation
        API->>API: Store analytics in ClickHouse
    end

    API->>Ocean: Generate summary
    activate Ocean
    Note right of Ocean: Summarization model<br/>processes full transcript
    Ocean-->>API: Call summary
    deactivate Ocean

    API->>App: Deliver summary to agent
    App->>Agent: Display summary

    rect rgb(255, 243, 205)
        Note right of API: Redaction Verification ⚙️
        API->>Temporal: Start verification workflow
        Temporal->>Temporal: Re-scan for PII
        alt PII Found
            Temporal->>Temporal: Retry redaction
        else Clean
            Temporal->>API: Mark complete
        end
    end

    Note over Agent,App: Summary available within seconds of call end
```

---

## Real-Time Translation Flow

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff','primaryTextColor':'#1f2937','primaryBorderColor':'#d1d5db','lineColor':'#1f2937','secondaryColor':'#fcfcfd','tertiaryColor':'#f9fafb'}}}%%
sequenceDiagram
    autonumber
    participant Customer as Customer (Spanish)
    participant Connect as Amazon Connect
    participant GW as gowalter
    participant STT as Speech-to-Text
    participant Lang as Language Detection
    participant MT as Machine Translation
    participant TTS as Text-to-Speech
    participant App as Agent App
    participant Agent as Agent (English)

    Note over Customer,Agent: Multilingual Call Handling

    Customer->>Connect: Speaks Spanish
    Connect->>GW: Audio stream

    rect rgb(254, 243, 199)
        Note right of GW: Real-Time Translation Pipeline ⏱️
        GW->>Lang: Detect language
        Lang-->>GW: Spanish detected
        GW->>STT: Spanish audio
        STT-->>GW: Spanish transcript
        GW->>MT: Translate Spanish → English
        MT-->>GW: English text
    end

    GW->>App: Display English translation
    App->>Agent: See translated conversation

    Note over Customer,Agent: Agent Response (Reverse)

    Agent->>App: Types English response
    App->>MT: Translate English → Spanish
    MT-->>App: Spanish text
    App->>TTS: Generate Spanish audio 🟡
    TTS-->>App: Spanish audio
    App->>Connect: Play to customer 🟡

    Note over Customer,Agent: Sub-second translation latency target
```

---

## Agent App Event Subscription Flow

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff','primaryTextColor':'#1f2937','primaryBorderColor':'#d1d5db','lineColor':'#1f2937','secondaryColor':'#fcfcfd','tertiaryColor':'#f9fafb'}}}%%
sequenceDiagram
    autonumber
    participant App as Agent App
    participant CS as clientsubscription
    participant Redis as Redis Streams
    participant API as apiserver
    participant Orch as Orchestrator

    Note over App,Orch: Agent Signs In

    App->>CS: Subscribe (AgentId, ConversationId)
    CS->>Redis: Register subscription

    Note over App,Orch: Conversation Active

    rect rgb(209, 250, 229)
        Note right of API: Transcript Updates (~1/second)
        API->>Redis: Publish transcript event
        Redis->>CS: Stream event
        CS->>App: WebSocket push (transcript)
    end

    rect rgb(209, 250, 229)
        Note right of Orch: ML Actions
        Orch->>Redis: Publish action event
        Redis->>CS: Stream event
        CS->>App: WebSocket push (hint/suggestion)
    end

    rect rgb(209, 250, 229)
        Note right of API: Call Updates
        API->>Redis: Publish call status
        Redis->>CS: Stream event
        CS->>App: WebSocket push (call ended)
    end

    Note over App,Orch: Agent Signs Out

    App->>CS: Unsubscribe
    CS->>Redis: Remove subscription
```

---

## Error Recovery Flow

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff','primaryTextColor':'#1f2937','primaryBorderColor':'#d1d5db','lineColor':'#1f2937','secondaryColor':'#fcfcfd','tertiaryColor':'#f9fafb'}}}%%
sequenceDiagram
    autonumber
    participant Connect as Amazon Connect
    participant GW as gowalter
    participant ASR as Deepgram ASR
    participant Buffer as Audio Buffer
    participant API as apiserver
    participant App as Agent App

    Note over Connect,App: Normal Operation
    Connect->>GW: Audio stream
    GW->>Buffer: Store chunks
    GW->>ASR: Forward audio
    ASR-->>GW: Transcripts
    GW->>API: Upsert

    Note over GW,ASR: ⚠️ ASR Connection Lost
    ASR--xGW: WebSocket disconnected

    rect rgb(254, 226, 226)
        Note right of GW: Recovery Sequence ⚙️
        GW->>GW: Detect disconnect
        GW->>Buffer: Get buffered audio
        GW->>ASR: Reconnect WebSocket
        GW->>ASR: Replay buffered chunks
        ASR-->>GW: Catch-up transcripts
    end

    GW->>API: Upsert recovered transcripts
    API->>App: Resume real-time updates

    Note over Connect,App: Service Restored

    alt Recovery Fails (Max Retries)
        GW->>GW: Log error ⚙️
        GW->>API: Mark gap in transcript
        API->>App: Notify partial transcript
    end
```

---

## Latency Budget Breakdown

### Target: Speech → Agent Guidance < 1.5 seconds

```mermaid
flowchart LR
    subgraph Outer[" "]
        subgraph Audio["Audio Processing (0-150ms)"]
            direction LR
            A1[Audio Chunk<br/>Capture<br/>0-100ms]
            A2[Network to<br/>Cresta<br/>100-150ms]
            A1 ==> A2
        end
        
        subgraph ASR["ASR Processing (150-500ms)"]
            direction LR
            A3[ASR<br/>Processing<br/>150-450ms]
            A4[Transcript<br/>Delivery<br/>450-500ms]
            A3 ==> A4
        end
        
        subgraph ML["ML Inference (500-1100ms)"]
            direction LR
            A5[Orchestrator<br/>Routing<br/>500-550ms]
            A6[Model<br/>Inference<br/>550-1050ms]
            A7[Result<br/>Aggregation<br/>1050-1100ms]
            A5 ==> A6 ==> A7
        end
        
        subgraph Delivery["Delivery (1100-1250ms)"]
            direction LR
            A8[Event<br/>Publish<br/>1100-1150ms]
            A9[WebSocket<br/>to App<br/>1150-1200ms]
            A10[UI<br/>Render<br/>1200-1250ms]
            A8 ==> A9 ==> A10
        end
        
        A2 ==>|"Audio Stream"| A3
        A4 ==>|"Transcript"| A5
        A7 ==>|"Annotations"| A8
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style Audio fill:none,stroke:#9ca3af,stroke-width:1.5px
    style ASR fill:none,stroke:#9ca3af,stroke-width:1.5px
    style ML fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Delivery fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style A1 fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style A2 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A3 fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style A4 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A5 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A6 fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style A7 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A8 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A9 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style A10 fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
```

| Stage | Target (ms) | Risk Level | Notes |
|-------|-------------|------------|-------|
| Audio Capture | 100 | Low | Chunk size dependent |
| Network (KVS → Cresta) | 50 | Medium | Regional proximity matters |
| ASR Processing | 300 | **High** ⏱️ | Critical path |
| Transcript Delivery | 50 | Low | Internal network |
| Orchestrator | 50 | Low | Cached policies |
| ML Inference | 500 | **High** ⏱️ | Model dependent |
| Event Publishing | 50 | Low | Redis streams |
| WebSocket Delivery | 50 | Medium | Client network |
| UI Render | 50 | Low | Client performance |
| **Total** | **1250** | - | 250ms buffer |

---

## Items Requiring Follow-up 🟡

1. **TTS for Translation** - How is translated audio played back to customer?
2. **WebSocket Protocol** - Exact protocol between components (ws/wss, message format)
3. **Retry Policies** - Specific retry counts and backoff strategies
4. **Circuit Breakers** - Are there circuit breakers for external services?
5. **Metric Collection** - How are latency metrics captured and monitored?
6. **SLA Definitions** - Published SLAs for real-time guidance latency

---

## Summary

This document provides sequence diagrams for key real-time data flows in the Cresta platform, focusing on latency-critical paths and end-to-end guidance delivery.

**Key Flows Documented**:
1. **End-to-End Real-Time Guidance**: Customer speech → KVS → gowalter → ASR → apiserver → Orchestrator → ML Services → clientsubscription → Agent App (target <1.5s)
2. **Knowledge Assist (RAG)**: Question detection → RAG pipeline → Knowledge Base search → Ocean-1 generation → Agent App (target <500ms)
3. **Call Summarization**: Call end → Audio encoding/redaction → Ocean-1 summarization → Agent App (within seconds of disconnect)
4. **Real-Time Translation**: Multi-language call handling with STT, translation, and TTS (TTS playback method requires verification)
5. **Agent App Event Subscription**: WebSocket-based real-time updates (transcripts, ML actions, call status)
6. **Error Recovery**: ASR disconnection → gowalter buffer replay → transcript continuity

**Latency Budget Breakdown** (Target: <1.5s total):
- Audio Processing: 0-150ms (capture + network to Cresta)
- ASR Processing: 150-500ms (ASR processing + transcript delivery)
- ML Inference: 500-1100ms (orchestrator routing + model inference + result aggregation)
- Delivery: 1100-1250ms (event publish + WebSocket to App + UI render)

**Verification Status**: Flow sequences are logically structured and consistent with real-time requirements. Latency targets are documented but should be verified against Cresta published SLAs. Translation flow includes TTS and audio playback components that require Cresta confirmation.
