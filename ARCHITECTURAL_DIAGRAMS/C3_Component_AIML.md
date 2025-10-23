# C3: AI/ML Component Diagram

This diagram is a Level 3 view, zooming into the AI/ML Services Layer to show the key components that make up the intelligence of the platform.

```
┌────────────────────────────────────────────────────────────────────────┐
│                     AI/ML ARCHITECTURE DETAIL                          │
└────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    DATA INGESTION & PREPARATION                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Kafka Topics ───────► Feature Store (Feast/Tecton)                │
│  • bookings.*           • Real-time features                        │
│  • vehicles.*           • Historical aggregates                     │
│  • weather.data         • Feature versioning                        │
│  • events.data                                                      │
│                                                                     │
│  S3 Data Lake ────────► Offline Training Data                       │
│  • Historical bookings  • Anonymized datasets                       │
│  • Telemetry archives   • Regional partitioning                     │
│  • Customer behavior    • Compliance-filtered                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    MODEL TRAINING PIPELINES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Airflow DAGs (Batch Training):                                    │
│  ┌────────────────────────────────────────────────────┐            │
│  │  1. Demand Forecasting (Weekly)                    │            │
│  │     • Model: XGBoost / LSTM                        │            │
│  │     • Features: Time, weather, events, history     │            │
│  │     • Output: 15-min demand predictions per 500m   │            │
│  │                                                    │            │
│  │  2. Predictive Maintenance (Weekly)                │            │
│  │     • Model: Isolation Forest + LSTM               │            │
│  │     • Features: Battery voltage curves, vibration  │            │
│  │     • Output: 7-14 day failure predictions         │            │
│  │                                                    │            │
│  │  3. Vision Models (Monthly)                        │            │
│  │     • Model: MobileNetV3 + YOLOv8-nano             │            │
│  │     • Training: Labeled damage images              │            │
│  │     • Output: Damage classification models         │            │
│  └────────────────────────────────────────────────────┘            │
│                                                                     │
│  MLflow / Weights & Biases:                                        │
│  • Experiment tracking                                             │
│  • Model versioning                                                │
│  • Hyperparameter tuning                                           │
│  • Performance metrics                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    MODEL REGISTRY & DEPLOYMENT                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Model Registry (MLflow):                                          │
│  • Versioned models per region                                     │
│  • Model metadata (training data, metrics, artifacts)              │
│  • Staging → Production promotion workflow                         │
│  • Rollback capability                                             │
│                                                                     │
│  Deployment Strategies:                                            │
│  • Canary rollouts (5% → 25% → 100%)                               │
│  • A/B testing infrastructure                                      │
│  • Shadow deployment for validation                                │
│  • Blue-green for zero-downtime updates                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    INFERENCE SERVICES (Runtime)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────┐              │
│  │  Demand Forecasting Service                       │              │
│  │  • Input: Location, time, weather, events         │              │
│  │  • Model: XGBoost/LSTM (region-specific)          │              │
│  │  • Output: Demand probability distribution        │              │
│  │  • Latency: <500ms                                │              │
│  │  • Publishes: predictions.demand_forecast         │              │
│  └───────────────────────────────────────────────────┘              │
│                                                                     │
│  ┌───────────────────────────────────────────────────┐             │
│  │  Dynamic Pricing Engine                           │             │
│  │  • Input: Demand forecast + current supply        │             │
│  │  • Logic: Supply-demand multipliers               │             │
│  │  • Output: Zone-specific pricing                  │             │
│  │  • Latency: <100ms                                │             │
│  │  • Publishes: pricing.updated                     │             │
│  └────────────────────────────────────────────────────              |
│                                                                     │
│  ┌───────────────────────────────────────────────────┐             │
│  │  Predictive Maintenance Service                   │             │
│  │  • Input: Vehicle telemetry streams               │             │
│  │  • Model: Isolation Forest + LSTM                 │             │
│  │  • Output: Failure probability + lead time        │             │
│  │  • Latency: Near real-time (streaming)            │             │
│  │  • Publishes: predictions.maintenance_alert       │             │
│  └───────────────────────────────────────────────────┘             │
│                                                                     │
│  ┌───────────────────────────────────────────────────┐             │
│  │  Vision AI Service (Damage Detection)             │             │
│  │  • Input: Pickup/dropoff photos                   │             │
│  │  • Model: MobileNetV3 + YOLOv8-nano               │             │
│  │  • Processing: Edge (privacy) + Cloud (backup)    │             │
│  │  • Output: Damage classification + confidence     │             │
│  │  • Latency: <2s                                   │             │
│  │  • Publishes: photos.verified                     │             │
│  └───────────────────────────────────────────────────┘             │
│                                                                     │
│  ┌───────────────────────────────────────────────────┐             │
│  │  Conversational AI (MCP Integration)              │             │
│  │  • Input: User text/voice messages                │             │
│  │  • NLU: Multi-provider LLM (OpenAI/Anthropic)     │             │
│  │  • Intent extraction + entity recognition         │             │
│  │  • MCP command execution (bookings, queries)      │             │
│  │  • Output: Natural language responses             │             │
│  │  • Latency: <3s                                   │             │
│  └───────────────────────────────────────────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```
