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
Adopt a **hybrid orchestration model** using **Airflow** for batch pipelines and **Temporal** for real-time, event-driven AI workflows.

Architecture:
- Airflow: Daily ETL, weekly retraining, data validation, and reporting
- Temporal: Event-triggered workflows, retraining triggers, real-time inference orchestration
- Both integrated with common monitoring and alerting stack

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
