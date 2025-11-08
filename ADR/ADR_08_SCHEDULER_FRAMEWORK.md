# ADR-08: Scheduler Framework for AI and Data Pipelines

## Status
Accepted  

## Context
The AI platform runs multiple recurring and event-based workflows, including:

- ML training pipelines (weekly retraining for demand prediction, damage detection, and anomaly models)
- Data pipelines (daily ETL jobs, feature computation, drift monitoring)
- System maintenance tasks (cache warmups, cleanup, report generation)
- AI validation workflows (A/B testing, shadow deployments, retraining triggers)

Requirements:
- Central scheduling and orchestration of workflows
- Monitoring, retries, and alerting for failed jobs
- Cron-based and event-triggered executions
- Dependency management between steps (e.g., feature extraction → model training → deployment)
- Integration with Prometheus, Grafana, and Slack alerts

Alternatives considered:
1. **Airflow only:** Simple, mature DAG orchestration, but limited for real-time tasks  
2. **Temporal only:** Reliable event-driven orchestration, but lacks rich DAG visualization  
3. **Kubernetes CronJobs:** Simple setup, but no dependency tracking or retry orchestration

## Decision
Adopt **AWS Step Functions** as the primary orchestration framework, with **EventBridge Scheduler** for simple cron jobs and **AWS Managed Workflows for Apache Airflow (MWAA)** for complex data pipelines requiring rich DAG visualization.

**AWS Implementation:**

**1. AWS Step Functions (Primary Orchestrator)**
- **Use cases:**
  - ML training pipelines (SageMaker integration)
  - Multi-step data transformations (Glue ETL → Athena → Redshift)
  - Event-driven workflows (Kafka event → Lambda → SageMaker inference → DynamoDB)
  - Model deployment pipelines (Train → Evaluate → A/B test → Deploy)
- **Features:**
  - **Visual workflow editor:** Drag-and-drop state machine designer
  - **Error handling:** Automatic retries, catch blocks, fallback states
  - **Service integrations:** Native SDK for SageMaker, Glue, Lambda, DynamoDB, SNS, SQS
  - **Monitoring:** CloudWatch metrics, X-Ray distributed tracing
- **ML Model Retraining Pipeline Workflow:**
  1. **PrepareTrainingData:** AWS Glue job for data preparation with retry logic
  2. **TrainModel:** SageMaker training job
  3. **EvaluateModel:** Lambda function evaluates model metrics
  4. **CheckMetrics:** Conditional logic validates performance thresholds
  5. **DeployModel:** SageMaker endpoint creation if metrics pass
  6. **NotifySuccess/NotifyFailure:** SNS notifications to operations channels
  - Error handling includes automatic retries, catch blocks, and fallback states
  - All steps include CloudWatch logging and X-Ray tracing
- **Triggers:**
  - **EventBridge schedule:** Periodic retraining on defined schedule
  - **EventBridge rule:** Model drift detected → trigger retraining
  - **Kafka event:** Drift events → Lambda → Start Step Functions execution

**2. Amazon EventBridge Scheduler (Simple Cron Jobs)**
- **Use cases:**
  - Daily cache warmup (Lambda invocation)
  - Periodic data quality checks (Lambda → Athena query)
  - Cleanup jobs (S3 lifecycle policies, DynamoDB TTL)
- **Features:**
  - **Flexible schedules:** Cron expressions, rate expressions, one-time executions
  - **Target integrations:** Lambda, Step Functions, SageMaker, ECS tasks, API Gateway
  - **Retry policies:** Configurable retry attempts with exponential backoff
- **Daily Demand Forecast Batch Inference Schedule:**
  - Scheduled daily execution via cron expression
  - Invokes Step Functions state machine for batch processing
  - Configurable retry policy for resilience
  - IAM role for EventBridge Scheduler execution

**3. AWS MWAA (Managed Airflow) - Optional for Complex DAGs**
- **Use cases:** Complex data pipelines with many interdependent tasks
  - Example: End-to-end data lakehouse pipeline (Bronze → Silver → Gold transformations)
- **When to use:** When Step Functions becomes unwieldy, use MWAA for better DAG visualization
- **Deployment:** Right-sized environment based on pipeline complexity
- **Trade-off:** Higher cost but richer UI and Python-based DAG definitions
- **Decision:** Start with Step Functions, migrate specific workflows to MWAA if needed

**Monitoring & Alerting:**
- **CloudWatch Logs:** All workflow executions logged with X-Ray trace IDs
- **CloudWatch Alarms:** Alert on workflow failures, timeouts, high execution duration
- **SNS Topics:**
  - Routine notifications (Slack integration)
  - Critical alerts (PagerDuty integration for immediate response)
- **CloudWatch Dashboard:** Visual overview of all scheduled jobs
  - Success/failure rates
  - Average execution duration per workflow
  - Cost tracking per workflow

## Consequences
✅ **Positive:**
- Unified orchestration for both batch and event-driven AI workflows  
- Reliable retries and task recovery (Temporal persistence)  
- Clear DAG management and visualization (Airflow UI)  
- Scalable worker-based execution  
- Simplified dependency tracking and logging

❌ **Negative:**
- Two systems to manage (Airflow + Temporal)  
- Added operational complexity and setup effort  
- Temporal workflow definitions require SDK familiarity  
- Higher infra overhead compared to a single orchestrator

AI-Specific Impacts:
- Retraining triggered via Temporal when drift thresholds exceed limits  
- Shadow deployments scheduled in Airflow  
- Feature pipelines and model training synchronized to prevent drift  
- Slack and Grafana integrated for monitoring job outcomes
