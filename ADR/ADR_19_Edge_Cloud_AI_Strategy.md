# ADR-19: Edge vs Cloud AI Compute Strategy

## Status
Accepted

## Context
MobilityCorp's fleet of 50,000 vehicles requires AI/ML capabilities for various use cases. Some decisions must be made in real-time on the vehicle (edge), while others can be processed in the cloud. We must determine:

1. Which ML models run on vehicles (edge)
2. Which models run in AWS (cloud)
3. Edge hardware requirements
4. Model deployment and OTA update strategy
5. Cost optimization (edge compute vs cloud API calls)

## Requirements

### Latency-Sensitive Operations (< 100ms)
- **Collision Detection:** Immediate braking/alert
- **Obstacle Avoidance:** Real-time steering adjustments (cars/vans only)
- **Geofence Violations:** Instant alerts when leaving operational zones
- **Tamper Detection:** Detect unauthorized access attempts

### Network-Dependent Operations
- **Demand Forecasting:** Requires historical data, cloud processing
- **Dynamic Pricing:** Needs real-time supply/demand across fleet
- **Predictive Maintenance:** Aggregate telemetry analysis
- **Route Optimization:** Traffic data, multi-vehicle coordination

### Offline Capability Requirements
- Vehicles must function when cellular connection is lost
- Critical safety features cannot depend on cloud
- Bookings can be queued and synced when reconnected

## Alternatives Considered

### Alternative 1: Edge-Cloud Hybrid (Recommended)

#### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  EDGE-CLOUD AI ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ON-VEHICLE (EDGE) - AWS IoT Greengrass v2                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ IoT Greengrass Core (Linux/ARM64 or x86_64)              │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │                                                           │  │
│  │  EDGE ML MODELS (Low-latency, Safety-Critical):          │  │
│  │                                                           │  │
│  │  1. Collision Detection (TensorFlow Lite)                │  │
│  │     ├─ Input: IMU (accelerometer/gyroscope) @ 100Hz      │  │
│  │     ├─ Model: CNN (compressed, quantized)                │  │
│  │     ├─ Output: Collision probability [0-1]               │  │
│  │     ├─ Latency: < 50ms                                    │  │
│  │     └─ Action: Trigger alert if > 0.8                    │  │
│  │                                                           │  │
│  │  2. Geofence Violation Detection (Rule-Based + ML)       │  │
│  │     ├─ Input: GPS coordinates @ 1Hz                      │  │
│  │     ├─ Model: Point-in-polygon + zone classifier         │  │
│  │     ├─ Output: In/out of zone                            │  │
│  │     ├─ Latency: < 100ms                                   │  │
│  │     └─ Action: Alert user + notify cloud                 │  │
│  │                                                           │  │
│  │  3. Tamper Detection (Anomaly Detection)                 │  │
│  │     ├─ Input: Voltage, door sensors, GPS delta           │  │
│  │     ├─ Model: Isolation Forest (scikit-learn)            │  │
│  │     ├─ Output: Anomaly score                             │  │
│  │     ├─ Latency: < 200ms                                   │  │
│  │     └─ Action: Lock vehicle + alert operations           │  │
│  │                                                           │  │
│  │  LOCAL CACHING:                                           │  │
│  │  ├─ Last 100 telemetry records (for offline queuing)     │  │
│  │  ├─ Operational zones (GeoJSON, ~1MB)                    │  │
│  │  └─ Vehicle configuration                                │  │
│  │                                                           │  │
│  │  COMPONENTS (Greengrass):                                │  │
│  │  ├─ ML Inference Component (Neo-compiled models)         │  │
│  │  ├─ Telemetry Publisher (MQTT to IoT Core)              │  │
│  │  ├─ OTA Update Manager                                   │  │
│  │  └─ Local Logger (for offline operation)                 │  │
│  │                                                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓ ↑ MQTT (Cellular)                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ AWS IoT Core (Frankfurt)                                 │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ Device Registry & Authentication                     │  │
│  │  ├─ Message Routing (Rules Engine)                       │  │
│  │  ├─ Device Shadow (desired/reported state)               │  │
│  │  └─ Fleet Provisioning                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ CLOUD ML MODELS (AWS SageMaker)                          │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │                                                           │  │
│  │  1. Demand Forecasting (LightGBM)                        │  │
│  │     ├─ Input: Historical bookings, weather, events       │  │
│  │     ├─ Frequency: Batch (daily at 2 AM)                  │  │
│  │     ├─ Output: Demand by zone/hour for next 48h          │  │
│  │     └─ Latency: N/A (batch)                              │  │
│  │                                                           │  │
│  │  2. Dynamic Pricing (XGBoost)                            │  │
│  │     ├─ Input: Current demand/supply, time, zone          │  │
│  │     ├─ Frequency: Real-time (on booking request)         │  │
│  │     ├─ Output: Optimal price per vehicle type            │  │
│  │     └─ Latency: < 200ms                                   │  │
│  │                                                           │  │
│  │  3. Battery Degradation Prediction (Random Forest)       │  │
│  │     ├─ Input: Historical telemetry (voltage, temp)       │  │
│  │     ├─ Frequency: Batch (weekly)                         │  │
│  │     ├─ Output: Remaining Useful Life (days)              │  │
│  │     └─ Latency: N/A (batch)                              │  │
│  │                                                           │  │
│  │  4. Damage Detection (ResNet-50)                         │  │
│  │     ├─ Input: User-submitted photos                      │  │
│  │     ├─ Frequency: On-demand (vehicle return)             │  │
│  │     ├─ Output: Damage classification + bounding boxes    │  │
│  │     └─ Latency: < 500ms                                   │  │
│  │                                                           │  │
│  │  5. Route Optimization (Graph Neural Network)            │  │
│  │     ├─ Input: Fleet locations, demand forecast           │  │
│  │     ├─ Frequency: Real-time (user trip planning)         │  │
│  │     ├─ Output: Optimal route recommendations             │  │
│  │     └─ Latency: < 1 second                               │  │
│  │                                                           │  │
│  │  6. Customer Segmentation (K-Means)                      │  │
│  │     ├─ Input: User behavior, trip history                │  │
│  │     ├─ Frequency: Batch (monthly)                        │  │
│  │     ├─ Output: User cluster assignments                  │  │
│  │     └─ Latency: N/A (batch)                              │  │
│  │                                                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Edge Hardware Specification

