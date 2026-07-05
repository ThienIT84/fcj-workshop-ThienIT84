---
title: "Week 9 - AI Dataset QA, Model Readiness & Validation Boundaries"
date: 2026-06-13
weight: 9
chapter: false
draft: true
pre: " <b> 1.9. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | AI dataset QA and model-readiness review |
| Main objective | Validate how far AI1, AI2A, and AI2B are ready for demo/integration |
| AI1 status | Anomaly-detection artifact contract reviewed |
| AI2A status | Classifier dataset QA and release-candidate evidence reviewed |
| AI2B status | HTTP semantic dataset, freeze review, and holdout boundary reviewed |
| Main output | QA matrix, readiness summary, and validation boundary |
| Week status | Implemented as docs-first draft; screenshots and exact time log require owner confirmation |

## Evidence Source Note

Week 9 uses the dataset and modeling evidence prepared in the project dataset workspace. The public worklog references logical project paths only:

```text
Dataset/tools/ai1_modeling/
Dataset/tools/attack_profiles/final_attack_dataset/
Dataset/tools/ai2a_modeling/
Dataset/tools/ai2b_modeling/
```

Internal Week 9 notes are stored under:

```text
docs/worklog/week9/
```

No raw model file, `.joblib` file, CSV, JSON report, local machine path, or private environment file is published on this Hugo page.

## Objective

The objective of Week 9 was to evaluate the AI dataset and model-readiness evidence after the dataset workspace had been organized in Week 8.

Week 9 answers four questions:

- Does AI1 have a clear anomaly-detection artifact contract?
- Does AI2A have a validated classifier dataset and QA report?
- Does AI2A have release-candidate evidence that can be explained honestly?
- Does AI2B have HTTP semantic QA evidence and a clear holdout boundary?

## Project Context

Week 8 mapped the dataset workspace and profile coverage. Week 9 moves one level deeper into quality and readiness.

The goal is not to claim that every model is ready for long-term operation. The goal is to document which parts are ready for demonstration or integration, which parts are release candidates, and which parts still need future validation.

The review path was:

```text
AI1 artifact contract
-> AI2A classifier dataset QA
-> AI2A release-candidate report
-> AI2B HTTP dataset and shortcut checks
-> AI2B freeze review and holdout boundary
-> final readiness matrix
```

## Daily Worklog

| Activity | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| AI1 readiness review | 13/06/2026 | Estimated 3.5 hours | Reviewed AI1 handoff, model card, feature list, threshold policy, smoke artifacts, and known limitation | AI1 has a documented anomaly-detection artifact contract | Backend still needs raw `conn.log` to AI1 feature extraction for live/replay use | Review AI2A dataset QA |
| AI2A dataset QA review | 14/06/2026 | Estimated 5 hours | Reviewed AI2A classifier dataset report, QA report, class counts, source rows, run count, model features, and excluded metadata | AI2A dataset has 207,881 rows, 77 columns, 281 runs, 53 model features, and duplicate UID 0 | Dataset quality can be explained, but rare-class limitations must remain visible | Review AI2A release candidate |
| AI2A release-candidate review | 15/06/2026 | Estimated 5 hours | Reviewed selected candidate, validation metrics, threshold freeze, release gate checks, per-class metrics, and no-collapse watch table | AI2A release candidate was accepted with validation macro F1 0.835883 and weighted F1 0.985125 | Holdout and rare-class drops require careful interpretation | Review AI2B HTTP semantic evidence |
| AI2B dataset and baseline review | 16/06/2026 | Estimated 5 hours | Reviewed AI2B HTTP semantic dataset summary, baseline report, shortcut sanity, endpoint-only baseline, parameter-only baseline, and label-shuffle sanity | AI2B HTTP semantic dataset and baseline evidence were available for `NONE`, `SQLI`, and `XSS` | Strong validation metrics must be presented with shortcut and holdout discipline | Review AI2B freeze/holdout boundary |
| AI2B freeze and holdout boundary | 17/06/2026 | Estimated 4.5 hours | Reviewed V1.4.9 freeze review and incomplete holdout summary | AI2B candidate was frozen, but holdout attempts were not scored because protocol guards found contamination or inventory mismatch before prediction | Present this as validation discipline, not as a model prediction failure | Consolidate readiness matrix |
| Evidence consolidation | 18/06/2026 | Estimated 3.5 hours | Built Week 9 internal QA summaries, screenshot checklist, and validation boundary notes | Week 9 was ready for a docs-first draft page | Public screenshots can be added later if needed | Move Week 10 toward API/backend/fusion design |

