# Week 10 Model Adapter Matrix

## Adapter Pattern

All adapters follow the same conceptual flow:

```text
supports(event)
-> build_input(event)
-> predict(model_input)
```

## Status Semantics

| Status | Meaning |
| --- | --- |
| `completed` | Adapter ran and returned a valid prediction |
| `not_applicable` | Event evidence does not match the model scope |
| `not_available` | Evidence is compatible, but artifact/config is missing or unavailable |
| `failed` | Adapter was called but inference/preprocessing failed |
| `simulated` | Intentional demo/mock output |

## Adapter Matrix

| Adapter | Evidence scope | Real behavior | Boundary |
| --- | --- | --- | --- |
| AI1 | Flow evidence with prepared `ai1_features` | Outputs `NORMAL` or `ANOMALY` | Feature vector must already be prepared |
| AI2A | Flow evidence with frozen AI2A features | Outputs supervised flow class or `unknown` | Requires 41 frozen features and threshold `0.9` |
| AI2B | HTTP method and URI | Outputs `NONE`, `SQLI`, or `XSS` | Applies to HTTP semantic evidence only |
| Mock adapters | Dev/demo evidence | Intentional simulated outputs | Must be shown as mock/simulated |
| Unavailable adapters | Compatible scope but unavailable artifacts | Explicit `not_available` | Must not pretend to be a real model |

## Important Rule

Do not force every event through every model. HTTP-only events should not run flow models, and flow-only events should not run the HTTP semantic model.
