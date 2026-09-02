# {{NOTEBOOK_LABEL}} Exact-Candidate Camera Audit

Audit the notebook candidate described here. Do not critique, rewrite, or review
this prompt. Do not edit the live notebook, candidate, or any pinned artifact.

Auditor lane: **{{AUDITOR_LANE}}**

## Pins and drift

Verify every pin byte-exact before auditing:

```text
live notebook          {{NOTEBOOK_PATH}}              {{NOTEBOOK_SHA256}}
frozen baseline        {{FROZEN_PATH}}                {{FROZEN_SHA256}}
exact candidate        {{CANDIDATE_PATH}}             {{CANDIDATE_SHA256}}
editor disposition     {{EDITOR_REPORT}}              {{EDITOR_REPORT_SHA256}}
ownership diff         {{OWNERSHIP_REPORT}}           {{OWNERSHIP_REPORT_SHA256}}
baseline render        {{BASELINE_RENDER}}            {{BASELINE_RENDER_SHA256}}
candidate render       {{CANDIDATE_RENDER}}           {{CANDIDATE_RENDER_SHA256}}
changed-run crops      {{CROP_MANIFEST}}              {{CROP_MANIFEST_SHA256}}
preflight report       {{PREFLIGHT_REPORT}}            {{PREFLIGHT_REPORT_SHA256}}
design guidelines      {{GUIDELINES_PATH}}             {{GUIDELINES_SHA256}}
course outline         {{OUTLINE_PATH}}                {{OUTLINE_SHA256}}
```

Expected live notebook SHA-256: `{{NOTEBOOK_SHA256}}`.

Permitted drift: {{EXACT_PERMITTED_DRIFT_OR_NONE}}. Stop on any other source or
metadata drift. Re-hash the live notebook immediately before the verdict. The
pinned preflight establishes deterministic facts; report only a discrepancy.

No paid or network calls. Do not rerun a stochastic demo to obtain friendlier
evidence. Mark any runtime or SDK claim not proved by the pins as `UNVERIFIED`.

## Audit order and burden

{{READ_ORDER_INSTRUCTION}}

This is an audit of one exact editorial candidate, not a fresh full-notebook
rewrite. Treat each editor change as a hypothesis. Do not manufacture an
objection to justify the lane, and do not accept a change because it is shorter.

For every changed surface, rule:

- `KEEP CANDIDATE`: the exact candidate is stronger and preserves meaning;
- `REVERT`: restore the baseline, naming the lost claim, false statement,
  weaker scan path, ownership loss, or visible regression;
- `AMEND`: give complete exact source for a small necessary repair; or
- `ALTERNATIVE`: {{ALTERNATIVE_ALLOWANCE}}.

An evidence-backed `REVERT` wins by default in adjudication. Taste alone is not
evidence. Test the full adjacent-markdown run, not an isolated edited cell.

For each change ask: **Would the baseline have been acceptable to ship
unchanged?** If dropping the change loses no concrete teaching, correctness,
ownership, transfer, or camera benefit, rule `REVERT`.

## Required audit

1. Read every cell in source order and inspect both complete 1440px renders.
2. Audit every changed full run from the crop manifest. Three spoken beats is
   the goal; four or more must earn itself. Spec lists are exempt when their
   one-to-one mapping is the teaching value.
3. Read labels as the instructor would. Their label-only path must carry the
   claim or polarity. Code identifiers that map directly to adjacent code are
   valid anchors.
4. Check the ownership report for every deleted phrase, identifier, concept,
   caveat, and safety rule. A recap, code cell, output, or exercise is not a
   sufficient first spoken home.
5. Scan the unchanged candidate render once for a missed high-impact camera
   surface. Report at most two, and only when a student would perceive the
   density, hierarchy, scope, or accuracy problem.
6. Verify outline scope before endorsing a new SDK symbol, mechanism, or claim.
7. Preserve existing double `<br>` pairs. Apply the markdown ban on semicolons
   and em dashes from the highest-priority Teaching Clarity rule.

## Required report

- Pins, ruling-time re-hash, and stale/not stale in one line.
- One line attesting every cell and both complete renders were inspected.
- One compact ruling per changed surface, in candidate priority order.
- Any missed high-impact surface, at most two.
- Ownership verdict and outline verdict.
- Exact final source for every `AMEND` or permitted `ALTERNATIVE`.
- Expected cell, markdown-word, and nonblank-code-line scope.
- One verification/no-paid-call paragraph.
- Exactly one overall verdict and an explicit next step.

Do not write to the notebook. Return the audit to Codex.
