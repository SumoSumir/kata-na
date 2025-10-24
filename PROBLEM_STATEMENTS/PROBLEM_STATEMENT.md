# Problem Statement

## Business Context

MobilityCorp operates a multi-modal last-mile transportation platform across multiple cities in the EU providing:

* Electric scooters and eBikes (short-term, flexible rentals)
* Electric cars and vans (advance bookings, fixed durations)

The company faces critical operational and customer experience challenges that threaten growth, profitability, and competitive positioning.

---

## Core Business Problems

### Problem 1: Demand-Supply Mismatch (Vehicle Imbalance)

* **Symptom:** Customers cannot find vehicles in high-demand locations while excess inventory sits idle in low-demand areas.
* **Impact:**
    * Lost revenue from unfulfilled bookings (~15-25% potential bookings)
    * Poor customer experience leading to churn
    * Increased manual relocation costs (€10-15 per move)
* **Root Cause:** Static pricing and reactive (not predictive) fleet positioning.

### Problem 2: Battery Management and Vehicle Availability

* **Symptom:** Vehicles run out of charge mid-operation or sit unused due to dead batteries.
* **Impact:**
    * Reduced fleet utilization (vehicles unavailable 20-30% of time)
    * Emergency manual interventions required
    * Customer frustration with unreliable availability
* **Root Cause:** No predictive prioritization for battery swaps; staff visit locations randomly or by schedule, not by urgency.

### Problem 3: Low Customer Engagement and Retention

* **Symptom:** Most customers use the service ad-hoc; few rely on MobilityCorp for regular commutes.
* **Impact:**
    * Low customer lifetime value (CLV)
    * High customer acquisition costs relative to retention
    * Unpredictable revenue patterns
* **Root Cause:** No personalization, incentives, or proactive engagement to build habitual usage.

### Problem 4: Operational Inefficiency

* **Symptom:** Staff lack real-time intelligence on where to focus efforts (battery swaps, rebalancing, maintenance).
* **Impact:**
    * Wasted staff time visiting low-priority locations
    * Delayed response to critical vehicle issues
    * Higher operational costs per vehicle
* **Root Cause:** No AI-driven prioritization or predictive maintenance.

---

## Technical Challenges

### AI/ML Immaturity

* No demand forecasting or predictive maintenance models in production.
* Manual decision-making for pricing, rebalancing, and maintenance.
* No infrastructure for model training, deployment, or monitoring.

### Observability Gaps

* Limited visibility into system health, AI model performance, or operational metrics.
* No distributed tracing for debugging microservices.
* Reactive incident response instead of proactive monitoring.