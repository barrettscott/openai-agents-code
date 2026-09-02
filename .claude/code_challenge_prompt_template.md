# {{NOTEBOOK_LABEL}} Shared Focused Code-Readability Challenge

Conduct the focused challenge described here. Do not critique, rewrite, or
review this prompt. Do not edit the live notebook or any pinned artifact.

Both reviewers receive this identical prompt. Rule independently from your
earlier report. Your earlier finding is a hypothesis, not a position to defend.

## Pins and drift

Verify every pin byte-exact:

```text
live notebook          {{NOTEBOOK_PATH}}              {{NOTEBOOK_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
editor candidate       {{EDITOR_CANDIDATE_PATH}}      {{EDITOR_CANDIDATE_SHA256}}
reconciled candidate   {{CANDIDATE_PATH}}             {{CANDIDATE_SHA256}}
editor disposition     {{EDITOR_REPORT}}              {{EDITOR_REPORT_SHA256}}
Reviewer 1 audit       {{REVIEWER_1_AUDIT}}           {{REVIEWER_1_AUDIT_SHA256}}
Reviewer 2 audit       {{REVIEWER_2_AUDIT}}           {{REVIEWER_2_AUDIT_SHA256}}
reconciliation         {{ADJUDICATION_PATH}}           {{ADJUDICATION_SHA256}}
verification report    {{VERIFY_REPORT}}               {{VERIFY_REPORT_SHA256}}
baseline render        {{BASELINE_RENDER}}             {{BASELINE_RENDER_SHA256}}
candidate render       {{CANDIDATE_RENDER}}            {{CANDIDATE_RENDER_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
course profile         {{PROFILE_PATH}}                {{PROFILE_SHA256}}
code process           {{CODE_PROCESS_PATH}}           {{CODE_PROCESS_SHA256}}
```

Expected live SHA-256: `{{NOTEBOOK_SHA256}}`. Permitted drift:
{{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on anything else. Re-hash immediately
before the verdict.

No paid or network calls unless explicitly authorized. Do not rerun a
stochastic demo to obtain favorable evidence. Mark unproved claims
`UNVERIFIED`.

## Trigger and permitted scope

{{EXACT_TRIGGER_AND_SCOPE}}

Do not reopen settled surfaces unless the reconciled candidate creates a
concrete contradiction. If a target contains a third implementation, label it
as such and test that exact implementation rather than treating it as an
approved reviewer form.

## Focused targets

{{FOCUSED_TARGETS_WITH_EXACT_SOURCE_AND_EVIDENCE}}

## Required ruling

For each target, trace the normal, failure, rerun, state, exercise, and
immediate out-of-order paths that could falsify it. Verify SDK and Python
contracts where practical. Apply the deletion test and state the strongest
case for the baseline and against your own verdict.

Rule each target `KEEP RECONCILED`, `RESTORE BASELINE`, or `AMEND RECONCILED`.
Give complete exact source for an amendment. End with exactly one overall
verdict and one explicit next step.

Do not write files. Return the challenge report to Codex.