## AI Dataset QA Overview

| Component | Readiness level | Evidence reviewed | Main boundary |
| --- | --- | --- | --- |
| AI1 | Artifact contract ready for integration planning | Handoff, model card, feature manifest, threshold, smoke response | Live/replay extraction from raw `conn.log` still needs implementation |
| AI2A | Dataset QA and release-candidate evidence available | Classifier dataset report, QA report, release-candidate report | Rare-class performance and holdout interpretation must remain visible |
| AI2B | HTTP semantic candidate frozen with strict holdout boundary | HTTP dataset summary, baseline report, freeze review, incomplete holdout summary | Holdout attempts were blocked before scoring by protocol guards |

## AI1 Anomaly Model Readiness

AI1 is the network anomaly-detection component. The reviewed evidence shows that AI1 has an artifact contract that can be integrated by the backend when feature extraction is available.

| Item | Evidence |
| --- | --- |
| Model type | Isolation Forest |
| Purpose | Network anomaly detection from Zeek `conn.log` flow features |
| Required features | 30-feature vector |
| Score direction | Higher confidence means more anomalous |
| Threshold | `0.398066` |
| Decision output | `NORMAL` or `ANOMALY` |
| Smoke evidence | API smoke response exists in the artifact folder |

Important limitation:

```text
AI1 does not consume raw conn.log rows directly in the current backend path.
A feature extraction step is still required before live/replay traffic can be
sent to the AI1 adapter.
```

## AI2A Classifier Dataset QA

AI2A is the supervised flow-level classifier. It consumes Zeek `conn.log`-derived features and predicts a network attack class.

The classifier dataset QA evidence reports:

| Metric | Value |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Run count | 281 |
| Target column | `ai2a_label` |
| Model feature count | 53 |
| Duplicate UID | 0 |
| Normal rows | 199,326 |
| Attack rows | 8,555 |

Class coverage includes:

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

The QA result is useful because it separates model features from metadata. Fields such as `uid`, `run_id`, timestamps, IP identity, event metadata, provenance labels, and target labels are excluded from model features.

## AI2A Release Candidate Validation

The AI2A release report selected:

```text
rf_v2_1_full_safe_plus_ssh_minimal
```

Checkpoint result:

```text
RELEASE_CANDIDATE_ACCEPTED
```

Validation metrics for the selected candidate:

| Metric | Value |
| --- | ---: |
| Validation macro F1 | 0.835883 |
| Validation weighted F1 | 0.985125 |
| Validation normal FPR | 0.003624 |
| A02 recall | 0.985455 |
| A03 recall | 0.861905 |
| A07 recall | 0.888889 |
| A08 recall | 0.636364 |
| A09 recall | 0.636364 |

The release-candidate evidence is strong enough for demo/model-readiness discussion. It is still important to show class-level behavior because some rare classes have lower or more fragile recall than the dominant normal class.

## AI2B HTTP Semantic Dataset QA

AI2B is the HTTP semantic detector for:

```text
NONE
SQLI
XSS
```

The HTTP dataset V1.4.8J evidence includes:

| Item | Value |
| --- | --- |
| Dataset rows | 11,829 |
| Duplicate request keys | 0 |
| Diagnostic overlap audit | Passed |
| Main source | A12 Web Semantic HTTP evidence |

The baseline report selected:

```text
tfidf_char_lex_v4_logreg_uri_query_norm_no_ident_no_class_weight
```

The shortcut sanity checks were important because an HTTP semantic detector can accidentally learn endpoint or parameter shortcuts instead of payload semantics.

| Check | Result |
| --- | --- |
| Endpoint-only baseline | Passed |
| Parameter-only baseline | Passed |
| Label-shuffle sanity | Passed |
| Metric stability | Passed |

