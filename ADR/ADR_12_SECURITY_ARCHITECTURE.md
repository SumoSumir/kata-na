# ADR-12: Security-by-Design Architecture

## Status
Accepted

## Context
MobilityCorp handles sensitive data across 50,000 IoT-enabled vehicles, millions of customers, and payment transactions totaling €80M annually. The architecture must comply with GDPR, PCI-DSS Level 1, and prepare for ISO 27001/SOC 2 Type II certification. Security cannot be an afterthought—it must be architected from day one.

**Business Requirements:**
- Zero data breaches (brand risk: €10M+ GDPR fines)
- PCI-DSS Level 1 compliance for payment processing
- GDPR compliance (data residency, right-to-erasure, consent management)
- IoT security for 50,000 vehicles (prevent unauthorized access, tampering)
- Zero-trust architecture (no implicit trust, verify everything)

**Persona Drivers:**
- **Raj Patel (CISO):** "Security isn't a feature you bolt on—it must be architected from day one. Every API call, every data store, every IoT message needs encryption and authentication."
- **David Park (CTO):** "I need an architecture that fails gracefully and isolates security breaches—one compromised service shouldn't take down the entire platform."
- **Marcus Weber (VP Ops):** "Security shouldn't slow us down—I need automated compliance checks, not manual audits every quarter."

---

## Decision

Implement a **Zero-Trust, Defense-in-Depth** security architecture on AWS with the following principles:

1. **Zero-Trust Identity:** All access requires authentication + authorization (no implicit trust)
2. **Encryption Everywhere:** AES-256 at rest, TLS 1.3 in transit, mTLS for IoT devices
3. **Least-Privilege Access:** IAM roles grant minimum required permissions
4. **Defense in Depth:** Multiple security layers (network, application, data, identity)
5. **Automated Compliance:** AWS Config, Macie, CloudTrail enforce policies automatically
6. **Security Observability:** Real-time threat detection with GuardDuty, Security Hub

---

## Architecture Components

### 1. Identity & Access Management (Zero-Trust)

**AWS IAM + AWS SSO**
- **Human Access:** AWS SSO with multi-factor authentication (MFA) mandatory
- **Service-to-Service:** IAM roles attached to ECS tasks, Lambda functions (no API keys)
- **Principle:** Least-privilege policies (e.g., `ReadOnlyAccess` for analytics, `PowerUserAccess` for engineers)

**Example IAM Policy (Booking Service):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:eu-central-1:*:table/bookings"
    },
    {
      "Effect": "Allow",
      "Action": ["sns:Publish"],
      "Resource": "arn:aws:sns:eu-central-1:*:booking-events"
    }
  ]
}
```

**Key Features:**
- No long-lived credentials (IAM roles assume temporary tokens)
- MFA enforcement for all human access (AWS Console, CLI, API)
- Service-to-service auth via IAM roles (ECS task roles, Lambda execution roles)

---

### 2. Data Encryption (Defense in Depth)

**Encryption at Rest (AES-256)**
- **AWS KMS:** Customer-managed keys with automatic rotation (90 days)
- **S3:** Server-side encryption (SSE-KMS) for all objects (Bronze/Silver/Gold layers)
- **RDS PostgreSQL:** Encrypted storage volumes, automated backups encrypted
- **DynamoDB:** Encryption at rest enabled for all tables
- **Redshift:** Column-level encryption for PII columns (name, email, phone, address)

**Encryption in Transit (TLS 1.3)**
- **API Gateway:** HTTPS only, TLS 1.3 enforced
- **ALB (Application Load Balancer):** TLS 1.3 for external traffic, HTTP/2 internally
- **IoT Core:** mTLS (mutual TLS) with X.509 device certificates
- **RDS/Redshift:** SSL/TLS connections enforced for all database connections

**Key Management Strategy:**
```
Customer Master Keys (CMKs):
- payments-kms-key → Payment data (PCI-DSS scope)
- pii-kms-key → Personal Identifiable Information (GDPR scope)
- telemetry-kms-key → Vehicle telemetry data
- analytics-kms-key → Aggregated analytics (non-sensitive)

