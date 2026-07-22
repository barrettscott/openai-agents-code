# Handoff Note — OpenAI Agents Course

## Current Session State

**Last updated:** 2026-07-22 (teaching-flow / outline-conformance pass: OA11, OA24, OA22, OA18, OA28)

### Latest milestone: teaching-flow / outline-conformance pass (2026-07-22)

Multi-round Codex↔Claude review with **outline conformance as the gate**: every correctness fix named the outline bullet it repaired; any SDK symbol/concept not in a lesson's outline stopped for approval; show-before-write, with a one-time acceptance run per notebook (markdown-only edits skip the rerun). All pushed to `master` (`10c85c0`).

- **NB11 Capstone 1** (`3556535`, squashed) — rewritten from a three-tool "live web research" agent to a **controlled-fixture two-tool capstone**: File Search over two synthetic course-fixture docs (internal survey + cross-industry benchmark) + Code Interpreter, same `ResearchReport`; `WebSearchTool` removed. **Why:** the Responses-API **web-search-citation span-loss corruption** (server-side, confirmed 2026-07-20 by a raw-vs-parsed probe; garbled prose observed again on the old NB11 2026-07-22) made a web-search-dependent capstone unrecordable. Removing web search takes NB11 off that path; accepted run clean of span-loss (say "controlled-fixture," NOT "deterministic" — the model is still stochastic). Diagnostics stay OUT of the student notebook; the human content read is the gate. Memory: `project_nb11_web_corruption_todo`.
- **NB24 Capstone 3** (`b7f0967`) — removed the untaught `RunContextWrapper[CustomerContext]` typed-context-injection pattern (a new SDK primitive inside a capstone) → application-owned `DEMO_CUSTOMER_ID` constant, labeled a single-customer demo (not production auth; real apps bind identity per request via their auth layer, not the `SQLiteSession`). Every fail-closed invariant preserved (ownership, delivered-status, trusted price, dedup, approval + threshold, session-resume); accepted run demonstrated unauthorized-order rejection, approval/resume, one refund, ledger identity, session persistence, cleanup. Missed sibling caught by the run (cell 27 passed the removed `customer` var — NameError → fixed).
- **NB22 HITL** (`badb177`) — simplified `handle_refund_request` from six fail-closed branches to four: one compact malformed-args `try/except` instead of separate JSON/non-dict/amount-type/empty-reason branches; kept reject-unknown-tool, trusted-amount validation, handler dedup, threshold + approval, and reason visibility. Exercise 2's list aligned. Same safety, clearer HITL + threshold lesson.
- **NB18 Persistent Memory + NB28 outline** (`10c85c0`) — **kept** the curated-memory reshape (decision: better than the original, which duplicated NB17's file-backed `SQLiteSession`); added a precise short-term/long-term bridge to The Problem (cell 5): transcript = short-term working context, durable-but-uncurated; long-term = the facts you deliberately retain (NB18 rebuilds the *same* session, NOT a cross-conversation store). Reframed outline entry 18 around persistence-vs-curation (not in-memory-vs-SQLite). NB28: replaced the stale `client.responses.create()` outline bullet (that detour was already removed from the notebook) with "When a single-purpose no-tool agent is enough" (keeps Part 5 in the outline).
- **NB08 Web Search** — acceptance check passed; no span loss was observed in this run (no notebook change needed).

All code-cell edits above were **explicitly approved per-item** — the standing "don't touch code cells" rule was waived case-by-case, not ignored.

**Last oa commit:** `10c85c0` (pushed; `master` == `origin/master` before this handoff commit).

### Earlier milestone: nbstripout repo hygiene (2026-06-22)

**Added `nbstripout` git clean filter (commit `c842971`).** Committed notebooks now carry **no outputs**, no execution counts, and no `language_info.version`. This ends cross-machine version churn (oa notebooks had drifted across Python 3.11.13/14/15) and gives a clean course-as-code state (only 1 notebook had stray outputs). Setup: committed `.gitattributes` (`*.ipynb filter=nbstripout`), `filter.nbstripout.extrakeys = metadata.language_info.version`, `required = true`. **kernelspec left as-is** (the `claude-agents` normalization was ca-specific; oa keeps its own kernel). Each *committing* machine needs `pip install nbstripout`; the Studio Mac is record-only → no install needed. Mirrors the ca setup; see memory `reference_nbstripout_setup`.

**Last oa commit:** `c842971`

### Earlier milestone: first headless run-verification sweep (2026-06-22)

**oa had never been run end-to-end before this session.** Found the SDK env (`/opt/anaconda3/envs/openai-agents/`, py3.11) and registered a Jupyter kernel named `openai-agents` (reversible: `jupyter kernelspec remove openai-agents`). Ran all 30 notebooks headless via nbclient (`allow_errors=True`). This caught two **fully-broken** lessons that prose review had missed.

**Pushed to `master` (3 commits, `5f64832..3e7a0bf`):**
- **NB20 Guardrails** (`ab7363e`) — every guardrail was double-wrapped (decorated `@input_guardrail`/`@output_guardrail` re-wrapped in `InputGuardrail(...)`) → `AttributeError: 'InputGuardrail' object has no attribute '__name__'` at runtime. Whole lesson was dead. Fixed: decorated guardrails attach **directly** (`input_guardrails=[topic_guardrail]`); for `run_in_parallel` wrap the underlying fn: `InputGuardrail(topic_guardrail.guardrail_function, run_in_parallel=False)`. Also added the missing `blocking_agent` cell. Verified green.
- **NB24 Capstone 3** (`13dff6c`) — same double-wrap (c17) → fixed `input_guardrails=[support_topic_guardrail]`.
- **NB22 Human-in-the-Loop** (`13dff6c`) — `state.reject(message=…)` is wrong kwarg → it's `rejection_message=…` (fixed c19/c25/c41). NB27 had the same and was corrected too.
- **`.gitignore`** (`3e7a0bf`) — ignore `*.sqlite*` session/memory stores that NB17/NB18 generate when run.

**Resolved the two prior "NEEDS A WORKING SDK" flags:**
- NB20 — was the `blocking_agent` NameError *plus* the deeper double-wrap bug. Both fixed. `InputGuardrail` **does** accept `run_in_parallel` (confirmed by running).
- NB27 `require_approval=` on `MCPServerStdio` — **FALSE ALARM**. Valid kwarg in SDK 0.13.0. No change needed.

**Tool-use gap closed** (in `5f64832`, NB07 Testing): added tool-call checking — `tool_names = [item.raw_item.name for item in result.new_items if isinstance(item, ToolCallItem)]` (note: `ToolCallItem` has no `.name`; name is on `.raw_item`).

**Sweep verdict — all 30:**
- 3 real bugs (NB20/24/22) — fixed & verified above.
- NB13–17 timed out in the burst run but are **clean** — re-run fresh & sequential they finish in 68–370s with zero errors (burst rate-limiting, not bugs).
- Env-blocked here (NOT code bugs, will run on the recording machine): NB19 (`chromadb`), NB26/NB27 (`uvx`).
- Remaining 19 clean.
- Sweep artifacts: `/tmp/oa_sweep/` (`sweep.py`, `results.txt`, `timeouts.py`, `timeouts_results.txt`).

### Earlier this session: cross-reference audit + fixes (2026-06-18)

Part of a 13-agent parallel review of NB02–30 (both courses) + a systematic lesson cross-reference audit. Full detail: `/tmp/nb_review/SESSION_RESULT.md`.

**oa — commit `35b36fd` (9 notebooks, NOT pushed; branch ahead 1):**
- NB03 (multi-tool assistant ref → L02), NB07 (deleted 2 orphaned MARKDOWN cells holding unfenced Python that duplicated runnable cells 27 & 29), NB10 (KT "upload to container" → "Files API + file_ids"), NB11 (built-in tools "Lessons 10–12" → "08–10"; "auto" sandbox-reuse comment corrected), NB16/24/29 ("judge agent … Lesson 09" → L07 — recurring stale ref), NB23 (reflowed a broken query literal), NB30 (capstone refs → L24/L27).

**NEEDS A WORKING oa SDK (could not run headless here — `agents` not installed):**
- **NB20 Guardrails:** `blocking_agent` is used (cell 29) but never defined → `NameError`. Suggested cell (mirrors `cooking_agent`, adds `run_in_parallel=False`) is in SESSION_RESULT.md — but VERIFY `InputGuardrail` accepts `run_in_parallel` in the pinned SDK; the whole Part 5 narrative assumes it.
- **NB27 Capstone 4:** cells 11 & 13 pass `require_approval=` to `MCPServerStdio` — possibly an invalid kwarg → `TypeError` failing the capstone. Run those cells before recording.

Left UNCOMMITTED for Scott: NB04 intro reword (parity with ca; await wording approval).

### What is in progress

Nothing. Course is now **6 week-sections** (NB01–30), parity-ported from the ca session.

### Latest milestone: 6-section restructure + ca-parity port (2026-06-16)

Ported the ca session's structural + content work to oa. **Boundary-only re-section (no renumbering)** — NB01–30 keep their numbers/order, so "Lesson N" cross-refs are untouched; nothing referenced Weeks 4–6 by name, so zero in-notebook week-ref remap.

- **6-week restructure** (commit `22aae13`): old Week 4 (Memory/Safety/Observability, 8 nbs/1,749 code lines — the lone outlier) split into **W4 Stateful & Guarded Agents** (17–20: Sessions, Persistent, Vector, Guardrails) + **W5 Safety & Observability** (21–24: Injection, HITL, Tracing, Capstone 3); old W5 MCP → W6. Mirrors ca's "Stateful & Observable Agents" move (folded the SDK's control mechanism — Guardrails here, Hooks in ca — in with memory). Course curve: 615/1266/1105/671/1078/1210, no outlier. `zip-oa` globs `Week_*` → no change. Also removed 9 pre-existing tracked junk files (2 old-numbered `.ipynb_checkpoints`, 7 `.DS_Store`).
- **NB06 Error Handling trim** (commit `35e9700`): same call as ca NB07 — cut Part 4 (Fallback) + Part 5 (Runner-Level Recovery), `idempotent`→"safe-to-repeat", removed Exercise 2 (fallback), trimmed KT. 48→35 cells. oa-specific: Part 1 left intact (OpenAI SDK *catches* tool exceptions by default; oa disables with `failure_error_function=None` — ca's "crashes the run" framing doesn't apply); KT was already correct bullet format.
- **Parity ports** (commit `3a6a61b`): design_guidelines capstone numbers fixed (13/18/26/29 → 11/16/24/27); adjacent-cell headers added (NB11 `## 🧹 Cleanup`, NB15 `#### Define the Critique Loop`) — no adjacent code-cell pairs remain anywhere; removed fully-applied `cc_corrections.md`.
- **course_outline.md synced** (commit `a6f3176`): was stale from the renumber too (old L03–L32, tracing as L25, etc.) — full reconciliation to Z_Setup + NB01–30 + the 6 weeks; all refs verified.

