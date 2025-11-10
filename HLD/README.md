# High-Level Design (HLD) Documentation

This directory contains detailed high-level design documentation for the MobilityCorp multi-modal mobility platform, including end-to-end scenario walkthroughs with sequence diagrams and data flows.

---

## Overview

The High-Level Design documents describe how different components of the architecture interact to fulfill specific business scenarios. Each scenario includes:

- **Business Context:** Why this scenario matters
- **Actors:** Users, systems, and services involved
- **Sequence Diagrams:** Visual representation of interactions
- **Data Flow:** How data moves through the system
- **Error Handling:** What happens when things go wrong
- **Performance Considerations:** Latency, throughput, scalability
- **Cost Implications:** Infrastructure costs for this scenario

---

## Scenarios

### 1. [Booking Workflow](scenarios/booking_workflow.md)

End-to-end user journey from searching for a vehicle to completing a rental.

**Key Components:**
- Mobile App (React Native)
- API Gateway
- Booking Service
- Fleet Service
- Payment Service (Stripe integration)
- Notification Service
- Aurora PostgreSQL
- ElastiCache (Redis)

**Flow:** User Search → Vehicle Selection → Payment → Booking Confirmation → Vehicle Unlock

---

### 2. [Demand Forecasting](scenarios/demand_forecasting.md)

ML pipeline predicting vehicle demand by zone, time, and vehicle type.

**Key Components:**
- Apache Beam (ETL)
- SageMaker (Training & Inference)
- Feature Store
- S3 Data Lake (Medallion Architecture)
- Apache Airflow (Scheduled Predictions)
- DynamoDB (Forecast Storage)

**Flow:** Historical Data → Feature Engineering → Model Training → Batch Prediction → Operational Integration

---

### 3. [Dynamic Pricing](scenarios/dynamic_pricing.md)

Real-time pricing adjustments based on demand, supply, weather, and events.

**Key Components:**
- Pricing Service (EKS auto scaling with KEDA)
- SageMaker Inference Endpoint (XGBoost)
- Feature Store (Online)
- ElastiCache (Redis Cache)
- Apache Airflow (Trigger Repricing)
- Third-party APIs (Weather, Events)

**Flow:** Trigger Event → Feature Retrieval → ML Prediction → Price Update → Cache Invalidation

---

### 4. [Predictive Maintenance](scenarios/predictive_maintenance.md)

Proactive vehicle maintenance scheduling using ML models on telemetry data.

**Key Components:**
- AWS IoT Core (Telemetry Ingestion)
- Kafka (Event Streaming)
- TimescaleDB (Time-Series Storage)
- SageMaker (Health Prediction)
- Apache Airflow (Alert Processing)
- SNS (Maintenance Alerts)

**Flow:** Telemetry → Anomaly Detection → Health Prediction → Alert Generation → Maintenance Scheduling

---

### 5. [Telemetry Processing](scenarios/telemetry_processing.md)

High-throughput ingestion and processing of vehicle telemetry (GPS, battery, sensors).

**Key Components:**
- AWS IoT Core (MQTT)
- Kafka
- Apache Airflow+BEAM (Stream Processing)
- TimescaleDB (Hot Storage)
- S3 (Cold Storage)

**Flow:** Vehicle → IoT Core → Kafka → Apache Airflow+BEAM → TimescaleDB/S3 → Analytics

---

## Architecture Principles

All scenarios adhere to these design principles:

### 1. **Event-Driven Architecture**
- Asynchronous communication via Kafka
- Loose coupling between services
- Enables scalability and resilience

### 2. **CQRS (Command Query Responsibility Segregation)**
- Separate read and write paths
- Optimized for different access patterns
- Example: Booking writes to Aurora, reads from Redis

### 3. **Microservices**
- Each service owns its data
- Independent deployment and scaling
- Domain-driven design

### 4. **Data Lakehouse (Medallion Architecture)**
- Bronze: Raw data
- Silver: Cleaned, validated
- Gold: Business-ready aggregations

### 5. **Hybrid AI (Edge + Cloud)**
- Safety-critical models on edge (collision detection)
- Complex models in cloud (demand forecasting)
- Optimized for latency and cost

### 6. **Multi-Region HA (High Availability)**
- Active-active in eu-central-1 (Frankfurt) and eu-west-1 (Ireland)
- Cross-region replication for Aurora, S3
- Failover in < 1 minute

---

## Data Flow Patterns

### Pattern 1: Request-Response (Synchronous)

```
Mobile App → API Gateway → Microservice → Database → Response
```

**Use Cases:** Booking, user profile updates, vehicle search  
**Latency:** < 200ms (P95)  
**Consistency:** Strong (ACID transactions)

---

### Pattern 2: Event Streaming (Asynchronous)

```
Producer → Kafka → Consumer(s) → Side Effects
```

**Use Cases:** Telemetry processing, booking events, price changes  
**Latency:** < 5 seconds (eventual consistency)  
**Consistency:** Eventual (idempotent consumers)

---

### Pattern 3: Batch Processing (Scheduled)

```
S3 (Raw Data) → Apache BEAM (ETL) → S3 (Transformed) → Analytics
```

**Use Cases:** Daily aggregations, model training, reporting  
**Latency:** Minutes to hours  
**Consistency:** Eventual (batch window)

---

### Pattern 4: ML Inference (Real-Time)

```
Service → Feature Store → SageMaker Endpoint → Prediction → Service
```

