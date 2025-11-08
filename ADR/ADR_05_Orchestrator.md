# ADR-05: Orchestrator 
      
## Status - accepted

### Context
Our business case requires frequent calls to various third-party services, including mapping APIs, payment gateways, and AI/LLM model providers. The following challenges motivate a robust architectural response:

Rapid evolution of external services (especially AI/LLMs, where “best” providers change fast)

Sudden provider price changes or new commercial terms

Potential provider shutdown or disruption (risk of business interruption)

To address these, we’re considering whether to introduce an independent orchestrator layer for each service that can:

Perform dynamic routing, switching providers as needed

Enable intelligent circuit breaking and fallback logic

Isolate business logic from provider specifics, maximizing flexibility


### Decision
We have accepted the idea of building an **independent orchestrator module** for each critical third-party service, which manages routing, circuit breaking, and provider selection based on business rules and system health.

**AWS Implementation:**

**1. AI/LLM Model Orchestrator**
- **LangChain + AWS Bedrock** (see [ADR-18](ADR_18_Agentic_AI_Framework.md) for agentic AI details)
  - **Primary:** AWS Bedrock (Claude models) for conversational AI
  - **Fallback:** OpenAI (via API Gateway → Lambda integration)
  - **Circuit breaker:** AWS Lambda with exponential backoff and Dead Letter Queue (SQS)
  - **Cost tracking:** CloudWatch Logs monitors token usage per provider
  - **Routing logic:** Lambda function checks availability and cost thresholds
- **Deployment:** ECS Fargate service for high-availability
- **Monitoring:** CloudWatch alarms on error rate, latency, and token cost

**2. Payment Gateway Orchestrator**
- **Primary:** Stripe (EU payments with strong 3DS support)
- **Fallback:** Adyen (backup for Stripe outages or compliance requirements)
- **Architecture:** Lambda function + Step Functions workflow
  - Attempts primary provider payment
  - Retries with fallback on failure
  - Publishes result to Kafka for event-driven processing
- **Circuit breaker:** DynamoDB tracks failure rate per provider for automatic switching

**3. Mapping API Orchestrator**
- **Primary:** Google Maps Platform (routing, geocoding, places)
- **Fallback:** Mapbox (backup and cost optimization for simpler queries)
- **Use case routing:**
  - Route optimization: Google Maps (superior traffic data)
  - Geocoding: Mapbox first (cost-effective), Google fallback
  - Static maps: Mapbox (cost optimization)
- **Architecture:** API Gateway → Lambda (provider selection logic) → External API
- **Caching:** ElastiCache Redis for frequently accessed data

**4. Weather/Events API Orchestrator** (see [ADR-04](ADR_04_EXTERNAL_APIS.md))
- **Weather:** OpenWeatherMap primary, WeatherAPI.com fallback
- **Events:** PredictHQ primary, Ticketmaster fallback
- **Architecture:** API Gateway → Lambda with intelligent routing
- **Caching:** Aggressive caching strategy for static/semi-static data

**Cross-Cutting Orchestrator Patterns:**
- **Circuit breaker state machine (DynamoDB):**
  - `CLOSED` → Normal operation
  - `OPEN` → Provider unavailable, use fallback immediately
  - `HALF_OPEN` → Test provider recovery periodically
- **Health checks:** Lambda functions ping providers regularly via EventBridge
- **Centralized monitoring:** CloudWatch dashboard showing:
  - Provider availability and uptime metrics
  - Request distribution (primary vs. fallback traffic)
  - Cost tracking per provider
  - Latency percentiles and performance metrics

### Consequences
Trade-offs and Impacts:

Scalability and Flexibility:
We gain agility in switching between providers as technologies or commercial terms evolve.

Resilience:
Orchestrators enable graceful degradation—automatic fallback across providers if one becomes unavailable.

Increased Complexity:
Implementing and maintaining orchestrators adds architectural complexity and requires careful design, especially around consistency, monitoring, and reliability.

Cost:
Orchestrator logic may increase development and operational cost compared to a simple integration.

Alternatives and Their Pros & Cons
| Approach                        | Pros                                                                 | Cons                                                                                             |
|----------------------------------|---------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| Direct Integration               | - Simpler architecture<br>- Lower initial cost                      | - Vendor lock-in<br>- Poor resilience if provider fails<br>- Difficult to switch providers quickly |
| Client-Side Logic                | - Some flexibility (custom fallback, e.g. in UI)<br>- Lightweight   | - Limited scalability<br>- Harder to enforce business rules centrally                            |
| Orchestrator (Accepted)          | - Centralized routing and fallback<br>- Dynamic provider selection<br>- Business logic separation | - Higher complexity<br>- Needs more monitoring and testing<br>- May add latency if not well-designed |
| API Gateway                      | - Central control over requests<br>- Some resilience features<br>- Easier to apply authentication, throttling | - Not business-logic-aware<br>- Less granular routing/circuit breaking compared to dedicated orchestrator |
| Service Mesh (e.g. Istio, Linkerd)| - Advanced traffic management<br>- Circuit breaking/failover at infra level | - Too heavy if your rules are business/domain-specific<br>- Steep learning curve<br>- Might not suit external API integrations |



### Summary:
The orchestrator approach is chosen for its flexibility and resilience in a rapidly changing service landscape. While it introduces complexity, it provides critical agility and helps future-proof our architecture against provider changes and failures.

If the need for business-logic-driven routing is low or switching providers is rare, simpler integrations (direct or gateway) could suffice; but with fast-moving AI/model providers and frequent business rule updates, orchestrators offer the most control with manageable trade-offs.
