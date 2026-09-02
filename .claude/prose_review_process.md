# Shared Notebook Prose and Camera Review Process v1.0.0

**Status:** Substance stable from prior course passes. Extracted wording is a
release candidate until one notebook completes this exact workflow.

## Purpose

Review one course notebook as a screen-recorded teaching artifact. Improve
correctness, spoken teaching, visual hierarchy, and camera pacing without
turning the notebook into a presenter script or manufacturing cosmetic churn.

This file contains the process shared by all courses. Each repository supplies
a local `prose_review_profile.md` for its kernel, SDK, outline, branch, protected
formatting, and deterministic checks. The local design guidelines govern exact
cell shapes and course language.

Follow the authority order in the repository README. Stop and surface a
conflict instead of silently overriding a higher-priority instruction.

## Governing principles

1. **Presume the baseline can ship.** A change must solve a demonstrated
   correctness, scope, spoken-teaching, ownership, or camera problem.
2. **Review the visible surface.** Consecutive markdown cells form one camera
   surface until code, output, or a visible divider separates them.
3. **Scope before hierarchy.** First ask whether every line serves the heading
   and whether the first spoken line begins answering it.
4. **Choose the container before the wording.** Decide whether the teaching job
   needs prose, anchored bullets, a diagram, a table, or a split or merge. Draft
   from the surviving claims instead of trimming the baseline in place.
5. **Three spoken beats is a goal, not a quota.** Four or more must earn their
   camera space. A short cell can still overload an adjacent markdown run.
6. **Metrics locate; they do not decide.** Beat counts, line width, word counts,
   labels, and lint results identify candidates. They do not prove a defect.
7. **Protect meaning and ownership.** Shorter prose is worse when it changes who
   acts, when a value exists, what a demo proves, or where a concept is first
   taught.
8. **Protect the label-only path.** Headings, labels, and anchors must carry the
   claim or polarity when read alone. A code identifier is a valid anchor when
   it maps directly to adjacent code.
9. **Require independent evidence.** Reviewer agreement is not a verdict. Two
   reviewers can share the same mistaken prior.
10. **Keep the live notebook frozen during review.** Every ruling applies only to
   an exact pinned artifact.
11. **Ignore sunk cost.** A built, rendered, or previously approved change still
    reverts when it does not defeat the baseline.

## Change classes

- `KEEP`: the baseline surface should ship unchanged
- `UPGRADE`: the baseline is acceptable, but a materially better teaching form
  is available
- `FIX`: the baseline fails a correctness, scope, ownership, or camera contract
- `MERGE/DROP`: the visible teaching container or cell boundary is wrong

An upgrade is not a finding. Optional upgrades are capped so they cannot crowd
out corrections or make the candidate impossible to audit.

## Stage 0: Select, freeze, and inspect

1. Select one notebook deliberately with Scott.
2. Read the repository's governing documents and local prose-review profile.
3. Pin the live notebook SHA-256 and a fixed comparison commit. Never use a
   moving branch name as the comparison artifact.
4. Freeze a baseline copy under `/private/tmp`. Do not edit the live notebook.
5. Generate established facts from the pinned artifact, never from memory:
   cells, types, IDs, markdown words, code lines, outputs, execution counts,
   metadata, kernelspec, parse status, headings, adjacent markdown runs, and
   protected formatting.
6. Read every markdown and code cell in source order. Inspect outputs and the
   complete continuous render at the presentation scale defined by the local
   profile.
7. Map every maximal adjacent-markdown run. Record its full span, headings,
   spoken beats, anchors, and full-run crop.
8. Re-hash the live notebook before every ruling. Stop on any drift not
   explicitly permitted by the local profile.

Preflight and lint output are evidence locators. Reviewers accept their pinned
facts and report only discrepancies.

## Stage 1: Editor-first review

Codex performs its own complete review before seeing either external report.

### Diagnose each presentation surface in this order

1. **Scope:** What job does the heading promise? Which lines do not serve it?
   Does the first spoken line begin the answer?
