---
title : "Fusion Schema and Decision Flow"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---


## Objective

Document the Fusion schema and how model signals should become a final alert contract.

## Fusion schema

Fusion combines AI outputs and security evidence into one final alert object with fields such as event ID, model results, severity, risk score, confidence score, reason, and downstream storage target.

![Fusion schema](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-tin-21-fusion-schema.png)

## Decision flow

1. Receive AI1 output and available evidence.
2. Check whether AI2A or AI2B output is available.
3. Mark unavailable model outputs explicitly.
4. Combine score and evidence into one final alert.
5. Pass the final alert to RDS and downstream dashboard consumers.

## Final alert contract

The final alert should be stable enough for storage and dashboard usage, even when some model outputs are mock or unavailable.
