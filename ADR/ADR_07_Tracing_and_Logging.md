# ADR-07: Tracing & Logging Technology – OpenTelemetry

## Status 
Accepted

## Context
Modern distributed applications require robust tracing and logging for monitoring, debugging, and performance optimization. Several solutions exist for this purpose, including but not limited to:

- **OpenTelemetry** (emerging open standard for observability)
- **Prometheus** (metrics-focused, some tracing support via integrations)
- **Zipkin**/ **Jaeger** (dedicated distributed tracing systems)
- **AWS X-Ray**, **Google Cloud Trace**, **Datadog**, **NewRelic** (vendor-managed solutions)
- **Opencensus** (predecessor to OpenTelemetry, less actively maintained)

Requirements include:
- Support for distributed tracing and contextual logging.
- Integration with multiple languages, cloud providers, and frameworks.
- Vendor-neutral and future-proof.

## Decision
We will adopt **OpenTelemetry** as the observability framework with **AWS CloudWatch** and **AWS X-Ray** as the backend for logs, metrics, and distributed tracing.

**Architecture:**

1. **OpenTelemetry Instrumentation**
   - OpenTelemetry SDKs in all microservices (Python, Node.js, Java)
   - Auto-instrumentation for common frameworks (FastAPI, Express, Spring Boot)
   - Custom spans for business-critical operations (booking, payment processing)
   - Baggage for cross-service context propagation (user_id, request_id, trace_id)

2. **AWS X-Ray (Distributed Tracing)**
   - OpenTelemetry Collector exports traces to X-Ray via OTLP exporter
   - Service map visualization showing microservices and dependencies
   - Configurable trace retention policy
   - Sampling strategy: Full sampling for errors, reduced sampling for successful requests (cost optimization)

3. **AWS CloudWatch Logs (Centralized Logging)**
   - OpenTelemetry Logs API → CloudWatch Logs via AWS for Fluent Bit sidecar
   - Log groups organized per microservice
   - Structured JSON logs with trace context (trace_id, span_id injected)
   - Tiered retention: Hot storage with cold archive to S3 via Kinesis Firehose
   - CloudWatch Logs Insights for querying (SQL-like syntax)

4. **AWS CloudWatch Metrics (Application Metrics)**
   - OpenTelemetry Metrics SDK → CloudWatch Metrics via EMF (Embedded Metric Format)
   - Custom application metrics (latency, availability, success rates)
   - High-resolution metrics for critical SLIs
   - CloudWatch Alarms for SLO violations

5. **OpenTelemetry Collector Deployment**
   - ECS Fargate sidecar pattern for containerized workloads
   - Batch processor for efficient data transmission
   - Memory limiter for resource management
   - Exporters: AWS X-Ray, CloudWatch Logs, CloudWatch EMF
   - High availability: Multi-AZ deployment with auto-scaling

6. **Grafana for Visualization (Optional)**
   - Amazon Managed Grafana workspace
   - Unified dashboards combining X-Ray traces, CloudWatch metrics, and logs

**Justification:**
- **OpenTelemetry** provides vendor-neutral instrumentation (can switch backends if needed)
- **AWS X-Ray** offers native service map and deep AWS service integration (RDS, DynamoDB, Lambda)
- **CloudWatch** is cost-effective for AWS-native workloads with native ECS/Fargate integration
- **Unified observability:** Trace IDs link logs, metrics, and traces for root cause analysis
- **Cost optimization:** Sampling strategy reduces trace ingestion while maintaining error visibility

## Consequences

**Trade-Offs and Impact:**

- **Pros:**
  - Vendor-agnostic: reduces lock-in and eases future migration.
  - Broad language/framework support.
  - Active community and rapid evolution.
  - Integrates with current and legacy tracing/logging backends.

- **Cons:**
  - Evolving standard; breaking changes and incomplete features can occur.
  - May require additional operational overhead versus fully-managed solutions.
  - Integration complexity for highly customized setups.
  - Smaller organizations may prefer simpler or vendor-specific tools (e.g., Datadog, Cloud-provider tracing).

**Comparison Against Alternatives:**
- Compared to **Prometheus**: OpenTelemetry covers logs and traces (not just metrics), but Prometheus is more battle-tested for metrics.
- Compared to **Jaeger/Zipkin**: These are focused on tracing alone; OpenTelemetry covers full observability, and can export to Jaeger/Zipkin.
- Compared to **Vendor Solutions**: OpenTelemetry avoids lock-in and gives more control, but vendors can simplify setup and offer advanced features out-of-the-box.

**Summary:**  
By adopting OpenTelemetry, we gain flexibility, interoperability, and reduce vendor risk. Some operational overhead and evolution risk remain, but this approach best fits our long-term engineering objectives for distributed tracing and logging.
