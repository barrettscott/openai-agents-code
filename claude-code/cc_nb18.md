# cc_nb18.md — 18_Capstone_2_Research_Team.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_3_Multi_Agent_Systems/18_Capstone_2_Research_Team.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add the missing demo run cell (CRITICAL)

5/5 reviewers — unanimous. The pipeline is defined but never called. "Phase 3: Demo" exists but contains only a cost note. Exercise 2 refers to `result["final_report"]` from "the demo cell above" — a cell that doesn't exist. Students lose trust in the notebook and can't complete Exercise 2.

> ⚠️ FIX CELL-TYPE BUGS FIRST, THEN ADD DEMO CALL:
>
> **Step 1 — Fix cell_type bugs:**
> - Cell 11 (which defines specialist agents) is incorrectly typed `markdown` — convert it to `code`.
> - Cell 14 (the `async def research_pipeline` definition) is incorrectly typed `markdown` — convert it to `code`.
> - The cell with content `## 🤖 Build the Specialist Agents` is incorrectly typed `code` — convert it to `markdown`.
> - The cell with content `## 🚀 The Research Pipeline...` (the Phase 2 section header) is incorrectly typed `code` — convert it to `markdown`.
> - The cell with content `## 🎬 Demo...` (the Phase 3 section header) is incorrectly typed `code` — convert it to `markdown`.
>
> **Step 2 — Add the demo call:**
> Confirm that `async def research_pipeline(question):` exists in a runnable code cell after the fix. Then locate the Phase 3 "Demo" section — if the demo code cell contains only a string like `"## 🎬 Demo\n\n⚠️ Cost note:..."` rather than actual Python, replace that cell's content rather than inserting a new cell.
>
> **Step 3 — Remove duplicate Exercise 1 code cells:**
> Cells 20 and 21 are duplicate Exercise 1 code cells — delete cell 20, keep cell 21.

Add the following code cell after the cost-note markdown in Phase 3 (or replace the malformed demo cell if it exists):

```python
result = await research_pipeline(
    "What is the current state of electric vehicle adoption globally?"
)
print("\n" + "="*60)
print("FINAL REPORT")
print("="*60)
print(result["final_report"])
```

---

## Change 2: Explain why procedural orchestration instead of handoffs/agents-as-tools

2 reviewers (Anthropic, OpenAI). Students just spent Lessons 14–15 learning SDK-native coordination; now the capstone uses neither, with no explanation. They carry the unanswered question through the whole lesson. This is a transfer blocker — they can't choose between procedural and agent-led orchestration in their own work.

**Find** the procedural orchestration bullet in the System Architecture section:

```
**Procedural orchestration** — Python coordinates the phases directly with `asyncio` and sequential `Runner.run()` calls
```

Add the following sentence immediately after that bullet (as a continuation or sub-point):

```
We coordinate phases in code here because the pipeline shape is fixed and we want explicit control over parallel execution and partial-failure handling — handoffs and agents-as-tools are better when the agent itself decides what to do next.
```

---

## Change 3: Explain `tool_config` / `container: auto` on the Analyst agent

3 reviewers (OpenAI, Gemini, Grok) flag this. `WebSearchTool()` needed no arguments; now `CodeInterpreterTool` takes a config dict with `tool_config`, `container`, and `auto` — none explained. The cost note later mentions "container reuse" as if the student already knows what that means.

Add the following inline comment immediately above the `analyst_agent` definition (the line before `analyst_agent = Agent(...)`):

```python
# container: "auto" lets the SDK reuse a Code Interpreter sandbox across calls — faster and cheaper than spinning a fresh one each time
```

---

## Change 4: Justify REASONING_MODEL on the Critic at the point of use

1 reviewer (Anthropic), but blocks transfer. Three agents use `MODEL`, one uses `REASONING_MODEL`, with no explanation at the point of choice. The rationale appears only in Key Takeaways — too late for the student to learn from the design decision.

Add the following sentence to the markdown cell introducing Phase 1 (or as a comment above the `critic_agent` definition):

```
We give the Critic `REASONING_MODEL` because critique is judgment-heavy — the other specialists do well-scoped work and run on `MODEL`.
```

---

## Change 5: Name the reusable pipeline pattern before the function definition

1 reviewer (OpenAI), blocks transfer. The `research_pipeline` function is the lesson's core takeaway, but no sentence frames it as a reusable template. Students can read it and still leave without "this is the shape I'd use for my own multi-agent workflow."

Add the following markdown sentence immediately before the `async def research_pipeline(question):` code cell:

```
This function is the reusable pattern: run independent specialists in parallel, convert failures into fallback text, then feed their outputs into downstream agents one phase at a time.
```

---

## Change 6: Add fallback-testing nudge after the demo run cell

1 reviewer (Grok), blocks transfer per reviewer. The failure-handling code is shown but never demonstrated — students see the pattern without feeling why it matters. This change depends on Change 1 (the demo cell must exist first).

Add the following sentence as a markdown cell or comment after the demo run cell added in Change 1:

```
Try temporarily breaking one specialist (e.g., pass an invalid tool) to watch the fallback text appear — the pipeline keeps running instead of crashing.
```

---

## Change 7: Add extension principle to Exercise 1 intro

1 reviewer (OpenAI), blocks transfer. Exercise 1 asks for a new FactChecker phase, but no sentence tells students that "adding/swapping a phase" is the extension mechanism for procedural pipelines.

**Find** the Exercise 1 intro that begins:

```
Extend the pipeline with a `FactChecker` agent that verifies the key claims in the final report using web search before it's delivered.
```

Add the following sentence immediately after it:

```
This is the main way you'll customize a pipeline in your own project — keep the orchestration shape, then insert or swap specialist phases where your workflow needs them.
```

---

## Change 8: Remove the duplicate Key Takeaways cell

Both the delivery notes and DeepSeek flag this. Two consecutive Key Takeaways cells exist — one using `<br>` tags for spacing, one without. Students read the takeaways, scroll down, and see them again.

Keep the version with `<br>` tags (cell 26). Delete the version without `<br>` tags (cell 25).

> ⚠️ VERIFY: Confirm the cell order before deleting — the notebook currently has the unformatted cell first and the formatted cell second. Delete the unformatted one (cell 25) and keep the `<br>`-formatted one (cell 26). If the order differs in the current notebook, apply the same rule: keep the `<br>`-formatted cell, delete the plain duplicate.
>
> Also note: cells 20 and 21 are duplicate Exercise 1 code cells — delete cell 20, keep cell 21 (this is handled in Change 1, Step 3, but verify it is done before applying this change).
