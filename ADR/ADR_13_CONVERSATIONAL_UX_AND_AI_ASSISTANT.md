# ADR-005: Conversational UX and AI Assistant Integration with MCP Server

## Status
Proposed

## Context
MobilityCorp aims to offer a seamless conversational experience where users can interact with the system via chat or voice to perform core actions such as booking rides, checking availability, or reporting issues.  

Example interaction:
> User: “Book me a scooter to Powai by 9 AM.”  
> Assistant: “Sure — a scooter is available. ETA: 8:55 AM. Confirm booking?”

The system must:
- Understand natural language intents and entities (e.g., location, time).
- Execute backend actions like ride booking, payments, and user updates.
- Maintain session context and continuity (multi-turn conversation).
- Support multimodal input (text, voice) and output (text, speech, push notifications).
- Work across regions with localized language models and data policies.

MCP (Mobility Control Platform) is the core orchestration server responsible for executing operational workflows (booking, dispatch, telemetry). Integration between the AI Assistant and MCP is essential for real-world actions.

## Decision
Implement a **Conversational AI Layer** backed by a **multi-channel assistant** that interfaces directly with the **MCP Server** for command execution.

Architecture overview:
- **User Interaction Layer:** Chat UI, voice interface (via WebSocket/SDK)
- **NLU Layer:** GenAI model (e.g., GPT, Gemini) extracts intent and entities
- **Orchestration Layer:** Routes actions to MCP via structured commands (e.g., `createBooking`, `getETA`)
- **MCP Server:** Executes business logic, ensures authorization, and returns structured responses
- **Response Formatter:** Converts MCP results into natural language replies
- **Fallback:** Human handoff or static FAQ responses when AI confidence is low

Flow example:
1. User → “Book me a scooter to Powai by 9 AM”  
2. AI Assistant → Extracts `intent=book_ride`, `destination=Powai`, `time=9:00 AM`  
3. Assistant calls MCP endpoint `/bookings/create` with parameters  
4. MCP returns booking details (ETA, cost, confirmation)  
5. Assistant generates human-readable response → “Scooter booked! ETA 8:55 AM.”

## Consequences

✅ **Positive**
- Intuitive user experience via chat or voice  
- Natural integration of AI-driven conversations with real backend actions  
- Reduced app navigation friction — bookings and FAQs handled conversationally  
- Extensible to new use cases (maintenance, feedback, payments)  
- Unified framework for voice and text channels  

❌ **Negative**
- Complex error handling between AI and operational APIs  
- Risk of incorrect actions if intent extraction fails  
- Need for continuous training and prompt updates for accuracy  
- Increased latency due to AI model invocation  

## Alternatives Considered

**Rule-Based Chatbot**
- Pros: Deterministic, easy to test
- Cons: Limited scalability, poor natural interaction
- Rejected: Insufficient flexibility for complex or contextual requests

**Manual App Navigation Only**
- Pros: Simple, no AI dependency
- Cons: Higher friction, slower user flow
- Rejected: Fails to deliver conversational experience goals

## AI-Specific Considerations
- Uses GenAI models for intent detection, FAQ response, and contextual dialogue.
- Employs guardrails and schema validation before executing MCP commands.
- Each region may use localized LLMs for language and compliance.
- Conversational memory limited to session scope for privacy.
- Model responses audited to ensure safe and authorized actions only.
