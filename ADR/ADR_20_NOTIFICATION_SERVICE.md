# ADR-20: Intelligent Notification Service

## Status
Accepted

## Context

MobilityCorp faces low customer engagement (Problem 3) with most users treating the service as ad-hoc rather than habitual. Additionally, operational inefficiency (Problem 4) means staff lack real-time intelligence for prioritizing maintenance and rebalancing tasks.

Key requirements:
- **Customer engagement:** Proactive, personalized notifications to drive habitual usage and relocation participation
- **Operational efficiency:** Real-time alerts to staff for maintenance, battery swaps, and incidents
- **Safety responsiveness:** Immediate incident notifications with user check-ins
- **Privacy compliance:** GDPR-compliant notification preferences, opt-outs, and data retention
- **Scalability:** Handle 10,000+ vehicles per city with real-time event processing

Current limitations:
- Generic, reactive notifications (e.g., booking confirmations only)
- No AI-driven personalization or commute pattern detection
- No relocation incentive notifications to balance fleet
- Staff receive delayed or manual alerts for critical issues

## Decision

Implement an **AI-powered, event-driven Notification Service** that delivers personalized customer notifications and prioritized staff alerts through multiple channels.

### Architecture Components

**1. Event-Driven Input Sources**
- **Kafka consumers:** Listen to events (`bookings.created`, `vehicles.battery_low`, `predictions.maintenance_alert`, `incentives.relocation_opportunity`, `collision.detected`)
- **Scheduled triggers:** Cron jobs for commute pattern analysis and usage milestones
- **API endpoints:** Direct triggers for feedback follow-ups and nearest station queries

**2. Notification Orchestrator**
- **Event router:** AWS Lambda triggered by Kafka (MSK) events, maps to notification types with priority (Critical/High/Medium/Low)
  - Lambda: Node.js 20.x, 512 MB RAM, 5s timeout (avg 300ms execution)
  - DLQ (Dead Letter Queue) for failed routing to SQS
- **User preferences:** DynamoDB table `user_notification_preferences` (on-demand, single-digit ms latency)
  - Attributes: `user_id`, `quiet_hours`, `channel_preferences`, `opt_outs`, `notification_limits`
  - GSI on `notification_type` for bulk operations
- **Rate limiter:** ElastiCache Redis with sliding window algorithm
  - Prevents notification fatigue (max 10/hour, 50/day per user, critical notifications bypass)
  - TTL-based key expiry: `user:{user_id}:limit:{window}`
- **Deduplication:** ElastiCache Redis with 15-min TTL for event hash
  - Prevents redundant notifications for same event (e.g., multiple `battery_low` events)

**3. AI-Driven Personalization (ADR-05, ADR-13)**

a) **Proactive Commute Recommendations**
- Pattern detection analyzes telemetry to identify regular trips (e.g., A→B at 9 AM daily)
- GenAI generates human-friendly suggestions with real-time traffic, weather, availability, and pricing
- Example: *"Heading to King's Cross at 9 AM? The e-scooter near you is fastest today (traffic on Euston Road). Or, grab the eBike for £2 less. Tap to book."*
- Trigger: 15-30 mins before user's typical commute time

b) **Relocation Incentive Offers (ADR-02)**
- Listens to `incentives.relocation_opportunity` events
- AI identifies beneficial relocations based on supply/demand and user's next likely trip
- Example: *"Park near Zone C instead of Zone B and get 20% off your next ride! Help us balance the fleet."*
- Gamification: Track relocations, monthly badges, credits

c) **Smart Alternative Mode Suggestions**
- Triggered when preferred vehicle unavailable or weather conditions poor
- Example: *"Scooters are limited near you, and heavy rain is expected. Consider a car from the nearby station? Or wait 15 mins for more scooters. [Tap for options]"*

d) **Usage Milestones & Gamification**
- Example: *"Congrats! You've completed 10 rides this month and helped rebalance 3 vehicles! Here's £5 credit for your next trip. 🎉"*
- Goal: Increase customer lifetime value and habitual usage

e) **Feedback Follow-Up**
- GenAI sentiment analysis parses negative feedback
- Example: *"Thanks for letting us know about the scooter's brakes. We've flagged it for maintenance. Here's a discount on your next ride."*
- Critical issues escalated to human support

**4. Staff-Facing Notifications**
- **Predictive maintenance:** *"Vehicle #1234 flagged: Likely battery failure in 5-7 days. Schedule maintenance."*
- **Collision alerts:** *"COLLISION DETECTED: Vehicle #5678 at [Location]. User check-in pending. Dispatch support."*
- **Battery cluster alerts:** *"Zone: Shoreditch - 8 vehicles < 20% battery. Priority area for swaps."*
- **Task assignments:** Optimized route updates pushed to staff dashboard

**5. Delivery Channels**
- **Push notifications:** Amazon SNS mobile push (iOS/Android) with FCM/APNs endpoints
  - SNS topic per notification type: `sns:mobility:notification:critical`, `sns:mobility:notification:promo`
  - Lambda fanout to 50K+ devices/sec
