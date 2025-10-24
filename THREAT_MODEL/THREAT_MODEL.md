# THREAT MODEL: MobilityCorp Multi-Modal Transportation Platform


## ASSETS


### Critical Data Assets
1. **Customer PII (Personal Identifiable Information)**
  - Names, email addresses, phone numbers, payment details
  - Location data (GPS coordinates, trip history)
  - KYC documents (identity verification)
  - User behavior patterns and preferences


2. **Operational Data**
  - Vehicle locations and telemetry (real-time GPS, battery status)
  - Fleet inventory and availability
  - Booking and reservation data
  - Vehicle maintenance records
  - Staff task assignments and completion logs


3. **Financial Assets**
  - Payment transactions and authorization tokens
  - Customer payment methods (credit cards, digital wallets)
  - Dynamic pricing algorithms and business logic
  - Revenue and financial analytics


4. **AI/ML Models and Training Data**
  - Demand forecasting models
  - Predictive maintenance models
  - Damage detection vision models
  - Training datasets (historical bookings, telemetry, images)
  - Model weights and hyperparameters


5. **Authentication and Authorization Credentials**
  - Customer OAuth2/JWT tokens
  - Staff authentication credentials
  - Vehicle mTLS certificates and device keys
  - API keys for external services (OpenWeatherMap, PredictHQ, payment gateways)
  - Database credentials and encryption keys


6. **System Configuration and Secrets**
  - Kafka broker configurations
  - Database connection strings
  - Redis cache credentials
  - S3 bucket access keys
  - Multi-provider AI orchestrator API keys (OpenAI, Anthropic, Gemini, Azure)


## TRUST BOUNDARIES


### TB1: Internet to API Gateway
- Separates untrusted public internet from internal services
- All customer mobile app and staff dashboard traffic crosses this boundary
- Authentication and rate limiting enforced at this boundary


### TB2: API Gateway to Core Services
- Separates authenticated frontend traffic from backend business logic
- JWT/OAuth2 token validation occurs here
- Request routing and authorization checks performed


### TB3: Core Services to AI/ML Services
- Separates transactional services from AI inference services
- May involve cross-region communication for model serving
- Data anonymization may occur at this boundary for compliance


### TB4: Core Services to Data Layer
- Separates application logic from data persistence
- Database credentials and connection security critical
- Encryption at rest enforced in data stores


### TB5: Vehicle Edge Devices to Cloud Ingestion
- Separates IoT devices (untrusted embedded systems) from cloud infrastructure
- mTLS authentication required for vehicle telemetry submission
- Physical access to vehicles represents threat vector


### TB6: Cloud to External APIs
- Separates internal systems from third-party services
- API keys and rate limiting for weather, events, payment gateways
- Data exfiltration risk if external APIs compromised


### TB7: Regional Deployments
- Separates EU (& future India, US) regions for data residency compliance
- PII must not cross regional boundaries (GDPR)
- Model registries and data stores region-scoped


### TB8: Event Bus (Kafka) to Consumers
- Separates event producers from consumers
- Multiple services consume same events
- Authorization and topic-level access control required


### TB9: Edge Processing to Cloud Processing
- Separates on-vehicle ML inference from cloud-based training/monitoring
- Privacy-preserving design
- OTA updates cross this boundary in reverse direction


### TB10: Multi-Provider AI Orchestrator to External LLM Providers
- Separates internal orchestration logic from external AI services (similar to API orchestrator)
- API keys and cost controls critical
- Data sent to third-party LLMs (OpenAI, Anthropic, Gemini) leaves organizational control


## DATA FLOWS


### DF1: Customer Mobile App → API Gateway → Booking Service (TB1, TB2)
- HTTPS encrypted, JWT authentication
- Booking requests with location, vehicle type, payment method
- Crosses internet-to-internal trust boundary


