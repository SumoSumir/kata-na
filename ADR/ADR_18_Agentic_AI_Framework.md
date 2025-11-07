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
- **Latency:** P95 response time < 2 seconds
- **Accuracy:** > 90% intent classification accuracy
- **Cost:** < $0.05 per conversation (target: $0.02)
- **Scalability:** Handle 10K concurrent conversations
- **Reliability:** 99.9% uptime for AI services

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
  2. Get demand forecast for next 4 hours (Redshift)
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

**AWS Bedrock:**
- Claude 3.5 Sonnet (primary LLM)
  - 200K context window
  - Tool use native support
  - Multilingual
  - Cost: $3/$15 per 1M tokens (input/output)
- Titan Embeddings (for RAG)
  - 1,536 dimensions
  - Cost: $0.10 per 1M tokens
- Bedrock Guardrails
  - Content filtering (hate, violence, etc.)
  - PII redaction
  - Topic blocking

**Amazon OpenSearch Serverless:**
- Vector database for RAG
- Semantic search over knowledge base
- Auto-scaling, no cluster management
- Cost: $0.24/OCU-hour

**Deployment:**
- Lambda functions (API endpoints)
- ECS Fargate (long-running agents)
- API Gateway (rate limiting, auth)
- CloudFront (caching, global distribution)

#### Example Code

```python
# LangChain Agent with Claude 3.5 on Bedrock

from langchain.agents import AgentExecutor, create_react_agent
from langchain_aws import ChatBedrock
from langchain.tools import Tool
from langchain.memory import DynamoDBChatMessageHistory
from langchain.prompts import PromptTemplate
import boto3

# Initialize Bedrock client
bedrock = boto3.client('bedrock-runtime', region_name='eu-central-1')

# LLM
llm = ChatBedrock(
    model_id="anthropic.claude-3-5-sonnet-20241022-v2:0",
    model_kwargs={
        "max_tokens": 2048,
        "temperature": 0.7,
        "top_p": 0.9
    },
    streaming=True,
    client=bedrock
)

# Define Tools
def get_vehicle_availability(zone_id: str, vehicle_type: str) -> dict:
    """Get available vehicles in a zone"""
    # Call Fleet Service API
    response = requests.get(f"{FLEET_API}/vehicles/available",
        params={"zone_id": zone_id, "vehicle_type": vehicle_type})
    return response.json()

def calculate_route(origin: str, destination: str, mode: str) -> dict:
    """Calculate optimal route"""
    # Call Mapbox API
    response = requests.get(f"{MAPBOX_API}/directions/v5/mapbox/{mode}/{origin};{destination}")
    return response.json()

def search_knowledge_base(query: str) -> str:
    """Search FAQs and documentation"""
    # RAG over OpenSearch
    embedding = get_embedding(query)
    results = opensearch.search(
        index="knowledge-base",
        body={
            "query": {
                "knn": {
                    "embedding": {
                        "vector": embedding,
                        "k": 3
                    }
                }
            }
        }
    )
    return "\n".join([hit["_source"]["text"] for hit in results["hits"]["hits"]])

# Create LangChain Tools
tools = [
    Tool(
        name="GetVehicleAvailability",
        func=get_vehicle_availability,
        description="Get list of available vehicles in a specific zone. Input: zone_id and vehicle_type (scooter/ebike/car/van)"
    ),
    Tool(
        name="CalculateRoute",
        func=calculate_route,
        description="Calculate route between two locations. Input: origin (lat,lon), destination (lat,lon), mode (driving/cycling/walking)"
    ),
    Tool(
        name="SearchKnowledge",
        func=search_knowledge_base,
        description="Search FAQs and documentation for answers to user questions"
    )
]

# Prompt Template (ReAct pattern)
prompt = PromptTemplate.from_template("""
You are a helpful mobility assistant for MobilityCorp. Help users find and book vehicles for their trips.

You have access to these tools:
{tools}

Use the following format:
Question: the input question
Thought: think about what to do
Action: the tool to use
Action Input: the input to the tool
Observation: the result of the action
... (repeat Thought/Action/Observation as needed)
Thought: I now know the final answer
Final Answer: the final response to the user

Question: {input}
{agent_scratchpad}
""")

# Memory (conversation history)
message_history = DynamoDBChatMessageHistory(
    table_name="conversational-ai-history",
    session_id=user_session_id
)

# Create Agent
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=message_history,
    verbose=True,
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

#### Cost (Monthly, 100K users)
```
LLM Inference (Claude 3.5 Sonnet):
  - Avg conversation: 1,500 input tokens, 500 output tokens
  - 100K users × 10 conversations/month = 1M conversations
  - Input: 1.5B tokens × $3/1M = $4,500
  - Output: 500M tokens × $15/1M = $7,500
  - Total: $12,000

Embeddings (Titan):
  - Knowledge base refresh: 100K docs × 500 tokens = 50M tokens
  - Queries: 1M × 100 tokens = 100M tokens
  - Total: 150M tokens × $0.10/1M = $15

OpenSearch Serverless:
  - 2 OCUs × 24 hours × 30 days × $0.24 = $345

Lambda/API Gateway:
  - 1M requests × $0.20/1M = $200

Total: $13,060/month ($0.13 per conversation) ⚠️ Above target
```

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
❌ **Cost:** More expensive ($10/$30 per 1M tokens)
❌ **Rate Limits:** Can hit limits at scale
❌ **No Guardrails:** Must build own content filtering

#### Cost (Monthly, 100K users)
```
GPT-4 Turbo:
  - Input: 1.5B × $10/1M = $15,000
  - Output: 500M × $30/1M = $15,000
  - Total: $30,000 ⚠️ 2.3x more expensive
```

#### Scoring (out of 10)
- Ease of Use: 9/10
- Feature Completeness: 10/10
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
❌ **Inferior Quality:** Llama 3.1 70B < Claude 3.5
❌ **Engineering Time:** 3-6 months to production

#### Cost (Monthly)
```
SageMaker Endpoint:
  - ml.g5.12xlarge (4 GPUs): $8/hour × 24 × 30 = $5,760
  - Multi-instance (HA): $5,760 × 3 = $17,280

Fine-Tuning (one-time):
  - Training: ml.p4d.24xlarge × 48 hours = $10,000
  
Total: $17,280/month (ongoing) + $10K (one-time)
```

#### Scoring (out of 10)
- Ease of Use: 4/10
- Feature Completeness: 7/10
- Cost: 5/10
- Performance: 7/10
- **Overall: 5.8/10**

---

## Decision Matrix

| Criteria | Weight | LangChain+Bedrock | OpenAI+LangChain | Custom Llama |
|----------|--------|-------------------|------------------|--------------|
| **LLM Quality** | 25% | 9 | 10 | 7 |
| **Cost** | 20% | 7 | 4 | 5 |
| **EU Compliance** | 20% | 10 | 5 | 10 |
| **Time to Market** | 15% | 9 | 10 | 3 |
| **Scalability** | 10% | 10 | 8 | 6 |
| **Operational Overhead** | 10% | 10 | 9 | 4 |
| **Weighted Score** | | **8.9** | **7.2** | **6.1** |

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