**Use Cases:** Dynamic pricing, demand forecasting, fraud detection  
**Latency:** < 50ms (P99)  
**Consistency:** Point-in-time features

---

### Pattern 5: Agentic AI (LLM)

```
User → LangChain Agent → LLM (Bedrock) → Tool(s) → Response
```

**Use Cases:** Trip planning assistant, operations copilot  
**Latency:** 2-5 seconds (streaming)  
**Consistency:** Non-deterministic (LLM variance)

---

## Component Inventory

| Component | Type | Purpose | Scaling Strategy |
|-----------|------|---------|------------------|
| **API Gateway** | AWS Managed | API routing, throttling | Auto-scales |
| **EKS** | Compute | Microservices | Horizontal (tasks) |
| **Aurora PostgreSQL** | RDBMS | Transactional data | Read replicas |
| **DynamoDB** | NoSQL | Operational data | On-demand |
| **ElastiCache** | Cache | Hot data, sessions | Sharding |
| **Kafka/MSK** | Streaming | Event backbone | Add brokers/partitions |
| **S3** | Object Store | Data lake | Unlimited |
| **SageMaker** | ML Platform | Training, inference | Add endpoints |
| **Bedrock** | LLM | Agentic AI | Auto-scales |
| **IoT Core** | IoT | Vehicle telemetry | Auto-scales |
| **TimescaleDB** | Time-Series | Telemetry storage | Auto-scales |
| **Redshift** | Data Warehouse | Analytics | Add nodes |
| **Apache BEAM** | ETL | Data transformation | Add DPUs |
| **OpenSearch** | Vector DB | RAG knowledge base | Add shards |

---

## Cross-Cutting Concerns

### Security

- **Authentication:** OAuth 2.0 + JWT (Cognito)
- **Authorization:** RBAC (Role-Based Access Control)
- **Encryption:** TLS 1.3 in transit, AES-256 at rest (KMS)
- **Secrets:** AWS Secrets Manager
- **Network:** VPC, Security Groups, NACLs
- **Compliance:** GDPR, SOC 2, ISO 27001

**Reference:** [Threat Model](../THREAT_MODEL/THREAT_MODEL.md)

---

### Observability

- **Metrics:** CloudWatch custom metrics (1-min granularity)
- **Logs:** Centralized in CloudWatch Logs (30-day retention)
- **Traces:** AWS X-Ray (distributed tracing)
- **Dashboards:** Grafana (operational dashboards)
- **Alerts:** CloudWatch Alarms + SNS (PagerDuty integration)

**KPIs:**
- API latency P95 < 200ms
- Error rate < 0.1%
- Vehicle connectivity > 99.5%
- ML model accuracy > 85%

---

### Resilience

- **Retries:** Exponential backoff with jitter
- **Circuit Breakers:** Prevent cascading failures
- **Bulkheads:** Isolate critical services
- **Timeouts:** 30s max for API calls
- **Graceful Degradation:** Fallback to cached data

**Example:** If SageMaker pricing endpoint is down, fall back to rule-based pricing.

---

### Data Consistency

- **Strong Consistency:** Aurora PostgreSQL (ACID)
- **Eventual Consistency:** DynamoDB, S3, Kafka
- **Idempotency:** All event consumers use idempotency keys
- **Event Sourcing:** Booking and payment services

**Trade-offs:** Strong consistency sacrifices availability (CAP theorem).

---

## Performance Targets

| Metric | Target | Measured |
|--------|--------|----------|
| **API Latency (P95)** | < 200ms | CloudWatch |
| **API Latency (P99)** | < 500ms | CloudWatch |
| **ML Inference (P95)** | < 50ms | SageMaker |
| **Telemetry Latency** | < 5s | Kafka lag |
| **Page Load Time** | < 2s | RUM (Real User Monitoring) |
| **Vehicle Unlock** | < 3s | End-to-end test |
| **Availability** | 99.95% | Uptime monitoring |

---

## Cost Attribution

| Scenario | Monthly Cost | Cost per Transaction | Key Drivers |
|----------|-------------|---------------------|-------------|
| **Booking Workflow** | $25,000 | $0.025 (per booking) | ECS, Aurora, Stripe |
| **Demand Forecasting** | $30,000 | N/A (batch) | SageMaker, Apache BEAM |
| **Dynamic Pricing** | $15,000 | $0.000015 (per query) | SageMaker, Redis |
| **Predictive Maintenance** | $18,000 | $0.36 (per alert) | TimescaleDB, SageMaker |
| **Telemetry Processing** | $71,335 | $0.000017 (per message) | IoT Core |
| **Total** | **$159,335** | - | - |

**Reference:** [Cost Analysis](../COST_ANALYSIS.md)

---

## Related Documentation

- **[README.md](../README.md)** - Project overview
- **[Architecture Diagrams](../ARCHITECTURAL_DIAGRAMS/)** - Visual architecture
- **[ADRs](./ADR/)** - Architectural decisions
- **[Functional Requirements](../FUNCTIONAL_REQUIREMENTS/FUNCTIONAL_REQUIREMENTS.md)**
- **[Non-Functional Requirements](../NON_FUNCTIONAL_REQUIREMENTS/NON_FUNCTIONAL_REQUIREMENTS.md)**
- **[Fitness Functions](../FITNESS_FUNCTIONS/FITNESS_FUNCTIONS.md)**
- **[Threat Model](../THREAT_MODEL/THREAT_MODEL.md)**
- **[Phased Implementation](../PHASED_IMPLEMENTATION.md)** - Migration & rollout strategy