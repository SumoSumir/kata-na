# ADR-16: Data Lakehouse Strategy (Medallion Architecture)

## Status
Accepted

## Context
MobilityCorp's AI-enabled platform generates massive amounts of data from multiple sources:
- **Vehicle Telemetry:** 50K vehicles × 1 message/sec = 4.3 billion events/day
- **Bookings:** ~500K transactions/day
- **User Events:** ~2M interactions/day
- **External Data:** Weather, events, traffic (continuous streams)
- **Feedback & Images:** ~100K photos/day

This data must support:
1. Real-time operational queries (vehicle status, availability)
2. Historical analytics (demand patterns, utilization)
3. ML model training (supervised learning with labels)
4. Business intelligence (dashboards, reports)
5. Compliance & audit (GDPR, data retention)

## Requirements

### Data Types & Volumes
- **Hot Data:** Recent data, sub-second query latency
- **Warm Data:** Recent historical data, fast query latency
- **Cold Data:** Historical archive, acceptable query latency
- **Total Volume:** ~5TB/month new data, 60TB total after 1 year

### Access Patterns
- **Real-time:** Vehicle location, availability checks
- **Batch:** Daily demand forecasting, weekly reports
- **Ad-hoc:** Data science exploration, debugging
- **Streaming:** Live dashboards, alerting

### Governance
- **Data Quality:** Validation, schema evolution
- **Access Control:** Role-based, column-level security
- **Audit:** Who accessed what data when
- **Lineage:** Track data flow from source to consumption
- **GDPR:** Right to erasure, data minimization

## Decision

**We adopt a Medallion Architecture (Bronze/Silver/Gold) on AWS using S3 + Delta Lake format.**

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DATA LAKEHOUSE ARCHITECTURE                      │
│                   (Medallion: Bronze → Silver → Gold)                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ BRONZE LAYER (Raw Data - Append Only)                        │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Format: Parquet (compressed with Snappy)                     │  │
│  │ Schema: Schema-on-read, minimal transformations              │  │
│  │ Partitioning: /year/month/day/hour/                          │  │
│  │ Retention: Configurable period, then archive to Glacier       │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Sources:                                                      │  │
│  │  ├─ MSK (Kafka) → Kinesis Firehose → S3                     │  │
│  │  ├─ DynamoDB Streams → Lambda → S3                          │  │
│  │  ├─ Aurora → Glue ETL → S3                                  │  │
│  │  └─ External APIs → Lambda → S3                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                             ↓                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ SILVER LAYER (Cleaned & Validated Data)                     │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Format: Delta Lake (ACID transactions)                       │  │
│  │ Schema: Strongly typed, validated, deduplicated             │  │
│  │ Partitioning: By date + vehicle_type/zone_id                │  │
│  │ Retention: 2 years, then archive                            │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Transformations:                                             │  │
│  │  ├─ Data Quality checks (Great Expectations)                │  │
│  │  ├─ Schema enforcement & evolution                          │  │
│  │  ├─ Deduplication (based on event_id + timestamp)           │  │
│  │  ├─ Type casting & normalization                            │  │
│  │  ├─ PII masking (emails, phone numbers)                     │  │
│  │  └─ Enrichment (geofencing, reverse geocoding)              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                             ↓                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ GOLD LAYER (Business-Ready Aggregations)                    │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Format: Delta Lake + Redshift                               │  │
│  │ Schema: Star schema (facts + dimensions)                     │  │
│  │ Optimization: Z-order clustering, data skipping             │  │
│  │ Retention: 5 years (compliance)                              │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │ Aggregations:                                                │  │
│  │  ├─ demand_by_zone_hour (forecasting)                       │  │
│  │  ├─ vehicle_utilization_daily (operations)                  │  │
│  │  ├─ revenue_by_vehicle_type_daily (BI)                      │  │
│  │  ├─ battery_health_metrics_weekly (maintenance)             │  │
│  │  └─ customer_journey_events (personalization)               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                             ↓                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ CONSUMPTION LAYER                                            │  │
│  ├──────────────────────────────────────────────────────────────┤  │
│  │  ├─ SageMaker Feature Store (ML features)                   │  │
│  │  ├─ Redshift (BI queries via Looker)                        │  │
│  │  ├─ Athena (ad-hoc SQL queries)                             │  │
│  │  ├─ QuickSight (executive dashboards)                       │  │
│  │  └─ S3 Select (fast key-value lookups)                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Technology Choices

