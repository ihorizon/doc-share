# Data Storage & Security Architecture

## Legend
- 🔒 **Security Risk** - Data protection, authentication, encryption concerns
- ⏱️ **Latency Risk** - Real-time performance critical path
- 📋 **Compliance Risk** - GDPR, PCI-DSS, HIPAA considerations
- ⚙️ **Operational Risk** - Availability, scaling, monitoring concerns
- 🟡 **Yellow/Orange** - Requires follow-up/verification

---

## Data Architecture Overview

```mermaid
flowchart TB
    subgraph Outer[" "]
        subgraph DataSources["Data Sources"]
            direction LR
            Audio["Audio Streams"]
            Transcripts["Transcripts"]
            Annotations["ML Annotations"]
            AgentActions["Agent Actions"]
            Config["Configuration"]
        end

        subgraph DataLayer["Data Storage Layer"]
            direction TB
            subgraph Operational["Operational Stores"]
                direction LR
                Postgres[(📋 🔒 PostgreSQL<br/>Conversations, Transcripts,<br/>Users, Policies)]
                Redis[(Redis<br/>Event Streams,<br/>Session Cache, Policy Cache)]
            end

            subgraph Analytics["Analytics Stores"]
                direction LR
                ES[(Elasticsearch<br/>Conversation Search,<br/>Full-text Index)]
                ClickHouse[(ClickHouse<br/>Analytics, Metrics,<br/>Aggregations)]
            end

            subgraph ObjectStorage["Object Storage"]
                direction LR
                S3Audio[(🔒 📋 S3<br/>Audio Files<br/>Encrypted)]
                S3Models[(S3<br/>Model Artifacts)]
            end
        end

        subgraph DataIsolation["Customer Data Isolation"]
            direction LR
            TenantDB["🔒 Separate Databases<br/>per Customer"]
            TenantBucket["🔒 Separate S3 Paths<br/>per Customer"]
            TenantIndex["🔒 Separate ES Indices<br/>per Customer"]
        end

        Audio ==>|"Store"| S3Audio
        Transcripts ==>|"Persist"| Postgres
        Transcripts ==>|"Index"| ES
        Annotations ==>|"Store"| Postgres
        Annotations ==>|"Analytics"| ClickHouse
        AgentActions ==>|"Analytics"| ClickHouse
        Config ==>|"Store"| Postgres
        Config ==>|"Cache"| Redis

        Postgres ==>|"Isolated by"| TenantDB
        S3Audio ==>|"Isolated by"| TenantBucket
        ES ==>|"Isolated by"| TenantIndex
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style DataSources fill:none,stroke:#9ca3af,stroke-width:1.5px
    style DataLayer fill:none,stroke:#9ca3af,stroke-width:1.5px
    style DataIsolation fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Operational fill:none,stroke:#9ca3af,stroke-width:1px
    style Analytics fill:none,stroke:#9ca3af,stroke-width:1px
    style ObjectStorage fill:none,stroke:#9ca3af,stroke-width:1px
    
    %% Node styling - very light gray backgrounds
    style Audio fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Transcripts fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Annotations fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AgentActions fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Config fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Postgres fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style Redis fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style ES fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style ClickHouse fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style S3Audio fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style S3Models fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style TenantDB fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style TenantBucket fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style TenantIndex fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
```

---

## Security Architecture