Known follow-up (not done): oa NB21 (Prompt Injection) still uses "idempotent" — a candidate for the same "safe-to-repeat" swap as NB06, left for now (NB06 was the approved scope). README.md + TROUBLESHOOTING.md were checked and are already clean (no stale week/lesson refs, no fixtures). oa still **not run end-to-end** (different SDK).

### Parity tidies with ca (2026-06-14)

Carried over from the ca work this session: (1) MCP notebooks (NB25–27) — made `workspace` absolute and changed setup prints to show `Week_…/name` instead of the full resolved path; (2) NB07 `test_cases.json` — added a `_source` key naming the notebook and gitignored it (twin of ca's `golden_tests.json`); (3) `.gitignore` — runtime artifacts (`*workspace*/`, `*_sandbox/`, `*_demo/`, `report.md`) + `test_cases.json`; (4) `.zshrc` `zip-oa` now bundles `Z_Setup`. oa has **no** path-resolution bug (no bundled-CLI cwd; MCP servers use `str(workspace.resolve())`) and **no** fixtures dependency. **Not run end-to-end** (different SDK); only ca got the execution-verification pass.

### Earlier milestone: Key Takeaways sub-header sweep (2026-06-11)

Swept all content-notebook `## 🎯 Key Takeaways` cells (in parallel with ca) to enforce two design-guideline rules: sub-headers state a claim, not a topic; takeaways must be accurate, not absolute. ~39 headers converted across NB01, 03, 08, 09, 11, 12, 13, 18, 19, 20, 22, 24, 25, 26, 27. NB03 brought in line with ca's version (headers + softened "Always pair…" bullet). "Always X:" headers (07/09/14) checked and kept as demo-supported. Setup notebooks left alone. The design-guideline source of truth lives in the ca repo's `design_guidelines.md`.

Also ported the repo-root-anchoring `.env` automation into `Z_Setup/Setup_2_OpenAI.ipynb` (mirrors ca Setup_2). Branch: `master`.

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

- The 2026-07-22 rework changed content in NB11/18/22/24 — those cells now feed the camera prose pass below.
- Watch-first course restructure: move setup section to END of Udemy curriculum
- Add "How to take this course" lecture introducing the two-pass model
- README "What's Next" section may need a rewrite tied to that restructure
- ~~NB27 duplicate Option A/B fix~~ — done in `a62113c` (Option B is now Git server)
- **Camera prose pass, NB14–18 first** (measured 2026-07-10: 88 lint findings — NB14: 17 LONG + 2 FAT + 1 BANNED, NB15: 13 LONG + 2 MULTI, NB16: 7 LONG + 3 MULTI, NB17: 12 LONG + 5 MULTI, NB18: 17 LONG + 8 MULTI + 1 BANNED). Run the ca protocol: Camera Shapes section (now in this repo's design_guidelines) + camera lint (spec: ca memory `reference_camera_lint.md`). The ca NB15–19 tightened cells are templates for the twin cells here (oa 18 short-vs-long ≈ ca 19 Part 3; oa 18 What-NOT-to-Store ≈ ca 19 Part 5; same-name notebooks 14/15/16 ≈ ca 16/17/18) — adapt SDK details, keep the shape. Content verified separately: oa 16's "Phase 5" exercise and oa 18's noisy-memory/stale-cleanup beats are real here (no ca-style outline drift).

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

1. Read `design_guidelines.md` — Examples 1–12 (few-shot) AND the Camera Shapes section, especially "Register first: write blunt". Aim pass 1 at the BLUNT register (one clause per line, merged opener, no signpost words) — not tightened-explanatory. Getting the register wrong on pass 1 and switching later is what cost ca's NB22 four rounds.
2. Run the camera lint BEFORE proposing (spec: memory `reference_camera_lint.md`; regenerate `nb_camera_lint.py` per session)
3. Read the target notebook's markdown cells in full
4. REWRITE every Problem / Part-intro / note cell to its Camera Shape from scratch — keep the rewrite only where it's clearly better. Trim-in-place is how wordy cells survive passes. The ca NB15–19 tightened cells are templates for twin cells here.
5. Adjudicate every lint hit: a LONG-LINE in a body cell is guilty by default — split/cut it or carry a one-line justification in the proposal. No silent acquittals.
6. Cross-cell dedup: flag any intro claim another cell already owns (security note, KT, code comment)
7. Exercise stubs get the same pass: objectives ≤ 2 comment lines, TODOs one instruction per line, no mid-clause wraps
8. Produce the proposal — one entry per cell, before/after. **Decide what content survives first (the trim), then apply mechanical formatting only to what survives.** Lead with the cut, not the splits. Hard ceilings: Key Takeaway ≤ 2 sentences; Part intros ≤ 3 short lines plus bullets; decision-guide cells ≤ 3 bullets.
9. Get approval before writing
10. Use `nbformat` to read/write — never `json.load`/`json.dump`
11. Use the safe_write pattern (write to `.tmp`, then `shutil.move`)
12. Verify the changes in the raw JSON after writing, then re-run the lint as a regression gate
13. Commit; push to `origin master` when Scott says push

**Do not touch code cells.**

**What earns a 3rd line in Key Takeaway** (override the 2-sentence ceiling): concrete numbers ("2–3× median latency"), critical safety reminders for high-blast-radius tools (Bash, write tools, `bypassPermissions`), the "when to reach for this" use-case sentence when the mechanism alone doesn't tell the student when to apply it.

**Preserved variants** (not all `### 💡` cells become Key Takeaway):
- `### 💡 Why This Matters` — keep when the lesson is structural rather than mechanical (e.g., NB21 prompt injection cells — pre-renumber NB23)
- `### 💡 Why This Breaks` — keep for failure-mode demos
- `### 💡 The Principle` — keep for design-rule explainers
- `### 💡 When to Use Each` — keep for decision tables