#### 1. Storage: Amazon S3
**Why S3:**
- ✅ Durability: 11 nines (S3 standard)
- ✅ Cost-effective tiered storage with lifecycle policies
- ✅ Scalability: Unlimited storage capacity
- ✅ Integration: Native with all AWS services
- ✅ Lifecycle Policies: Auto-tiering to save costs
- ✅ Versioning: Data recovery and audit trails

**Alternatives Considered:**
- HDFS: Too complex to manage
- EBS: Not optimized for data lake scale
- EFS: Not optimized for data lake workloads

#### 2. Table Format: Delta Lake
**Why Delta Lake:**
- ✅ ACID Transactions: Atomic writes, no partial updates
- ✅ Time Travel: Query data as of specific timestamp
- ✅ Schema Evolution: Add/modify columns safely
- ✅ Unified Batch/Streaming: One format for both
- ✅ Performance: Z-order clustering, data skipping
- ✅ Open Format: Not vendor-locked

**Alternatives Considered:**
- Apache Iceberg: Excellent, but Delta Lake has better AWS integration
- Apache Hudi: More complex, less mature
- Parquet alone: No ACID, no time travel

#### 3. Processing: AWS Glue + PySpark
**Why Glue:**
- ✅ Serverless: No cluster management
- ✅ Auto-Scaling: Based on job requirements
- ✅ Catalog: Central metadata repository
- ✅ Pay-per-use pricing model
- ✅ DPU Flex: Optimize for cost or performance

**Alternatives Considered:**
- EMR: More control but higher management overhead
- Databricks: Higher operational costs
- Lambda: Limited for large transformations

#### 4. Query Engine: Athena + Redshift Spectrum
**Why Athena:**
- ✅ Serverless SQL: No infrastructure
- ✅ Standard SQL: Easy for analysts
- ✅ Delta Lake Support: Query Delta tables directly
- ✅ Pay-per-query with partition optimization

**Why Redshift Spectrum:**
- ✅ Joins: Combine S3 data with Redshift tables
- ✅ Performance: Faster for complex queries
- ✅ BI Integration: Looker, Tableau, QuickSight

#### 5. Orchestration: AWS Step Functions + EventBridge
**Why Step Functions:**
- ✅ Visual Workflows: Ea sy to understand
- ✅ Error Handling: Retries, catch, fallback
- ✅ Integration: Native with Glue, Lambda, SageMaker
- ✅ Serverless: No infrastructure

**Alternatives Considered:**
- Airflow (MWAA): More powerful but overkill, higher cost
- Prefect/Dagster: Not AWS-native

### Data Flow Details

#### Bronze Layer: Raw Data Ingestion

**Kafka to S3 Pipeline:**
- Kinesis Firehose buffers streaming data with configurable size and time thresholds
- Automatic conversion to Snappy-compressed Parquet format
- Partitioned by timestamp (year/month/day/hour) for efficient querying

**Partitioning Structure:**
```
s3://mobilitycorp-datalake-bronze/
├─ telemetry/year=2025/month=11/day=07/hour=00/
├─ bookings/year=2025/month=11/day=07/
├─ user_events/year=2025/month=11/day=07/
└─ external_data/weather/year=2025/month=11/day=07/
```

#### Silver Layer: Data Quality & Transformation

**AWS Glue PySpark Processing:**
- Reads last hour of Bronze layer data
- Runs Great Expectations validation suite:
  - Null checks on critical fields (vehicle_id required)
  - Set validation (vehicle_type must be: scooter/ebike/car/van)
  - Range validation (battery percentage)
  - Format validation (vehicle ID regex pattern)
- Failed records quarantined for investigation
- Valid records transformed:
  - Timestamp parsing and date extraction
  - Status categorization (battery: critical/low/medium/good)
  - Deduplication by vehicle ID and timestamp
  - Geofence enrichment (zone assignment based on GPS coordinates)

**Delta Lake Storage:**
- ACID transactions ensure data consistency
- Automatic schema evolution when new fields added
- Partitioned by date and vehicle type for query optimization
- Z-ordering by vehicle_id and timestamp (weekly optimization)
- 30-day retention with automatic vacuum of old files

#### Gold Layer: Business Aggregations

