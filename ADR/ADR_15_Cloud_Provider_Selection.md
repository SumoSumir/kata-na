# ADR-15: Cloud Provider Selection - AWS

## Status
Accepted

## Context
MobilityCorp requires a cloud platform to host its AI-enabled multi-modal transportation platform across multiple EU countries. The platform must support:

- High-volume real-time telemetry ingestion (50K+ vehicles)
- Advanced AI/ML capabilities (demand forecasting, predictive maintenance, computer vision)
- GDPR compliance with EU data residency requirements
- Multi-region deployment for high availability
- Cost-effective scaling from pilot to production
- Comprehensive IoT device management
- Real-time analytics and data warehousing

## Business Requirements

### Regulatory & Compliance
- **GDPR Compliance:** Data residency in EU, right to be forgotten, data portability
- **Data Sovereignty:** Customer and operational data must remain in EU
- **Audit Trails:** Complete logging and monitoring for compliance

### Technical Requirements
- **Scalability:** Support 100K+ concurrent users and 50K vehicles
- **Availability:** 99.95% uptime for core services
- **Latency:** P95 API latency < 200ms, vehicle unlock < 3 seconds
- **AI/ML:** Production-ready ML platform with AutoML, feature store, model serving
- **IoT:** Secure device connectivity, OTA updates, edge computing
- **Data:** Real-time streaming, data lake, data warehouse, caching

### Business Constraints
- **Budget:** ~$100K/month infrastructure costs
- **Timeline:** 11-month implementation (Phase 1-5)
- **Skills:** Team familiar with cloud-native architectures, moderate cloud experience
- **Vendor Lock-in:** Minimize but accept pragmatic choices for best services

## Alternatives Considered

### Alternative 1: Amazon Web Services (AWS)

#### Strengths
✅ **AI/ML Excellence:**
- SageMaker: Comprehensive ML platform (AutoML, pipelines, feature store, model registry)
- Bedrock: Managed LLM service with Claude, Llama, Titan models
- Rekognition: Pre-trained computer vision models
- Forecast: Time-series forecasting service
- Comprehend: NLP for feedback analysis

✅ **IoT & Edge:**
- IoT Core: Scalable MQTT broker with device management
- IoT Greengrass: Edge runtime with local ML inference
- IoT FleetWise: Vehicle data collection optimized for automotive
- IoT Device Defender: Security monitoring for IoT devices

✅ **Data Platform:**
- MSK (Managed Kafka): Fully managed Apache Kafka
- Kinesis: Real-time streaming alternative
- S3: Industry-standard object storage with data lake support
- Timestream: Purpose-built time-series database
- Redshift: Mature data warehouse with Spectrum for S3 queries
- DynamoDB: Low-latency NoSQL with global tables

✅ **EU Presence:**
- Frankfurt (eu-central-1): Primary region
- Ireland (eu-west-1): Secondary region
- Paris (eu-west-3): Tertiary option
- Milan (eu-south-1): Additional coverage

✅ **Cost Optimization:**
- Spot Instances: 70-90% savings for batch ML training
- Savings Plans: Committed use discounts
- S3 Intelligent-Tiering: Automatic storage class optimization
- Graviton2/3: ARM processors with 40% better price-performance

✅ **Maturity & Ecosystem:**
- Largest cloud provider (32% market share)
- Most comprehensive service catalog (200+ services)
- Extensive third-party integrations
- Largest community and knowledge base

#### Weaknesses
❌ **Complexity:**
- Steep learning curve with service proliferation
- IAM complexity can lead to misconfigurations
- Multiple overlapping services (Kinesis vs MSK, Aurora vs RDS)

❌ **Cost Transparency:**
- Complex pricing models
- Easy to incur unexpected charges
- Data transfer costs between AZs

❌ **Support:**
- Basic support requires paid plans
- Enterprise support expensive ($15K+/month minimum)

#### Cost Estimate (Monthly, Production)
```
Compute (ECS Fargate + Lambda):        $35,000
SageMaker (Training + Endpoints):      $28,000
Data Storage (S3 + RDS + DynamoDB):    $12,000
Streaming (MSK):                        $6,000
Networking (VPC + Transit + API GW):    $8,000
IoT (IoT Core + Greengrass):           $5,000
Monitoring (CloudWatch + X-Ray):        $3,000
Security (KMS + Secrets + WAF):         $2,000
Other (Backup, Support, etc.):          $1,000
────────────────────────────────────────────
TOTAL:                                ~$100,000
```

