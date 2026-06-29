# Evidence Intake and Worklog Layout Guide

This guide explains how to send screenshots and how they will be converted into clean Hugo worklog evidence.

## Can Screenshots Be Sent Without Filenames?

Yes.

If you paste or upload screenshots directly into the chat, I can usually understand what they show from:

- AWS Console page title
- Service name in the sidebar
- Table headings
- Resource names
- Success messages
- Error messages such as `AccessDenied`
- URL or breadcrumb, if visible

However, filenames are still useful when the image becomes part of the Hugo site. Hugo needs stable paths like:

```markdown
![Expected AccessDenied result](/images/worklog/week1/13-access-denied-test.png)
```

So the workflow can be:

```text
You send raw screenshots
-> I identify the evidence type
-> I suggest clean filenames
-> Images are placed under static/images/worklog/weekX/
-> The Hugo page references those stable filenames
```

## Recommended Folder Structure

Use one folder per week:

```text
static/images/worklog/week1/
static/images/worklog/week2/
static/images/worklog/week3/
...
static/images/worklog/week12/
```

Suggested naming style:

```text
01-short-description.png
02-short-description.png
03-short-description.png
```

Examples:

```text
01-root-mfa-status.png
04-iam-users-list.png
13-access-denied-test.png
```

## What To Send With Each Screenshot

Best case:

```text
Week 1 - root MFA status
```

Still okay:

```text
Week 1 IAM evidence, please classify
```

Also okay:

```text
I do not know where this should go. Please place it.
```

## Evidence Quality Rules

Good evidence should be:

- Directly related to a claim in the worklog.
- Clear enough for a reviewer to understand without opening AWS.
- Cropped to the important area.
- Free of passwords, access keys, MFA QR codes, and full account IDs.
- Accompanied by one sentence explaining what it proves.

## Layout Rule

Do not dump all screenshots at the bottom.

Place evidence near the related section:

```text
IAM design section
-> users/groups screenshots

Validation section
-> sign-in, allowed operation, AccessDenied screenshots

Security section
-> MFA, no root access key screenshots
```

## Evaluator-Friendly Page Structure

For each weekly worklog page, use this structure:

```text
1. Objective
2. Context
3. Implementation
4. Validation
5. Evidence
6. Issues and troubleshooting
7. Lessons learned
8. Outcome
9. Next week
```

## How To Get a High Score

A strong worklog does not only list tasks. It proves progress.

Each week should answer:

- What problem did this week solve?
- What did I implement or validate?
- Which AWS services or tools were involved?
- What evidence proves it?
- What issue did I encounter?
- What did I learn?
- How does this prepare the next week?

Strong wording:

```text
Implemented and validated
Configured and tested
Verified using AccessDenied
Measured with CloudWatch
Troubleshot using Reachability Analyzer
Documented trade-offs
```

Weak wording:

```text
Learned about
Studied
Prepared
Tried
Explored
```

Use weak wording only when the activity was truly research or design.

## Minimum Evidence Per Week

Aim for 5 to 8 strong evidence items per week:

- 1 architecture or design screenshot/diagram
- 2 implementation screenshots
- 1 successful test
- 1 failure or denied test
- 1 log/metric/output, when applicable
- 1 short reflection or troubleshooting note

For Week 1 IAM, the strongest evidence set is:

- Root MFA enabled
- No root access key
- IAM users list
- IAM groups list
- Group membership
- MFA enabled for active IAM users
- Successful sign-in
- Expected `AccessDenied`

## When There Are Too Many Screenshots

Use only the best screenshots in the main worklog.

Move the rest to:

```text
Evidence appendix
Additional screenshots
```

or keep them in the folder without displaying every image.