**Demand Analysis Pipeline:**
- Aggregates bookings by zone, vehicle type, date, and hour
- Metrics: booking count, unique users, average trip duration, total revenue
- Joins with supply metrics (available vehicles, average battery level)
- Calculates rolling averages (7-day and 28-day windows)
- Computes utilization rate (bookings per available vehicle)
- Writes to Delta Lake and syncs to Redshift for BI tools

### Cost Considerations

**Storage Strategy:**
- Bronze Layer: Parquet format with lifecycle policies for archival
- Silver Layer: Delta Lake with tiered storage (hot/warm/cold)
- Gold Layer: Aggregated tables optimized for query performance
- Feature Store: Offline (S3) and Online (DynamoDB) storage

**Processing Approach:**
- Glue ETL Jobs: Bronze → Silver and Silver → Gold transformations
- Athena Queries: Ad-hoc analytics with partitioning optimization
- Redshift Spectrum: BI tool integration for federated queries

**Cost Optimization:**
- Lifecycle policies for automated data archival
- Partition pruning reduces scan costs
- Spot instances for non-critical ETL jobs
- Caching frequently accessed data

**Comparison with Alternatives:**
AWS-based lakehouse provides cost advantages over managed platforms like Databricks or Snowflake for this use case, particularly due to granular control over storage tiers and compute resources.

### Data Governance

#### Access Control (Lake Formation)

```
Role-Based Permissions:
- Data Scientists: Access to Silver and Gold layers
- ML Engineers: Access to Silver, Gold, and Feature Store
- Analysts: Access to Gold layer via Redshift
- Operations: Access to operational data in Silver/Gold
- Admins: Full access with audit logging

Security Features:
- Column-level security for PII (masked for non-admins)
- Encrypted payment information with access logging
- Row-level security based on data classification
```

#### Data Lineage (AWS Glue Data Catalog + Lake Formation)

```
Automatic Tracking:
- Source systems to Bronze layer ingestion
- Bronze to Silver transformations (Glue job IDs)
- Silver to Gold aggregations with dependencies
- Gold to consumption with query and user tracking

Enables traceability from final reports back to source systems
```

#### GDPR Compliance

```
Right to Erasure:
- Automated deletion workflow via Lambda
- Tombstone marking and data anonymization
- Physical file cleanup via vacuum jobs

Data Minimization & Retention:
- Collect only necessary telemetry fields
- Bronze: Configurable retention period → Glacier
- Silver: Moderate retention → Delete
- Gold: Long-term retention (aggregated, anonymized)
- Audit logs: Extended retention for compliance
```

## Consequences

### Positive
✅ **Cost-Effective:** Significantly cheaper than Databricks/Snowflake
✅ **Scalable:** Handles 5TB/month growth effortlessly
✅ **Flexible:** Batch, streaming, ad-hoc queries all supported
✅ **Governed:** Lake Formation provides fine-grained access control
✅ **ACID:** Delta Lake ensures data consistency
✅ **Open Format:** Not vendor-locked to AWS
✅ **Time Travel:** Query historical data easily
✅ **Performance:** Z-order clustering, partitioning, caching

### Negative
❌ **Complexity:** Medallion architecture has learning curve
❌ **Glue Limitations:** Less powerful than Databricks for complex ETL
❌ **Athena Query Limits:** Not ideal for heavy analytical workloads
❌ **Delta Lake on S3:** Slower than Delta Lake on Databricks
❌ **Operational Overhead:** More pieces to manage than unified platform

### Mitigation
- **Complexity:** Comprehensive documentation and training
- **Glue Limits:** Use EMR for complex transformations if needed
- **Athena Limits:** Redshift for heavy BI workloads
- **Performance:** Optimize partitioning, use caching
- **Overhead:** Automate with Terraform, monitoring with CloudWatch

## Related Decisions
- **ADR-15:** MLOps Pipeline - Feature Store built on Silver/Gold data
- **ADR-06:** Event-Driven Architecture - Feeds data into Bronze layer
- **ADR-03:** Vehicle Telemetry - Major data source

## References
- [Medallion Architecture Guide](https://www.databricks.com/glossary/medallion-architecture)
- [Delta Lake Documentation](https://docs.delta.io/)
- [AWS Lake Formation](https://aws.amazon.com/lake-formation/)
- [Data Lakehouse Book](https://www.oreilly.com/library/view/data-lakehouse/9781098141240/)

## Revision History
- **2025-11-07:** Initial decision - Medallion architecture with Delta Lake
- **Next Review:** 2026-02-07 (3 months) - After Phase 2 data platform build
