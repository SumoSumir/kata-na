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
We will use **VictoriaMetrics** as our primary metrics data store.

**Justification:**
- Offers lower memory footprint and resource usage compared to standard Prometheus, reducing operational costs.
- Highly performant and scalable for high-cardinality, long-retention setups.
- Maintains compatibility with Prometheus protocols (scraping, querying, remote write/read), ensuring seamless integration with Grafana and existing alert systems (including PagerDuty).
- Open-source and widely adopted in performance-intensive environments.

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