2. **Form:** Which container best fits the job? Apply the local camera shapes
   and choose the blunt register before polishing.
3. **Claims:** Which conditions, boundaries, causal links, examples, and
   consequences must survive?
4. **Replacement:** Trace every sentence through the actual cell sequence and
   visible evidence. Do not improve rhythm by making the lesson false.
5. **Measurement:** Compare spoken beats, anchors, longest spoken line, wraps,
   and mouthfuls across the full baseline and candidate run.
6. **Render:** Judge baseline and candidate crops at the actual presentation
   scale, then inspect each inside the continuous notebook render.

### Required from-scratch surfaces

Apply the from-scratch test to the opening/problem framing, every Part intro,
important pre-code bridges, post-demo explanations and takeaways, safety and
note surfaces, the final recap, every adjacent-markdown run, every surface over
six beats, and every surface with nested structure. A trigger requires review,
not a rewrite.

### Candidate limits

- Include every real `FIX`, regardless of count.
- Rank ordinary `UPGRADE` surfaces and include at most six.
- A recorded notebook is `FIX`-only unless Scott explicitly authorizes
  discretionary or structural work.
- For every `MERGE/DROP`, show the complete surviving source, remapped adjacent
  runs, and resulting cell count.
- Do not invent edits to make the pass look productive.

For every proposed change, record the full-run before and after source, teaching
job, chosen form, measurements, crop paths, deleted-claim ownership, and the
strongest concrete case for leaving the baseline unchanged.

## Stage 2: Build and freeze one exact candidate

Build one candidate under `/private/tmp` with `nbformat`. Verify the exact cell,
source, metadata, output, ID, and protected-formatting delta. Validate the
notebook, parse code cells, and run every course-specific deterministic check
named by the local profile.

The mechanical gate must also check banned presented punctuation, stripped
outputs and execution counts, and byte-identical code sources unless each code
change has separate explicit approval.

Render the complete candidate and each changed adjacent-markdown run. A crop of
an isolated cell is not evidence when another markdown cell is adjacent.

In the same work step, freeze:

- the candidate and editor disposition;
- the ownership report and mechanical verification;
- the baseline and candidate renders and crop manifest; and
- both coordinated initial reviewer prompts.

Record every SHA-256. Do not distribute one reviewer prompt before the other
prompt and the candidate are final.

## Stage 3: Independent exact-candidate audits

The initial prompts are coordinated but intentionally different. They do not
show Codex's rationale or the other reviewer's report.

- **Camera and meaning lane:** read the candidate first, then the baseline.
  Audit scope, spoken path, form, hierarchy, adjacent surfaces, render quality,
  and exercise or recap transfer.
- **Technical and proportionality lane:** read the baseline first, then the
  candidate. Audit factual accuracy, cell sequence, ownership, outline scope,
  SDK meaning, state implications, and whether each change earns its cost.

Both reviewers read every cell and both complete renders. Each changed surface
receives exactly one ruling:

- `KEEP CANDIDATE`
- `REVERT`
- `AMEND`
- `ALTERNATIVE`, only where the prompt explicitly permits it

Every reviewer must answer: **Would the baseline have been acceptable to ship
unchanged?** A reviewer must give the strongest concrete case against its own
overall verdict. The goal is independent judgment, not performative
disagreement.

## Stage 4: Reconcile with a candidate burden

Treat every reviewer statement as a hypothesis. For every item:

1. write the strongest concrete defense of the baseline;
2. identify the exact contract the baseline or candidate fails;
3. show that failure in the demonstrated lesson, recorded surface, or an
   immediate learner variation invited by it;
4. test the entire adjacent-markdown run and claim-ownership chain; and
5. show that the exact cure improves the contract without a larger teaching,
   accuracy, state-visibility, or camera cost.

Classify the result as `ACCEPT`, `CHALLENGE`, or `REJECT`.

An evidence-backed reviewer `REVERT` restores the baseline by default. Codex
may overrule it only with exact falsifying source, render, sequence, or runtime
evidence recorded beside the item. That overrule triggers the focused
challenge.

