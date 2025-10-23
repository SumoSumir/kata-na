# ADR-02: AI-Driven Dynamic Pricing and Relocation Incentives

## Status
Accepted  

## Context
MobilityCorp operates multi-modal vehicle fleets (bikes, scoopers, cars, vans) across multiple EU cities. A critical operational challenge is **vehicle imbalance**: popular pickup locations become depleted while low-demand areas accumulate excess vehicles, leading to:

- **Lost revenue:** Customers unable to find vehicles in high-demand areas
- **Increased costs:** Manual relocation by operations staff is expensive and slow
- **Poor customer experience:** Long wait times or unavailability drives users to competitors
- **Inefficient fleet utilization:** Vehicles sit idle in low-demand zones while high-demand areas are underserved

Traditional approaches include:
- **Fixed pricing:** Same rates regardless of supply/demand dynamics
- **Manual relocation:** Staff drive/transport vehicles to balanced locations (costly, slow)
- **Static incentives:** Fixed discounts for specific routes (not adaptive to real-time conditions)

### Requirements
- Dynamically adjust pricing and incentives based on real-time supply/demand predictions
- Encourage customers to self-relocate vehicles from low to high-demand areas
- Minimize manual relocation costs while maintaining fleet availability
- Provide transparent, fair pricing that customers understand and trust
- Support multi-modal fleet with different vehicle economics (bikes vs. cars)

### Alternatives Considered

**1. Manual Relocation Only**  
- Operations staff physically relocate vehicles based on daily patterns  
- ✅ Pros: Simple, predictable, no customer friction  
- ❌ Cons: Expensive (~€10-15 per relocation), slow response to demand spikes, doesn't scale  
- **Rejected:** Cannot keep pace with rapid demand changes or fleet growth  

**2. Fixed-Price Relocation Incentives**  
- Offer standard discounts (e.g., 20% off) for predefined "rebalancing routes"  
- ✅ Pros: Easy to implement, clear customer communication  
- ❌ Cons: Not responsive to real-time conditions, suboptimal incentive levels (too low = ignored, too high = lost revenue)  
- **Rejected:** Insufficient responsiveness and cost optimization  

**3. Surge Pricing Only (No Relocation Incentives)**  
- Increase prices in high-demand areas like ride-sharing surge pricing  
- ✅ Pros: Maximizes revenue, reduces demand in constrained areas  
- ❌ Cons: Poor customer perception, doesn't solve supply imbalance, no self-relocation mechanism  
- **Rejected:** Does not address root cause of vehicle distribution problem  

## Decision
Implement an **AI-driven dynamic pricing and relocation incentive system** that combines demand forecasting, real-time fleet monitoring, and personalized incentives to optimize vehicle distribution and revenue.

### Architecture Components

**1. Demand Forecasting Engine**
- **Real-time prediction:** Forecast demand at 15-minute intervals for each 500m grid cell in each city
- **Input features:**
  - Historical rental patterns (time of day, day of week, seasonality)
  - Weather conditions (temperature, precipitation, wind via Weather & Events API - [ADR-04](ADR_04_EXTERNAL_APIS.md))
  - Events (concerts, sports, festivals via PredictHQ - [ADR-04](ADR_04_EXTERNAL_APIS.md))
  - Public holidays and school schedules (via Calendarific API - [ADR-04](ADR_04_EXTERNAL_APIS.md))
  - Transit disruptions (metro closures, traffic accidents, grid lock via Map API - [ADR-04](ADR_04_EXTERNAL_APIS.md))
- **Model:** Gradient boosting (XGBoost) or deep learning (LSTM) trained on historical data
- **Output:** Predicted demand probability distribution per location and time window

**2. Supply Tracking and Availability Prediction**
- **Real-time inventory:** Current vehicle locations from GPS telemetry (ADR-03, ADR-06)
- **Availability forecasting:** Predict when rented vehicles will return based on:
  - Historical trip duration distributions
  - Current trip progress (distance traveled, time elapsed)
  - Destination prediction models
- **Supply score:** Calculate available capacity per location considering in-transit returns

**3. Dynamic Pricing Engine**
- **Pricing zones:** Divide cities into 500m grid cells with zone-specific pricing
- **Pricing algorithm:**
  - **Base price:** Standard rate per vehicle type and city
  - **Demand multiplier:** Increase price in high-demand, low-supply zones (up to 2.5x base)
  - **Supply multiplier:** Decrease price in low-demand, high-supply zones (down to 0.5x base)
  - **Elasticity modeling:** Adjust multipliers based on observed price sensitivity per customer segment
