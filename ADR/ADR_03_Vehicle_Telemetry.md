# ADR-03: Vehicle Telemetry – Collision Detection, Visual Verification, and Predictive Maintenance

## Status

Accepted  

## Context

MobilityCorp's multi-modal vehicle fleet (bikes, scooters, cars, vans) generates continuous telemetry data that must support three critical AI-driven capabilities:

1. **Collision Detection:** Real-time detection of vehicle collisions using accelerometer/IMU sensors, wheel/ABS status to trigger immediate safety responses to generate alerts/emergency braking assist/incident logging.

2. **Visual Pickup/Dropoff Verification:** Camera-based verification via end users mobile app at rental start/end events to confirm vehicle condition, detect damage, and resolve disputes while minimizing privacy exposure and bandwidth usage.

3. **Predictive Maintenance:** Continuous monitoring of vehicle health metrics (battery temperature, vibration patterns, error codes, usage patterns) to forecast component failures and optimize maintenance scheduling, reducing downtime and operational costs.

### Requirements and Constraints

- **Safety-critical latency:** Collision detection must respond within milliseconds; cloud-only processing is too slow.
- **Privacy compliance:** GDPR and data protection regulations require minimizing raw image uploads and obtaining user consent.
- **Intermittent connectivity:** Vehicles may operate in areas with poor network coverage; edge processing is essential.
- **Limited edge compute:** Vehicle IoT hardware has constrained CPU/GPU resources; models must be lightweight and optimized.
- **Bandwidth costs:** Continuous raw telemetry and image streaming would be prohibitively expensive at fleet scale (10,000+ vehicles per country).
- **Fleet-wide learning:** Models should improve over time using aggregated data from all vehicles while preserving privacy.

### Alternatives Considered

**1. Fully Cloud-Based Processing**

- Stream all raw telemetry and images to cloud for processing
- ✅ Pros: Unlimited compute, easy model updates, centralized storage
- ❌ Cons: High latency (unsuitable for safety), massive bandwidth costs, privacy risks, connectivity dependency
- **Rejected:** Cannot meet safety-critical latency requirements

**2. Fully On-Device (Edge-Only) Processing**

- Run all models on vehicle hardware, only send alerts/metrics
- ✅ Pros: Minimal latency, low bandwidth, strong privacy
- ❌ Cons: Limited by edge compute, difficult model updates, no fleet-wide learning, model drift
- **Rejected:** Cannot leverage fleet data for continuous improvement

**3. Rule-Based Systems Only**

- Use deterministic thresholds (e.g., g-force > 5G = collision)
- ✅ Pros: Predictable, simple, no ML complexity
- ❌ Cons: High false positive rates, cannot learn patterns, rigid thresholds fail edge cases
- **Rejected:** Insufficient accuracy for production deployment

## Decision

Adopt a **hybrid edge-cloud vehicle telemetry architecture** with intelligent workload distribution based on latency requirements, privacy constraints, and bandwidth optimization.

### Architecture Components

**1. Collision Detection (Edge-First with Rule-Based Fallback)**

- **Primary:** Lightweight on-device ML model fusing accelerometer/IMU data and wheel slip/ABS status
- **Fallback:** Deterministic rule engine (e.g., IMU g-force >4G + wheel slip + sudden deceleration) triggers emergency response if ML model is unavailable or low confidence
- **Output:** Immediate safety actions (emergency alerts, automatic braking assist if applicable), incident metadata logged locally and synced to cloud
- **Model specs:** Quantized neural network (<5MB), optimized for ARM processors, trained on labeled collision scenarios

**2. Visual Pickup/Dropoff Verification (Cloud-Based)**

- **How it Works**
  - Mobile app captures and encrypts images, uploads to cloud
  - Cloud ML models (YOLOv8, EfficientNet) detect damage and verify condition
  - Returns: verification score, damage classification, blurred thumbnails

- **Privacy & Security**
  - Metadata: timestamp, GPS, verification score, damage classification
  - Raw images stored encrypted for 7 days, then deleted
  - Full images retained only for: disputes, user opt-in (training), legal requirements
  - Device attestation prevents tampering; all access audited

- **Bandwidth & Performance**
  - WiFi preferred; compression for cellular
  - <1 second verification latency
  - Full resolution requested only when needed

- **Model Training**
  - User opt-in required
  - Images anonymized before training
  - Differential privacy applied

**3. Predictive Maintenance (Local Aggregation + Cloud Training)**

- **Edge aggregation:** Vehicles continuously collect, compress & stream minimal sensor data at short intervals:
  - Battery: voltage curves, temperature profiles, charging cycles, degradation rate
  - Operational: error codes, usage intensity, environmental conditions (temperature, humidity), GPS.
