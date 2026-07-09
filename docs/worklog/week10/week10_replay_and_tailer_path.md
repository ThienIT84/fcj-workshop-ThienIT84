# Week 10 Replay And Tailer Path

## Purpose

Week 10 connects local IDS evidence to the backend API through replay and tailer scripts.

## Scripts

| Script | Purpose |
| --- | --- |
| `backend/scripts/replay_local_lab_logs.py` | Replay saved Zeek logs into the backend API |
| `backend/scripts/tail_zeek_conn_to_backend.py` | Stream Zeek `conn.log` rows as flow events |
| `backend/scripts/tail_zeek_http_to_backend.py` | Stream Zeek `http.log` rows as HTTP events |
| `backend/scripts/tail_zeek_correlated_to_backend.py` | Future/advanced correlated stream path |
| `backend/scripts/replay_demo.py` | Generate demo alerts |

## Data Flow

```text
Zeek conn.log / http.log
-> parser or tailer
-> normalized event
-> POST /api/events
-> adapters
-> fusion
-> alert store and WebSocket
```

## Boundary

The scripts provide the bridge from local lab evidence to the backend. Runtime screenshots or logs should be added before presenting the live path as fully demonstrated.