- **Price caps:** Maximum surge limits to maintain customer trust and regulatory compliance
- **Transparency:** Display dynamic pricing clearly in app with explanations ("High demand in this area")

**4. Relocation Incentive Engine**
- **Route identification:** Identify beneficial relocation routes (high-supply to high-demand zones)
- **Incentive calculation:**
  - **Distance-based:** Longer relocations receive higher incentives
  - **Urgency-based:** Critical imbalances trigger higher incentives
  - **Vehicle type:** Cars/vans receive higher incentives than bikes/scooters (higher relocation value)
  - **Customer segmentation:** Frequent users may receive personalized incentive offers
- **Incentive types:**
  - Percentage discounts (e.g., "50% off if you drop off in Zone B")
  - Flat-rate bonuses (e.g., "Earn €5 credit for this relocation")
  - Loyalty points multipliers (e.g., "2x points for relocations")
- **Real-time updates:** Incentives adjust every 5-15 minutes based on current conditions

**5. Optimization and Constraints**
- **Revenue optimization:** Balance incentive costs against avoided manual relocation costs and increased availability revenue
- **Customer fairness:** Avoid discriminatory pricing; ensure transparent, explainable pricing logic
- **Operational constraints:** Respect vehicle maintenance schedules, parking regulations, and city-specific rules
- **A/B testing framework:** Continuous experimentation to optimize pricing and incentive strategies

### Integration with Existing Systems
- **Event-Driven Architecture (ADR-06):** Pricing updates published to Kafka topics (`pricing.updated`, `incentives.offered`)
- **Booking Service:** Consumes pricing events to display current rates and incentives
- **Vehicle Telemetry Service (ADR-03):** Provides real-time vehicle locations and availability
- **Weather & Events API ([ADR-04](ADR_04_EXTERNAL_APIS.md)):** Supplies demand forecasting features
- **Multi-Provider AI (ADR-12):** Uses ML models for demand forecasting and optimization

### Operational Practices
- **Model monitoring:** Track forecast accuracy, pricing elasticity, and incentive acceptance rates
- **Automated retraining:** Weekly model updates with latest demand patterns
- **Manual overrides:** Operations team can adjust pricing/incentives for special events or emergencies
- **Customer communication:** In-app notifications explain dynamic pricing and highlight relocation opportunities

## Consequences

### ✅ Positive Impacts

**Cost Reduction**
- **30-50% reduction in manual relocation costs:** Self-relocations replace expensive staff-driven moves
- **Improved fleet utilization:** Vehicles spend more time rented vs. idle or in transit for rebalancing
- **Reduced idle inventory:** Better distribution reduces need for oversized fleet to compensate for imbalances

**Revenue Optimization**
- **10-20% revenue increase:** Dynamic pricing captures willingness-to-pay during high-demand periods
- **Reduced lost sales:** Improved availability increases successful bookings in high-demand areas
- **Premium pricing justified:** Customers accept higher prices when supply is constrained and explained transparently

**Customer Experience**
- **Higher availability:** Better vehicle distribution reduces "no vehicles available" scenarios
- **Fair pricing:** Transparent dynamic pricing builds trust vs. opaque or unfair surge pricing
- **Gamification:** Relocation incentives engage customers and create positive interactions

**Operational Efficiency**
- **Automated decision-making:** Reduces operational team workload for manual relocation planning
- **Predictive planning:** Demand forecasts enable proactive positioning for events and patterns
- **Data-driven insights:** Continuous learning improves understanding of customer behavior and demand drivers

### ❌ Negative Impacts and Trade-offs

**Customer Perception Risks**
- **Surge pricing backlash:** Dynamic pricing may be perceived as unfair or exploitative (like ride-sharing surge)
- **Complexity:** Customers may be confused by fluctuating prices or relocation incentives
- **Equity concerns:** Lower-income customers may be priced out during high-demand periods

**Technical and Operational Complexity**
- **Model accuracy risk:** Inaccurate demand forecasts lead to suboptimal pricing (too high = lost sales, too low = lost revenue)
- **System failures:** Pricing engine downtime forces fallback to static pricing, losing optimization benefits
- **Data dependencies:** Requires high-quality real-time data from telemetry, weather APIs, and booking systems