```mermaid
flowchart TB
    subgraph Outer[" "]
        subgraph Perimeter["Perimeter Security"]
            direction LR
            WAF["🔒 Wallarm WAF"]
            DDoS["🔒 DDoS Protection"]
            TLS["🔒 TLS 1.2+ Termination"]
            
            WAF ==> TLS
        end

        subgraph Network["Network Security"]
            direction TB
            VPC["AWS VPC"]
            PrivateSubnet["Private Subnets"]
            SecurityGroups["Security Groups"]
            NACLs["Network ACLs"]
            
            TLS ==> VPC ==> PrivateSubnet ==> SecurityGroups
        end

        subgraph Identity["Identity & Access"]
            direction TB
            IAM["🔒 AWS IAM Roles"]
            RBAC["🔒 Cresta RBAC"]
            SSO["🔒 SSO Integration<br/>Verify"]
            MFA["🔒 MFA<br/>Verify"]
            
            IAM ==> RBAC ==> SSO ==> MFA
        end

        subgraph DataProtection["Data Protection"]
            direction TB
            KMS["🔒 AWS KMS<br/>Encryption Keys"]
            AtRest["🔒 Encryption at Rest<br/>AES-256"]
            InTransit["🔒 Encryption in Transit<br/>TLS"]
            PIIRedaction["📋 🔒 PII Redaction"]
            
            KMS ==> AtRest
            KMS ==> InTransit
            AtRest ==> PIIRedaction
        end

        subgraph Monitoring["Security Monitoring"]
            direction LR
            AuditLogs["Audit Logs"]
            SIEM["SIEM Integration<br/>Verify"]
            Alerts["Security Alerts"]
            
            AuditLogs ==> SIEM ==> Alerts
        end
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style Perimeter fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Network fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Identity fill:none,stroke:#9ca3af,stroke-width:1.5px
    style DataProtection fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Monitoring fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style WAF fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style DDoS fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style TLS fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style VPC fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style PrivateSubnet fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style SecurityGroups fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style NACLs fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style IAM fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style RBAC fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style SSO fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style MFA fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style KMS fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style AtRest fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style InTransit fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style PIIRedaction fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style AuditLogs fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style SIEM fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Alerts fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
```

---

## PII Redaction Pipeline

```mermaid
flowchart TB
    subgraph Outer[" "]
        subgraph Input["Input Data"]
            RawAudio["Raw Audio"]
            RawTranscript["Raw Transcript"]
        end

        subgraph Detection["PII Detection"]
            NER["📋 🔒 Named Entity<br/>Recognition"]
            Regex["📋 🔒 Regex Patterns<br/>SSN, CC, Phone"]
            MLDetect["📋 🔒 ML-based<br/>Detection"]
        end

        subgraph PIITypes["Detected PII Types"]
            Names["Full Names<br/>[FULLNAME]"]
            SSN["SSN<br/>[SSN]"]
            CreditCard["Credit Card<br/>[CREDITCARD]"]
            Phone["Phone Numbers<br/>[PHONE]"]
            Address["Addresses<br/>[ADDRESS]"]
            DOB["Date of Birth<br/>[DOB]"]
        end

        subgraph Redaction["Redaction Actions"]
            TextRedact["📋 🔒 Text Masking<br/>Replace with tags"]
            AudioRedact["📋 🔒 Audio Beeping<br/>Mute segments"]
        end

        subgraph Verification["Verification"]
            Rescan["📋 🔒 Re-scan for<br/>Missed PII"]
            Decision{PII Found?}
            Manual["Manual Review<br/>Queue"]
            Alert["⚙️ Alert on<br/>Failure"]
            Complete["Mark<br/>Complete"]
        end

        subgraph Output["Clean Data"]
            CleanAudio["Redacted Audio<br/>→ S3"]
            CleanTranscript["Redacted Transcript<br/>→ PostgreSQL"]
        end

        RawAudio ==>|"Process"| NER
        RawTranscript ==>|"Process"| NER
        RawAudio ==>|"Match Patterns"| Regex
        RawTranscript ==>|"Match Patterns"| Regex
        RawAudio ==>|"ML Detection"| MLDetect
        RawTranscript ==>|"ML Detection"| MLDetect

        NER ==>|"Detected"| Names
        NER ==>|"Detected"| Address
        Regex ==>|"Detected"| SSN
        Regex ==>|"Detected"| CreditCard
        Regex ==>|"Detected"| Phone
        MLDetect ==>|"Detected"| DOB

        Names ==>|"Apply"| TextRedact
        Names ==>|"Apply"| AudioRedact
        SSN ==>|"Apply"| TextRedact
        SSN ==>|"Apply"| AudioRedact
        CreditCard ==>|"Apply"| TextRedact
        CreditCard ==>|"Apply"| AudioRedact

        TextRedact ==>|"Output"| CleanTranscript
        AudioRedact ==>|"Output"| CleanAudio

        CleanAudio ==>|"Verify"| Rescan
        CleanTranscript ==>|"Verify"| Rescan
        Rescan ==> Decision
        Decision ==>|"Yes"| Manual
        Decision ==>|"Max Retries"| Alert
        Decision ==>|"No"| Complete
        Manual ==> Rescan
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style Input fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Detection fill:none,stroke:#9ca3af,stroke-width:1.5px
    style PIITypes fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Redaction fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Verification fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Output fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style RawAudio fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style RawTranscript fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style NER fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style Regex fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style MLDetect fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Names fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style SSN fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style CreditCard fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Phone fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Address fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style DOB fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style TextRedact fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style AudioRedact fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style Rescan fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style Decision fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Manual fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Alert fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style Complete fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style CleanAudio fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style CleanTranscript fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
```

