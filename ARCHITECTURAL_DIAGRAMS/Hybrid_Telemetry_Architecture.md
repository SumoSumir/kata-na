# Hybrid Telemetry Architecture

This diagram details the hybrid edge-cloud architecture for processing vehicle telemetry, focusing on how data is collected, processed at the edge, and ingested into the cloud.

```
┌────────────────────────────────────────────────────────────────────────┐
│              EDGE-CLOUD HYBRID TELEMETRY ARCHITECTURE                  │
└────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                        VEHICLE EDGE LAYER                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌────────────────────────────────────────────────────┐            │
│  │  IoT Hardware (ARM Processor + Sensors)            │            │
│  │  • GPS/GNSS Module                                 │            │
│  │  • IMU/Accelerometer (collision detection)         │            │
│  │  • Battery Management System (BMS)                 │            │
│  │  • Camera (pickup/dropoff verification)            │            │
│  │  • NFC/Bluetooth (unlock/lock)                     │            │
│  │  • 4G/5G Modem (connectivity)                      │            │
│  └────────────────────────────────────────────────────┘            │
│                         │                                          │
│                         ▼                                          │
│  ┌────────────────────────────────────────────────────┐            │
│  │  Edge Processing Module (Lightweight ML)           │            │
│  │                                                    │            │
│  │  Collision Detection Model:                       │            │
│  │  • Input: IMU data (50Hz), wheel slip, ABS        │            │
│  │  • Model: Quantized Neural Network (<5MB)         │            │
│  │  • Latency: <100ms                                │            │
│  │  • Fallback: Rule-based (g-force >4G)             │            │
│  │  • Action: Immediate alert + disable vehicle      │            │
│  │                                                    │            │
│  │  Damage Detection Model:                          │            │
│  │  • Input: Pickup/dropoff photos                   │            │
│  │  • Model: MobileNetV3 (<10MB)                     │            │
│  │  • Privacy: Generate blurred thumbnails           │            │
│  │  • Output: Damage classification + confidence     │            │
│  │  • Upload: Only metadata + embeddings (not raw)   │            │
│  │                                                    │            │
│  │  Telemetry Aggregation:                           │            │
│  │  • GPS: 30s intervals → compressed batches        │            │
│  │  • Battery: 60s intervals → voltage curves        │            │
│  │  • Vibration: FFT analysis → anomaly scores       │            │
│  │  • Compression: Protocol Buffers + gzip           │            │
│  └────────────────────────────────────────────────────┘            │
│                         │                                          │
│                         ▼                                          │
│  ┌────────────────────────────────────────────────────┐            │
│  │  Secure Communication (mTLS)                       │            │
│  │  • Device certificates for authentication          │            │
│  │  • End-to-end encryption (TLS 1.3)                 │            │
│  │  • Retry logic with exponential backoff            │            │
│  │  • Offline buffering (up to 24h)                   │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                     MQTT/HTTPS over 4G/5G
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        CLOUD INGESTION LAYER                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌────────────────────────────────────────────────────┐            │
│  │  IoT Gateway (AWS IoT Core / Azure IoT Hub)        │            │
│  │  • Device authentication (mTLS)                    │            │
│  │  • Message validation and routing                  │            │
│  │  • Protocol translation (MQTT → Kafka)             │            │
│  │  • Device shadow state management                  │            │
│  └────────────────────────────────────────────────────┘            │
│                         │                                          │
│                         ▼                                          │
│  ┌────────────────────────────────────────────────────┐            │
│  │  Kafka Topics (Event Bus)                          │            │
│  │  • vehicles.telemetry (GPS, battery)               │            │
│  │  • vehicles.collision_detected                     │            │
│  │  • photos.uploaded (damage verification)           │            │
│  │  • vehicles.health (predictive maintenance)        │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CLOUD PROCESSING LAYER                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Stream Processing (Kafka Streams):                                │
│  ┌────────────────────────────────────────────────────┐            │
│  │  • Real-time aggregations (fleet status)           │            │
│  │  • Anomaly detection (battery drain, vibration)    │            │
│  │  • Geofencing alerts (wrong location)              │            │
│  │  • Trip reconstruction from GPS points             │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
│  Batch Processing (Airflow):                                       │
│  ┌────────────────────────────────────────────────────┐            │
│  │  • Daily: Historical telemetry ETL to S3           │            │
│  │  • Weekly: Feature extraction for ML training      │            │
│  │  • Monthly: Vehicle health reports                 │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
│  Data Storage:                                                     │
│  ┌────────────────────────────────────────────────────┐            │
│  │  TimescaleDB (Time-Series):                        │            │
│  │  • GPS coordinates, battery voltage curves         │            │
│  │  • Retention: 90 days hot, 2 years cold (S3)       │            │
│  │                                                    │            │
│  │  S3 (Object Storage):                              │            │
│  │  • Raw telemetry archives (compliance)             │            │
│  │  • Anonymized datasets (ML training)               │            │
│  │  • Damage photos (encrypted, 7-day TTL)            │            │
│  │                                                    │            │
│  │  PostgreSQL (Relational):                          │            │
│  │  • Vehicle metadata, trip summaries                │            │
│  │  • Maintenance records, incident logs              │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```
