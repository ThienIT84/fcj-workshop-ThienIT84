# Week 8 AI Pipeline Mapping

## Architecture

| Component | Input | Output | Dataset owner |
| --- | --- | --- | --- |
| AI1 | 30-feature vector from Zeek `conn.log` rows | `NORMAL` or `ANOMALY` | `Dataset/tools/ai1_modeling/` |
| AI2A | Flow-level Zeek `conn.log` features | Supervised attack class | `Dataset/tools/attack_profiles/final_attack_dataset/` |
| AI2B | HTTP request evidence from Zeek `http.log` | `NONE`, `SQLI`, or `XSS` | `Dataset/tools/ai2b_modeling/` |
| Fusion | AI outputs and rule evidence | Final risk score | Later backend/system layer |

## AI1

AI1 is an Isolation Forest anomaly detector. It expects a fixed 30-feature vector derived from Zeek flow evidence. Week 8 records AI1 as dataset architecture context only.

Known limitation:

```text
Backend live/replay mode still needs a raw conn.log row -> AI1 feature extraction step.
```

## AI2A

AI2A is the supervised flow-level classifier. It uses the final classifier dataset with:

| Metric | Value |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Model features | 53 |
| Run count | 281 |
| Duplicate UID | 0 |

AI2A labels are derived from the attack-profile mapping. Metadata such as `uid`, `run_id`, IP addresses, timestamps, provenance label, and event metadata must not be used as model features.

## AI2B

AI2B is the HTTP semantic detector. It uses HTTP request-line and URI/query evidence to classify:

```text
NONE
SQLI
XSS
```

A12 is the main Week 8 evidence source for AI2B. Final holdout and freeze-readiness discussion belongs to Week 9.
