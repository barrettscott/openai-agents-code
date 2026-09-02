# {{NOTEBOOK_LABEL}} Focused Exact-Candidate Challenge

Conduct the focused challenge described here. Do not critique, rewrite, or
review this prompt. Do not edit any pinned artifact.

Verify every pin byte-exact:

```text
live notebook          {{NOTEBOOK_PATH}}              {{NOTEBOOK_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
exact candidate        {{CANDIDATE_PATH}}             {{CANDIDATE_SHA256}}
editor report          {{EDITOR_REPORT}}              {{EDITOR_REPORT_SHA256}}
camera audit           {{CAMERA_AUDIT}}               {{CAMERA_AUDIT_SHA256}}
technical audit        {{TECHNICAL_AUDIT}}            {{TECHNICAL_AUDIT_SHA256}}
adjudication           {{ADJUDICATION}}               {{ADJUDICATION_SHA256}}
ownership diff         {{OWNERSHIP_REPORT}}           {{OWNERSHIP_REPORT_SHA256}}
baseline render        {{BASELINE_RENDER}}            {{BASELINE_RENDER_SHA256}}
candidate render       {{CANDIDATE_RENDER}}           {{CANDIDATE_RENDER_SHA256}}
challenge crops        {{CROP_MANIFEST}}              {{CROP_MANIFEST_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
```

Expected live notebook SHA-256: `{{NOTEBOOK_SHA256}}`. Permitted drift:
{{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on anything else and re-hash immediately
before the verdict.

> Rule independently from your earlier report. Your earlier finding is a
> hypothesis, not a position to defend.

No paid or network calls. Mark unproved runtime or SDK claims `UNVERIFIED`.

## Why this challenge fired

{{EXACT_TRIGGER_AND_SCOPE}}

Valid triggers are limited to an overruled `REVERT`, both auditors flagging one
surface with neither cure accepted, an adjudicated form nobody submitted, a
change to executable/model-visible behavior or what the demo proves, or an
ownership loss that leaves teaching only in code/output/exercise/recap.

## Challenge contract

- Attack the proposed cure, not the reviewer.
- Test the complete adjacent-markdown run at native scale.
- Read the label-only spoken path for claims and polarity.
- Trace every deleted teaching claim to its remaining spoken markdown home.
- Verify the exact cell sequence and outline scope.
- Prefer the baseline when the cure has not cleared its burden.
- A form nobody previously submitted must be labelled `THIRD IMPLEMENTATION`
  and shown beside the baseline, editor form, and auditor forms.

## Targets

{{FOCUSED_TARGETS_WITH_EXACT_SOURCE_AND_CROPS}}

## Required report

- Pins and ruling-time re-hash in one line.
- One ruling per target with the strongest concrete defense of the baseline.
- Exact final source for every accepted or amended target.
- Full-run crop path and measured scan path for every markdown ruling.
- Ownership and outline disposition.
- Exact expected scope and verification boundary.
- Exactly one overall verdict and an explicit next step.

Do not write to the notebook. Return the report to Codex.
