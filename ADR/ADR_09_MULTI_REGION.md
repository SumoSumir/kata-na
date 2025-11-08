# ADR-09: Multi-Region Deployment and Region-Wise AI Model Strategy

## Status
Accepted

## Context
MobilityCorp’s AI services operate across multiple countries in Europe (e.g., EU). Regional deployment of AI workloads is essential to meet latency, data compliance, and availability requirements.

Challenges include:
- **Latency:** Centralized inference increases response time for distant regions.
- **Compliance:** Data privacy laws (GDPR, FDPA Act) require local data residency.
- **Availability:** Outages or throttling in one region should not impact global operations.
- **Model Variability:** Certain AI models or providers may perform better or be allowed only in specific regions.
- **Cost Optimization:** Different regions have varying compute and provider costs.

To ensure compliance, performance, and resilience, the AI platform needs a multi-region strategy with localized deployments and model routing.

## Decision
Adopt a **multi-region active-active deployment architecture** with **region-aware AI model selection** on AWS.

**AWS Multi-Region Architecture:**

1. **Primary Regions**
   - **eu-central-1 (Frankfurt):** Primary EU region for Germany, Central Europe
   - **eu-west-1 (Ireland):** Secondary EU region for UK, Western Europe
   - **eu-west-3 (Paris):** Future expansion for France-specific compliance
   - Active-active: Both regions serve production traffic with local data residency

2. **Data Layer (Regional Isolation)**
   - **Aurora PostgreSQL Global Database**
     - Primary cluster in each region (eu-central-1, eu-west-1)
     - Cross-region replication RPO < 1s, RTO < 1 min
     - Regional read replicas for low-latency queries
     - Failover: Automatic promotion of secondary to primary
   - **DynamoDB Global Tables**
     - Multi-region replication for session data, user preferences
     - Last-writer-wins conflict resolution
     - Replication latency < 1s
   - **S3 Cross-Region Replication (CRR)**
     - Vehicle images, ML artifacts replicated to secondary region
     - Versioning enabled for compliance audit trails
     - Replication time control (RTC) for 99.99% replication within 15 mins

3. **Compute Layer (Regional Isolation)**
   - **ECS Fargate clusters** in each region
     - Microservices deployed independently per region
     - Auto-scaling based on regional traffic patterns
   - **Lambda functions** with regional endpoints
     - Real-time processing stays within region boundaries
   - **SageMaker endpoints** per region for AI inference (see Model Registry below)

4. **Traffic Routing**
   - **Amazon Route 53 Geolocation Routing**
     - EU users → eu-central-1 or eu-west-1 based on latency
     - Failover: Health checks promote secondary region within 30s
     - Example: `api.mobilitycorp.eu` resolves to nearest healthy region
   - **AWS Global Accelerator (optional)**
     - Anycast IPs for faster failover (< 1 min RTO)
     - TCP/UDP performance optimization
   - **Application Load Balancer (ALB)** in each region
     - Regional endpoint with health checks on microservices
     - Sticky sessions for booking flows

5. **AI Model Registry per Region**
   - **SageMaker Model Registry** in each region
     - Regional model versions: `eu-central-1/booking-demand-forecast:v1.2.3`
     - Provider selection: Bedrock (Claude 3.5) in eu-central-1, eu-west-1
     - Fallback: If Bedrock throttles, route to OpenAI (with compliance logging)
   - **Model synchronization:** AWS CodePipeline with cross-region deployment
     - Canary deployment: 10% traffic → 50% → 100% (per region)
     - Blue-green model rollback within 5 mins
   - **Regional training:** SageMaker training jobs use regional S3 buckets for data residency

6. **Compliance & Data Residency**
   - **GDPR enforcement:** User data never leaves EU regions
   - **Encryption:** AES-256 in-transit (TLS 1.3) and at-rest (KMS regional keys)
   - **KMS keys per region:** `eu-central-1-kms-key`, `eu-west-1-kms-key`
   - **CloudTrail regional logs:** Audit trail stored in regional S3 buckets (7-year retention)
   - **Bedrock data residency:** All inference requests processed within selected region

7. **Failover Policy**
   - **Automated failover:** Route 53 health checks → promote secondary region
   - **Manual failover:** For planned maintenance (< 30s downtime)
   - **Compliance exceptions:** If primary region unavailable, secondary region logs compliance override
   - **Data consistency:** Aurora cross-region replication ensures < 1s lag

**Cost Estimate (Multi-Region):**
- Cross-region data transfer: ~$0.02/GB (inter-region)
- Aurora Global Database: +20% over single-region Aurora ($4,800/month base → $5,760/month)
- DynamoDB Global Tables: +2x write capacity costs
- S3 CRR: $0.02/GB replicated
- Route 53: $0.50/million queries ($100/month for 200M queries)
- **Total multi-region overhead:** ~$15,000-20,000/month additional

**Model Provider Strategy:**
- **EU regions (eu-central-1, eu-west-1):** AWS Bedrock (Claude 3.5 Sonnet) - GDPR compliant, data stays in EU
- **Fallback:** OpenAI GPT-4o (with explicit user consent for US data processing)
- **Future US region:** OpenAI primary, Bedrock secondary
- **Provider selection logic:** Region-based routing with compliance rules in DynamoDB

## Consequences

✅ **Positive**
- Reduced latency through regional inference  
- Compliance with local data protection laws  
- Improved fault tolerance and service continuity  
- Flexibility to use the best or allowed provider per region  
- Enables region-based A/B testing and performance optimization  

❌ **Negative**
- Increased infrastructure and DevOps complexity  
- Higher cost for maintaining multiple regional environments  
- Model version drift risk across regions  
- More complex monitoring and synchronization  

## Alternatives Considered

**Single Global Deployment**  
- Pros: Easier to maintain, lower operational cost  
- Cons: Violates data residency laws, higher latency for distant users  
- Rejected: Non-compliant for EU and India data protection  

**Multi-Region Data Only, Centralized AI Models**  
- Pros: Partial compliance for data storage  
- Cons: Inference still crosses borders, latency remains high  
- Rejected: Does not fully meet regulatory or performance goals  

## AI-Specific Considerations
- Each region maintains its own model registry and provider mapping.  
- Model performance monitoring is region-scoped to capture local variations.  
- Training data aggregation may require anonymization before cross-region transfers.  
- Supports progressive rollout of new models region by region.  
- Facilitates compliance audits with localized model and data control.
