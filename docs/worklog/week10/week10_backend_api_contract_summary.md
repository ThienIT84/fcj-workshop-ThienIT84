# Week 10 Backend API Contract Summary

## Purpose

Week 10 documents the backend API contract that connects model-ready artifacts to the SOC MVP.

## Main Flow

```text
POST /api/events
-> normalize event envelope
-> EventOrchestrator
-> AI1 / AI2A / AI2B adapters when supported
-> FusionService
-> final alert DTO
-> in-memory alert store
-> WebSocket alert.created
-> dashboard
```

## Endpoints

| Endpoint | Purpose |
| --- | --- |
| `GET /health` | Runtime health check |
| `POST /api/events` | Main ingestion endpoint |
| `POST /api/events/http` | HTTP-only wrapper |
| `GET /api/alerts` | List alerts |
| `GET /api/summary` | Store summary |
| `POST /api/replay/demo` | Demo event replay |
| `WS /ws/alerts` | Alert stream |

## Event Evidence

| Evidence | Model path |
| --- | --- |
| `evidence.flow` | AI1 and AI2A |
| `evidence.http` | AI2B |
| `evidence.suricata` | Fusion rule evidence / future adapter |

## Alert DTO

The dashboard consumes the final alert object with:

```text
attack_type
severity
risk_score
detected_by
ai_analysis.ai1
ai_analysis.ai2a
ai_analysis.ai2b
ai_analysis.fusion
```

Week 10 keeps this DTO as the current contract and does not introduce a second response shape.
