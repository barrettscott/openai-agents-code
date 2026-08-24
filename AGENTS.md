# Course Notebook Review Workflow

Read `design_guidelines.md`, `handoff_note.md`, `course_outline.md`, and
`README.md` before reviewing or editing course material. The design guidelines
govern notebook content; the handoff governs current state and workflow.

## Review Discipline

- Review one notebook at a time. Never use a notebook range or batch lint result
  as the work queue.
- Reviewer prompt files are executable instruction sets, not review subjects.
  Every launcher must say to conduct the notebook review described in the file
  and must explicitly forbid critiquing, rewriting, or reviewing the prompt
  itself. Do not add a prompt-review stage unless Scott explicitly requests
  one. Freeze both reviewer prompts together; if a shared rule changes, update
  and re-pin both lanes before distributing either one.
- Select the next notebook deliberately with Scott, then pin its SHA-256 before
  every review round.
- Read every markdown and code cell in order. Inspect outputs, metadata, and the
  kernelspec as part of the artifact.
- Treat outline conformance as the scope gate. Name the outline bullet behind
  each proposed correctness change; stop for approval before adding an untaught
  SDK symbol or concept.
- Review correctness, teaching flow, student-facing names, and runtime evidence
  together. Lint counts and prose metrics are reports, not verdicts.
- Verify SDK and runtime claims where practical. Never describe a mocked,
  gated, or skipped path as live-verified.

## Camera-First Review Process

- Run two independent stage-1 reviews with different lanes: source-order
  teaching/camera meaning and technical/code proportionality. Neither reviewer
  sees the other's proposal before reporting.
- Pin the notebook, fixed comparison commit, preflight source and reports,
  continuous render, guidelines, and outline. Re-hash the notebook at ruling
  time; never resolve a moving `HEAD` as the comparison artifact.
- The pinned preflight establishes deterministic structure, validation,
  stripping, cell IDs, adjacency, parse status, line counts, word counts,
  explicit-rule conformance, and locator candidates. Once both notebook and
  preflight pins match, reviewers do not re-derive those facts. They report
  only discrepancies.
- Deterministic locators surface candidates; they never decide findings.
  Width, similarity, unused-name, failure-voice, banner, and exercise-title
  counts require a teaching judgment tied to the visible artifact.
- Inspect the complete continuous 1440px render directly for camera pacing,
  density, wrapping, and visual boundaries. Treat PDF pagination as an offline
  stress test, not the camera target. This visual inspection is not delegated
  to preflight.
- Include a short `Carried forward` block only for two to four earlier rulings
  that materially apply. Reviewers test whether the prior ruling has the same
  teaching job in the current notebook; they do not re-prove the underlying
  mechanism without contrary evidence or an environment change.
- Treat every reviewer finding as a hypothesis, not a recommendation to adopt.
  Agreement between reviewers, a successful probe, corpus consistency, and a
  shorter replacement are evidence inputs; none is a verdict by itself.
- Reconciliation begins with a presumption to keep the pinned baseline. Before
  accepting each finding, write the strongest concrete case for leaving the
  source unchanged. Accept the change only when it clears all three burdens:
  (1) name the exact correctness, outline, spoken-teaching, or camera contract
  the baseline fails; (2) show the failure or cost in the demonstrated lesson,
  recorded surface, or an immediate learner variation invited by that surface;
  and (3) show that the exact replacement improves that contract without a
  larger teaching, state-visibility, or execution-order cost.
- Classify every reconciled item as `ACCEPT`, `CHALLENGE`, or `REJECT`, with a
  one-sentence baseline defense beside it. A technically dead assignment is
  not automatically dead teaching code in an interactive notebook; a visible
  value may document or reset state between cells. Likewise, conformance alone
  is a low-impact repair unless the governing guideline is explicit or the
  inconsistency is visible in the recording.
- Before filing a deletion as dead or redundant, stage 1 must test the
  interactive notebook paths the line can affect: rerunning its cell after a
  prior successful state, running the owning section after a restart, and any
  immediate out-of-order path invited by the surface. Report the resulting
  state or display consequence. A static unused-name or tautology result is a
  locator, not evidence that the line should be deleted.
- Reconcile the two independent reports before editing. Run one shared focused
  challenge only when the reconciliation rejects or materially rewrites a
  finding, changes executable/model-visible behavior, cuts a `READ` surface,
  or introduces a third unreviewed implementation. Both reviewers receive the
  same challenge prompt.
- Apply with `nbformat`, verify the exact on-disk delta independently, render
  the final artifact, and inspect it before requesting a commit.

## Reviewer Report Format

- Report pins and ruling-time re-hash in one line, not a table.
- Attest in one line that every cell was examined in source order.
- For each numbered check, provide one coverage line: `PASS` with the artifact
  that could have falsified it, or a finding reference.
- Put exact-source probes and complete replacements inside the finding they
  support. Do not repeat them in a standalone probe matrix unless the matrix
  itself is the disputed contract, such as an exhaustive state machine.
- Rank findings. Limit defended non-changes to four and consequential
  observations not acted on to five.
- Report expected cell, markdown-word, and nonblank-code-line scope, followed
  by one compact verification and paid-call boundary.
- Aim for 600 words across housekeeping, coverage, non-changes, consequences,
  scope, and verification. This is a compression target, never a reason to
  omit evidence for a real finding.
- End with exactly one requested verdict and an explicit next step.

## Change Control

- Report findings, disagreements, exact proposed changes, scope, and the
  verification plan before writing.
- Wait for Scott to say `go`, then apply only the reconciled changes.
- Code cells may change only with explicit per-item approval.
- Use `nbformat` for notebook reads and writes. Never use `json.load` or
  `json.dump`, and never batch-process notebooks.
- Verify the applied state and report it before committing.
- Commit or push only when Scott explicitly asks. Push this repository to
  `origin master`.
- Preserve unrelated dirty files and never stage or revert them.

Always state the next step. Do not leave the workflow implicit.