---

## Regional Data Residency

```mermaid
flowchart TB
    subgraph Outer[" "]
        subgraph Customers["Customer Regions"]
            USCustomer["US Customer"]
            EUCustomer["EU Customer"]
            APACCustomer["APAC Customer<br/>Verify"]
        end

        subgraph Routing["DNS-Based Routing"]
            USSubdomain["customer.us-west-2.cresta.ai"]
            EUSubdomain["customer.eu-west-1.cresta.ai"]
            APACSubdomain["customer.ap-southeast-1.cresta.ai<br/>Verify"]
        end

        subgraph USRegion["US-West-2 Region"]
            USCluster["EKS Cluster"]
            USRDS[(PostgreSQL)]
            USS3[(S3)]
        end

        subgraph EURegion["EU-West-1 Region"]
            EUCluster["EKS Cluster"]
            EURDS[(PostgreSQL)]
            EUS3[(S3)]
        end

        subgraph APACRegion["AP-Southeast-1 Region"]
            APACCluster["EKS Cluster<br/>Verify"]
            APACRDS[(PostgreSQL<br/>Verify)]
            APACS3[(S3<br/>Verify)]
        end

        USCustomer ==>|"Route"| USSubdomain
        EUCustomer ==>|"Route"| EUSubdomain
        APACCustomer ==>|"Route"| APACSubdomain

        USSubdomain ==>|"DNS"| USCluster
        EUSubdomain ==>|"DNS"| EUCluster
        APACSubdomain ==>|"DNS"| APACCluster

        USCluster ==>|"Store"| USRDS
        USCluster ==>|"Store"| USS3
        EUCluster ==>|"Store"| EURDS
        EUCluster ==>|"Store"| EUS3
        APACCluster ==>|"Store"| APACRDS
        APACCluster ==>|"Store"| APACS3
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style Customers fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Routing fill:none,stroke:#9ca3af,stroke-width:1.5px
    style USRegion fill:none,stroke:#9ca3af,stroke-width:1.5px
    style EURegion fill:none,stroke:#9ca3af,stroke-width:1.5px
    style APACRegion fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style USCustomer fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style EUCustomer fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style APACCustomer fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style USSubdomain fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style EUSubdomain fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style APACSubdomain fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style USCluster fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style USRDS fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style USS3 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style EUCluster fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style EURDS fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style EUS3 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style APACCluster fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style APACRDS fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
    style APACS3 fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
```

---

## Compliance Framework