**Revenue Cannibalization**
- **Over-discounting:** Excessive relocation incentives may reduce revenue more than saved relocation costs
- **Incentive gaming:** Customers may exploit incentives (e.g., fake relocations, gaming rules)

**Regulatory and Legal Risks**
- **Price discrimination:** Dynamic pricing must comply with EU consumer protection and non-discrimination laws
- **Transparency requirements:** GDPR and consumer rights laws require clear pricing explanations
- **Caps and restrictions:** Some cities may impose pricing regulations (e.g., maximum surge multipliers)

### Mitigation Strategies

**Customer Trust and Communication**
- **Transparent pricing:** Always display base price vs. dynamic price with clear explanations
- **Price caps:** Limit maximum surge to 2-2.5x base price to maintain affordability
- **Fair warning:** Notify customers of price increases before confirming bookings
- **Loyalty protections:** Offer price guarantees or caps for frequent users

**Model Quality and Monitoring**
- **Continuous monitoring:** Track forecast accuracy, pricing elasticity, and customer satisfaction metrics
- **A/B testing:** Gradually roll out pricing changes and measure impact before full deployment
- **Fallback mechanisms:** Revert to static pricing if models show degraded performance or anomalies
- **Human oversight:** Operations team reviews and approves significant pricing changes or anomalies

**Incentive Controls**
- **Fraud detection:** Monitor for suspicious patterns (repeated short trips, location spoofing)
- **Incentive caps:** Limit maximum discount/bonus per relocation to prevent excessive costs
- **Adaptive incentives:** Adjust incentive levels based on observed acceptance rates and costs

**Regulatory Compliance**
- **Legal review:** Ensure dynamic pricing complies with EU consumer protection and competition laws
- **Data privacy:** Anonymize customer data in pricing models; GDPR-compliant data handling
- **City coordination:** Collaborate with city regulators to ensure pricing aligns with local policies

### Metrics and Acceptance Criteria

**Operational Efficiency**
- **Manual relocation reduction:** ≥40% decrease in staff-driven relocation trips within 6 months
- **Fleet utilization:** ≥15% increase in average vehicle rental hours per day
- **Availability improvement:** ≥20% reduction in "no vehicles available" customer searches

**Financial Performance**
- **Revenue increase:** ≥10% revenue growth from dynamic pricing and improved availability
- **Cost savings:** ≥€50,000 per city per year in reduced relocation costs
- **ROI:** Positive return on investment within 12 months (AI development + infrastructure costs)

**Customer Experience**
- **Satisfaction scores:** Maintain or improve Net Promoter Score (NPS) ≥60
- **Incentive acceptance:** ≥25% of relocation incentive offers accepted by customers
- **Price perception:** ≤10% of customer complaints related to dynamic pricing

**Model Performance**
- **Forecast accuracy:** ≥80% accuracy for demand predictions within ±20% error margin
- **Pricing elasticity:** Observed demand response aligns with predicted elasticity within ±15%

### Open Questions and Next Steps

**Technical**
- Finalize grid cell size (500m vs. 1km) and update frequency (5 min vs. 15 min) based on performance testing
- Prototype demand forecasting model with historical data and validate accuracy
- Define fallback pricing strategy for model failures or data outages

**Business**
- Establish surge pricing caps and discount limits with legal and operations teams
- Define customer communication templates and in-app messaging for dynamic pricing
- Calculate expected ROI per city and prioritize rollout order

**Operational**
- Design operations dashboard for monitoring pricing, incentives, and fleet distribution
- Create runbooks for manual overrides during special events or system issues
- Train customer support team on dynamic pricing explanations and dispute resolution

**Regulatory**
- Conduct legal review of dynamic pricing for GDPR and EU consumer protection compliance
- Engage with city regulators to ensure pricing aligns with local transportation policies
- Prepare transparency reports for pricing and incentive decision-making

---

**Decision Owner:** AI/ML and Product Teams  
**Stakeholders:** Operations, Finance, Legal, Customer Experience  
**Review Cadence:**  
- **Model performance:** Monthly for first 6 months, then quarterly  
- **Business impact:** Quarterly revenue and cost analysis  
- **Customer feedback:** Continuous monitoring with monthly reviews
