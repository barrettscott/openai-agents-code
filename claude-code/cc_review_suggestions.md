# Codex Review of Current CC Files

Reviewed path: `~/Dropbox/Notebooks/openai-agents/claude-code`

Compared against: `~/Dropbox/Notebooks/openai-agents/Week_*/*.ipynb`

Scope: `cc_nb01.md` through `cc_nb32.md`

Review lens: whether each CC file still applies to the current notebook, whether the suggested edit is correct, and whether the CC file misses any blocker visible in the notebook.

## Bottom Line

Most of the CC files are worth applying. They usually add missing definitions, decision rules, and transfer guidance at the right point in the lesson.

Do not apply the batch blindly. A few CC files have stale `Find` text, a few need semantic application rather than literal search/replace, and several notebooks have execution or cell-type blockers that should be fixed before prose polish.

## Highest Priority Fixes

### `cc_nb09.md` - apply, but add missing cell-type fixes

Agree with Change 1: `check_output()` is called multiple times and is never defined.

Additional blocker not covered by the CC file: the current notebook also has cell-type errors:

- Cell containing `## ✅ Part 2: Pass/Fail Checks` is a code cell, but it is markdown prose.
- Cell beginning `# Select a few test cases for rubric evaluation` is markdown, but it contains runnable Python.
- Cell beginning `# Reuse v1 baseline from Part 3` is markdown, but it contains runnable Python.

Fix those cell types before or alongside the missing `check_output()` helper.

### `cc_nb13.md` - apply

Agree with Change 1. The Phase 3 runnable cell is incomplete and the intended `try/except` run appears as markdown. This is a real execution blocker.

Agree with Change 5. The current notebook uses bare `CodeInterpreterTool()`, while the previous Lesson 12 pattern uses explicit tool config. Replace it with:

```python
CodeInterpreterTool(tool_config={
    "type": "code_interpreter",
    "container": {"type": "auto"}
})
```

### `cc_nb18.md` - apply, but add missing cell-type fixes

Agree with Change 1. `research_pipeline()` appears in a markdown cell, not a runnable code cell, and the capstone needs an actual demo run.

Additional blockers not fully covered by the CC file:

- Cell `## 🤖 Build the Specialist Agents` is currently a code cell.
- Cell `## 🚀 The Research Pipeline...` is currently a code cell.
- Cell `## 🎬 Demo...` is currently a code cell.
- There is also an exercise block duplicated as both code and markdown.

Fix the cell types while applying Change 1. Agree with Change 8 as currently written: delete the unformatted duplicate Key Takeaways cell and keep the formatted one.

### `cc_nb20.md` - apply, hard blocker

Agree with Change 1. The notebook calls `await noisy_session.get_items()` before `noisy_session` is defined. That breaks run-in-order execution.

Apply this before the memory guidance edits.

### `cc_nb22.md` - apply, hard blocker

Agree with Change 1. The fuller output-guardrail test is stored as markdown, while a shorter duplicate exists as code. Convert the fuller short-plus-long test to code and delete the short-only duplicate.

### `cc_nb26.md` - apply, but add one missing blocker

Agree with Change 1: there are duplicate guardrail definitions, and the typed version uses names that are not imported.

Agree with Change 2 and Change 3: there are duplicate `handle_customer_message` implementations with conflicting approval thresholds/signatures, and the exercise needs the `agent` parameter path to work.

Additional blocker not covered by the CC file: the Key Takeaways cell is currently a code cell beginning `## 🎯 Key Takeaways`. Convert it to markdown.

### `cc_nb31.md` - apply with one wording correction

Agree with Change 1 in substance: the Topics list promises "Course wrap-up and suggested next projects", but the notebook ends after Part 7 and has no project list.

Adjust the rationale text before applying. The current exercise says "Use the decision framework on a real or suggested project"; it does not currently say "pick one of the six suggested projects from Part 8." The fix is still valid, but the CC explanation overstates the current broken reference.

## Per-File Review

### `cc_nb01.md` - apply

All four changes match the current notebook and improve setup comprehension. The Gradio version-pin explanation is useful because the package check otherwise looks arbitrary.

### `cc_nb02.md` - apply

All changes are appropriate. Change 1 is especially useful because soft limits are easy to misread as hard spending caps. Change 5 is low risk and improves wording.

### `cc_nb03.md` - apply

Changes 1-4 are good. Change 5 is valid if the duplicate ending cells are still present when applying. Confirm the duplicate Key Takeaways and duplicate troubleshooting cells before deleting.

### `cc_nb04.md` - apply semantically

The suggestions are good, but Change 1's literal `Find` text has drifted. The current notebook separates the type-hints bullet and "No manual JSON required." into separate lines. Apply the schema definition to the type-hints bullet rather than relying on exact string replacement.

Change 2 already warns that the find text may drift; follow that warning.

### `cc_nb05.md` - apply

The changes strengthen instruction-writing transfer. Change 3 is especially useful because it names the actual mechanism for clarifying questions: list required fields explicitly.

### `cc_nb06.md` - apply semantically

Changes 1, 2, 3, 5, and 6 are good. Change 4's header `Find` text has drifted because the current notebook heading includes an emoji: `## 🧠 Why Pydantic Instead of Raw JSON Schemas`. Apply the rename semantically.

### `cc_nb07.md` - apply semantically

The changes are aligned with the notebook. Change 6 is especially important: structured outputs guarantee shape, not truth. Change 7's Key Takeaways `Find` text may drift because the current heading includes an emoji; apply semantically.

### `cc_nb08.md` - apply

The changes correctly protect students from copying demo-only error handling. Change 2, the inline comment on `failure_error_function=None`, is important.