**Vehicle Type: Scooters & eBikes**
```
Compute Module: Raspberry Pi 4 Model B (4GB RAM) or equivalent
  - CPU: ARM Cortex-A72 (Quad-core @ 1.5GHz)
  - RAM: 4GB LPDDR4
  - Storage: 32GB microSD (industrial-grade)
  - Cost: Affordable per unit
  - Power: 3W typical (from vehicle battery)
  
OS: Ubuntu 22.04 ARM64
Runtime: AWS IoT Greengrass v2 Core
ML Framework: TensorFlow Lite 2.15

Sensors:
  - GPS: u-blox NEO-M9N (10Hz)
  - IMU: MPU-9250 (100Hz)
  - Voltage Sensor: INA219
  - Door/Lock Sensors: Reed switches

Connectivity:
  - Cellular: 4G LTE Cat-M1 modem
  - Fallback: Store-and-forward when offline
```

**Vehicle Type: Cars & Vans**
```
Compute Module: NVIDIA Jetson Nano (4GB) or Jetson Xavier NX
  - GPU: 128-core Maxwell (Nano) or 384-core Volta (Xavier)
  - CPU: Quad-core ARM A57 @ 1.43GHz
  - RAM: 4GB-8GB
  - Storage: 64GB eMMC + 128GB SSD
  - Cost: Moderate per unit
  - Power: 5-15W (from 12V vehicle system)
  
OS: JetPack 5.1 (Ubuntu 20.04)
Runtime: AWS IoT Greengrass v2 Core + CUDA
ML Framework: TensorFlow/PyTorch with GPU acceleration

Sensors:
  - GPS: High-precision (RTK)
  - Cameras: 4× 1080p (front, rear, sides)
  - LIDAR: Optional for advanced obstacle detection
  - CAN Bus: Full vehicle telemetry integration
  - IMU: High-frequency (1kHz)

Connectivity:
  - Cellular: 5G modem (for cameras/LIDAR data)
  - WiFi: For software updates when parked
```

#### Model Deployment & OTA Updates