### DF2: Vehicle Edge Device → IoT Gateway → Kafka (TB5, TB8)
- mTLS encrypted telemetry (GPS, battery, collision alerts)
- Crosses device-to-cloud trust boundary
- High-frequency data flow (30-60s intervals)


### DF3: Kafka → Telemetry Service → TimescaleDB (TB8, TB4)
- Internal event stream processing
- Writes time-series data to persistent storage
- Retention policies enforce data lifecycle


### DF4: Booking Service → Payment Service → External Payment Gateway (TB6)
- Payment authorization requests
- Crosses internal-to-external trust boundary
- Contains sensitive payment credentials


### DF5: Demand Forecasting Service → External Weather/Events/Maps APIs (TB6)
- Fetches weather and event data for ML features
- API keys transmitted over HTTPS
- Cached in Redis to reduce external calls


### DF6: AI/ML Training Pipeline (Airflow) → S3 Data Lake → Model Registry (TB4)
- Reads historical data, trains models, stores artifacts
- Scheduled batch jobs (weekly/monthly)
- Anonymized datasets for cross-region training


### DF7: Conversational AI → Multi-Provider Orchestrator → External LLMs (TB3, TB10)
- User queries sent to OpenAI/Anthropic/Gemini
- API keys and cost tracking
- User PII may be included in prompts (privacy risk)


### DF8: Damage Detection → Cloud Vision AI Service (TB9, TB3)
- Raw images retained locally for 7 days, uploaded only for disputes
- Crosses edge-to-cloud trust boundary


### DF9: Operations Dashboard → Fleet Operations Service → Vehicle Management (TB1, TB2, TB3)
- Staff task assignments and vehicle relocations
- Authenticated staff sessions (OAuth2)
- Real-time updates via WebSocket


### DF10: Dynamic Pricing Engine → Kafka → Booking Service (TB3, TB8)
- Publishes pricing updates every 5-15 minutes
- Consumed by booking service for current rates
- Business-critical data flow affecting revenue


### DF11: Predictive Maintenance Service → Kafka → Operations Dashboard (TB3, TB8)
- Publishes maintenance alerts (7-14 day battery failure warnings)
- Consumed by operations team for task prioritization
- Event-driven workflow triggers


### DF12: OTA Update Service → Vehicle Edge Devices (TB9 reverse)
- Pushes updated ML models and firmware to vehicles
- Staged canary rollout (5% → 25% → 100%)
- Integrity verification (signed updates) critical


### DF13: Multi-Region Data Replication (TB7)
- Operational data replicated across regions
- PII strictly isolated per region (GDPR, DPDP compliance)
- Anonymized aggregates may cross regions for analytics


### DF14: OpenTelemetry → Grafana/VictoriaMetrics (TB4)
- Distributed tracing and metrics collection
- Logs may contain sensitive data (requires sanitization)
- Monitoring infrastructure access control critical


## THREAT MODEL

See PDF


QUESTIONS & ASSUMPTIONS
Questions


Authentication Implementation Details: What specific OAuth2 flows are used (authorization code, client credentials)? Are refresh tokens rotated? What is JWT token expiration policy?
Network Segmentation: Are microservices deployed in separate VPCs or subnets with network-level isolation? What firewall rules exist between service tiers?
Secrets Management: Which secrets management solution is used (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)? How are secrets rotated?
Database Encryption: Is encryption at rest enabled for all databases (PostgreSQL, TimescaleDB, Redis)? What encryption keys are used (AWS KMS, customer-managed)?
Incident Response: What incident response procedures exist for security breaches? Who is the security team, and what are escalation paths?
Penetration Testing: How frequently are security assessments and penetration tests conducted? Are both application and infrastructure tested?
Compliance Audits: How are GDPR, DPDP, and CCPA compliance validated? Are third-party audits conducted annually?
Employee Training: What security awareness training do developers, operations staff, and data scientists receive? How often is training refreshed?
Third-Party Risk: How are external API providers (OpenWeatherMap, PredictHQ, payment gateways) security-assessed? Are vendor security questionnaires required?
Backup and Disaster Recovery: What backup strategies exist for databases and data lakes? How are backups encrypted and access-controlled? What is RPO/RTO?


