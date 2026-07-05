# Week 8 Worklog Draft

## Title

Week 8 - AI Dataset Workspace & Attack Profile Coverage

## Narrative

During Week 8, I reviewed and organized the dataset workspace used by the Hybrid IDS/AI SOC project. The work focused on connecting local IDS telemetry to the AI components rather than presenting the dataset as one flat collection output.

The dataset workspace was separated into three responsibilities:

- AI1 uses Zeek flow features for anomaly detection.
- AI2A uses flow-level Zeek `conn.log` features for supervised multi-class attack classification.
- AI2B uses Zeek `http.log` evidence for SQLi/XSS HTTP semantic classification.

The normal baseline was documented through the P01-P12 final normal dataset. Attack profile coverage was reviewed across A01-A10 and A12. A11 was not claimed because no artifact was found in the current workspace scan.

## Key Outcomes

- Mapped the dataset workspace around AI1, AI2A, and AI2B.
- Confirmed the P01-P12 normal baseline as a validated dataset source.
- Confirmed AI2A profile coverage using A01-A10 evidence, with A07 marked as aggregate evidence only.
- Corrected A02 status from documented-only to validated dataset evidence.
- Kept the A02 SSH limitation explicit because encrypted authentication results are not visible in Zeek `conn.log`.
- Confirmed A12 as the HTTP semantic evidence source for AI2B.
- Deferred model-readiness and holdout interpretation to Week 9.

## Draft Daily Worklog

| Date | Work completed | Result |
| --- | --- | --- |
| 07/06/2026 | Reviewed the dataset workspace layout and separated normal, attack, and AI-specific artifacts | Workspace responsibilities were mapped to AI1, AI2A, and AI2B |
| 08/06/2026 | Reviewed normal P01-P12 and AI2A schema/contract evidence | Normal baseline and AI2A dataset policy were identified |
| 09/06/2026 | Reviewed attack profile coverage for A01-A10 | A01-A06 and A08-A10 were validated; A07 was marked aggregate; A11 was not claimed |
| 10/06/2026 | Reviewed A12 HTTP logs and AI2B dataset evidence | A12 was confirmed as SQLi/XSS semantic evidence |
| 11/06/2026 | Reviewed AI1 handoff and feature requirements | AI1 was documented as anomaly-detection dataset context |
| 12/06/2026 | Consolidated evidence matrix and claim boundaries | Week 8 became ready for a docs-first Hugo draft |

## Handoff To Week 9

Week 9 should focus on deeper QA and model-readiness:

- AI1 feature contract and smoke evidence.
- AI2A classifier dataset QA and release-candidate validation.
- AI2B shortcut checks, freeze review, and incomplete holdout boundary.
- Split/leakage policy and production-readiness limitations.
