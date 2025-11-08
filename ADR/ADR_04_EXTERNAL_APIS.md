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
Implement **API Gateway + Lambda orchestrators** (see ADR-05 for orchestrator pattern) which integrate with external APIs and provide normalized data to our services.

**AWS Implementation:**

**1. Weather API Integration**
- **Primary:** OpenWeatherMap API
  - Endpoints: Current weather, 5-day forecast, historical
  - Rate limit: 1,000 calls/day (free tier) or 60 calls/min (paid: $40/month)
  - Data: Temperature, precipitation, wind speed, humidity, visibility
- **Fallback:** WeatherAPI.com
  - Endpoints: Real-time, forecast, historical
  - Rate limit: 1M calls/month ($4/month per 10K calls)
- **Architecture:**
  - API Gateway REST API → Lambda (Node.js 20.x, 512 MB RAM)
  - Lambda function fetches weather, normalizes to common schema, returns JSON
  - **Caching:** ElastiCache Redis (30-min TTL for current weather, 24-hour TTL for forecasts)
  - **Circuit breaker:** DynamoDB tracks provider health (switch to fallback if error rate > 10%)
  - **Historical storage:** Lambda → Kinesis Firehose → S3 (`s3://mobilit ycorp-external-data/weather/`)
- **Cost:** $50/month (API subscriptions) + $50/month (AWS infrastructure) = $100/month

**2. Public Holidays API Integration**
- **Primary:** Calendarific API
  - Endpoints: Holidays by country, year
  - Rate limit: 1,000 calls/month (free), 100K calls/month (paid: $10/month)
  - Covers 230+ countries with local and national holidays
- **Architecture:**
  - Lambda function triggered monthly (EventBridge cron: `0 0 1 * ? *`)
  - Fetches holidays for next 12 months, stores in DynamoDB `public_holidays` table
  - **Caching:** DynamoDB with no TTL (static data, refreshed monthly)
  - **Fallback:** Manual CSV upload to S3 if API unavailable
- **Cost:** $10/month (API) + $5/month (DynamoDB) = $15/month

**3. Events API Integration**
- **Primary:** PredictHQ API
  - Endpoints: Events by location, category, date range
  - Categories: Concerts, sports, festivals, conferences
  - Rate limit: 5,000 calls/month (starter: $199/month), 50K calls/month (growth: $499/month)
  - Provides: Event attendance estimates, category, venue coordinates
- **Fallback:** Ticketmaster Discovery API
  - Free tier: 5,000 calls/day
  - Less rich metadata than PredictHQ but sufficient for basic demand signals
- **Architecture:**
  - Lambda function triggered daily (EventBridge cron: `0 6 * * ? *`)
  - Fetches events for next 30 days within 5 km of each operational zone
  - Stores in DynamoDB `local_events` table with GSI on `zone_id` + `event_date`
  - **Enrichment:** Lambda adds geofence mapping (zone_id) using H3 hexagonal grid
  - **ML Feature Store:** Daily batch job syncs events to SageMaker Feature Store
- **Cost:** $199/month (PredictHQ starter) + $50/month (AWS infrastructure) = $249/month

**4. Maps & Traffic API Integration** (see ADR-05 for orchestrator details)
- **Primary:** Google Maps Platform
  - Distance Matrix API: ETA calculations for fleet rebalancing
  - Directions API: Optimal routes for relocation
  - Geocoding API: Address → coordinates conversion
  - Traffic API: Real-time traffic conditions
  - Rate limit: 40K requests/month (free), then $5 per 1,000 requests
- **Fallback:** Mapbox
  - Directions API: $0.50 per 1,000 requests (50% cheaper than Google)
  - Geocoding API: Free tier 100K requests/month
  - Traffic: Real-time traffic tiles included
- **Architecture:** 
  - API Gateway → Lambda (intelligent routing based on request type - see ADR-05)
  - **Caching:** ElastiCache Redis
    - Geocoding: 7-day TTL (addresses rarely change)
    - Traffic: 5-min TTL (real-time, but high query volume)
    - Directions: 1-hour TTL (routes stable unless traffic changes)
  - **Historical storage:** S3 for traffic patterns (used in demand forecasting)
- **Cost:** $8,000/month (Google Maps) + $2,000/month (Mapbox) = $10,000/month

**Unified Data Schema (JSON):**
```json
{
  "external_data_type": "weather",
  "timestamp": "2025-11-07T14:30:00Z",
  "location": {
    "zone_id": "89283472bffffff",  // H3 index
    "lat": 52.5200,
    "lon": 13.4050
  },
  "weather": {
    "temperature_celsius": 18,
    "precipitation_mm": 2.5,
    "wind_speed_kph": 15,
    "condition": "light_rain"
  },
  "events": [
    {
      "event_id": "evt_12345",
      "name": "Berlin Marathon",
      "category": "sports",
      "attendance": 45000,
      "start_time": "2025-11-07T09:00:00Z",
      "impact_radius_km": 5
    }
  ],
  "holidays": [
    {
      "date": "2025-11-07",
      "name": "German Unity Day",
      "type": "national"
    }
  ],
  "traffic": {
    "congestion_level": "moderate",  // low/moderate/heavy
    "avg_speed_kph": 35,
    "duration_in_traffic_multiplier": 1.4
  }
}
```

**Monitoring & Alerting:**
- **CloudWatch Metrics:** API response times, error rates, cost per provider
- **CloudWatch Alarms:** Alert if provider error rate > 5% for 10 mins
- **Cost tracking:** CloudWatch Logs Insights queries aggregate API costs daily
- **SNS notifications:** Alerts sent to #engineering-ops Slack channel

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
