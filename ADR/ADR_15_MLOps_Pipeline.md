# ADR-15: MLOps Pipeline Architecture

## Status
Accepted

## Context
MobilityCorp's AI-enabled platform relies on multiple machine learning models for demand forecasting, dynamic pricing, predictive maintenance, damage detection, and personalization. These models must be:

- **Trained regularly** on fresh data to maintain accuracy
- **Versioned** for reproducibility and rollback
- **Monitored** for performance degradation and drift
- **Deployed** with zero downtime
- **A/B tested** to validate improvements
- **Governed** with audit trails and compliance

Without a robust MLOps pipeline, we risk:
- Model staleness leading to poor predictions
- No visibility into model performance degradation
- Manual, error-prone deployment processes
- Inability to roll back bad models
- Compliance and audit challenges

## Requirements

### Model Lifecycle Management
- Automated retraining on schedule or trigger
- Model versioning with metadata (metrics, hyperparameters, data version)
- Model registry for all production models
- Rollback capability to previous versions

### Model Monitoring
- Performance metrics tracking (accuracy, MAPE, F1, etc.)
- Data drift detection (input distribution changes)
- Model drift detection (prediction distribution changes)
- Alerting on degradation

### Deployment
- Canary and blue-green deployments
- A/B testing framework
- Multi-model endpoints (champion/challenger)
- Zero-downtime updates

### Governance
- Model lineage tracking
- Feature lineage tracking
- Audit logs for all model operations
- Compliance with AI regulations (EU AI Act)

### Integration
- Seamless integration with data platform (S3, Redshift, Feature Store)
- CI/CD integration for automated deployments
- Monitoring integration (CloudWatch, Grafana)

## Alternatives Considered

### Alternative 1: AWS SageMaker MLOps (Recommended)

#### Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                      MLOps Pipeline Flow                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Data Preparation                                             │
│     ├─ S3 (Raw Data: Bronze Layer)                              │
│     ├─ Airflow + BEAM DAGs.                                     │
│     └─ Feature Store (Online + Offline)                         │
│                                                                  │
│  2. Model Training (SageMaker Pipelines)                        │
│     ├─ Data Validation (Great Expectations)                     │
│     ├─ Feature Engineering                                       │
│     ├─ Hyperparameter Tuning (Automatic)                        │
│     ├─ Model Training (Spot Instances)                          │
│     ├─ Model Evaluation                                          │
│     └─ Model Registry (with approval workflow)                  │
│                                                                  │
│  3. Model Deployment                                             │
│     ├─ SageMaker Endpoints (Real-time inference)                │
│     ├─ Lambda (Batch inference)                                  │
│     ├─ Multi-Model Endpoints (multiple models per instance)     │
│     └─ A/B Testing (traffic splitting)                          │
│                                                                  │
│  4. Model Monitoring                                             │
│     ├─ SageMaker Model Monitor (drift detection)                │
│     ├─ CloudWatch Metrics (latency, errors)                     │
│     ├─ Custom Metrics (business KPIs)                           │
│     └─ Alerts (SNS → PagerDuty/Slack)                           │
│                                                                  │
│  5. Feedback Loop                                                │
│     ├─ Ground Truth Labeling (new data)                         │
│     ├─ Model Retraining Trigger                                 │
│     └─ Continuous Improvement                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Components

**SageMaker Pipelines:**
- Orchestrates end-to-end ML workflows
- DAG-based with conditional steps
- Integrates with SageMaker Processing, Training, Tuning
- Supports pipeline versioning and caching

**SageMaker Feature Store:**
- Online store: Low-latency feature retrieval (DynamoDB)
- Offline store: Historical features for training (S3)
- Feature groups with versioning
- Point-in-time correct queries

**SageMaker Model Registry:**
- Centralized model repository
- Version management with lineage
- Approval workflows (pending → approved → deployed)
- Model metadata (metrics, hyperparameters, artifacts)

**SageMaker Endpoints:**
- Real-time inference with auto-scaling
- Multi-model endpoints (cost optimization)
- Async inference for batch predictions
- Built-in A/B testing and traffic splitting

**SageMaker Model Monitor:**
- Data quality monitoring
- Model quality monitoring
- Bias drift detection
- Feature attribution drift

