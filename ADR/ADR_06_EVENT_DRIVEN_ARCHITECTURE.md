# ADR-06: Event-Driven Architecture with Kafka

## Status
Accepted  

## Context
The AI platform needs to react to real-time events (bookings, battery status changes, GPS updates) and perform asynchronous processing.  

Key requirements include:
- Real-time demand prediction updates when bookings occur  
- Battery orchestrator monitoring vehicle status continuously  
- Pattern detection analyzing booking history  
- Decoupling between event producers and consumers  
- Ability to replay events for ML model training  
- Support for multiple consumers of the same event (e.g., analytics, ML, core services)

### Alternatives Considered

**1. Direct Service-to-Service REST Calls**  
- ✅ Pros: Simple, synchronous, easier to debug  
- ❌ Cons: Tight coupling, cascading failures, no event replay  
- **Rejected:** Doesn’t support real-time AI processing needs  

**2. AWS SQS/SNS**  
- ✅ Pros: Managed service, lower ops overhead  
- ❌ Cons: Limited replay capability, not ideal for high-throughput streaming  
- **Rejected:** Kafka better for ML training data replay and stream processing  

**3. RabbitMQ**  
- ✅ Pros: Feature-rich, good for task queues  
- ❌ Cons: Not optimized for high-throughput event streaming  
- **Rejected:** Kafka’s log-based architecture better suits real-time workloads  

## Decision
Implement an **event-driven architecture** using **Apache Kafka** as the central event bus.

### Core Event Topics
- `bookings.created` – New booking initiated  
- `bookings.completed` – Rental ended  
- `vehicles.telemetry` – GPS location updates (30s interval)  
- `vehicles.battery_status` – Battery level updates (60s interval)  
- `customers.app_events` – User interactions  
- `photos.uploaded` – Return validation photos  
- `predictions.generated` – AI prediction results  
- `operations.task_completed` – Staff task completion  

### Event Schema
- Use **Avro schemas** with **Schema Registry** for schema evolution  
- Include **correlation IDs** for distributed tracing  
- Timestamp all events with **event time** (not processing time)

### Processing Patterns
- Real-time stream processing using **Kafka Streams**  
- **Event sourcing** for audit trail  
- **CQRS** pattern for read-optimized views  

### Event Flow
`Producer → Kafka Topic → Multiple Consumers`  
(asynchronous, parallel processing)

## Consequences

### ✅ Positive
- **Decoupling:** Services operate independently  
- **Scalability:** Consumers process at their own pace  
- **Resilience:** Failed consumers can recover from offsets  
- **Replay capability:** Enables historical reprocessing for ML training  
- **Real-time analytics:** Supports live dashboards and streaming insights  
- **Flexibility:** Easy to onboard new consumers (e.g., AI services)

### ❌ Negative
- **Eventual consistency:** Updates are asynchronous  
- **Complex debugging:** Distributed event tracing adds complexity  
- **Ordering challenges:** Partition keys required for ordered processing  
- **Duplicate handling:** Consumers must be idempotent  
- **Infra overhead:** Kafka cluster adds operational cost

## AI-Specific Considerations
- **ML Training:** Supports event replay for historical model retraining  
- **Feature Engineering:** Kafka Streams enables real-time feature generation  
- **Model Monitoring:** Events provide ground truth for validation  
- **Real-time Inference:** Low-latency event flow enables live demand prediction