- **Periodic uploads:** Compressed, anonymized telemetry aggregates sent to cloud every couple of hours or when on WiFi.
  - E.g. Mechanical: vibration spectra (FFT), brake wear indicators, suspension stress
- **Cloud-based training:** Fleet-wide models trained on aggregated data to predict:
  - Battery failure (7-14 day advance warning)
  - Brake replacement needs (distance-based prediction)
  - Component degradation patterns (personalized maintenance schedules)
- **Model deployment:** Updated predictive models pushed to vehicles via OTA updates (staged canary rollout)
- **Model specs:** Time-series forecasting (LSTM/Transformer), anomaly detection (Isolation Forest), fleet-level ensemble models

### Operational Practices

**Safety and Reliability**

- **Rule-based safety net:** Deterministic fallbacks ensure critical functions operate even if ML models fail
- **Model validation:** All edge models undergo shadow deployment and A/B testing before production rollout
- **Confidence thresholds:** Low-confidence ML predictions trigger rule-based fallback or manual review
- **Quick rollback:** OTA updates enable rapid model reversion if issues detected

**Privacy and Compliance**

- **Data minimization:** Only collect and transmit necessary data (stream at regular intervals for time sensitive data); raw images ephemeral by default
- **Consent management:** Users explicitly opt-in to image uploads and data sharing for model training
- **Regional compliance:** Separate data retention policies per jurisdiction (GDPR, DPDP Act, etc.)
- **Encryption:** End-to-end encryption for all telemetry data in transit and at rest
- **Audit trails:** Comprehensive logging of data collection, processing, and deletion events

**ML Lifecycle Management**

- **Continuous monitoring:** Track model performance metrics (latency, accuracy, drift, false positive/negative rates)
- **Automated retraining:** Trigger retraining pipelines when drift detected or performance degrades
- **Model cards:** Document training data, performance benchmarks, failure modes, and ethical considerations
- **Explainability:** Generate SHAP values or attention maps for critical predictions (collision, damage classification)

**Cost Optimization**

- **Adaptive telemetry:** Adjust upload frequency based on vehicle status (stationary vs. active, WiFi vs. cellular)
- **Compression:** Use efficient encoding (Protocol Buffers, AVRO) and compression (gzip, Brotli) for uploads
- **Batch processing:** Group telemetry uploads to reduce API overhead and connection costs

## Consequences

### ✅ Positive Impacts

**Safety and Responsiveness**

- Sub-100ms collision detection enables immediate safety interventions
- Rule-based fallback ensures safety systems never fail completely
- Real-time damage detection improves dispute resolution and customer trust

**Cost Efficiency**

- 80-90% reduction in bandwidth costs vs. cloud-only approach (selective uploads, compression)
- Predictive maintenance reduces unplanned downtime by 30-40% (industry benchmarks)
- Extended vehicle lifespan through optimized maintenance scheduling

**Privacy and Compliance**

- Privacy-preserving architecture minimizes GDPR exposure and regulatory risk
- User consent flows and data minimization align with best practices
- Ephemeral image handling reduces data breach liability

**Scalability and Fleet Learning**

- Cloud aggregation enables fleet-wide model improvements without compromising individual privacy
- New vehicle types onboard easily via model transfer learning
- Cross-city pattern detection improves operational efficiency

### ❌ Negative Impacts and Trade-offs

**Increased System Complexity**

- Hybrid architecture requires managing both edge and cloud ML pipelines
- Device orchestration, OTA updates, and model versioning add operational overhead
- Debugging distributed ML systems is more complex than cloud-only approaches

**Higher Engineering Investment**

- Edge model optimization requires specialized ML engineering (quantization, pruning, hardware acceleration)
- CI/CD pipelines must support model training, validation, deployment, and rollback
- Cross-functional expertise needed (embedded systems, ML ops, privacy engineering)

**Operational Costs**

- Edge inference hardware costs (GPU/NPU modules on vehicles if needed)
- Cloud storage for fleet telemetry archives and model training data
- Monitoring infrastructure for model health, drift detection, and incident response

**Model Performance Risks**

- **False positives:** Nuisance collision alerts or incorrect damage classifications degrade user experience
- **False negatives:** Missed collisions or undetected damage create safety/liability risks
- **Model drift:** Sensor aging, environmental changes, or new vehicle types can degrade accuracy over time
- **Edge compute limits:** Model complexity constrained by vehicle hardware capabilities

**Privacy and Legal Risks**

- Even anonymized data may be re-identifiable; requires ongoing privacy audits
- Multi-jurisdictional compliance complexity (different regulations per EU country, India, etc.)
- User consent management adds UX friction and operational overhead
- Data breaches of telemetry data could expose sensitive location/behavior patterns