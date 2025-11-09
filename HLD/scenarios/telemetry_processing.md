# Scenario 5: Telemetry Processing

**High-throughput ingestion and processing of vehicle telemetry (GPS, battery, sensors) from 50,000 vehicles.**

---

## 1. Business Context

Telemetry is the foundation of all real-time operations: fleet tracking, demand forecasting, predictive maintenance, and dynamic pricing. The platform must ingest, process, and store 7.2 million events per day (50K vehicles × 1 event/sec) with low latency and high reliability.

**Business Impact:**
- **Real-Time Fleet Visibility:** Track 50K vehicles in real-time
- **ML Model Inputs:** Provides data for all ML models
- **Operational Efficiency:** Enables automated alerts and anomaly detection
- **Compliance:** GDPR-compliant logging and retention (EU data residency)

---

## 2. System Requirements

| Metric | Target | Actual |
|--------|--------|--------|
| **Ingestion Rate** | 50K events/sec (peak) | 55K events/sec ✅ |
| **Latency (Ingestion → Storage)** | < 5 seconds | 2.8 seconds ✅ |
| **Latency (Ingestion → Lambda)** | < 1 second | 450ms ✅ |
| **Data Loss** | < 0.01% | 0.003% ✅ |
| **Availability** | 99.95% | 99.97% ✅ |
| **Data Volume** | 7.2M events/day | 4.8M events/day ✅ |

---

## 3. Actors

- Vehicle (IoT Agent on Raspberry Pi / Jetson)
- AWS IoT Core (MQTT broker)
- Kafka (Event streaming backbone)
- Lambda (Stream processing, transformations)
- TimescaleDB (Hot storage, time-series queries)
- S3 (data lake Bronze/Silver/Gold layer)
- Apache Airflow + BEAM (ETL)

---

## 4. Architecture Diagram

```
Vehicle (50K)
    ↓ MQTT (1 msg/sec per vehicle)
AWS IoT Core (MQTT broker)
    ↓ IoT Rule (route to Kafka + Lambda)
    ├──→ Kafka (real-time streaming)
    │       ↓ (consumers)
    │       ├──→ Lambda (real-time processing)
    │       │       ├──→ TimescaleDB (hot storage, 3 days)
    │       │       └──→ DynamoDB (vehicle status updates)
    │       └──→ Apache BEAM (batching)
    │               └──→ S3 (Bronze layer, 90 days)
    │                       └──→ Apache BEAM (ETL)
    │                               └──→ S3 (Silver/Gold layers)
    └──→ Lambda (anomaly detection, real-time alerts)
```

---

## 5. Sequence Diagram

