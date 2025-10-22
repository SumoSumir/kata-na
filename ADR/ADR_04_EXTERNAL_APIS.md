# ADR-04: Integration with External APIs (Weather, Holidays, Events)

## Status
Accepted  
**Date:** 2024-10-22

## Context
AI demand forecasting and operational intelligence require external context data:
- Weather (affects demand, battery drain, and damage rates)
- Public holidays (influence on weekday/weekend demand patterns)
- Local events (concerts, sports, festivals causing spikes in demand)

Requirements:
- Unified abstraction layer over multiple APIs  
- Retry, fallback, and caching mechanisms  
- Historical data availability for model retraining  
- Normalized schema across providers  
- Fault tolerance and cost optimization

Alternatives considered:
1. **Direct calls from ML pipelines:** Simple, but inconsistent and rate-limit prone  
2. **Third-party data aggregators:** Managed feeds, but expensive and inflexible  
3. **Static offline datasets:** No API dependency, but outdated and not real-time  

## Decision
Implement a **Weather & Events Aggregation Service (WEA Service)** that integrates with external APIs and provides normalized data to AI services.

Key design:
- Providers: OpenWeatherMap (primary), Tomorrow.io (fallback); Calendarific for holidays; PredictHQ for events  
- Caching: Redis for low-latency responses, S3 for historical storage  
- Fallback logic: Circuit breaker pattern for failed providers  
- Normalized schema for consistent access by ML and analytics services

## Consequences
✅ **Positive:**
- Consistent schema across providers and models  
- Reliable and resilient against provider outages  
- Cached data reduces cost and latency  
- Supports historical reprocessing for retraining  

❌ **Negative:**
- Possible data staleness due to caching  
- Slightly higher infra overhead (cache + aggregator)  
- Data quality variance across providers  
- Complex provider backfill when switching APIs  

AI-Specific Impacts:
- Weather and event data used as regressors in demand and battery models  
- Metadata supports augmentations in vision models (e.g., rain/fog)  
- Historical archives enable model drift analysis and retraining consistency  
