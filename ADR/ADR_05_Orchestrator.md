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
- **LangChain + AWS Bedrock** (see ADR-18 for agentic AI details)
  - **Primary:** AWS Bedrock (Claude 3.5 Sonnet) for conversational AI
  - **Fallback:** OpenAI GPT-4o (via API Gateway → Lambda integration)
  - **Circuit breaker:** AWS Lambda with exponential backoff and DLQ (SQS)
  - **Cost tracking:** CloudWatch Logs + Lambda extension records tokens used per provider
  - **Routing logic:** Python Lambda function (512 MB RAM, 30s timeout)
    ```python
    # Simplified routing logic
    if bedrock_available and cost_per_token < threshold:
        return invoke_bedrock(prompt)
    elif openai_available:
        return invoke_openai(prompt)
    else:
        raise ServiceUnavailableError()
    ```
- **Deployment:** ECS Fargate service (2 vCPU, 4GB RAM, 2 tasks min)
- **Monitoring:** CloudWatch alarms on error rate, latency, token cost

**2. Payment Gateway Orchestrator**
- **Primary:** Stripe (EU payments, strong 3DS support)
- **Fallback:** Adyen (backup for Stripe outages or EU compliance)
- **Architecture:** Lambda function + Step Functions workflow
  - Step 1: Attempt Stripe payment
  - Step 2: If Stripe fails (5xx, timeout), retry with Adyen
  - Step 3: Send result to Kafka `payments.completed` topic
- **Circuit breaker:** DynamoDB tracks failure rate per provider (5-min windows)
  - If Stripe error rate > 10% for 5 mins → auto-switch to Adyen
- **Cost:** ~$200/month (Lambda + Step Functions)

**3. Mapping API Orchestrator**
- **Primary:** Google Maps Platform (routing, geocoding, places)
- **Fallback:** Mapbox (backup for cost optimization on simpler queries)
- **Use case routing:**
  - **Route optimization:** Always use Google Maps (superior traffic data)
  - **Geocoding:** Use Mapbox first (cheaper), fallback to Google if quality low
  - **Static maps:** Always use Mapbox (50% cheaper)
- **Architecture:** API Gateway → Lambda (provider selection logic) → External API
  - **API Gateway:** REST API with usage plans and API keys
  - **Lambda:** Node.js 20.x, 1GB RAM, intelligent routing based on request type
  - **Caching:** ElastiCache Redis caches geocoding results (TTL = 7 days)
- **Cost optimization:** $8K/month (Google Maps) + $2K/month (Mapbox) vs. $15K/month (Google only)

**4. Weather/Events API Orchestrator** (ADR-04)
- **Weather:** OpenWeatherMap primary, WeatherAPI.com fallback
- **Events:** PredictHQ primary, Ticketmaster fallback
- **Architecture:** Similar to Mapping API (API Gateway → Lambda → External)
- **Caching:** Aggressive caching (weather: 30 mins, events: 24 hours)

**Cross-Cutting Orchestrator Patterns:**
- **Circuit breaker state machine (DynamoDB):**
  - `CLOSED` → Normal operation
  - `OPEN` → Provider unavailable, use fallback immediately
  - `HALF_OPEN` → Test if provider recovered (every 5 mins)
- **Health checks:** Lambda functions ping providers every 1 min via EventBridge
- **Centralized monitoring:** CloudWatch dashboard showing:
  - Provider availability (% uptime per 5-min window)
  - Request distribution (% traffic to primary vs. fallback)
  - Cost per provider (tracked via CloudWatch Logs Insights)
  - Latency percentiles (p50, p95, p99)

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
