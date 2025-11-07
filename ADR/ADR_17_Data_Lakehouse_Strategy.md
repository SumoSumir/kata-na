# ADR-17: Data Lakehouse Strategy with Medallion Architecture

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
- **Hot Data:** Last 7 days, sub-second query latency
- **Warm Data:** Last 90 days, query latency < 5 seconds
- **Cold Data:** 90+ days, query latency < 30 seconds
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
│  │ Retention: 90 days, then archive to Glacier                  │  │
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
- ✅ Durability: 99.999999999% (11 nines)
- ✅ Cost: $0.023/GB/month (Standard), $0.004/GB (Glacier)
- ✅ Scalability: Unlimited storage
- ✅ Integration: Native with all AWS services
- ✅ Lifecycle Policies: Auto-tiering to save costs
- ✅ Versioning: Data recovery and audit trails

**Alternatives Considered:**
- HDFS: Too complex to manage
- EBS: Too expensive for large data
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
- ✅ Cost: Pay only for execution time
- ✅ DPU Flex: Optimize for cost or performance

**Alternatives Considered:**
- EMR: More control but higher management overhead
- Databricks: More expensive ($40K/month vs $8K)
- Lambda: Limited for large transformations

#### 4. Query Engine: Athena + Redshift Spectrum
**Why Athena:**
- ✅ Serverless SQL: No infrastructure
- ✅ Standard SQL: Easy for analysts
- ✅ Delta Lake Support: Query Delta tables directly
- ✅ Cost: $5/TB scanned (with partitioning = pennies)

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

```python
# Example: Vehicle Telemetry to Bronze Layer

# 1. Kafka (MSK) → Kinesis Firehose → S3
{
  "delivery_stream": "vehicle-telemetry-bronze",
  "s3_configuration": {
    "bucket_arn": "arn:aws:s3:::mobilityco rp-datalake-bronze",
    "prefix": "telemetry/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/",
    "buffering_hints": {
      "size_in_mbs": 128,  # Flush every 128 MB
      "interval_in_seconds": 60  # Or every 60 seconds
    },
    "compression_format": "SNAPPY",
    "data_format_conversion": {
      "enabled": true,
      "output_format": "PARQUET",
      "schema": {
        "database": "mobilit ycorp",
        "table": "telemetry_raw"
      }
    }
  }
}

# 2. Partitioning Strategy
s3://mobilitycorp-datalake-bronze/
├─ telemetry/
│  ├─ year=2025/
│  │  ├─ month=11/
│  │  │  ├─ day=07/
│  │  │  │  ├─ hour=00/
│  │  │  │  │  ├─ part-00000.snappy.parquet
│  │  │  │  │  └─ part-00001.snappy.parquet
│  │  │  │  └─ hour=01/
│  │  │  └─ day=08/
│  │  └─ month=12/
│  └─ year=2026/
├─ bookings/
├─ user_events/
└─ external_data/
```

#### Silver Layer: Data Quality & Transformation