Rotation Policy: Automatic 90-day rotation
Access Control: IAM policies restrict key usage by service
```

---

### 3. Network Security (Perimeter Defense)

**AWS VPC Architecture**
- **Private Subnets:** All microservices run in private subnets (no direct internet access)
- **Public Subnets:** Only ALB, NAT Gateway, Bastion hosts
- **Security Groups:** Stateful firewall rules (e.g., Booking Service only accepts traffic from ALB)
- **Network ACLs:** Subnet-level firewall (defense-in-depth redundancy)

**AWS WAF (Web Application Firewall)**
- **Ruleset:** AWS Managed Rules for OWASP Top 10
  - SQL injection protection
  - Cross-site scripting (XSS) prevention
  - DDoS mitigation (rate limiting: 2,000 req/5min per IP)
- **Custom Rules:** Block non-EU IP addresses (GDPR data residency)
- **Cost:** $3,500/month (includes 10M requests)

**AWS Shield Standard**
- **DDoS Protection:** Network-layer (L3/L4) DDoS mitigation (free tier)
- **Protects:** ALB, CloudFront, Route53
- **Capacity:** Auto-scales to absorb 10 Gbps attacks

**VPC Flow Logs**
- **Purpose:** Network traffic analysis for compliance audits
- **Retention:** 90 days (GDPR data minimization)
- **Integration:** CloudWatch Logs → S3 → Athena for querying
- **Use Case:** Detect unauthorized access attempts, analyze traffic patterns

---

### 4. IoT Security (50,000 Vehicles)

**AWS IoT Core with Device Defender**
- **Authentication:** X.509 certificate per vehicle (50K unique certificates)
- **Authorization:** IoT policies restrict each vehicle to publish only its own telemetry
- **Transport:** mTLS (mutual TLS) for all device-to-cloud communication
- **OTA Updates:** Signed firmware updates via AWS IoT Jobs

**Example IoT Policy (Vehicle 12345):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["iot:Publish"],
      "Resource": ["arn:aws:iot:eu-central-1:*:topic/vehicles/12345/telemetry"]
    },
    {
      "Effect": "Allow",
      "Action": ["iot:Subscribe", "iot:Receive"],
      "Resource": ["arn:aws:iot:eu-central-1:*:topic/vehicles/12345/commands"]
    }
  ]
}
```

**Device Defender Features:**
- **Audit:** Continuous compliance checks (certificate expiration, weak ciphers)
- **Detect:** Anomaly detection (unusual message volume, unauthorized topics)
- **Metrics:** Message count, connection duration, protocol errors
- **Alerts:** SNS notifications for security violations

**Edge Device Security (Raspberry Pi 4):**
- **Hardware:** TPM 2.0 chip for secure boot, certificate storage
- **OS:** Hardened Linux (minimal attack surface, disable unused services)
- **Encryption:** AES-256 encryption for data at rest on SD card
- **Updates:** Signed OTA updates, rollback capability

---

### 5. Application Security (Microservices)

**API Gateway Security**
- **Authentication:** JWT tokens (OAuth 2.0) for user API calls
- **Authorization:** IAM roles for service-to-service API calls
- **Rate Limiting:** 1,000 requests/minute per user (prevent abuse)
- **Throttling:** 10,000 requests/second burst limit (DDoS protection)
- **Validation:** Input validation at API Gateway (reject malformed requests)

**Payment Service (PCI-DSS Level 1)**
- **Isolation:** Dedicated VPC, no cross-service communication except via API Gateway
- **Tokenization:** Stripe handles card data, we store tokens only (no raw card numbers)
- **Logging:** Minimal PCI-scope logging (no card numbers, CVV in logs)
- **Encryption:** TLS 1.3 for Stripe API calls, AES-256 for token storage
- **Compliance:** Quarterly ASV scans, annual PCI audit

