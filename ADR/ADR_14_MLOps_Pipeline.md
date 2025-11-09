# ADR-14: MLOps Pipeline Architecture

- Status: Accepted
- Deciders: ML Platform Team, Data Science Lead
- Drivers: Reproducible, observable, cost-efficient model lifecycle for forecasting, pricing, maintenance, and personalization

## Context
Multiple ML models (demand forecasting, dynamic pricing, predictive maintenance, damage detection, personalization, dynamic insurance pricing) require automated retraining, versioning, low-latency serving, drift detection, safe rollouts, and audit trails for SLAs and compliance (EU AI Act).

## Decision
Adopt AWS-native MLOps using **Apache Airflow** (orchestration) and **Amazon SageMaker** (lifecycle management) with **Kafka** for streaming predictions.

```
┌─────────────────────────────────────────────────────────────────┐
│                      MLOps Pipeline Flow                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Orchestration (Apache Airflow)                               │
│     ├─ DAGs trigger SageMaker Pipelines via operators           │
│     ├─ Schedule: Daily/Weekly/Monthly retraining                │
│     └─ Approval gates for model deployment                      │
│                                                                  │
│  2. Feature Store (SageMaker - Fully Managed)                   │
│     ├─ Online Store: DynamoDB (AWS-managed, zero config)        │
│     ├─ Offline Store: S3 (Parquet, for training)               │
│     ├─ Feature Groups: user, vehicle, zone, weather, events    │
│     └─ Point-in-time correct queries for historical joins       │
│                                                                  │
│  3. Model Training (SageMaker Pipelines)                        │
│     ├─ Data Validation (Great Expectations)                     │
│     ├─ Feature Engineering → Feature Store ingestion           │
│     ├─ Hyperparameter Tuning (Bayesian Optimization)           │
│     ├─ Training (Spot Instances for cost savings)              │
│     ├─ Model Evaluation (custom metrics)                        │
│     └─ Model Registry (versioning + approval workflow)         │
│                                                                  │
│  4. Model Serving                                                │
│     ├─ Real-Time: SageMaker Endpoints (auto-scaling)            │
│     ├─ Streaming: Async Inference (Kafka consumer → S3 queue)  │
│     ├─ Batch: SageMaker Transform Jobs (Airflow-triggered)     │
│     └─ A/B Testing: Traffic splitting on endpoints             │
│                                                                  │
│  5. Streaming Integration (Kafka)                                │
│     ├─ Kafka topics receive real-time events                    │
│     ├─ Consumer app: Fetch features from Feature Store         │
│     ├─ Invoke: SageMaker Async Endpoint (S3-backed queue)      │
│     └─ Response: Predictions back to Kafka or downstream       │
│                                                                  │
│  6. Monitoring & Observability                                   │
│     ├─ SageMaker Model Monitor → CloudWatch (drift detection)  │
│     ├─ CloudWatch → YACE exporter → Victoria Metrics           │
│     ├─ OpenTelemetry: Custom spans for SageMaker invocations   │
│     ├─ Logs: CloudWatch → Firehose → OpenSearch               │
│     └─ Alerts: Victoria Metrics → PagerDuty                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Consequences

✅ **Positive:**
- Fast time-to-prod with managed services and AWS-native integrations
- Scalable feature serving with point-in-time correctness
- Built-in drift detection, versioning, and audit trails
- Cost optimization via spot instances, multi-model endpoints, batch inference

❌ **Negative:**
- AWS vendor lock-in increases future migration costs
- Requires SageMaker-specific operational knowledge
- Distributed debugging complexity vs monolithic systems

**Operations:** CI/CD pipelines, IAM least-privilege, approval gates, per-model retraining cadence

## Alternatives Considered
**MLflow + Kubernetes** — Rejected: High ops overhead, longer time-to-prod, deep K8s expertise required from ML team as well

**Generative AI (GenAI/LLMs)** — Rejected: Our use cases (demand forecasting, dynamic pricing, predictive maintenance, risk scoring) require deterministic, interpretable predictions with quantifiable accuracy metrics. GenAI excels at content generation and natural language tasks but lacks the precision, explainability, and cost-efficiency needed for structured prediction problems where traditional ML models consistently outperform while maintaining regulatory compliance and audit trails.

## ML Use Case: Dynamic Insurance Pricing
- Insurance tiers: Basic (minimal coverage), Standard (balanced), Premium (comprehensive).
- Telemetry (speed, braking, acceleration) is used to compute driving risk scores via ML.
- Behavioral nudge: Standard tier priced closer to premium to encourage upgrades to Premium.
- Data: historical telemetry, claims, and driving patterns; retrain weekly.
- Serving: real-time risk scoring integrated into the booking flow.
Example Case study - 


### Feature Store Architecture

```
┌──────────────────────────────────────────────────────┐
│              SageMaker Feature Store                  │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Feature Groups:                                      │
│  ├─ user_features (profile, history, preferences)    │
│  ├─ vehicle_features (specs, telemetry, maintenance) │
│  ├─ zone_features (demand, pricing, availability)    │
│  ├─ weather_features (temp, rain, wind)              │
│  └─ event_features (concerts, sports, holidays)      │
│                                                       │
│  Online Store (DynamoDB):                             │
│  ├─ Low-latency reads (< 10ms P99)                   │
│  ├─ Used for real-time inference                     │
│  └─ TTL for automatic cleanup                        │
│                                                       │
│  Offline Store (S3):                                  │
│  ├─ Historical features for training                 │
│  ├─ Point-in-time correct queries                    │
│  ├─ Parquet format for efficient reads               │
│  └─ Athena queries for analysis                      │
│                                                       │
└──────────────────────────────────────────────────────┘
```

### Monitoring & Alerting

```
┌──────────────────────────────────────────────────────┐
│                 Model Monitoring                      │
├──────────────────────────────────────────────────────┤
│                                                       │
│  SageMaker Model Monitor:                             │
│  ├─ Data Quality (hourly):                           │
│  │  ├─ Feature distribution drift                    │
│  │  ├─ Missing values                                │
│  │  └─ Schema violations                             │
│  │                                                    │
│  ├─ Model Quality (daily):                           │
│  │  ├─ Prediction accuracy vs ground truth           │
│  │  ├─ MAPE, RMSE, F1 score                          │
│  │  └─ Confusion matrix                              │
│  │                                                    │
│  ├─ Bias Drift (weekly):                             │
│  │  ├─ Check for emerging biases                     │
│  │  └─ Fairness metrics                              │
│  │                                                    │
│  └─ Feature Attribution Drift (weekly):               │
│     ├─ SHAP value changes                            │
│     └─ Feature importance shifts                     │
│                                                       │
│  CloudWatch Logs/Metrics:                            │
│  ├─ Exported to Otel Collector for regular           |
|  |  log & alert flow                                 │
│  └─ Model loading time                               │
│                                                      
│                                                       │
└──────────────────────────────────────────────────────┘
```

### Deployment Strategies

#### Real-Time Endpoints (Demand, Pricing, Damage)
```
Strategy: Canary → Blue-Green