```python
# Example: Bronze → Silver Transformation (AWS Glue PySpark)

from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from delta.tables import *
import great_expectations as gx

# Initialize Spark with Delta Lake
spark = SparkSession.builder \
    .appName("Bronze to Silver - Telemetry") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read from Bronze (last hour of data)
bronze_path = "s3://mobilitycorp-datalake-bronze/telemetry/"
df_bronze = spark.read.parquet(bronze_path) \
    .where(col("ingestion_time") >= current_timestamp() - expr("INTERVAL 1 HOUR"))

# Data Quality Checks (Great Expectations)
context = gx.get_context()
datasource = context.sources.add_or_update_pandas(name="telemetry_bronze")
batch_request = datasource.add_dataframe_asset(name="bronze_df")
batch = batch_request.build_batch_request(dataframe=df_bronze.toPandas())

# Validation Suite
expectations = [
    gx.expectations.ExpectColumnValuesToNotBeNull(column="vehicle_id"),
    gx.expectations.ExpectColumnValuesToBeInSet(column="vehicle_type", value_set=["scooter", "ebike", "car", "van"]),
    gx.expectations.ExpectColumnValuesToBeBetween(column="battery_level", min_value=0, max_value=100),
    gx.expectations.ExpectColumnValuesToMatchRegex(column="vehicle_id", regex=r"^[A-Z]{2}\d{6}$")
]

validation_result = context.run_checkpoint(checkpoint_name="telemetry_quality", batch_request=batch)

if not validation_result.success:
    # Log failed records to quarantine
    df_bronze.where(~validation_result.passed_mask) \
        .write.mode("append") \
        .parquet("s3://mobilitycorp-datalake-quarantine/telemetry/")
    
    # Continue with valid records only
    df_bronze = df_bronze.where(validation_result.passed_mask)

# Transformations
df_silver = df_bronze \
    .withColumn("event_timestamp", to_timestamp(col("timestamp"))) \
    .withColumn("date", to_date(col("event_timestamp"))) \
    .withColumn("hour", hour(col("event_timestamp"))) \
    .withColumn("battery_status", 
        when(col("battery_level") < 10, "critical")
        .when(col("battery_level") < 30, "low")
        .when(col("battery_level") < 70, "medium")
        .otherwise("good")) \
    .dropDuplicates(["vehicle_id", "timestamp"]) \
    .withColumn("ingestion_date", current_date()) \
    .withColumn("processing_timestamp", current_timestamp())

# Add geofence enrichment
df_geofences = spark.read.format("delta").load("s3://mobilitycorp-datalake-silver/geofences/")
df_silver = df_silver.join(
    df_geofences,
    expr("ST_Contains(df_geofences.polygon, ST_Point(df_silver.longitude, df_silver.latitude))"),
    "left"
).select(
    df_silver["*"],
    df_geofences["zone_id"],
    df_geofences["zone_name"]
)

# Write to Silver (Delta Lake with ACID)
silver_path = "s3://mobilitycorp-datalake-silver/telemetry/"

df_silver.write \
    .format("delta") \
    .mode("append") \
    .partitionBy("date", "vehicle_type") \
    .option("mergeSchema", "true") \
    .option("overwriteSchema", "false") \
    .save(silver_path)

# Optimize Delta Table (weekly job)
deltaTable = DeltaTable.forPath(spark, silver_path)
deltaTable.optimize() \
    .where(f"date >= current_date() - 7") \
    .executeZOrderBy("vehicle_id", "event_timestamp")

# Vacuum old files (retention = 30 days)
deltaTable.vacuum(retentionHours=720)  # 30 days
```

#### Gold Layer: Business Aggregations

```python
# Example: Silver → Gold Aggregation (demand_by_zone_hour)

from pyspark.sql.window import Window

# Read from Silver
df_bookings = spark.read.format("delta") \
    .load("s3://mobilitycorp-datalake-silver/bookings/") \
    .where(col("date") >= current_date() - 90)  # Last 90 days

df_telemetry = spark.read.format("delta") \
    .load("s3://mobilitycorp-datalake-silver/telemetry/") \
    .where(col("date") >= current_date() - 90)

# Aggregate demand by zone, vehicle_type, hour
df_demand = df_bookings \
    .withColumn("hour", hour(col("booking_timestamp"))) \
    .groupBy("zone_id", "vehicle_type", "date", "hour") \
    .agg(
        count("*").alias("booking_count"),
        countDistinct("user_id").alias("unique_users"),
        avg("trip_duration_minutes").alias("avg_trip_duration"),
        sum("revenue").alias("total_revenue")
    )

# Add supply metrics from telemetry
df_supply = df_telemetry \
    .groupBy("zone_id", "vehicle_type", "date", "hour") \
    .agg(
        countDistinct("vehicle_id").alias("available_vehicles"),
        avg("battery_level").alias("avg_battery_level")
    )

# Join demand + supply
df_gold = df_demand.join(
    df_supply,
    on=["zone_id", "vehicle_type", "date", "hour"],
    how="outer"
).fillna(0)

# Add rolling averages (7-day, 28-day)
window_7d = Window.partitionBy("zone_id", "vehicle_type", "hour") \
    .orderBy("date") \
    .rowsBetween(-6, 0)

window_28d = Window.partitionBy("zone_id", "vehicle_type", "hour") \
    .orderBy("date") \
    .rowsBetween(-27, 0)

df_gold = df_gold \
    .withColumn("booking_count_7d_avg", avg("booking_count").over(window_7d)) \
    .withColumn("booking_count_28d_avg", avg("booking_count").over(window_28d)) \
    .withColumn("utilization_rate", 
        (col("booking_count") / col("available_vehicles")) * 100)

# Write to Gold (Delta Lake + Redshift)
gold_path = "s3://mobilitycorp-datalake-gold/demand_by_zone_hour/"

df_gold.write \
    .format("delta") \
    .mode("overwrite") \
    .partitionBy("date") \
    .option("replaceWhere", f"date >= '{(current_date() - 90).isoformat()}'") \
    .save(gold_path)

# Also sync to Redshift for BI tools
df_gold.write \
    .format("jdbc") \
    .option("url", "jdbc:redshift://mobilit ycorp-redshift.xxx.eu-central-1.redshift.amazonaws.com:5439/analytics") \
    .option("dbtable", "gold.demand_by_zone_hour") \
    .option("user", get_secret("redshift_user")) \
    .option("password", get_secret("redshift_password")) \
    .option("driver", "com.amazon.redshift.jdbc.Driver") \
    .mode("overwrite") \
    .save()
```

