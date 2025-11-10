# Data Flow Diagram: AI/ML Pipeline

This diagram illustrates the flow of data through the AI/ML pipeline, from initial ingestion to model training and deployment.

```
+-----------------------------------------------------+
|                     INGESTION                       |
|                                                     |
|  [Kafka Events] (A) --> (Stream Processor) (B) --+  |
|                                                  |  |
|  [S3 Data Lake] (C) --> (Batch ETL) (D)      ----+  |
|                                                     |
+-----------------------------------------------------+
                               |
                               v
+-----------------------------------------------------+
|                 FEATURE ENGINEERING                 |
|                                                     |
|                  {Feature Store} (E)                |
|                                                     |
+-----------------------------------------------------+
                               |
                               v
+-----------------------------------------------------+ <----------------+
|                   MODEL TRAINING                    |                  |
|                                                     |                  |
|      [Training Pipeline on Airflow+BEAM] (F)        |                  |
|                         |                           |                  |
|                         v                           |                  |
|                  (SageMaker) (G)                    |                  |
|                                                     |                  |
+-----------------------------------------------------+                  |
                               |                                         |
                               | "Model Approval"                          |
                               v                                         |
+-----------------------------------------------------+                  |
|                  MODEL DEPLOYMENT                   |                  |
|                                                     |                  |
|                (CI/CD Pipeline) (H)                 |                  |
|                   /              \                  |                  |
|                  /                \                 |                  |
|                 v                  v                |                  |
|  (Canary Deployment) (I)   (Shadow Deployment) (J)  |                  |
|                                                     |                  |
+------------------+------------------+----------------+                  |
                   |                  |                                  |
                   v                  v                                  |
+------------------+------------------+----------------+                  |
|                     INFERENCE                       |                  |
|                                                     |                  |
| [Prod Inference Svc] (K) (Shadow Inference Svc) (L) |                  |
|                                                     |                  |
+------------------+------------------+----------------+                  |
                   |                  |                                  |
                   v                  v                                  |
+------------------+------------------+----------------+                  |
|                     MONITORING                      |                  |
|                                                     |                  |
|               {Monitoring Service} (M)              |                  |
|                  /                  \                 |                  |
|                 /                    \ "Drift Detected"                |
|                v                      +----------------------------------+
|    (Grafana Dashboards) (N)                         |
|                                                     |
+-----------------------------------------------------+
```