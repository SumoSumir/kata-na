# ADR-16: MLOps Pipeline Architecture

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
│     ├─ Glue ETL / Airflow DAGs                                  │
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

```python
# Simplified SageMaker Pipeline Definition
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import ProcessingStep, TrainingStep, CreateModelStep

# Step 1: Data Validation & Feature Engineering
processing_step = ProcessingStep(
    name="PreprocessData",
    processor=sklearn_processor,
    code="preprocess.py",
    inputs=[
        ProcessingInput(source=s3_input_data, destination="/opt/ml/processing/input")
    ],
    outputs=[
        ProcessingOutput(output_name="train", source="/opt/ml/processing/train"),
        ProcessingOutput(output_name="validation", source="/opt/ml/processing/validation")
    ]
)

# Step 2: Hyperparameter Tuning & Training
training_step = TrainingStep(
    name="TrainModel",
    estimator=lightgbm_estimator,
    inputs={
        "train": TrainingInput(
            s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["train"].S3Output.S3Uri
        ),
        "validation": TrainingInput(
            s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["validation"].S3Output.S3Uri
        )
    }
)

# Step 3: Model Evaluation
evaluation_step = ProcessingStep(
    name="EvaluateModel",
    processor=sklearn_processor,
    code="evaluate.py",
    inputs=[
        ProcessingInput(source=training_step.properties.ModelArtifacts.S3ModelArtifacts)
    ],
    outputs=[
        ProcessingOutput(output_name="evaluation", source="/opt/ml/processing/evaluation")
    ],
    property_files=[
        PropertyFile(name="EvaluationReport", output_name="evaluation", path="evaluation.json")
    ]
)

# Step 4: Conditional Model Registration (only if MAPE < 15%)
register_step = CreateModelStep(
    name="RegisterModel",
    model=model,
    model_package_group_name="demand-forecasting-models",
    approval_status="PendingManualApproval",
    depends_on=[evaluation_step]
)

# Step 5: Deploy to Endpoint (manual approval required)
# Handled separately via EventBridge + Lambda

# Create Pipeline
pipeline = Pipeline(
    name="DemandForecastingPipeline",
    parameters=[...],
    steps=[processing_step, training_step, evaluation_step, register_step]
)

pipeline.upsert(role_arn=sagemaker_role)
pipeline.start()
```

#### Strengths
✅ **Fully Managed:** No infrastructure to manage
✅ **Integrated:** Native integration with AWS services
✅ **Feature Store:** Built-in online/offline feature serving
✅ **Model Monitor:** Automatic drift detection
✅ **Cost-Effective:** Spot instances for training (70% savings)
✅ **Auto-Scaling:** Endpoints scale based on traffic
✅ **Multi-Model:** Host multiple models on one endpoint
✅ **Proven:** Used by Intuit, Lyft, Vanguard at scale

#### Weaknesses
❌ **AWS Lock-in:** Tightly coupled to AWS
❌ **Learning Curve:** SageMaker-specific concepts
❌ **Debugging:** Harder to debug than self-managed
❌ **Flexibility:** Less flexible than custom solutions

