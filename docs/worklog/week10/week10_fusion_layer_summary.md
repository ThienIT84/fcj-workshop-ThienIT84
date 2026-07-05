# Week 10 Fusion Layer Summary

## Purpose

Fusion turns partial model outputs into one alert-level decision for the SOC dashboard.

## Rule Summary

| Condition | Final interpretation |
| --- | --- |
| AI2B `SQLI` | SQL Injection |
| AI2B `XSS` | Cross-Site Scripting |
| AI1 `ANOMALY` and AI2A non-normal | Suspicious Network Activity |
| AI1 `ANOMALY` only | Network Anomaly |
| No confirmed signal | Benign / No Confirmed Attack |

## Fusion Output

Fusion records:

```text
confidence_score
risk_score
reason
mode
contributors
excluded_models
decision_version
```

## Boundary

The dashboard should use the final top-level `attack_type`. Raw model labels are evidence, not the final SOC decision by themselves.
