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
Implement **API Gateway + Lambda orchestrators** (see [ADR-05](ADR_05_Orchestrator.md) for orchestrator pattern) which integrate with external APIs and provide normalized data to our services.

**AWS Implementation:**

**1. Weather API Integration**
- **Primary Provider:** OpenWeatherMap API
  - Data: Current weather, forecasts, temperature, precipitation, wind, humidity, visibility
- **Fallback Provider:** WeatherAPI.com for redundancy
- **Architecture:**
  - API Gateway REST API → Lambda orchestrator
  - Lambda fetches weather data and normalizes to common schema
  - **Caching:** ElastiCache Redis for current weather and forecasts
  - **Circuit breaker:** DynamoDB tracks provider health for automatic fallback
  - **Historical storage:** Kinesis Firehose → S3 for ML training data

**2. Public Holidays API Integration**
- **Primary Provider:** Calendarific API
  - Covers 230+ countries with local and national holidays
- **Architecture:**
  - Lambda function triggered monthly via EventBridge
  - Fetches holidays for upcoming months, stores in DynamoDB
  - **Caching:** DynamoDB (static data, refreshed monthly)
  - **Fallback:** Manual CSV upload to S3 if API unavailable

**3. Events API Integration**
- **Primary Provider:** PredictHQ API
  - Event types: Concerts, sports, festivals, conferences
  - Provides attendance estimates, venue coordinates, category
- **Fallback Provider:** Ticketmaster Discovery API
- **Architecture:**
  - Lambda function triggered daily via EventBridge
  - Fetches upcoming events for operational zones
  - Stores in DynamoDB with geofence mapping using H3 hexagonal grid
  - **ML integration:** Syncs to SageMaker Feature Store for demand forecasting

**4. Maps & Traffic API Integration** (see [ADR-05](ADR_05_Orchestrator.md) for orchestrator details)
- **Primary Provider:** Google Maps Platform
  - Distance Matrix API: ETA calculations for fleet rebalancing
  - Directions API: Optimal routes for relocation
  - Geocoding API: Address → coordinates conversion
  - Traffic API: Real-time traffic conditions
- **Fallback Provider:** Mapbox
  - Cost-effective alternative with comparable functionality
- **Architecture:** 
  - API Gateway → Lambda with intelligent routing based on request type
  - **Caching:** ElastiCache Redis with tiered TTL strategy
    - Geocoding: Long TTL (addresses rarely change)
    - Traffic: Short TTL (real-time data)
    - Directions: Medium TTL (routes stable unless traffic changes)
  - **Historical storage:** S3 for traffic patterns used in demand forecasting

**Unified Data Schema:**
External data normalized into consistent format:
- Data type classification (weather/events/traffic/holidays)
- Timestamp and location (H3 index, lat/lon)
- Type-specific attributes (temperature, event details, traffic metrics)
- Enables consistent processing across all AI/ML models and services

**Monitoring & Alerting:**
- **CloudWatch Metrics:** API response times, error rates, cost tracking
- **CloudWatch Alarms:** Alert on provider errors or degraded performance
- **SNS notifications:** Operational alerts for engineering team

## Consequences
✅ **Positive:**
- **Consistency:** Unified schema across providers and models.
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
- Historical archives enable model drift analysis and retraining consistency  