#### Scoring (out of 10)
- AI/ML Capabilities: 10/10
- IoT Platform: 10/10
- EU Compliance: 9/10
- Cost: 8/10
- Developer Experience: 7/10
- Support: 8/10
- **Overall: 9.0/10**

---

### Alternative 2: Microsoft Azure

#### Strengths
✅ **AI/ML Excellence:**
- Azure Machine Learning: Comprehensive platform with AutoML
- Azure AI Foundry: Agent orchestration and GenAI
- Cognitive Services: Pre-built vision, speech, language APIs
- Azure OpenAI: Access to GPT-4, GPT-3.5, DALL-E
- Synapse Analytics: Integrated analytics platform

✅ **Enterprise Integration:**
- Best Office 365/Microsoft ecosystem integration
- Active Directory integration
- Strong enterprise support and partnerships
- Hybrid cloud capabilities (Azure Arc)

✅ **Data Platform:**
- Cosmos DB: Globally distributed NoSQL with multiple APIs
- Fabric: Unified data analytics platform (OneLake)
- Event Hubs: Managed Kafka-compatible streaming
- Azure Data Lake: Hierarchical namespace for big data

✅ **EU Presence:**
- West Europe (Netherlands): Primary
- North Europe (Ireland): Secondary
- France Central: Additional
- Germany West Central: Additional

✅ **Cost:**
- Competitive pricing with AWS
- Azure Hybrid Benefit for Windows workloads
- Reserved Instances with flexible cancellation

#### Weaknesses
❌ **IoT Maturity:**
- IoT Hub less mature than AWS IoT Core
- Edge runtime (IoT Edge) less feature-rich than Greengrass
- Smaller IoT ecosystem

❌ **Service Stability:**
- Newer services sometimes have reliability issues
- More frequent breaking changes
- Documentation quality varies

❌ **Market Share:**
- 23% market share (smaller ecosystem)
- Fewer third-party integrations than AWS

#### Cost Estimate (Monthly, Production)
```
Compute (AKS + Functions):             $38,000
ML (Azure ML + AI Services):           $30,000
Data (Cosmos + SQL + Storage):         $14,000
Streaming (Event Hubs):                 $7,000
Networking + API Management:            $9,000
IoT (IoT Hub + Edge):                   $6,000
Monitoring (Monitor + Insights):        $3,000
Security (Key Vault + Sentinel):        $2,500
Other:                                  $1,500
────────────────────────────────────────────
TOTAL:                                ~$111,000
```

#### Scoring (out of 10)
- AI/ML Capabilities: 9/10
- IoT Platform: 7/10
- EU Compliance: 9/10
- Cost: 7/10
- Developer Experience: 8/10
- Support: 9/10
- **Overall: 8.2/10**

---

### Alternative 3: Google Cloud Platform (GCP)

#### Strengths
✅ **AI/ML Leadership:**
- Vertex AI: Most advanced unified ML platform
- AutoML: Best-in-class automated machine learning
- Gemini: State-of-the-art LLM with multimodal capabilities
- TensorFlow: Native integration with Google's ML framework
- BigQuery ML: SQL-based ML model training

✅ **Data & Analytics:**
- BigQuery: Best-in-class data warehouse (serverless, fast)
- Dataflow: Fully managed Apache Beam for streaming/batch
- Pub/Sub: Highly scalable message broker
- Bigtable: High-performance NoSQL for time-series

✅ **Developer Experience:**
- Clean, consistent APIs
- Excellent documentation
- Kubernetes (GKE) - created and maintained by Google
- Cloud Run: Best serverless container platform

✅ **EU Presence:**
- Belgium (europe-west1): Primary
- Netherlands (europe-west4): Secondary
- Frankfurt (europe-west3): Additional
- Zurich (europe-west6): Additional

✅ **Networking:**
- Best global network performance
- Free egress to some Google services
- Premium tier networking superior to AWS/Azure

#### Weaknesses
❌ **IoT Platform:**
- IoT Core deprecated (sunset Aug 2023)
- No managed IoT platform currently
- Must build custom solution on Pub/Sub + Cloud Functions

❌ **Service Breadth:**
- Fewer services than AWS (100+ vs 200+)
- Some gaps in service catalog
- Less mature enterprise features

