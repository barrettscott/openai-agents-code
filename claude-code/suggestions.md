# Suggestions: cc_nbXX vs Notebook Reviews

Per-change agree/disagree judgments comparing each `cc_nbNN.md` proposal against the current state of its corresponding notebook.

---

# Part 1 — Notebooks 01–08

## cc_nb01 — Environment Check
- **Change 1: Acknowledge unfamiliar packages in Check 4** — Agree. The "Let's verify all course packages are installed." text exists verbatim in cell 10 and the addition usefully addresses fastapi/uvicorn/pydantic appearing unannotated.
- **Change 2: Clarify `agents` vs `openai`** — Agree. `import openai` exists in cell 11 and the inline comment is well-placed.
- **Change 3: Explain Gradio version pin** — Agree. `if major >= 6:` exists verbatim in cell 11; inline comment is accurate.
- **Change 4: Name the failure mode for not pinning** — Agree. The Find text exists verbatim in cell 14; the addition is concrete and helpful.

**Overall:** All four changes match the current notebook and are small, accurate clarity wins — apply them all.

## cc_nb02 — OpenAI Setup
- **Change 1: Soft-limit answer** — Agree. Find matches cell 4 exactly; the prepaid-credit clarification closes a real ambiguity.
- **Change 2: MODEL comment** — Agree. `MODEL = "gpt-5-mini"` (no comment) exists in cell 18; matches the convention used in nb03/nb04.
- **Change 3: Step 4 orienting sentence** — Agree. Find matches cell 8; the addition usefully reorients before the OS/method split.
- **Change 4: Explain `load_dotenv` arguments** — Partial. The Find text exists in cell 15, but the inline comment is very long and may wrap awkwardly; the content is correct.
- **Change 5: Resolve "same pattern" sentence** — Agree (apply replacement branch). The "This same pattern…" sentence IS present in cell 17, so the softened replacement applies; it removes the contradiction with "don't worry about the code."

**Overall:** All five changes are valid against the current state; consider shortening the Change 4 comment for readability.

## cc_nb03 — How Agents Work
- **Change 1: REASONING_MODEL comment** — Agree. Find matches cell 3 verbatim; "Orchestrators" is indeed unintroduced jargon at this point.
- **Change 2: Tracing "when to use"** — Agree. Find matches cell 18 verbatim; extension is well-placed.
- **Change 3: Name three core pieces in "Why This Works"** — Agree. Find matches cell 8 verbatim; replacement aligns with the lesson's promise.
- **Change 4: Extend instructions takeaway** — Partial. The exact Find string is in cell 16, which is itself a duplicate of cell 15. Change 5 deletes a duplicate "Why This Works"; the instruction should specify *which* duplicate to keep so Change 4 isn't applied to a cell that's about to be removed.
- **Change 5: Remove duplicate cells** — Agree. Cells 15/16 are duplicate "Why This Works" cells, and cells 26/27 plus 30/31 are duplicate Takeaways/Troubleshooting pairs — all real duplicates confirmed.

**Overall:** Sound changes, but Change 4 + Change 5 need ordering/coordination since one of the duplicate "Why This Works" cells is the target of Change 4.

## cc_nb04 — Building Tools for Agents
- **Change 1: Define "schema"** — Agree. Find matches cell 10 verbatim; the parenthetical gloss is well-placed.
- **Change 2: Real-projects bridge** — Agree. The `@function_tool` anchor exists in cell 10; appending the sentence after the bullet list fits naturally.
- **Change 3: Docstring guidance at end of Part 3** — Agree. Part 3 currently ends with test runs and no docstring checklist; this delivers on a promised Topics line.
- **Change 4: Point to traces after first tool run** — Agree. Sensible reinforcement of the Lesson 03 debugging habit; cell 12 is the right insertion point.

**Overall:** All four are accurate and address real gaps — apply all.

## cc_nb05 — Writing Agent Instructions
- **Change 1: Connect back to Lesson 04 assistant** — Agree. The Lesson 04 bridge is currently absent after cell 10; the sentence is well-placed.
- **Change 2: Add "Why This Works" to Part 1** — Agree. Part 1 ends at cell 10 with no Why-This-Works cell, unlike Parts 2/3/4; the addition is needed.
- **Change 3: Name mechanism in Part 2 "Why This Works"** — Agree. Find matches cell 17 verbatim; the addition is the missing transferable principle.
- **Change 4: Note before Part 2 agent run — no tools** — Agree. `agent_clarify` indeed has no tools (cell 14); the note prevents real confusion.
- **Change 5: Real-world payoff and instructions-vs-docstrings hierarchy** — Agree. Find matches cell 21 verbatim; both add-ons are accurate and connect back to Lesson 04.
- **Change 6: Define refusal and escalation** — Agree. Find matches cell 23 verbatim; the two-line definitions land before the code cell.
- **Change 7: Rename Part 4 header** — Partial. Current header (cell 23) is `## 🛑 Part 4: Refusal and Escalation Patterns` — already includes "Refusal" and "Escalation," so per the change's own skip clause this should be skipped or minimized to scope-add only.
- **Change 8: Replace "load-bearing" with "critical"** — Agree. Find matches cell 39 verbatim.

**Overall:** Strong set; skip Change 7 per its own guard and apply the rest.

