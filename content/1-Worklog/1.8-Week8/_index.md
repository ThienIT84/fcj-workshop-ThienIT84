---
title: "Week 8 - AI Dataset Workspace & Attack Profile Coverage"
date: 2026-06-07
weight: 8
chapter: false
draft: false
pre: " <b> 1.8. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | AI dataset workspace organization |
| Main objective | Map local IDS telemetry, normal profiles, attack profiles, and AI-specific datasets |
| Dataset source | Local IDS / Zeek telemetry artifacts |
| Normal baseline | P01-P12 normal traffic dataset |
| AI components | AI1 anomaly detector, AI2A flow classifier, AI2B HTTP semantic detector |
| Main attack coverage | A01–A06 and A08–A10 artifact-backed; A07 aggregate-only; A12 HTTP semantic evidence reviewed |
| Known evidence gap | A11 not found in the current workspace scan |
| Not claimed | A11 completion, production readiness, model training completion |
| Week status | Implemented as docs-first draft; optional screenshots can be added later |

## Evidence Source Note

Week 8 is based on the dataset workspace and reports stored under logical project paths such as:

```text
Dataset/tools/final_dataset/
Dataset/tools/attack_profiles/
Dataset/tools/ai1_modeling/
Dataset/tools/ai2a_modeling/
Dataset/tools/ai2b_modeling/
```

This page does not publish raw report files, CSV files, logs, credentials, or environment files. Internal evidence indexes are maintained under:

```text
docs/worklog/week8/
```

Public screenshots are optional during the docs-first drafting stage. Before publication, selected sanitized evidence should be added to support the main workspace, metric, and coverage claims. Only redacted image files should be placed under:

```text
static/images/worklog/week8/
```

## Evidence Methodology and Status Legend

Week 8 reviewed project-relative workspace paths, dataset summaries, model handoff documents, feature manifests, QA reports, and selected raw-log evidence. Unless explicitly stated as reproduced, the metrics on this page are artifact-backed values rather than results independently regenerated during the Week 8 worklog session.

The following status labels are used in this page:

| Status | Meaning |
| --- | --- |
| Reproduced | The command or script was run again during Week 8 and produced fresh output |
| Artifact-backed | An existing report or dataset artifact supports the claim |
| Aggregate-only | Evidence exists only in a combined dataset or report, not as a standalone profile summary |
| Context-only | Reviewed to understand architecture or input contracts, not presented as a Week 8 implementation result |
| Not found | No relevant artifact was found in the current workspace scan |
| Deferred | Available material is reserved for deeper Week 9 analysis |

## Objective

The objective of Week 8 was to organize the dataset workspace behind the Hybrid IDS/AI SOC project and explain how different telemetry artifacts support different AI components.

Instead of treating the dataset work as a single flat collection task, Week 8 separates the evidence into three AI responsibilities:

- AI1 uses Zeek flow features for anomaly detection.
- AI2A uses Zeek `conn.log`-derived flow features for supervised network attack classification.
- AI2B uses Zeek `http.log`-derived HTTP evidence for SQLi/XSS semantic classification.

## Project Context

Week 7 validated that the local lab could produce Zeek telemetry. Week 8 moves from "can the lab generate telemetry?" to "how is the telemetry organized into datasets for the AI system?"

The dataset workspace connects the local IDS lab to the later SOC MVP:

```text
Local IDS traffic
-> Zeek conn.log and http.log
-> normal and attack profile datasets
-> AI1 / AI2A / AI2B dataset inputs
-> backend and fusion-layer integration in later weeks
```

Week 8 therefore focuses on dataset coverage and evidence boundaries, not on final model performance.

## AI Dataset Architecture

| Component | Main input | Purpose | Week 8 position |
| --- | --- | --- | --- |
| AI1 | Flow features from Zeek `conn.log` | Unsupervised anomaly detection | Artifact context reviewed |
| AI2A | Flow-level Zeek `conn.log` features | Supervised multi-class network attack classification | Dataset workspace and profile coverage reviewed |
| AI2B | Normalized URI/query/request-line evidence derived from Zeek `http.log` | HTTP semantic classification for `NONE`, `SQLI`, and `XSS` | A12 evidence and dataset workspace reviewed |
| Fusion layer | AI outputs and rule evidence | Final risk score | Deferred to later backend/system design work |

