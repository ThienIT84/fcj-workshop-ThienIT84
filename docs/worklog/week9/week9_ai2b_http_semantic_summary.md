# Week 9 AI2B HTTP Semantic Summary

## Evidence

| Artifact | Purpose |
| --- | --- |
| `Dataset/tools/ai2b_modeling/artifacts/http_dataset_v1_4_8j/ai2b_http_dataset_v1_4_8j_summary.json` | HTTP semantic dataset QA |
| `Dataset/tools/ai2b_modeling/docs/ai2b_v1_4_8j_overlap_cleanup_baseline_report.md` | Baseline, candidate ranking, shortcut sanity |
| `Dataset/tools/ai2b_modeling/docs/ai2b_v1_4_9_freeze_review_report.md` | Candidate freeze evidence |
| `Dataset/tools/ai2b_modeling/docs/ai2b_v1_4_9_holdout_incomplete_evaluation_summary.md` | Holdout boundary and protocol guard explanation |

## Dataset QA

| Item | Value |
| --- | --- |
| Dataset rows | 11,829 |
| Labels | `NONE`, `SQLI`, `XSS` |
| Duplicate request keys | 0 |
| Diagnostic overlap audit | Passed |

## Baseline And Shortcut Checks

| Check | Result |
| --- | --- |
| Selected candidate | `tfidf_char_lex_v4_logreg_uri_query_norm_no_ident_no_class_weight` |
| Baseline conclusion | `AI2B_V1_4_8J_BASELINE_PASS_PENDING_STRESS` |
| Endpoint-only baseline | Passed |
| Parameter-only baseline | Passed |
| Label-shuffle sanity | Passed |
| Metric stability | Passed |

## Freeze And Holdout Boundary

| Item | Value |
| --- | --- |
| Freeze conclusion | `AI2B_CANDIDATE_FROZEN_FOR_HOLDOUT_121_124` |
| Threshold | `0.50` |
| Source lock | Passed |
| Canary | Passed |
| Candidate consistency | Passed |
| Holdout scoring | Not started because protocol guards blocked attempts before prediction |

## Boundary

AI2B is suitable as a release-candidate/demo model with clear validation discipline. Do not report final holdout recall or false-positive rate because the strict attempts were not scored.
