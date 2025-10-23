# ADR-03: Vehicle Telemetry – Collision Detection, Visual Verification, and Predictive Maintenance

## Status

Accepted  

## Context

MobilityCorp's multi-modal vehicle fleet (bikes, scooters, cars, vans) generates continuous telemetry data that must support three critical AI-driven capabilities:

1. **Collision Detection:** Real-time detection of vehicle collisions using accelerometer/IMU sensors, wheel/ABS status, and camera-based motion analysis to trigger immediate safety responses (alerts, emergency braking assist, incident logging).

2. **Visual Pickup/Dropoff Verification:** Camera-based verification of rental start/end events to confirm vehicle condition, detect damage, and resolve disputes while minimizing privacy exposure and bandwidth usage.

3. **Predictive Maintenance:** Continuous monitoring of vehicle health metrics (battery temperature, vibration patterns, error codes, usage patterns) to forecast component failures and optimize maintenance scheduling, reducing downtime and operational costs.

### Requirements and Constraints

- **Safety-critical latency:** Collision detection must respond within milliseconds; cloud-only processing is too slow.
- **Privacy compliance:** GDPR and data protection regulations require minimizing raw image uploads and obtaining user consent.
- **Intermittent connectivity:** Vehicles may operate in areas with poor network coverage; edge processing is essential.
- **Limited edge compute:** Vehicle IoT hardware has constrained CPU/GPU resources; models must be lightweight and optimized.
- **Bandwidth costs:** Continuous raw telemetry and image streaming would be prohibitively expensive at fleet scale (10,000+ vehicles per city).
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

Adopt a **hybrid edge-cloud telemetry architecture** with intelligent workload distribution based on latency requirements, privacy constraints, and bandwidth optimization.

### Architecture Components

**1. Collision Detection (Edge-First with Rule-Based Fallback)**

- **Primary:** Lightweight on-device ML model fusing accelerometer/IMU data, wheel slip/ABS status, and camera motion vectors for immediate collision detection (<100ms latency)
- **Fallback:** Deterministic rule engine (e.g., IMU g-force >4G + wheel slip + sudden deceleration) triggers emergency response if ML model is unavailable or low confidence
- **Output:** Immediate safety actions (emergency alerts, automatic braking assist if applicable), incident metadata logged locally and synced to cloud
- **Model specs:** Quantized neural network (<5MB), optimized for ARM processors, trained on labeled collision scenarios

**2. Visual Pickup/Dropoff Verification (Privacy-Preserving Edge Inference)**

- **On-device processing:** Run lightweight computer vision models to detect damage, verify vehicle condition, and generate verification score (0-1 confidence)
- **Privacy-first uploads:** Only upload privacy-preserving artifacts:
  - Blurred thumbnails with damage regions highlighted
  - Perceptual hashes or embeddings (not raw images)
  - Metadata: timestamp, GPS, verification score, damage classification
- **Selective full-image upload:** Raw images retained locally for 7 days, uploaded only for:
  - Disputes requiring manual review (with user consent)
  - Model improvement (after anonymization and user opt-in)
  - Regulatory compliance (insurance claims, legal requirements)
- **Model specs:** MobileNetV3-based damage classifier, YOLOv8-nano for object detection, encrypted local storage

**3. Predictive Maintenance (Local Aggregation + Cloud Training)**

- **Edge aggregation:** Vehicles continuously collect and compress sensor data:
  - Battery: voltage curves, temperature profiles, charging cycles, degradation rate
  - Mechanical: vibration spectra (FFT), brake wear indicators, suspension stress
  - Operational: error codes, usage intensity, environmental conditions (temperature, humidity)
- **Periodic uploads:** Compressed, anonymized telemetry aggregates sent to cloud every 6 hours or when on WiFi
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

- **Data minimization:** Only collect and transmit necessary data; raw images ephemeral by default
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

### Mitigation Strategies

**Safety Fallbacks**

- Define and test deterministic thresholds for all safety-critical functions
- Require dual confirmation (ML + rule-based) for high-stakes decisions
- Implement watchdog timers and fail-safe modes for edge systems

**Privacy Controls**

- Conduct regular privacy impact assessments (PIAs) and third-party audits
- Implement automated data deletion workflows (7-day TTL for images, 30-day for aggregates)
- Deploy differential privacy techniques for fleet-wide analytics where applicable
- Use secure enclaves or trusted execution environments for sensitive processing

**Model Monitoring and Quality Assurance**

- Real-time dashboards for model latency, confidence distributions, and error rates
- Automated alerts for drift detection, performance degradation, or anomalous predictions
- Human-in-the-loop validation for edge cases and low-confidence predictions
- Quarterly model audits and fairness assessments

**Operational Excellence**

- Blue-green deployments for model updates with automated rollback on failure
- Chaos engineering tests to validate system resilience under failures
- Comprehensive documentation and runbooks for incident response
- Cross-training teams on ML ops, edge computing, and privacy compliance

### Metrics and Acceptance Criteria

**Collision Detection**

- **Recall:** ≥99.5% for verified collisions (minimize false negatives)
- **Precision:** ≥90% (tolerate some false positives for safety)
- **Latency:** <100ms from event to alert
- **Availability:** 99.9% uptime for collision detection system

**Pickup/Dropoff Verification**

- **Accuracy:** ≥98% for damage classification (precision & recall)
- **Dispute resolution:** ≥95% of disputes resolved via automated evidence
- **Latency:** <1s for in-vehicle verification display
- **Privacy compliance:** 0 GDPR violations, 100% consent coverage

**Predictive Maintenance**

- **Lead time:** 7-14 days advance warning for battery failures
- **True positive rate:** ≥85% for predicted failures that actually occur
- **Cost savings:** 25-30% reduction in unplanned maintenance vs. reactive approach
- **Downtime reduction:** 30-40% decrease in vehicle unavailability

### Open Questions and Next Steps

**Technical**

- Define specific numeric thresholds for collision detection fallback rules (g-force, wheel slip, etc.)
- Prototype edge model performance on target vehicle hardware (latency, power consumption)
- Evaluate hardware acceleration options (Edge TPU, NVIDIA Jetson) for future vehicle generations

**Privacy and Compliance**

- Design consent UI/UX flows for image uploads per jurisdiction
- Define data retention schedules and deletion workflows for each telemetry type
- Establish privacy review board for ongoing compliance monitoring

**Operational**

- Create detailed runbooks for model deployment, rollback, and incident response
- Set up pilot program with 100-vehicle fleet for shadow deployment and metric validation
- Define SLAs and escalation procedures for model performance degradation

**Business**

- Calculate ROI for predictive maintenance vs. current reactive approach
- Benchmark false positive tolerance with customer support and operations teams
- Establish pricing model for customer data sharing opt-in incentives

---

**Decision Owner:** AI/ML Architecture Team  
**Stakeholders:** Safety Engineering, Privacy/Legal, Fleet Operations, Customer Experience  
**Review Cadence:**

- **Model performance:** Monthly for first 6 months, then quarterly
- **Privacy compliance:** Quarterly audits, annual third-party assessment
- **Safety systems:** Continuous monitoring with immediate escalation for failures
