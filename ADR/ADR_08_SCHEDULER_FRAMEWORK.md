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
- **Example workflow:** ML Model Retraining Pipeline
  ```json
  {
    "Comment": "Weekly demand forecast model retraining",
    "StartAt": "PrepareTrainingData",
    "States": {
      "PrepareTrainingData": {
        "Type": "Task",
        "Resource": "arn:aws:states:::glue:startJobRun.sync",
        "Parameters": {
          "JobName": "prepare-training-data-demand-forecast"
        },
        "Next": "TrainModel",
        "Catch": [{
          "ErrorEquals": ["States.ALL"],
          "Next": "NotifyFailure"
        }],
        "Retry": [{
          "ErrorEquals": ["States.ALL"],
          "IntervalSeconds": 60,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }]
      },
      "TrainModel": {
        "Type": "Task",
        "Resource": "arn:aws:states:::sagemaker:createTrainingJob.sync",
        "Parameters": {
          "TrainingJobName.$": "$.TrainingJobName",
          "AlgorithmSpecification": {
            "TrainingImage": "683313688378.dkr.ecr.eu-central-1.amazonaws.com/sagemaker-xgboost:1.7-1",
            "TrainingInputMode": "File"
          },
          "RoleArn": "arn:aws:iam::ACCOUNT:role/SageMakerRole",
          "InputDataConfig": [...],
          "OutputDataConfig": {...},
          "ResourceConfig": {
            "InstanceType": "ml.m5.4xlarge",
            "InstanceCount": 1,
            "VolumeSizeInGB": 50
          },
          "StoppingCondition": {
            "MaxRuntimeInSeconds": 7200
          }
        },
        "Next": "EvaluateModel"
      },
      "EvaluateModel": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
          "FunctionName": "evaluate-model-metrics",
          "Payload": {
            "model_uri.$": "$.ModelArtifacts.S3ModelArtifacts"
          }
        },
        "Next": "CheckMetrics"
      },
      "CheckMetrics": {
        "Type": "Choice",
        "Choices": [{
          "Variable": "$.Payload.mape",
          "NumericLessThan": 15,
          "Next": "DeployModel"
        }],
        "Default": "NotifyPoorPerformance"
      },
      "DeployModel": {
        "Type": "Task",
        "Resource": "arn:aws:states:::sagemaker:createEndpoint",
        "Parameters": {...},
        "Next": "NotifySuccess"
      },
      "NotifySuccess": {
        "Type": "Task",
        "Resource": "arn:aws:states:::sns:publish",
        "Parameters": {
          "TopicArn": "arn:aws:sns:eu-central-1:ACCOUNT:ml-ops-notifications",
          "Message": "Model retraining succeeded"
        },
        "End": true
      },
      "NotifyFailure": {
        "Type": "Task",
        "Resource": "arn:aws:states:::sns:publish",
        "Parameters": {
          "TopicArn": "arn:aws:sns:eu-central-1:ACCOUNT:ml-ops-critical",
          "Message": "Model retraining failed"
        },
        "End": true
      }
    }
  }
  ```
- **Triggers:**
  - **EventBridge schedule:** Weekly retraining (Sundays 2 AM)
  - **EventBridge rule:** Model drift detected → trigger retraining
  - **Kafka event:** `model.drift_detected` → Lambda → Start Step Functions execution
- **Cost:** $25 per 1M state transitions (avg $200/month for all workflows)

**2. Amazon EventBridge Scheduler (Simple Cron Jobs)**
- **Use cases:**
  - Daily cache warmup (Lambda invocation)
  - Hourly data quality checks (Lambda → Athena query)
  - Cleanup jobs (S3 lifecycle policies, DynamoDB TTL)
- **Features:**
  - **Flexible schedules:** Cron expressions, rate expressions, one-time executions
  - **Target integrations:** Lambda, Step Functions, SageMaker, ECS tasks, API Gateway
  - **Retry policies:** Configurable retry attempts with exponential backoff
- **Example:** Daily demand forecast batch inference
  ```json
  {
    "Name": "daily-demand-forecast-batch",
    "ScheduleExpression": "cron(0 6 * * ? *)",
    "FlexibleTimeWindow": {"Mode": "OFF"},
    "Target": {
      "Arn": "arn:aws:states:eu-central-1:ACCOUNT:stateMachine:demand-forecast-batch",
      "RoleArn": "arn:aws:iam::ACCOUNT:role/EventBridgeSchedulerRole",
      "RetryPolicy": {
        "MaximumRetryAttempts": 2,
        "MaximumEventAge": 3600
      }
    }
  }
  ```
- **Cost:** $1 per 1M invocations (typically <$10/month)

**3. AWS MWAA (Managed Airflow) - Optional for Complex DAGs**
- **Use cases:** Complex data pipelines with 20+ interdependent tasks
  - Example: End-to-end data lakehouse pipeline (Bronze → Silver → Gold with 30+ tables)
- **When to use:** If Step Functions becomes unwieldy (50+ states), use MWAA for better DAG visualization
- **Deployment:** Small environment (1 worker, 1 scheduler), $0.49/hour = ~$350/month
- **Trade-off:** Higher cost but richer UI and Python-based DAG definitions
- **Decision:** Start with Step Functions, migrate specific workflows to MWAA if needed

**Monitoring & Alerting:**
- **CloudWatch Logs:** All workflow executions logged with X-Ray trace IDs
- **CloudWatch Alarms:** Alert on workflow failures, timeout exceeded, high execution duration
- **SNS Topics:**
  - `ml-ops-notifications` (Slack integration for routine updates)
  - `ml-ops-critical` (PagerDuty integration for failures requiring immediate attention)
- **CloudWatch Dashboard:** Visual overview of all scheduled jobs
  - Success/failure rates (last 7 days)
  - Average execution duration per workflow
  - Cost per workflow (tracked via CloudWatch Logs Insights)

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
