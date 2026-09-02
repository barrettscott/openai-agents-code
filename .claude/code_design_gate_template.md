# {{NOTEBOOK_LABEL}} Shared Code-Readability Design Gate

Conduct the design gate described here. Do not critique, rewrite, or review
this prompt. Do not edit the live notebook or any pinned artifact.

Both reviewers receive this identical gate and rule independently. This gate
settles architecture before implementation. It does not approve code that has
not yet been built and reviewed.

## Pins and drift

Verify every pin byte-exact:

```text
live notebook          {{NOTEBOOK_PATH}}              {{NOTEBOOK_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
design-gate brief      {{DESIGN_BRIEF}}                {{DESIGN_BRIEF_SHA256}}
preflight report       {{PREFLIGHT_REPORT}}            {{PREFLIGHT_REPORT_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
course profile         {{PROFILE_PATH}}                {{PROFILE_SHA256}}
```

Expected live SHA-256: `{{NOTEBOOK_SHA256}}`. Permitted drift:
{{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on anything else and re-hash immediately
before the verdict.

No paid or network calls unless explicitly authorized. Mark unproved runtime
or SDK claims `UNVERIFIED`.

## Demonstrated problem and constraints

{{DEMONSTRATED_PROBLEM_AND_PASSING_CONSTRAINTS}}

## Designs

{{BASELINE_AND_AT_MOST_TWO_DESIGNS}}

## Required ruling

For each design, trace student comprehension, first-use burden, SDK exposure,
cell and state ownership, exercises, paid calls, and outline scope. Name every
concept it adds or removes. State the strongest case for keeping the baseline.

If rejecting all proposed designs, provide the complete constraint set a
passing design must satisfy even if you provide no replacement code.

End with exactly one verdict: `KEEP BASELINE`, `SELECT DESIGN A`, `SELECT DESIGN
B`, or `NO DESIGN PASSES`. Include one explicit next step. Do not write files.