## cc_nb06 — Pydantic Basics
- **Change 1: Explain `Annotated`** — Agree. Find matches cell 16 verbatim; the gloss closes the only unanimous gap.
- **Change 2: Define `ge` and `le`** — Agree. Folded into the same Part 3 intro edit; appropriate.
- **Change 3: Why/decision rule for serialization** — Agree. Find matches cell 26 verbatim; addition closes both gaps in one place.
- **Change 4: Reframe "Why Pydantic Instead of Raw JSON Schemas"** — Agree. Both Find strings match cell 29 verbatim; reframing as "Why This Course Uses Pydantic" is more grounded.
- **Change 5: Bridge `ValidationError` to agent context** — Agree. Find matches cell 10 verbatim; the bridge sentence is well-placed for Lesson 07 setup.
- **Change 6: "(and hope it listens)"** — Agree. Find matches cell 22 verbatim; tonal hit lands.

**Overall:** All six changes are textually accurate and address real gaps — apply all.

## cc_nb07 — Structured Outputs
- **Change 1: Fix Problem-cell failure message** — Agree. Find matches cell 6 verbatim; the demo raises `TypeError`, so the replacement reflects what students actually saw.
- **Change 2: Clarify "silently passing through"** — Agree. Find matches cell 9 verbatim; replacement is more concrete.
- **Change 3: Field-design guidance** — Agree. Cell 9 contains the anchor; appending the field-design heuristic is helpful.
- **Change 4: Field names as prompt** — Agree. Find matches cell 13 verbatim; addition is a genuine quality lesson.
- **Change 5: Validation failure behavior** — Partial. Useful information, but the cc_nb file itself flags "Verify exact SDK behavior" — the wording should be checked against actual SDK behavior (constraint violations may surface as a retry/repair loop, not a raw `ValidationError`) before committing.
- **Change 6: Shape, not truth** — Agree. Important caveat; the Part 1 Why-This-Works cell (cell 13) is the right home.
- **Change 7: "When to use" bullet in Key Takeaways** — Agree. Key Takeaways (cell 26) currently lacks a "when to use" line; the bullet delivers a Topic promise.
- **Change 8: Real-world grounding for Optional fields** — Agree. Find matches cell 19 verbatim; addition makes Optional feel motivated.

**Overall:** All accurate; only Change 5 needs an SDK-behavior verification before final wording.

## cc_nb08 — Error Handling and Recovery
- **Change 1: Explain default SDK behavior before Part 1** — Agree. Find matches cell 8 verbatim; the new framing prevents Part 2 feeling like a reversal.
- **Change 2: Inline comment on `failure_error_function=None`** — Agree. Decorator line exists verbatim in cell 9; the demo-only warning is important.
- **Change 3: Define exponential backoff with jitter** — Agree. Find matches cell 22 verbatim; the gloss is the missing "what to do instead."
- **Change 4: Define "idempotent"** — Agree. Find matches cell 23 verbatim; parenthetical is clear.
- **Change 5: Name the fallback mechanism in Part 4** — Partial. The intent is correct, but Part 4 currently has no "Why This Works" cell to append to (cells 25–28 cover Part 4 with no Why-This-Works) — the change instructs "append to the existing Why This Works cell for Part 4" which doesn't exist; the change should instead create a new Why-This-Works markdown cell after cell 28.
- **Change 6: Name what `run_safely()` catches** — Agree. Useful clarification for Part 5 (around cell 33); content is accurate.

**Overall:** All changes valuable; Change 5 needs to be re-specified as "add new Why-This-Works cell" rather than "append to existing."

---

# Part 2 — Notebooks 09–16

## cc_nb09 — Testing & Evaluating Agents
- **Change 1: Add `check_output()` helper** — Agree. The function is called in cells 18, 25, 30, 31 but never defined; real NameError blocker and the proposed helper matches the calling signature.
- **Change 2: Name the test-set coverage strategy in Part 1** — Partial. The same three-category framing already exists in Key Takeaways ("happy paths, edge cases, and known failure modes"), so adding it to Part 1 is reasonable but duplicative — consider rewording to avoid repetition.
- **Change 3: Explain why the judge needs ground truth in Part 3** — Agree. Cell 24 passes the full `catalog` into `judge_input` with no surrounding explanation; the proposed sentence fills a real teaching gap.
- **Change 4: State the hypothesis before Part 4 comparison** — Agree. Cell 28 jumps straight into `v2_instructions` with no statement of what v2 changes or why — the proposed comment is well-placed.
- **Change 5: Clarify pass-rate vs avg-score decision rule** — Agree. Both metrics are printed in cells 30/31 but only avg score picks the winner; the proposed clarifying print is a one-liner that lands well.
- **Change 6: Add json save/load snippet after persistence note** — Agree. Cell 13 states the principle without any code; the snippet is a clean demonstration.

**Overall:** Solid set; Change 1 is a real blocker. The notebook also has unrelated structural bugs (cell 16 is a code cell containing markdown, cells 25 and 31 are markdown cells containing executable Python) that this cc_nb file does not address but should be flagged separately.

## cc_nb10 — Web Search
- **Change 1: Bridge "hosted tool" and "grounding"** — Agree. Find text matches cell 12; replacement adds two genuine bridges (to `@function_tool` and to "training data").
- **Change 2: Clarify when search actually fires** — Agree. Instructions in cell 11 say "always search" but the principle isn't named anywhere; one sentence in the Why This Works cell is appropriate.
- **Change 3: Replace citation guidance with copyable pattern** — Agree. Find text matches cell 17 exactly; proposed pattern is more actionable than the current vague advice.
- **Change 4: Explain inline URL reference** — Partial. The Find text appears in cell 17 (same cell as Change 3); both changes target overlapping prose, so they must be applied in order or merged to avoid one stomping the other.
- **Change 5: Clarify bias + location targeting** — Agree. Cell 21's Why This Works is currently a single thin sentence; added guidance is on-target.
- **Change 6: Name the `user_location` dict structure** — Partial. Cell 20 already has an inline comment `# "approximate" is the only supported type for user_location`, so this is largely redundant — could be skipped or used to replace the existing comment with a fuller prose sentence.

