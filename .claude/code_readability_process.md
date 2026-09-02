# Shared Notebook Code-Readability Review Process v3.0.0

**Status:** Provisional while it is calibrated beyond CA07 and CA08

## Purpose

Review one notebook's executable teaching surface so a student can trace,
explain, rerun, and transfer the code without replacing standard Python or SDK
patterns with personal style.

This file contains the process shared by all courses. Apply the repository's
local `course_review_profile.md` for its SDK, kernel, outline, branch, protected
structure, and deterministic checks.

Follow the authority order in the repository README. Stop and surface a
conflict instead of silently overriding a higher-priority instruction.

CA07 and CA08 showed that the highest-value findings come from first-use and
student-path checks. They also showed that architecture questions become
expensive when resolved through chains of fully built candidates. Version 3
makes those checks primary and settles design before implementation.

## Governing principles

1. **Presume the baseline can ship.** A change needs a concrete comprehension,
   execution, transfer, rerun, camera, or local cost-disclosure benefit.
2. **Metrics locate; they do not decide.** Width, nesting, line count, and name
   length cannot justify a change alone.
3. **Preserve authentic conventions.** Explain unfamiliar Python, SDK, and
   course patterns at first use instead of rewriting them into personal style.
4. **Use the cheapest effective instrument.** Prefer prose or a short comment
   when code is correct.
5. **Judge the student path.** Review what students run, rerun, copy, and infer,
   not only the candidate diff.
6. **Classify each surface independently.** A notebook can contain narrow
   clarifications and one design question. Only the design question enters the
   design gate.
7. **The outline is a scope map, not a veto.** If better teaching changes scope,
   propose the notebook and outline amendments together for Scott's approval.
8. **Ignore sunk cost.** Built, rendered, or previously approved changes still
   revert when they do not earn their place.
9. **Review the artifact that will ship.** A verdict applies only to the exact
   candidate a reviewer examined. Later edits never inherit an earlier ruling.

## Change classes

- `KEEP`: the baseline is already clear and transferable
- `EXPLAIN`: correct standard code needs adjacent prose or a short comment
- `CLARIFY`: a behavior-preserving local code improvement
- `RESTRUCTURE`: responsibility, teaching container, or execution order moves
- `BEHAVIOR FIX`: behavior, state, output, or a teaching claim is wrong

`RENAME` is not a class. A rename must qualify under another class.

## Stage 0: Freeze and map

Before editorial work:

1. Pin the live SHA-256, frozen baseline, and fixed comparison commit. Reconcile
   the worktree blob with that commit by content hash.
2. Keep the live notebook unchanged. Work in `/private/tmp`.
3. Read every markdown and code cell in order. Inspect outputs, execution
   counts, metadata, kernelspec, adjacent markdown, and the continuous
   presentation-scale render.
4. Map definitions, reads, cross-cell dependencies, side effects, mutable
   state, paid calls, stochastic paths, and rerun assumptions.
5. Verify deterministic output claims offline, including random-state
   initialization and consumption order.
6. Extract affected exercise instructions and TODOs. Where practical, write
   exactly what the student is asked to write and execute it offline.
7. For each non-obvious SDK or Python construct central to the notebook, find
   its earliest course use and determine whether this notebook must introduce
   it. Do not census ordinary local variables.
8. If a proposal changes an SDK, Python, output-state, or course convention,
   count that construct across the course once and report the blast radius.
9. Put paid-call disclosures beside the cells that spend them. A missing
   disclosure is normally prose, not a code rewrite.
10. Compare the full pin set with the previous round. If every input is
    byte-identical, stop instead of repeating the review.

These are verified facts and locators, not findings.

## Stage 1: Codex editorial review

Codex reviews the entire notebook before building a candidate or writing any
reviewer prompt.

### Student path and state

- Trace inputs, transformations, side effects, and outputs in execution order.
- Test a fresh kernel, section rerun, and immediate out-of-order paths invited
  by the notebook.
- Determine whether failures stop, return values, retain partial state, or look
  successful. Verify that later cells consume only valid state.

### Prose and first use