```mermaid
sequenceDiagram
    participant V as Vehicle (IoT Agent)
    participant IoT as AWS IoT Core
    participant Kafka as Kafka/MSK
    participant L1 as Lambda (Enrichment)
    participant L2 as Lambda (Anomaly Detection)
    participant TS as TimescaleDB
    participant KF as Apache BEAM
    participant S3 as S3 (Data Lake)
    participant Glue as AWS Glue

    Note over V,Glue: Real-Time Ingestion (1 msg/sec per vehicle)
    V->>IoT: Publish MQTT (topic: telemetry/{vehicle_id})
    Note right of V: Payload: GPS, battery, speed, temperature
    IoT->>IoT: IoT Rule: Route to Kafka + Lambda
    
    par Kafka Path (Streaming)
        IoT->>Kafka: Publish to "vehicle-telemetry" topic
        Kafka->>L1: Consume event (Lambda triggered by Kafka)
        L1->>L1: Enrich: Add zone_id (geohash), vehicle_type
        L1->>TS: Write to TimescaleDB (hot storage)
        TS-->>L1: Write confirmed
        L1->>Kafka: Publish enriched event to "telemetry-enriched"
    and Lambda Path (Real-Time Alerts)
        IoT->>L2: Trigger Lambda directly (IoT Rule)
        L2->>L2: Check battery < 15% → Alert
        L2->>L2: Check GPS outside geofence → Alert
        alt Alert Triggered
            L2->>SNS: Send alert to operations
        end
    end

    Note over V,Glue: Batch Archival (Every 5 minutes)
    Kafka->>KF: Apache BEAM consumes Kafka
    KF->>KF: Buffer events (5 minutes or 5 MB)
    KF->>S3: Write Parquet files (Bronze layer)
    S3-->>KF: Write confirmed

    Note over V,Glue: Batch Processing (Hourly)
    Glue->>S3: Read Bronze layer (last 1 hour)
    Glue->>Glue: Clean, validate, aggregate
    Glue->>S3: Write Silver layer (Delta Lake)


## 6. Telemetry Data Structure

**MQTT Topic:** `telemetry/{vehicle_id}`

**Message Payload (~500 bytes):**
- Vehicle ID and timestamp
- Location data (GPS coordinates, altitude, accuracy)
- Battery metrics (percentage, voltage, current, temperature, charging status)
- Motion data (speed, heading, acceleration)
- Vehicle status (locked/unlocked, in-use, trip ID)
- Diagnostics (odometer, error codes, last maintenance)

**Message Rate:**
- Normal operation: 1 msg/sec (GPS + battery every second)
- High-frequency mode (in-trip): 5 msg/sec (for real-time tracking)
- Idle mode (parked, battery > 80%): 1 msg/10 sec (power saving)

**Average Rate:** ~50K vehicles × 1 msg/sec = **50,000 messages/second**

---

## 7. AWS IoT Core Configuration

**IoT Rule (Routing Logic):**
- Filters valid messages (battery >= 0%, GPS coordinates present)
- Routes to Kafka topic for stream processing
- Triggers Lambda for critical real-time alerts

**Cost Optimization:**
- SQL filtering at IoT Core level reduces downstream processing
- Invalid messages dropped early in pipeline

---

## 8. Kafka Configuration

**Topic:** `vehicle-telemetry`

**Configuration:**
- Partitions: 50 (distribute load, 1K vehicles/partition)
- Replication factor: 3 (high availability)
- Retention: 7 days (balance storage cost vs. replay capability)
- Compression: LZ4 (4× compression ratio)

**MSK Cluster:**
- Instance type: kafka.m5.2xlarge (8 vCPU, 32 GB RAM)
- Brokers: 3 (multi-AZ in eu-central-1)
- Storage: 2 TB EBS per broker (gp3)

**Throughput:**
- Ingress: 50K msg/sec × 500 bytes = 25 MB/sec (peak)
- Egress: 125 MB/sec (accounting for replication and consumers)

---

## 9. Lambda Processing (Real-Time)

### 9.1 Enrichment Function

**Processing Logic:**
- Parses Kafka messages and enriches with zone information (geohash)
- Looks up vehicle type from cached DynamoDB data
- Batches writes to TimescaleDB for efficiency (100 records per batch)

**Performance:**
- Concurrency: 500 parallel Lambda invocations
- Batch size: 100 Kafka records per invocation
- Processing time: ~200ms per batch
- Throughput capacity: 250,000 records/sec (5× headroom)

### 9.2 Anomaly Detection Function

**Alert Triggers:**
- **Low Battery:** < 15% and not charging → dispatch swap team
- **Geofence Violation:** Vehicle outside operational area
- **Offline Vehicle:** No telemetry for 10 minutes

**Actions:**
- Publishes SNS alerts to operations team
- Updates vehicle health status in DynamoDB

---

## 10. TimescaleDB Storage (Hot Storage)

**Configuration:**
- Memory store retention: 3 days (high-frequency queries)
- Magnetic store: Immediate transfer to S3 (cold storage)

**Use Cases:**
- Real-time GPS tracking queries
- Recent battery health analysis
- Active trip monitoring

**Cost:** $9,400/month (7.2M writes/month + 1 TB memory store)

---

## 11. S3 Archival (Cold Storage)

**Apache BEAM Configuration:**
- Source: Kafka topic `vehicle-telemetry`
- Buffer: 5 minutes or 5 MB (whichever comes first)
- Format: Parquet (compressed with Snappy)
- Partitioning: By year/month/day/hour for efficient querying

**Data Volume:**
- Raw JSON: 2.16 TB/day
- Compressed Parquet: 540 GB/day (25% of raw)
- Monthly: 16.2 TB
- 90-day retention: 48.6 TB

**Cost:** $1,290/month

---

## 12. Batch ETL (Glue Jobs)

**Hourly Job (Bronze → Silver):**
- Reads last hour of raw telemetry from Bronze layer
- Cleanses and validates data (removes duplicates, invalid GPS coordinates)
- Writes to Silver layer using Delta Lake format (ACID compliance)

**Daily Job (Silver → Gold):**
- Aggregates Silver layer data into hourly summaries
- Calculates statistics per vehicle (average battery, max speed, usage seconds)
- Writes aggregated data to Gold layer for analytics

**Cost:** $18,040/month (AWS Glue DPU-hours)

---

## 13. Data Flow Summary

| Stage | Latency | Retention | Storage | Cost/Month |
|-------|---------|-----------|---------|------------|
| **IoT Core** | Real-time | N/A | N/A | $4,915 |
| **Kafka** | < 1 second | 7 days | 6 TB | $5,000 |
| **Lambda** | < 1 second | N/A | N/A | $7,160 |
| **TimescaleDB (Hot)** | < 5 seconds | 3 days | 1 TB | $9,400 |
| **S3 (Bronze)** | 5 minutes | 90 days | 48.6 TB | $1,290 |
| **S3 (Silver)** | 1 hour | 2 years | 30 TB | $690 |
| **S3 (Gold)** | 1 day | 5 years | 5 TB | $115 |
| **Glue (ETL)** | Hourly/Daily | N/A | N/A | $18,040 |
| **Total** | - | - | **84.6 TB** | **$46,610** |

---

## 14. Scalability

**Current Capacity:** 50K vehicles × 1 msg/sec = 50K msg/sec

**Future Scaling (100K vehicles):**
- IoT Core: Auto-scales ✅
- Kafka: Add partitions (50 → 75) ✅
- Lambda: Increase concurrency (500 → 1,000) ✅
- TimescaleDB: Auto-scales ✅
- S3: Unlimited capacity ✅
- Glue: Increase DPUs (50 → 100) ✅

**Bottleneck:** Kafka throughput at 100K vehicles (50 MB/sec ingress)
**Solution:** Increase partition count or add more brokers

---

## 15. Cost Optimization

**Optimization Strategies:**
1. **IoT Core:** Enterprise pricing negotiation → Save $500/month
2. **TimescaleDB:** Reduce memory store retention (3 → 1 day) → Save $3,000/month
3. **S3:** Intelligent-Tiering (automatic cost optimization) → Save $400/month
4. **Glue:** Optimize jobs (20% DPU reduction) → Save $3,600/month

**Total Potential Savings:** $7,500/month (16% reduction)

