# {{NOTEBOOK_LABEL}} Post-Apply Prose and Camera Adversarial Review

Conduct the exact post-apply review described here. Do not critique, rewrite,
or review this prompt. Do not edit the live notebook or any pinned artifact.

Auditor lane: **{{AUDITOR_LANE}}**

## Pins and drift

Verify every pin byte-exact:

```text
live notebook          {{NOTEBOOK_PATH}}              {{APPLIED_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
approved candidate     {{CANDIDATE_PATH}}             {{CANDIDATE_SHA256}}
final adjudication     {{ADJUDICATION_PATH}}           {{ADJUDICATION_SHA256}}
applied delta report   {{DELTA_REPORT}}                {{DELTA_REPORT_SHA256}}
final render           {{FINAL_RENDER}}                {{FINAL_RENDER_SHA256}}
changed-run crops      {{CROP_MANIFEST}}               {{CROP_MANIFEST_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
course profile         {{PROFILE_PATH}}                {{PROFILE_SHA256}}
```

Expected live SHA-256: `{{APPLIED_SHA256}}`. Permitted drift:
{{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on anything else. Re-hash immediately
before the verdict.

No paid or network calls. Mark unproved runtime or SDK claims `UNVERIFIED`.

## Applied scope

{{EXACT_APPLIED_SCOPE}}

Verify that the live notebook reproduces this scope exactly before reviewing
quality. Report any extra or missing delta as blocking.

## Adversarial task

Read every cell in source order and inspect the complete final render at the
documented presentation scale. Treat adjacent markdown cells as one surface.

Look specifically for:

1. a regression created by the union of applied changes;
2. a scope, ownership, or causal claim that moved to the wrong surface;
3. a newly overloaded adjacent-markdown run;
4. a label-only path that hides polarity or the main claim;
5. a recap that became the first spoken owner of a concept; and
6. a remaining high-impact defect that candidate-scoped review could not see.

Do not restart the prose pass or manufacture polish. Classify every observation
as `REAL`, `WORTHWHILE UPGRADE`, or `NIT`. For `REAL`, name what a student would
misunderstand or what would visibly fail. Give complete exact source only for a
necessary repair.

## Required report

- Pins, ruling-time re-hash, and stale/not stale in one line.
- One line attesting every cell and the complete render were inspected.
- Applied-scope verification.
- Ranked findings with classifications and baseline defenses.
- Ownership, outline, adjacent-run, and camera verdicts.
- Exact expected scope and verification boundary.
- Exactly one overall verdict and an explicit next step.

Do not write to the notebook. Return the review to Codex.
