# ADR-15: Data Lakehouse Strategy (Medallion Architecture)

## Status
Accepted

## Context
MobilityCorp's AI-enabled platform generates massive amounts of data from multiple sources:
- **Vehicle Telemetry:** 50K vehicles × 1 message/10 min = 7.2 million events/day
- **Bookings:** ~500K transactions/day
- **User Events:** ~2M interactions/day
- **External Data:** Weather, events, traffic (continuous streams)
- **Feedback & Images:** ~100K photos/day


## Requirements
- **Multi-modal access:** Real-time queries, batch analytics, ML training
- **Data quality:** Validation, deduplication, schema evolution
- **Governance:** RBAC, audit trails, GDPR compliance
- **Cost-efficiency:** Tiered storage with lifecycle policies

## Decision

**Adopt Medallion Architecture (Bronze/Silver/Gold) using S3 + Delta Lake, orchestrated by Apache Airflow + Apache Beam.**

### Architecture

```
Data Sources → Bronze (Raw) → Silver (Cleaned) → Gold (Aggregated) → Consumers
                      ↓              ↓                    ↓
                 Airflow DAGs orchestrate transformations
```

### Layer Details

**Bronze Layer (Raw Data)**
- Format: Parquet (Snappy compression)
- Partitioning: `/year/month/day/hour/`
- Retention: 90 days → Glacier
- Sources: Kafka → Kinesis Firehose → S3

**Silver Layer (Cleaned & Validated)**
- Format: Delta Lake (ACID transactions)
- Transformations:
  - Data quality checks (Great Expectations)
  - Deduplication, type casting, PII masking
  - Schema enforcement with evolution support
- Retention: 2 years
- Orchestration: Airflow DAGs trigger hourly/daily

**Gold Layer (Business Aggregations)**
- Format: Delta Lake + Redshift Spectrum
- Schema: Star schema for BI
- Aggregations: demand_by_zone, vehicle_utilization, revenue metrics
- Retention: 5 years (compliance)
- Orchestration: Airflow DAGs for daily aggregations

### Technology Stack

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Storage** | S3 | Durable (11 nines), tiered lifecycle, unlimited scale |
| **Table Format** | Delta Lake | ACID, time travel, schema evolution, open format |
| **Orchestration** | Airflow + Beam | Already adopted (ADR-08), DAG visualization, distributed ETL |
| **Query** | Athena + Redshift | Serverless SQL, BI integration |
| **Catalog** | AWS Glue Catalog | Metadata repository, schema registry |
| **Governance** | Lake Formation | RBAC, column-level security, audit logs |

### Data Governance

**Access Control (Lake Formation):**
- Data Scientists: Silver + Gold read access
- ML Engineers: Silver, Gold + Feature Store
- Analysts: Gold via Redshift (BI tools)
- Operations: Real-time operational views
- Column-level security for PII fields

**GDPR Compliance:**
- Right to erasure: Airflow DAG triggers Delta Lake `DELETE` + `VACUUM`
- Data minimization: Only necessary fields collected
- Retention policies: Automated lifecycle via S3 + Airflow cleanup jobs

## Consequences

### Positive
- ✅ **Leverages Existing Stack:** Uses Airflow already adopted in ADR-08
- ✅ **Cost-Effective:** S3 tiered storage + serverless Athena queries
- ✅ **Scalable:** Handles 5TB/month growth with S3 + Delta Lake
- ✅ **Flexible:** Supports batch, streaming, ad-hoc access patterns
- ✅ **ACID Guarantees:** Delta Lake ensures data consistency
- ✅ **Open Format:** Delta Lake is portable, not AWS-locked
- ✅ **Governed:** Lake Formation provides RBAC and audit trails

### Negative
- ❌ **Airflow Overhead:** DAG management complexity for large pipelines
- ❌ **Delta on S3 Slower:** Compared to Databricks-optimized Delta
- ❌ **Manual Optimization:** Requires tuning (Z-order, partitioning)

### Mitigation
- **Airflow Complexity:** Modular DAGs, comprehensive monitoring
- **Performance:** Optimize partitioning, use Athena caching, Z-order weekly
- **Expertise:** Team training on Delta Lake best practices

## Related Decisions
- **[ADR-08](ADR_08_SCHEDULER_FRAMEWORK.md):** Scheduler Framework - Airflow orchestrates lakehouse ETL
- **[ADR-15](ADR_15_MLOps_Pipeline.md):** MLOps Pipeline - Feature Store sources from Silver/Gold
- **[ADR-06](ADR_06_EVENT_DRIVEN_ARCHITECTURE.md):** Event-Driven Architecture - Kafka feeds Bronze layer

## References
- [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- [Delta Lake Documentation](https://docs.delta.io/)
- [AWS Lake Formation](https://aws.amazon.com/lake-formation/)
