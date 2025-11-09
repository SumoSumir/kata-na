# Glossary

This glossary defines key terms, acronyms, and concepts used throughout the MobilityCorp Architecture Documentation.

---

## A

**ADR (Architecture Decision Record)**
- Documented architectural decisions with context, alternatives, and rationale
- Format: ADR-XX: Title
- Example: ADR-15: Cloud Provider Selection

**Agent (AI Agent)**
- Autonomous AI system that can perceive environment, make decisions, and take actions
- Uses LLMs with tool calling capabilities
- Examples: Trip Planning Assistant, Operations Copilot

**API Gateway**
- Service that handles all API requests, provides rate limiting, authentication
- AWS API Gateway used in our architecture

**Aurora PostgreSQL**
- AWS managed relational database, PostgreSQL-compatible
- Used for transactional data (bookings, users, payments)

---

## B

**Batch Inference**
- ML predictions run on large datasets at scheduled intervals
- Contrast with real-time inference
- Examples: Weekly battery degradation predictions

**Bedrock (AWS Bedrock)**
- AWS managed service for foundation models (LLMs)
- Provides Claude, Llama, Mistral, Titan models
- Used for conversational AI agents

**Blue-Green Deployment**
- Deployment strategy with two identical environments (blue/current, green/new)
- Traffic switched instantly from blue to green after validation
- Enables zero-downtime updates

**Bronze Layer**
- First layer in Medallion Architecture
- Raw, unprocessed data from sources
- Append-only, schema-on-read
- Retention: 30 days

---

## C

**Canary Deployment**
- Gradual rollout strategy starting with small percentage of traffic
- Monitor metrics before increasing traffic
- Example: 5% → 10% → 25% → 50% → 100%

**CAN Bus (Controller Area Network)**
- Vehicle communication protocol for sensors and control units
- Provides rich telemetry (speed, RPM, battery, diagnostics)
- Used in cars and vans, not scooters/eBikes

**CQRS (Command Query Responsibility Segregation)**
- Pattern separating read and write operations
- Optimizes for different access patterns
- Used in high-traffic services (booking, fleet)

**Claude (Claude 3.5 Sonnet)**
- Large Language Model by Anthropic
- 200K context window, excellent tool use
- Our primary LLM via AWS Bedrock

---

## D

**Data Drift**
- Change in input data distribution over time
- Causes model accuracy degradation
- Detected by SageMaker Model Monitor

**Data Lakehouse**
- Hybrid architecture combining data lake (S3) + data warehouse (Redshift)
- Enables both batch analytics and real-time queries
- Our implementation uses Medallion Architecture

**Delta Lake**
- Open-source storage format providing ACID transactions on data lakes
- Adds reliability and performance to Parquet files
- Used in Silver and Gold layers

**DPU (Data Processing Unit)**
- AWS Glue compute unit
- 1 DPU = 4 vCPU + 16GB RAM
- Billed per DPU-hour ($0.44)

**DynamoDB**
- AWS managed NoSQL database
- Low-latency key-value store
- Used for real-time operational data (vehicle status, conversations)

---

## E

**eBike (Electric Bike)**
- Electric bicycle with pedal assist
- Max speed: ~25 km/h (EU regulation)
- Battery-powered, requires manual battery swaps

**ECS (Elastic Container Service)**
- AWS container orchestration service
- Fargate mode: serverless containers
- Used for all microservices

**Edge Computing**
- Processing data on device (vehicle) instead of cloud
- Reduces latency, works offline, saves bandwidth
- Used for safety-critical ML models

**Event-Driven Architecture**
- Architecture where components communicate via events
- Asynchronous, loosely coupled
- Kafka/MSK as event backbone

**Event Sourcing**
- Pattern storing all changes as sequence of events
- Enables audit trail and temporal queries
- Used for booking and payment services

---

## F

**Fargate**
- AWS serverless container platform
- No EC2 instances to manage
- Pay per vCPU/GB used

