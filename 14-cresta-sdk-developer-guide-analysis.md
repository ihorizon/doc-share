# Cresta Client SDK Developer Guide – Analysis & Validation

**Source**: `Cresta Client SDK Developer Guide.pdf` (extracted to text)  
**Date Analyzed**: January 2025  
**Status**: Age of guide unknown – confirm currency with Cresta

---

## Executive Summary

The Cresta Client SDK Developer Guide provides comprehensive documentation for integrating Cresta AI services into custom contact center applications. Key findings validate and expand upon the architecture documentation, particularly around Agent App integration, authentication, and real-time event delivery.

**Key Validations:**
- ✅ **ClientSubscription WebSocket service** confirmed (matches architecture docs)
- ✅ **API endpoint pattern**: `https://api-CUSTOMER_ID.cresta.com`
- ✅ **Voice session detection** via SDK (conversation ID, status, timing)
- ✅ **Profile ID** required for both chat and voice integrations
- ✅ **Authentication**: BYOID (OIDC), SAML, username/password, Twilio Flex token exchange

**New Information:**
- SDK packages and versions
- WebSocket fallback mechanisms (polling, SSE)
- Network timeout/retry configuration
- Error handling patterns
- Headless mode APIs for custom UI components

---

## SDK Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `@cresta/client` | 0.21.0 | Core Vanilla JS SDK (framework-agnostic) |
| `@cresta/react-client` | 0.10.0 | React component wrappers |
| `@cresta/client-sdk-sample` | 0.18.0 | Vanilla JS quickstart sample app |
| `@cresta/client-sdk-react-sample` | 0.22.0 | React quickstart sample app |
| `@cresta/client-sdk-emulator` | 0.14.0 | Mock Cresta AI server for testing |
| `@cresta/client-sdk-auth-sample` | 0.3.0 | Sample OIDC auth server |
| `@cresta/client-sdk-utils` | 0.1.0 | Utilities (HMAC auth, etc.) |

**NPM Registry**: `https://npm.cresta.com` (scoped packages under `@cresta/`)

---

## Authentication Methods

### 1. Popup Flow (SAML/Username-Password)
- Uses `signInWithPopup()`
- Supports SAML IdP or Cresta username/password
- Requires authorized origins (security measure)
- **Note**: Not needed if using BYOID

### 2. BYOID (Bring Your Own IDentity) ✅ **Recommended**
- **OAuth 2.0 Token Exchange** (RFC 8693)
- Exchanges platform OIDC ID token for Cresta access token
- **Token Requirements**:
  - **Signing**: RS256 (asymmetric)
  - **Required fields**: `iss`, `sub`, `aud`, `exp`, `iat`
  - **Optional**: `nbf`, `email_verified`, `nonce`, `picture`, `given_name`, `family_name`
  - **Cresta-specific**: `sub`, `email`, `name`
- **Public key retrieval**: Via OIDC metadata endpoint (`/.well-known/openid-configuration` → `jwks_uri`)
- **Benefits**: No popup, session sync, better UX

### 3. Twilio Flex Integration
- Direct token exchange: Twilio Flex tokens → Cresta access tokens
- No BYOID implementation needed
- Requires: Twilio AccountSID, AuthToken, Cresta client ID

### 4. Username/Password (Inline) ⚠️ **Not Recommended**
- Requires reCAPTCHA (score-based + checkbox fallback)
- Only use if proxy/firewall restrictions prevent popup

---

## API Endpoints & Configuration

### Service Endpoints

| Endpoint Type | Pattern | Example |
|---------------|---------|---------|
| **Service Endpoint** | `https://api-CUSTOMER_ID.cresta.com` | Primary API endpoint |
| **Customer Origin** | `https://CUSTOMER_ID.cresta.com` | Authentication URL |
| **Auth Endpoint** | `https://auth.cresta.com` (default) | Token exchange |