**Secrets Management**
- **AWS Secrets Manager:** Store API keys, database passwords, Stripe keys
- **Rotation:** Automatic rotation every 90 days
- **Access Control:** IAM policies restrict secret access by service
- **Auditing:** CloudTrail logs all secret access (who, when, what)

**Example Secret Rotation:**
```
Secrets:
- stripe-api-key → Payment Service only
- db-password-booking → Booking Service only
- bedrock-api-key → Conversational AI Service only

Rotation Schedule: Every 90 days
Notification: SNS alert 7 days before expiration
```

---

### 6. Compliance Automation (GDPR, PCI-DSS, ISO 27001)

**AWS Config**
- **Purpose:** Continuous compliance monitoring (infrastructure as code)
- **Rules:** 50+ AWS-managed rules + 10 custom rules
  - S3 buckets must have encryption enabled
  - RDS instances must have backups enabled
  - Security groups cannot allow 0.0.0.0/0 SSH access
- **Remediation:** Auto-remediate non-compliant resources via Lambda

**AWS CloudTrail**
- **Purpose:** Immutable audit log for compliance (GDPR, SOC 2)
- **Coverage:** All API calls across AWS account (who, what, when, where)
- **Retention:** 7 years (compliance requirement)
- **Integrity:** CloudTrail log file validation (detect tampering)
- **Storage:** S3 bucket with versioning, MFA delete enabled

**AWS Macie**
- **Purpose:** Automated PII detection in S3 data lake
- **Coverage:** Scan 100TB of Bronze/Silver/Gold layers
- **Detection:** Credit card numbers, passport numbers, email addresses, phone numbers
- **Alerting:** SNS notification for PII discovered in wrong location
- **Remediation:** Auto-quarantine files with unexpected PII

**GDPR Compliance:**
- **Data Residency:** All data stored in eu-central-1 (Frankfurt) region
- **Right to Erasure:** Automated deletion within 30 days (Lambda + DynamoDB TTL)
- **Consent Management:** User consent tracked in DynamoDB (booking service)
- **Data Minimization:** PII stored only where necessary (column-level encryption)

---

### 7. Threat Detection & Incident Response

**AWS GuardDuty**
- **Purpose:** Intelligent threat detection (ML-powered)
- **Data Sources:** VPC Flow Logs, CloudTrail logs, DNS logs
- **Detection:** 
  - Compromised instances (cryptocurrency mining, C&C communication)
  - Unusual API calls (account compromise, privilege escalation)
  - Unauthorized infrastructure changes
- **Cost:** $2,800/month (50K accounts × $0.05 + data volume)
- **Integration:** SNS → PagerDuty → On-call engineer

**AWS Security Hub**
- **Purpose:** Centralized security dashboard (aggregates findings)
- **Integrations:** GuardDuty, Macie, Inspector, Config, IAM Access Analyzer
- **Compliance Checks:** CIS AWS Foundations Benchmark, PCI-DSS
- **Workflow:** Prioritize findings by severity, assign to teams, track remediation
- **Cost:** $1,500/month

**Incident Response Playbook:**
```
1. Detection (GuardDuty alert → SNS → PagerDuty)
   ↓
2. Triage (Security Hub dashboard, CloudTrail logs)
   ↓
3. Containment (Isolate compromised resources, revoke credentials)
   ↓
4. Eradication (Terminate compromised instances, rotate secrets)
   ↓
5. Recovery (Restore from backups, validate integrity)
   ↓
6. Post-Incident (Root cause analysis, update runbooks)

Target MTTD (Mean Time to Detect): <24 hours
Target MTTR (Mean Time to Respond): <4 hours
```

---

### 8. Security Monitoring & Observability