The key design decision is to keep collection evidence, model features, labels, and downstream risk scoring separate. Week 8 does not claim that the risk score is already part of the dataset.

AI2B focused on normalized URI/query and request-line evidence derived from Zeek HTTP telemetry. Week 8 does not claim HTTP request-body inspection or full application-layer payload coverage.

## Evidence Review Sequence

| Activity | Date | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- |
| Dataset workspace review | 07/06/2026 | Reviewed the local dataset workspace and separated normal data, attack profiles, and AI-specific modeling folders | The dataset workspace was mapped around AI1, AI2A, and AI2B responsibilities | Avoided writing Week 8 as only a raw traffic collection page | Build the attack profile coverage matrix |
| Normal and AI2A baseline review | 08/06/2026 | Reviewed the P01-P12 normal dataset summary and AI2A classifier dataset contract | Normal baseline and AI2A schema policy were identified | Keep row counts as evidence, but defer deeper QA interpretation to Week 9 | Review attack profile evidence |
| Attack profile coverage review | 10/06/2026 | Checked A01-A10 attack profile folders, summaries, merged datasets, and QA/report artifacts | A01-A06 and A08-A10 had validated evidence; A07 was available through aggregate dataset evidence; A11 was not found | Do not claim A11 or overstate A07 profile-level status | Review AI2B/A12 artifacts |
| HTTP semantic evidence review | 12/06/2026 | Reviewed A12 SQLi/XSS raw HTTP logs and AI2B dataset/report artifacts | A12 was confirmed as the HTTP semantic source for AI2B | Keep A12 separate from AI2A flow profile coverage | Review AI1 context |
| AI1 and claim-boundary review | 13/06/2026 | Reviewed AI1 handoff, model card, feature manifest context, and known limitation | AI1 was documented as anomaly-detection context rather than Week 8 training output | Backend live/replay feature extraction remains a later integration item | Prepare Week 8 docs |
| Evidence consolidation | 13/06/2026 | Built the Week 8 evidence index, source inventory, AI mapping, and claim boundary | Week 8 could be explained from docs and tables without screenshot placeholders | Optional screenshots can be added later if needed | Use Week 9 for deeper QA and model-readiness validation |

## Dataset Workspace Layout

| Workspace area | Purpose | Week 8 interpretation |
| --- | --- | --- |
| `Dataset/tools/final_dataset/` | Normal P01-P12 dataset and merge reports | Artifact-backed |
| `Dataset/tools/attack_profiles/` | Attack campaign profiles, profile summaries, merged datasets, and QA reports | Main source for AI2A attack coverage |
| `Dataset/tools/attack_profiles/final_attack_dataset/` | AI2A classifier dataset contract, merged CSV, QA report, and training strategy | AI2A dataset workspace anchor |
| `Dataset/tools/ai1_modeling/` | AI1 handoff, model card, feature manifest, threshold, and smoke evidence | AI1 anomaly-detection context |
| `Dataset/tools/ai2a_modeling/` | AI2A release-candidate and model-readiness reports | Deferred to Week 9 for deeper validation |
| `Dataset/tools/ai2b_modeling/` | AI2B HTTP semantic datasets, shortcut checks, freeze review, and holdout notes | Deferred to Week 9 for deeper validation |

## Normal Dataset Baseline

The normal baseline uses the P01-P12 final normal dataset summary stored under `Dataset/tools/final_dataset/`. Metrics below are artifact-backed values from the existing summary report.

| Metric | Value |
| --- | ---: |
| Rows | 197,797 |
| Columns | 70 |
| Source profiles | 12 |
| `missed_bytes` rows | 0 |
| IPv6 rows | 0 |
| Duplicate UID | 0 |
| Missing cells | 0 |
| Dropped missing rows | 154 |
| Dropped duplicate UID | 0 |

The reviewed report describes a large normal reference corpus for AI1 and AI2A. Week 8 records it as dataset workspace evidence. Deeper verification of split suitability, leakage risk, and model readiness is deferred to Week 9.