❌ **Market Position:**
- 10% market share (smallest of big three)
- Smaller partner ecosystem
- Fewer pre-built integrations

❌ **IoT Gap - Critical Flaw:**
- **DEALBREAKER:** No managed IoT device management
- Would require building custom MQTT broker, device auth, OTA updates
- Significant engineering effort and operational overhead

#### Cost Estimate (Monthly, Production)
```
Compute (GKE + Cloud Run):             $32,000
ML (Vertex AI + AutoML):               $26,000
Data (BigQuery + Firestore + GCS):     $11,000
Streaming (Pub/Sub + Dataflow):         $8,000
Networking + API Gateway:               $7,000
Custom IoT Solution:                   $12,000 (⚠️ custom build)
Monitoring (Cloud Monitoring):          $2,500
Security (KMS + Secret Manager):        $1,500
Other:                                  $1,000
────────────────────────────────────────────
TOTAL:                                ~$101,000
```

#### Scoring (out of 10)
- AI/ML Capabilities: 10/10
- IoT Platform: 3/10 ⚠️ **Major Gap**
- EU Compliance: 9/10
- Cost: 8/10
- Developer Experience: 10/10
- Support: 7/10
- **Overall: 7.8/10**

---

### Alternative 4: Multi-Cloud Approach

#### Concept
Use best-of-breed services from each cloud:
- **GCP:** AI/ML (Vertex AI), Data Warehouse (BigQuery)
- **AWS:** IoT (IoT Core + Greengrass), Streaming (MSK)
- **Azure:** Enterprise integration, Backup

#### Strengths
✅ Best services from each platform
✅ Avoid vendor lock-in
✅ Geographic redundancy across providers

#### Weaknesses
❌ **Operational Complexity:**
- Multiple consoles, APIs, CLIs to manage
- Cross-cloud networking challenges
- Increased security surface area
- Complex identity management

❌ **Cost:**
- Data egress charges between clouds (~$0.09/GB)
- Duplicate tooling and monitoring
- Higher staff training costs
- No bulk discounts

❌ **Reliability:**
- Cross-cloud dependencies create fragility
- Network latency between providers
- Harder to troubleshoot issues

❌ **Skill Requirements:**
- Team must learn 2-3 cloud platforms
- Harder to hire specialists
- More documentation and runbooks

#### Cost Estimate (Monthly, Production)
```
Base services across clouds:          $105,000
Cross-cloud data transfer (20TB):      $1,800
Duplicate tooling:                      $5,000
Additional operational overhead:       $8,000
────────────────────────────────────────────
TOTAL:                                ~$120,000
```

#### Scoring (out of 10)
- AI/ML Capabilities: 10/10
- IoT Platform: 10/10
- EU Compliance: 8/10
- Cost: 5/10 ⚠️
- Developer Experience: 4/10 ⚠️
- Support: 6/10
- **Overall: 7.2/10**

---

## Decision Matrix

| Criteria | Weight | AWS | Azure | GCP | Multi-Cloud |
|----------|--------|-----|-------|-----|-------------|
| **AI/ML Capabilities** | 20% | 10 | 9 | 10 | 10 |
| **IoT Platform** | 20% | 10 | 7 | 3 ⚠️ | 10 |
| **EU Compliance** | 15% | 9 | 9 | 9 | 8 |
| **Cost** | 15% | 8 | 7 | 8 | 5 ⚠️ |
| **Developer Experience** | 10% | 7 | 8 | 10 | 4 ⚠️ |
| **Service Maturity** | 10% | 10 | 8 | 7 | 8 |
| **Support & Ecosystem** | 10% | 8 | 9 | 7 | 6 |
| **Weighted Score** | | **9.0** | **8.2** | **7.8** | **7.2** |

## Decision

**We choose Amazon Web Services (AWS) as our cloud provider.**

### Primary Rationale

#### 1. IoT Platform Maturity (Critical)
- **AWS IoT Core** is production-ready with 50K+ vehicle support
- **IoT Greengrass v2** enables edge ML inference for collision detection
- **IoT FleetWise** purpose-built for automotive telemetry
- GCP's lack of managed IoT is a **dealbreaker** - would require months of custom development

#### 2. AI/ML Completeness
- **SageMaker** provides end-to-end ML lifecycle (comparable to Vertex AI)
- **Bedrock** offers Claude 3.5 for agentic AI (competitive with Azure OpenAI)
- **Rekognition** for damage detection (pre-trained models save time)
- **Forecast** for demand prediction (specialized time-series service)

