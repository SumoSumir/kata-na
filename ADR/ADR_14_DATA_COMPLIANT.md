# ADR-14: Data Residency and Classification Compliance

## Status
Accepted

## Context
MobilityCorp operates across multiple regions with strict data protection laws such as GDPR (Europe), FDPA (France) (Future with DPDP (India), and CCPA (US)).  
The platform processes diverse data types — user profiles, ride telemetry, and aggregated analytics — each with unique residency and replication requirements.  

Compliance and architecture must ensure:
- Data is stored and processed according to local laws.
- Personally identifiable information (PII) is never transferred outside the user’s originating region.
- Machine learning workflows only use anonymized or aggregated datasets.
- Deletion requests cascade across all regions and backups.

## Decision
Adopt a **data residency and classification policy** with region-scoped storage and processing rules.

### Data Residency Compliance

**GDPR (Europe)**
- **User Data Storage:** `eu-west-1` only  
- **Processing:** Allowed only within `eu-west-1`  
- **ML Training:** Only anonymized data can be exported to external regions example `us-east-1`  
- **Model Serving:** Inference runs locally in `eu-west-1`  
- **Deletion Requests:** Must delete records and derived artifacts from all regions  

Other regions (e.g., India, US) will follow equivalent local laws (DPDP, CCPA) with similar data boundary principles.

### Data Classification

**1. PII (Personal Identifiable Information)**  
Examples: Name, email, phone number, payment details  
- **Storage:** Restricted to user’s signup region  
- **Replication:** Strictly prohibited  
- **Access:** Allowed only for authorized services within that region  

**2. Operational Data**  
Examples: Vehicle locations, telemetry, battery status, booking IDs  
- **Storage:** In all active operational regions  
- **Replication:** Allowed across regional boundaries  
- **Use:** For fleet operations, maintenance, and dispatch optimization  

**3. Aggregated / Anonymized Data**  
Examples: Usage statistics, aggregated demand data  
- **Storage:** Allowed in all regions  
- **Replication:** Permitted globally for analytics and AI training  
- **Privacy:** Must ensure irreversible anonymization before cross-region transfer  

## Consequences

✅ **Positive**
- Ensures full compliance with regional data protection regulations  
- Reduces risk of legal violations and fines  
- Supports regional autonomy for data and ML workloads  
- Enables global insights through anonymized aggregation  

❌ **Negative**
- Increased complexity in storage and synchronization management  
- Higher infrastructure cost due to region-specific data silos  
- Requires rigorous governance and monitoring for data flows  
- Data deletion workflows become more complex and cross-regional  

## Alternatives Considered

**Centralized Global Data Lake**  
- Pros: Simpler to manage, unified analytics  
- Cons: Violates GDPR/DPDP compliance, risks data exposure  
- Rejected: Not compliant for user data or PII transfer  

**Partial Residency (Metadata Global, PII Local)**  
- Pros: Easier query federation  
- Cons: Still risky if metadata leaks sensitive associations  
- Rejected: Does not fully meet strict data residency and deletion guarantees  

## AI-Specific Considerations
- ML pipelines operate only on anonymized or aggregated datasets.  
- Region-specific training ensures compliance (e.g., EU-only data for EU models).  
- Cross-region model retraining uses privacy-preserving data exports.  
- Global analytics dashboards rely only on non-identifiable, aggregated data.  
- Automated deletion jobs ensure PII removal across training and inference stores.
