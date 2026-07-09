# Week 10 Validation Boundary

## Safe Claims

- Backend MVP uses FastAPI.
- `POST /api/events` is the main ingestion endpoint.
- HTTP evidence routes to AI2B.
- Flow evidence routes to AI1 and AI2A.
- Unsupported model/event combinations return `not_applicable`.
- Compatible but unavailable artifacts return `not_available`.
- Fusion produces final alert fields such as `attack_type`, `risk_score`, `severity`, contributors, and excluded models.
- Replay and tailer scripts exist for local Zeek evidence.

## Avoided Claims

- Do not say Week 10 re-created all models from the beginning.
- Do not say the backend is ready for long-term operations.
- Do not say AWS runtime deployment is finished in Week 10.
- Do not say every model is running real mode on AWS.
- Do not say the anomaly adapter can infer complete features from minimal flow evidence.
- Do not say the live SOC workflow is fully demonstrated without runtime evidence.

## Recommended Wording

```text
Week 10 consolidated trained/model-ready artifacts into backend API contracts,
real adapters, replay/tailer scripts, and a rule-based fusion layer.
```