#### 3. Cost-Effective at Scale
- Spot Instances save 70-90% on ML training
- S3 Intelligent-Tiering automatically optimizes storage costs
- Graviton processors provide 40% better price-performance
- **Est. $100K/month** vs $111K (Azure) or $120K (Multi-cloud)

#### 4. EU Compliance & Presence
- Frankfurt + Ireland provide multi-region HA
- GDPR compliance tools built-in
- Data residency guarantees
- All three providers are comparable here

#### 5. Ecosystem & Support
- Largest partner ecosystem (terraform, datadog, etc.)
- Most community resources and documentation
- Extensive third-party integrations
- Well-trodden path for similar use cases

#### 6. Team Capabilities
- Team has moderate AWS experience
- Extensive online training resources
- Strong hiring pool for AWS skills
- Multi-cloud would require 2-3x learning curve

### Why Not Azure?

Azure is a close second (8.2/10 vs 9.0/10) and would be viable if:
- We had strong Microsoft ecosystem integration needs
- We prioritized Azure OpenAI access (GPT-4 Turbo)
- Enterprise support was critical
- IoT maturity gap was acceptable

**However:**
- IoT Hub less mature than AWS IoT Core
- 10% higher costs
- Smaller IoT ecosystem

### Why Not GCP?

GCP excels at AI/ML (Vertex AI is best-in-class) but has critical gaps:
- **No managed IoT platform** (IoT Core deprecated)
- Building custom IoT solution would take 3-4 months
- Higher risk and operational burden
- Smaller ecosystem

**GCP would be reconsidered if:**
- We outsourced IoT to a third-party platform (e.g., Particle, Losant)
- AI/ML complexity warranted best-in-class tools
- We had existing GCP expertise

### Why Not Multi-Cloud?

Multi-cloud sounds appealing but:
- 20% higher costs ($120K vs $100K)
- 2-3x operational complexity
- Cross-cloud networking challenges
- Team skill dilution

**Multi-cloud makes sense for:**
- Large enterprises with existing multi-cloud commitments
- Extreme vendor lock-in concerns
- Deep pockets for operational overhead

**Not appropriate for:**
- Greenfield projects (like ours)
- Budget-conscious startups/scale-ups
- Teams wanting to move fast

## Architecture Implications

### Regional Deployment
```
Primary Region:   eu-central-1 (Frankfurt)
Secondary Region: eu-west-1 (Ireland)
DR Strategy:      Active-Active for core services
                  Warm standby for ML services
```

### Service Mapping

| Capability | AWS Service | Alternative Considered |
|------------|-------------|------------------------|
| **Compute** | ECS (Fargate) + Lambda | EKS (Kubernetes) |
| **API Gateway** | API Gateway (HTTP API) | ALB + Service Mesh |
| **Messaging** | MSK (Kafka) | SNS/SQS |
| **Databases** | Aurora PostgreSQL, DynamoDB | RDS, DocumentDB |
| **Cache** | ElastiCache (Redis) | DynamoDB DAX |
| **Object Storage** | S3 | - |
| **Data Warehouse** | Redshift | Redshift Serverless |
| **Time-Series DB** | Timestream | Self-managed InfluxDB |
| **ML Platform** | SageMaker | Self-managed on EC2 |
| **ML Serving** | SageMaker Endpoints | Lambda + ECS |
| **Feature Store** | SageMaker Feature Store | Self-managed Feast |
| **LLM/GenAI** | Bedrock (Claude 3.5) | SageMaker JumpStart |
| **Computer Vision** | Rekognition Custom Labels | SageMaker |
| **IoT Core** | IoT Core (MQTT) | Self-managed MQTT |
| **IoT Edge** | IoT Greengrass v2 | - |
| **Streaming** | Kinesis Data Streams | MSK |
| **ETL** | Glue | Airflow on MWAA |
| **Monitoring** | CloudWatch + X-Ray | Prometheus + Grafana |
| **Tracing** | X-Ray + OTEL | Jaeger |
| **Secrets** | Secrets Manager | Parameter Store |
| **Identity** | Cognito + IAM | Auth0 + IAM |

### Cost Management Strategy

