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
- **Opt-in Service:** Users can enable this while booking. Our preferred destination will be within 2 km of theirs, and they won’t be billed if going slightly past their stop to our target zone. The system checks for existing bookings and nearby riders (within 2 km) finishing trips before scheduling a pickup.
- **Route identification:** Identify beneficial relocation routes from high-supply to high-demand zones.
- **Incentive calculation:**
 - Distance-based: Longer relocations earn higher incentives.
 - Urgency-based: Zones with critical imbalances trigger higher rewards.
 - Vehicle type: Cars/vans get higher incentives than bikes/scooters due to higher relocation value.
 - Customer segmentation: Frequent users may receive personalized offers.
- **Incentive types:**
 - Percentage discounts (e.g., “50% off if you drop off in Zone B”)
 - Flat-rate bonuses (e.g., “Earn €5 credit for this relocation”)
 - Loyalty points multipliers (e.g., “2× points for relocations”)
- **Real-time updates:** Incentives adjust every 5–15 minutes based on live demand-supply conditions.​

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
- **Multi-Provider AI ([ADR-05](ADR_05_Orchestrator.md)):** Uses orchestrator to manage model deployments and updates

### Operational Practices
- **Model monitoring:** Track forecast accuracy, pricing elasticity, and incentive acceptance rates
- **Automated retraining:** Weekly model updates with latest demand patterns
- **Manual overrides:** Operations team can adjust pricing/incentives for special events or emergencies
- **Customer communication:** In-app notifications explain dynamic pricing and highlight relocation opportunities

## Consequences

### ✅ Positive Impacts

**Financial**
- 30-50% reduction in manual relocation costs (€10-15/vehicle → customer self-relocations)
- 10-20% revenue increase from dynamic pricing capturing willingness-to-pay during peak demand
- Improved fleet utilization and reduced idle inventory from better vehicle distribution

**Customer Experience**
- Higher availability reduces "no vehicles available" scenarios by 20%+
- Transparent dynamic pricing builds trust when supply/demand explained clearly
- Gamification of relocation incentives creates positive engagement

**Operational**
- Automated AI-driven pricing decisions reduce manual planning workload
- Predictive demand forecasts enable proactive vehicle positioning for events
- Data-driven insights continuously improve demand understanding

### ❌ Negative Impacts and Trade-offs

**Customer Risks**
- Surge pricing may be perceived as unfair (ride-sharing surge backlash)
- Price fluctuations and relocation incentives add complexity
- Lower-income customers may be priced out during high-demand periods

**Technical Risks**
- Inaccurate demand forecasts cause suboptimal pricing (too high = lost sales, too low = lost revenue)
- Pricing engine downtime forces fallback to static pricing
- Depends on high-quality real-time data from telemetry, weather APIs, and booking systems

**Business Risks**
- Over-discounting via excessive relocation incentives may exceed saved relocation costs
- Customers may game incentives (fake relocations, exploiting rules)
- Dynamic pricing must comply with EU consumer protection and non-discrimination laws
- Some cities may impose pricing caps or restrictions
