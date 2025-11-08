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
- **Scalability:** Support high concurrent users and large vehicle fleet
- **Availability:** High uptime for core services
- **Latency:** Low API latency, fast vehicle unlock response
- **AI/ML:** Production-ready ML platform with AutoML, feature store, model serving
- **IoT:** Secure device connectivity, OTA updates, edge computing
- **Data:** Real-time streaming, data lake, data warehouse, caching

### Business Constraints
- **Budget:** Moderate infrastructure costs
- **Timeline:** Phased implementation approach
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
- Spot Instances: Significant savings for batch ML training
- Savings Plans: Committed use discounts
- S3 Intelligent-Tiering: Automatic storage class optimization
- Graviton2/3: ARM processors with better price-performance

✅ **Maturity & Ecosystem:**
- Largest cloud provider
- Most comprehensive service catalog
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
- Enterprise support expensive

#### Cost Estimate (Monthly, Production)
```
Compute, ML, Data Storage, Streaming, Networking, IoT, 
Monitoring, Security services - within budget constraints
```

#### Scoring
- AI/ML Capabilities: Excellent
- IoT Platform: Excellent
- EU Compliance: Strong
- Cost: Good
- Developer Experience: Good
- Support: Good
- **Overall: Best fit**

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
- Smaller market share than AWS
- Fewer third-party integrations than AWS

#### Cost Estimate (Monthly, Production)
```
Compute, ML, Data, Streaming, Networking, IoT, Monitoring, 
Security services - slightly higher than AWS
```

#### Scoring
- AI/ML Capabilities: Excellent
- IoT Platform: Good
- EU Compliance: Strong
- Cost: Moderate
- Developer Experience: Good
- Support: Excellent
- **Overall: Strong alternative**

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
- Smaller market share than AWS/Azure
- Smaller partner ecosystem
- Fewer pre-built integrations

❌ **IoT Gap - Critical Flaw:**
- **DEALBREAKER:** No managed IoT device management
- Would require building custom MQTT broker, device auth, OTA updates
- Significant engineering effort and operational overhead

#### Cost Estimate (Monthly, Production)
```
Compute, ML, Data, Streaming, Networking services - comparable to AWS
Custom IoT Solution - significant additional cost and effort
```

#### Scoring
- AI/ML Capabilities: Excellent
- IoT Platform: Poor ⚠️ **Major Gap**
- EU Compliance: Strong
- Cost: Good
- Developer Experience: Excellent
- Support: Good
- **Overall: Limited by IoT gap**

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
- Data egress charges between clouds
- Duplicate tooling and monitoring
- Higher staff training costs
- No bulk discounts

❌ **Reliability:**
- Cross-cloud dependencies create fragility
- Network latency between providers
- Harder to troubleshoot issues

❌ **Skill Requirements:**
- Team must learn multiple cloud platforms
- Harder to hire specialists
- More documentation and runbooks

#### Cost Estimate (Monthly, Production)
```
Base services across clouds - higher than single provider
Cross-cloud data transfer, duplicate tooling, operational overhead
```

#### Scoring
- AI/ML Capabilities: Excellent
- IoT Platform: Excellent
- EU Compliance: Good
- Cost: Poor ⚠️
- Developer Experience: Poor ⚠️
- Support: Moderate
- **Overall: Too complex for our needs**

---

## Decision Matrix

| Criteria | AWS | Azure | GCP | Multi-Cloud |
|----------|-----|-------|-----|-------------|
| **AI/ML Capabilities** | Excellent | Excellent | Excellent | Excellent |
| **IoT Platform** | Excellent | Good | Poor ⚠️ | Excellent |
| **EU Compliance** | Strong | Strong | Strong | Good |
| **Cost** | Good | Moderate | Good | Poor ⚠️ |
| **Developer Experience** | Good | Good | Excellent | Poor ⚠️ |
| **Service Maturity** | Excellent | Good | Good | Good |
| **Support & Ecosystem** | Good | Excellent | Good | Moderate |
| **Overall** | **Best fit** | **Strong** | **Limited** | **Complex** |

## Decision

**We choose Amazon Web Services (AWS) as our cloud provider.**

### Primary Rationale