## AI2A Attack Profile Coverage

AI2A is the supervised flow-level classifier. It uses Zeek `conn.log`-derived features and maps attack-profile evidence into supervised classes such as `port_scan_or_recon`, `ssh_bruteforce_indicator`, and `web_initial_access_indicator`.

The AI2A classifier dataset evidence comes from the final classifier dataset contract stored under `Dataset/tools/attack_profiles/final_attack_dataset/`. Metrics below are artifact-backed values.

| Metric | Value |
| --- | ---: |
| Rows | 207,881 |
| Columns | 77 |
| Run count | 281 |
| Model feature count | 53 |
| Duplicate UID | 0 |

Week 8 uses these numbers to show that the dataset workspace exists and is organized. Final model-readiness, release-candidate evaluation, and per-class validation details are reserved for Week 9.

| Profile | AI2A label or role | Week 8 status | Evidence interpretation |
| --- | --- | --- | --- |
| A01 Discovery | `port_scan_or_recon` | Artifact-backed | Final summary and merged dataset evidence found |
| A02 SSH authentication | `ssh_bruteforce_indicator` | Artifact-backed | Core and holdout dataset reports found. SSH flow evidence exists; because SSH authentication content is encrypted, Zeek `conn.log` alone does not provide definitive authentication outcomes |
| A03 Web initial access | `web_initial_access_indicator` | Artifact-backed | Final summary and core/holdout dataset reports found |
| A04 HTTP beaconing | `http_beaconing_indicator` | Artifact-backed | Final summary and dataset evidence found |
| A05 DNS anomaly | `dns_anomaly_indicator` | Artifact-backed | Final summary and dataset evidence found |
| A06 Collection staging | `collection_staging_indicator` | Artifact-backed | Final summary and dataset evidence found |
| A07 Controlled exfiltration | `controlled_exfiltration` | Aggregate-only | Referenced in aggregate AI2A dataset documentation; no standalone profile-level final summary was found in the current scan |
| A08 Suspicious admin | `suspicious_admin` | Artifact-backed | Final summary and QA/report evidence found |
| A09 East-west simulation | `east_west_simulation` | Artifact-backed | Setup summary, final summary, and QA/report evidence found |
| A10 Mixed attack chain | Per-flow stage labels | Artifact-backed | Final summary and QA/report evidence found |
| A11 | Not claimed | Not found | No current artifact found, so Week 8 does not claim A11 |

## AI2B HTTP Semantic Coverage

AI2B is separate from AI2A. It focuses on HTTP request semantics rather than general network-flow classification.

The main Week 8 evidence comes from A12 Web Semantic artifacts:

| Evidence area | Interpretation |
| --- | --- |
| A12 SQLi HTTP logs | Zeek `http.log` rows contain SQLi-style normalized URI/query evidence |
| A12 XSS HTTP logs | Zeek `http.log` rows contain XSS-style normalized URI/query evidence |
| AI2B HTTP datasets | Structured HTTP semantic datasets were built for `NONE`, `SQLI`, and `XSS` |
| AI2B freeze/holdout reports | Useful model-readiness context, but deeper discussion is deferred to Week 9 |

Week 8 can safely state that A12 is the HTTP semantic evidence source for AI2B. It does not claim final AI2B blind-holdout metrics.

AI2B focused on normalized URI/query and request-line evidence. Week 8 does not claim HTTP request-body inspection, full payload coverage, cookie/session inspection, or browser-rendered content analysis.

## AI1 Anomaly Dataset Context

AI1 is the anomaly-detection component. The reviewed handoff and model card describe an Isolation Forest model that expects a 30-feature vector extracted from Zeek `conn.log` rows.

Week 8 treats AI1 as dataset-architecture context:

| Item | Week 8 note |
| --- | --- |
| Model purpose | Network anomaly detection |
| Expected input | Flow feature vector from Zeek `conn.log` |
| Feature count | 30 |
| Decision type | `NORMAL` or `ANOMALY` |
| Known limitation | Backend does not yet extract AI1 features directly from raw `conn.log` rows |