```
┌─────────────────────────────────────────────────────────────────┐
│               OTA MODEL UPDATE WORKFLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. ML Engineer trains new collision detection model            │
│     ├─ SageMaker training job                                   │
│     ├─ Model evaluation (accuracy, latency)                     │
│     └─ Register in Model Registry                               │
│                                                                  │
│  2. Model Optimization for Edge                                 │
│     ├─ SageMaker Neo compilation for ARM/GPU                    │
│     ├─ Quantization: FP32 → INT8 (4x smaller, faster)          │
│     ├─ Pruning: Remove significant portion of weights           │
│     └─ Result: 50MB → 5MB model                                 │
│                                                                  │
│  3. Testing                                                      │
│     ├─ Deploy to test fleet (10 vehicles)                       │
│     ├─ Monitor for 7 days (accuracy, latency, crashes)          │
│     └─ If OK, proceed; else rollback                            │
│                                                                  │
│  4. Staged Rollout                                               │
│     ├─ Phase 1: Small fleet subset                              │
│     ├─ Phase 2: Expanded rollout                                │
│     ├─ Phase 3: Majority rollout                                │
│     └─ Phase 4: Full fleet                                      │
│                                                                  │
│  5. OTA Update Mechanism (IoT Greengrass)                       │
│     ├─ Vehicle connects to WiFi/4G                              │
│     ├─ Greengrass checks for updates                            │
│     ├─ Downloads new model component (delta updates)            │
│     ├─ Verifies signature (code signing)                        │
│     ├─ Stops old model component                                │
│     ├─ Starts new model component                               │
│     ├─ Health check (inference test)                            │
│     └─ If fail: Auto-rollback to previous version               │
│                                                                  │
│  6. Monitoring                                                   │
│     ├─ CloudWatch metrics per vehicle                           │
│     ├─ Alerts if error rate exceeds threshold                   │
│     └─ Rollback command if critical issues                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Cost Analysis (Monthly, 50K Vehicles)

**Edge Compute Costs:**
```
Hardware (One-time):
  - Scooters/eBikes: Substantial initial investment
  - Cars/Vans: Higher per-unit cost
  - Total (amortized over equipment lifetime)

Cellular Data (Ongoing):
  - Fleet connectivity costs for data transmission

OTA Updates (Model updates):
  - Periodic model update distribution costs

Edge Compute Power:
  - Powered by vehicle battery

Total Edge: Within acceptable operational costs
```

**Cloud Compute Costs:**
```
IoT Core (MQTT messages):
  - Large fleet message volume

SageMaker Endpoints (real-time):
  - Multiple model endpoints for cloud-based inference

SageMaker Batch (weekly/monthly):
  - Demand, Battery, Segmentation: $5,000