Assumptions


Cloud Provider: System is deployed on major cloud provider (AWS, Azure, GCP) with baseline security controls (IAM, security groups, VPCs) properly configured.
TLS Everywhere: All inter-service communication uses TLS 1.2+ encryption. Certificate management and rotation are automated.
Logging and Monitoring: Centralized logging (ELK, Splunk) and SIEM (Security Information and Event Management) are deployed for security monitoring and alerting.
Patch Management: Regular patching process exists for operating systems, application dependencies, and database engines. Critical vulnerabilities patched within 30 days.
Secure Development: Developers follow secure coding guidelines. Code reviews include security checks. SAST/DAST tools integrated in CI/CD pipeline.
Access Control: Production environment access is restricted to authorized personnel. Multi-factor authentication (MFA) required for privileged access.
Data Retention: Data retention policies align with regulatory requirements (GDPR Article 5: storage limitation). Automated deletion workflows implemented.
Vendor SLAs: External API providers (payment gateways, LLMs, weather APIs) have contractual SLAs for availability and security. Data processing agreements (DPAs) in place for GDPR compliance.
Edge Device Security: Vehicle IoT devices have secure boot, hardware-backed secure enclaves, and tamper detection. Physical security measures (locked enclosures) protect critical components.
Regulatory Compliance: Legal and compliance teams review architecture decisions. Data protection impact assessments (DPIAs) conducted for high-risk processing activities.
Default Deny: Security posture follows principle of least privilege and default deny. All access must be explicitly granted and justified.
Separation of Environments: Production, staging, and development environments are strictly separated with no data sharing (except anonymized test data).


NOTES ON THREAT PRIORITIZATION
Why Certain Threats Lack Controls


Physical Vehicle Theft (Not Included): While vehicles could be physically stolen (breaking locks, towing), this is primarily an operational/insurance concern rather than cybersecurity threat. GPS tracking enables recovery. Impact is bounded to individual vehicles, not systemic.
DDoS Attacks (Not Included): Distributed Denial of Service attacks against API Gateway are possible but mitigated by cloud provider infrastructure (AWS Shield, Cloudflare). Impact is temporary service disruption, not data breach. Standard DDoS protection assumed in cloud deployment.
Social Engineering (Not Included): Phishing attacks against employees could compromise credentials, but this is a general threat not specific to architecture. Addressed through security awareness training and MFA enforcement (assumed baseline control).
Ransomware (Not Included): While microservices could be targeted by ransomware, immutable infrastructure (containers, infrastructure-as-code) and automated backups provide resilience. Threat is not unique to this architecture.
Nation-State APTs (Low Priority): Advanced Persistent Threats from nation-state actors are theoretically possible but unlikely for commercial mobility platform. Threat model focuses on realistic threats (cybercriminals, hacktivists, insiders) rather than APTs with unlimited resources.


Real-World Risk Assessment
The threat model prioritizes threats based on:


Likelihood in Similar Systems: Threats commonly observed in real-world breaches of mobility platforms, IoT systems, and AI/ML services (e.g., exposed databases, API key leaks, data residency violations).
Regulatory Focus: Threats with high compliance risk (GDPR violations, PCI-DSS non-compliance) prioritized due to certain financial penalties and reputational damage.
Business Impact: Threats affecting revenue (payment fraud, pricing manipulation), safety (collision detection failures, maintenance prediction errors), and customer trust (data breaches) prioritized over pure availability concerns.
Defense Complexity: Some threats (e.g., sophisticated supply chain attacks, zero-day exploits) are acknowledged but not primary focus because defense requires enterprise-wide programs beyond architecture decisions.
Cascading Effects: Threats enabling lateral movement or privilege escalation (Kafka without ACLs, compromised CI/CD) prioritized due to wide blast radius.

