# Week 8 AI Dataset Workspace Index

## Purpose

Week 8 documents the dataset workspace behind the Hybrid IDS/AI SOC project. The focus is coverage and organization, not final model performance.

The workspace is organized around three AI responsibilities:

| Component | Dataset role | Primary evidence |
| --- | --- | --- |
| AI1 | Flow anomaly detection | AI1 handoff, model card, feature manifest, threshold files |
| AI2A | Flow-level supervised attack classification | Normal P01-P12, attack profiles A01-A10, final AI2A classifier dataset |
| AI2B | HTTP semantic classification | A12 SQLi/XSS HTTP logs and AI2B HTTP datasets |

## Source Areas

| Area | Logical path | Week 8 use |
| --- | --- | --- |
| Normal baseline | `Dataset/tools/final_dataset/` | Normal P01-P12 baseline metrics |
| Attack profiles | `Dataset/tools/attack_profiles/` | AI2A attack coverage |
| AI2A final dataset | `Dataset/tools/attack_profiles/final_attack_dataset/` | AI2A classifier dataset contract and QA anchor |
| AI1 artifacts | `Dataset/tools/ai1_modeling/` | Anomaly-detection feature and threshold context |
| AI2A modeling | `Dataset/tools/ai2a_modeling/` | Release-candidate context for Week 9 |
| AI2B modeling | `Dataset/tools/ai2b_modeling/` | HTTP semantic dataset and holdout context |

## Key Metrics

| Dataset | Rows | Notes |
| --- | ---: | --- |
| Normal P01-P12 | 197,797 | Clean normal baseline with duplicate UID `0` and missing cells `0` |
| AI2A classifier dataset | 207,881 | 77 columns, 281 runs, 53 model features, duplicate UID `0` |
| AI2B HTTP dataset V1.4.8J | 11,829 | HTTP semantic dataset for `NONE`, `SQLI`, and `XSS`; duplicate request keys `0` |

## Week 8 Boundary

Week 8 can claim dataset workspace coverage and evidence organization. It must not claim production-readiness, blind-holdout success, or model-training completion.
