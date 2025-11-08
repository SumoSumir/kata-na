# Quick Reference Guide - For Different Readers

This guide helps you navigate kata-na documentation based on your role and interests.

---

## 👔 For Executives (CFO, CXO)

**Time commitment:** 15 minutes

### Start Here
1. **[README.md](README.md)** - Executive Summary (scroll to Business Impact)
2. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Focus on:
   - Financial Summary (page 15)
   - NPV Calculation (€30.5M over 3 years)
   - Payback Period (8.7 months)

### Key Metrics
- **Annual Savings:** €11.176M
- **Total Investment:** €1.693M
- **IRR:** 187%
- **Revenue Increase:** +20-30%
- **Cost Reduction:** -40-50%

### Questions Answered
- ❓ "What's the ROI?" → 8.7-month payback, 187% IRR
- ❓ "How much risk?" → Phased approach, rollback at each stage
- ❓ "When do we see results?" → Phase 2 (Month 9) delivers €10.3M annually

---

## 🎯 For Product Leaders (CPO, VP Product)

**Time commitment:** 30 minutes

### Start Here
1. **[PERSONAS.md](PERSONAS.md)** - Meet your customers and stakeholders
2. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain events and business rules
3. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Persona success stories

### Key Insights
- **Emma (Commuter):** Retention 22% → 58%
- **Alex (Tourist):** NPS increased from 45 → 67
- **Lisa (Family User):** Trust in vehicle quality restored

### Business Metrics
- **Customer Retention:** 20% → 55% (Target: 55%)
- **NPS:** 45 → 67 (Target: 60)
- **DAU Growth:** 120K → 168K (+40%)
- **Revenue per Vehicle:** €200 → €238/month

### Questions Answered
- ❓ "How do we improve retention?" → AI demand forecasting + relocation incentives
- ❓ "What about customer satisfaction?" → Conversational AI increases NPS by 20 points
- ❓ "Can we predict demand?" → 85% forecast accuracy enables proactive positioning

---

## ⚙️ For Operations Leaders (VP Ops, COO)

**Time commitment:** 45 minutes