**Overall:** Good, but Changes 3 and 4 edit the same cell and need careful ordering, and Change 6 is partially redundant.

## cc_nb11 — File Search
- **Change 1: Plain-English vector/RAG explanation** — Agree. Find text matches cell 8 exactly; replacement is materially clearer for first exposure.
- **Change 2: Name the grounding payoff in Part 4 third test** — Agree. Cell 25 demos refusal but cell 26 never names it as the lesson; high-leverage add.
- **Change 3: Explain `max_num_results` and `vector_store_ids`** — Agree. Both appear in cell 19 with no explanation; addition is accurate and useful.
- **Change 4: Setup-vs-runtime distinction in Part 3** — Disagree on Find string. The exact phrase "This is a one-time setup step." appears inside cell 14 but is followed by additional sentences in the same paragraph; the Find/Replace will work but the replacement should be integrated, not appended after the existing "Vector stores act as collections..." text. Content is right, placement instructions need care.
- **Change 5: Transfer recipe in Part 4 intro** — Agree. Cell 18 currently has only one short sentence; the addition is well-placed.
- **Change 6: Demo vs production lifecycle in Cleanup** — Agree. Find text matches cell 43 exactly; clean one-sentence clarification.
- **Change 7: Part 4 bridge sentence** — Agree. After three setup cells, a one-line reorienting bridge is appropriate.
- **Change 8: Vocabulary inconsistency in Key Takeaways** — Agree. Find text matches cell 37; the swap restores consistency with Part 1's "converted into vectors" phrasing.
- **Change 9: Reframe security note** — Agree. Find text matches cell 27 exactly; reframe makes the note land for demo users.

**Overall:** Strong set; only Change 4's placement needs adjustment for the actual paragraph structure.

## cc_nb12 — Code Interpreter
- **Change 1: Fix Part 5 → Part 4 numbering** — Agree. Cell 25 currently reads "## 📁 Part 5: File Input" and there is no Part 4 — direct numbering bug.
- **Change 2: Explain `CodeInterpreterTool` config dict** — Agree. Cell 14 introduces the `tool_config` dict without explanation; proposed sentence is accurate.
- **Change 3: Set expectations before "Generate a Plot"** — Partial. Cell 23 already states "The chart renders inside the sandbox — you'll see the agent's text description of what it generated, not an image." Moving it *before* the code cell is still worthwhile, but the change should explicitly say "move" rather than risk duplicating the sentence. The "save as image file" addition is a fresh, useful add.
- **Change 4: Inline vs upload decision rule** — Agree. Cell 17 currently has only the bare statement; addition is well-placed and actionable.
- **Change 5: Explain `purpose="assistants"`** — Agree. Cell 26 uses `purpose="assistants"` with no explanation; markdown cell and inline comment are both appropriate.
- **Change 6: Resolve "we just deleted it" contradiction** — Agree. Find text matches cell 26 exactly; replacement resolves a genuine contradiction in the same cell.
- **Change 7: Clarify "container config" jargon** — Disagree. The exact Find text "then pass that ID into the container config." appears in cell 25 (the Part 5 intro), but Change 1 is renumbering that section to Part 4 — make sure both changes target the same cell and don't conflict; otherwise content is good.

