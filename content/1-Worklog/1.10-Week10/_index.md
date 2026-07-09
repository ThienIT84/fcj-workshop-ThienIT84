---
title: "Week 10 - AI Model Handoff, Backend API & Fusion Layer"
date: 2026-06-19
weight: 10
chapter: false
draft: true
pre: " <b> 1.10. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | Backend API and multi-model integration |
| Main objective | Consolidate AI artifacts into API contracts, adapters, replay paths, and fusion logic |
| Backend framework | FastAPI |
| Main endpoint | `POST /api/events` |
| AI routing | Flow evidence -> AI1/AI2A; HTTP evidence -> AI2B |
| Fusion output | `attack_type`, `risk_score`, `severity`, contributors, excluded models |
| AWS migration | Deferred to Week 11 |
| Week status | Implemented as docs-first draft; screenshots and exact time log require owner confirmation |

## Evidence Source Note

Week 10 uses backend and integration evidence from logical project paths:

```text
backend/
docs/backend_integration_contract.md
docs/multi_model_fusion_mvp_code_reading_guide.md
```

The public Hugo page does not publish raw code files, model binaries, environment files, local absolute paths, or private credentials.

Internal Week 10 notes are stored under:

```text
docs/worklog/week10/
```

Optional screenshots can be added later under:

```text
static/images/worklog/week10/
```

## Objective

The objective of Week 10 was to move from dataset and model-readiness evidence into backend integration.

Week 8 organized the dataset workspace. Week 9 reviewed QA and model-readiness boundaries. Week 10 focuses on how the prepared AI artifacts connect to the SOC backend:

```text
AI artifacts
-> backend adapters
-> API contracts
-> fusion service
-> final alert DTO
-> dashboard / WebSocket path
```

Week 10 did not re-train every model. It consolidated trained or model-ready artifacts into backend contracts, real adapters, replay/tailer scripts, and a rule-based fusion layer.

## Project Context

The Hybrid IDS/AI SOC project needs more than separate AI reports. A useful SOC MVP needs an API layer that can receive events, route evidence to the correct AI components, and return a consistent alert object.

The backend MVP therefore separates responsibilities:

- API routes receive normalized security events.
- The orchestrator decides which AI adapters apply to each event.
- AI adapters return standardized model outputs.
- The fusion service converts AI outputs and rule evidence into a final alert.
- The store and WebSocket layer expose alerts to the dashboard.

This week is the bridge between AI work and cloud deployment. AWS runtime migration is intentionally left for Week 11.

## Daily Worklog

| Activity | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| Backend MVP structure review | 19/06/2026 | Estimated 4 hours | Reviewed FastAPI app structure, routes, in-memory store, WebSocket manager, and backend README | Main backend entrypoints and runtime modes were identified | Keep Week 10 focused on backend/API integration, not AWS migration | Review API contract |
| API contract review | 20/06/2026 | Estimated 4.5 hours | Reviewed event envelope, HTTP wrapper endpoint, model output contract, and final alert DTO | `POST /api/events` became the main ingestion contract | Keep one final alert DTO instead of parallel schema variants | Review adapter design |
| Model adapter review | 21/06/2026 | Estimated 5 hours | Reviewed AI1, AI2A, AI2B, mock, unavailable, and base adapter behavior | The `supports -> build_input -> predict` pattern was documented | Adapter output must distinguish `not_applicable` from `not_available` | Review fusion behavior |
| Fusion layer review | 22/06/2026 | Estimated 5 hours | Reviewed fusion rules, contributors, excluded models, severity, risk score, and final `attack_type` mapping | Fusion produced a consistent alert-level decision from partial or full AI outputs | Dashboard must use the final top-level `attack_type`, not raw model labels alone | Review replay and live paths |
| Replay and tailer path review | 23/06/2026 | Estimated 4.5 hours | Reviewed local replay script, Zeek `conn.log` tailer, Zeek `http.log` tailer, and AI2A feature enrichment path | Replay/live ingestion paths were mapped for local lab telemetry | Live path needs runtime evidence before being presented as a complete operations deployment | Consolidate validation matrix |
| Validation and evidence consolidation | 24/06/2026 | Estimated 4 hours | Reviewed backend tests and created Week 10 docs for API contract, adapters, fusion, replay, and boundaries | Week 10 was ready as a docs-first worklog | Screenshots can be added later if runtime commands are captured | Use Week 11 for AWS MVP deployment |