**SageMaker Clarify:**
- Bias detection in training data
- Model explainability (SHAP values)
- Feature importance analysis

#### Example Pipeline: Demand Forecasting Model

**SageMaker Pipeline Components:**

1. **Data Validation & Feature Engineering:**
   - ProcessingStep runs data preprocessing script
   - Validates data quality and generates features
   - Outputs training and validation datasets to S3

2. **Hyperparameter Tuning & Training:**
   - TrainingStep with LightGBM estimator
   - Consumes preprocessed datasets
   - Trains model with optimized hyperparameters

3. **Model Evaluation:**
   - ProcessingStep runs evaluation script
   - Calculates metrics (MAPE, RMSE, R²)
   - Outputs evaluation report as JSON

4. **Conditional Model Registration:**
      - Model Evaluation
   - Registers model if quality threshold met
   - Assigns "PendingManualApproval" status
   - Adds model to Model Registry

5. **Deployment:**
   - Manual approval process via EventBridge + Lambda
   - Blue-green deployment to minimize downtime

#### Strengths
✅ **Fully Managed:** No infrastructure to manage
✅ **Integrated:** Native integration with AWS services
✅ **Feature Store:** Built-in online/offline feature serving
✅ **Model Monitor:** Automatic drift detection
✅ **Cost-Effective:** Spot instances for training with significant savings
✅ **Auto-Scaling:** Endpoints scale based on traffic
✅ **Multi-Model:** Host multiple models on one endpoint
✅ **Proven:** Used by Intuit, Lyft, Vanguard at scale

#### Weaknesses
❌ **AWS Lock-in:** Tightly coupled to AWS
❌ **Learning Curve:** SageMaker-specific concepts
❌ **Debugging:** Harder to debug than self-managed
❌ **Flexibility:** Less flexible than custom solutions

#### Cost Considerations

**Infrastructure Components:**
- Model Training: Spot instances for cost optimization, GPU instances for deep learning
- Model Endpoints: Multi-instance deployment for high availability with auto-scaling
- Feature Store: Online (low-latency serving) and Offline (training) storage
- Model Monitor: Automated data quality and model quality checks
- SageMaker Pipelines: Workflow orchestration for training and deployment

**Cost Optimization Strategies:**
- Spot instances reduce training costs significantly
- Auto-scaling adjusts endpoint capacity based on demand
- Feature store caching reduces redundant computations
- Batch inference for non-real-time predictions

#### Scoring (out of 10)
- Ease of Use: 8/10
- Feature Completeness: 10/10
- Cost: 8/10
- Flexibility: 7/10
- **Overall: 8.8/10**

---

### Alternative 2: MLflow + Kubernetes (Self-Managed)

#### Architecture
```
Kubernetes Cluster (EKS)
├─ MLflow Server (Tracking + Model Registry)
├─ Airflow + Beam (Pipeline Orchestration)
├─ Seldon Core / KFServing (Model Serving)
├─ Prometheus + Grafana (Monitoring)
└─ Feast (Feature Store)
```

#### Components
- **MLflow:** Model tracking, registry, deployment
- **Airflow + Beam:** Pipeline orchestration
- **Kubeflow / Seldon:** Model serving on Kubernetes
- **Feast:** Open-source feature store
- **Prometheus:** Metrics collection
- **Evidently AI:** Drift detection

#### Strengths
✅ **Open Source:** No vendor lock-in
✅ **Flexibility:** Full control over all components
✅ **Portability:** Runs anywhere (AWS, GCP, Azure, on-prem)
✅ **Community:** Large open-source community
✅ **Customization:** Extend to fit exact needs

#### Weaknesses
❌ **Operational Overhead:** Must manage all infrastructure
❌ **Engineering Time:** Significant time to build production-ready
❌ **Expertise Required:** Need Kubernetes + MLOps expertise
❌ **Maintenance:** Ongoing patches, upgrades, debugging
❌ **No Auto-Scaling:** Must implement yourself
❌ **Integration:** Custom integrations with AWS services