**Feature Store**
- Repository of ML features for training and serving
- Online store (low-latency) + Offline store (historical)
- SageMaker Feature Store used

**Fitness Function**
- Quantifiable metric measuring architectural success
- Examples: P95 latency < 200ms, uptime > 99.95%
- Automated checks prevent architectural drift

---

## G

**GDPR (General Data Protection Regulation)**
- EU regulation for data protection and privacy
- Requirements: data residency, right to erasure, consent
- All our data stays in EU (Frankfurt + Ireland)

**Geofence**
- Virtual geographic boundary
- Triggers alerts when vehicles enter/exit zones
- Used for operational areas and parking validation

**Glue (AWS Glue)**
- AWS serverless ETL service
- Used for Bronze → Silver → Gold transformations
- Supports PySpark for big data processing

**Gold Layer**
- Third layer in Medallion Architecture
- Business-ready aggregations and metrics
- Optimized for BI and reporting
- Retention: 3 years

**Greengrass (AWS IoT Greengrass v2)**
- AWS edge runtime for IoT devices
- Runs ML models locally on vehicles
- Supports OTA updates and offline operation

---

## H

**HLD (High-Level Design)**
- Detailed technical documentation of system components
- Includes data flow diagrams, sequence diagrams
- More detailed than ADRs

**Hybrid AI**
- Combination of traditional ML (XGBoost, LightGBM) + GenAI (LLMs)
- Leverages strengths of each approach
- Example: ML for predictions, LLM for explanations

---

## I

**IAM (Identity and Access Management)**
- AWS service for authentication and authorization
- Role-based access control (RBAC)
- Used for all service-to-service auth

**Iceberg (Apache Iceberg)**
- Open table format for data lakes
- Alternative to Delta Lake
- Not used (we chose Delta Lake)

**IMU (Inertial Measurement Unit)**
- Sensor measuring acceleration and rotation
- Includes accelerometer + gyroscope
- Used for collision detection

**IoT Core (AWS IoT Core)**
- AWS managed MQTT broker
- Handles 50K+ vehicle connections
- Routes telemetry to Kafka and Lambda

**IoT Greengrass** - See **Greengrass**

---

## K

**Kafka (Apache Kafka)**
- Distributed event streaming platform
- MSK (Managed Streaming for Kafka) on AWS
- Our event backbone for telemetry and events

**KPI (Key Performance Indicator)**
- Measurable value demonstrating business effectiveness
- Examples: vehicle utilization rate, revenue per vehicle
- Tracked in Gold layer aggregations

**KYC (Know Your Customer)**
- Identity verification process
- Required for user registration
- Documents stored encrypted in S3

---

## L

**Lake Formation (AWS Lake Formation)**
- AWS service for data lake governance
- Fine-grained access control (row/column level)
- Tracks data lineage

**Lambda (AWS Lambda)**
- AWS serverless functions
- Event-driven, auto-scaling
- Used for lightweight processing, API endpoints

**LangChain**
- Framework for building LLM applications
- Provides agents, tools, memory, chains
- Our agentic AI framework

**LightGBM**
- Gradient boosting framework by Microsoft
- Fast, efficient for tabular data
- Used for demand forecasting

**LLM (Large Language Model)**
- AI model trained on vast text data
- Examples: Claude 3.5, GPT-4, Llama 3.1
- Used for conversational AI

---

## M

**MAPE (Mean Absolute Percentage Error)**
- ML accuracy metric for regression
- Target: < 15% for demand forecasting
- Lower is better

**Medallion Architecture**
- Data lakehouse pattern with Bronze/Silver/Gold layers
- Each layer adds value (cleaning, transformation, aggregation)
- Our data platform design

**Microservices**
- Architectural style with small, independent services
- Each service owns its data and API
- Enables independent scaling and deployment

**MLOps**
- Practices for ML model lifecycle management
- Training, versioning, deployment, monitoring
- Implemented with SageMaker Pipelines