#### 1. IoT Platform Maturity (Critical)
- **AWS IoT Core** is production-ready with large vehicle fleet support
- **IoT Greengrass v2** enables edge ML inference for collision detection
- **IoT FleetWise** purpose-built for automotive telemetry
- GCP's lack of managed IoT is a **dealbreaker** - would require significant custom development

#### 2. AI/ML Completeness
- **SageMaker** provides end-to-end ML lifecycle (comparable to Vertex AI)
- **Bedrock** offers Claude for agentic AI (competitive with Azure OpenAI)
- **Rekognition** for damage detection (pre-trained models save time)
- **Forecast** for demand prediction (specialized time-series service)

#### 3. Cost-Effective at Scale
- Spot Instances save significantly on ML training
- S3 Intelligent-Tiering automatically optimizes storage costs
- Graviton processors provide better price-performance
- Within budget constraints

#### 4. EU Compliance & Presence
- Frankfurt + Ireland provide multi-region HA
- GDPR compliance tools built-in
- Data residency guarantees
- All three providers are comparable here

#### 5. Ecosystem & Support
- Largest partner ecosystem
- Most community resources and documentation
- Extensive third-party integrations
- Well-trodden path for similar use cases

#### 6. Team Capabilities
- Team has moderate AWS experience
- Extensive online training resources
- Strong hiring pool for AWS skills
- Multi-cloud would require significant learning curve

### Why Not Azure?

Azure is a strong alternative and would be viable if:
- We had strong Microsoft ecosystem integration needs
- We prioritized Azure OpenAI access
- Enterprise support was critical
- IoT maturity gap was acceptable

**However:**
- IoT Hub less mature than AWS IoT Core
- Higher costs
- Smaller IoT ecosystem

### Why Not GCP?

GCP excels at AI/ML (Vertex AI is best-in-class) but has critical gaps:
- **No managed IoT platform** (IoT Core deprecated)
- Building custom IoT solution would take significant time
- Higher risk and operational burden
- Smaller ecosystem

**GCP would be reconsidered if:**
- We outsourced IoT to a third-party platform
- AI/ML complexity warranted best-in-class tools
- We had existing GCP expertise

### Why Not Multi-Cloud?

Multi-cloud sounds appealing but:
- Higher costs
- Significantly increased operational complexity
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

1. **Reserved Capacity:** Purchase commitments for predictable workloads
2. **Spot Instances:** Use for ML training and batch jobs
3. **Graviton:** Migrate to ARM-based instances for better efficiency
4. **S3 Lifecycle:** Auto-tier data to lower-cost storage classes
5. **Redshift:** Use serverless for variable analytics workloads
6. **Budget Alerts:** Set up CloudWatch billing alarms at multiple thresholds

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

**To GCP:**
1. Build custom IoT platform
2. Migrate data to BigQuery
3. Port ML to Vertex AI
4. Migrate compute to GKE

**To Azure:**
1. Migrate IoT to IoT Hub
2. Migrate data to Cosmos + Synapse
3. Port ML to Azure ML
4. Migrate compute to AKS

**To On-Prem/Hybrid:**
- Significantly more complex
- Not recommended without strong regulatory requirements

## Consequences

### Positive
✅ **Production-Ready IoT:** AWS IoT Core supports large vehicle fleet from day one
✅ **Comprehensive ML:** SageMaker provides complete ML lifecycle without custom builds
✅ **Cost-Effective:** Spot Instances and Graviton reduce costs significantly
✅ **EU Compliance:** Frankfurt + Ireland meet GDPR requirements
✅ **Team Velocity:** Moderate AWS knowledge accelerates delivery
✅ **Ecosystem:** Largest pool of integrations and partners
✅ **Proven at Scale:** Thousands of companies run similar workloads on AWS

### Negative
❌ **Vendor Lock-in:** Some AWS-specific services
❌ **Complexity:** Large service catalog can be overwhelming
❌ **Cost Surprises:** Need careful cost management
❌ **IAM Complexity:** Permissions can be tricky to get right
❌ **Support Costs:** Enterprise support adds significant expense

### Neutral
⚪ **Skills:** Team needs AWS training to reach proficiency
⚪ **Not Cutting-Edge:** GCP's Vertex AI is slightly more advanced
⚪ **Azure OpenAI:** Miss out on GPT-4 Turbo (but Bedrock Claude is comparable)

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