#### Cost (Monthly, Production)
```
Infrastructure Components:
- EKS Cluster: Control plane management
- EC2 Instances: Worker nodes
- Storage: Persistent volumes for artifacts and models
- Load Balancers: Traffic distribution
- Engineering resources for setup and maintenance
```

#### Scoring
- Ease of Use: Moderate
- Feature Completeness: Good
- Cost: Moderate (includes engineering overhead)
- Flexibility: Excellent
- **Overall: Viable but requires significant effort**

---

### Alternative 3: Databricks MLOps

#### Architecture
```
Databricks Workspace
├─ MLflow (managed, built-in)
├─ Delta Lake (feature store)
├─ Databricks Workflows (orchestration)
├─ Model Serving (real-time endpoints)
└─ Lakehouse Monitoring (drift detection)
```

#### Strengths
✅ **Unified Platform:** Data + ML in one place
✅ **Delta Lake:** ACID transactions for ML features
✅ **Collaborative:** Notebooks for experimentation
✅ **Unity Catalog:** Data governance and lineage
✅ **Auto-scaling:** Clusters scale automatically

#### Weaknesses
❌ **Cost:** Higher operational costs than AWS-native
❌ **Vendor Lock-in:** Databricks-specific
❌ **AWS Integration:** Not as tight as SageMaker
❌ **Feature Store:** Less mature than SageMaker

#### Cost Considerations
**Infrastructure Components:**
- Databricks Workspace licensing
- Compute resources for training and serving
- Storage (Delta Lake)
- MLflow Enterprise features

#### Scoring (out of 10)
- Ease of Use: 9/10
- Feature Completeness: 8/10
- Cost: 5/10 ⚠️
- Flexibility: 7/10
- **Overall: 7.2/10**

---

## Decision Matrix

| Criteria | SageMaker | MLflow+K8s | Databricks |
|----------|-----------|------------|------------|
| **Ease of Use** | Good | Moderate | Excellent |
| **Feature Completeness** | Excellent | Good | Good |
| **Cost** | Good | Moderate | Higher |
| **AWS Integration** | Excellent | Moderate | Good |
| **Time to Production** | Fast | Slow | Fast |
| **Flexibility** | Good | Excellent | Good |
| **Overall** | **Best fit** | **Complex** | **Costly** |

## Decision

**We choose AWS SageMaker for our MLOps platform.**

### Primary Rationale

#### 1. Fully Managed & Production-Ready
- **No infrastructure management:** Focus on models, not infrastructure
- **Built-in best practices:** Drift detection, A/B testing, monitoring
- **Auto-scaling:** Handles traffic spikes automatically
- **Multi-region:** Easy to deploy across Frankfurt + Ireland

#### 2. Comprehensive Feature Set
- **Feature Store:** Online + offline with point-in-time correctness
- **Model Monitor:** Automatic drift detection out-of-the-box
- **Pipelines:** DAG-based ML workflows with caching
- **Clarify:** Bias detection and explainability
- **Ground Truth:** Data labeling for continuous improvement

#### 3. Cost-Effective with Spot Instances
- **Training cost optimization:** Spot instances for all training jobs
- **Multi-model endpoints:** Host multiple models on shared infrastructure
- **Serverless inference:** Pay only for usage (batch jobs)
- **Cost advantage** vs MLflow+K8s or Databricks alternatives

#### 4. AWS Ecosystem Integration
- Native integration with S3, Glue, Redshift, Lambda
- EventBridge triggers for automation
- IAM for fine-grained access control
- CloudWatch for unified monitoring

#### 5. Time to Market
- Rapid production deployment compared to self-managed solutions
- Pre-built containers for popular frameworks (TensorFlow, PyTorch, XGBoost, LightGBM)
- Jupyter notebooks for experimentation
- No Kubernetes expertise required

### Why Not MLflow + Kubernetes?

MLflow is excellent but:
- **Longer implementation time** to production-ready state
- **Higher operational burden:** Kubernetes cluster management
- **Engineering overhead:** When factoring setup and maintenance
- **Skills gap:** Team doesn't have deep Kubernetes expertise

**MLflow would be better if:**
- We had strong multi-cloud requirements
- We had existing Kubernetes expertise
- We needed extreme customization
- We had extended timeline for platform development

```



```