### Start Here
1. **[PERSONAS.md](PERSONAS.md)** - Marcus (VP Ops) and Javier (Field Ops Manager)
2. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Operational efficiency gains
3. **[ADR-02](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - Dynamic pricing and relocation

### Key Operational Wins
- **Ops Cost per Vehicle:** €40 → €18/month (-55%)
- **Battery Swap Efficiency:** AI prioritization reduces wasted trips by 43%
- **Maintenance:** Predictive approach reduces unplanned downtime by 50%
- **Fleet Utilization:** 45% → 75%

### Automation Delivered
- ✅ AI-powered task prioritization for field teams
- ✅ Predictive battery swap scheduling
- ✅ Automated damage detection (95% accuracy)
- ✅ Dynamic relocation incentives (40% user acceptance)

### Questions Answered
- ❓ "How do we reduce manual ops?" → AI prioritizes tasks, optimizes routes
- ❓ "Can we predict failures?" → 70% of failures predicted 7 days in advance
- ❓ "What about battery swaps?" → Demand-based prioritization, not just low battery

---

## 💻 For Technical Leaders (CTO, VP Engineering)

**Time commitment:** 2 hours (deep dive)

### Start Here
1. **[README.md](README.md)** - Architecture highlights and tech stack
2. **[ADR-01](ADR/ADR_01_microservices_architecture.md)** - Microservices foundation
3. **[ADR-15](ADR/ADR_15_Cloud_Provider_Selection.md)** - Cloud provider selection
4. **[ARCHITECTURAL_DIAGRAMS/C2_Container.md](ARCHITECTURAL_DIAGRAMS/C2_Container.md)** - Container architecture

### Key Technical Decisions
- **Architecture:** Microservices (ADR-01)
- **Cloud:** AWS (ADR-15)
- **Communication:** Event-driven (ADR-06)
- **Data:** Lakehouse with Medallion layers (ADR-17)
- **AI/ML:** Traditional ML + LLMs (ADR-16, ADR-18)

### Non-Functional Requirements
- **Uptime:** 99.95% (multi-region)
- **Latency:** P95 <200ms for APIs
- **Scalability:** 50K vehicles → 500K (linear scaling)
- **MTTR:** 4 hours → 28 minutes

### Deep Dives
- **AI/ML:** [ADR-16 (MLOps)](ADR/ADR_16_MLOps_Pipeline.md)
- **Telemetry:** [ADR-03 (Vehicle Telemetry)](ADR/ADR_03_Vehicle_Telemetry.md)
- **Security:** [THREAT_MODEL.md](THREAT_MODEL/THREAT_MODEL.md)
- **Testing:** [TESTING_APPROACHES.md](TESTING_APPROACHES/TESTING_APPROACHES.md)

### Questions Answered
- ❓ "Why microservices?" → Independent scaling, fault isolation
- ❓ "Why AWS?" → Best ML services, EU compliance, cost-effective
- ❓ "How do we handle failures?" → Circuit breakers, retries, multi-region
- ❓ "What about observability?" → OpenTelemetry + Grafana + X-Ray

---

## 🤖 For AI/ML Engineers

**Time commitment:** 90 minutes

### Start Here
1. **[ADR-16](ADR/ADR_16_MLOps_Pipeline.md)** - MLOps pipeline architecture
2. **[ADR-02](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - Demand forecasting & pricing
3. **[ADR-18](ADR/ADR_18_Agentic_AI_Framework.md)** - Conversational AI with LLMs
4. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain events for ML features

### ML Models Deployed
1. **Demand Forecasting:** LSTM + Prophet + XGBoost ensemble
2. **Dynamic Pricing:** Reinforcement Learning (DQN) + Gradient Boosting
3. **Predictive Maintenance:** Isolation Forest + Survival analysis
4. **Damage Detection:** ResNet-50 fine-tuned CNN
5. **Conversational AI:** Claude 3.5 Sonnet (LLM)

### Why NOT Gen AI for Everything
- ❌ **Pricing:** Non-deterministic, expensive, slow
- ❌ **Forecasting:** Needs explainability, auditability
- ❌ **Maintenance:** Safety-critical, requires determinism
- ✅ **Conversational UI:** Natural dialogue, acceptable variability

### Model Performance
- **Demand Forecast:** MAPE <15%, 85% accuracy
- **Damage Detection:** 98% precision, 200ms inference
- **Pricing Model:** +15% revenue vs baseline
- **Maintenance Prediction:** 70% failures detected 7 days ahead

### Questions Answered
- ❓ "Which models do we use?" → Traditional ML for deterministic tasks, LLM for conversation
- ❓ "Why not Gen AI for pricing?" → Cost (€80K vs €3K) + hallucination risk
- ❓ "How do we retrain?" → Weekly automated retraining via SageMaker Pipelines
- ❓ "What about model drift?" → Continuous monitoring + human-in-the-loop

---

## 🏗️ For Solution Architects

**Time commitment:** 3 hours (comprehensive review)

### Start Here
1. **[README.md](README.md)** - Overview and quick start
2. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain modeling
3. **All ADRs** - Read in dependency order:
   - ADR-01 → ADR-15 → ADR-06 → ADR-03 → ADR-02 → ADR-16 → ADR-17
4. **[ARCHITECTURAL_DIAGRAMS/](ARCHITECTURAL_DIAGRAMS/)** - Visual architecture

### Architecture Patterns Used
- ✅ Microservices with bounded contexts
- ✅ Event-driven architecture (Kafka)
- ✅ CQRS for read/write optimization
- ✅ Event sourcing (Booking domain only)
- ✅ API Gateway pattern
- ✅ Circuit Breaker + Retry
- ✅ Strangler Fig (migration pattern)
- ✅ Edge-Cloud Hybrid

### Integration Points
- **External APIs:** Weather, Events, Maps, Payment (ADR-04)
- **IoT:** MQTT via AWS IoT Core (ADR-11)
- **Data:** Lakehouse with Medallion layers (ADR-17)
- **AI/ML:** SageMaker + Lambda inference (ADR-16)

### Trade-offs Documented
- Microservices vs monolith (ADR-01)
- AWS vs GCP vs Azure (ADR-15)
- Gen AI vs traditional ML (ADR-02, ADR-16, ADR-18)
- Build vs buy (PHASED_IMPLEMENTATION.md)

### Questions Answered
- ❓ "How do services communicate?" → Event-driven via Kafka + sync APIs
- ❓ "How do we handle consistency?" → Eventual consistency + idempotency
- ❓ "What about security?" → Zero Trust, mTLS, IAM roles
- ❓ "How do we scale?" → Horizontal scaling per service + auto-scaling

---

## 🔐 For Security Engineers

**Time commitment:** 1 hour

### Start Here
1. **[THREAT_MODEL.md](THREAT_MODEL/THREAT_MODEL.md)** - Comprehensive threat analysis
2. **[ADR-14](ADR/ADR_14_DATA_COMPLIANT.md)** - Data compliance (GDPR, PCI-DSS)
3. **[ADR-11](ADR/ADR_11_IoT_Enabled_Vehicles.md)** - IoT security

### Security Highlights
- ✅ **Zero Trust Architecture:** No implicit trust
- ✅ **mTLS:** Service-to-service encryption
- ✅ **Secrets Management:** AWS Secrets Manager
- ✅ **PCI-DSS:** Stripe for payment processing
- ✅ **GDPR:** Data residency, right to erasure
- ✅ **IoT Security:** Certificate-based auth, encrypted MQTT

### Threat Scenarios Addressed
- ⚠️ Unauthorized vehicle access
- ⚠️ Payment fraud
- ⚠️ Data breaches (PII)
- ⚠️ DDoS attacks
- ⚠️ IoT device compromise

### Questions Answered
- ❓ "How do we secure payments?" → Stripe (PCI-DSS compliant), no card storage
- ❓ "What about GDPR?" → Data residency, encryption at rest/transit, deletion APIs
- ❓ "How do we protect IoT?" → Certificate rotation, anomaly detection, kill switch

---

## 🧪 For QA Engineers

**Time commitment:** 45 minutes

### Start Here
1. **[TESTING_APPROACHES.md](TESTING_APPROACHES/TESTING_APPROACHES.md)** - Testing strategy
2. **[FITNESS_FUNCTIONS.md](FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md)** - Architectural metrics
3. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Testing per phase

### Testing Layers
- **Unit Tests:** 80%+ coverage per service
- **Integration Tests:** API contracts, database interactions
- **E2E Tests:** Critical user journeys (Emma's booking flow)
- **Load Tests:** 10x peak load (100K concurrent users)
- **Chaos Engineering:** Monthly disaster recovery drills

### Success Criteria
- ✅ P95 latency <200ms
- ✅ 99.95% uptime
- ✅ Zero data loss
- ✅ <30 min MTTR

### Questions Answered
- ❓ "How do we test microservices?" → Contract testing + service virtualization
- ❓ "What about ML models?" → Shadow mode + A/B testing + human validation
- ❓ "How do we test at scale?" → Load testing + chaos engineering

---

## 📊 For Data Engineers

**Time commitment:** 1 hour

### Start Here
1. **[ADR-17](ADR/ADR_17_Data_Lakehouse_Strategy.md)** - Data Lakehouse
2. **[ADR-16](ADR/ADR_16_MLOps_Pipeline.md)** - MLOps pipeline
3. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain events

### Data Architecture
- **Bronze Layer:** Raw data (Parquet on S3)
- **Silver Layer:** Cleaned/validated (Delta Lake)
- **Gold Layer:** Business aggregations (Delta Lake)
- **Feature Store:** SageMaker Feature Store

### Data Volumes
- **Telemetry:** 4.32B events/month
- **Bookings:** 1M/month
- **Analytics:** 500GB/day processed

### Questions Answered
- ❓ "How do we ingest data?" → Kafka → Lambda → S3 (Bronze)
- ❓ "How do we transform data?" → Spark on EMR (Silver → Gold)
- ❓ "How do we serve features?" → SageMaker Feature Store (low latency)

---

## 🎓 For Students & Learners

**Time commitment:** Variable (2-5 hours)

### Recommended Path
1. **[PERSONAS.md](PERSONAS.md)** - Understand the problem space (15 min)
2. **[README.md](README.md)** - Overview (15 min)
3. **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain modeling (30 min)
4. **[ADR-01](ADR/ADR_01_microservices_architecture.md)** - Core architectural decision (20 min)
5. **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - Real-world migration (45 min)
6. **[ADR-02](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - AI/ML decision-making (30 min)

### Key Learnings
- ✅ **Event Storming:** How to model domains collaboratively
- ✅ **ADR Best Practices:** How to document decisions
- ✅ **Business-Technology Balance:** One foot in each
- ✅ **AI Strategy:** When to use traditional ML vs Gen AI
- ✅ **Migration Strategy:** Phased approach vs big bang

### Practice Exercises
1. Create personas for your own project
2. Run an event storming session
3. Write an ADR with alternatives
4. Calculate NPV for an architecture change

---

## 📚 Document Map

### Core Documents
| Document | Purpose | Time | Audience |
|----------|---------|------|----------|
| **README.md** | Overview & quick start | 15 min | Everyone |
| **PERSONAS.md** | Stakeholder personas | 15 min | Product, Business |
| **EVENT_STORMING.md** | Domain modeling | 30 min | Architects, Product |
| **PHASED_IMPLEMENTATION.md** | Migration plan with ROI | 1 hour | Executives, Ops |
| **COST_ANALYSIS.md** | Infrastructure costs | 30 min | CFO, Engineering |

### Technical Deep Dives
| Document | Purpose | Time | Audience |
|----------|---------|------|----------|
| **ADR-01** | Microservices architecture | 20 min | Engineers, Architects |
| **ADR-02** | Dynamic pricing & ML | 30 min | ML Engineers, Product |
| **ADR-16** | MLOps pipeline | 30 min | ML Engineers, Data |
| **ADR-17** | Data Lakehouse | 30 min | Data Engineers |
| **THREAT_MODEL.md** | Security analysis | 1 hour | Security Engineers |

### Supporting Documents
| Document | Purpose | Time | Audience |
|----------|---------|------|----------|
| **COST_ANALYSIS.md** | Infrastructure costs | 30 min | CFO, Engineering |
| **TESTING_APPROACHES.md** | Testing strategy | 30 min | QA, Engineering |
| **ROLLOUT_STRATEGY.md** | Deployment phases | 30 min | Ops, Product |
| **GLOSSARY.md** | Domain terminology | 15 min | Everyone |

---

## 🔗 Quick Links by Topic

### Business Value
- [Phased Implementation - Financial Summary](PHASED_IMPLEMENTATION.md#-financial-summary)
- [Business Impact Metrics](PHASED_IMPLEMENTATION.md#-business-metrics-tracking)
- [Persona Success Stories](PHASED_IMPLEMENTATION.md#-success-stories-persona-based)

### Architecture Decisions
- [All ADRs](ADR/)
- [ADR Decision Graph](ADR/ADR_01_microservices_architecture.md#integration-with-other-architectural-decisions)
- [Alternative Solutions](ADR/ADR_01_microservices_architecture.md#alternatives-considered)

### AI/ML Details
- [ML Strategy Overview](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)
- [Demand Forecasting Models](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md#1-demand-forecasting-engine)
- [MLOps Pipeline](ADR/ADR_16_MLOps_Pipeline.md)
- [Conversational AI](ADR/ADR_18_Agentic_AI_Framework.md)

### Domain Modeling
- [Event Storming](EVENT_STORMING.md)
- [Bounded Contexts](EVENT_STORMING.md#-aggregates--bounded-contexts)
- [Business Policies](EVENT_STORMING.md#-policies-business-rules)
### Implementation
- [Phased Migration](PHASED_IMPLEMENTATION.md)
- [Rollout Strategy](ROLLOUT_STRATEGY/ROLLOUT_STRATEGY.md)
- [Risk Mitigation](PHASED_IMPLEMENTATION.md#-risk-mitigation--rollback-plans)

---

## 💡 Tips for Reviewers

### If You Have 10 Minutes
1. Read [README.md](README.md) - Executive Summary
2. Skim [PERSONAS.md](PERSONAS.md) - Meet the stakeholders
3. Review [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) - Financial Summary

### If You Have 30 Minutes
Add to above:
4. Read [ADR-01](ADR/ADR_01_microservices_architecture.md) - Core architecture decision
5. Read [ADR-02](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) - AI/ML strategy
6. Skim [EVENT_STORMING.md](EVENT_STORMING.md) - Domain modeling

### If You Have 1 Hour
Add to above:
7. Read [ADR-16](ADR/ADR_16_MLOps_Pipeline.md) - MLOps pipeline
8. Review [ARCHITECTURAL_DIAGRAMS/C2_Container.md](ARCHITECTURAL_DIAGRAMS/C2_Container.md)
9. Read [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) - Full migration plan

### If You Have 2+ Hours (Deep Dive)
- Read all ADRs in dependency order
- Review all architectural diagrams
- Deep dive into scenarios in HLD/
- Review threat model and testing approaches

---

## 🎯 Common Questions → Documents

| Question | Document | Section |
|----------|----------|---------|
| **"What's the ROI?"** | [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) | Financial Summary |
| **"Why microservices?"** | [ADR-01](ADR/ADR_01_microservices_architecture.md) | Decision |
| **"Why not Gen AI for pricing?"** | [ADR-02](ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) | Why ML, Not Gen AI section |
| **"How do we migrate?"** | [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) | All phases |
| **"What about security?"** | [THREAT_MODEL.md](THREAT_MODEL/THREAT_MODEL.md) | All sections |
| **"How do we test ML models?"** | [ADR-16](ADR/ADR_16_MLOps_Pipeline.md) | Model Evaluation |
| **"What are the personas?"** | [PERSONAS.md](PERSONAS.md) | All personas |
| **"What's the domain model?"** | [EVENT_STORMING.md](EVENT_STORMING.md) | Domain Events |
| **"Build vs buy decisions?"** | [PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md) | Opportunity Costs |
| **"What are the costs?"** | [COST_ANALYSIS.md](COST_ANALYSIS.md) | All sections |

---

**Need help navigating?** Open an issue on GitHub or contact the team.

**Remember:** *"Architecture is one foot in business, one foot in technology."* - O'Reilly Katas Judges
