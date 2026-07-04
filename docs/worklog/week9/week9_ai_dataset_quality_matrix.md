# Week 9 AI Dataset Quality Matrix

## Purpose

Week 9 records how far AI1, AI2A, and AI2B are ready for demo and integration after the Week 8 dataset workspace review.

| Component | Evidence | Status | Boundary |
| --- | --- | --- | --- |
| AI1 | Handoff, model card, feature manifest, thresholds, smoke response | Artifact contract ready for integration planning | Raw `conn.log` feature extraction still needs implementation |
| AI2A | Classifier dataset report, QA report, release candidate report | Dataset QA and release-candidate evidence available | Rare-class performance must remain visible |
| AI2B | HTTP dataset summary, baseline report, freeze review, incomplete holdout summary | Candidate frozen with strict validation boundary | Holdout attempts were not scored due to protocol guards |

## Main Metrics

| Area | Metric | Value |
| --- | --- | ---: |
| AI1 | Required features | 30 |
| AI1 | Threshold | 0.398066 |
| AI2A | Rows | 207,881 |
| AI2A | Columns | 77 |
| AI2A | Run count | 281 |
| AI2A | Model features | 53 |
| AI2A | Duplicate UID | 0 |
| AI2B | HTTP dataset rows | 11,829 |
| AI2B | Duplicate request keys | 0 |

## Interpretation

AI1, AI2A, and AI2B each have useful evidence, but they should not be collapsed into one claim. AI1 is an anomaly artifact contract, AI2A is the flow-level classifier track, and AI2B is the HTTP semantic track.