**Overall:** Strong, mostly mechanical edits. Watch for Change 3 (avoid duplication) and Change 7 (depends on Change 1's cell).

## cc_nb13 — Capstone 1 Research Agent
- **Change 1: Fix broken Phase 3 try/except cell** — Agree. Cell 18 is a markdown cell containing executable Python (try/except + report printing) — rendered as raw text, never runs. The proposed action (delete the broken markdown cell, wrap the runnable cell 17 in try/except) is correct.
- **Change 2: Make tool selection visible** — Agree with caveat noted in the cc_nb file itself. `result.new_items` and `.type` are plausible for the OpenAI Agents SDK but should be verified at runtime; the fallback markdown pointer is a safe alternative.
- **Change 3: Reusable pattern note before Phase 2** — Agree. Cell 12 intro is thin; the addition gives students the transfer recipe.
- **Change 4: Explain why `output_type=ResearchReport` is worth it** — Agree. Cell 13 defines the Pydantic class without prose justification; addresses a real "why bother" gap.
- **Change 5: Replace bare `CodeInterpreterTool()` with explicit form** — Agree. Cell 13 contains `CodeInterpreterTool()` with no args; lesson 12 uses the explicit `tool_config` form. Aligns the capstone with the lesson and avoids a likely `TypeError`.
- **Change 6: Tighten Exercise 2** — Agree. Find text matches cell 25 exactly; replacement narrows scope appropriately.
- **Change 7: "Pipeline" → "Workflow"** — Agree. Find text matches cell 5 exactly; preempts a real terminology collision with Capstone 2.
- **Change 8: Transition sentence after Phase 1 setup** — Agree. After cells 8/10's file/upload sequence, a one-line bridge is useful before Phase 2.
- **Change 9: Define "vector store" at first use** — Agree. Find text matches cell 7 exactly; parenthetical definition is appropriate.

**Overall:** Excellent diagnostics; Change 1 (broken markdown cell) and Change 5 (bare CodeInterpreterTool) are real bugs worth applying first.

## cc_nb14 — Handoffs
- **Change 1: Explain why triage doesn't get control back** — Agree. Find text matches cell 8 exactly; replacement adds the missing rationale cleanly.
- **Change 2: Name agent-name → tool-name connection** — Agree. Cell 12 defines `BillingAgent`/etc. and cell 15 references `transfer_to_billing_agent` without ever explaining the derivation; one sentence fills the gap.
- **Change 3: Routing is prompt-driven, not enforced** — Agree. Cell 15 is the right place; appended sentence is accurate and useful for debugging.
- **Change 4: Explain `result.last_agent` and motivate it** — Agree. Find text matches cell 17 exactly; replacement adds motivation without bloat.
- **Change 5: Clarify "more control" in Part 4** — Agree. Find text matches cell 22 exactly; one-clause edit removes genuine ambiguity.
- **Change 6: Set expectation before ambiguous-message comparison** — Agree. Cell 25's comparison can show identical routing; prefacing sentence prevents student confusion.
- **Change 7: Transferable rule for custom descriptions** — Agree. Cell 26's Why This Works is missing exactly this decision rule; the proposed two sentences are well-targeted and the strongest fix in the file.
- **Change 8: Orienting sentence before three specialists** — Partial. Cell 12 already defines the three specialists in one cell with no preceding markdown; a one-line bridge is fine but optional.

**Overall:** Clean, accurate set; Find strings all match.

## cc_nb15 — Agents as Tools
- **Change 1: Add Why This Works after Part 3 demo** — Agree. Cell 23 is the demo; nothing follows it before Practice Exercises (cell 25). Symmetry with Part 2 argues for this addition, and the content names a real new idea.
- **Change 2: Explain how input reaches the specialist** — Agree. Cell 10's `.as_tool()` intro is currently bare; the two-sentence append addresses a real gap.
- **Change 3: Reword Step 3 to remove implied manual passing** — Agree. Find text matches cell 21 exactly; replacement removes a misleading implication about manual string injection.
- **Change 4: Explain `tool_name=` argument** — Agree. Cell 13 introduces `tool_name=` without explanation; sentence is accurate.
- **Change 5: Add use-case sentence to Part 2 Why This Works** — Agree. Find text matches cell 16; appended sentence adds real-world motivation.
- **Change 6: Replace "Coordinating" in table** — Agree. Find text matches cell 8 exactly; replacement is more concrete and matches the cell 7 description.

**Overall:** All Find strings check out; uniformly good additions.

## cc_nb16 — Parallel Execution
- **Change 1: Delete duplicated markdown cell and add Part 7 header** — Agree. Cell 35 is a markdown cell containing the `run_with_timeout` function as raw text (no code fence), and cell 36 is the runnable code cell — exact duplicate copy-paste artifact. There is no Part 7 header. The proposed fix is correct.
- **Change 2: Generalize partial-result threshold and clarify variables** — Agree. Cell 32 uses `topics` and `results` from the prior cell with no comment, and the thresholds `0`/`<2` are tied to the demo. Both proposed additions land.
- **Change 3: Name fan-out / fan-in pattern in body** — Agree. "Fan-out / Fan-in" appears in Key Takeaways (cell 46) but is never named in the body; cell 19 is the right place to introduce it.
- **Change 4: Gloss "coroutine" in Async Recap** — Agree. Find text matches cell 7 exactly; gloss approach is correct.
- **Change 5: Name sequential→parallel conversion pattern** — Agree. Cell 16 intro is thin; one-sentence transfer recipe is well-placed.

**Overall:** Accurate diagnosis; Change 1 is a real bug fix worth applying first.

---

# Part 3 — Notebooks 17–24

## cc_nb17 — Debate & Critique
- **Change 1: Add critique-vs-debate decision rule** — Agree. Part 4 intro cell exists with the exact header; sentence is well-placed and addresses a real gap.
- **Change 2: Explain REASONING_MODEL switch on judge** — Partial. The `quality_judge_agent` does use `REASONING_MODEL` (cell 17), but there is no dedicated markdown cell introducing it — only the "Run the Critique Loop" header (cell 18). The note would have to be inserted as a new markdown cell or as an inline comment above the code. The synthesis_agent verify note is also valid.
- **Change 3: Improve max iterations message** — Agree. Find text matches exactly in cell 19; replacement is more actionable.
- **Change 4: Replace for…else with tracked boolean** — Agree. `for…else` is present in cell 19; refactor improves clarity.
- **Change 5: Frame parallelism as optimization** — Agree. Part 4 "Why This Works" cell (27) exists; appending the sentence addresses a real conflation risk.

**Overall:** Solid set. Change 2 needs minor placement adjustment (no existing intro markdown for `quality_judge_agent`).

## cc_nb18 — Capstone 2 Research Team
- **Change 1: Add missing demo run cell (CRITICAL)** — Agree, with major verify caveat confirmed. Cell 11 (which defines specialist agents) is incorrectly typed `markdown`; cell 13 ("research_pipeline" intro prose) is incorrectly typed `code`; cell 14 (the `async def research_pipeline` definition) is `markdown`; cell 16 (the Phase 3 demo) contains only the cost-note string as code. So the pipeline truly never runs. Adding the demo call is correct but multiple cell-type fixes are required first.
- **Change 2: Procedural orchestration explanation** — Agree. Find text matches exactly in cell 5.
- **Change 3: Explain tool_config/container auto** — Agree. `analyst_agent` uses the config dict; comment is well-placed and accurate.
- **Change 4: Justify REASONING_MODEL on Critic** — Agree. Critic uses REASONING_MODEL with no in-place rationale.
- **Change 5: Name reusable pipeline pattern** — Agree. The `research_pipeline` definition lacks a framing sentence (and the cell currently mistyped as markdown).
- **Change 6: Fallback-testing nudge** — Agree. Reasonable and depends on Change 1.
- **Change 7: Extension principle on Exercise 1** — Agree. Find text matches cell 19 exactly.
- **Change 8: Remove duplicate Key Takeaways** — Agree. Two Key Takeaways cells exist (25 without `<br>`, 26 with `<br>`); guidance to keep the `<br>` version is correct. Note: also two duplicate Exercise 1 code cells (20/21) — not flagged but worth noting.

**Overall:** Comprehensive and accurate. Notebook has more structural cell-type bugs than just Change 1 mentions.

## cc_nb19 — Sessions & Conversation State
- **Change 1: Shared vs isolated session decision rule** — Agree. Find text matches Part 4 "Why This Works" cell 27 exactly.
- **Change 2: Shape of items returned by get_items()** — Agree. The Part 5 code (cell 30) accesses `role`/`content`; no prose explains the shape.
- **Change 3: Explain why SQLiteSession for in-memory** — Agree. Cell 12 creates `SQLiteSession("demo_session")` with no rationale.
- **Change 4: Real-app session ID guidance** — Partial. The Part 2 "Why This Works" cell (13) exists but bridges to Notebook 20; appending the sentence still works but placement could be earlier (e.g., near cell 12).
- **Change 5: When to call get_items/clear_session** — Agree. Useful and well-placed.
- **Change 6: Ground "why memory matters"** — Agree. Find text matches cell 5 exactly.
- **Change 7: Inline context window definition** — Agree. Find text matches cell 13 exactly.
- **Change 8: Hint for Exercise 2** — Agree. Exercise 2 TODO block (cell 41) lacks scaffolding.

**Overall:** All changes accurate and well-targeted.

## cc_nb20 — Persistent Memory
- **Change 1: Fix cell ordering bug in Part 3** — Agree, critical. Cell 22 calls `await noisy_session.get_items()` before `noisy_session` is defined in cell 24. Confirmed NameError blocker.
- **Change 2: Bridge from demo to "my own code"** — Agree. Find text matches cell 29 exactly.
- **Change 3: Motivate why clean session needs summarizing** — Agree. Reasonable gap; placement between Part 3 intro and noisy-session demo makes sense.
- **Change 4: Explain why Part 4 correction works** — Agree. Correction agent (cell 36) uses that instruction line; tying the success to it is well-placed in the Part 4 "Why This Works" (cell 39).
- **Change 5: Acknowledge user-controlled forgetting gap** — Agree. Topics list (cell 0) does promise "user-controlled forgetting"; cell 38 currently only says "for repeated conflicts apply summarize → clear → store." Adding the deletion clarification fills the gap.
- **Change 6: Prevent sensitive data from being persisted** — Agree. Security note (cell 34) makes a "never store" claim with no how-to.

**Overall:** All accurate; Change 1 is a real blocker that must land.

## cc_nb21 — Vector Memory
- **Change 1: Persistent vector store explanation** — Agree. Cell 14 uses `EphemeralClient()` with no persistence guidance.
- **Change 2: Document shape of results dict** — Agree. Inline comment is well-placed at the `.query()` site in cell 17.
- **Change 3: Acknowledge capture half of lifecycle** — Agree. Part 4 has no `add_memory` pattern; gap is real.
- **Change 4: Make Part 4 demo require the stored fact** — Partial. Find text matches cell 23 exactly, and the proposed replacement references "Alex" — semantically valid given the stored memory ("Alex prefers Python for data science and JavaScript for web apps"). The `print(f"[Memory] Querying: {query}")` addition also works since the function uses `query` parameter.
- **Change 5: Two "adapt to your project" sentences** — Agree. Both placements (after collection creation, in Part 4 "Why This Works") are appropriate.
- **Change 6: Inline comment for ids parameter** — Agree. Find text matches cell 14 exactly.
- **Change 7: Clarify what is doing the embedding/cost** — Agree. Find text matches cell 15 exactly. Replacement is accurate (Chroma uses a default local sentence-transformer).
- **Change 8: Decision rule vector vs session memory** — Partial. Useful, but the Part 5 table (cell 26) already compares the two; the proposed sentence overlaps somewhat. Still adds a sharper heuristic at the Part 4 intro level.

**Overall:** All accurate; minor overlap with existing Part 5 table on Change 8.

## cc_nb22 — Guardrails
- **Change 1: Fix broken Part 4 test cell (cell type bug)** — Agree, critical. Cell 27 is incorrectly typed `markdown` and contains the long-test code (would never run); cell 26 contains only the short-answer test as code. Fix sequence is correct.
- **Change 2: Explain `@input_guardrail` decorator + `InputGuardrail()` wrapper** — Agree. Both appear together in cell 11 with no rationale.
- **Change 3: Explain guardrail function signature** — Agree. Imports (cell 3) include `RunContextWrapper` and `TResponseInputItem` with no explanation.
- **Change 4: Explain ctx.context in Part 3** — Agree. Cell 18 has `context=ctx.context` with no explanation.
- **Change 5: Bridging sentence on output guardrails** — Agree. Useful clarification; placement after Part 4 demo works.
- **Change 6: Decision heuristic for blocking mode** — Agree. Find text matches cell 28 exactly.
- **Change 7: Orienting sentence at top of Part 5** — Agree. Fits cell 28's opening.
- **Change 8: Explain `output_info` on first use** — Agree. Cell 11 introduces `output_info` with no explanation.
- **Change 9: Define "tripwire" in Part 1** — Partial. The proposed location says "after the first use of 'tripwire'" — but the first use is in cell 7's diagram, and cell 8 already partially defines it ("If a guardrail triggers its **tripwire**, execution halts and raises an exception"). The proposed sentence is slightly redundant; could replace existing definition rather than append.
- **Change 10: Instructions-vs-guardrails distinction** — Agree. The Problem cell (5) doesn't make this distinction.
- **Change 11: Rule-based vs LLM-based heuristic** — Agree. Useful at Part 2 close.
- **Change 12: Cut parallel/blocking preamble from Part 1** — Agree. Cell 8 currently contains the cited material almost verbatim; pruning it is valid pacing fix. Note proposed Find text doesn't exactly match — slight adjustment needed for current wording.

**Overall:** Strong set. Change 9 partially redundant; Change 12 Find text needs slight adjustment.

## cc_nb23 — Prompt Injection & Tool Safety
- **Change 1: Reframe injection demo "Why This Matters"** — Agree. Find text matches cell 11 exactly.
- **Change 2: Define idempotency keys** — Agree. Find text matches cell 30 exactly.
- **Change 3: Confirmation round-trip mechanic in Part 5** — Agree. Cell 36 "Why This Works" lacks the second-turn mechanic.
- **Change 4: Tie retry danger to agents specifically** — Agree. Part 4 (cells 30) doesn't currently tie back to agents explicitly.
- **Change 5: Inline comment for path traversal** — Agree. Cell 21 has `if not safe_path.is_relative_to(workspace):` with no explanation.
- **Change 6: "Watch for" marker before injection comparison** — Agree. Cell 26 is the payoff; markdown before it improves signposting.
- **Change 7: Define blast radius** — Agree. Cell 22 uses the term without definition.
- **Change 8: Answer hint to Exercise 2 reflection** — Agree. Exercise 2 TODO 4 (cell 42) is reflection with no answer key.

**Overall:** All accurate and surgical.

## cc_nb24 — Human-in-the-Loop
- **Change 1: Add `result.to_state()` to Part 1 conceptual overview** — Agree. Find text matches cell 7 step 3 exactly. Real gap — `to_state()` first appears unannounced in Part 2.
- **Change 2: Comment that `arguments` is a JSON string** — Agree. Cell 23 uses `json.loads(interruption.raw_item.arguments)`; cell 12 prints `arguments` without noting its type.
- **Change 3: Explain `raw_item` and what interruption contains** — Agree. Cell 12 inspects `interruption.raw_item.name/arguments` without explanation.
- **Change 4: Name reusable pattern in Part 2 "Why This Works"** — Agree. Cell 15 lacks an extractable template summary.
- **Change 5: Explain `while` loop reasoning** — Agree. Find text matches cell 23 exactly.
- **Change 6: SDK-level vs instruction-level distinction** — Agree. Useful bridge from Lesson 23's instruction-based confirmation pattern.
- **Change 7: Tell students what line to replace in production** — Agree. Cell 23 has `decision = "yes"` with vague comment.
- **Change 8: Threshold is developer-defined** — Agree. Useful framing; Part 4 intro (cell 20) doesn't make this explicit.
- **Change 9: "Why This Works" beat after Part 3 rejection demo** — Agree. Part 3 (cells 17-18) ends without a closer cell.

**Overall:** All accurate and well-targeted; no issues found.

---

# Part 4 — Notebooks 25–32

## cc_nb25 — Tracing & Observability
- **Change 1: Add dashboard access instructions before Part 1** — Agree. No dashboard URL/orientation exists before the first checklist; cell 4 only mentions inspection in the dashboard generically.
- **Change 2: Add "what to do with a trace in your own work" sentences to Part 2** — Agree. Part 2 currently ends at the debugging checklist with "symptom → root cause → fix" but no transferable diagnostic framework or eval/guardrail tie-in.
- **Change 3: Add reusable model-comparison pattern before Part 3 code cell** — Agree. Part 3 intro (cell 23) doesn't articulate this as a reusable "hold constant, change one variable" method.
- **Change 4: Move span definition before "What Tracing Captures"** — Partial. The span definition cell (cell 8) currently sits inside Part 1, after "What Tracing Captures" (cell 5) which already mentions "handoff spans". Moving it earlier is correct. The proposed appended sentence about trace-vs-spans dashboard mapping is a good addition.
- **Change 5: Bridge sentence at top of Part 3** — Agree. Part 3 intro (cell 23) lacks a transition from debugging-mode to cost/performance-mode.
- **Change 6: Token definition + reframe first checklist item** — Agree. "Token" is used without definition; the Run 1 checklist line `□ What is the total token count?` exists verbatim in cell 11.
- **Change 7: Rename "LLM spans" to "model-call (LLM) spans"** — Partial. The exact phrase "how many LLM spans fired" appears in cell 14, but the proposed Find string only changes the one cell-14 instance and leaves "LLM span" elsewhere (e.g., Key Takeaways "Check the LLM span last"). Apply broader rename.

**Overall:** Solid set; all changes land on real gaps. Change 7 needs broader application to other "LLM span" mentions.

## cc_nb26 — Capstone 3 Customer Service
- **Change 1: Delete duplicate guardrail code cell** — Agree. Cells 11 (markdown-typed) and 12 (code) both define `SupportTopicCheck`/`support_topic_guardrail`. Cell 12 uses `RunContextWrapper` and `TResponseInputItem` which are not imported in the setup. Note: cell 11 is actually a markdown cell containing code — that's its own bug worth noting; the kept version should be a code cell.
- **Change 2: Delete duplicate pipeline cell and reconcile threshold** — Agree. Cell 19 (markdown containing code) has `AUTO_APPROVE_THRESHOLD = 100.0` with `agent=None`; cell 20 (code) has 50.0 without `agent`. Markdown cells 24 and 25 reference $100 and $50 respectively — inconsistent. Keep the $100/agent=None version but note cell 19 needs converting to code.
- **Change 3: Fix `handle_customer_message` signature for Exercise 1** — Partial (partly applied). Cell 19's version already has `agent=None`, `run_agent = agent or support_agent`, and uses `run_agent` in both `Runner.run` calls. The orienting sentence before Practice Exercises is missing. Only the prose addition is still needed.
- **Change 4: Replace vague tracing callout with concrete checklist** — Agree. Cell 21 contains the exact Find text. Replacement is well-targeted.
- **Change 5: Add Phase 2 header** — Agree. There's no `## 🛡️ Phase 2` markdown header; the notebook jumps from the security note (cell 9) and divider straight into the guardrail code.
- **Change 6: Explain why agent (not keyword) for guardrail** — Agree. No such rationale appears anywhere near the guardrail definition.
- **Change 7: Point student to Turn 2 session payoff** — Agree. Cell 24 mentions threshold but not the session-memory payoff at Turn 2.
- **Change 8: Explain interruption/state machinery above `while` loop** — Agree. The current cell 19 has no comment explaining the SDK's pause-resume mechanism.
- **Change 9: Comment on `needs_approval=True`** — Agree. Cell 8 defines `process_refund` with the decorator but no inline explanation of what the flag does or the read-vs-write rule.
- **Change 10: Orchestration layer explanation in Phase 4** — Agree. Cell 18 mentions "orchestration layer" without grounding it as a plain wrapper.
- **Change 11: Warn about `input()` prompt at Turn 2** — Agree. The demo will block on `input()` and no pre-demo cell tells students what to type.

**Overall:** Strong set. Critical: cells 11 and 19 are currently markdown cells containing Python code (a notebook bug) — the duplicate-cell fixes also need to convert kept cells to code cells. Change 3 prose addition still needed; function signature already correct.

## cc_nb27 — MCP Fundamentals
- **Change 1: Fix "external server" wording** — Agree. Cell 9 contains the exact Find text. Good clarification.
- **Change 2: Append ecosystem payoff to MCP bullet** — Agree. Builds on Change 1's replacement; sequence is correct.
- **Change 3: `MCPServerStdio` / `list_tools()` gloss in Part 2 intro** — Partial. Cell 14 (Part 2 intro) is brief and doesn't explain `MCPServerStdio` or why call `list_tools()` first. The "after the existing opening sentence" placement works.
- **Change 4: Resources clarifying clause** — Agree. Cell 12 contains the exact Find text. Replacement is more precise.
- **Change 5: "Transfer to other servers" sentence after Part 3 demo** — Agree. Cell 22 ends after the verify-on-disk print; no transfer/adaptation hint. Adding it before the Cleanup header is well-placed.
- **Change 6: Approval-gate caveat in security note** — Agree. Cell 18 (security note) doesn't mention `needs_approval` not applying to MCP tools or forward-reference Lesson 29. Important safety bridge.
- **Change 7: Remove `MCPServerSse` bullet** — Agree. Cell 33 Key Takeaways contains the bullet verbatim; SSE is never taught in this notebook.

**Overall:** Tight, well-targeted change set. All Find texts exist.

## cc_nb28 — Real-World MCP Servers
- **Change 1: `uvx` one-line gloss in Prerequisites** — Agree. Cell 5 has the exact comment. Replacement is informative and parallel with Lesson 27's `npx` treatment.
- **Change 2: `create_static_tool_filter` gloss** — Agree. Cell 20 (Part 4 intro) introduces tool filtering without explaining what the filter mechanically does.
- **Change 3: "Two safety layers" sentence** — Agree. Critical least-privilege point currently absent. Good placement after the gloss.
- **Change 4: `list_tools()`-first adaptation hint** — Partial. The proposed insertion location (`tools = await readonly_fs.list_tools()` in Part 4 code) does exist in cell 21. Comment is useful, though arguably overlaps with similar advice in Part 5.
- **Change 5: MCP composition payoff sentence in Part 3 intro** — Agree. Cell 17 intro is one sentence; adding the payoff line is well-placed.
- **Change 6: Replace "adversarial text" with named threat model** — Partial. "adversarial text" appears in two places: cell 23 (Part 5 checklist) and cell 35 (Key Takeaways). Find text is not unique; replace_all needed, or specify Part 5.
- **Change 7: Make "test a tool directly" actionable** — Agree. Cell 23 contains the exact Find bullet. Replacement adds concrete steps without inventing an unverified method name.

**Overall:** Good set. Change 6 needs handling of the duplicate occurrence.

## cc_nb29 — Capstone 4 MCP Assistant
- **Change 1: Server-Level Approval markdown cell before Phase 2** — Agree. No prose introduces `require_approval`, the policy dict shape, or the interruption loop pattern before the dense Phase 2 code (cell 11). Critical pedagogical gap.
- **Change 2: Pull "saving is a side effect" into markdown** — Agree. The comment in cell 11 is buried and confusingly worded ("auto-approving bypasses the same controls you just built"). Promoting it to a callout markdown cell is right. Note: cell is a single monolithic code cell, so "between Task 1 and Task 2" requires also splitting the cell (Change 3).
- **Change 3: Split Phase 2 monolithic cell at Task 1/Task 2 boundary** — Agree. Cell 11 is ~100 lines containing both tasks inside a single `async with` block. Splitting is right but requires restructuring the context manager (the VERIFY note acknowledges this).
- **Change 4: Standardize the two approval loops** — Agree. Task 1 loop prints only tool name and accepts `["yes","y"]`; Task 2 loop prints arguments and only accepts `"yes"`. The richer + lenient version should be applied to both. Pairs naturally with Change 3.
- **Change 5: Approval loop template and concrete Option B** — Agree. Cell 18 contains the exact Find texts for both edits. Option B currently is vague ("another fetch/search approach" — same `uvx mcp-server-fetch` command as already used) — Memory server is a genuinely different choice.

**Overall:** All five changes target real gaps. Changes 2-4 are interrelated and should be applied together.

## cc_nb30 — Project Structure & CLI
- **Change 1: Fix Part 7 → Part 6 numbering** — Agree. Cell 29 has `## 🌐 Part 7: When to Consider a Simple API Wrapper`; the previous section is Part 5, so Part 6 is skipped.
- **Change 2: Explain streaming event types** — Agree. Cell 23 ("Why This Works" for Part 4) doesn't name or explain `RawResponsesStreamEvent`/`ResponseTextDeltaEvent`/`data.delta`. Critical adaptation gap.
- **Change 3: Remove unused imports from streaming template** — Partial. Concept right (`from config import MODEL` and `from tools import my_tool` are unused), but the proposed replacement strips too much — it removes `Runner` and stream event imports that ARE used. Apply the concept but write a correct replacement that keeps `Runner` and the stream event imports.
- **Change 4: SDK-vs-OpenAI-package relationship sentence** — Agree. Cell 25 (Part 5 intro) introduces `client.responses.create()` with no explanation of how `openai` relates to `agents`.
- **Change 5: API key comment to `RESPONSES_TEMPLATE`** — Agree. Cell 26 has `client = OpenAI()` verbatim with no comment about env var sourcing.
- **Change 6: "Why files matter" sentence to Part 3 intro** — Agree. Cell 11 has the exact Find text. Replacement adds concrete motivation.
- **Change 7: Decision rule between Part 5's two paragraphs** — Agree. Cell 25 has the two-paragraph structure; adding a one-line rule of thumb between them is well-placed.
- **Change 8: Workflow note above Practice Exercises** — Agree. Real ambiguity for students: nothing tells them to leave Jupyter for these exercises.
- **Change 9: Streaming "why" sentence anchored to MCP context** — Agree. Cell 23 has the exact Find text. Anchoring to the MCP research/file-write task makes the value tangible.

**Overall:** Strong set. Change 3 has a flawed replacement (strips imports that are actually used); apply the concept but write a correct replacement.

## cc_nb31 — Architecture Decisions
- **Change 1: Add Part 8 Course Wrap-Up & Suggested Projects** — Agree. Topics list in cell 0 includes "Course wrap-up and suggested next projects", and Exercise 1 (cell 25) references "the six suggested projects from Part 8" — but no Part 8 exists. Real blocker.
- **Change 2: Cost and quality hook after `MODEL_EVAL_TEMPLATE`** — Agree. Cell 9 template shows latency and stubs quality; no cost guidance.
- **Change 3: Fix Guardrails row in summary table** — Agree. Cell 21 contains the exact Find row. Replacement reframes guardrails as proactive risk-protection rather than purely reactive, consistent with Lesson 22.
- **Change 4: Add `uvx` context in Part 5** — Agree. Cell 17 contains "requires Node.js or `uvx`"; parenthetical gloss aligns with rest-of-course treatment.
- **Change 5: Parallel-specialists dependency rule** — Agree. Cell 7 lists parallel specialists' use cases but doesn't define what "independent" means.

**Overall:** Tight, high-value set. Change 1 fixes the only true blocker.

## cc_nb32 — Deploying with Gradio
- **Change 1: Concrete HF Spaces deployment steps** — Agree. Cell 26 has the exact Find text (one sentence about Spaces). Replacement adds the actionable workflow Exercise 2 depends on.
- **Change 2: Tool visibility note above `APP_PY`** — Agree. `APP_PY` (cell 27) drops the `RunItemStreamEvent` block from Part 4. Students deploying tool-using agents would silently lose tool visibility.
- **Change 3: `history` object shape one-liner** — Agree. Cell 12 contains the dict-key code with no description of what Gradio actually passes.
- **Change 4: History-vs-sessions decision rule** — Agree. Cell 12 shows the mechanics but never tells the student when to use history-stitching vs sessions.
- **Change 5: Progressive-build orienting sentence** — Agree. Cell 9 (Part 2 intro) is brief and doesn't tell the student that Parts 2-5 are layered builds of the same app.
- **Change 6: Replace "Bridge" with plain-English action** — Agree. Cell 14 contains the exact Find sentence. Replacement is clearer.
- **Change 7: Cut Part 4 forward-reference to Part 5 try/except** — Agree. Cell 20 contains the exact Find sentence; removing the forward reference is the simplest fix.
- **Change 8: Tool-visibility "why it matters" sentence** — Agree. Cell 18 contains the exact Find sentence. Replacement adds the user-experience grounding.
- **Change 9: Reframe `config.py` / `os.getenv()` deployment note** — Agree. Cell 28 contains the exact Find sentence. Replacement clarifies Spaces doesn't forbid the Lesson 30 structure.
- **Change 10: Streaming event type conceptual frame in Part 3** — Skip (per cc_nb30 Change 2). cc_nb30 Change 2 adds the same conceptual explanation to Lesson 30; assuming it's applied, this Lesson 32 addition would duplicate.

**Overall:** Strong, well-targeted set. All Find texts verified. Change 10 should be skipped if cc_nb30 Change 2 is applied.
