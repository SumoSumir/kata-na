# ADR-01: Microservices Architecture Selection

## Status
Accepted

## Context
MobilityCorp operates a multi-modal transportation platform across multiple EU countries, managing bikes, scooters, cars, and vans with distinct operational characteristics. The system must support vehicle diversity, scaling variance across services, geographic expansion, and critical reliability.

Business requirements include high testability for minimal bug and rapid feature iteration, independent service scaling to optimize costs, and fault isolation to prevent cascading failures.

## Alternatives Considered

**1. Service-Based Architecture**
- Coarse-grained services (e.g., "Fleet Operations", "User Management")
- ✅ **Pros:** Some independent scaling, simpler than microservices, reduced operational overhead
- ❌ **Cons:** Still couples dissimilar scaling profiles (telemetry + booking in same service), limited fault isolation, coordinated deployment needed for related features
- **Rejected:** Insufficient scaling flexibility and fault isolation

**2. Microservices Architecture**
- Fine-grained, independently deployable services per business capability
- ✅ **Pros:** Maximum scaling flexibility, strong fault isolation, independent deployment cycles, technology diversity
- ❌ **Cons:** Distributed system complexity, higher infrastructure costs, eventual consistency challenges
- **Accepted:** Best meets scaling, reliability, and development velocity requirements

## Decision
Implement microservices architecture with services organized by business capability. Core services include Vehicle Telemetry, Booking, Payment, User/KYC, and Fleet Operations.

**Deployment Platform:** AWS ECS Fargate (serverless containers)
**Container Orchestration:** ECS with Service Discovery (AWS Cloud Map)
**Load Balancing:** Application Load Balancer (ALB) with health checks
**Service Mesh:** AWS App Mesh for advanced traffic management (optional, Phase 3+)

**Core Services:**
- **Vehicle Telemetry Service:** Node.js (real-time processing), 10 tasks, 2 vCPU, 4GB RAM
- **Booking Service:** Python FastAPI, 20 tasks (peak), 2 vCPU, 4GB RAM
- **Payment Service:** Python, 15 tasks, 2 vCPU, 4GB RAM (PCI-DSS compliant)
- **User/KYC Service:** Python FastAPI, 12 tasks, 2 vCPU, 4GB RAM
- **Fleet Operations Service:** Python, 10 tasks, 2 vCPU, 4GB RAM
- **Pricing Service:** Python, 8 tasks, 1 vCPU, 2GB RAM
- **Analytics Service:** Python, 6 tasks, 4 vCPU, 8GB RAM
- **Notification Service:** Python, 5 tasks, 1 vCPU, 2GB RAM

**Rationale for Microservices over Service-Based Architecture:**
- **Independent Scaling:** Telemetry processes continuous high-volume data streams requiring aggressive scaling, while booking/payment services handle low-volume transactional loads. Service-based architecture would force scaling entire service groups together, wasting resources. Microservices enable precise capacity allocation per service.
- **Fault Isolation:** A booking service failure shouldn't disrupt vehicle tracking. Service-based architecture couples dissimilar functions, risking cascading failures. Microservices contain failures within service boundaries.
- **Technology Flexibility:** Telemetry benefits from time-series databases, booking from relational databases, and fleet operations from graph databases. Service-based architecture forces technology compromises; microservices allow optimal stacks per capability.
- **Development Velocity:** Enables independent changes and rollouts (e.g., Pricing, Booking) using backward‑compatible APIs, consumer‑driven contracts, and feature flags — reducing coordination and supporting incremental releases.
- **Geographic Expansion:** Adding countries means deploying telemetry instances regionally while potentially reusing existing booking infrastructure within the country. Service-based architecture complicates regional scaling; microservices support flexible geographic distribution.

Key principles:
- Business capability alignment with independent deployability
- Technology diversity allowing optimal stacks per service
- Data ownership with API-based cross-service access
- Horizontal scaling based on specific load patterns

## Integration with Other Architectural Decisions
This microservices foundation enables and is supported by other architectural decisions:
- **Event-Driven Architecture (ADR-06):** Asynchronous communication between services using Kafka
- **Distributed Tracing (ADR-07):** OpenTelemetry for observability across service boundaries
- **Multi-Region Deployment (ADR-09):** Regional service deployment for geographic expansion
- **Vehicle Telemetry Architecture (ADR-03):** Edge-cloud hybrid processing within microservices framework

## Consequences
✅ **Positive:**
- Granular scaling matching actual load patterns reduces infrastructure waste
- Service failures contained within boundaries; system degrades gracefully
- Teams deploy independently without coordination overhead
- Technology choices optimized per service capability
- New countries onboard faster with regional service deployment

❌ **Negative:**
- Distributed system complexity with network latency and eventual consistency
- Higher infrastructure costs for service discovery, monitoring, and inter-service communication
- Increased cognitive load for developers managing inter-service contracts