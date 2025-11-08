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
     - Example metrics: `booking_latency_p99`, `vehicle_availability_ratio`, `payment_success_rate`
   - **High-resolution metrics:** 1-second granularity for critical SLIs (vs. 1-min standard)
   - **Metric retention:** 15 months by default
   - **Pricing:** Per custom metric, per alarm, and per API request

2. **CloudWatch Alarms & Anomaly Detection**
   - **Static alarms:** SLO violations (e.g., `booking_latency_p99 > 500ms` for 2 consecutive minutes)
   - **Anomaly detection:** ML-based anomaly detection for metrics (automatically adjusts to patterns)
   - **Composite alarms:** Logical combinations (e.g., "High latency AND high error rate")
   - **SNS integration:** Alarms → SNS topics → Lambda → PagerDuty/Slack/Email
   - **Example:** Critical alarms → PagerDuty (on-call), warning alarms → Slack #engineering

3. **VictoriaMetrics (Long-Term Storage, Optional)**
   - **Use case:** Store 2+ years of metrics for capacity planning and compliance
   - **Data flow:** CloudWatch → Lambda → VictoriaMetrics remote write API
   - **Deployment:** ECS Fargate, 4 vCPU, 16 GB RAM, gp3 SSD storage
   - **Storage optimization:** 10x compression vs. Prometheus
   - **PromQL queries:** Grafana can query VictoriaMetrics for historical analysis
   - **Cost advantage:** Significant savings vs extended CloudWatch retention
   - **Justification:** Reduces long-term storage costs dramatically

4. **Amazon Managed Grafana (Visualization)**
   - **Grafana workspace** with CloudWatch and VictoriaMetrics data sources
   - **Pre-built dashboards:**
     - Service health: Request rate, latency, error rate (RED metrics)
     - Infrastructure: ECS CPU/memory, Aurora connections, MSK lag
     - Business KPIs: Active bookings, revenue/hour, vehicle utilization
     - AI model performance: Inference latency, prediction accuracy, cost per request
   - **Alerting:** Grafana alerts → SNS → PagerDuty (alternative to CloudWatch Alarms)
   - **Pricing:** Workspace and per-editor user pricing

5. **Amazon Timestream (Optional for IoT Telemetry Metrics)**
   - **Use case:** Store high-frequency vehicle telemetry metrics (battery, GPS, speed)
   - **Time-series optimized:** 1/10 cost of relational DB for time-series data
   - **Retention:** 7 days in memory, 90 days in magnetic storage
   - **Query:** SQL-like syntax for time-series analysis
   - **Pricing:** Based on data scanned and writes

6. **PagerDuty Integration (Incident Management)**
   - **On-call rotation:** Primary/secondary engineers for critical alerts
   - **Escalation policy:** Critical → Page immediately, High → Slack + Page after 5 min
   - **Incident timeline:** All alerts, CloudWatch graphs, and X-Ray traces linked
   - **Pricing:** Per user subscription

7. **Monitoring Agents & Instrumentation**
   - **OpenTelemetry Collector:** ECS Fargate sidecar exports metrics to CloudWatch EMF
   - **CloudWatch Agent:** EC2/Fargate logs and custom metrics
   - **AWS Distro for OpenTelemetry (ADOT):** AWS-supported OpenTelemetry distribution
   - **Auto-scaling triggers:** CloudWatch alarms → ECS service auto-scaling policies

**Metrics Strategy:**
- **Golden Signals (SRE best practices):**
  - **Latency:** P50, P95, P99 for all API endpoints
  - **Traffic:** Requests/sec per microservice
  - **Errors:** Error rate % (4xx, 5xx)
  - **Saturation:** CPU, memory, database connections
- **Business metrics:**
  - Revenue per hour
  - Active bookings
  - Vehicle utilization %
  - Customer satisfaction (NPS proxy: app rating)
- **AI-specific metrics:**
  - Model inference latency (P99 < 200ms)
  - Prediction accuracy (monitored via A/B testing)
  - Cost per inference optimization targets

**Monitoring Infrastructure Costs:**
- CloudWatch: Metrics, alarms, and logs (see ADR-07 for details)
- Amazon Managed Grafana: Workspace and user licensing
- VictoriaMetrics: Optional long-term storage with cost optimization
- PagerDuty: Incident management platform

**Cost Considerations:**
- CloudWatch provides native AWS integration
- VictoriaMetrics significantly reduces long-term storage costs vs CloudWatch alone
- Grafana enables superior visualization and cross-source querying

**Justification:**
- **CloudWatch** provides zero-config integration with all AWS services (ECS, Aurora, MSK, Lambda)
- **VictoriaMetrics** reduces long-term storage costs significantly
- **Grafana** offers superior visualization and cross-source querying (CloudWatch + VictoriaMetrics)
```
- **PagerDuty** ensures < 5 min response time for critical incidents (99.9% uptime SLA)

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
