# {{NOTEBOOK_LABEL}} Exact-Candidate Code-Readability Audit

Conduct the exact-candidate audit described here. Do not critique, rewrite, or
review this prompt. Do not edit the live notebook, candidate, or pinned
artifacts.

Auditor lane: **{{AUDITOR_LANE}}**

## Pins and drift

Verify every pin byte-exact:

```text
live notebook          {{NOTEBOOK_PATH}}              {{NOTEBOOK_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
exact candidate        {{CANDIDATE_PATH}}             {{CANDIDATE_SHA256}}
editor disposition     {{EDITOR_REPORT}}              {{EDITOR_REPORT_SHA256}}
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
stochastic demo to obtain favorable evidence. Mark unproved claims `UNVERIFIED`.

## Audit order and independence

{{READ_ORDER_INSTRUCTION}}

Do not infer Codex's rationale. Do not see or rely on the other reviewer's
report. Inspect every markdown and code cell in source order and both complete
renders.

## Required audit

For every changed surface:

1. state the learner problem the candidate claims to solve;
2. trace inputs, transformations, state, failures, and outputs;
3. test fresh-kernel, section-rerun, and invited out-of-order behavior where
   relevant;
4. verify SDK and Python shapes against pinned evidence;
5. classify names as framework, Python, course, or domain conventions;
6. verify that any expansion, helper, guard, or abstraction earns its camera
   and explanation cost;
7. test affected exercise instructions as a student would where practical;
8. identify every new teaching claim and its evidence owner; and
9. answer: **Would the baseline have been acceptable to ship unchanged?**

Rule with exactly one of:

- `KEEP CANDIDATE: DEMONSTRATED BENEFIT`
- `RESTORE BASELINE: NO DEMONSTRATED BENEFIT`
- `AMEND CANDIDATE`

The work already spent, shorter code, consistency, and corpus frequency do not
demonstrate benefit. Give the strongest concrete case against your overall
verdict.

## Required report

- Pins, ruling-time re-hash, and stale/not stale in one line.
- One line attesting every cell and both renders were inspected.
- One compact ruling per changed surface.
- Exact source for every amendment.
- Student-path, state, exercise, outline, and paid-call verdicts.
- Expected cells, markdown words, and nonblank code lines.
- Exactly one overall verdict and an explicit next step.

Do not write files. Return the audit to Codex.