## Backend Architecture Overview

The backend MVP follows this flow:

```text
POST /api/events
-> normalize event envelope
-> EventOrchestrator
-> AI1 / AI2A / AI2B adapters when supported
-> FusionService
-> final alert DTO
-> in-memory alert store
-> WebSocket alert.created
-> frontend dashboard
```

The key design choice is that the backend does not force every event through every model. Each model receives only the evidence type it supports.

| Event evidence | AI path |
| --- | --- |
| `evidence.flow` | AI1 and AI2A |
| `evidence.http` | AI2B |
| `evidence.suricata` | Fusion rule evidence, future adapter path |
| Combined evidence | Multiple adapters may run |

## API Contract

Main endpoints reviewed in Week 10:

| Endpoint | Purpose |
| --- | --- |
| `GET /health` | Runtime health check |
| `POST /api/events` | Main event ingestion endpoint |
| `POST /api/events/http` | Convenience wrapper for HTTP-only events |
| `GET /api/alerts` | List alert DTOs from the in-memory store |
| `GET /api/summary` | Return summary metrics from the store |
| `POST /api/replay/demo` | Generate demo events |
| `WS /ws/alerts` | Broadcast initial data and new alerts |

The final alert DTO exposes fields such as:

```text
attack_type
severity
risk_score
detected_by
zeek_evidence
suricata_evidence
ai_analysis.ai1
ai_analysis.ai2a
ai_analysis.ai2b
ai_analysis.fusion
```

Week 10 keeps the current DTO shape. It does not introduce a parallel response schema.

## Model Adapter Design

All adapters follow the same conceptual interface:

```text
supports(event)
-> build_input(event)
-> predict(model_input)
```

Model output status is part of the contract:

| Status | Meaning |
| --- | --- |
| `completed` | Adapter ran and produced a valid prediction |
| `not_applicable` | Event does not contain evidence for that model |
| `not_available` | Evidence is compatible, but model artifact/config is unavailable |
| `failed` | Adapter was called but inference or preprocessing failed |
| `simulated` | Intentional mock/demo output |

This distinction matters because a model that does not apply to an event should not be treated as a model failure.

## AI1 Integration Boundary

AI1 is the flow-level anomaly detector.

| Item | Week 10 finding |
| --- | --- |
| Adapter | `ai1_real.py` |
| Evidence scope | `evidence.flow` |
| Expected prepared input | `evidence.flow.ai1_features` |
| Output label | `NORMAL` or `ANOMALY` |
| Missing artifact behavior | `not_available` |
| Missing feature vector behavior | `not_available` with reason |

AI1 is integration-ready as an artifact contract, but the backend still needs prepared AI1 features before real inference can run. The adapter should not infer hidden feature values from a minimal flow object.

## AI2A Integration Boundary

AI2A is the flow-level supervised attack classifier.

| Item | Week 10 finding |
| --- | --- |
| Adapter | `ai2a_real.py` |
| Evidence scope | Zeek flow evidence |
| Candidate | `rf_v2_1_full_safe_plus_ssh_minimal` |
| Frozen threshold | `0.9` |
| Feature vector | 41 frozen features |
| Below-threshold behavior | Label becomes `unknown` |
| Missing feature behavior | `not_available`, not guessed |

The replay bridge and `conn.log` tailer can enrich normalized Zeek flow rows into the frozen AI2A feature vector before posting to the backend. A small hand-written flow payload without the frozen vector is not enough for real AI2A inference.

## AI2B Integration Boundary

AI2B is the HTTP semantic detector.

| Item | Week 10 finding |
| --- | --- |
| Adapter | `ai2b_real.py` |
| Evidence scope | HTTP method and URI |
| Labels | `NONE`, `SQLI`, `XSS` |
| Model context | V1.4.9 release-candidate path |
| Output | probabilities and semantic label |

AI2B applies to HTTP events. HTTP-only events should not force AI1 or AI2A to run; those models are correctly marked `not_applicable`.