### Why Not Databricks?

Databricks is a great unified platform but:
- **Higher costs** compared to SageMaker
- **Less AWS-native** than SageMaker
- **Overkill for our use case:** We don't need unified data+ML workspace
- Feature Store less mature than SageMaker

**Databricks would be better if:**
- We were already using Databricks for data engineering
- We needed tight Spark integration
- Budget wasn't a constraint
- We had complex data engineering + ML workflows

## Implementation Details

### Model Inventory

| Model | Purpose | Algorithm | Retraining | Endpoint Type |
|-------|---------|-----------|------------|---------------|
| **Demand Forecast** | Zone-level demand prediction | LightGBM | Daily | Real-time |
| **Dynamic Pricing** | Optimal price calculation | XGBoost | Hourly | Real-time |
| **Battery Degradation** | RUL prediction | Random Forest | Weekly | Batch |
| **Damage Detection** | Vehicle damage classification | ResNet-50 | Monthly | Real-time |
| **Route Optimization** | Optimal route suggestions | Graph Neural Network | Weekly | Real-time |
| **Customer Segmentation** | User clustering | K-Means | Monthly | Batch |
| **Churn Prediction** | User churn probability | Logistic Regression | Weekly | Batch |
| **Sentiment Analysis** | Feedback classification | BERT | Bi-weekly | Real-time |

### Pipeline Architecture per Model

#### 1. Demand Forecasting Pipeline (Daily)

```
Trigger: Daily at 2 AM UTC
├─ Extract: Last 90 days booking data (Redshift → S3)
├─ Feature Engineering:
│  ├─ Time features (hour, day_of_week, month)
````

## Implementation Details

### Model Inventory

| Model | Purpose | Algorithm | Retraining | Endpoint Type |
|-------|---------|-----------|------------|---------------|
| **Demand Forecast** | Zone-level demand prediction | LightGBM | Daily | Real-time |
| **Dynamic Pricing** | Optimal price calculation | XGBoost | Hourly | Real-time |
| **Battery Degradation** | RUL prediction | Random Forest | Weekly | Batch |
| **Damage Detection** | Vehicle damage classification | ResNet-50 | Monthly | Real-time |
| **Route Optimization** | Optimal route suggestions | Graph Neural Network | Weekly | Real-time |
| **Customer Segmentation** | User clustering | K-Means | Monthly | Batch |
| **Churn Prediction** | User churn probability | Logistic Regression | Weekly | Batch |
| **Sentiment Analysis** | Feedback classification | BERT | Bi-weekly | Real-time |

### Pipeline Architecture per Model

#### 1. Demand Forecasting Pipeline

```
Trigger: Scheduled daily run
├─ Extract: Historical booking data (Redshift → S3)
├─ Feature Engineering:
│  ├─ Time features
│  ├─ Weather data (OpenWeatherMap API)
│  ├─ Events data (PredictHQ API)
│  ├─ Historical features (rolling averages, lag features)
│  └─ Feature Store update
├─ Data Validation (Great Expectations):
│  ├─ Schema validation
│  ├─ Range checks
│  └─ Null checks
├─ Train/Validation Split: Time-based
├─ Hyperparameter Tuning (Bayesian Optimization)
├─ Model Training (LightGBM on Spot instances)
├─ Model Evaluation:
│  ├─ MAPE target threshold
│  ├─ RMSE calculation
│  └─ Residual analysis
├─ Conditional Registration (if quality threshold met):
│  ├─ Register in Model Registry
│  ├─ Approval Status: PendingManualApproval
│  └─ Notify ML team via Slack
├─ Manual Approval:
│  ├─ Review evaluation metrics
│  ├─ A/B test plan
│  └─ Approve for deployment
└─ Deployment:
   ├─ Canary: Small traffic for validation period
   ├─ Monitor: Error rate, latency, accuracy metrics
   ├─ If OK: Blue-green swap to full traffic
   └─ If NOT OK: Rollback to previous version