**Model Drift**
- Degradation of model accuracy over time
- Caused by data drift or concept drift
- Triggers model retraining

**Model Registry**
- Repository of trained ML models with metadata
- Tracks versions, metrics, lineage
- SageMaker Model Registry used

**MSK (Managed Streaming for Kafka)**
- AWS fully managed Apache Kafka service
- Our event streaming backbone
- 3-broker cluster in production

**MQTT (Message Queuing Telemetry Transport)**
- Lightweight pub/sub protocol for IoT
- Used for vehicle-to-cloud communication
- IoT Core as MQTT broker

---

## N

**Neo (SageMaker Neo)**
- AWS service for ML model optimization
- Compiles models for edge devices
- Reduces model size and improves inference speed

**NFR (Non-Functional Requirement)**
- System quality attribute (performance, security, scalability)
- Examples: 99.95% uptime, P95 latency < 200ms
- Enforced by fitness functions

---

## O

**OCU (OpenSearch Compute Unit)**
- Pricing unit for OpenSearch Serverless
- ~$0.24/OCU-hour
- Auto-scales based on workload

**OpenSearch**
- AWS managed search and analytics engine
- Used as vector database for RAG
- Stores knowledge base embeddings

**OTA (Over-The-Air)**
- Remote software/model updates
- Delivered via IoT Greengrass
- No physical access required

---

## P

**Parquet**
- Columnar storage format
- Efficient for analytics workloads
- Used in Bronze layer

**PII (Personally Identifiable Information)**
- Data that identifies individuals
- Examples: name, email, phone, address
- Must be protected per GDPR

**Point-in-Time Correctness**
- Feature Store capability ensuring features match training data time
- Prevents data leakage
- Critical for accurate ML models

**Prompt Engineering**
- Crafting LLM prompts for optimal responses
- Includes system prompt, few-shot examples, context
- Iterative optimization process

**Pub/Sub (Publisher/Subscriber)**
- Messaging pattern where publishers send to topics, subscribers receive
- Decouples producers and consumers
- Used in Kafka, AWS IoT Core

---

## Q

**Quantization**
- Reducing ML model precision (FP32 → INT8)
- 4x smaller models, faster inference
- Used for edge deployment

---

## R

**RAG (Retrieval-Augmented Generation)**
- Technique combining LLM with external knowledge base
- Retrieves relevant documents, then generates response
- Used for FAQ chatbot

**ReAct (Reasoning + Acting)**
- Agent pattern alternating between thinking and tool use
- Format: Thought → Action → Observation → repeat
- Our agentic AI approach

**Redshift**
- AWS data warehouse
- SQL-based analytics on petabyte scale
- Used for BI dashboards (Gold layer)

**Redis (Amazon ElastiCache)**
- In-memory cache
- Sub-millisecond latency
- Used for hot data (availability, pricing)

**RUL (Remaining Useful Life)**
- Predicted time until component failure
- Example: Battery RUL = 45 days
- Enables proactive maintenance

---

## S

**S3 (Simple Storage Service)**
- AWS object storage
- Unlimited scale, 11 nines durability
- Foundation of our data lakehouse

**SageMaker**
- AWS ML platform
- End-to-end: data prep, training, deployment, monitoring
- Our MLOps backbone

**Scooter (Electric Scooter)**
- Stand-up electric scooter
- Max speed: ~20 km/h
- Battery-powered, requires manual swaps

**Silver Layer**
- Second layer in Medallion Architecture
- Cleaned, validated, deduplicated data
- Delta Lake format with ACID
- Retention: 2 years

**Spot Instances**
- AWS EC2 instances at 70-90% discount
- Can be interrupted with 2-minute notice
- Used for ML training jobs

**STRIDE**
- Threat modeling framework
- Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- Used in our threat model

---

## T

**Telemetry**
- Automatic measurement and transmission of data
- Vehicle telemetry: GPS, battery, speed, IMU
- ~50K vehicles × 1 msg/sec = 4.3B events/day

