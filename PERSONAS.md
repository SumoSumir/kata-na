# Personas - MobilityCorp Stakeholders

This document introduces the key people and personas that guide our architectural decisions. Understanding their needs helps us design solutions that deliver real business value.

---

## 🎯 Business Stakeholders

### Sarah Chen - Chief Product Officer (CPO)
**Background:** Former Head of Growth at a leading ride-sharing company. Joined MobilityCorp to scale the platform across Europe.

**Goals:**
- Increase customer retention from 20% to 55% within 18 months
- Reduce vehicle unavailability from 25% to under 5%
- Launch in 5 new cities by end of 2026

**Pain Points:**
- "We're losing €2.3M monthly in missed bookings because vehicles are in the wrong locations"
- "Our ad-hoc usage model isn't building customer loyalty"
- "Manual operations don't scale—we need intelligent automation"

**Success Metrics:**
- Daily Active Users (DAU) growth
- Customer Lifetime Value (CLV)
- Net Promoter Score (NPS)

**Quote:** *"I need data-driven insights to position vehicles where customers actually need them, not where we guess they'll be."*

**How We Solve This:**
- **[ADR-02: AI-Driven Demand Forecasting](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - ML models predict demand 24 hours ahead using 150+ features (weather, events, holidays, transit disruptions)
- **[ADR-16: MLOps Pipeline](./ADR/ADR_16_MLOps_Pipeline.md)** - Automated model training, validation, and deployment with A/B testing
- **Analytics Dashboards** - Real-time KPI tracking showing retention improvements from 20% → 55%
- **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - €30.5M NPV business case with 8.7-month payback

**Real-World Impact:**
> *"In Phase 1, demand forecasting reduced vehicle unavailability from 25% to 8% in Amsterdam pilot, directly increasing Sarah's DAU by 22%."*

---

### Marcus Weber - VP of Fleet Operations
**Background:** 15 years in fleet management, previously optimized delivery operations for a major logistics company. Rose from field coordinator managing 12-person teams to VP level, bringing hands-on operational expertise to strategic leadership.

**Current Role:**
- Age: 47, oversees fleet operations across all EU cities
- Direct reports: 8 regional operations managers, 120+ field staff total
- Manages: 50,000 vehicles across Amsterdam, Barcelona, Madrid, Copenhagen, Munich

**Strategic Goals:**
- Reduce manual battery swap costs from €15/swap to under €6
- Improve fleet utilization from 45% to 75%
- Cut vehicle downtime by 40%
- Scale operations to support 5 new cities by 2026

**Daily Operational Challenges:**
- "Staff spend 60% of their time driving between locations inefficiently"
- "We're swapping batteries on vehicles that won't be used for hours"
- "Reactive maintenance is 3x more expensive than preventive"
- "I waste 2 hours every morning reviewing reports from regional managers"
- "By the time field teams reach vehicles, they're already rented—we need predictive intelligence"

**Success Metrics:**
- Cost per vehicle operation
- Fleet utilization rate
- Mean time between failures (MTBF)
- Field staff productivity (tasks completed per shift)
- Planning time reduction

**Quote:** *"Show me which vehicles need attention RIGHT NOW and optimize my team's routes. Every wasted trip costs us money. I need AI to tell my field managers exactly where to send their teams and why."*

**How We Solve This:**
- **[ADR-02: Dynamic Relocation Incentives](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - AI automatically offers users €3-5 credits to relocate vehicles, reducing manual swap costs from €15 → €6
- **[ADR-02: AI Task Prioritization](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - Prioritizes battery swaps based on predicted demand (not just current battery level)
- **[ADR-16: Predictive Maintenance](./ADR/ADR_16_MLOps_Pipeline.md)** - Machine learning models predict battery failures and maintenance needs
- **[WORKFLOWS/STAFF_WORKFLOWS.md](./WORKFLOWS/STAFF_WORKFLOWS.md)** - AI-powered task prioritization system for field operations
- **[ADR-01: Microservices Architecture](./ADR/ADR_01_microservices_architecture.md)** - Fleet Operations Service scales independently to handle 50K+ vehicles
- **Route Optimization** - Google Maps Directions API calculates most efficient multi-stop routes

**Marcus's AI-Powered Morning Workflow:**
```
6:00 AM: Marcus reviews consolidated dashboard across all cities
  ↓ [AI analyzes 50,000 vehicles across 5 cities]
  ↓ [Predictive model identifies critical tasks per region]

AI Task List for Madrid Region (Prioritized by Urgency × Impact):
┌─────────────────────────────────────────────────────────────┐
│ 1. Priority: CRITICAL - Battery Swap                        │
│    Vehicle: E-SCOOT-1247 (Salamanca district)              │
│    Reason: 15% battery, HIGH demand predicted 8-9 AM        │
│    Impact: €45 lost revenue if unavailable                  │
│    Assigned to: Carlos (2.3 km away, ETA 8 min)            │
├─────────────────────────────────────────────────────────────┤
│ 2. Priority: HIGH - Preventive Maintenance                  │
│    Vehicle: E-BIKE-0893 (Retiro Park)                      │
│    Reason: Brake sensor anomaly detected, 7-day failure     │
│             prediction with 92% confidence                   │
│    Impact: Safety risk + €200 reactive repair cost          │
│    Assigned to: Maria (1.8 km away, ETA 12 min)            │
├─────────────────────────────────────────────────────────────┤
│ 3. Priority: MEDIUM - Relocation                            │
│    Vehicle: E-CAR-0456 (low-demand suburb)                  │
│    Reason: Move to city center, €8 manual cost vs €3 AI     │
│             incentive offered to user (no takers yet)        │
│    Impact: €5 cost savings if AI incentive works            │
│    Status: Wait 2 hours, escalate if no user accepts        │
└─────────────────────────────────────────────────────────────┘

6:15 AM: Marcus approves AI task lists, regional managers notified
9:30 AM: All priority tasks completed, vehicles available for peak demand
```

**Operational & Business Impact:**
- **Planning Time:** 2 hours → 15 minutes (88% reduction)
- **Wasted Trips:** 40% → 8% (vehicles already rented before team arrives)
- **Cost Savings:** €15/swap manual → €6/swap AI-optimized
- **Annual Savings:** €11.176M from relocation automation + predictive maintenance
- **Fleet Utilization:** 45% → 75% (+30 percentage points)

---

### David Park - Chief Technology & Security Officer (CTO/CISO)
**Background:** Ex-AWS solutions architect with 18 years spanning cloud platforms and cybersecurity. Previously CISO at a fintech company before joining MobilityCorp to modernize tech stack and establish security-first architecture.

**Goals:**
- Achieve 99.95% system uptime with zero security incidents
- Scale infrastructure cost-effectively as fleet grows
- Implement AI/ML capabilities without vendor lock-in
- Ensure ISO 27001, SOC 2 Type II, and GDPR/PCI-DSS compliance
- Secure IoT fleet against unauthorized access (50,000+ devices)

**Pain Points:**
- "Our monolithic architecture can't handle peak demand during events"
- "We lack observability—troubleshooting takes hours"
- "50,000 IoT devices are potential attack vectors if not properly secured"
- "One data breach destroys our brand and costs millions in fines"
- "GDPR right-to-erasure requests are manual and error-prone"

**Success Metrics:**
- System availability (99.95% SLA compliance)
- P95 API response time (<200ms)
- Infrastructure cost per transaction
- Zero security incidents or data breaches
- <24 hour mean time to detect (MTTD) threats
- 100% IoT device certificate rotation compliance

**Quote:** *"I need an architecture that scales independently per service, fails gracefully, and embeds security from day one—not as an afterthought. Every API call, every data store, every IoT message needs encryption and authentication."*

**How We Solve This:**

**Infrastructure & Scalability:**
- **[ADR-01: Microservices Architecture](./ADR/ADR_01_microservices_architecture.md)** - 8 independently scalable services on AWS ECS Fargate (Telemetry, Booking, Payment, Fleet Ops, Pricing, Analytics, Notification, User/KYC)
- **[ADR-09: Multi-Region Deployment](./ADR/ADR_09_MULTI_REGION.md)** - Active-passive setup in eu-central-1 (Frankfurt) + eu-west-1 (Ireland) with 99.95% uptime SLA
- **[ADR-10: Observability Stack](./ADR/ADR_10_Monitoring_and_Metrics.md)** - VictoriaMetrics, Grafana, Loki for unified metrics, logs, and traces
- **[ADR-06: Event-Driven Architecture](./ADR/ADR_06_EVENT_DRIVEN_ARCHITECTURE.md)** - Kafka topics isolate failures, prevent cascading outages

**Security & Compliance:**
- **[ADR-12: Security-by-Design Architecture](./ADR/ADR_12_SECURITY_ARCHITECTURE.md)** - Comprehensive zero-trust model with AWS IAM, Secrets Manager, KMS encryption
- **[ADR-14: Data Compliance](./ADR/ADR_14_DATA_COMPLIANT.md)** - GDPR data residency (EU regions only), automated right-to-erasure
- **[ADR-11: IoT Security](./ADR/ADR_11_IoT_Enabled_Vehicles.md)** - X.509 certificates for 50K devices, AWS IoT Core mutual TLS (mTLS)
- **[COST_ANALYSIS.md](./COST_ANALYSIS.md)** - Security ROI: €295K/year positive ROI (breach prevention)

**Architecture Highlights:**
- **Independent Scaling:** Telemetry (10 tasks, 2 vCPU) vs. Booking (20 tasks peak) vs. Analytics (4 vCPU, 8GB RAM)
- **Fault Isolation:** Booking failure doesn't disrupt vehicle tracking
- **Cost Efficiency:** $45K/month infrastructure for 50K vehicles (~$0.90 per vehicle/month)
- **Zero-Trust Security:** Multi-layered defense with network isolation, IAM roles, encryption at rest/in transit
- **Compliance Readiness:** GDPR, PCI-DSS Level 1, ISO 27001, SOC 2 Type II (in progress)

---

## 👥 End Users

### Emma Thompson - Commuter (Regular User)
**Profile:**
- Age: 28, Marketing Manager
- Location: Amsterdam city center
- Usage: Daily 4km commute (home → office)

**Scenario:**
Emma leaves for work at 8:15 AM every weekday. She needs a scooter within 100m of her apartment and wants to pre-book it the night before to guarantee availability.

**Pain Points:**
- "Half the time, there's no scooter available when I need it"
- "I'd use the app daily if I could rely on it, but I keep Uber as backup"
- "The pricing varies wildly—I can't budget my transport costs"

**Needs:**
- Predictable availability at her location
- Transparent, consistent pricing
- Ability to pre-book for recurring trips

**Quote:** *"I want to delete Uber and just use MobilityCorp, but I can't risk being late to work."*

**How We Solve This:**
- **[ADR-02: Predictive Rebalancing](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - ML model predicts Emma's 8:15 AM demand and triggers relocation incentives the night before
- **Pre-Booking Feature** - Emma reserves her scooter at 11:00 PM for next morning (30-minute advance booking)
- **[WORKFLOWS/CUSTOMER_WORKFLOWS.md](WORKFLOWS/CUSTOMER_WORKFLOWS.md)** - Step-by-step booking, unlock, and return process with AI verification
- **Dynamic Pricing Transparency** - Real-time surge/discount indicators in-app

**Emma's Typical Day:**
```
11:00 PM: Emma pre-books scooter for 8:15 AM pickup
  ↓ [Demand Forecasting Service predicts high morning demand]
  ↓ [System offers €3 credit to user near Emma's apartment to relocate scooter]
8:10 AM: Emma opens app, sees reserved scooter 50m away
8:15 AM: NFC unlock, AI takes pickup photo (damage detection)
8:35 AM: Arrival at office, return photo, auto-verification
8:36 AM: Payment processed, €4.20 charged, receipt sent
```

**Business Result:** *Emma's retention increased from occasional user → daily commuter (35% retention improvement target).*

---

### Alex Kumar - Tourist (Ad-hoc User)
**Profile:**
- Age: 34, Tourist from London
- Location: Barcelona for 3 days
- Usage: Spontaneous sightseeing trips

**Scenario:**
Alex wants to explore Barcelona's beaches and Gothic Quarter. He needs on-demand access to vehicles near major attractions, simple payment, and multilingual support.

**Pain Points:**
- "I don't know where vehicles are available"
- "The app should suggest routes and attractions"
- "I'm worried about getting fined if I don't park correctly"

**Needs:**
- Real-time vehicle availability map
- Route recommendations with tourist attractions
- Clear parking instructions in English

**Quote:** *"Make it as easy as Google Maps—I shouldn't need to think about logistics."*

**How We Solve This:**
- **[ADR-13: Conversational AI Assistant](./ADR/ADR_13_CONVERSATIONAL_UX_AND_AI_ASSISTANT.md)** - Claude 3.7 Sonnet LLM with multilingual support (English, Spanish, German, French)
- **[ADR-04: External APIs](./ADR/ADR_04_EXTERNAL_APIS.md)** - Google Maps integration for route planning and parking zone detection
- **Real-Time Telemetry** - Vehicle availability updated every 30 seconds via IoT Core
- **[WORKFLOWS/CUSTOMER_WORKFLOWS.md](WORKFLOWS/CUSTOMER_WORKFLOWS.md)** - Visual parking verification with AI-powered location checks

**Alex's User Journey:**
```
Tourist opens app: "Show me scooters near Sagrada Familia"
  ↓ [Conversational AI understands natural language query]
  ↓ [Maps API locates 8 scooters within 200m]
AI Assistant: "I found 8 scooters nearby. The closest one is €2.50 for 
              30 min. Want me to reserve it?"
Alex: "Yes, and suggest a route to Park Güell"
  ↓ [Maps API generates scenic route with estimated 18 min travel time]
AI Assistant: "Reserved! Here's your route. When you arrive at Park Güell, 
              park in the blue zone marked on the map. I'll guide you."
```

**Business Impact:** *Tourist conversion rate improved 40% with conversational UI (vs. traditional map-only interface).*

---

### Lisa Müller - Family User (Weekend Renter)
**Profile:**
- Age: 41, Teacher with two kids
- Location: Munich suburbs
- Usage: Weekend errands and family outings

**Scenario:**
Lisa needs a van on Saturday mornings for grocery shopping and ferrying kids to activities. She books 2-3 days in advance and needs vehicles that are clean and well-maintained.

**Pain Points:**
- "Last time, the van was dirty with someone's coffee cup inside"
- "I've had a van run out of charge mid-trip—terrifying with kids"
- "Booking is complicated—too many steps"

**Needs:**
- Cleanliness verification with photos
- Reliable battery range predictions
- Simple, family-friendly booking flow

**Quote:** *"I need to trust that the vehicle will be clean, charged, and reliable—my kids depend on it."*

**How We Solve This:**
- **[ADR-18: Computer Vision for Quality](./ADR/ADR_18_Agentic_AI_Framework.md)** - ResNet-50 CNN detects damage, dirt, and trash with 88% accuracy
- **[ADR-07: Battery Health Prediction](./ADR/ADR_16_MLOps_Pipeline.md)** - ML model predicts remaining range with 95% accuracy (±2 km)
- **Pre-Rental Photos** - Lisa sees timestamped photos of van interior/exterior before booking
- **Automated Quality Scoring** - Vehicles below 7/10 cleanliness score automatically flagged for cleaning

**Lisa's Weekend Booking:**
```
Thursday 8:00 PM: Lisa books family van for Saturday 9:00 AM
  ↓ [System shows van's latest inspection photos with 9/10 quality score]
  ↓ [Battery health check: 85% charge = 180 km range predicted]
Lisa reviews: "Van looks clean, battery sufficient for 40 km trip"
Saturday 8:50 AM: Arrives at van, takes pickup photo
  ↓ [AI verifies no new damage, cleanliness matches booking photos]
  ↓ [Kid car seat detected in back row—matches family profile]
Rental starts, family drives to store + activities
Return: AI detects minor coffee stain (not charged, below damage threshold)
```

**Trust Building:** *Lisa's NPS score: 9/10. "Finally, a rental service I can rely on with kids."*

---

## 🔧 Operations Team

### Javier Rodriguez - Field Operations Manager
**Profile:**
- Age: 32, Fleet Coordinator
- Location: Madrid operations center
- Manages: 12-person field team

**Scenario:**
Javier starts each day reviewing battery levels across 2,000 vehicles. He manually assigns tasks to field staff based on experience and intuition about demand patterns.

**Pain Points:**
- "I waste 2 hours every morning planning routes"
- "By the time my team reaches a vehicle, it's already been rented"
- "We don't know which vehicles will actually be needed today"

**Needs:**
- AI-powered task prioritization
- Dynamic route optimization
- Real-time vehicle status updates

**Quote:** *"Give me a smart system that tells my team exactly where to go and why—don't make me guess."*

**How We Solve This:**
- **[ADR-02: AI Task Prioritization](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md)** - Prioritizes battery swaps based on predicted demand (not just current battery level)
- **[ADR-07: Predictive Maintenance Alerts](./ADR/ADR_16_MLOps_Pipeline.md)** - Flags vehicles needing preventive service before they fail
- **[WORKFLOWS/STAFF_WORKFLOWS.md](WORKFLOWS/STAFF_WORKFLOWS.md)** - Detailed workflow for Javier's morning routine with AI-optimized task list
- **Route Optimization** - Google Maps Directions API calculates most efficient multi-stop routes

**Javier's AI-Powered Morning Workflow:**
```
6:00 AM: Javier logs into Field Ops Dashboard
  ↓ [AI analyzes 2,000 vehicles across Madrid]
  ↓ [Predictive model identifies critical tasks]

AI Task List (Prioritized by Urgency × Impact):
┌─────────────────────────────────────────────────────────────┐
│ 1. Priority: CRITICAL - Battery Swap                        │
│    Vehicle: E-SCOOT-1247 (Salamanca district)              │
│    Reason: 15% battery, HIGH demand predicted 8-9 AM        │
│    Impact: €45 lost revenue if unavailable                  │
│    Assigned to: Carlos (2.3 km away, ETA 8 min)            │
├─────────────────────────────────────────────────────────────┤
│ 2. Priority: HIGH - Preventive Maintenance                  │
│    Vehicle: E-BIKE-0893 (Retiro Park)                      │
│    Reason: Brake sensor anomaly detected, 7-day failure     │
│             prediction with 92% confidence                   │
│    Impact: Safety risk + €200 reactive repair cost          │
│    Assigned to: Maria (1.8 km away, ETA 12 min)            │
├─────────────────────────────────────────────────────────────┤
│ 3. Priority: MEDIUM - Relocation                            │
│    Vehicle: E-CAR-0456 (low-demand suburb)                  │
│    Reason: Move to city center, €8 manual cost vs €3 AI     │
│             incentive offered to user (no takers yet)        │
│    Impact: €5 cost savings if AI incentive works            │
│    Status: Wait 2 hours, escalate if no user accepts        │
└─────────────────────────────────────────────────────────────┘

6:15 AM: Javier approves AI task list, team dispatched
9:30 AM: All priority tasks completed, vehicles available for peak demand
```

**Efficiency Gains:** 
- **Planning Time:** 2 hours → 15 minutes (88% reduction)
- **Wasted Trips:** 40% → 8% (vehicles already rented before team arrives)
- **Cost Savings:** €15/swap manual → €6/swap AI-optimized

---

### Nina Petersen - Customer Support Agent
**Profile:**
- Age: 27, Support Team Lead
- Location: Copenhagen support center
- Handles: 200+ tickets/day

**Scenario:**
Nina resolves disputes about fines, parking violations, and damage claims. Most tickets involve "he said, she said" situations without clear evidence.

**Pain Points:**
- "Customers dispute fines because we lack photo proof"
- "I can't tell if damage was pre-existing or caused by the renter"
- "Manual verification takes 10-15 minutes per case"

**Needs:**
- Automated photo verification
- Damage detection with timestamps
- Self-service dispute resolution

**Quote:** *"If the system could automatically verify returns and detect damage, I could focus on real customer issues."*

**How We Solve This:**
- **[ADR-18: Computer Vision Automation](./ADR/ADR_18_Agentic_AI_Framework.md)** - ResNet-50 CNN + AWS Rekognition for damage detection and location verification
- **Automated Dispute Resolution** - 70% of cases auto-resolved with photo evidence + timestamps
- **[WORKFLOWS/CUSTOMER_WORKFLOWS.md](WORKFLOWS/CUSTOMER_WORKFLOWS.md)** - Return verification workflow with AI-powered checks
- **Audit Trail** - Immutable photo history stored in S3 with 90-day retention

**Nina's Automated Support Workflow:**
```
BEFORE (Manual Process):
Customer: "I was charged €50 for damage I didn't cause!"
Nina: [Spends 10 min reviewing booking history, photos]
Nina: [Contacts previous renter, waits 24 hours for response]
Nina: [Manually compares photos, makes judgment call]
Resolution: 15 minutes per case, low customer satisfaction

AFTER (AI-Powered Automation):
Customer: "I was charged €50 for damage I didn't cause!"
  ↓ [System automatically retrieves pickup/return photos]
  ↓ [Computer Vision compares photos, timestamps damage to previous renter]
  ↓ [AI generates evidence report with confidence score]

System Auto-Response:
"Our AI review shows the scratch appeared during the previous 
 rental (12/05 3:42 PM). We've refunded your €50 and updated 
 the correct account. Apologies for the error!"

Nina's dashboard: ✅ Auto-resolved in 2 seconds
```

**Support Efficiency:**
- **Ticket Resolution Time:** 15 min → 2 min (87% reduction)
- **Auto-Resolution Rate:** 0% → 70%
- **Customer Satisfaction:** +25% improvement
- **Nina's Focus:** Now handles complex cases (privacy requests, payment disputes, escalations)

---

## 📊 How Personas Drive Architecture

---

## 📊 How Personas Drive Architecture

| Persona | Key Need | Architectural Solution |
|---------|----------|------------------------|
| **Sarah (CPO)** | Predictive demand insights | ML demand forecasting + analytics dashboards |
| **Marcus (VP Fleet Ops)** | Efficient resource allocation | AI route optimization + battery prioritization |
| **David (CTO/CISO)** | Scalable, resilient, secure infrastructure | Microservices + multi-region + zero-trust security |
| **Emma (Commuter)** | Reliable availability | Predictive rebalancing + pre-booking |
| **Alex (Tourist)** | On-demand discovery | Real-time availability + conversational AI |
| **Lisa (Family User)** | Vehicle quality assurance | Computer vision damage detection |
| **Nina (Support)** | Automated verification | Vision AI + automated dispute resolution |

---

## 🎭 Using Personas in Documentation

Throughout our documentation, we reference these personas to illustrate:
- **Requirements**: "Emma needs reliable availability → Predictive rebalancing"
- **ADRs**: "David requires independent scaling → Microservices architecture"
- **Workflows**: "Javier's morning routine → AI task prioritization flow"
- **Success Metrics**: "Sarah measures retention → Track CLV improvements"

**Example Usage:**
> "When Emma opens the app at 8:00 AM, the **Demand Forecasting Service** has already predicted high demand for her neighborhood and triggered the **Relocation Incentive System** to position scooters nearby (see ADR-02)."

---

## 💡 Design Principles from Personas

1. **Reliability First** (Emma, Lisa) → 99.95% uptime, predictive maintenance
2. **Operational Efficiency** (Marcus) → AI-driven automation, cost reduction
3. **Scalability & Security** (David) → Microservices, multi-region, zero-trust architecture
4. **User Experience** (Alex, Nina) → Conversational AI, automated verification
5. **Data-Driven Decisions** (Sarah, Marcus) → Real-time analytics, ML insights

These personas guide every architectural decision, ensuring we solve real problems for real people.

---

## 🔗 Persona-to-Solution Cross-Reference Map

This matrix shows exactly where each persona's needs are addressed in our architecture:

| Persona | Core Solution | Primary ADR | Workflow | Business Metric |
|---------|---------------|-------------|----------|-----------------|
| **Sarah (CPO)** | ML Demand Forecasting | [ADR-02](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) | [Customer Discovery](WORKFLOWS/CUSTOMER_WORKFLOWS.md) | Retention: 20% → 55% |
| **Marcus (VP Fleet Ops)** | AI Relocation Incentives | [ADR-02](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) | [Staff Workflows](WORKFLOWS/STAFF_WORKFLOWS.md) | Cost: €15 → €6/swap |
| **David (CTO/CISO)** | Microservices + Multi-Region + Zero-Trust | [ADR-01](./ADR/ADR_01_microservices_architecture.md), [ADR-09](./ADR/ADR_09_MULTI_REGION.md), [ADR-12](./ADR/ADR_12_SECURITY_ARCHITECTURE.md) | [Event Storming](EVENT_STORMING.md) | Uptime: 99.95% SLA, Zero breaches |
| **Emma (Commuter)** | Predictive Rebalancing | [ADR-02](./ADR/ADR_02_AI_DRIVEN_RELOCATION_INCENTIVES.md) | [Booking Flow](WORKFLOWS/CUSTOMER_WORKFLOWS.md#workflow-vehicle-discovery-and-booking) | Availability: 75% → 95% |
| **Alex (Tourist)** | Conversational AI | [ADR-13](./ADR/ADR_13_CONVERSATIONAL_UX_AND_AI_ASSISTANT.md) | [Customer Workflows](WORKFLOWS/CUSTOMER_WORKFLOWS.md) | Conversion: +40% |
| **Lisa (Family User)** | Computer Vision Quality | [ADR-18](./ADR/ADR_18_Agentic_AI_Framework.md) | [Return Verification](WORKFLOWS/CUSTOMER_WORKFLOWS.md#5-return--verification) | NPS: +25 points |
| **Nina (Support)** | Automated Dispute AI | [ADR-18](./ADR/ADR_18_Agentic_AI_Framework.md) | [Support Automation](WORKFLOWS/CUSTOMER_WORKFLOWS.md#6-payment--feedback) | Auto-resolve: 70% |

---

## 🎬 Use Cases: Personas in Action

### Use Case 1: Emma's Morning Commute (Predictive Rebalancing)

**Actors:** Emma (Commuter), Sarah (CPO tracking retention), Marcus (VP Fleet Ops executing AI tasks)

**Scenario Timeline:**
```
Day Before (11:00 PM):
• Emma pre-books scooter for 8:15 AM
• Demand Forecasting Service predicts high demand in Emma's neighborhood
• System identifies scooter surplus 2 km away
• AI offers €3 credit to User-XYZ to relocate scooter near Emma's apartment
• User-XYZ accepts, drops scooter 50m from Emma's location

Morning (8:00 AM):
• Emma opens app, sees reserved scooter with 95% battery
• System sends push notification: "Your scooter is ready at corner of Main St"
• Emma walks 50m, NFC unlock, AI takes pickup photo (no damage detected)

Post-Rental:
• Sarah's dashboard: Emma completed 22 consecutive daily commutes this month
• Retention algorithm marks Emma as "High-Value Commuter" (CLV: €180/month)
• Business Impact: Emma deleted Uber app, now 100% MobilityCorp user
```

**Systems Involved:**
- Demand Forecasting Service (ADR-02)
- Relocation Incentive Engine (ADR-02)
- Booking Service (ADR-01)
- Vehicle Telemetry (ADR-03)
- Analytics Dashboard (ADR-17)

**Success Metrics:** Emma's retention → 100% | Sarah's KPI: +22% DAU | Marcus's cost: €3 AI incentive vs €15 manual

---

### Use Case 2: Alex's Tourist Experience (Conversational AI)

**Actors:** Alex (Tourist), David (CTO monitoring system performance)

**Conversation Flow:**
```
Alex: "I'm at Sagrada Familia. Show me scooters nearby."

AI Assistant (Claude 3.7 Sonnet):
"I found 8 scooters within 200m. The closest one is €2.50 for 30 min. 
Would you like me to reserve it?"

Alex: "Yes, and suggest a route to Park Güell with scenic views."

AI Assistant:
"Reserved! Here's a 4.2 km route passing through Gothic Quarter. 
Estimated 18 min. I'll guide you turn-by-turn. 
Note: Park Güell has paid parking in blue zones—I'll show you where."

[Alex completes rental, AI prompts for feedback]

Alex: "Great experience! How do I explore more neighborhoods tomorrow?"

AI Assistant:
"I recommend the Waterfront Loop: 6 km through Barceloneta Beach and 
Port Vell. Pre-book an eBike tonight—better for longer distances."
```

**Systems Involved:**
- Conversational AI Service (ADR-13)
- Google Maps API (ADR-04)
- Vehicle Telemetry (ADR-03)
- Booking Service (ADR-01)

**David's Tech Monitoring:**
- API Response Time: 120ms P95 (target: <200ms) ✅
- LLM Token Cost: $0.08 per conversation (budget: $0.15) ✅
- Uptime: 99.97% during tourist season ✅

**Business Impact:** Tourist conversion +40% | Alex's NPS: 10/10 | LLM cost: $0.08 vs €4.20 booking revenue (1.9% overhead)

---

### Use Case 3: Marcus's AI-Optimized Morning (Operational Efficiency)

**Actors:** Marcus (VP Fleet Operations)

**Before AI (Manual Process):**
```
6:00 AM: Marcus reviews spreadsheet of 2,000 vehicles (Madrid region)
6:30 AM: Manually identifies 50 vehicles needing attention
7:00 AM: Assigns tasks based on staff proximity (rough estimates)
8:00 AM: Team deployed, but 15 vehicles already rented (wasted trips)
10:00 AM: Reactive battery swaps as vehicles die mid-rental

Cost: €15 per manual swap × 50 vehicles = €750/day
Efficiency: 40% wasted trips, 2 hours planning time
```

**After AI (Automated Prioritization):**
```
6:00 AM: Marcus opens AI-powered dashboard
6:05 AM: AI task list ready (prioritized by urgency × impact)

Top Priority Tasks:
1. Battery Swap - E-SCOOT-1247 (Salamanca)
   • Current: 15% battery
   • Predicted Demand: 8-9 AM peak (85% probability)
   • Lost Revenue if Unavailable: €45
   • Action: Assign to Carlos (2.3 km away, ETA 8 min)

2. Preventive Maintenance - E-BIKE-0893 (Retiro Park)
   • Brake sensor anomaly detected
   • 7-day failure prediction (92% confidence)
   • Cost Avoidance: €200 reactive repair → €50 preventive
   • Action: Assign to Maria (1.8 km away, ETA 12 min)

3. AI Relocation - E-CAR-0456 (Low-demand suburb)
   • Relocation Incentive: €3 credit offered to 5 users
   • Status: Wait 2 hours before manual intervention
   • Cost Savings: €5 (€8 manual - €3 AI incentive)

6:15 AM: Marcus approves, team dispatched with optimized routes
9:30 AM: All critical tasks completed, zero wasted trips

Cost: €6 per AI-optimized swap × 35 vehicles = €210/day
Efficiency: 8% wasted trips, 15 min planning time
```

**Marcus's Business Dashboard:**
- Daily Cost Savings: €540/day (€750 - €210)
- Annual Savings: €197K per city (€540 × 365)
- Fleet Utilization: 45% → 75% (+30 percentage points)
- Wasted Trips: 40% → 8% (-32 percentage points)

**Systems Involved:**
- AI Task Prioritization (ADR-02)
- Predictive Maintenance (ADR-07)
- Route Optimization (ADR-04 Maps API)
- Fleet Operations Service (ADR-01)

---

### Use Case 4: Lisa's Family Rental (Quality Assurance)

**Actors:** Lisa (Family User), Nina (Support Agent), David (CTO/CISO auditing data handling)

**Booking Journey:**
```
Thursday 8:00 PM: Lisa books family van for Saturday 9:00 AM
  ↓ [System shows van's latest inspection photos]
  ↓ [Computer Vision Quality Score: 9/10 (clean, no damage)]
  ↓ [Battery Health: 85% charge = 180 km range (±2 km accuracy)]

Lisa reviews photos:
• Exterior: No scratches, clean body
• Interior: Vacuumed, no trash, car seat anchor visible
• Battery: Sufficient for 40 km round trip to store + activities

Lisa confirms booking, pre-authorized payment: €45

Saturday 8:50 AM: Lisa arrives at van, opens app
  ↓ [App prompts: "Take 4 photos - front, rear, interior, dashboard"]
  ↓ [Lisa uploads photos, AI processes in 3 seconds]
  ↓ [Computer Vision detects: No new damage, cleanliness matches]
  ↓ [Family profile detected: Kid car seat in back row ✅]

Rental starts: 9:00 AM - 2:30 PM (5.5 hours)
Return: Lisa parks van, takes return photos
  ↓ [AI detects: Minor coffee stain on floor mat]
  ↓ [Damage Assessment: Below damage threshold (€0 charge)]
  ↓ [Cleanliness Score: 8/10 (acceptable, flagged for cleaning)]

Payment: €42.50 charged (base rate, no damage fees)
Lisa's Receipt: "Thank you! Minor cleaning needed (no charge to you)."
```

**Nina's Support View (No Action Needed):**
```
Dashboard Alert: Van-0456 needs cleaning before next rental
Action: Automated task assigned to cleaning crew
Status: ✅ No customer dispute (AI auto-resolved with photo evidence)
```

**David's Security Audit:**
```
Data Handling Check:
• Photos stored in S3 with AES-256 encryption ✅
• Retention: 90 days, auto-deleted after (GDPR compliant) ✅
• PII redaction: License plates blurred in stored images ✅
• Access logs: Only Lisa, Nina, and authorized staff viewed photos ✅
```

**Business Impact:**
- Lisa's NPS: 9/10 ("Finally, a rental I can trust with kids")
- Nina's workload: 0 tickets from this rental (AI handled everything)
- Damage dispute cost: €0 (auto-resolved with photo evidence)
- David's compliance audit: ✅ Passed GDPR data handling review

**Systems Involved:**
- Computer Vision Service (ADR-18)
- Booking Service (ADR-01)
- Payment Service (ADR-19)
- Data Compliance (ADR-14)
- Support Dashboard (ADR-10 observability)

---

### Use Case 5: David's Security Incident Response (Zero-Trust in Action)

**Actors:** David (CTO/CISO), Operations Team

**Incident Timeline:**
```
Monday 2:14 AM: Suspicious activity detected
  ↓ [CloudWatch Alarm: 5 failed MFA attempts from non-EU IP (185.x.x.x)]
  ↓ [VictoriaMetrics: Login attempts for admin account "fleet-ops-admin"]
  ↓ [Automated Response: Account temporarily locked after 3rd failure]

2:15 AM: David's SOC team receives PagerDuty alert
  ↓ [On-call engineer reviews CloudWatch logs]
  ↓ [Identifies credential stuffing attack (leaked password database)]
  ↓ [Attacker attempted 47 passwords in 2 minutes]

2:20 AM: Immediate Response
  ↓ [WAF rule deployed: Block IP range 185.x.x.0/24]
  ↓ [User account password reset enforced]
  ↓ [Email sent to account owner: "Suspicious login detected, password reset required"]

2:25 AM: Threat Assessment
  ↓ [Zero-trust architecture prevented breach]
  ↓ [No services accessed (MFA blocked attacker at auth layer)]
  ↓ [No data exfiltrated (attacker never reached API layer)]

Tuesday 9:00 AM: Post-Incident Review
  ↓ [David presents incident report to executive team]
  ↓ [MTTD: 1 minute (alarm triggered immediately)]
  ↓ [MTTR: 11 minutes (attacker blocked, user secured)]
  ↓ [Root Cause: Employee password leaked in 3rd-party breach]
  ↓ [Action Plan: Mandatory password rotation + security training]
```

**David's Resilience Dashboard:**
```
Incident Impact Analysis:
• Services Affected: 0 (zero-trust prevented lateral movement)
• Data Breached: 0 bytes
• Downtime: 0 seconds
• Customer Impact: 0 users affected
• SLA Compliance: ✅ 99.97% uptime maintained

Security Posture:
• MFA Enforcement: 100% (blocked attacker)
• WAF Rules: Updated with new threat intelligence
• Incident Response Time: 11 minutes (target: <24 hours)
```

**David's Compliance Report:**
```
ISO 27001 Audit Trail:
✅ Incident detected within SLA (<24 hours MTTD)
✅ Zero-trust prevented unauthorized access
✅ User notified within 6 hours (GDPR requirement)
✅ Threat intelligence shared with AWS Shield
✅ Post-incident training scheduled for affected user

SOC 2 Type II Evidence:
• Access controls prevented breach
• Monitoring systems detected anomaly
• Incident response playbook executed correctly
• No customer data compromised
```

**Business Impact:**
- Financial Loss: €0 (no data breach, no downtime)
- Reputational Risk: Mitigated (incident contained before escalation)
- Compliance Status: ✅ Maintained (GDPR, ISO 27001, SOC 2)
- Customer Trust: Preserved (zero users affected)

**David's Quote:** *"This is why we invested in zero-trust. The attacker never got past the front door."*

**Systems Involved:**
- Zero-Trust Architecture (ADR-12)
- AWS IAM + MFA (ADR-12)
- CloudWatch Logs (ADR-10)
- VictoriaMetrics (ADR-10)
- WAF Rules (ADR-12)
- Incident Response Playbook (ADR-12)

---

## 📚 Further Reading

- **[README.md](README.md)** - Complete architecture overview and business case
- **[PHASED_IMPLEMENTATION.md](PHASED_IMPLEMENTATION.md)** - €30.5M NPV migration plan with persona success stories
- **[EVENT_STORMING.md](EVENT_STORMING.md)** - Domain events and bounded contexts derived from persona workflows
- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - Navigation guide by role (CTO, CFO, CPO, Engineers)
- **[ADR/](./ADR/)** - 19 Architecture Decision Records with persona-driven rationale

---

**Document Version:** 1.2  
**Last Updated:** November 8, 2025  
**Status:** Production-Ready for O'Reilly Architectural Katas Q4 2025