### `cc_nb09.md` - apply with additional blocker fixes

Apply all CC changes, especially the missing `check_output()` helper.

Also fix the three cell-type problems listed in Highest Priority Fixes. Without those, Parts 2-4 still will not run correctly even after `check_output()` exists.

### `cc_nb10.md` - apply

The search-trigger and citation-pattern changes are useful. Change 2 is the most important because the notebook can otherwise send mixed signals about when web search runs.

### `cc_nb11.md` - apply

The changes improve the RAG/vector-store mental model and the setup-vs-runtime distinction. Change 6 is useful because cleanup differs sharply between demo and production vector stores.

### `cc_nb12.md` - apply

Change 1 is a real structural fix: Part 5 should become Part 4. The Code Interpreter config explanations are worth adding because students otherwise copy a dictionary shape they do not understand.

### `cc_nb13.md` - apply

Apply all changes. Prioritize Change 1 and Change 5 because they affect execution correctness and API correctness.

### `cc_nb14.md` - apply

The handoff notes are accurate. Change 1 is particularly important: in this pattern, the triage agent does not get control back after handoff.

### `cc_nb15.md` - apply

The changes clarify the difference between handoffs and agents-as-tools. Change 2 and Change 5 are the most transferable: the orchestrator passes input to the specialist and remains `result.last_agent`.

### `cc_nb16.md` - apply

Change 1 is important: remove the duplicated markdown copy of `run_with_timeout` and add the Part 7 header. The other changes improve async transfer.

### `cc_nb17.md` - apply

The changes are sound. Change 4 is a reasonable teaching simplification: replacing `for...else` with an explicit boolean reduces Python syntax distraction.

### `cc_nb18.md` - apply with additional cell-type fixes

Apply the CC file, but also fix the markdown-heading code cells listed in Highest Priority Fixes. Those cell-type bugs will confuse execution and notebook rendering.

### `cc_nb19.md` - apply

The session guidance is good. Change 4, real-app session ID guidance, is especially valuable because demo session IDs do not transfer directly to deployed apps.

### `cc_nb20.md` - apply

Apply Change 1 first. Then apply the memory design changes. Change 5 and Change 6 are especially important because user-controlled forgetting and sensitive-data persistence are production requirements.

### `cc_nb21.md` - apply

The suggestions fill the main lifecycle gaps: persistence, `results` shape, memory capture/write path, and vector-vs-session decision rule. Change 4 is good because the demo query should force use of the stored fact.

### `cc_nb22.md` - apply

Apply Change 1 first. The remaining changes are mostly strong teaching additions. Change 12 is also useful if the Part 1 preamble still over-explains parallel/blocking before students have seen guardrails.

### `cc_nb23.md` - apply

The changes are good. Change 2 is the most important because "idempotency key" is otherwise undefined right when students need to reason about retries.

### `cc_nb24.md` - apply

The changes clarify HITL mechanics. Change 2 is important: `arguments` being a JSON string rather than a dict is a common source of broken approval code.

### `cc_nb25.md` - apply

The suggestions improve trace transfer. Change 1 is useful if students need to open the dashboard; Change 6 is useful because "token" is often assumed but not defined.

### `cc_nb26.md` - apply with additional Key Takeaways cell-type fix

Apply the duplicate-cell and signature fixes first. Also convert the Key Takeaways code cell to markdown.

### `cc_nb27.md` - apply semantically

The changes are good, but Change 1/2 literal find text may drift depending on whether Change 1 has already been applied. The current notebook still has "MCP — an external server hosts the tool", so apply the wording fix and ecosystem payoff in one edit.

Change 6 is useful but should be phrased as a forward pointer, not a claim that Notebook 29 already taught it.

### `cc_nb28.md` - apply

The changes are good. Change 3, the two safety layers sentence, is the most important transfer point: server scope plus tool filtering.

### `cc_nb29.md` - apply

The approval-loop guidance is useful. Change 3, splitting the monolithic Phase 2 cell at the Task 1 / Task 2 boundary, should improve readability and recovery if one step fails.

### `cc_nb30.md` - apply

Change 1 is a real structural fix: Part 7 should become Part 6. Change 8 is also important because students need to know whether to create project files in the notebook, editor, or terminal.

### `cc_nb31.md` - apply with corrected rationale

Apply Change 1, but revise the explanation as noted above. The missing wrap-up/project list is real because the Topics list promises it.

Apply Changes 2-5. Change 3 is especially good because the current Guardrails row implies guardrails are only reactive after bad inputs have been observed.

### `cc_nb32.md` - apply

The changes are good. Change 1 fills the biggest deployment gap by making HF Spaces steps concrete. Change 3 and Change 4 are important because Gradio `history` is UI state, not the same thing as course session memory.

## Stale or Drifted Find Text

These CC files should be applied semantically rather than with literal search/replace only:

- `cc_nb04.md`: Change 1 text is split across lines in the current notebook.
- `cc_nb06.md`: Change 4 heading includes an emoji in the notebook.
- `cc_nb07.md`: Change 7 Key Takeaways heading includes an emoji.
- `cc_nb27.md`: Change 1 and Change 2 target the same MCP bullet; combine them.
- `cc_nb31.md`: Change 4 already warns that the `uvx` find text may drift.

## Recommended Apply Order

1. Fix execution/cell-type blockers: `cc_nb09`, `cc_nb13`, `cc_nb18`, `cc_nb20`, `cc_nb22`, `cc_nb26`.
2. Fix structural numbering/wrap-up issues: `cc_nb12`, `cc_nb16`, `cc_nb30`, `cc_nb31`.
3. Apply remaining teaching clarifications in notebook order.

