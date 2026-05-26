# Handoff Note — OpenAI Agents Course

## Current Session State

**Last updated:** 2026-05-25 (renumber session)

### What is in progress

Nothing. Course is renumbered and calibrated.

### Latest milestone: course renumber

Both ca and oa courses renumbered. Content notebooks now occupy NB01–30 (clean sequential, no setup gaps). Setup notebooks moved to `Z_Setup/`:

- `Z_Setup/Setup_1_Environment_Check.ipynb`
- `Z_Setup/Setup_2_OpenAI.ipynb`

The renumber supports the watch-first / two-pass course model: content is what students watch first; setup is reframed as the pass-two onboarding to actually run the code.

**Last oa commit:** `c25ca85 design_guidelines: rename Lesson 01 → Setup 1 in Environment Check rules`

### Renumber mapping (old → new)

For oa:

- Old NB01 (Environment Check) → `Z_Setup/Setup_1_Environment_Check.ipynb`
- Old NB02 (OpenAI Setup) → `Z_Setup/Setup_2_OpenAI.ipynb`
- Old NB03 through NB32 → NB01 through NB30 (shifted down by 2)

For ca: same shift but ca had a `16A_Async_Python.ipynb` interlude (now NB14). See `~/Library/CloudStorage/Dropbox/Notebooks/claude-agents/handoff_note.md`.

When reading older commit messages, design guideline references, or this handoff's historical section, apply the mapping above to interpret NB## references.

### Pre-renumber prose review state (historical — NB numbers are pre-renumber)

All prose review work complete. Calibration pass (May 24–25, 2026) applied:

1. **WTW → Key Takeaway rename** across (old) NB07–NB32 (now NB05–NB30), except preserved variants (Why This Matters, Why This Breaks, The Principle, When to Use Each)
2. **Single-sentence punchlines** in KT cells where the two-sentence form was redundant
3. **Section intros trimmed** from 3–5 paragraphs to 1–3 where multi-paragraph framing was throat-clearing
4. **Structural cleanups** (pre-renumber NB numbers):
   - NB18 (now NB16, Capstone 2 Research Team): removed 2 duplicate Phase intro cells
   - NB25 (now NB23, Tracing): removed duplicated Part 3 intro, removed misplaced KT, added missing Exercise 2 intro
   - NB26 (now NB24, Capstone 3 Customer Service): normalized KT bullet spacing
5. **2-paragraph security notes** combined into single callouts

### What is next

- Watch-first course restructure: move setup section to END of Udemy curriculum
- Add "How to take this course" lecture introducing the two-pass model
- README "What's Next" section may need a rewrite tied to that restructure
- **One pending spawned task:** NB27 (was old NB29 — Capstone 4 MCP Assistant) Exercise 1 code cell has duplicate Option A/B (both point to `@modelcontextprotocol/server-memory`). Replace one with a different MCP server (Git, SQLite, etc.) for a genuine three-option exercise. Code cell touch required.

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
- `### 💡 Why This Matters` — keep when the lesson is structural rather than mechanical (e.g., NB21 prompt injection cells — pre-renumber NB23)
- `### 💡 Why This Breaks` — keep for failure-mode demos
- `### 💡 The Principle` — keep for design-rule explainers
- `### 💡 When to Use Each` — keep for decision tables
