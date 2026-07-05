# Week 8 Claim Boundary

## Safe Claims

- The dataset workspace was organized for AI1, AI2A, and AI2B.
- Normal traffic P01-P12 exists as a validated baseline dataset.
- AI2A uses flow-level Zeek `conn.log` features and attack profiles A01-A10.
- AI2B uses HTTP semantic evidence from A12 SQLi/XSS work.
- A02 has validated dataset reports.
- A02 must still mention the SSH limitation: Zeek `conn.log` observes flow behavior, not encrypted authentication result details.
- A11 is not claimed because no evidence was found in the current workspace scan.

## Avoided Claims

- Do not say the full numbered attack-profile range is complete.
- Do not say the dataset is production-ready.
- Do not say model training completion happened in Week 8.
- Do not say the leakage-free production split is complete.
- Do not say AI2B blind-holdout scores are available.
- Do not say A07 has a profile-level final summary unless new evidence is provided.

## Wording To Prefer

```text
Week 8 organized the AI dataset workspace and documented which evidence supports
AI1, AI2A, and AI2B. Deeper QA, split policy, release-candidate interpretation,
and holdout limitations are deferred to Week 9.
```

## Wording To Avoid

```text
The full numbered attack-profile range was validated.
AI2B holdout scoring passed.
The model was fully trained and ready for production use in Week 8.
```