```mermaid
flowchart LR
    subgraph Outer[" "]
        subgraph Certifications["Certifications"]
            SOC2["SOC 2 Type II"]
            ISO27001["ISO 27001"]
            ISO27701["ISO 27701<br/>Privacy"]
            ISO42001["ISO 42001<br/>AI Management"]
            PCIDSS["PCI-DSS"]
            HIPAA["HIPAA<br/>BAA Available"]
        end

        subgraph Controls["Key Controls"]
            Encryption["Encryption<br/>At Rest + Transit"]
            AccessControl["Access Control<br/>RBAC + IAM"]
            AuditTrail["Audit Trail<br/>All Actions Logged"]
            DataRetention["Data Retention<br/>Policies"]
            IncidentResponse["Incident Response<br/>Plan"]
            PenTest["Annual Pen Testing"]
        end

        subgraph Compliance["Compliance Areas"]
            Privacy["Privacy<br/>GDPR, CCPA"]
            Financial["Financial<br/>PCI-DSS"]
            Healthcare["Healthcare<br/>HIPAA"]
            AI["AI Governance<br/>Fairness, Transparency"]
        end

        SOC2 ==>|"Requires"| AccessControl
        SOC2 ==>|"Requires"| AuditTrail
        ISO27001 ==>|"Requires"| Encryption
        ISO27001 ==>|"Requires"| IncidentResponse
        ISO27701 ==>|"Covers"| Privacy
        ISO42001 ==>|"Covers"| AI
        PCIDSS ==>|"Covers"| Financial
        HIPAA ==>|"Covers"| Healthcare
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style Certifications fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Controls fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Compliance fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style SOC2 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style ISO27001 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style ISO27701 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style ISO42001 fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style PCIDSS fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style HIPAA fill:#fcfcfd,stroke:#059669,stroke-width:2.5px
    style Encryption fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AccessControl fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AuditTrail fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style DataRetention fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style IncidentResponse fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style PenTest fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Privacy fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Financial fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Healthcare fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AI fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
```

---

## Data Retention & Lifecycle

```mermaid
flowchart TB
    subgraph Outer[" "]
        subgraph DataTypes["Data Types"]
            AudioData["Audio Files"]
            TranscriptData["Transcripts"]
            AnnotationData["Annotations"]
            AnalyticsData["Analytics"]
        end

        subgraph Retention["Retention Periods"]
            AudioRetention["Audio: Customer-defined<br/>Default: 90 days"]
            TranscriptRetention["Transcripts: Customer-defined<br/>Default: 1 year"]
            AnnotationRetention["Annotations: Customer-defined"]
            AnalyticsRetention["Analytics: Aggregated<br/>Indefinite"]
        end

        subgraph Lifecycle["Lifecycle Actions"]
            Active["Active Storage"]
            Archive["Archive (S3 Glacier)"]
            Delete["Secure Deletion"]
        end

        AudioData ==>|"Retain"| AudioRetention
        TranscriptData ==>|"Retain"| TranscriptRetention
        AnnotationData ==>|"Retain"| AnnotationRetention
        AnalyticsData ==>|"Retain"| AnalyticsRetention

        AudioRetention ==>|"After Period"| Active
        TranscriptRetention ==>|"After Period"| Active
        Active ==>|"Archive"| Archive
        Archive ==>|"After Archive Period"| Delete
    end
    
    %% Outer container with white background
    style Outer fill:#ffffff,stroke:#d1d5db,stroke-width:2px
    
    %% Transparent group backgrounds with light gray borders
    style DataTypes fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Retention fill:none,stroke:#9ca3af,stroke-width:1.5px
    style Lifecycle fill:none,stroke:#9ca3af,stroke-width:1.5px
    
    %% Node styling - very light gray backgrounds
    style AudioData fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style TranscriptData fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AnnotationData fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AnalyticsData fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style AudioRetention fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style TranscriptRetention fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style AnnotationRetention fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style AnalyticsRetention fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style Active fill:#fcfcfd,stroke:#1f2937,stroke-width:2px
    style Archive fill:#fcfcfd,stroke:#d97706,stroke-width:2.5px
    style Delete fill:#fcfcfd,stroke:#dc2626,stroke-width:2px
```