**CloudWatch Logs**
- **Application Logs:** All microservices send structured logs (JSON format)
- **Audit Logs:** CloudTrail logs stored for 7 years
- **Security Logs:** VPC Flow Logs, WAF logs, GuardDuty findings
- **Retention:** 
  - Application logs: 30 days (cost optimization)
  - Audit logs: 7 years (compliance)
  - Security logs: 90 days (GDPR data minimization)

**VictoriaMetrics (Security Metrics)**
- **Failed Login Attempts:** Track 5+ failed logins per user (account takeover detection)
- **IAM Policy Changes:** Alert on privilege escalation attempts
- **Certificate Expiration:** Alert 30 days before IoT certificate expiration
- **API Gateway Errors:** 5xx errors spike (potential attack)

**Grafana Dashboards:**
- **Security Overview:** GuardDuty findings, WAF blocks, failed auth attempts
- **Compliance Status:** AWS Config rule compliance percentage
- **IoT Security:** Device Defender alerts, certificate expiration timeline
- **PCI-DSS Scope:** Payment service metrics, Stripe API latency, tokenization success rate

---

## Cost Breakdown (Security-by-Design)

| Security Component | Monthly Cost | Annual Cost | ROI |
|-------------------|-------------|-------------|-----|
| **AWS IAM + SSO** | $0 | $0 | Free tier |
| **AWS KMS** | $1,200 | $14,400 | Compliance requirement |
| **AWS Secrets Manager** | $500 | $6,000 | Prevents breaches |
| **AWS WAF** | $3,500 | $42,000 | Blocks 99% attacks |
| **AWS GuardDuty** | $2,800 | $33,600 | MTTD <24hr |
| **AWS Shield Standard** | $0 | $0 | Free tier |
| **AWS Certificate Manager** | $0 | $0 | Free tier |
| **VPC Flow Logs** | $1,500 | $18,000 | Compliance audit |
| **IoT Device Defender** | $5,000 | $60,000 | Prevents IoT attacks |
| **AWS CloudTrail** | $3,000 | $36,000 | GDPR/SOC 2 |
| **AWS Config** | $2,000 | $24,000 | Automated governance |
| **AWS Security Hub** | $1,500 | $18,000 | Unified visibility |
| **AWS Macie** | $3,500 | $42,000 | GDPR compliance |
| **TOTAL** | **$24,500** | **$294,000** | **6% of total spend** |

**Security ROI:**
- **Prevented Breach Cost:** €500,000 (average data breach cost)
- **Prevented GDPR Fine:** €10,000,000 (4% of annual revenue)
- **Net ROI:** €295,000/year positive (prevented loss - security cost)

---

## Alternatives Considered

### Alternative 1: Manual Security (No Automation)
- **Cost:** $8,000/month (manual audits, security engineers)
- **Pros:** Lower upfront tooling cost
- **Cons:** 
  - High risk of human error (misconfigured security groups)
  - Slow incident response (hours vs. minutes)
  - Non-compliant by default (no automated enforcement)
- **Rejected:** Unacceptable risk for IoT fleet + payment data

### Alternative 2: Third-Party Security Platform (e.g., Palo Alto Prisma Cloud)
- **Cost:** $35,000/month (cloud security posture management)
- **Pros:** Multi-cloud support, advanced threat detection
- **Cons:** 
  - 40% more expensive than native AWS tools
  - Vendor lock-in, integration complexity
  - Redundant with AWS-native features (GuardDuty, Config)
- **Rejected:** AWS-native tools provide better integration, lower cost

### Alternative 3: Hybrid (Some Automation, Some Manual)
- **Cost:** $15,000/month (partial automation + security engineers)
- **Pros:** Balanced approach, flexibility
- **Cons:** 
  - Inconsistent security posture (some services automated, some manual)
  - Still high risk of configuration drift
  - Slower incident response than full automation
- **Rejected:** Full automation required for 50K IoT devices + payment compliance

---

