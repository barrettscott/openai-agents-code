# OpenAI Agents Course Review Profile

**Profile ID:** `oa`

This profile supplements both shared review processes. It contains only
OA-specific authority, environment, and release details.

## Repository

- Root: resolve the course repository root at runtime. Exact absolute paths
  belong in the generated pinned prompt.
- Remote repository name: `openai-agents-code`
- Push target: `origin master`
- Notebook kernel: `openai-agents`
- Preferred verification Python: `/opt/anaconda3/envs/openai-agents/bin/python`

## Governing documents

Read before reviewing or editing course material:

- `AGENTS.md`
- `design_guidelines.md`
- `handoff_note.md`
- `course_outline.md`
- `README.md`

The outline is a scope map, not an absolute veto. If better teaching changes
scope, propose the notebook and outline changes together for Scott's approval.

## Course-specific teaching and SDK boundary

- Verify OpenAI Agents SDK claims against the installed OA environment or
  authoritative OpenAI documentation where practical.
- Preserve authentic public SDK patterns such as `Agent`, `Runner.run`,
  `@function_tool`, handoffs, sessions, guardrails, and MCP configuration.
- Never describe a mocked, gated, skipped, or synthetic path as live-verified.
- Model IDs are reproducibility pins, not claims about the current model
  catalog.
- Name the relevant outline bullet before introducing an SDK concept not
  already owned by the lesson.

## Rendering and protected structure

- Default continuous-render width: 1440 pixels unless the recording viewport
  for the notebook is explicitly documented differently.
- Judge adjacent markdown as one surface and crop the complete run.
- Preserve existing double-`<br>` pairs unless Scott approves the exact change.
- Preserve outputs, execution counts, cell IDs, and metadata according to the
  pinned baseline and repository strip policy.
- Do not change code during a prose pass without explicit per-item approval.

## Deterministic checks

- Validate with `nbformat` and parse every code cell.
- Verify the precise source and metadata delta.
- Check lesson cross-references against `course_outline.md`.
- Verify cleanup names and cross-cell bindings for the notebook under review.
- Inspect paid-call placement and disclose cost beside the cells that incur it.
- Do not use paid or network calls during a prose pass unless Scott explicitly
  authorizes them.

## Local snapshot inventory

OA should vendor all twelve canonical snapshots under `.claude/`:

- `.claude/prose_review_process.md`
- `.claude/code_readability_process.md`
- `.claude/course_review_profile.md`
- `.claude/review_template_placeholders.md`
- `.claude/camera_editor_candidate_template.md`
- `.claude/camera_stage1_prompt_template.md`
- `.claude/camera_challenge_prompt_template.md`
- `.claude/camera_post_apply_prompt_template.md`
- `.claude/code_design_gate_template.md`
- `.claude/code_stage1_prompt_template.md`
- `.claude/code_challenge_prompt_template.md`
- `.claude/code_post_apply_prompt_template.md`