**Configuration Structure:**
```json
{
  "customerId": "CUSTOMER_ID",
  "customerOrigin": "https://CUSTOMER_ID.cresta.com",
  "serviceEndpoint": "https://api-CUSTOMER_ID.cresta.com",
  "clientId": "ID_TOKEN_AUDIENCE_FIELD"  // For BYOID
}
```

**Proxy Support**: Can substitute proxy endpoint in `serviceEndpoint` and `authEndpoint` fields.

---

## ClientSubscription WebSocket Service ✅ **Confirmed**

**From SDK Guide:**
> "The underlying REST APIs and ClientSubscription WebSocket service can be quite complex to set up and wire correctly."

**Key Details:**
- **Purpose**: Subscribes to events for Cresta features (reply suggestions, behavioral hints)
- **Recovery**: WebSocket recovers automatically on disconnect
- **Fallback Options**:
  - Server-Sent Events (SSE) - **Recommended** when WebSocket disabled
  - Polling (default 2s interval) - More expensive, not recommended
- **Configuration**: Can disable WebSockets via `disableWebSockets: true`
- **Known Issues**: "Overzealous emission of WebSocket offline and network errors" - SDK handles recovery automatically

**Matches Architecture Docs**: ✅ Confirms `clientsubscription` service mentioned in architecture docs.

---

## Voice Integration Details

### Voice Session Detection

**From SDK Guide:**
- Voice sessions detected via `voiceManager.onVoiceSession()` callback
- **Session Properties**:
  - `id`: Voice conversation ID
  - `status`: "started", "ended", "on-hold", "unknown"
  - `startTime`: ISO date string
  - `endTime`: ISO date string (null if unavailable)

**Key Points:**
- Voice integrations require **server-to-server** API calls for audio streaming (separate from SDK)
- SDK is responsible for **detecting conversations** and **displaying AI features** (hints, transcription, summarization)
- Only **one voice session** can be active at a time
- Profile ID required: `client.voiceManager("PROFILE_ID")`

**Matches Architecture**: ✅ Confirms voice session detection pattern; audio streaming is server-side (KVS → gowalter).

---

## Chat Integration Details

### Chat Session Management

**From SDK Guide:**
- Chat sessions initialized: `client.chatManager(profileId).chat(chatId)`
- **Session Start**: Requires visitor information, optional historical messages
- **Agent Assignment**: `chat.assignToCurrentAgent({ displayName })`
- **Visibility Control**: Can set `chat.visibility = 'hidden'/'visible'` to pause/resume polling
- **Message Sending**: `chat.addMessage()` with speaker role, text, timestamps

**Key Difference from Voice:**
- **Chat**: Events/messages sent to Cresta **via client SDK** (frontend)
- **Voice**: Audio streamed **server-to-server** (separate from SDK)

---

## Network & Error Handling

### Network Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| **Timeout** | Browser default | Network request timeout (can set to e.g., 30s) |
| **Retry Delay** | 2 seconds | Delay before retrying failed requests (can disable with negative number) |
| **WebSocket** | Enabled | ClientSubscription WebSocket (can disable, use SSE/polling) |
| **Polling Interval** | 2 seconds | When WebSocket disabled, polling interval |

### Error Manager

- Subscribes to background auth errors, API errors, network errors, UI component errors
- **Known Issue**: WebSocket offline/network errors may be overzealous; SDK handles recovery automatically
- Errors logged to console if no handlers subscribed

---

## UI Components

### Built-in Components

1. **Smart Compose** (Standalone or Overlay)
   - Auto-complete hints as agent types
   - Shortcuts: Right Arrow (word-by-word), Tab (full hint)
   - Customizable keyboard shortcuts

2. **Hints** (Behavioral suggestions)
   - Types: `standardHint`, `knowledgeBaseHint`, `guidedWorkflowHint`, `complianceHint`
   - Auto-dismiss after duration (typically 30s)
   - Can be manually dismissed

3. **Reply Suggestions**
   - AI-generated (1-3 suggestions)
   - Click to populate in Smart Compose

4. **Canned Suggestions**
   - Preconfigured in Cresta Director
   - Triggered by conversation criteria
   - Similar appearance to reply suggestions

