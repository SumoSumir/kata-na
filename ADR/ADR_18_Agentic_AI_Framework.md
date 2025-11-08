# ADR-18: Agentic AI Framework Selection - LangChain on AWS Bedrock

## Status
Accepted

## Context
MobilityCorp's AI-enabled platform requires conversational AI capabilities for:

1. **User Assistant** - Help customers find vehicles, plan trips, answer questions
2. **Operations Copilot** - Assist staff with fleet management, maintenance decisions
3. **Smart Recommendations** - Suggest optimal routes, pricing, pickup/dropoff locations

These AI agents must:
- Understand natural language (text + voice)
- Access real-time data (vehicle availability, weather, traffic)
- Make multi-step decisions
- Use tools (APIs, databases, calculators)
- Provide explainable recommendations
- Support multiple languages (English, German, French, Italian, Spanish)
- Operate reliably at scale (100K+ users)

## Requirements

### Functional
- Multi-turn conversations with context retention
- Tool/function calling for external data access
- RAG (Retrieval-Augmented Generation) for knowledge base queries
- Streaming responses for better UX
- Support for multiple LLM providers (avoid lock-in)

### Non-Functional
- **Latency:** Fast response time
- **Accuracy:** High intent classification accuracy
- **Cost:** Cost-effective per-conversation economics
- **Scalability:** Handle high concurrent conversations
- **Reliability:** High uptime for AI services

### Use Cases

#### 1. User Trip Planning Assistant
```
User: "I need to get to Frankfurt Airport by 6 AM tomorrow"
Agent:
  1. Check current location (GPS API)
  2. Calculate route options (Mapbox API)
  3. Check vehicle availability (Fleet Service API)
  4. Consider weather forecast (Weather API)
  5. Recommend: "eBike to train station (15 min), then train"
  6. Offer booking: "Shall I reserve an eBike at Station X?"
```

#### 2. Battery Swap Prioritization Copilot
```
Staff: "Which vehicles need battery swaps most urgently?"
Agent:
  1. Query predictive maintenance model (SageMaker)
  2. Get demand forecast for upcoming period (Redshift)
  3. Check current vehicle locations (DynamoDB)
  4. Calculate optimal route (OR-Tools API)
  5. Respond: "Top 5 priority vehicles: [list with reasons]"
  6. Generate route map
```

#### 3. Dynamic Pricing Explanation
```
User: "Why is this car €45/hour?"
Agent:
  1. Query pricing model (SageMaker)
  2. Get current demand/supply (Redis cache)
  3. Check user's booking history (Aurora)
  4. Explain: "High demand (concert nearby) + low supply (3 cars available) + peak hour"
  5. Suggest: "€30/hour if you book for 8 PM instead"
```

## Alternatives Considered

### Alternative 1: LangChain on AWS Bedrock (Recommended)

#### Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                   AGENTIC AI ARCHITECTURE                        │
│                   (LangChain + AWS Bedrock)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 1. USER INPUT (Multi-Modal)                              │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ Text: Mobile app chat interface                      │  │
│  │  ├─ Voice: AWS Transcribe → Text                         │  │
│  │  └─ Context: User ID, location, session history          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 2. CONVERSATION ORCHESTRATOR (LangChain Agent)           │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ Memory: DynamoDB (conversation history)              │  │
│  │  ├─ LLM: AWS Bedrock (Claude 3.5 Sonnet)                │  │
│  │  ├─ Prompt Templates: S3 (versioned)                     │  │
│  │  └─ Agent Type: ReAct (Reasoning + Acting)               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 3. TOOL EXECUTION LAYER                                  │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  Tools (LangChain Tools):                                │  │
│  │  ├─ get_vehicle_availability(zone_id, vehicle_type)      │  │
│  │  ├─ calculate_route(origin, destination, mode)           │  │
│  │  ├─ get_weather_forecast(location, time)                │  │
│  │  ├─ search_knowledge_base(query) [RAG]                   │  │
│  │  ├─ get_demand_forecast(zone_id, time)                   │  │
│  │  ├─ calculate_dynamic_price(params)                      │  │
│  │  ├─ book_vehicle(vehicle_id, user_id, duration)          │  │
│  │  └─ get_user_history(user_id)                            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 4. KNOWLEDGE BASE (RAG)                                  │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ Vector Store: Amazon OpenSearch Serverless          │  │
│  │  ├─ Embeddings: AWS Bedrock (Titan Embeddings)          │  │
│  │  ├─ Documents:                                            │  │
│  │  │  ├─ FAQs (How to unlock, pricing, areas)             │  │
│  │  │  ├─ Vehicle manuals (specifications, features)        │  │
│  │  │  ├─ Policies (cancellation, fines, insurance)        │  │
│  │  │  └─ City guides (parking, traffic rules)             │  │
│  │  └─ Refresh: Weekly via Glue ETL                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 5. RESPONSE GENERATION                                   │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ LLM generates response (streaming)                   │  │
│  │  ├─ Content filter (AWS Bedrock Guardrails)             │  │
│  │  ├─ Translate if needed (AWS Translate)                 │  │
│  │  ├─ Text-to-Speech (AWS Polly) if voice output          │  │
│  │  └─ Log conversation (CloudWatch + S3)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 6. MONITORING & IMPROVEMENT                              │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  ├─ Metrics: Success rate, latency, cost per query      │  │
│  │  ├─ Feedback: User thumbs up/down                        │  │
│  │  ├─ Evaluation: Ground truth comparison (weekly)        │  │
│  │  └─ Prompt optimization: A/B testing                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Components