- Read the markdown immediately before and after each code cell.
- Explain correct non-obvious constructs at their first course use.
- Do not restructure authentic code merely to avoid teaching a concept the
  lesson needs.

### Names and shape

Classify names as framework, Python, course, or domain conventions.

- Preserve framework names exactly as the local profile's SDK defines them,
  including handler parameters and callback signatures.
- Preserve short bindings used once in an obvious expression when the
  collection supplies the type.
- Consider a domain name when an object crosses lines, exposes multiple fields,
  survives beyond one expression, competes with another type name, or forces
  the reader to reconstruct its type.
- A rename must describe the value. Longer is not automatically clearer.
- Reformatting a working one-line expression requires teaching justification
  independent of any rename.

### Framework and data boundaries

- Verify non-obvious SDK fields and response shapes against the installed SDK
  or authoritative reference where practical.
- Preserve public shapes students will meet in real SDK work.
- Distinguish authoritative output from intermediate messages, requests,
  partial state, and errors.
- Never call a mocked, skipped, or synthetic path live-verified.

### Proportionality and camera

- Add no helper, wrapper, class, or abstraction unless it removes more
  explanation than it introduces.
- Separate deliberate simulation from production behavior.
- Judge whether the instructor can point through the code in execution order.
- Expand dense code only when the expansion clarifies the lesson or gives
  sibling checks a useful shared scan path.

### New teaching claims

Every new label or claim must name something demonstrated here or established
earlier. Cut unsupported words such as `boundary`, `testable`, `reusable`,
`robust`, or `production` when the notebook does not demonstrate that property.

### Required evidence

For each proposal record the learner problem, complete before and after
surface, chosen instrument, convention classification, behavior and state
effects, new concepts, paid-call effect, deterministic verification, and the
strongest case for shipping the baseline. Reject the proposal when that defense
wins.

## Architecture design gate

Use this gate before building a candidate when a proposal changes responsibility
placement, SDK exposure, helper or class structure, cell identity or state
ownership, a multi-cell result policy, or outline scope. A small local bug fix
does not trigger it merely because behavior changes.

Show:

1. the demonstrated problem and passing constraints;
2. the baseline plus at most two viable designs;
3. concepts added or removed by each;
4. affected cells, prose, exercises, state, and paid calls;
5. one compact sketch per design; and
6. the strongest case for the baseline.

Both reviewers receive the same gate and rule independently from their normal
lanes. A rejection must state the complete constraints a passing design must
satisfy even when it offers no code. Scott selects the design. Codex then builds
one candidate.

## Stage 2: One exact candidate

Build the temporary candidate with `nbformat`. The live notebook stays frozen.
Verify:

- notebook validation and parsing of every code cell;
- exact changed cells, cell identities, metadata, outputs, and unrelated text;
- cross-cell definitions, state, cleanup, and preserved `<br>` pairs;
- normal, error, rerun, and invited out-of-order paths affected by the change;
- exact affected exercise TODOs where practical;
- call counts without paid or stochastic execution unless Scott authorizes it;
- every new teaching claim's evidence owner; and
- the complete presentation-scale render.

## Stage 3: Independent candidate audit

Codex is always one independent perspective. Route external review by risk.

**Narrow route:** use one reviewer when every change is `EXPLAIN` or local
`CLARIFY` and none touches SDK meaning, behavior, state, paid calls, exercises,
cell identity, or architecture. Choose the lane matching the primary risk.

**Full route:** use both reviewers when any change touches behavior, state, SDK
meaning, paid or stochastic calls, an exercise contract, dependent cells, or
architecture.

For the full route, freeze both prompts together before distributing either.
For the narrow route, freeze the selected prompt before distribution. No
reviewer sees Codex's rationale or another reviewer's report.

- **Reviewer 1:** student comprehension, source-order teaching, first use,
  narration burden, render, and exercise transfer.
- **Reviewer 2:** SDK and Python contracts, state, reruns, output semantics,
  paid calls, and abstraction cost.

Each reviewer must answer for every changed surface:

> Would the baseline have been acceptable to ship unchanged?

Use one unambiguous ruling:

- `KEEP CANDIDATE: DEMONSTRATED BENEFIT`
- `RESTORE BASELINE: NO DEMONSTRATED BENEFIT`
- `AMEND CANDIDATE`

Without a concrete benefit, the ruling is
`RESTORE BASELINE: NO DEMONSTRATED BENEFIT`.

## Stage 4: Reconciliation

Treat every finding as a hypothesis. Agreement, shorter code, corpus
consistency, and isolated probes are evidence, not verdicts.

For each item, defend the baseline, trace the full student path, and rule
`ACCEPT`, `CHALLENGE`, or `REJECT`. Reject formulaic changes and catches that
hide an error by relabeling it.

Run the deletion test on every proposed change:

> What concrete teaching, execution, transfer, state, cost, or camera benefit
> disappears if this change is dropped?

Consistency, polish, corpus frequency, and work already spent are not answers.
If no demonstrated benefit disappears, restore the baseline.

If an architecture question appears, discard that surface's candidate and use
the design gate. Do not patch successive candidates while design is unsettled.

Send both reviewers one identical focused challenge only when evidence
conflicts, reconciliation introduces a third implementation, a behavior or SDK
policy remains disputed, or the proposed cure may create a larger teaching
cost. Keeping the baseline or accepting an already reviewed mechanical change
does not trigger a challenge.

## Stage 5: Approval, application, and verification

Show Scott the complete surfaces, exact scope, behavior and state effects,
paid-call consequences, verification, and unresolved tradeoffs. Wait for `go`.
Code cells require explicit per-item approval, and `go` applies only to the
exact candidate shown.

Before requesting `go`, diff the proposed final candidate against the most
recent candidate each reviewer actually examined. List every later surface as
`UNREVIEWED`. An unreviewed surface cannot inherit an earlier verdict: route it
through the required review lane or remove it from the candidate before
approval.

After approval:

1. re-hash and stop on unpermitted drift;
2. apply only approved changes with `nbformat`;
3. verify the live delta independently from the build script;
4. rerun changed deterministic paths;
5. run paid or stochastic paths only with explicit approval; and
6. render, validate the capture, inspect, and re-hash the applied notebook.

Capture validation must record image height, last content row, and the largest
interior blank band. Reject the render as evidence when a loading indicator is
visible, when an unexplained blank band exceeds roughly 200 pixels, or when an
uncropped blank tail exceeds roughly 150 pixels. Intentional spacing must be
identified rather than silently exempted.

Use a post-apply adversarial review only for `RESTRUCTURE`, `BEHAVIOR FIX`,
state or cell-identity changes, paid-path changes, or a new multi-cell contract.
Narrow `EXPLAIN` and `CLARIFY` changes need exact verification and final render,
not another review round.

Commit or push only when Scott explicitly requests it. Stage only the reviewed
notebook and approved companion files.

## Process maintenance

After each notebook record the number of review rounds, candidate forms,
changed cells, and accepted findings by change class. Also answer: **What did
Scott identify that this process should have found?** Improve the process only
for repeated or material misses, normally between notebooks. A new mandatory
check must replace an overlapping check or explain why none can be removed.

Maintain a targeted first-use and convention map. Update it only when a
notebook introduces or disputes a construct. It is evidence, not a batch queue.

## Calibration from CA07 and CA08

- Preserve `args` as a framework convention and one-use `r` or `e` bindings in
  obvious comprehensions.
- Use a domain name when one object exposes several fields, as with `product`.
- Remove an unfamiliar SDK construct when the lesson does not need it (CA07
  removed a Claude Agent SDK handler-field access). Otherwise explain it at
  first use.
- Expand a dense expression only for an independent teaching reason, as with
  CA08's one structurally different pass/fail check.
- Use deterministic message streams to test output boundaries. CA08 proved
  that grading intermediate narration caused false failures.
- Preserve loud run errors and guard downstream paid work from missing state.

## Definition of done

The notebook is complete when every cell and final render have been inspected,
first-use and student-path checks are resolved, accepted changes defeat their
baseline defenses, the live file matches the approved candidate, deterministic
verification passes, and the required review route has no unresolved finding.
Scott alone authorizes commit or push.