- **SMS:** Amazon SNS SMS (critical alerts) via AWS SMS routes (0.0047 USD/message in EU)
  - Delivery receipts via CloudWatch Events
  - Fallback to Twilio for non-AWS-covered regions
- **Email:** Amazon SES (Simple Email Service) for transactional emails
  - Sender reputation > 95% with DKIM/SPF/DMARC
  - Bounce and complaint handling via SNS topics
  - Cost: $0.10 per 1,000 emails
- **In-app messages:** Real-time via WebSocket (API Gateway + Lambda + ElastiCache Redis pub/sub)
  - WebSocket connections persist for instant delivery
  - Fallback to polling for low-battery devices
- **Voice notifications (future):** Amazon Polly text-to-speech for accessibility

**6. Analytics & Feedback Loop**
- Track delivery metrics (sent, opened, clicked, dismissed)
- Engagement metrics feed back to ML models (demand forecasting, personalization)
- A/B testing framework for message variants
- Publishes `notifications.engagement_event` to Kafka

### Integration Points
- **AI/ML Services:** Multi-Provider AI Orchestrator (ADR-05), Conversational AI (ADR-13), Demand Forecasting, Pricing Engine
- **Core Services:** Booking, Telemetry, Fleet Operations, Vehicle Management
- **External APIs:** Weather, traffic, events (ADR-04)
- **Infrastructure:** Kafka event bus (ADR-06), OpenTelemetry tracing (ADR-07)

### Privacy & Compliance (ADR-14)
- User consent management for personalized notifications
- Opt-out mechanisms for all notification types
- 90-day notification history retention with automated deletion
- Regional quiet hours and language preferences
- GDPR-compliant audit trails

## Consequences

### ✅ Positive

**Customer Engagement**
- AI-powered personalization makes platform habitual vs. ad-hoc
- Relocation incentive uptake >25% reduces manual rebalancing costs (£10-15/vehicle)
- Usage milestones and gamification increase customer lifetime value
- Smart alternatives retain users when preferred vehicle unavailable

**Operational Efficiency**
- Predictive maintenance alerts reduce unplanned downtime by 30-40%
- Battery cluster alerts prioritize staff routes, reducing wasted trips
- Collision alerts enable immediate safety response (<1 min)
- Task assignments optimize field operations

**Revenue & Cost Optimization**
- Proactive recommendations increase booking frequency by 20%+
- Relocation incentives reduce manual fleet redistribution costs by 30-50%
- Improved availability from balanced fleet increases revenue by 10-15%

**Scalability**
- Horizontally scalable with Kafka consumers
- Multi-region deployment ready (ADR-09)
- 10,000+ notifications/sec throughput during peak

### ❌ Negative

**Complexity**
- Event-driven architecture adds debugging complexity
- Multi-channel delivery requires vendor management (FCM, Twilio, SendGrid)
- GenAI integration adds latency (<2s for message generation)

**Privacy Risks**
- Personalization requires behavioral data collection (requires explicit consent)
- Pattern detection may be perceived as invasive surveillance
- Must balance personalization with privacy (GDPR compliance)

**User Experience Risks**
- Notification fatigue if rate limiting fails
- False positive recommendations damage trust
- Poor message quality from GenAI undermines brand voice

**Operational Costs**
- Multi-provider AI costs for message generation
- SMS costs for critical alerts at scale
- Storage costs for 90-day notification history

### Mitigation Strategies

**Privacy Controls**
- Transparent opt-in/opt-out for personalization
- Anonymous pattern detection where possible
- Quarterly privacy impact assessments
- Automated deletion workflows (90-day TTL)

**Quality Assurance**
- A/B testing for message variants before full rollout
- Human review for GenAI-generated messages initially
- Confidence thresholds for AI recommendations
- Fallback to template-based messages if GenAI fails

**Notification Fatigue Prevention**
- Strict rate limiting (max N per hour/day)
- User-controlled quiet hours
- Engagement metrics monitoring (dismiss rate >30% = reduce frequency)
- Priority-based routing (critical notifications always delivered)

## Alternatives Considered

**1. Generic Notification Service (No AI)**
- Pros: Simpler, lower cost, no GenAI dependency
- Cons: Doesn't solve engagement problem, no personalization, reactive only
- Rejected: Insufficient to address low customer engagement (Problem 3)

**2. Third-Party Notification Platform (OneSignal, Braze)**
- Pros: Managed service, lower ops overhead, built-in analytics
- Cons: Vendor lock-in, limited AI customization, higher per-notification costs
- Rejected: Cannot integrate deeply with custom AI/ML models (ADR-05)

**3. Staff-Only Notifications (No Customer Notifications)**
- Pros: Lower complexity, fewer privacy concerns
- Cons: Doesn't address customer engagement problem
- Rejected: Misses opportunity to drive habitual usage and relocation participation