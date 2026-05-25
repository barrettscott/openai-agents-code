# Handoff Note — OpenAI Agents Course

## Current Session State

**Last updated:** May 25, 2026

### What happened

Calibration pass complete across all 32 oa notebooks. This was a further pass on top of the May 11, 2026 prose review — applying the conventions established during the ca course calibration:

1. **WTW → Key Takeaway rename** across NB07–NB32 (except where "Why This Matters" / "Why This Breaks" / "The Principle" variants were intentionally preserved per design guidelines)
2. **Single-sentence punchlines** in KT cells where the two-sentence form was redundant
3. **Section intros trimmed** from 3–5 paragraphs to 1–3 where multi-paragraph framing was throat-clearing or restated the next paragraph
4. **Structural cleanups** found during the pass:
   - NB18: removed 2 duplicate Phase intro cells (Cell 10 = bare H2 dup of Cell 9; Cell 13 = near-dup of Cell 12)
   - NB25: removed duplicated Part 3 intro (Cells 22/25), removed misplaced KT (Cell 34), added missing Exercise 2 markdown intro
   - NB26: normalized KT bullet spacing to match course-wide blank-line-between-bullets pattern
5. **2-paragraph security notes** combined into single callouts where the second paragraph elaborated rather than added a distinct point

### Status of notebooks

- All 32 notebooks: calibrated (post-May-25 standard)
- All WTW cells renamed to Key Takeaway (where appropriate variant)
- Per-notebook commit granularity preserved — easy rollback
- Git repo: github.com/barrettscott/openai-agents-code
- Branch: `master`

### What is in progress

Nothing.

### What is next

- No further prose review work planned for oa
- One spawned task pending (Scott can dismiss or run): NB29 Exercise 1 code cell has duplicate Option A/B — both point to `@modelcontextprotocol/server-memory`. Replace one with a different MCP server (Git, SQLite, etc.) for genuine three-option exercise. Out of scope for prose pass since it touches a code cell.

### Last commit

`984ddef NB32: rename 4 WTW → KT, trim 4 section intros + deployment checklist`

### Important rules

- Always use `nbformat` to read and write notebooks
- Never batch-process — one file at a time
- Show proposed changes and get approval before writing
- Never commit `.env`
- Push to `origin master` (not main)
- Prepend `rm .git/index.lock 2>/dev/null; true` to every git command

---

## Prose Review Workflow (for reference)

If returning to make further prose changes, follow this order:

1. Read `design_guidelines.md` — especially Examples 1–12 (the few-shot section)
2. Read the target notebook's markdown cells in full
3. Produce a proposed changes list — one entry per cell, showing before/after. **For each cell, decide what content survives first (the trim), then apply mechanical formatting (one sentence per line, em-dashes, blank lines) only to what survives.** Lead each proposal with the cut, not the splits. Hard ceilings: Key Takeaway ≤ 2 sentences; Part intros ≤ 3 short lines; decision-guide cells ≤ 3 bullets.
4. Get approval before writing
5. Use `nbformat` to read/write — never `json.load`/`json.dump`
6. Use the safe_write pattern (write to `.tmp`, then `shutil.move`)
7. Verify the changes in the raw JSON after writing
8. Commit and push to `origin master`

**Do not touch code cells.**

**What earns a 3rd line in Key Takeaway** (override the 2-sentence ceiling): concrete numbers ("2–3× median latency"), critical safety reminders for high-blast-radius tools (Bash, write tools, `bypassPermissions`), the "when to reach for this" use-case sentence when the mechanism alone doesn't tell the student when to apply it.

**Preserved variants** (not all `### 💡` cells become Key Takeaway):
- `### 💡 Why This Matters` — keep when the lesson is structural rather than mechanical (e.g., NB23 prompt injection cells)
- `### 💡 Why This Breaks` — keep for failure-mode demos
- `### 💡 The Principle` — keep for design-rule explainers
- `### 💡 When to Use Each` — keep for decision tables
