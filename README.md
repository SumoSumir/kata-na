# MobilityCorp AI-Enabled Architecture - kata-na

> A comprehensive, production-ready architecture for transforming MobilityCorp's multi-modal transportation platform with AI-enabled intelligence

[![ADRs](https://img.shields.io/badge/ADRs-16-blue)]()

---

## 📋 Table of Contents

- [Executive Summary](#executive-summary)
- [Meet the Personas](#meet-the-personas)
- [Problem Statement](#problem-statement)
- [Solution Overview](#solution-overview)
- [Architecture Highlights](#architecture-highlights)
- [Repository Structure](#repository-structure)
- [Key Architectural Decisions](#key-architectural-decisions)
- [Technology Stack](#technology-stack)
- [Documentation](#documentation)
- [Transparency & Best Practices](#transparency--best-practices)

---

## 🎭 Meet the [Personas](PERSONAS.md)

**Understanding the humans behind the architecture**

Our solution addresses real problems for real people. Meet the stakeholders who guide our decisions:

- **Sarah Chen (CPO):** Needs to increase customer retention from 20% to 55% within 18 months
- **Marcus Weber (VP Fleet Operations):** Must reduce battery swap costs by 50%-60% and improve fleet utilization
- **David Park (CTO/CISO):** Requires 99.95% uptime with cost-effective scaling and zero-trust security compliance
- **Emma Thompson (Commuter):** Wants reliable scooter availability for her daily 8:15 AM commute
- **Alex Kumar (Tourist):** Needs conversational AI to discover vehicles and explore Barcelona
- **Lisa Müller (Family User):** Requires quality assurance—clean vehicles, reliable battery range
- **Nina Petersen (Support Agent):** Wants automated dispute resolution with photo evidence


These personas inform every architectural decision, ensuring we solve business problems, not just technical ones.

**See personas in action:**
- [Emma's predictive rebalancing journey](PERSONAS.md#use-case-1-emmas-morning-commute-predictive-rebalancing)
- [Alex's conversational AI experience](PERSONAS.md#use-case-2-alexs-tourist-experience-conversational-ai)
- [Marcus's AI-optimized morning routine](PERSONAS.md#use-case-3-marcuss-ai-optimized-morning-operational-efficiency)
- [Lisa's quality assurance workflow](PERSONAS.md#use-case-4-lisas-family-rental-quality-assurance)
- [David's security incident response](PERSONAS.md#use-case-5-davids-security-incident-response-zero-trust-in-action)

---

## 🎯 Executive Summary

MobilityCorp operates a multi-modal last-mile transportation platform across EU cities, providing electric scooters, eBikes, cars, and vans. This architecture addresses three critical business challenges:

1. **Vehicle Availability Crisis** - 15-25% of potential bookings lost due to demand-supply mismatch
2. **Battery Management Inefficiency** - Reactive maintenance and manual relocation operations create significant operational overhead
3. **Low Customer Retention** - Most users rely on ad-hoc trips rather than regular commutes

### Our Solution

An **AI-enabled microservices architecture** featuring:
- 🤖 **Predictive Demand Forecasting** - ML-powered zone-level demand prediction
- ⚡ **Dynamic Pricing & Relocation Incentives** - AI-driven fleet rebalancing
- 🔋 **Predictive Maintenance** - Proactive battery and component failure detection
- 🎨 **Computer Vision** - Automated damage detection and return verification
- 💬 **Conversational AI Assistant** - Multi-modal user support and guidance
- 📊 **Real-time Telemetry** - Edge-cloud hybrid processing for 50K+ vehicles

### Business Impact

- **Revenue:** +20-30% from improved vehicle availability
- **Costs:** -40-50% reduction in manual relocation expenses
- **Retention:** +35% increase in daily active users
- **Uptime:** 99.95% system availability with multi-region deployment

---
## 🚨 Problem Statement

### Business Context

MobilityCorp operates across multiple EU cities with a fixed fleet size per country. The company faces critical operational and customer experience challenges that threaten growth and competitive positioning.

### Core Challenges

#### 1. Demand-Supply Mismatch
- **Impact:** 15-25% of potential bookings unfulfilled
- **Root Cause:** Vehicles in wrong locations at wrong times
- **Cost:** Lost revenue + poor customer experience leading to churn

#### 2. Battery Management Crisis
- **Impact:** Manual battery swaps cost approx €10-15 each
- **Root Cause:** Reactive vs. predictive maintenance approach
- **Cost:** High operational overhead + vehicle downtime

#### 3. Low Customer Engagement
- **Impact:** 80%+ users are ad-hoc, not daily commuters
- **Root Cause:** Unreliable service reduces trust in platform
- **Cost:** Low Customer Lifetime Value (CLV)

**Detailed problem analysis:** [PROBLEM_STATEMENTS/PROBLEM_STATEMENT.md](PROBLEM_STATEMENTS/PROBLEM_STATEMENT.md)

---

## 💡 Solution Overview

### AI-Enabled Capabilities

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI-ENABLED PLATFORM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  🔮 Demand Forecasting      🎯 Dynamic Pricing                  │
│     Zone-level predictions     Real-time optimization           │
│     Multi-factor analysis      Relocation incentives            │
│                                                                  │
│  🔧 Predictive Maintenance  🤖 Conversational AI                │
│     Component failure          Multi-modal assistant            │
│     Battery health             Voice/text interface             │
│                                                                  │
│  📸 Computer Vision         📊 Real-time Analytics              │
│     Damage detection           Fleet monitoring                 │
│     Return verification        Performance metrics              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Pillars

1. **Microservices Architecture** - Independently scalable, fault-isolated services
2. **Event-Driven Communication** - Kafka-based asynchronous messaging
3. **Edge-Cloud Hybrid** - Low-latency processing where needed
4. **MLOps Pipeline** - Automated model lifecycle management
5. **Multi-Region Deployment** - Geographic redundancy and data residency

---

## 🏗️ Architecture Highlights

### System Context (C1)

High-level view of system interactions with users, vehicles, and external services.

![System Context Diagram](ARCHITECTURAL_DIAGRAMS/C1_System_Context.md)

### Container Architecture (C2)

Microservices, databases, message queues, and AI/ML components.

![Container Diagram](ARCHITECTURAL_DIAGRAMS/C2_Container.md)

### AI/ML Component Details (C3)

Deep dive into machine learning pipelines and model serving.

![AI/ML Components](ARCHITECTURAL_DIAGRAMS/C3_Component_AIML.md)

### Key Architectural Patterns

- ✅ Domain-Driven Design with bounded contexts
- ✅ CQRS for read/write optimization
- ✅ Event Sourcing for audit trails
- ✅ Circuit Breakers for fault tolerance
- ✅ API Gateway for unified access
- ✅ Service Mesh for observability

---

## 📁 Repository Structure

```
kata-na/
├── README.md                          # This file - Start here
├── PERSONAS.md                        # 🆕 Stakeholder personas (Sarah, Marcus, Emma...)
├── EVENT_STORMING.md                  # 🆕 Domain events & bounded contexts
├── PHASED_IMPLEMENTATION.md           # 🆕 Migration plan with ROI & timelines
├── GLOSSARY.md                        # Domain terminology
├── COST_ANALYSIS.md                   # Infrastructure cost estimates (percentage-based)
│
├── ADR/                               # Architecture Decision Records (16)
│   ├── ADR_01_microservices_architecture.md
│   ├── ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md
│   ├── ADR_03_Vehicle_Telemetry.md
│   ├── ADR_04_EXTERNAL_APIS.md
│   ├── ADR_05_Orchestrator.md
│   ├── ADR_06_EVENT_DRIVEN_ARCHITECTURE.md
│   ├── ADR_07_Tracing_and_Logging.md
│   ├── ADR_08_SCHEDULER_FRAMEWORK.md
│   ├── ADR_09_MULTI_REGION.md
│   ├── ADR_10_Monitoring_and_Metrics.md
│   ├── ADR_11_IoT_Enabled_Vehicles.md
│   ├── ADR_12_CONVERSATIONAL_UX_AND_AI_ASSISTANT.md
│   ├── ADR_13_DATA_COMPLIANT.md
│   ├── ADR_14_MLOps_Pipeline.md
│   ├── ADR_15_Data_Lakehouse_Strategy.md
│   └── ADR_16_NOTIFICATION_SERVICE.md
│
├── ARCHITECTURAL_DIAGRAMS/            # Visual architecture (C4 model)
│   ├── C1_System_Context.md
│   ├── C2_Container.md
│   ├── C3_Component_AIML.md
│   ├── Data_Flow_AIPipeline.md
│   ├── Deployment_Multi_Region.md
│   └── Hybrid_Telemetry_Architecture.md
│
├── HLD/                               # High-Level Design
│   ├── README.md
│   ├── scenarios/
│   │   ├── booking_workflow.md
│   │   ├── demand_forecasting.md
│   │   ├── dynamic_pricing.md
│   │   ├── predictive_maintenance.md
│   │   └── telemetry_processing.md
│   └── data_architecture/
│       ├── medallion_layers.md
│       └── feature_store.md
│
├── FUNCTIONAL_REQUIREMENTS/
│   └── FUNCTIONAL_REQUIREMENTS.md
│
├── NON_FUNCTIONAL_REQUIREMENTS/
│   └── NON_FUNCTIONAL_REQUIREMENTS.md
│
├── PROBLEM_STATEMENTS/
│   └── PROBLEM_STATEMENT.md
│
├── WORKFLOWS/
│   ├── CUSTOMER_WORKFLOWS.md
│   └── STAFF_WORKFLOWS.md
│
├── FITNESS_FUNCTIONS/
│   └── FITNESS_FUNCTIONS.md           # Architectural metrics
│
├── THREAT_MODEL/
│   └── THREAT_MODEL.md                # Security analysis
│
└── TESTING_APPROACHES/
    └── TESTING_APPROACHES.md
```

### For Everyone
1. **Understand the Problem**
Start with the [Problem Statement](PROBLEM_STATEMENTS/PROBLEM_STATEMENT.md) to understand business context.

2. **Meet the People**
Read [PERSONAS.md](PERSONAS.md) to understand who we're building for (Sarah, Marcus, Emma, etc.)

### For Business Leaders (CFO, CPO)
3. **Review Business Value**
- [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) - €30.5M NPV, 8.7-month payback
- Focus on Financial Summary and Persona Success Stories

### For Technical Leaders (CTO, Architects)
4. **Review Architecture Decisions**
Read key ADRs in order:
1. [ADR-01: Microservices Architecture](./ADR/ADR_01_microservices_architecture.md)
2. [ADR-14: MLOps Pipeline](./ADR/ADR_14_MLOps_Pipeline.md)
3. [ADR-16: Data Lakehouse](./ADR/ADR_15_Data_Lakehouse_Strategy.md)

5. **Explore Architecture**
- **High-Level:** [System Context Diagram](ARCHITECTURAL_DIAGRAMS/C1_System_Context.md)
- **Detailed:** [Container Architecture](ARCHITECTURAL_DIAGRAMS/C2_Container.md)
- **AI/ML:** [Component Details](ARCHITECTURAL_DIAGRAMS/C3_Component_AIML.md)

6. **Deep Dive into Scenarios**
- [Booking Workflow](HLD/scenarios/booking_workflow.md)
- [Demand Forecasting](HLD/scenarios/demand_forecasting.md)
- [Dynamic Pricing](HLD/scenarios/dynamic_pricing.md)

### For AI/ML Engineers
7. **Understand AI Usage**
- [ADR-02: Dynamic Pricing](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) - ML models and demand forecasting
- [ADR-14: MLOps Pipeline](./ADR/ADR_14_MLOps_Pipeline.md) - Training and deployment

### For Domain Experts
8. **Review Domain Modeling**
- [EVENT_STORMING.md](EVENT_STORMING.md) - Domain events and bounded contexts

### For Security/QA Teams
9. **Review Non-Functional Aspects**
- [Security Threats](THREAT_MODEL/THREAT_MODEL.md) - Comprehensive threat analysis
- [Fitness Functions](FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md) - Success metrics
- [Testing Strategy](TESTING_APPROACHES/TESTING_APPROACHES.md)

---

## 🔑 Key Architectural Decisions

### 1. Cloud Provider: AWS
**Why:** Best ML services (SageMaker), global infrastructure, cost-effective for EU deployment

### 2. Microservices over Monolith
**Why:** Independent scaling, fault isolation, technology flexibility
- **ADR:** [ADR-01: Microservices Architecture](./ADR/ADR_01_microservices_architecture.md)

### 3. AI-Driven Relocation Incentives
**Why:** Reduce manual relocation costs by 40-50%, improve fleet utilization
- **ADR:** [ADR-02: AI-Driven Relocation](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)

### 4. Event-Driven Architecture
**Why:** Loose coupling, scalability, real-time processing
- **ADR:** [ADR-06: Event-Driven Architecture](./ADR/ADR_06_EVENT_DRIVEN_ARCHITECTURE.md)

### 5. Edge-Cloud Hybrid Processing
**Why:** Low latency for critical operations, cost optimization

**All ADRs:** [ADR Directory](./ADR/)

---

## 🛠️ Technology Stack

### Cloud Platform
- **Provider:** AWS (Amazon Web Services)
- **Regions:** EU-Central-1 (Frankfurt), EU-West-1 (Ireland)
- **Justification:** Best ML services, GDPR compliance, cost-effective

### AI/ML Platform
- **Training:** AWS SageMaker
- **Serving:** SageMaker Endpoints + Lambda
- **Feature Store:** SageMaker Feature Store
- **MLOps:** SageMaker Pipelines
- **Models:** LightGBM, XGBoost (custom training)
- **Agentic AI:** LangChain on AWS Bedrock (Claude 3.5)

### Data Platform
- **Streaming:** Apache Kafka (MSK - Managed Streaming for Kafka)
- **Data Lake:** Amazon S3 with Medallion Architecture
  - Bronze: Raw data (Parquet)
  - Silver: Cleaned/validated (Delta Lake)
  - Gold: Business aggregations (Delta Lake)
- **Data Warehouse:** Amazon Redshift
- **Real-time DB:** DynamoDB
- **Relational DB:** Amazon Aurora PostgreSQL
- **Cache:** Amazon ElastiCache (Redis)
- **Time-Series:** Amazon Timestream

### Compute & Orchestration
- **Containers:** Amazon ECS (Fargate)
- **Functions:** AWS Lambda
- **API Gateway:** Amazon API Gateway
- **Workflow:** Apache Airflow + Apache Beam + Temporal

### IoT & Edge
- **IoT Core:** AWS IoT Core (MQTT)
- **Edge Runtime:** AWS IoT Greengrass v2
- **Edge ML:** AWS IoT Greengrass ML Inference
- **Device Management:** AWS IoT Device Management

### Observability
- **Metrics:** VictoriaMetrics + Prometheus + CloudWatch
- **Tracing:** OpenTelemetry
- **Logging:** CloudWatch Logs + OpenSearch
- **Monitoring:** Grafana
- **Alerting:** PagerDuty

### Security
- **Identity:** AWS IAM (zero-trust) + AWS SSO (MFA)
- **Secrets:** AWS Secrets Manager (auto-rotation)
- **Encryption:** AWS KMS (AES-256 at rest), TLS 1.3 (in transit)
- **Network:** VPC, Security Groups, AWS WAF, Shield
- **Threat Detection:** AWS GuardDuty, Security Hub, Macie
- **IoT Security:** X.509 certificates, mTLS, IoT Device Defender
- **Compliance:** AWS Config, CloudTrail (GDPR, PCI-DSS, ISO 27001)

### Frontend
- **Mobile:** React Native (iOS/Android)
- **Web:** Next.js + React
- **Maps:** Mapbox GL JS

### Development & CI/CD
- **Source Control:** GitHub
- **CI/CD:** GitHub Actions
- **Infrastructure:** Terraform
- **Configuration:** AWS Systems Manager Parameter Store

---

## 📚 Documentation

### Architecture Documentation
- **ADRs (16):** [ADR Directory](./ADR/) - All architectural decisions with rationale
- **Diagrams:** [ARCHITECTURAL_DIAGRAMS](ARCHITECTURAL_DIAGRAMS/) - C4 model diagrams
- **HLD:** [HLD Directory](HLD/) - Detailed scenario walkthroughs
- **Glossary:** [GLOSSARY.md](GLOSSARY.md) - Domain terminology

### Requirements
- **Functional:** [FUNCTIONAL_REQUIREMENTS.md](FUNCTIONAL_REQUIREMENTS/FUNCTIONAL_REQUIREMENTS.md)
- **Non-Functional:** [NON_FUNCTIONAL_REQUIREMENTS.md](NON_FUNCTIONAL_REQUIREMENTS/NON_FUNCTIONAL_REQUIREMENTS.md)
- **Problem:** [PROBLEM_STATEMENT.md](PROBLEM_STATEMENTS/PROBLEM_STATEMENT.md)

### Quality & Operations
- **Security:** [THREAT_MODEL.md](THREAT_MODEL/THREAT_MODEL.md) - Comprehensive threat analysis
- **Testing:** [TESTING_APPROACHES.md](TESTING_APPROACHES/TESTING_APPROACHES.md)
- **Metrics:** [FITNESS_FUNCTIONS.md](FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md)
- **Deployment:** [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) - Migration & feature rollout strategy
- **Cost:** [COST_ANALYSIS.md](COST_ANALYSIS.md) - TCO estimates

### Workflows
- **Customer:** [CUSTOMER_WORKFLOWS.md](WORKFLOWS/CUSTOMER_WORKFLOWS.md)
- **Staff:** [STAFF_WORKFLOWS.md](WORKFLOWS/STAFF_WORKFLOWS.md)

---

## 🎯 Getting Started

### For Architects
1. Read [Problem Statement](PROBLEM_STATEMENTS/PROBLEM_STATEMENT.md)
2. Review [System Context](ARCHITECTURAL_DIAGRAMS/C1_System_Context.md)
3. Study key ADRs (01, 14, 15, 16)
4. Explore [HLD Scenarios](HLD/scenarios/)

### For Developers
1. Understand [Container Architecture](ARCHITECTURAL_DIAGRAMS/C2_Container.md)
2. Review [Technology Stack](#technology-stack)
3. Read service-specific ADRs
4. Check [Workflows](WORKFLOWS/)

### For Product Managers
1. Review [Business Impact](#business-impact)
2. Study [Functional Requirements](FUNCTIONAL_REQUIREMENTS/FUNCTIONAL_REQUIREMENTS.md)
3. Understand [Customer Workflows](WORKFLOWS/CUSTOMER_WORKFLOWS.md)
4. Review [Cost Analysis](COST_ANALYSIS.md)

### For Security Engineers
1. Study [Threat Model](THREAT_MODEL/THREAT_MODEL.md)
2. Review security ADRs
3. Check compliance requirements
4. Audit [Security Stack](#security)

### For DevOps/SRE
1. Review [Deployment Architecture](ARCHITECTURAL_DIAGRAMS/Deployment_Multi_Region.md)
2. Study [Phased Implementation](PHASED_IMPLEMENTATION.md)
3. Check [Observability](#observability) stack
4. Review [Fitness Functions](FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md)

---

## 🏆 What Makes This Architecture Unique

### ✅ Comprehensive Security
- **Only solution with complete threat model**
- STRIDE analysis across all components
- Asset-based threat identification
- Mitigation strategies documented

### ✅ Quantifiable Success Metrics
- **Clear fitness functions with targets**
- Business and technical KPIs
- Architectural drift prevention
- Measurable outcomes

### ✅ Production-Ready Specificity
- **Named technologies, not generic references**
- Version numbers and deployment patterns
- Complete MLOps pipeline
- Cost estimates and TCO

### ✅ Edge-Cloud Hybrid Intelligence
- **Low-latency critical operations on edge**
- Cost-optimized cloud processing
- Seamless edge-cloud orchestration
- OTA model updates

### ✅ Business-Aligned Architecture
- **Clear problem-to-solution traceability**
- ROI calculations
- Risk mitigation strategies
- Phased implementation plan

---

## 📈 Success Metrics (Fitness Functions)

### System Performance
- **API Uptime:** > 99.95%
- **P95 API Latency:** < 200ms
- **Vehicle Unlock Time:** < 3 seconds
- **Concurrent Users:** 100,000+
- **Vehicles Supported:** 50,000+

### Operational Efficiency
- **Predictive Maintenance Lead Time:** > 7 days
- **Rebalancing Completion Rate:** > 90%
- **MTTR:** < 24 hours
- **Vehicle Utilization:** +15% YoY

### AI Model Performance
- **Demand Forecast MAPE:** < 15%
- **Damage Detection Accuracy:** > 95%
- **Price Optimization Uplift:** +10-15%

**Full metrics:** [FITNESS_FUNCTIONS.md](FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md)

---

## 🔒 Security & Compliance

- ✅ **Zero-Trust:** All access authenticated + authorized (no implicit trust)
- ✅ **Encryption Everywhere:** AES-256 at rest, TLS 1.3 in transit, mTLS for IoT
- ✅ **GDPR Compliant:** Data residency (EU-only), right-to-erasure, consent management
- ✅ **PCI-DSS Level 1:** Payment tokenization via Stripe, dedicated VPC isolation
- ✅ **ISO 27001:** Security controls, risk assessments, incident response playbooks
- ✅ **IoT Security:** X.509 certificates for 50K vehicles, Device Defender monitoring
- ✅ **Automated Compliance:** AWS Config, Macie, CloudTrail enforce policies
- ✅ **Threat Detection:** GuardDuty (<24hr MTTD), Security Hub (unified dashboard)

**Security Cost:** $24,500/month (6% of total spend, industry standard 5-8%)  
**Security ROI:** €295K/year positive ROI (breach prevention - security cost)

**Details:** [THREAT_MODEL.md](THREAT_MODEL/THREAT_MODEL.md)


---

## 🗓️ Implementation Timeline

**Phased approach with continuous value delivery:**

| Phase | Duration | Investment | Annual Savings | Key Outcome |
|-------|----------|------------|----------------|-------------|
| **Phase 1:** Foundation | 4 months | €206K | €460K | Observability & data foundation |
| **Phase 2:** AI/ML | 5 months | €420K | €10.3M | Demand forecasting, dynamic pricing |
| **Phase 3:** Microservices | 5 months | €725K | €1.875M | Independent scaling, zero downtime |
| **Phase 4:** Conversational AI | 2 months | €126K | €776K | NPS +20 points, support automation |
| **Phase 5:** Multi-Region | 2 months | €216K | Expansion enabler | 99.95% uptime, rapid city launch |

**Payback Period:** 8.7 months | **3-Year NPV:** €12.4M | **IRR:** 187%

**Detailed migration & rollout plan:** [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)

---

## 🔍 Transparency & Best Practices

### AI Usage Declaration
**Where AI/ML Was Used:**
- ✅ **Traditional ML:** Demand forecasting, dynamic pricing, predictive maintenance (deterministic, explainable)
- ✅ **Gen AI (LLMs):** Conversational UI only (Claude 3.5 Sonnet for natural dialogue)
- ❌ **Gen AI NOT Used:** Pricing, payments, vehicle control (safety/financial critical)

**Why Not Gen AI Everywhere?**
> "Considering the cost & non-deterministic nature, we would NOT use Gen AI for financially or safety-critical operations. Gen AI's hallucination risk makes it unsuitable for pricing decisions that directly impact revenue and customer trust."

**Cost Comparison:**
- Gen AI for pricing: €80K/month
- Traditional ML: €3K/month
- **Savings: €924K/year by choosing the right tool**


---

### Domain-Driven Design
**Event Storming workshops guided our architecture:**
- 3 sessions with Sarah (CPO), Marcus (VP Ops), David (CTO), and product/engineering teams
- Identified 15+ domain events, 8 bounded contexts, and key business policies
- Informed microservices boundaries and event-driven patterns

**Full event storming results:** [EVENT_STORMING.md](EVENT_STORMING.md)

---

### Build vs Buy Decisions

**Strategic "Buy" Decisions (Opportunity Costs):**
| Function | Build Cost | Buy Cost | Decision | Rationale |
|----------|------------|----------|----------|-----------|
| **Payment Processing** | €200K + PCI-DSS compliance | €1K/month (Stripe 2.9%) | ✅ Buy | Risk transfer worth the fee |
| **Kafka Management** | 2 FTE (€16K/month) | €6K/month (MSK) | ✅ Buy | Focus eng team on differentiation |
| **LLM Hosting** | €50K setup + 1 FTE | €1K/month (Claude API) | ✅ Buy | Reliability > cost savings |

**Full analysis:** [PHASED_IMPLEMENTATION.md - Opportunity Costs](PHASED_IMPLEMENTATION.md#-opportunity-costs)

---

### Alternative Solutions Considered

Every ADR includes:
- ✅ **Alternatives evaluated** (e.g., monolith vs microservices vs service-based)
- ✅ **Trade-offs documented** (pros/cons with business context)
- ✅ **Rejection rationale** (why alternatives don't meet requirements)

**Example:** [ADR-01 (Microservices)](./ADR/ADR_01_microservices_architecture.md) compares against monolith and service-based architecture with specific cost/scaling implications.

---

## 💰 Cost Estimate

### Monthly Infrastructure Costs (Production)
- **Compute:** ~$45,000 (ECS + Lambda + SageMaker)
- **Data Storage:** ~$12,000 (S3 + RDS + DynamoDB)
- **AI/ML:** ~$28,000 (SageMaker training + inference)
- **Networking:** ~$8,000 (Data transfer + API Gateway)
- **Other:** ~$7,000 (Monitoring, security, etc.)

**Total:** ~$100,000/month for 50K vehicles across EU

**Detailed breakdown:** [COST_ANALYSIS.md](COST_ANALYSIS.md)

---

## 🤝 Contributing

This architecture is designed for O'Reilly Architectural Katas Q4 2025. For questions or discussions:

- **Issues:** [GitHub Issues](https://github.com/SumoSumir/kata-na/issues)
- **Discussions:** [GitHub Discussions](https://github.com/SumoSumir/kata-na/discussions)
- **Email:** [kata-na@googlegroups.com](mailto:kata-na@googlegroups.com)

---

## 📄 License

This architectural documentation is provided under MIT License for educational purposes.

---

## 📚 Key Documentation

**Start Here:**
1. **[PERSONAS.md](PERSONAS.md)** - Meet Sarah, Marcus, David, Emma, and the people driving our decisions
2. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain events, bounded contexts, and business rules
3. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Migration plan with timelines, costs, and ROI
4. **[ADR Directory](./ADR/)** - All architectural decisions with alternatives and trade-offs

**Architecture is:** *"One foot in business, one foot in technology"* - O'Reilly Architectural Katas Judges

---

## 🙏 Acknowledgments

- **O'Reilly Media** - For hosting the Architectural Katas
- **Competing Teams** - For excellent solutions that raised the bar
- **Reviewers** - For feedback and insights

---