## Consequences

### Positive
1. **Zero-Trust Foundation:** All access authenticated + authorized (prevents 95% of breaches)
2. **Compliance Automation:** AWS Config, Macie, CloudTrail enforce policies (saves €132K/year in manual audits)
3. **Scalable Security:** Security tools scale with fleet growth (50K → 200K vehicles)
4. **Fast Incident Response:** GuardDuty + Security Hub enable <4hr MTTR (industry avg: 24hr)
5. **Cost-Optimized:** Free tier services (IAM, Shield, ACM) save $15K/month

### Negative
1. **Initial Complexity:** Zero-trust requires careful IAM policy design (2-3 months setup)
2. **Learning Curve:** Team must learn AWS security services (Config, GuardDuty, Macie)
3. **Monitoring Overhead:** Security logs generate 500GB/day (storage + analysis cost)
4. **Key Management:** KMS key rotation requires coordination across teams

### Mitigation
- **Training:** 2-week AWS security workshop for engineering team
- **Documentation:** Security runbooks for common tasks (rotate secrets, revoke access)
- **Automation:** Terraform modules for security group creation, IAM role templates
- **Monitoring:** Automated alerts reduce manual log analysis (GuardDuty, Security Hub)

---

## Compliance Validation

### GDPR
- ✅ Data residency: eu-central-1 (Frankfurt) region only
- ✅ Right to erasure: Automated deletion within 30 days
- ✅ Consent management: Tracked in DynamoDB
- ✅ Data minimization: Column-level encryption, PII stored only where necessary
- ✅ Breach notification: <72 hour notification via CloudTrail + GuardDuty

### PCI-DSS Level 1
- ✅ Network segmentation: Payment Service in dedicated VPC
- ✅ Encryption: TLS 1.3 in transit, AES-256 at rest
- ✅ Access control: IAM roles, MFA enforced
- ✅ Monitoring: CloudWatch Logs, WAF, GuardDuty
- ✅ Quarterly scans: AWS Inspector, third-party ASV

### ISO 27001
- ✅ Risk assessment: Threat modeling in ADR-12
- ✅ Access control: IAM policies, MFA, least-privilege
- ✅ Incident response: Playbook documented, tested quarterly
- ✅ Audit trail: CloudTrail logs, 7-year retention
- ✅ Continuous monitoring: AWS Config, Security Hub

### SOC 2 Type II (In Progress)
- ✅ Security: Zero-trust, encryption, monitoring
- ✅ Availability: Multi-region deployment (ADR-09)
- ✅ Processing Integrity: Data validation at API Gateway
- ✅ Confidentiality: KMS encryption, IAM access control
- ⏳ Privacy: GDPR compliance, consent management (Q2 2026)

---

## Related ADRs
- **[ADR-01: Microservices Architecture](ADR_01_microservices_architecture.md)** - Service isolation improves security blast radius containment
- **[ADR-03: Vehicle Telemetry](ADR_03_Vehicle_Telemetry.md)** - Edge device security, data encryption
- **[ADR-09: Multi-Region Deployment](ADR_09_MULTI_REGION.md)** - Disaster recovery, data residency
- **[ADR-11: IoT Security & Edge Computing](ADR_11_IoT_Enabled_Vehicles.md)** - X.509 certificates, mTLS, Device Defender
- **[ADR-14: Data Compliance](ADR_14_DATA_COMPLIANT.md)** - GDPR data residency, right-to-erasure
- **[ADR-15: Cloud Provider Selection](ADR_15_Cloud_Provider_Selection.md)** - AWS chosen for IoT security, compliance tools
- **[ADR-19: Payment Integration](ADR_19_Payment_Integration.md)** - PCI-DSS compliance via Stripe tokenization

---

**Document Version:** 1.0  
**Last Updated:** November 8, 2025  
**Status:** Production-Ready  
**Owner:** Security Team (Raj Patel - CISO)
