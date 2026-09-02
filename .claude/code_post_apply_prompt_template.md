# {{NOTEBOOK_LABEL}} Post-Apply Code-Readability Adversarial Review

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
verification report    {{VERIFY_REPORT}}               {{VERIFY_REPORT_SHA256}}
final render           {{FINAL_RENDER}}                {{FINAL_RENDER_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
course profile         {{PROFILE_PATH}}                {{PROFILE_SHA256}}
code process           {{CODE_PROCESS_PATH}}           {{CODE_PROCESS_SHA256}}
```

Expected live SHA-256: `{{APPLIED_SHA256}}`. Permitted drift:
{{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on anything else. Re-hash immediately
before the verdict.

No paid or network calls unless explicitly authorized. Mark unproved runtime
or SDK claims `UNVERIFIED`.

## Applied scope

{{EXACT_APPLIED_SCOPE}}

Verify this scope before reviewing quality. Any extra or missing delta is
blocking.

## Adversarial task

Read every cell in source order and inspect the finished render. Try to falsify
the applied result through the changed normal, failure, rerun, state, exercise,
and immediate out-of-order paths. Look for a regression created by the union of
changes, a later cell still depending on the old shape, an abstraction or name
that moved explanation cost elsewhere, or a new claim without evidence.

This is not another style pass. Classify observations as `REAL`, `WORTHWHILE
UPGRADE`, or `NIT`. A real issue must identify what a student misunderstands,
what execution fails, or what paid work is wasted.

## Required report

- Pins, ruling-time re-hash, and stale/not stale in one line.
- One line attesting every cell and the complete render were inspected.
- Applied-scope verification.
- Ranked findings with classifications and hostile probes.
- Student-path, state, exercise, outline, and paid-call verdicts.
- Exact source only for necessary repairs.
- Exactly one overall verdict and an explicit next step.

Do not write files. Return the review to Codex.