Total Cloud: $150,000/month
```

**Grand Total: $220,933/month**

**Comparison:**
- **If All Cloud:** ~$400K/month (no edge = 10x more API calls)
- **Our Hybrid:** $221K/month
- **Savings:** $179K/month (45% reduction)

#### Strengths
✅ **Optimal Latency:** Safety-critical models run on edge (< 100ms)
✅ **Cost-Effective:** 45% cheaper than all-cloud
✅ **Offline Capable:** Vehicles work without connectivity
✅ **Scalable:** Cloud handles complex models
✅ **Flexible:** Easy to move models between edge/cloud

#### Weaknesses
❌ **Hardware Cost:** $4M upfront for edge devices
❌ **Operational Complexity:** Manage both edge and cloud
❌ **OTA Risk:** Model updates could brick vehicles
❌ **Limited Edge Compute:** Can't run large models on edge

---

### Alternative 2: All Cloud (No Edge ML)

#### Architecture
- All ML models run in AWS SageMaker
- Vehicles send raw telemetry to cloud
- Cloud sends back decisions (lock, alert, etc.)

#### Strengths
✅ **Simplicity:** Only manage cloud infrastructure
✅ **Flexibility:** Easy to update models
✅ **Powerful:** Can run any model size

#### Weaknesses
❌ **Latency:** Round-trip to cloud = 200-500ms ⚠️ Too slow for safety
❌ **Cost:** 10x more API calls ($400K vs $221K)
❌ **Offline:** Vehicles don't work without connectivity
❌ **Bandwidth:** 50K vehicles × 1KB/sec = 50MB/sec continuous

#### Scoring: 4.5/10 ❌ Not viable for safety-critical operations

---

### Alternative 3: All Edge (No Cloud ML)

#### Architecture
- All ML models run on vehicles
- Powerful edge hardware (NVIDIA Xavier NX for all vehicles)
- Cloud only for data storage and BI

#### Strengths
✅ **Ultra-Low Latency:** All decisions instant
✅ **Fully Offline:** No cloud dependency
✅ **Privacy:** No data sent to cloud

#### Weaknesses
❌ **Hardware Cost:** $20M (50K × $399) ⚠️ 5x more expensive
❌ **Power:** High-power GPUs drain batteries
❌ **Limited Models:** Can't run huge models on edge
❌ **Slow Updates:** OTA to 50K vehicles is risky
❌ **No Historical Analysis:** Can't do demand forecasting

#### Scoring: 5.0/10 ❌ Too expensive and limited

---

## Decision Matrix

| Criteria | Weight | Edge-Cloud Hybrid | All Cloud | All Edge |
|----------|--------|-------------------|-----------|----------|
| **Latency (Safety)** | 30% | 10 | 4 | 10 |
| **Cost** | 25% | 9 | 6 | 3 |
| **Offline Capability** | 15% | 10 | 2 | 10 |
| **Model Flexibility** | 15% | 9 | 10 | 5 |
| **Operational Complexity** | 10% | 6 | 9 | 7 |
| **Scalability** | 5% | 9 | 10 | 6 |
| **Weighted Score** | | **8.9** | **5.9** | **6.7** |

## Decision

**We adopt an Edge-Cloud Hybrid strategy with AWS IoT Greengrass v2 for edge deployment and SageMaker for cloud ML.**

### Rationale

#### 1. Safety First (Critical)
- **Collision detection on edge:** < 50ms latency
- Cloud round-trip (200-500ms) too slow for safety
- Edge ensures vehicle works offline

#### 2. Cost Optimization
- **45% cheaper** than all-cloud ($221K vs $400K/month)
- Amortized hardware cost acceptable
- Reduced cellular data costs

#### 3. Best of Both Worlds
- **Edge:** Low-latency, safety-critical, offline
- **Cloud:** Complex models, historical analysis, flexibility

## Implementation Plan

### Phase 1: Edge Infrastructure (Months 1-2)
- Select and procure hardware
- Develop Greengrass components
- Train and deploy collision detection model
- Test OTA update mechanism

### Phase 2: Cloud ML Platform (Months 2-4)
- Build SageMaker pipelines (ADR-16)
- Deploy demand forecasting
- Deploy dynamic pricing

### Phase 3: Integration (Month 4-5)
- Connect edge → cloud telemetry
- Implement fallback mechanisms
- Load testing (1K → 10K → 50K vehicles)

### Phase 4: Production Rollout (Months 5-11)
- Deploy to 1% → 10% → 100% of fleet
- Monitor and optimize
- Iterate on models

## Consequences

### Positive
✅ **Safety:** Edge models ensure sub-100ms responses
✅ **Cost-Effective:** 45% cheaper than all-cloud
✅ **Offline Capable:** Vehicles work without connectivity
✅ **Scalable:** Cloud handles complex analysis
✅ **Flexible:** Can move models between edge/cloud

### Negative
❌ **Complexity:** Manage both edge and cloud
❌ **Hardware Investment:** $4M upfront
❌ **OTA Risk:** Updates could cause issues
❌ **Limited Edge Compute:** Large models must be cloud

### Mitigation
- **Complexity:** Greengrass simplifies edge management
- **Hardware:** Amortize over 5-year lifespan
- **OTA Risk:** Staged rollout + auto-rollback
- **Edge Limits:** Regularly evaluate moving models to edge as hardware improves

## Related Decisions
- **ADR-15:** AWS chosen - enables IoT Core + Greengrass
- **ADR-16:** MLOps - Manages cloud ML models
- **ADR-03:** Vehicle Telemetry - Uses IoT Core
- **ADR-18:** Agentic AI - Runs entirely in cloud (not edge)

## References
- [AWS IoT Greengrass Documentation](https://docs.aws.amazon.com/greengrass/)
- [SageMaker Neo (Edge Optimization)](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html)
- [TensorFlow Lite for Microcontrollers](https://www.tensorflow.org/lite/microcontrollers)

## Revision History
- **2025-11-07:** Initial decision - Edge-Cloud Hybrid
- **Next Review:** 2026-05-07 - After full fleet deployment