## AI2B Freeze Review and Holdout Boundary

The V1.4.9 freeze review concluded:

```text
AI2B_CANDIDATE_FROZEN_FOR_HOLDOUT_121_124
```

Candidate:

```text
tfidf_char_lex_v4_logreg_uri_query_norm_no_ident_no_class_weight
```

Freeze evidence:

| Item | Result |
| --- | --- |
| Source lock | Passed |
| Canary | Passed |
| Candidate consistency | Passed |
| Threshold | `0.50` |

The holdout review is intentionally reported as incomplete. The strict protocol blocked scoring before prediction because the holdout attempts had contamination or inventory mismatch issues:

| Attempt | Result | Scored | Reason |
| --- | --- | --- | --- |
| `121-124` | Contamination guard failed | No | Exact overlap with release dataset |
| `125-128` | Inventory guard failed | No | Actual rows were not in the frozen planned inventory |
| `129-132` | Inventory guard failed | No | Actual multiset differed from the frozen manifest |

This is a validation pipeline limitation, not evidence that the model predicted incorrectly. Because scoring did not start, Week 9 does not report final holdout recall or false-positive rate for AI2B.

## Validation Matrix

| Area | Evidence | Status | Week 9 interpretation |
| --- | --- | --- | --- |
| AI1 artifact contract | Handoff, model card, threshold, smoke response | Ready for integration planning | Feature extraction remains required |
| AI2A dataset QA | Classifier report and QA report | Passed core QA checks | Good basis for model-readiness discussion |
| AI2A release candidate | Release report | Accepted | Strong demo candidate with class-level caveats |
| AI2B HTTP dataset | Dataset summary and baseline report | Baseline and shortcut checks available | Semantic detector evidence is strong but still bounded |
| AI2B freeze review | V1.4.9 freeze review | Candidate frozen | Holdout evaluation remains incomplete |
| AI2B holdout attempts | Incomplete holdout summary | Not scored | Guardrails blocked scoring before prediction |

## Leakage, Split and Safety Decisions

The key safety decision is to avoid training on identity or provenance shortcuts.

For AI2A, excluded fields include:

```text
uid
run_id
ts
flow_end_ts
src_ip
dst_ip
src_port
event metadata
source_profile
attack metadata
label
ai2a_label
```

For AI2B, the reports emphasize validation discipline:

- threshold selection uses validation only,
- holdout adjustment is disabled,
- endpoint-only and parameter-only shortcut checks are reviewed,
- contaminated or inventory-mismatched holdout attempts are not scored.

## Issues and Limitations

| Limitation | Impact | Follow-up |
| --- | --- | --- |
| AI1 raw `conn.log` feature extraction is not yet integrated | Live/replay AI1 use needs an extraction layer | Implement extraction before claiming live AI1 processing |
| AI2A rare classes have smaller support | Some class metrics are more fragile | Keep per-class metrics visible |
| AI2B holdout attempts were not scored | No final AI2B holdout score can be reported | Fix capture/inventory protocol before another one-shot attempt |
| Screenshots are not yet attached | Current Week 9 is docs-first | Add selected redacted screenshots later if needed |

## Lessons Learned

Week 9 showed that model readiness is not just a high validation score. A useful AI security report must also explain feature contracts, leakage boundaries, split discipline, class imbalance, shortcut checks, and holdout status.

The most important lesson was from AI2B: a strict validation protocol can block scoring before prediction. That is uncomfortable, but it is more honest than scoring contaminated or misaligned holdout data.

## Weekly Outcome

Week 9 produced a clear readiness view for AI1, AI2A, and AI2B.

AI1 has an artifact contract and smoke evidence. AI2A has classifier dataset QA and an accepted release candidate. AI2B has HTTP semantic dataset evidence and a frozen candidate, but its strict holdout evaluation remains future validation work.

## Next Week Plan

Week 10 will move from dataset/model evidence into system design and integration:

- define API contracts for AI1, AI2A, AI2B, and fusion,
- describe backend request/response flow,
- document how alert evidence becomes a risk score,
- prepare the path for the AWS MVP migration and SOC demo.
