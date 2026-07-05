# Week 8 Screenshot Checklist

Week 8 is docs-first. Screenshots are optional and should only support claims that are difficult to read from the tables.

## Recommended Screenshots

| ID | Screenshot | Source |
| --- | --- | --- |
| `w8-e01-ai2a-dataset-contract.png` | AI2A dataset contract showing AI1/AI2A/AI2B boundary and mapping | `Dataset/tools/attack_profiles/final_attack_dataset/docs/ai2a_classifier_dataset_contract.md` |
| `w8-e02-attack-profile-coverage.png` | Attack profile coverage matrix or final attack dataset source rows | `docs/worklog/week8/week8_attack_profile_coverage_matrix.csv` |
| `w8-e03-a12-sqli-http-log.png` | SQLi Zeek HTTP JSONL rows showing URI/query evidence | `Dataset/tools/attack_profiles/attack_012_web_semantic/http_logs/attack_a12_run_001_sqli_search_login_calibration_http.log` |
| `w8-e04-a12-xss-http-log.png` | XSS Zeek HTTP JSONL rows showing URI/query evidence | `Dataset/tools/attack_profiles/attack_012_web_semantic/http_logs/attack_a12_run_004_xss_search_comment_calibration_http.log` |
| `w8-e05-ai1-handoff.png` | AI1 handoff or model card showing feature contract context | `Dataset/tools/ai1_modeling/AI1_HANDOFF.md` |

## Not Required

- Do not screenshot every A01-A10 run.
- Do not screenshot large CSV files directly.
- Do not screenshot credentials, passwords, `.env` files, SSH secrets, API keys, or private paths.
- Do not show public absolute machine paths in report screenshots.

## Caption Style

Use captions that explain the claim:

```text
Figure X - Week 8 evidence matrix mapping normal profiles, attack profiles, and AI-specific datasets to AI1, AI2A, and AI2B responsibilities.
```