**LangChain Framework:**
- Agent orchestration with ReAct pattern
- Memory management (conversation history)
- Tool/function calling abstraction
- Prompt template management
- Streaming response support
- Callback handlers for logging

- Callback handlers for logging

**AWS Bedrock:**
- Claude 3.5 Sonnet (primary LLM)
  - 200K context window
  - Tool use native support
  - Multilingual capabilities
  - Token-based pricing
- Titan Embeddings (for RAG)
  - 1,536 dimensions
  - Optimized for vector search
- Bedrock Guardrails
  - Content filtering (hate, violence, etc.)
  - PII redaction
  - Topic blocking

**Amazon OpenSearch Serverless:**
- Vector database for RAG
- Semantic search over knowledge base
- Auto-scaling, no cluster management
- OCU-based pricing

**Deployment:**
- Lambda functions (API endpoints)
- ECS Fargate (long-running agents)
- API Gateway (rate limiting, auth)
- CloudFront (caching, global distribution)

#### Example Architecture

**LangChain Agent with Claude 3.5 on Bedrock:**
```

**Components:**
1. **LLM:** Claude 3.5 Sonnet via AWS Bedrock
   - max_tokens: 2048, temperature: 0.7
   - Streaming responses for better UX

2. **Tools:**
   - GetVehicleAvailability: Calls Fleet Service API
   - CalculateRoute: Calls Mapbox API for routing
   - SearchKnowledge: RAG over OpenSearch knowledge base

3. **Memory:** DynamoDB stores conversation history per session

4. **Prompt Pattern:** ReAct (Reasoning + Acting)
   - Question → Thought → Action → Observation → Final Answer
   - Agent iterates until it has sufficient information

5. **Agent Executor:**
   - Orchestrates tool calls and reasoning
   - Handles errors and retries
   - Manages token limits and costs
    max_iterations=5,
    handle_parsing_errors=True
)

# Run Agent
response = agent_executor.invoke({
    "input": "I need to get to Frankfurt Airport by 6 AM tomorrow",
    "user_id": user_id,
    "current_location": "50.1109,8.6821"  # Frankfurt Hauptbahnhof
})

print(response["output"])
```

#### Strengths
✅ **Mature Framework:** LangChain is industry standard for agents
✅ **AWS Native:** Bedrock fully managed, no infrastructure
✅ **Claude 3.5:** State-of-the-art LLM with great tool use
✅ **Cost-Effective:** Pay-per-token, no minimum fees
✅ **Multi-LLM:** Can switch between Claude, Llama, Mistral on Bedrock
✅ **Guardrails:** Built-in content filtering and PII redaction
✅ **Scalability:** Serverless, auto-scales to millions
✅ **EU Compliance:** Bedrock available in Frankfurt region

#### Weaknesses
❌ **LangChain Complexity:** Steep learning curve
❌ **Bedrock Latency:** Slightly slower than OpenAI (due to AWS overhead)
❌ **Limited Claude Models:** Only Claude 3.5, not GPT-4
❌ **Vendor Lock-in:** Bedrock-specific features

#### Cost Considerations

**Infrastructure Components:**
- LLM Inference (Claude 3.5 Sonnet): Token-based pricing for input/output
- Embeddings (Titan): Vector generation for knowledge base and queries  
- OpenSearch Serverless: Knowledge base vector storage
- Lambda/API Gateway: Request handling and orchestration

**Cost Optimization Strategies:**
- Prompt engineering to minimize token usage
- Caching frequently asked questions
- Knowledge base pre-filtering to reduce LLM calls
- Async processing for non-urgent queries

**Target:** Cost-effective per-conversation economics at scale

#### Scoring (out of 10)
- Ease of Use: 7/10
- Feature Completeness: 10/10
- Cost: 6/10 ⚠️
- Performance: 8/10
- **Overall: 8.0/10**

---

### Alternative 2: OpenAI GPT-4 Turbo with LangChain

#### Strengths
✅ **Best LLM:** GPT-4 Turbo is most capable
✅ **Lower Latency:** Faster than Bedrock
✅ **Better Function Calling:** More reliable tool use
✅ **Strong Ecosystem:** Tons of examples and resources

#### Weaknesses
❌ **Data Privacy:** Conversations sent to OpenAI servers
❌ **EU Compliance Risk:** Data leaves AWS, GDPR concerns
❌ **Cost:** More expensive per token
❌ **Rate Limits:** Can hit limits at scale
❌ **No Guardrails:** Must build own content filtering

