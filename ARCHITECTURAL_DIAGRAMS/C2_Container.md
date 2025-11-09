# C2: Container Diagram (Updated)

This diagram zooms into the MobilityCorp platform, showing the high-level containers (applications, microservices, data stores) that constitute the system. This updated version includes Edge Computing, Orchestrator Layer, Data Lakehouse, and all missing components from the review.

[![](https://img.plantuml.biz/plantuml/svg/ZLZ9SXit4BtpAn1TJ5KaJn8vjrJA53VJ8YrQfAFuf09dG18X1fW086KwbMk-85-OBqatC9YLjEs31alZeJTqF_griLJRVIewtt4PYtt2o2_zLr7_rLDVIKkvPFhNJiToAnWfLiY0quscARacCxNYWjjZN-cCp0MrQwNJJkS5cJDjb3I4Mc9Lrl7JRhmtLgLCNv6BVZwyWB53OOHc6TaRMBeCyg3TUWq22_oYv7t6DBKSaIsZQIb9avHB4EZX5oKNHsDPIjmwY85ACr2UPBcEPANsCg7wsF62JxVvl1iaWbvDwoukoLyT0lzUa946bp2PeEQ2nrunjrVugfiwsqeZUrc6wZqm6blo3euSs4N57TTh6dEv8MdXaW0CHjIHLMOIQhPrhmnWPQMeJefBBeB_Y2h5EEYjuIhMGdKEovIsL0HtahcRrX4hxkN6UjmMREwdtdn2njIoPtfitfRnpJFUKqeW-fsIcwZtk22DzVlUtcv_Yju-Bgy8NCc8u2ctuAyh8XHDo8eAAft_kZ_-S4Sq-pCoBBwiQmFHP0JIxC1ZrepMIgcTWxkuzIEoy89u-pkLi9T_61o-eJ8HpC0D29Gx5TQ8rJmpBQTcz9WoQU7Wt8-gi6_fWU9thXKeP00r5s_ZuEi0oG_mEQEOsqgff1lcxa6TA3Yj1RKxnkZeZ_rJgamIE0k9JmvCy_Mv13omBOzTRl_cH_34miM8CbRuToGOinMLMZ2M2TYm-WZ2op2kQkLnmXq9jHGIRSFCIQMaieN6pZNbz8pBPdjX-NMyfL8oGQhdsm608ecUEQ3Hap0cGxc1NHV3ezqgwL-AK75u5RcWgIVYlOwtpBqTfPk9IBcgxE8BcKJJwQnsfdxXBKiWWNRalt_-9L0GP6z2rfGBTN05jPMF3VYGbqtya14BJ6pcm7l9aSdps50NilLUnkuGoyW8njwPQCNZ0Z-FIk_MGZsJKeiobg5eJABPzDmxZPKyK8vr4mRG0TnvAk1GwPHeIkMclwMyvfxloKDlt58wOP2W2OWDt82Ce4R4r61fSTP3xixepdUhtyTG6uoDfelbhErPpMDVG0P7ILCUaxbVmPDJcguI6jNGS9j1QRPGoJwVVcSfnRegeR8nTu0bF7PvDoitMbhsf4gfE9A4MHQVX5qpQlTGm23IY3loKxB0UGlkNWeu1DwxNw_TA9nkQmilIsYsmQvmT7qFvkGXNsW1-ssC8UY8VgebmG0U-ugQPlm7goxTQwKf6ke6qAECtMYsU3zrhs_WycofgJGqpfC3AqTJNK1AC50gMOMOgS6jhn0vLYWyG4OiE2gFy0aW38o10qE1cTEdaDfWDwov1e_JCk0nJgyWlOqvLMW03HA2cPYw-heCOwyQJg_XPN3ceCkYsNtrykOsIkZnyWGWj7zf68KNvoc3NBY9Dp66g1EF3k-3xAHw5YpPC8BUhyTaUC120S440eOT4jyBkhRPOEXwv-AJEKVSuTG5V767doPAzuQij8AaC2tTk-O6i-R6PGEPGkPi5NH2bqetqO29VDKzSw1adrcqu09IAXehaH11Zu3Q0js8FRfs31yoN4xpwaHmZcDtbyNyGT9Nj8E_aKaEFm7DmOmoZWMPm25XlSiLjlc9Mf8yRro8VWOsidQDy9dRBS7jiMPC0eqs25i1O9zicoxa7CYKfVZAr-7wsPHcc80pceLUPtm3LHlNdN4N_gxK9q-9FNa6XaxMWSfg9honMo0o7UXYJmFrN6Nk1Q45gk6vq78vNm1U8PkpzfAVdfNCMNkTSpQbEnMY3PfMoMG159IgyabuB-BqjOLctoPUK--BHperhelQ5lqBq0BRBKdcWzfRR-p3SB6CDW_plhyuY9QPy5NHd1j-LQvaRLyLBRfiNT8RcYUOv_o-ndeh2_D21PZlLu8RwFE47HojHUBfoe6_hJ3oF2q19h-7PNUqecVhGFtn14BTY6f9NAlKiyel6zTKn5esLOt1y4z2oDkE2UviSJiaTcSi36TgnHihD4Y7-byrBSXNcbgL8f4jntxls_3WpkliIbq6DGKRkqF9hVHrrvLe-BtjMFWTY1Xko8XhgCAkDU01bCmBQithYupgNIxZzntce-PvFbMgj_-HFWtCWZXYuaxMcKh1rKQ1WXJ6j8LBFdPQki5M2cn7IqpBJTWDlv3n1Ic1tyc9YKepaKhu2_qp_GR18aaheMn95tOKTWOIKl0Dnq20XIghOYNQ6Ltb6KqzshskRlmjRu3vxbSGagYlKdIe_vtrVkbAnxJtyVx3ykbnidppD1sEX-y6NLXz3UKT_n_gVm00)](https://editor.plantuml.com/uml/ZLZ9SXit4BtpAn1TJ5KaJn8vjrJA53VJ8YrQfAFuf09dG18X1fW086KwbMk-85-OBqatC9YLjEs31alZeJTqF_griLJRVIewtt4PYtt2o2_zLr7_rLDVIKkvPFhNJiToAnWfLiY0quscARacCxNYWjjZN-cCp0MrQwNJJkS5cJDjb3I4Mc9Lrl7JRhmtLgLCNv6BVZwyWB53OOHc6TaRMBeCyg3TUWq22_oYv7t6DBKSaIsZQIb9avHB4EZX5oKNHsDPIjmwY85ACr2UPBcEPANsCg7wsF62JxVvl1iaWbvDwoukoLyT0lzUa946bp2PeEQ2nrunjrVugfiwsqeZUrc6wZqm6blo3euSs4N57TTh6dEv8MdXaW0CHjIHLMOIQhPrhmnWPQMeJefBBeB_Y2h5EEYjuIhMGdKEovIsL0HtahcRrX4hxkN6UjmMREwdtdn2njIoPtfitfRnpJFUKqeW-fsIcwZtk22DzVlUtcv_Yju-Bgy8NCc8u2ctuAyh8XHDo8eAAft_kZ_-S4Sq-pCoBBwiQmFHP0JIxC1ZrepMIgcTWxkuzIEoy89u-pkLi9T_61o-eJ8HpC0D29Gx5TQ8rJmpBQTcz9WoQU7Wt8-gi6_fWU9thXKeP00r5s_ZuEi0oG_mEQEOsqgff1lcxa6TA3Yj1RKxnkZeZ_rJgamIE0k9JmvCy_Mv13omBOzTRl_cH_34miM8CbRuToGOinMLMZ2M2TYm-WZ2op2kQkLnmXq9jHGIRSFCIQMaieN6pZNbz8pBPdjX-NMyfL8oGQhdsm608ecUEQ3Hap0cGxc1NHV3ezqgwL-AK75u5RcWgIVYlOwtpBqTfPk9IBcgxE8BcKJJwQnsfdxXBKiWWNRalt_-9L0GP6z2rfGBTN05jPMF3VYGbqtya14BJ6pcm7l9aSdps50NilLUnkuGoyW8njwPQCNZ0Z-FIk_MGZsJKeiobg5eJABPzDmxZPKyK8vr4mRG0TnvAk1GwPHeIkMclwMyvfxloKDlt58wOP2W2OWDt82Ce4R4r61fSTP3xixepdUhtyTG6uoDfelbhErPpMDVG0P7ILCUaxbVmPDJcguI6jNGS9j1QRPGoJwVVcSfnRegeR8nTu0bF7PvDoitMbhsf4gfE9A4MHQVX5qpQlTGm23IY3loKxB0UGlkNWeu1DwxNw_TA9nkQmilIsYsmQvmT7qFvkGXNsW1-ssC8UY8VgebmG0U-ugQPlm7goxTQwKf6ke6qAECtMYsU3zrhs_WycofgJGqpfC3AqTJNK1AC50gMOMOgS6jhn0vLYWyG4OiE2gFy0aW38o10qE1cTEdaDfWDwov1e_JCk0nJgyWlOqvLMW03HA2cPYw-heCOwyQJg_XPN3ceCkYsNtrykOsIkZnyWGWj7zf68KNvoc3NBY9Dp66g1EF3k-3xAHw5YpPC8BUhyTaUC120S440eOT4jyBkhRPOEXwv-AJEKVSuTG5V767doPAzuQij8AaC2tTk-O6i-R6PGEPGkPi5NH2bqetqO29VDKzSw1adrcqu09IAXehaH11Zu3Q0js8FRfs31yoN4xpwaHmZcDtbyNyGT9Nj8E_aKaEFm7DmOmoZWMPm25XlSiLjlc9Mf8yRro8VWOsidQDy9dRBS7jiMPC0eqs25i1O9zicoxa7CYKfVZAr-7wsPHcc80pceLUPtm3LHlNdN4N_gxK9q-9FNa6XaxMWSfg9honMo0o7UXYJmFrN6Nk1Q45gk6vq78vNm1U8PkpzfAVdfNCMNkTSpQbEnMY3PfMoMG159IgyabuB-BqjOLctoPUK--BHperhelQ5lqBq0BRBKdcWzfRR-p3SB6CDW_plhyuY9QPy5NHd1j-LQvaRLyLBRfiNT8RcYUOv_o-ndeh2_D21PZlLu8RwFE47HojHUBfoe6_hJ3oF2q19h-7PNUqecVhGFtn14BTY6f9NAlKiyel6zTKn5esLOt1y4z2oDkE2UviSJiaTcSi36TgnHihD4Y7-byrBSXNcbgL8f4jntxls_3WpkliIbq6DGKRkqF9hVHrrvLe-BtjMFWTY1Xko8XhgCAkDU01bCmBQithYupgNIxZzntce-PvFbMgj_-HFWtCWZXYuaxMcKh1rKQ1WXJ6j8LBFdPQki5M2cn7IqpBJTWDlv3n1Ic1tyc9YKepaKhu2_qp_GR18aaheMn95tOKTWOIKl0Dnq20XIghOYNQ6Ltb6KqzshskRlmjRu3vxbSGagYlKdIe_vtrVkbAnxJtyVx3ykbnidppD1sEX-y6NLXz3UKT_n_gVm00)

###  Components:

1. **Admin Portal** (Frontend)
2. **KYC Service** (split from User Service)
3. **Incentive Engine** (relocation incentives calculation)
4. **Incident Management Service** (collision alerts, dispatch)
5. **Orchestrator Layer** (AI/LLM, Payment, Map, Weather orchestrators)
6. **Feature Store** (SageMaker - online/offline)
7. **Model Registry** (SageMaker - versioning, approval)
8. **MLOps Pipelines** (SageMaker Pipelines)
9. **OpenSearch Serverless** (Vector DB for RAG)
10. **ElastiCache Redis** (caching layer)
11. **Data Lakehouse** (Bronze/Silver/Gold layers)
12. **Glue Data Catalog, Athena, Glue ETL Jobs**
13. **EventBridge** (scheduled events)
14. **Edge Compute Layer** (IoT Greengrass + Edge ML)
15. **SageMaker Model Monitor** (drift detection)
16. **Step Functions** (workflow orchestration)
17. **Multi-Region Infrastructure** (Route 53, Global DB, Global Tables, S3 CRR)

### 📊 Scale & Performance:

- **API Gateway:** 10K requests/second
- **Kafka:** 50 partitions/topic, 3 brokers
- **Telemetry:** 4.3 billion events/day
- **Edge Devices:** 50K IoT Greengrass instances
- **IoT Core:** 130B MQTT messages/month
- **Multi-Region:** <30s failover RTO

### 💰 Cost Estimate:

- **Edge Hardware:** $68K/month (amortized)
- **MSK (Kafka):** $5K/month
- **SageMaker MLOps:** $28K/month
- **Data Lakehouse:** $2.4K/month
- **Multi-Region Overhead:** $15-20K/month
- **Total:** ~$132K/month

## Related Documents:

- [C1: System Context](C1_System_Context.md)
- [C3: AI/ML Component Diagram](C3_Component_AIML.md)
- [ADR-05: Orchestrator Pattern](./ADR/ADR_05_Orchestrator.md)
- [ADR-15: MLOps Pipeline](./ADR/ADR_15_MLOps_Pipeline.md)
- [ADR-16: Data Lakehouse Strategy](./ADR/ADR_16_Data_Lakehouse_Strategy.md)
- [ADR-18: Edge vs Cloud AI Strategy](./ADR/ADR_18_Edge_Cloud_AI_Strategy.md)

---