1. Canary Deployment:
   - Deploy new model as Variant 2
   - Route small percentage of traffic to new model
   - Monitor for validation period:
     ✓ Latency within acceptable range
     ✓ Error rate within threshold
     ✓ Accuracy metrics within tolerance
   
2. If Canary Success:
   - Gradually increase traffic percentage
   - Monitor at each step
   
3. If Canary Failure:
   - Instant rollback to previous variant
   - Investigate issues
   - Re-deploy after fixes

4. Blue-Green Swap:
   - Once at 100% on Variant 2
   - Delete Variant 1 (old model)
   - Variant 2 becomes primary
```

#### Batch Endpoints (Battery, Churn, Segmentation)
```
Strategy: Shadow Mode → Full Replacement

1. Shadow Deployment:
   - Run new model alongside old model
   - Compare predictions on same data
   - Analyze differences
   
2. If Agreement > 95%:
   - Replace old model with new model
   - No traffic splitting needed
   
3. If Agreement < 95%:
   - Investigate discrepancies
   - Domain expert review
   - Fix and re-test
```

### Retraining Triggers

| Trigger Type | Condition | Action |
|--------------|-----------|--------|
| **Schedule** | Daily at 2 AM | Demand forecasting |
| **Schedule** | Weekly Sunday | Battery degradation |
| **Schedule** | Monthly 1st | Damage detection |
| **Drift Detection** | Data drift > 10% | Retrain immediately |
| **Performance** | MAPE > 20% for 3 days | Retrain + investigate |
| **Manual** | New feature added | On-demand retrain |
| **Data Volume** | 10K new labels | Damage model retrain |

## Consequences

### Positive
✅ **Fast Time to Market:** 4-6 weeks vs 3-6 months for self-managed
✅ **Reduced Operational Burden:** No infrastructure management
✅ **Auto-Scaling:** Handles traffic spikes automatically
✅ **Model Registry:** Version control and lineage tracking
✅ **Feature Store:** Online + offline feature serving
✅ **Proven at Scale:** Used by major enterprises

### Negative
❌ **Customization Limits:** Less flexible than custom solutions
❌ **Cost at Low Scale:** Minimum endpoint costs even with low traffic

## Related Decisions
- **ADR-16:** Data Lakehouse Strategy - Feature Store integration

- **ADR-06:** Event-Driven Architecture - Triggers for retraining

## References
- [SageMaker MLOps Workshop](https://github.com/aws-samples/amazon-sagemaker-mlops-workshop)
- [SageMaker Pipelines Documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html)
- [SageMaker Feature Store](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html)
- [ML Engineering Book](https://www.mlebook.com/) by Andriy Burkov