# Week 9 AI1 Readiness Summary

## Evidence

| Artifact | Purpose |
| --- | --- |
| `Dataset/tools/ai1_modeling/AI1_HANDOFF.md` | Defines AI1 purpose, input path, output, threshold source, and known limitation |
| `Dataset/tools/ai1_modeling/artifacts/release_candidate_v1/latest/model_card.md` | Documents Isolation Forest model card and 30-feature contract |
| `Dataset/tools/ai1_modeling/artifacts/release_candidate_v1/latest/thresholds_frozen.json` | Stores frozen decision threshold |
| `Dataset/tools/ai1_modeling/artifacts/release_candidate_v1/latest/api_smoke_ai1_response.json` | Stores API smoke evidence |

## Readiness

| Item | Value |
| --- | --- |
| Model | Isolation Forest |
| Purpose | Network anomaly detection |
| Input | 30-feature vector from Zeek flow evidence |
| Output | `NORMAL` or `ANOMALY` |
| Threshold | `0.398066` |

## Boundary

AI1 should be described as having a documented artifact contract and smoke evidence. Do not claim live raw `conn.log` replay support until a raw-flow feature extraction layer is implemented.
