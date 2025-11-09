# Deployment Diagram: Multi-Region

This diagram shows a high-level overview of the multi-region deployment strategy, ensuring high availability and data residency.

```
┌───────────────────────────────────────────────────────────────────────────┐
│                               Global Layer                                │
├───────────────────────────────────────────────────────────────────────────┤
│ ┌───────────────────────────────────────────────────────────────────────┐ │
│ │                            Route 53 (Global DNS)                      │ │
│ └────────────────────────┬────────────────────────┬────────────────────────┘ │
│                          │                        │                         │
│                          │                        │                         │
│                          ▼                        ▼                         │
│      (latency-based routing)           (latency-based routing)              │
└───────────────────────────────────────────────────────────────────────────┘


     ┌─────────────────────────────┐                    ┌─────────────────────────────┐                     ┌────────────────────────────┐
     │       EU Region (Ireland)   │                    │      France Region (Paris)  │                     │     India Region (Possible Future)  │
     │            eu-west-1        │                    │           eu-west-3         │                     │         ap-south-1         │
     ├─────────────────────────────┤                    ├─────────────────────────────┤                     ├───────────────────────────┤
     │ ┌─────────────────────────┐ │                    │ ┌─────────────────────────┐ │                     │ ┌─────────────────────────┐ │
     │ │ Regional Load Balancer  │ │                    │ │ Regional Load Balancer  │ │                     │ │ Regional Load Balancer  │ │
     │ └───────────┬────────────┘ │                     │ └───────────┬────────────┘ │                     │ └───────────┬────────────┘ │
     │             │              │                     │             │              │                     │             │              │
     │ ┌───────────▼────────────┐ │                     │ ┌───────────▼────────────┐ │                     │ ┌───────────▼────────────┐ │
     │ │  Kubernetes Cluster    │ │                     │ │  Kubernetes Cluster    │ │                     │ │  Kubernetes Cluster    │ │
     │ │  (Microservices)       │ │                     │ │  (Microservices)       │ │                     │ │  (Microservices)       │ │
     │ └────────────────────────┘ │                     │ └────────────────────────┘ │                     │ └────────────────────────┘ │
     │                            │                     │                            │                     │                            │
     │ ┌────────────────────────┐ │                     │ ┌────────────────────────┐ │                     │ ┌────────────────────────┐ │
     │ │ PostgreSQL (PII)       │ │                     │ │ PostgreSQL (PII)       │ │                     │ │ PostgreSQL (PII)       │ │
     │ │ GDPR Compliant         │ │                     │ │ GDPR + FDPA Compliant  │ │                     │ │ DPDP Compliant          │ │
     │ │ (No PII Replication)   │ │                     │ │ (No PII Replication)   │ │                     │ │ (No PII Replication)   │ │
     │ └────────────────────────┘ │                     │ └────────────────────────┘ │                     │ └────────────────────────┘ │
     │                            │                     │                            │                     │                            │
     │ ┌────────────────────────┐ │<────────────────────>│ ┌────────────────────────┐ │<────────────────────>│ ┌────────────────────────┐ │
     │ │ TimescaleDB (Vehicle Telemetry)│ │             │ │ TimescaleDB (Vehicle Telemetry)│ │         │ │ TimescaleDB (Vehicle Telemetry)│ │
     │ │ Anonymized Replication │ │                     │ │ Anonymized Replication │ │                     │ │ Anonymized Replication │ │
     │ └────────────────────────┘ │                     │ └────────────────────────┘ │                     │ └────────────────────────┘ │
     └─────────────────────────────┘                     └─────────────────────────────┘                     └────────────────────────────┘
```
