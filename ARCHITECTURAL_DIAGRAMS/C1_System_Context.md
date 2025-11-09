# C1: System Context Diagram

This diagram provides a high-level overview of the MobilityCorp platform and its interactions with external users and systems.

[![](https://img.plantuml.biz/plantuml/svg/ZLR1RkCs4BtxAwOvR1siirwWsKjH5ElOsIHed7XZfK5F0IsDPIAKA92K4-_Nzv6KR3dnjhdO56czUJFpvj5VhHF6rONg_IBBLDKPqn_Zq-7uz76iIySlxizUpqcdcEORwxYWPfjEQAeNKact6MjJqKm9jzIcwFL-fHaRgqjBmf7JLIy-zjFQEbsm6T3Hk5aUOHrGc4HLKMtPq4Bh9rdc9CeC6twrvfLC5LlgV_dy-I-QZEwFZuvRLD1VBW4pzzyUxxPY8vpqyHsBWWfHYjp3B1MpexKKb0cxMcXXiXsCo0fPuksH__OmJRPXNre7I6ti3YpLvLBcTOoqWyc44pQLNAOCZ0cUQ1uVAG6PaEyrSGaCnXjDWjzOkDgWmeZ18N74GrWS5JfZPH40PPrYZspaXLNAQ8JfHQBzggb_Mz7NVGcXPMeqQhoMAQhvaKPNGx3vIETPpkXhKTNEnyJ66jWd4oFnJIEJhgJZr5Fyr7ArlzCLmYibS_oKRhIJIvd6kWzeiYb58LEQ6Pa2SU15iqQ-uLme1AR-6MT6fq_7WnWUHFfpiMItEHaRwO2bS3IjbPCdTvpZFUgVXyQZ4kSloDm3nMe4fIsqWwQQSZpY5tsf7PA-qWdztIZ85-4B91Iby2pT2XfHNB0pcm5TpEOTCgUVVZiZzeMmmqniVEGETYKs1Ow0DuihkX2EdyKcn9Wx8okcFl8lXDaWaL6sGIdxtuHI2u6izz4AKLdV7r7Hl6cFXxdGEiTeJCCfFXVwXVgfjevqvMGXVuHwli9wHeDMOSeUuee-n7vQ6iskR3CYPNC8r07Qn9bCtUNtVII8iBGov5Wl5DgPn8qjJkmE98fE-8m98YozRFTnKYXBIH5dOhnz0AzHdKaNCftCxhpeqwSW5-7tmn04xNTxMbeCNOIwQjTRGcCb9DJmWTP2oQoTpSxhkT5rcJbJkvLlayXvAfvGeekmZmHnq0of7sLIOi4g9WJPtR6A2b_9oi80uLUKIOFHYB9U2uFCK6WC72tOGQJUA9pHgeTt7oVwkMk3dP4zYwOteBfK60gSjVfQCcTUCPxLPPBCviDec63a8U-hpkHt77OEtJagzySIJ7K1s5YPKB5PRLV3-NJk4znIR3ormoyHzYcQ7qTQ1Rp8SHcghtHkT_GUU768M-lJ8yN6WOkmLkQb1yD38sVE27rm_0PjHwtnwGwrngWjoeqA0YpS8aLUU9QwbF1Kty3_uzTU07TSwK1mUqbuie7qprdjsNw7sojZJp7NTX0zVMln7XNw7OumZgKuH6t_Ke4qCGGoNxajbFSlcTe6oJCxarjkEvpeXqbhUDIVVa-IZl_6ksP0jyc8wWfHOoFyhuRUHN6VUDaTn7xhXnNS7UdkrJwoEmZGME2T3dVGW7BMgSxU-_PRq_D56qJ_3kjtmUmShxKwcwuugt5r3Dv8C-8SGFcPtvbMMj7UziVcFrWTzhoEnRq7PD_valR_HInJiBatP7hNetzkxvF7v7RsEBbzkEaVjpiFLydbu_NvnVdD19kzhx0K_8Fy5m00)](https://editor.plantuml.com/uml/ZLR1RkCs4BtxAwOvR1siirwWsKjH5ElOsIHed7XZfK5F0IsDPIAKA92K4-_Nzv6KR3dnjhdO56czUJFpvj5VhHF6rONg_IBBLDKPqn_Zq-7uz76iIySlxizUpqcdcEORwxYWPfjEQAeNKact6MjJqKm9jzIcwFL-fHaRgqjBmf7JLIy-zjFQEbsm6T3Hk5aUOHrGc4HLKMtPq4Bh9rdc9CeC6twrvfLC5LlgV_dy-I-QZEwFZuvRLD1VBW4pzzyUxxPY8vpqyHsBWWfHYjp3B1MpexKKb0cxMcXXiXsCo0fPuksH__OmJRPXNre7I6ti3YpLvLBcTOoqWyc44pQLNAOCZ0cUQ1uVAG6PaEyrSGaCnXjDWjzOkDgWmeZ18N74GrWS5JfZPH40PPrYZspaXLNAQ8JfHQBzggb_Mz7NVGcXPMeqQhoMAQhvaKPNGx3vIETPpkXhKTNEnyJ66jWd4oFnJIEJhgJZr5Fyr7ArlzCLmYibS_oKRhIJIvd6kWzeiYb58LEQ6Pa2SU15iqQ-uLme1AR-6MT6fq_7WnWUHFfpiMItEHaRwO2bS3IjbPCdTvpZFUgVXyQZ4kSloDm3nMe4fIsqWwQQSZpY5tsf7PA-qWdztIZ85-4B91Iby2pT2XfHNB0pcm5TpEOTCgUVVZiZzeMmmqniVEGETYKs1Ow0DuihkX2EdyKcn9Wx8okcFl8lXDaWaL6sGIdxtuHI2u6izz4AKLdV7r7Hl6cFXxdGEiTeJCCfFXVwXVgfjevqvMGXVuHwli9wHeDMOSeUuee-n7vQ6iskR3CYPNC8r07Qn9bCtUNtVII8iBGov5Wl5DgPn8qjJkmE98fE-8m98YozRFTnKYXBIH5dOhnz0AzHdKaNCftCxhpeqwSW5-7tmn04xNTxMbeCNOIwQjTRGcCb9DJmWTP2oQoTpSxhkT5rcJbJkvLlayXvAfvGeekmZmHnq0of7sLIOi4g9WJPtR6A2b_9oi80uLUKIOFHYB9U2uFCK6WC72tOGQJUA9pHgeTt7oVwkMk3dP4zYwOteBfK60gSjVfQCcTUCPxLPPBCviDec63a8U-hpkHt77OEtJagzySIJ7K1s5YPKB5PRLV3-NJk4znIR3ormoyHzYcQ7qTQ1Rp8SHcghtHkT_GUU768M-lJ8yN6WOkmLkQb1yD38sVE27rm_0PjHwtnwGwrngWjoeqA0YpS8aLUU9QwbF1Kty3_uzTU07TSwK1mUqbuie7qprdjsNw7sojZJp7NTX0zVMln7XNw7OumZgKuH6t_Ke4qCGGoNxajbFSlcTe6oJCxarjkEvpeXqbhUDIVVa-IZl_6ksP0jyc8wWfHOoFyhuRUHN6VUDaTn7xhXnNS7UdkrJwoEmZGME2T3dVGW7BMgSxU-_PRq_D56qJ_3kjtmUmShxKwcwuugt5r3Dv8C-8SGFcPtvbMMj7UziVcFrWTzhoEnRq7PD_valR_HInJiBatP7hNetzkxvF7v7RsEBbzkEaVjpiFLydbu_NvnVdD19kzhx0K_8Fy5m00)


## Key External Actors:

### Human Users
-   **Customers:** Mobile app users booking and using vehicles (500K+ daily active users)
-   **Staff:** Operations team managing the fleet via web dashboard (task mgmt, maintenance, incidents)
-   **Admin:** System administrators managing platform configuration, user roles, and monitoring
-   **Data Science Team:** ML engineers and analysts using Feature Store and running model training

### External Systems

#### Core Integrations
-   **IoT Vehicles (50K fleet):** Electric scooters, eBikes, cars, and vans with embedded sensors
    - Real-time telemetry (GPS, battery, IMU @ 4.3B events/day)
    - Edge ML models (collision detection, geofence, tamper)
    - Bi-directional communication via AWS IoT Core

-   **Payment Gateways:** Multi-provider strategy with orchestration
    - Primary: Stripe (EU payments, 3DS support)
    - Fallback: Adyen (backup & compliance)
    
-   **Map Services:** Dual-provider for cost optimization & resilience
    - Google Maps Platform (routing, traffic, places)
    - Mapbox (geocoding, static maps - 50% cost savings)

#### Contextual Data Providers
-   **Weather APIs:** OpenWeatherMap (primary), WeatherAPI.com (fallback)
-   **Events APIs:** PredictHQ (concerts, sports, festivals)
-   **Public Holidays:** Calendarific API
-   **Public Transit APIs:** Multi-modal trip planning integration
    
#### Compliance & Operations
-   **Compliance & Audit Systems:** GDPR compliance tracking, audit trails, data retention
-   **Insurance Provider APIs:** Vehicle insurance validation and claims
-   **Ground Truth Labeling (SageMaker):** Continuous ML model improvement via human labeling

## System Responsibilities:

The MobilityCorp Platform handles:
- Vehicle booking, unlock/lock, trip tracking
- AI-driven dynamic pricing and relocation incentives
- Real-time demand forecasting and fleet optimization
- Predictive maintenance and incident management
- Multi-language conversational AI assistant
- Payment processing with multi-provider failover
- Edge computing for safety-critical operations
- Multi-region deployment (EU: Frankfurt, Ireland)
- Event-driven architecture with Kafka event bus
- Data lakehouse (Bronze/Silver/Gold medallion)
- MLOps pipeline (training, deployment, monitoring)

## Key Metrics & Scale

| Metric | Value | Details |
|--------|-------|---------|
| **Daily Active Users** | 500,000 | Peak usage during commute hours (7-9 AM, 5-7 PM) |
| **Fleet Size** | 50,000 vehicles | Mixed fleet: eBikes (40%), eScooters (35%), eCars (20%), eVans (5%) |
| **Geographic Coverage** | 25 EU cities | Primary: Frankfurt, Paris, Amsterdam, Berlin, Dublin |
| **Telemetry Volume** | 4.3B events/day | Real-time GPS, battery, IMU data @ 1Hz-100Hz |
| **Daily Bookings** | 500,000 transactions | Average trip duration: 18 minutes |
| **Availability SLA** | 99.9% uptime | <43 minutes downtime/month |
| **API Request Volume** | 10,000 req/sec | Peak: 15,000 req/sec during events |
| **ML Models in Production** | 14 models | 11 cloud + 3 edge models |
| **Edge Devices** | 50,000 IoT devices| Real-time safety-critical inference |
| **Multi-Region** | 8 active regions | Separate deployments in countries with localisation requirements |
| **Data Lakehouse** | 2.5 PB | Bronze (raw) → Silver (clean) → Gold (aggregated) |
| **Event Streaming** | 130B MQTT msg/month | Kafka 3 brokers, 50 partitions/topic |

## Architecture Characteristics

### Quality Attributes

| Attribute | Requirement | Implementation |
|-----------|-------------|----------------|
| **Scalability** | Handle 3x traffic spikes during events | Auto-scaling EKS, ElastiCache, multi-region |
| **Reliability** | 99.9% uptime | Multi-AZ, multi-region, circuit breakers |
| **Performance** | API p99 <500ms, ML inference <200ms | Feature Store caching, SageMaker endpoints |
| **Security** | GDPR compliant, zero-trust | OAuth2/JWT, encryption at rest/transit, audit logs |
| **Observability** | Full distributed tracing | OpenTelemetry, CloudWatch, Grafana, PagerDuty |
| **Cost Efficiency** | <$0.15 per trip | Spot instances, serverless, multi-provider optimization |
| **Maintainability** | Independent service deployments | Microservices, CI/CD, feature flags |
| **Evolvability** | Add new ML models without downtime | MLOps pipelines, canary deployments |

### Technology Stack Summary

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React Native (mobile), React (web dashboards) |
| **API Gateway** | AWS API Gateway, Kong (hybrid) |
| **Compute** | EKS, AWS Lambda, EC2 (ML training) |
| **Event Streaming** | Apache Kafka, EKS |
| **Databases** | PostgreSQL, DynamoDB, ElastiCache Redis, TimescaleDB |
| **Data Lake** | S3 + Delta Lake (Bronze/Silver/Gold) |
| **ML Platform** | SageMaker (training, endpoints, pipelines, Feature Store) |
| **AI/LLM** | AWS Bedrock (Claude 3.5 Sonnet), OpenAI GPT-4o (fallback) |
| **Edge Computing** | TensorFlow Lite |
| **Observability** | OpenTelemetry, VictoriaMetrics, Grafana, CloudWatch, OpenSearch |
| **Orchestration** | Airflow, Temporal |
| **CI/CD** | GitHub Actions, ArgoCD |

## Related Documents

- [C2: Container Diagram](C2_Container_UPDATED.md) - Detailed service architecture
- [C3: AI/ML Component Diagram](C3_Component_AIML_UPDATED.md) - ML model architecture
- [ADR-01: Microservices Architecture](./ADR/ADR_01_microservices_architecture.md)
- [ADR-09: Multi-Region Deployment](./ADR/ADR_09_MULTI_REGION.md)