```

#### 2. Damage Detection Pipeline

```
Trigger: Scheduled monthly + on-demand for new labeled data
├─ Extract: New damage photos (S3 + Ground Truth labels)
├─ Data Augmentation
│  ├─ Color jittering
│  └─ Synthetic data generation
├─ Transfer Learning (ResNet-50 pre-trained on ImageNet)
├─ Fine-Tuning (on Spot instances):
│  ├─ Freeze initial layers
│  ├─ Train final layers on damage classes
│  └─ Learning rate schedule
├─ Model Evaluation:
│  ├─ Accuracy target threshold
│  ├─ Precision/Recall per class
│  ├─ Confusion matrix
│  └─ F1 score
├─ Bias Detection (SageMaker Clarify):
│  ├─ Check for vehicle type bias
│  ├─ Check for lighting condition bias
│  └─ Mitigation strategies if needed
├─ Registration + Approval
└─ Deployment (Multi-Model Endpoint):
   ├─ Multiple models on shared endpoint (cost savings)
   ├─ A/B testing capability
   └─ Monitor: Inference latency
```

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
│  CloudWatch Metrics:                                  │
│  ├─ Endpoint latency (percentiles)                   │
│  ├─ Error rate (4xx, 5xx)                            │
│  ├─ Invocations per minute                           │
│  └─ Model loading time                               │
│                                                       │
│  Alerting (SNS → PagerDuty):                         │
│  ├─ Critical: Accuracy below threshold (page on-call)│
│  ├─ High: Data drift detected (Slack alert)          │
│  ├─ Medium: Latency threshold exceeded (email)       │
│  └─ Low: Training job failed (Slack)                 │
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

### CI/CD Integration

```
GitHub Actions Workflow:

1. On PR to main:
   ├─ Lint code (Ruff, Black)
   ├─ Unit tests (pytest)
   ├─ Integration tests
   └─ SageMaker pipeline validation

2. On Merge to main:
   ├─ Build Docker images
   ├─ Push to ECR
   ├─ Update SageMaker pipeline
   └─ Trigger pipeline execution (if config changed)

3. On Pipeline Success:
   ├─ Notify ML team (Slack)
   ├─ Update Model Registry
   └─ Await manual approval for deployment

4. On Manual Approval:
   ├─ Deploy to staging endpoint
   ├─ Run E2E tests
   ├─ If OK: Deploy to production (canary)
   └─ If NOT OK: Rollback + investigate
```

## Consequences

### Positive
✅ **Fast Time to Market:** 4-6 weeks vs 3-6 months for self-managed
✅ **Reduced Operational Burden:** No infrastructure management
✅ **Cost-Effective:** $28K/month with spot instances
✅ **Built-in Best Practices:** Drift detection, A/B testing, monitoring
✅ **Auto-Scaling:** Handles traffic spikes automatically
✅ **Model Registry:** Version control and lineage tracking
✅ **Feature Store:** Online + offline feature serving
✅ **Proven at Scale:** Used by major enterprises

### Negative
❌ **AWS Lock-in:** Tightly coupled to SageMaker
❌ **Learning Curve:** SageMaker-specific concepts
❌ **Debugging:** Harder to debug than local development
❌ **Customization Limits:** Less flexible than custom solutions
❌ **Cost at Low Scale:** Minimum endpoint costs even with low traffic

### Mitigation Strategies

**Lock-in Mitigation:**
- Abstract SageMaker APIs behind internal interfaces
- Store model artifacts in portable format (ONNX, PMML)
- Use open-source algorithms (LightGBM, XGBoost)
- Document migration path to MLflow

**Learning Curve:**
- Team training (AWS ML course)
- Internal documentation and examples
- Pair programming for knowledge sharing

**Debugging:**
- Local testing with SageMaker Local Mode
- Comprehensive logging to CloudWatch
- Step-by-step pipeline execution

## Related Decisions
- **ADR-16:** Data Lakehouse Strategy - Feature Store integration
- **ADR-17:** Agentic AI Framework - Model serving for LLMs
- **ADR-06:** Event-Driven Architecture - Triggers for retraining

## References
- [SageMaker MLOps Workshop](https://github.com/aws-samples/amazon-sagemaker-mlops-workshop)
- [SageMaker Pipelines Documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html)
- [SageMaker Feature Store](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html)
- [ML Engineering Book](https://www.mlebook.com/) by Andriy Burkov