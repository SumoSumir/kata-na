# ADR-11: IoT Security & Edge Computing for Vehicle Telemetry

## Status - accepted

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
We will implement **mutual TLS (mTLS)** for secure authentication of IoT vehicles to the backend server, ensuring only valid devices send telemetry and battery data. Edge computing modules will process sensor data locally—detecting collision events and aggregating driver analytics (speeding events, collision frequency) before transmitting summaries to the backend.

**Justification:**
- mTLS provides strong cryptographic assurance of device identity and secure transmission.
- Edge computing delivers lower latency for safety-related features (collision detection), improves bandwidth efficiency, and supports real-time analytics.
- Local analysis enables quicker feedback and reduces backend load while maintaining rich, actionable insights (speed violations, collision stats).

## Consequences

**Trade-Offs and Impact:**

- **Pros:**
  - Security: mTLS prevents unauthorized data ingestion, isolates device streams, and protects data in transit.
  - Reliability: Edge analytics ensures collision detection is not dependent on cloud latency or connectivity.
  - Efficiency: Offloading processing to the edge conserves bandwidth and reduces backend compute costs.
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