Run the deletion test on every candidate change:

> What concrete correctness, spoken-teaching, ownership, transfer, or camera
> benefit disappears if this change is dropped?

Shorter wording, guideline conformity, reviewer agreement, corpus frequency,
and work already spent are not sufficient answers.

If reconciliation introduces a form neither reviewer examined, label it
`THIRD IMPLEMENTATION`. It cannot inherit approval from a nearby proposal.

## Stage 5: Focused challenge when triggered

Use one identical focused challenge for both reviewers only when:

- Codex overrules an evidence-backed `REVERT`;
- both reviewers flag the same surface and neither cure is accepted;
- reconciliation creates a third implementation;
- a change alters executable or model-visible behavior or what a demo proves;
- a change introduces or changes an SDK claim; or
- ownership leaves a teaching concept only in code, output, an exercise, or
  the final recap.

Do not add a challenge merely because reconciliation rejects a finding or
accepts an already reviewed mechanical change.

Every challenge prompt includes this sentence verbatim:

> Rule independently from your earlier report. Your earlier finding is a
> hypothesis, not a position to defend.

Show the baseline, editor form, both reviewer forms, and third implementation
when one exists. Test the complete adjacent run at presentation scale.

## Stage 6: Approval and application

Before requesting `go`:

1. show Scott the exact final surfaces, scope, measurements, tradeoffs, and
   verification plan;
2. diff the final candidate against the most recent candidate each reviewer
   actually examined;
3. label every later surface `UNREVIEWED` and route it through the required
   lane or remove it; and
4. re-hash the live notebook and stop on unpermitted drift.

Scott's `go` applies only to the exact candidate shown. Apply it with
`nbformat`, then independently verify that the live notebook reproduces the
approved delta. Revalidate, reparse, rerun allowed deterministic checks, render
once, inspect the changed full-run crops, and record the final hash.

## Stage 7: Post-apply adversarial review

Run an exact post-apply adversarial review after a structural change, cell move
or merge, ownership relocation, new multi-cell teaching contract, executable or
model-visible change, or whenever Scott requests it. Both reviewers inspect the
finished notebook, not the proposal.

The post-apply task is to find regressions created by the union of changes and
remaining high-impact defects the candidate-scoped audit could not see. It is
not another polishing pass. Require reviewers to distinguish `REAL`,
`WORTHWHILE UPGRADE`, and `NIT`, and to identify what a student would
misunderstand or what would visibly fail.

If the post-apply review finds only nits, stop. If it finds a real issue, build
one focused candidate and repeat the relevant verification and approval steps.

## Prompt and report discipline

- Prompt files are executable instruction sets, not review subjects. Every
  launcher must tell the reviewer to conduct the review in the file and forbid
  critiquing, rewriting, or reviewing the prompt itself.
- Every prompt states the expected live SHA-256, fixed comparison commit,
  complete artifact pins, permitted drift, no-paid-call boundary, and exact
  requested verdict.
- Initial reviewer prompts differ by lane. Focused challenge prompts are
  identical. Post-apply prompts inspect the same artifact while retaining each
  reviewer's standing lane.
- Launchers are short copyable text blocks that point to the full prompt file.
- Generate established facts with deterministic probes. Never copy counts,
  metadata, source quotes, SDK facts, or corpus claims from memory.
- Mark unproved runtime or SDK statements `UNVERIFIED`. Never rerun a stochastic
  demo until it produces favorable evidence.
- Rank findings. Defend consequential non-changes. End with exactly one verdict
  and one explicit next step.
- Report full-notebook coverage once. Do not enumerate every clean surface.

## Definition of done

The prose pass is complete when every cell and the complete final render have
been inspected, every adjacent-markdown run has been considered as one surface,
all accepted changes defeat their baseline defenses, no unreviewed late delta
remains, the live file matches the approved candidate, the required
post-application checks pass, and no real adversarial finding remains.

Commit or push only when Scott explicitly requests it. Stage only the reviewed
notebook and approved companion files.
