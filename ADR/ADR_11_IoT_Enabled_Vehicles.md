# ADR-11: IoT Security & Edge Computing for Vehicle Telemetry

## Status
Accepted

## Context
Our automotive telematics platform requires collecting sensor and telemetry data from IoT-enabled vehicles to detect collisions (via accelerometer and other sensors), analyze driver behavior (speeding, collision events), and monitor battery health. The system must:

- **Secure data transmission** between vehicles and backend.
- **Process data at the edge** for real-time responsiveness.
- **Authenticate vehicle identity** to ensure data integrity and privacy.

Available alternatives include:
- Only cloud-based analytics, less edge processing
- Simple authentication (API keys, tokens) between devices and backend
- Event-driven analytics without ML or sensor fusion

## Decision
We will implement **AWS IoT Core with X.509 certificate-based mTLS authentication** for secure communication between 50,000 vehicles and the cloud backend. Edge computing will be powered by **AWS IoT Greengrass v2** running on vehicle hardware.

**IoT Infrastructure:**
- **Protocol:** MQTT over TLS 1.3 (lightweight, pub/sub)
- **Broker:** AWS IoT Core (fully managed, auto-scaling)
- **Authentication:** X.509 certificates issued by AWS IoT CA
- **Device Registry:** AWS IoT Device Registry (identity, metadata, certificates)
- **Connection:** 50,000 persistent connections, 1 msg/sec per vehicle

**Edge Computing Platform:**
- **Software:** AWS IoT Greengrass v2 (edge runtime)
- **Hardware:** 
  - Scooters/eBikes: Raspberry Pi 4 (ARM, 4GB RAM, $65/unit)
  - Cars/Vans: NVIDIA Jetson Xavier NX (ARM, 8GB RAM, $399/unit)
- **OTA Updates:** Greengrass component deployment (zero-touch updates)
- **Local Storage:** 32GB SD card (telemetry buffering during offline periods)

**Edge Processing:**
- **Collision Detection:** TensorFlow Lite model (< 50ms latency)
- **Battery Health:** On-device analytics (Remaining Useful Life estimation)
- **Data Aggregation:** Local buffering, batch uploads every 60 seconds
- **Offline Operation:** 24-hour buffer, sync when connectivity restored

**Security:**
- **Certificate Rotation:** Automated via IoT Device Management (annual rotation)
- **Least Privilege:** Each vehicle has unique certificate, can only publish to own topics
- **Encryption:** AES-256 for data at rest on device, TLS 1.3 for data in transit
- **Device Defender:** AWS IoT Device Defender for anomaly detection (compromised devices)

**Cost:** ~$49,775/month (IoT Core connectivity + messaging for 50K vehicles)

**Justification:**
- mTLS provides strong cryptographic assurance of device identity and secure transmission.
- Edge computing delivers lower latency for safety-related features (collision detection), improves bandwidth efficiency, and supports real-time analytics.
- Local analysis enables quicker feedback and reduces backend load while maintaining rich, actionable insights (speed violations, collision stats).

## Consequences

**Trade-Offs and Impact:**

- **Pros:**
  - Security: mTLS prevents unauthorized data ingestion, isolates device streams, and protects data in transit.
  - Reliability: Edge analytics ensures collision detection is not dependent on cloud latency or connectivity.
  - Efficiency: Offloading processing to the edge conserves bandwidth and reduces backend compute costs. Only actionable/aggregated data sent to backend which will be easier to scale as well.
  - Rich analytics: Enables granular driver analysis (over-speed, collision frequency) with summarized reporting to backend.

- **Cons:**
  - Added complexity and operational overhead in managing certificates and edge deployment.
  - Firmware updates needed for mTLS and analytics logic on IoT devices.
  - Some analytics might require backend processing for longitudinal, cross-vehicle analysis.

**Comparison Against Alternatives:**
- **Simple Auth (API keys/tokens):** Lower security, risks forged/fraudulent data streams.
- **Cloud-centric Analytics:** Higher latency for urgent events, significantly increased bandwidth usage.
- **No Edge Analytics:** Limits ability to respond in real time and puts heavy load on backend infrastructure.

**Summary:**
Applying mTLS for secure authentication and edge computing for sensor analysis provides a robust, scalable, and secure solution for vehicle IoT telemetry. This design prioritizes safety, privacy, and valuable driver behavior analytics, ensuring that only authenticated vehicles contribute trusted data to system monitoring and reporting.