**TensorFlow Lite**
- Lightweight ML framework for mobile/edge
- Optimized for constrained devices
- Used for collision detection on edge

**Time-Series Database**
- Database optimized for time-stamped data
- AWS Timestream used for vehicle telemetry
- Efficient queries on time ranges

**Timestream (Amazon Timestream)**
- AWS managed time-series database
- Serverless, auto-scaling
- Used for high-frequency telemetry

---

## U

**Utilization Rate**
- Percentage of time vehicle is rented
- Formula: (rental_time / total_available_time) × 100%
- Target: +15% YoY improvement

---

## V

**Vertex AI**
- Google Cloud's unified ML platform
- Alternative to SageMaker (not chosen)
- Excellent AutoML capabilities

**VPC (Virtual Private Cloud)**
- Isolated network in AWS
- Security groups, subnets, route tables
- All services deployed within VPC

---

## X

**X-Ray (AWS X-Ray)**
- Distributed tracing service
- Tracks requests across microservices
- Identifies performance bottlenecks

**XGBoost**
- Gradient boosting library
- Fast, accurate for structured data
- Used for dynamic pricing model

---

## Z

**Z-Order Clustering**
- Delta Lake optimization technique
- Co-locates related data on storage
- Improves query performance (data skipping)

**Zone**
- Geographic area for operations
- Examples: "Frankfurt Downtown", "Airport District"
- Used for demand forecasting and pricing

---

## Acronyms Quick Reference

| Acronym | Full Form |
|---------|-----------|
| ADR | Architecture Decision Record |
| API | Application Programming Interface |
| ACID | Atomicity, Consistency, Isolation, Durability |
| AWS | Amazon Web Services |
| CAN | Controller Area Network |
| CDN | Content Delivery Network |
| CI/CD | Continuous Integration/Continuous Deployment |
| CQRS | Command Query Responsibility Segregation |
| DPU | Data Processing Unit |
| ECS | Elastic Container Service |
| ETL | Extract, Transform, Load |
| GDPR | General Data Protection Regulation |
| GPS | Global Positioning System |
| HA | High Availability |
| HLD | High-Level Design |
| IAM | Identity and Access Management |
| IMU | Inertial Measurement Unit |
| IoT | Internet of Things |
| KPI | Key Performance Indicator |
| KYC | Know Your Customer |
| LLM | Large Language Model |
| MAPE | Mean Absolute Percentage Error |
| ML | Machine Learning |
| MLOps | Machine Learning Operations |
| MQTT | Message Queuing Telemetry Transport |
| MSK | Managed Streaming for Kafka |
| NFR | Non-Functional Requirement |
| NLP | Natural Language Processing |
| OCU | OpenSearch Compute Unit |
| OTA | Over-The-Air |
| PII | Personally Identifiable Information |
| RAG | Retrieval-Augmented Generation |
| RBAC | Role-Based Access Control |
| REST | Representational State Transfer |
| RUL | Remaining Useful Life |
| S3 | Simple Storage Service (AWS) |
| SLA | Service Level Agreement |
| SQL | Structured Query Language |
| TCO | Total Cost of Ownership |
| TLS | Transport Layer Security |
| VPC | Virtual Private Cloud |

---

## Related Documentation

- **[README.md](README.md)** - Project overview and navigation
- **[COST_ANALYSIS.md](COST_ANALYSIS.md)** - Cost breakdown (percentage-based)
- **[ADRs](./ADR/)** - All architectural decisions
- **[Functional Requirements](FUNCTIONAL_REQUIREMENTS/FUNCTIONAL_REQUIREMENTS.md)**
- **[Non-Functional Requirements](NON_FUNCTIONAL_REQUIREMENTS/NON_FUNCTIONAL_REQUIREMENTS.md)**

---

**Last Updated:** 2025-11-07  
**Maintained By:** Team kata-na