## Fusion Layer

Fusion is rule-based in the MVP. It converts model outputs into one final alert decision.

Current behavior:

| Condition | Final alert interpretation |
| --- | --- |
| AI2B `SQLI` | SQL Injection |
| AI2B `XSS` | Cross-Site Scripting |
| AI1 `ANOMALY` and AI2A non-normal | Suspicious Network Activity |
| AI1 `ANOMALY` only | Network Anomaly |
| No confirmed AI/rule signal | Benign / No Confirmed Attack |

Fusion also records:

```text
confidence_score
risk_score
mode
contributors
excluded_models
decision_version
```

Important rule:

```text
The dashboard should use the top-level final attack_type,
not the raw AI2B label alone.
```

## Replay and Live Tailer Path

Week 10 reviewed the bridge between local lab logs and the backend API.

| Script | Purpose |
| --- | --- |
| `backend/scripts/replay_local_lab_logs.py` | Replay local Zeek logs through `POST /api/events` |
| `backend/scripts/tail_zeek_conn_to_backend.py` | Stream Zeek `conn.log` rows into backend flow events |
| `backend/scripts/tail_zeek_http_to_backend.py` | Stream Zeek `http.log` rows into backend HTTP events |
| `backend/scripts/tail_zeek_correlated_to_backend.py` | Future/advanced correlated stream path |
| `backend/scripts/replay_demo.py` | Generate demo events |

The replay/tailer design is valuable because it connects Week 7 local telemetry and Week 8/9 AI datasets to the backend API.

## Validation Matrix

| Area | Evidence | Status | Week 10 interpretation |
| --- | --- | --- | --- |
| FastAPI routes | `backend/app/main.py` | Implemented | API endpoints exist |
| API contract | Backend integration contract | Implemented | Event and alert schemas are documented |
| Adapter pattern | Base, mock, real, unavailable adapters | Implemented | Model routing is explicit |
| AI1 real adapter | `ai1_real.py`, adapter tests | Partial / artifact contract | Prepared features are required |
| AI2A real adapter | `ai2a_real.py`, replay tests | Implemented with boundary | Frozen feature vector is required |
| AI2B real adapter | `ai2b_real.py` | Implemented with HTTP scope | Applies to HTTP semantic evidence |
| Fusion | `fusion.py`, fusion tests | Implemented | Rule-based final alert decision |
| Replay/tailer scripts | backend scripts | Implemented / needs runtime screenshots | Local lab logs can be routed toward API |

## Issues and Limitations

| Limitation | Impact | Follow-up |
| --- | --- | --- |
| Screenshots are not yet attached | Current Week 10 is docs-first | Add health/API/test screenshots later |
| Live tailing needs runtime evidence | Worklog should not claim full live operations deployment | Capture dry-run or live-tail output if needed |
| AI1 requires prepared feature vectors | Minimal flow evidence cannot run real AI1 | Add extraction layer before claiming full replay support |
| AWS runtime is not part of Week 10 | Cloud migration remains separate | Use Week 11 for EC2/backend deployment |

## Lessons Learned

The main lesson from Week 10 was that model integration is a contract problem, not only a model problem. A backend must define when a model applies, what happens when an artifact is missing, how predictions are represented, and how multiple partial results become one SOC alert.

Another lesson was that honest degraded modes are valuable. A system can still produce a useful alert when only AI2B applies to an HTTP event or when AI1/AI2A do not apply. That is better than forcing fake predictions just to show every model as active.

## Weekly Outcome

Week 10 established the backend integration story for the Hybrid IDS/AI SOC MVP. The project now has a clear API contract, model adapter pattern, fusion rule layer, and replay/tailer path from Zeek evidence into backend alerts.

This completes the handoff from dataset/model-readiness work into application integration. The next step is to deploy the MVP runtime on AWS in a cost-aware way.

## Next Week Plan

Week 11 will focus on AWS MVP migration and runtime deployment:

- prepare the EC2 runtime environment,
- package and run the backend,
- validate `GET /health` and API behavior on AWS,
- document runtime versions and deployment steps,
- connect the migration back to the earlier AWS foundation.
