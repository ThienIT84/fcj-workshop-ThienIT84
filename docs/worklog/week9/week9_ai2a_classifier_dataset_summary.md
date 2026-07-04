# Week 9 AI2A Classifier Dataset Summary

## Evidence

| Artifact | Purpose |
| --- | --- |
| `Dataset/tools/attack_profiles/final_attack_dataset/reports/ai2a_classifier_dataset_v1_report.json` | Dataset shape, source rows, class counts, excluded metadata |
| `Dataset/tools/attack_profiles/final_attack_dataset/reports/ai2a_classifier_dataset_v1_qa.json` | QA status, duplicate UID, run count, model feature count |
| `Dataset/tools/attack_profiles/final_attack_dataset/docs/ai2a_classifier_dataset_contract.md` | Dataset schema, label mapping, leakage policy |
| `Dataset/tools/ai2a_modeling/docs/ai2a_release_candidate_v1_report.md` | Release candidate, validation metrics, threshold freeze, holdout comparison |

## Dataset QA

| Metric | Value |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Run count | 281 |
| Model features | 53 |
| Duplicate UID | 0 |
| Normal rows | 199,326 |
| Attack rows | 8,555 |

## Class Counts

| Class | Rows |
| --- | ---: |
| `normal` | 199,326 |
| `web_initial_access_indicator` | 3,811 |
| `ssh_bruteforce_indicator` | 2,177 |
| `port_scan_or_recon` | 1,035 |
| `dns_anomaly_indicator` | 617 |
| `http_beaconing_indicator` | 498 |
| `collection_staging_indicator` | 212 |
| `controlled_exfiltration` | 70 |
| `suspicious_admin` | 69 |
| `east_west_simulation` | 66 |

## Release Candidate

| Item | Value |
| --- | --- |
| Conclusion | `RELEASE_CANDIDATE_ACCEPTED` |
| Selected candidate | `rf_v2_1_full_safe_plus_ssh_minimal` |
| Validation macro F1 | `0.835883` |
| Validation weighted F1 | `0.985125` |
| Validation normal FPR | `0.003624` |

## Boundary

AI2A has strong demo/model-readiness evidence, but class-level caveats should remain visible because rare classes have smaller support and more fragile recall.
