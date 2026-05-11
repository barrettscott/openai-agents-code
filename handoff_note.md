# Handoff Note — OpenAI Agents Course

## Current Session State

**Last updated:** May 11, 2026

### What happened
Camera-ready prose review completed across all 32 notebooks (NB01–NB32). Three passes were run:

1. **First pass** — CC reviewed NB04–NB32 sequentially, applying one-sentence-per-line formatting, em-dash conversions, and content cuts per the design guidelines.
2. **WTW correction pass** — Over-ceiling Why-This-Works cells (3+ sentences) were identified by diff review and cut to ≤2 sentences via `cc_corrections.md`.
3. **Setup notebook pass** — NB01–NB03 (Environment Check, OpenAI Setup, How Agents Work) reviewed and trimmed with the same standard.

### Status of notebooks
- All 32 notebooks: prose review complete
- All WTW cells: verified at ≤2 sentences
- Git repo: github.com/barrettscott/openai-agents-code
- Branch: `master`

### What is next
- No further prose review work planned
- `openai-agents` course set aside — focus is on `claude-agents`

### Last commit
`6154c13 prose review: NB03 — 6 cells trimmed`

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
3. Produce a proposed changes list — one entry per cell, showing before/after. **For each cell, decide what content survives first (the trim), then apply mechanical formatting (one sentence per line, em-dashes, blank lines) only to what survives.** Lead each proposal with the cut, not the splits. Hard ceilings: Why This Works ≤ 2 sentences; Part intros ≤ 3 short lines; decision-guide cells ≤ 3 bullets.
4. Get approval before writing
5. Use `nbformat` to read/write — never `json.load`/`json.dump`
6. Use the safe_write pattern (write to `.tmp`, then `shutil.move`)
7. Verify the changes in the raw JSON after writing
8. Commit and push to `origin master`

**Do not touch code cells.**

**What earns a 3rd line in WTW** (override the 2-sentence ceiling): concrete numbers ("2–3× median latency"), critical safety reminders for high-blast-radius tools (Bash, write tools, `bypassPermissions`), the "when to reach for this" use-case sentence when the mechanism alone doesn't tell the student when to apply it.
