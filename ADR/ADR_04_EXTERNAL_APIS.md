# ADR-04: Integration with External APIs (Weather, Holidays, Events)

## Status
Accepted  

## Context
AI demand forecasting and operational intelligence require external context data:
- **Weather data:** Affects demand, battery drain, and vehicle damage rates.
- **Public holidays:** Influences weekday/weekend demand patterns.
- **Local events:** Concerts, sports, and festivals cause predictable spikes in demand.
- **Map and Traffic data:** Helps predict demand (e.g., high traffic encourages two-wheeler usage) and estimate vehicle arrival times for relocation, preventing delays for waiting customers.

Requirements:
- Unified abstraction layer over multiple APIs  
- Retry, fallback, and caching mechanisms  
- Historical data availability for model retraining  
- Normalized schema across providers  
- Fault tolerance and cost optimization

Alternatives considered:
1. **Third-party data aggregators:** Managed feeds, but expensive and inflexible.  
2. **Static offline datasets:** No API dependency, but outdated and not real-time.
3. **Build in-house data source:** Fetches and curates data internally. High initial effort but offers maximum control and avoids licensing fees.
4. **Orchestrator-based integration:** As detailed in [ADR-05](./ADR_05_Orchestrator.md), an orchestrator could manage API calls, but this might centralize failure points.

## Decision
Implement **orchestrators** (see [ADR-05](ADR_05_Orchestrator.md) for orchestrator pattern) which integrate with external APIs and provide normalized data to our services while dynamically scaling up/down the service based on requests in queue using KEDA.

**Unified Data Schema:**
External data normalized into consistent format:
- Data type classification (weather/events/traffic/holidays)
- Timestamp and location (H3 index, lat/lon)
- Type-specific attributes (temperature, event details, traffic metrics)
- Enables consistent processing across all AI/ML models and services

## Consequences
✅ **Positive:**
- **Consistency:** Unified schema across providers and models for consistent access by ML and analytics services.
- **Reliability:** Resilient against provider outages.
- **Efficiency:** Cached data reduces API cost and latency.
- **Reusability:** Enables historical reprocessing for retraining.

❌ **Negative:**
- **Data Staleness:** Caching might introduce data staleness.
- **Infrastructure Cost:** Requires a dedicated cache and aggregator service.
- **Provider Reliability:** Data quality can be inconsistent across different APIs.
- **API Switching Costs:** Migrating to a new provider requires complex data backfilling.

AI-Specific Impacts:
- Weather and event data used as regressors in demand and battery models
- Storage of API responses allows historical archives enable model drift analysis and retraining consistency