The implementation and API integration evidence for AI1 belongs to later backend and model-readiness work, not to Week 8 dataset coverage.

## Evidence Coverage Matrix

| Area | Primary evidence | Status | Week 8 claim |
| --- | --- | --- | --- |
| Normal P01-P12 | Final normal dataset summary (`Dataset/tools/final_dataset/`) | Artifact-backed | Normal traffic baseline exists and has quality metrics |
| AI2A profile coverage | Attack profile summaries (`Dataset/tools/attack_profiles/`), final classifier dataset | Artifact-backed / aggregate-only by profile | AI2A has a structured flow-classification dataset workspace |
| A02 SSH | Core/holdout dataset reports (`Dataset/tools/attack_profiles/`) | Artifact-backed with limitation | SSH flow evidence exists; encrypted auth results are not visible in Zeek `conn.log` |
| A07 exfiltration | Final AI2A aggregate dataset and strategy evidence | Aggregate-only | Referenced in aggregate AI2A dataset; no profile-level final summary is claimed |
| A11 | No artifact found | Not found | Excluded from Week 8 claims |
| A12 HTTP semantic | Raw HTTP logs and AI2B artifacts (`Dataset/tools/ai2b_modeling/`) | Artifact-backed | A12 supports AI2B normalized URI/query semantic evidence |
| AI1 | Handoff, model card, feature manifest context | Context-only | AI1 dataset input requirements are understood |

## Claim Boundary and Gaps

Safe Week 8 claims:

- A dataset workspace was organized for AI1, AI2A, and AI2B.
- Normal traffic P01-P12 exists as a validated baseline dataset.
- AI2A uses flow-level Zeek `conn.log` features and attack profiles A01-A10.
- AI2B uses HTTP semantic evidence from A12 SQLi/XSS work.
- A02 has validated dataset reports, with SSH encryption limitation clearly explained.
- A11 is not claimed because evidence was not found.

Out-of-scope claims:

- The full numbered attack-profile range is complete.
- The dataset is ready for production use.
- Model training completion belongs to later validation work.
- Production-grade split policy is complete.
- AI2B blind-holdout scores are available.
- A07 has a profile-level final summary.

## Issues and Decisions

| Issue / observation | Decision |
| --- | --- |
| Dataset workspace contains artifacts for multiple AI components | Separate ownership by AI1, AI2A, and AI2B |
| A07 lacks standalone final summary | Report it as aggregate-only |
| A11 artifact not found | Do not claim A11 |
| SSH authentication content is encrypted | Limit A02 claims to flow-pattern indicators only |
| AI2B uses normalized URI/query/request-line evidence | Do not claim request-body inspection or full payload coverage |
| Existing metrics came from saved reports, not re-run scripts | Label them artifact-backed |
| AI1 live feature extraction is incomplete | Defer backend integration to later weeks |
| Model-readiness reports are extensive | Move deeper evaluation to Week 9 |

## Lessons Learned

The main lesson from Week 8 was that a security dataset should not be described only by the number of collected rows. The more important question is whether each dataset artifact has a clear source, a clear AI owner, a defensible label meaning, and a known limitation.

Another lesson was that different AI components need different evidence. AI2A can rely mainly on flow-level `conn.log` features, while AI2B needs HTTP request-level evidence. AI1 has a separate anomaly-detection feature contract and should not be mixed into the supervised classifier dataset without a clear boundary.

## Weekly Outcome

Week 8 produced a clean map of the AI dataset workspace. The project now has a documented relationship between local IDS telemetry, normal traffic profiles, attack profiles, and AI-specific datasets.

The Week 8 worklog intentionally stays at the coverage and organization layer. Week 9 will move into deeper dataset QA, leakage/split policy, AI2A model-readiness evidence, and AI2B freeze/holdout limitations.

## Next Week Plan

In Week 9, the worklog will focus on AI dataset QA and model-readiness validation:

- AI1 anomaly model handoff and feature contract.
- AI2A classifier dataset QA, class distribution, split policy, and release-candidate evidence.
- AI2B HTTP semantic dataset QA, shortcut checks, freeze review, and incomplete holdout boundary.
- Clear explanation of what is ready for demo and what remains future validation work.