#### Cost (Monthly, Production)
```
Higher token costs compared to AWS Bedrock
```

#### Scoring
- Ease of Use: Excellent
- Feature Completeness: Excellent
- Cost: 4/10 ⚠️
- Performance: 10/10
- EU Compliance: 5/10 ⚠️
- **Overall: 7.6/10**

---

### Alternative 3: Custom Fine-Tuned Llama 3.1 on SageMaker

#### Strengths
✅ **Open Source:** No vendor lock-in
✅ **Full Control:** Complete customization
✅ **Data Privacy:** Everything stays in AWS
✅ **Lower Variable Cost:** No per-token fees after training

#### Weaknesses
❌ **High Fixed Cost:** GPU instances ($5K-10K/month)
❌ **Operational Overhead:** Must manage infrastructure
❌ **Inferior Quality:** Llama quality lower than Claude
❌ **Engineering Time:** Significant time to production

#### Cost (Monthly)
```
SageMaker Endpoint:
  - GPU instances for high availability
  
Fine-Tuning (one-time):
  - Training infrastructure costs
  
Total: Higher ongoing costs and one-time setup
```

#### Scoring
- Ease of Use: Poor
- Feature Completeness: Good
- Cost: Moderate
- Performance: Good
- **Overall: Viable for specific needs**

---

## Decision Matrix

| Criteria | LangChain+Bedrock | OpenAI+LangChain | Custom Llama |
|----------|-------------------|------------------|--------------|
| **LLM Quality** | Excellent | Excellent | Good |
| **Cost** | Good | Higher | Moderate |
| **EU Compliance** | Excellent | Lower | Excellent |
| **Time to Market** | Fast | Fast | Slow |
| **Scalability** | Excellent | Good | Moderate |
| **Operational Overhead** | Low | Low | High |
| **Overall** | **Best fit** | **Compliance risk** | **Complex** |

## Decision

**We choose LangChain on AWS Bedrock (Claude 3.5 Sonnet) as our agentic AI framework.**

### Primary Rationale

#### 1. EU Compliance (Critical)
- **All data stays in Frankfurt region** (eu-central-1)
- GDPR compliant with AWS DPA
- No data sent to third-party (OpenAI) servers
- Bedrock Guardrails for PII redaction

#### 2. Balanced Cost
- **More expensive than target** ($0.13 vs $0.02) but acceptable
- Much cheaper than OpenAI GPT-4 ($13K vs $30K)
- No fixed infrastructure costs (vs Llama)
- Cost optimization via caching and prompt engineering

#### 3. Production-Ready
- Fully managed, no infrastructure
- Auto-scaling to millions of requests
- Built-in monitoring and logging
- 99.9% SLA from AWS

#### 4. Framework Flexibility
- LangChain abstracts LLM provider
- Can switch from Claude → Llama → GPT-4 if needed
- Portable to other clouds (GCP Vertex, Azure OpenAI)

### Cost Optimization Strategies

To reduce from $0.13 to closer to $0.02 target:

1. **Caching:** Cache common queries (weather, availability) - 40% reduction
2. **Shorter Prompts:** Optimize system prompts - 20% reduction
3. **Fewer Tool Calls:** Batch API calls - 15% reduction
4. **Streaming:** Stop generation early if sufficient - 10% reduction
5. **Model Switching:** Use Claude Haiku for simple queries - 30% reduction

**New cost:** $13K → $4K/month = $0.04 per conversation ✅ Close to target

## Implementation

### Phase 1: MVP (4 weeks)
- Basic user assistant with 3 tools
- RAG over FAQ knowledge base
- Text-only (no voice)
- English only

### Phase 2: Enhanced (4 weeks)
- Add 5 more tools
- Voice input/output
- Multi-language support
- Staff copilot

### Phase 3: Optimization (4 weeks)
- Implement all cost optimizations
- A/B test prompts
- Fine-tune on conversation data
- Advanced monitoring

## Consequences

### Positive
✅ **EU Compliant:** All data in Frankfurt
✅ **Fast to Market:** 4 weeks to MVP
✅ **Scalable:** Serverless, auto-scales
✅ **Flexible:** Can switch LLMs easily
✅ **State-of-the-Art:** Claude 3.5 is excellent

### Negative
❌ **Cost:** Above initial target (optimizable)
❌ **Learning Curve:** LangChain complexity
❌ **Latency:** Slightly slower than OpenAI

## Related Decisions
- **ADR-15:** AWS chosen - enables Bedrock
- **ADR-16:** MLOps - Can fine-tune models if needed
- **ADR-17:** Data Lakehouse - Conversation logs for training

## References
- [LangChain Documentation](https://python.langchain.com/)
- [AWS Bedrock](https://aws.amazon.com/bedrock/)
- [Claude 3.5 Announcement](https://www.anthropic.com/claude)

## Revision History
- **2025-11-07:** Initial decision - LangChain + Bedrock
- **Next Review:** 2026-02-07 - After Phase 3 optimization
