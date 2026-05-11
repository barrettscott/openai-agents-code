# Handoff Note — OpenAI Agents Course

## Current Session State

**Last updated:** May 11, 2026

### What happened
A Claude session incorrectly used `json.dump` to save all 32 notebooks, corrupting the JSON structure. All notebooks were restored one-by-one via Dropbox version history by Scott.

### Status of notebooks
- All 32 notebooks restored via Dropbox — should be back to original state
- Prose review per design guidelines has NOT been correctly applied yet
- Git repo initialized at github.com/barrettscott/openai-agents-code

### What is next
- Prose review per design_guidelines.md — not yet started (previous attempt was incorrect)
- Use `nbformat` only — never `json.load` / `json.dump`
- Work one notebook at a time, show proposed changes, get explicit approval before writing
- Run `git add . && git commit -m "pre-edit snapshot"` before touching any file

### Last commit
`983c966 Initial commit — current state before any edits`

### Important rules
- Always use `nbformat` to read and write notebooks
- Never batch-process — one file at a time
- Show proposed changes and get approval before writing
- Never commit `.env`