---

## Risk Summary

### 🔒 Security Risks

| Risk | Severity | Status | Mitigation |
|------|----------|--------|------------|
| Data breach | High | Mitigated | Encryption, access controls |
| Unauthorized access | High | Mitigated | RBAC, MFA, audit logs |
| PII exposure | High | Mitigated | Auto-redaction + verification |
| Cross-tenant access | High | Mitigated | Database isolation |

### 📋 Compliance Risks

| Risk | Severity | Status | Mitigation |
|------|----------|--------|------------|
| GDPR violation | High | Mitigated | EU data residency |
| PCI scope creep | Medium | Mitigated | PII redaction |
| Audit failure | Medium | Mitigated | SOC 2 certification |
| Data retention violation | Medium | 🟡 Verify | Customer-configurable retention |

### ⚙️ Operational Risks

| Risk | Severity | Status | Mitigation |
|------|----------|--------|------------|
| Data loss | High | Mitigated | S3 durability, backups |
| Slow search | Medium | Mitigated | Elasticsearch indexing |
| Storage costs | Low | Monitor | Lifecycle policies |

---

## Items Requiring Follow-up 🟡

1. **SSO Providers** - Which SSO providers are supported (Okta, Azure AD, etc.)?
2. **SIEM Integration** - What SIEM platforms can ingest Cresta logs?
3. **Data Retention Defaults** - What are the default retention periods?
4. **APAC Region** - Is AP-Southeast-1 available for data residency?
5. **Archive Strategy** - Is S3 Glacier used for long-term audio storage?
6. **Deletion Process** - What is the process for customer data deletion requests?
7. **Backup RPO/RTO** - What are the backup and recovery objectives?

---

## Summary

This document describes the data storage architecture, security controls, compliance framework, and data lifecycle management for the Cresta platform.

**Data Storage Architecture**:
- **Operational Stores**: PostgreSQL (conversations, transcripts, users, policies), Redis (event streams, session cache, policy cache)
- **Analytics Stores**: Elasticsearch (conversation search, full-text index), ClickHouse (analytics, metrics, aggregations)
- **Object Storage**: AWS S3 (encrypted audio files, model artifacts)
- **Customer Isolation**: Separate databases, S3 paths, and ES indices per customer

**Security Controls**:
- **Perimeter**: Wallarm WAF, DDoS protection, TLS 1.2+ termination
- **Network**: AWS VPC, private subnets, security groups, NACLs
- **Identity**: AWS IAM roles, Cresta RBAC, SSO integration (providers require verification), MFA (requires verification)
- **Data Protection**: AWS KMS encryption keys, AES-256 at rest, TLS in transit, PII redaction

**PII Redaction Pipeline**:
- **Detection**: Named Entity Recognition (NER), regex patterns (SSN, credit card, phone), ML-based detection
- **Redaction**: Text masking (replace with tags), audio beeping (mute segments)
- **Verification**: Temporal workflow re-scans for missed PII, retry on failure, manual review queue

**Compliance Framework** (Confirmed via Cresta Trust Center):
- **Certifications**: SOC 2 Type II, ISO 27001 (Information Security), ISO 27701 (Privacy), ISO 42001 (AI Management), PCI-DSS, HIPAA (BAA available)
- **Regional Data Residency**: US (us-west-2), EU (eu-west-1) confirmed; APAC (ap-southeast-1) requires verification

**Data Retention & Lifecycle**:
- **Retention**: Customer-configurable (defaults require verification)
- **Lifecycle**: Active storage → Archive (S3 Glacier assumed, requires verification) → Secure deletion

**Verification Status**: Compliance certifications confirmed via Cresta Trust Center. Data isolation approach consistent with Cresta blog. SSO providers, SIEM integration, APAC region availability, data retention defaults, archive strategy, and backup RPO/RTO require Cresta confirmation.