5. **Omni Search**
   - Knowledge Base search
   - Guided Workflow search
   - Generative Knowledge Assist (beta)

### Headless Mode (Build Your Own Components)

- Event-based APIs available for custom UI
- Must trigger analytics signals (`show()`, `select()`, `dismiss()`, etc.)
- Compliance testing available via emulator (`--run-compliance-tests`)

---

## Integration Patterns

### Chat Integration Pattern
1. Initialize client with config
2. Authenticate (popup or BYOID)
3. Initialize chat manager with profile ID
4. Start chat session with visitor info
5. Assign to current agent
6. Mount UI components or use headless APIs
7. Send messages via `addMessage()`
8. End session with `chat.end()`

### Voice Integration Pattern
1. Initialize client with config
2. Authenticate (popup or BYOID)
3. Initialize voice manager with profile ID
4. Listen for voice sessions via `onVoiceSession()`
5. Mount UI components (hints, omni-search)
6. **Note**: Audio streaming is server-side (not SDK responsibility)

---

## Validation Against Architecture Docs

| Architecture Doc Claim | SDK Guide Validation | Status |
|------------------------|----------------------|--------|
| `clientsubscription` service | ✅ Confirmed: "ClientSubscription WebSocket service" | ✅ Validated |
| WebSocket event delivery | ✅ Confirmed: WebSocket for real-time events | ✅ Validated |
| Agent App integration | ✅ Confirmed: SDK provides UI components or headless APIs | ✅ Validated |
| Profile ID requirement | ✅ Confirmed: Required for chat/voice managers | ✅ Validated |
| Voice session detection | ✅ Confirmed: `onVoiceSession()` callback | ✅ Validated |
| API endpoint pattern | ✅ Confirmed: `api-CUSTOMER_ID.cresta.com` | ✅ Validated |
| Authentication methods | ✅ Expanded: BYOID, SAML, Twilio Flex, username/password | ✅ Expanded |

---

## New Information Not in Architecture Docs

1. **SDK Package Versions**: Current versions documented
2. **WebSocket Fallback**: SSE and polling options when WebSocket disabled
3. **Network Configuration**: Timeout and retry settings
4. **Error Manager**: Centralized error handling
5. **Headless Mode**: Event-based APIs for custom components
6. **Compliance Testing**: Emulator compliance test mode
7. **Twilio Flex Integration**: Direct token exchange (no BYOID needed)
8. **reCAPTCHA**: Required for inline username/password auth
9. **Authorized Origins**: Security measure for popup auth
10. **SharedWorker Support**: For BYOID token management across tabs

---

## Gaps & Questions

1. **SDK Guide Age**: Unknown – confirm with Cresta that this is current version
2. **Amazon Connect Specific**: Guide mentions Twilio Flex but not Amazon Connect-specific patterns
3. **Audio Streaming**: Guide notes voice audio is "server-to-server" but doesn't detail KVS integration
4. **Agent App Deployment**: Guide covers SDK usage but not deployment method (browser extension, desktop app, embedded)
5. **Profile ID Source**: Guide says "provided by Cresta team" but doesn't explain mapping to Connect queues/attributes

---

## Recommendations

1. **Confirm SDK Guide Currency**: Verify with Cresta that this is the latest version
2. **Request Amazon Connect SDK Examples**: Ask for Connect-specific integration patterns (if different from general SDK)
3. **Profile ID Mapping**: Clarify how Profile ID maps to Connect queues/attributes
4. **Agent App Deployment**: Confirm deployment method for Connect integration
5. **Update Architecture Docs**: Add SDK details to 02-amazon-connect-integration.md and 01-overall-architecture.md

---

## References

- **SDK Guide**: `Cresta Client SDK Developer Guide.pdf` (extracted text)
- **Architecture Docs**: 01-overall-architecture.md, 02-amazon-connect-integration.md, 05-realtime-dataflow-sequences.md
- **NPM Registry**: https://npm.cresta.com