#### Cost (Monthly, Production)
```
Model Training (Spot Instances):       $8,000
  - 10 models × 2 retrainings/month
  - ml.p3.2xlarge (GPU) for 20 hours
  
Model Endpoints (Real-time):          $15,000
  - 5 production endpoints
  - ml.m5.xlarge × 3 instances each (HA)
  - Auto-scaling 3-10 instances
  
Feature Store:                         $3,000
  - Online: 100M requests/month
  - Offline: 1TB storage
  
Model Monitor:                         $1,000
  - Data quality checks hourly
  - Model quality checks daily
  
SageMaker Pipelines:                   $1,000
  - 50 pipeline executions/month
────────────────────────────────────────────
TOTAL:                                $28,000
```

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
├─ Airflow (Pipeline Orchestration)
├─ Seldon Core / KFServing (Model Serving)
├─ Prometheus + Grafana (Monitoring)
└─ Feast (Feature Store)
```

#### Components
- **MLflow:** Model tracking, registry, deployment
- **Airflow:** Pipeline orchestration
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
❌ **Engineering Time:** 3-6 months to build production-ready
❌ **Expertise Required:** Need Kubernetes + MLOps expertise
❌ **Maintenance:** Ongoing patches, upgrades, debugging
❌ **No Auto-Scaling:** Must implement yourself
❌ **Integration:** Custom integrations with AWS services

#### Cost (Monthly, Production)
```
EKS Cluster (Control Plane):          $2,160
EC2 Instances (12 × m5.2xlarge):     $16,000
EBS Storage (500GB):                  $1,000
ELB Load Balancers:                     $500
S3 Storage (artifacts):                 $500
Engineering Time (1 FTE):            $15,000 (amortized)
────────────────────────────────────────────
TOTAL:                                $35,160
```

#### Scoring (out of 10)
- Ease of Use: 4/10
- Feature Completeness: 7/10
- Cost: 6/10 (higher with engineering time)
- Flexibility: 10/10
- **Overall: 6.8/10**

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
❌ **Cost:** Most expensive option (~$40K/month)
❌ **Vendor Lock-in:** Databricks-specific
❌ **AWS Integration:** Not as tight as SageMaker
❌ **Feature Store:** Less mature than SageMaker

#### Cost (Monthly, Production)
```
Databricks Workspace:                 $12,000
Compute (for training/serving):       $20,000
Storage (Delta Lake):                  $4,000
MLflow Enterprise:                     $4,000
────────────────────────────────────────────
TOTAL:                                $40,000
```

#### Scoring (out of 10)
- Ease of Use: 9/10
- Feature Completeness: 8/10
- Cost: 5/10 ⚠️
- Flexibility: 7/10
- **Overall: 7.2/10**

---

## Decision Matrix

| Criteria | Weight | SageMaker | MLflow+K8s | Databricks |
|----------|--------|-----------|------------|------------|
| **Ease of Use** | 20% | 8 | 4 | 9 |
| **Feature Completeness** | 25% | 10 | 7 | 8 |
| **Cost** | 20% | 8 | 6 | 5 |
| **AWS Integration** | 15% | 10 | 6 | 7 |
| **Time to Production** | 10% | 9 | 4 | 8 |
| **Flexibility** | 10% | 7 | 10 | 7 |
| **Weighted Score** | | **8.8** | **6.5** | **7.3** |

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
- **70% savings on training:** Spot instances for all training jobs
- **Multi-model endpoints:** Host 10+ models on one instance
- **Serverless inference:** Pay only for usage (batch jobs)
- **$28K/month** vs $35K (MLflow) or $40K (Databricks)

#### 4. AWS Ecosystem Integration
- Native integration with S3, Glue, Redshift, Lambda
- EventBridge triggers for automation
- IAM for fine-grained access control
- CloudWatch for unified monitoring

#### 5. Time to Market
- Production-ready in **4-6 weeks** vs 3-6 months for self-managed
- Pre-built containers for popular frameworks (TensorFlow, PyTorch, XGBoost, LightGBM)
- Jupyter notebooks for experimentation
- No Kubernetes expertise required

### Why Not MLflow + Kubernetes?

MLflow is excellent but:
- **3-6 months engineering time** to production-ready
- **Higher operational burden:** Kubernetes cluster management
- **More expensive:** When factoring engineering time
- **Skills gap:** Team doesn't have deep Kubernetes expertise

**MLflow would be better if:**
- We had strong multi-cloud requirements
- We had existing Kubernetes expertise
- We needed extreme customization
- We had 6+ months for platform development

### Why Not Databricks?

Databricks is a great unified platform but:
- **43% more expensive** ($40K vs $28K)
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
│  ├─ Weather data (OpenWeatherMap API)
│  ├─ Events data (PredictHQ API)
│  ├─ Historical features (rolling averages, lag features)
│  └─ Feature Store update
├─ Data Validation (Great Expectations):
│  ├─ Schema validation
│  ├─ Range checks
│  └─ Null checks
├─ Train/Validation Split: 80/20 time-based
├─ Hyperparameter Tuning (Bayesian Optimization):
│  ├─ num_leaves: [20, 100]
│  ├─ learning_rate: [0.01, 0.3]
│  ├─ n_estimators: [50, 500]
├─ Model Training (LightGBM on ml.p3.2xlarge Spot)
├─ Model Evaluation:
│  ├─ MAPE target: < 15%
│  ├─ RMSE calculation
│  └─ Residual analysis
├─ Conditional Registration (if MAPE < 15%):
│  ├─ Register in Model Registry
│  ├─ Approval Status: PendingManualApproval
│  └─ Notify ML team via Slack
├─ Manual Approval:
│  ├─ Review evaluation metrics
│  ├─ A/B test plan
│  └─ Approve for deployment
└─ Deployment:
   ├─ Canary: 5% traffic for 1 hour
   ├─ Monitor: Error rate, latency, MAPE
   ├─ If OK: Blue-green swap to 100%
   └─ If NOT OK: Rollback to previous version
```

