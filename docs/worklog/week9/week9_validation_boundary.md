# Week 9 Validation Boundary

## Safe Claims

- AI1 has a documented artifact contract, feature list, threshold, and smoke evidence.
- AI2A has classifier dataset QA and an accepted release candidate.
- AI2A metrics should be reported with per-class context.
- AI2B has HTTP semantic dataset QA, shortcut sanity evidence, and a frozen candidate.
- AI2B holdout attempts were blocked before prediction by contamination or inventory guards.

## Avoided Claims

- Do not claim live AI1 replay from raw `conn.log` until feature extraction is implemented.
- Do not claim AI2A class performance is uniformly strong across every rare class.
- Do not claim AI2B holdout scoring success.
- Do not claim long-term operational readiness.
- Do not hide the AI2B holdout boundary.

## Recommended Wording

```text
Week 9 reviewed AI dataset QA and model-readiness evidence. AI1 has an artifact
contract, AI2A has classifier QA and release-candidate evidence, and AI2B has a
frozen HTTP semantic candidate. AI2B holdout scoring remains future validation
because protocol guards blocked the current attempts before prediction.
```
