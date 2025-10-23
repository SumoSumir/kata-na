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
1. **Direct calls from ML pipelines:** Simple, but inconsistent and prone to rate-limiting.
2. **Third-party data aggregators:** Managed feeds, but expensive and inflexible.  
3. **Static offline datasets:** No API dependency, but outdated and not real-time.
4. **Build in-house data source:** Fetches and curates data internally. High initial effort but offers maximum control and avoids licensing fees.
5. **Orchestrator-based integration:** As detailed in [ADR-05](./ADR_05_Orchestrator.md), an orchestrator could manage API calls, but this might centralize failure points.

## Decision
Implement a **Weather & Events Aggregation Service (WEA Service)** that integrates with external APIs and provides normalized data to AI services.

Key design:
- **Providers:** OpenWeatherMap (primary), Tomorrow.io (fallback); Calendarific for holidays; PredictHQ for events.  
- **Caching:** Redis for low-latency responses, S3 for historical storage. The caching strategy will be tiered:
    - **Real-time data (e.g., traffic, current weather):** Fetched frequently with a short TTL (Time To Live).
    - **Semi-static data (e.g., event schedules, weather forecasts):** Fetched less often, with a medium TTL.
    - **Static data (e.g., public holidays):** Fetched infrequently with a long TTL.
- **Fallback logic:** Circuit breaker pattern for failed providers.  
- **Normalized schema:** For consistent access by ML and analytics services.

## Consequences
✅ **Positive:**
- Consistent schema across providers and models  
- Reliable and resilient against provider outages  
- Cached data reduces cost and latency  
- Supports historical reprocessing for retraining  

❌ **Negative:**
- **Data Freshness:** Caching might introduce data staleness.
- **Infrastructure Cost:** Requires a dedicated cache and aggregator service.
- **Provider Reliability:** Data quality can be inconsistent across different APIs.
- **API Switching Costs:** Migrating to a new provider requires complex data backfilling.

AI-Specific Impacts:
- Weather and event data used as regressors in demand and battery models  
- Metadata supports augmentations in vision models (e.g., rain/fog)  
- Historical archives enable model drift analysis and retraining consistency  