#### 2. Damage Detection Pipeline (Monthly)

```
Trigger: Monthly + on-demand for new labeled data
├─ Extract: New damage photos (S3 + Ground Truth labels)
├─ Data Augmentation:
│  ├─ Random rotation, flips, crops
│  ├─ Color jittering
│  └─ Synthetic data generation
├─ Transfer Learning (ResNet-50 pre-trained on ImageNet)
├─ Fine-Tuning (on ml.p3.8xlarge Spot):
│  ├─ Freeze first N layers
│  ├─ Train final layers on damage classes
│  └─ Learning rate: 0.001 → 0.00001
├─ Model Evaluation:
│  ├─ Accuracy target: > 95%
│  ├─ Precision/Recall per class
│  ├─ Confusion matrix
│  └─ F1 score
├─ Bias Detection (SageMaker Clarify):
│  ├─ Check for vehicle type bias
│  ├─ Check for lighting condition bias
│  └─ Mitigation strategies if needed
├─ Registration + Approval
└─ Deployment (Multi-Model Endpoint):
   ├─ 4 models on 1 endpoint (cost savings)
   ├─ A/B test: 90/10 split
   └─ Monitor: Inference latency < 500ms
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
│  ├─ Endpoint latency (P50, P95, P99)                 │
│  ├─ Error rate (4xx, 5xx)                            │
│  ├─ Invocations per minute                           │
│  └─ Model loading time                               │
│                                                       │
│  Alerting (SNS → PagerDuty):                         │
│  ├─ Critical: MAPE > 20% (page on-call)              │
│  ├─ High: Data drift detected (Slack alert)          │
│  ├─ Medium: Latency P95 > 1s (email)                 │
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
   - Route 5% traffic to new model
   - Monitor for 1 hour:
     ✓ Latency < 200ms
     ✓ Error rate < 0.1%
     ✓ MAPE within 2% of current model
   
2. If Canary Success:
   - Gradually increase traffic: 10% → 25% → 50% → 100%
   - Monitor at each step (30 min intervals)
   
3. If Canary Failure:
   - Instant rollback to 100% Variant 1
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
- **ADR-15:** Cloud Provider Selection (AWS) - Enables SageMaker choice
- **ADR-17:** Data Lakehouse Strategy - Feature Store integration
- **ADR-18:** Agentic AI Framework - Model serving for LLMs
- **ADR-06:** Event-Driven Architecture - Triggers for retraining

## References
- [SageMaker MLOps Workshop](https://github.com/aws-samples/amazon-sagemaker-mlops-workshop)
- [SageMaker Pipelines Documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html)
- [SageMaker Feature Store](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html)
- [ML Engineering Book](https://www.mlebook.com/) by Andriy Burkov

## Revision History
- **2025-11-07:** Initial decision - SageMaker MLOps platform selected
- **Next Review:** 2026-02-07 (3 months) - After Phase 3 implementation