1. **Reserved Capacity:** Purchase 1-year commitments for predictable workloads (30% savings)
2. **Spot Instances:** Use for ML training and batch jobs (70% savings)
3. **Graviton:** Migrate to ARM-based instances (40% better price-performance)
4. **S3 Lifecycle:** Auto-tier data to Glacier after 90 days
5. **Redshift:** Use serverless for variable analytics workloads
6. **Budget Alerts:** Set up CloudWatch billing alarms at $80K, $90K, $100K

### Vendor Lock-in Mitigation

While we accept AWS, we'll minimize lock-in:

✅ **Use Open Standards:**
- Kubernetes API (even if using ECS/Fargate)
- PostgreSQL (Aurora is PostgreSQL-compatible)
- Kafka (MSK is managed Apache Kafka)
- OpenTelemetry for observability
- Terraform for infrastructure as code

✅ **Abstract Provider-Specific Services:**
- Wrapper libraries for AWS SDK calls
- Interface-based design for storage, cache, queues
- Database migrations via Liquibase/Flyway

✅ **Containerize Everything:**
- All services in Docker containers
- Can be moved to GKE, AKS, or on-prem Kubernetes

✅ **Multi-Region Architecture:**
- Design supports active-active across AWS regions
- Could be extended to multi-cloud if needed

❌ **Accept Lock-in Where Beneficial:**
- SageMaker (saves 6+ months of ML platform development)
- IoT Core (no good alternatives)
- Bedrock (managed LLM access)
- Rekognition (pre-trained models)

### Migration Path (if needed)

If we must migrate from AWS in the future:

**To GCP:** (6-9 months)
1. Build custom IoT platform (3 months)
2. Migrate data to BigQuery (2 months)
3. Port ML to Vertex AI (2 months)
4. Migrate compute to GKE (2 months)
5. Overlap: 1 month

**To Azure:** (4-6 months)
1. Migrate IoT to IoT Hub (2 months)
2. Migrate data to Cosmos + Synapse (2 months)
3. Port ML to Azure ML (2 months)
4. Migrate compute to AKS (1 month)
5. Overlap: 1 month

**To On-Prem/Hybrid:** (12+ months)
- Significantly more complex
- Not recommended without strong regulatory requirements

## Consequences

### Positive
✅ **Production-Ready IoT:** AWS IoT Core supports 50K+ vehicles from day one
✅ **Comprehensive ML:** SageMaker provides complete ML lifecycle without custom builds
✅ **Cost-Effective:** Spot Instances and Graviton reduce costs by 40%+
✅ **EU Compliance:** Frankfurt + Ireland meet GDPR requirements
✅ **Team Velocity:** Moderate AWS knowledge accelerates delivery
✅ **Ecosystem:** Largest pool of integrations and partners
✅ **Proven at Scale:** Thousands of companies run similar workloads on AWS

### Negative
❌ **Vendor Lock-in:** Some AWS-specific services (SageMaker, IoT Core)
❌ **Complexity:** 200+ services can be overwhelming
❌ **Cost Surprises:** Need careful cost management
❌ **IAM Complexity:** Permissions can be tricky to get right
❌ **Support Costs:** Enterprise support adds $15K+/month

### Neutral
⚪ **Skills:** Team needs AWS training (3-6 months to proficiency)
⚪ **Not Cutting-Edge:** GCP's Vertex AI is slightly more advanced
⚪ **Azure OpenAI:** Miss out on GPT-4 Turbo (but Bedrock Claude 3.5 is comparable)

## Related Decisions
- **ADR-03:** Vehicle Telemetry - Uses AWS IoT Core
- **ADR-16:** MLOps Pipeline - Uses AWS SageMaker
- **ADR-17:** Data Lakehouse - Uses AWS S3 + Redshift
- **ADR-18:** Agentic AI Framework - Uses AWS Bedrock
- **ADR-19:** Edge AI Strategy - Uses AWS IoT Greengrass

## References
- [AWS IoT Core Documentation](https://docs.aws.amazon.com/iot/)
- [AWS SageMaker Documentation](https://docs.aws.amazon.com/sagemaker/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [Gartner Magic Quadrant for Cloud Infrastructure](https://www.gartner.com/en/documents/magic-quadrant-cloud-infrastructure)

## Revision History
- **2025-11-07:** Initial decision - AWS selected
- **Next Review:** 2026-05-07 (6 months) - Reassess after Phase 3 completion
