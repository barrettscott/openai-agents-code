# Course Notebook Review Workflow

Read `design_guidelines.md`, `handoff_note.md`, `course_outline.md`, and
`README.md` before reviewing or editing course material. The design guidelines
govern notebook content; the handoff governs current state and workflow.

## Review Discipline

- Review one notebook at a time. Never use a notebook range or batch lint result
  as the work queue.
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