### Cost Analysis

#### Storage Costs (Monthly, Year 1)

```
Bronze Layer (Parquet, 90-day retention):
  - 5TB/month × 3 months = 15TB
  - S3 Standard: 15TB × $0.023/GB = $345
  - After 90 days → Glacier: $0.004/GB = $60

Silver Layer (Delta Lake, 2-year retention):
  - 3TB/month × 24 months = 72TB (deduped, cleaned)
  - S3 Standard (last 90 days): 9TB × $0.023 = $207
  - S3 IA (90d-2yr): 63TB × $0.0125 = $788

Gold Layer (Aggregated, 5-year retention):
  - 500GB/month × 12 months = 6TB
  - S3 Standard: 6TB × $0.023 = $138

Feature Store (SageMaker):
  - Offline: 2TB × $0.023 = $46
  - Online (DynamoDB): 100M requests/month = $125

Total Storage: $1,709/month
```

#### Processing Costs (Monthly)

```
Glue ETL Jobs:
  - Bronze → Silver: 20 DPU × 2 hrs/day × 30 days = 1,200 DPU-hours
  - Silver → Gold: 10 DPU × 1 hr/day × 30 days = 300 DPU-hours
  - Total: 1,500 DPU-hours × $0.44/DPU-hour = $660

Athena Queries:
  - 10TB scanned/month × $5/TB = $50

Redshift Spectrum:
  - 5TB scanned/month × $5/TB = $25

Total Processing: $735/month
```

#### Grand Total: $2,444/month (~$29K/year)

Compare to:
- Databricks Lakehouse: ~$8,000/month
- Snowflake Data Cloud: ~$6,000/month

**AWS is 60-70% cheaper for our use case.**

### Data Governance

#### Access Control (Lake Formation)

```
Permissions Matrix:

Role: data_scientists
  - Silver layer: SELECT (all tables)
  - Gold layer: SELECT (all tables)
  - Bronze layer: DENIED (raw PII)

Role: ml_engineers
  - Silver layer: SELECT, DESCRIBE
  - Gold layer: SELECT, DESCRIBE
  - Feature Store: SELECT, INSERT

Role: analysts
  - Gold layer: SELECT (via Redshift)
  - Silver layer: DENIED

Role: operations
  - Silver layer: SELECT (telemetry, bookings)
  - Gold layer: SELECT
  
Role: admins
  - All layers: ALL permissions
  - Audit logs: SELECT

Column-Level Security:
  - PII columns (email, phone): Masked for non-admins
  - Payment info: Encrypted, access logged
```

#### Data Lineage (AWS Glue Data Catalog + Lake Formation)

```
Tracking:
  - Source → Bronze: Kafka topic, ingestion timestamp
  - Bronze → Silver: Glue job ID, transformation logic
  - Silver → Gold: Aggregation logic, dependencies
  - Gold → Consumption: Query ID, user, timestamp

Example Lineage Query:
"Where did this demand_by_zone_hour record come from?"

Answer:
  demand_by_zone_hour (Gold)
    ← bookings (Silver) 
      ← booking_events (Bronze)
        ← kafka.bookings-topic
          ← Booking Service (DynamoDB Stream)
    ← telemetry (Silver)
      ← telemetry_events (Bronze)
        ← kafka.telemetry-topic
          ← IoT Core (MQTT)
```

#### GDPR Compliance

```
Right to Erasure:
1. User requests deletion
2. Lambda triggers deletion workflow:
   - Mark records in Silver/Gold with tombstone
   - Overwrite PII with NULL/anonymized values
   - Log deletion event
3. Next vacuum job removes physical files

Data Minimization:
  - Collect only necessary telemetry fields
  - Drop unused columns in Silver layer
  - Aggregate/anonymize in Gold layer

Data Retention:
  - Bronze: 90 days → Glacier
  - Silver: 2 years → Delete
  - Gold: 5 years (aggregated, anonymized)
  - Audit logs: 7 years
```

## Consequences

### Positive
✅ **Cost-Effective:** 60-70% cheaper than Databricks/Snowflake
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
- **ADR-15:** Cloud Provider Selection (AWS) - Enables S3/Glue choice
- **ADR-16:** MLOps Pipeline - Feature Store built on Silver/Gold data
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
