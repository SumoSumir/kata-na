# ADR-10: Monitoring Data Store – VictoriaMetrics

## Status
Accepted

## Context
Selecting a backend data store for metrics is key to monitoring system performance and health. While **Prometheus** is the prevalent choice for time-series data, alternatives exist, including:

- **Prometheus** (widely used, but memory footprint can be high for large-scale workloads)
- **VictoriaMetrics** (drop-in Prometheus replacement, optimized for efficiency)
- **Thanos**, **Cortex** (scalable, multi-server Prometheus solutions)
- Cloud-based time-series databases (Amazon Timestream, Google Bigtable, etc.)

Key requirements:
- Efficiency and scalability for large volumes of high-cardinality metrics
- Reliable storage and retention
- Compatibility with Grafana dashboards, alerting, and integrations

## Decision
We will use **Amazon CloudWatch** as the primary metrics and monitoring platform, with **VictoriaMetrics** as an optional long-term storage backend for cost optimization, and **Amazon Managed Grafana** for visualization.

**AWS-Native Monitoring Architecture:**

1. **Amazon CloudWatch (Primary Metrics Store)**
   - **Native AWS service integration:** Automatic metrics from ECS, Lambda, Aurora, DynamoDB, MSK, etc.
   - **Custom application metrics:** OpenTelemetry → CloudWatch EMF (Embedded Metric Format)
   - **High-resolution metrics:** Sub-minute granularity for critical SLIs
   - **Configurable metric retention**

2. **CloudWatch Alarms & Anomaly Detection**
   - **Static alarms:** SLO violations with threshold-based alerting
   - **Anomaly detection:** ML-based anomaly detection for metrics
   - **Composite alarms:** Logical combinations for complex conditions
   - **SNS integration:** Alarms → SNS topics → Lambda → PagerDuty/Slack/Email

3. **VictoriaMetrics (Long-Term Storage, Optional)**
   - **Use case:** Store extended metric history for capacity planning and compliance
   - **Data flow:** CloudWatch → Lambda → VictoriaMetrics remote write API
   - **Deployment:** ECS Fargate with appropriate resource allocation
   - **Storage optimization:** Superior compression compared to Prometheus
   - **PromQL queries:** Grafana queries VictoriaMetrics for historical analysis
   - **Cost advantage:** Significant savings vs extended CloudWatch retention

4. **Amazon Managed Grafana (Visualization)**
   - **Grafana workspace** with CloudWatch and VictoriaMetrics data sources
   - **Pre-built dashboards:**
     - Service health: Request rate, latency, error rate (RED metrics)
     - Infrastructure: ECS CPU/memory, Aurora connections, MSK lag
     - Business KPIs: Active bookings, revenue, vehicle utilization
     - AI model performance: Inference latency, prediction accuracy, cost per request
   - **Alerting:** Grafana alerts → SNS → PagerDuty (alternative to CloudWatch Alarms)

5. **Amazon Timestream (Optional for IoT Telemetry Metrics)**
   - **Use case:** Store high-frequency vehicle telemetry metrics (battery, GPS, speed)
   - **Time-series optimized:** Cost-effective for time-series data
   - **Tiered retention:** Hot and cold storage tiers
   - **Query:** SQL-like syntax for time-series analysis

6. **PagerDuty Integration (Incident Management)**
   - **On-call rotation:** Primary/secondary engineers for critical alerts
   - **Escalation policy:** Automated escalation based on severity
   - **Incident timeline:** Integrates alerts, CloudWatch graphs, and X-Ray traces

7. **Monitoring Agents & Instrumentation**
   - **OpenTelemetry Collector:** ECS Fargate sidecar exports metrics to CloudWatch EMF
   - **CloudWatch Agent:** EC2/Fargate logs and custom metrics
   - **AWS Distro for OpenTelemetry (ADOT):** AWS-supported OpenTelemetry distribution
   - **Auto-scaling triggers:** CloudWatch alarms → ECS service auto-scaling policies

**Metrics Strategy:**
- **Golden Signals (SRE best practices):**
  - **Latency:** Percentiles (P50, P95, P99) for all API endpoints
  - **Traffic:** Requests/sec per microservice
  - **Errors:** Error rate percentage
  - **Saturation:** Resource utilization (CPU, memory, database connections)
- **Business metrics:**
  - Revenue tracking
  - Active bookings
  - Vehicle utilization
  - Customer satisfaction indicators
- **AI-specific metrics:**
  - Model inference latency
  - Prediction accuracy (monitored via A/B testing)
  - Cost per inference

**Justification:**
- **CloudWatch** provides zero-config integration with all AWS services
- **VictoriaMetrics** reduces long-term storage costs significantly
- **Grafana** offers superior visualization and cross-source querying
- **PagerDuty** ensures rapid incident response with escalation policies

## Consequences

**Trade-Offs and Impact:**

- **Pros:**
  - Better memory and resource efficiency for large-scale metric workloads.
  - Supports high availability and long-term metric storage.
  - Full Prometheus compatibility (configuration, querying, integration).
  - Reduced hardware requirements, potential cost savings.

- **Cons:**
  - Slightly smaller ecosystem than Prometheus, but rapidly growing.
  - Newer features/updates may lag behind upstream Prometheus initially.
  - May require custom configuration for some advanced Prometheus features.

**Comparison Against Alternatives:**
- Compared to **Prometheus**: VictoriaMetrics offers superior performance and lower memory usage at scale.
- Compared to **Thanos/Cortex**: VictoriaMetrics is generally simpler to operate and less resource-intensive, but lacks some multi-tenancy/distributed options.
- Compared to **Cloud databases**: Provides more control, avoids vendor lock-in, usually lower cost for self-hosted scenarios.

**Summary:**  
VictoriaMetrics delivers the efficiency, scale, and compatibility needed for our metric storage requirements, supporting Grafana dashboards and PagerDuty alerting with reduced resource costs versus standard Prometheus.
