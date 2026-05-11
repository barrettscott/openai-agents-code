# cc_nb13.md — 13_Capstone_1_Research_Agent.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/13_Capstone_1_Research_Agent.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix the broken Phase 3 cell — move try/except into the runnable cell

4 of 5 reviewers (Anthropic, Gemini, DeepSeek — twice). This is the critical fix. The Topics list and Key Takeaways both promise try/except error handling; the runnable Phase 3 cell has none. The cell directly below contains the try/except pattern but is mis-typed as `cell_type: "markdown"`, so it renders as raw text and never runs. The promised teaching beat is invisible.

**Action:** Delete the broken markdown cell that contains the try/except code as plain text. Then update the runnable Phase 3 cell:

**Find** (in the runnable Phase 3 code cell):
```
research_result = await Runner.run(research_agent, input=research_topic)
```

**Replace with:**
```
# Robust version — wrap multi-tool runs in try/except; failures can happen at any tool step
try:
    research_result = await Runner.run(research_agent, input=research_topic)
except Exception as e:
    print(f"Research failed: {e}")
    research_result = None
```

> ⚠️ VERIFY: Read the Phase 3 section of the notebook to confirm the exact cell structure — there should be one runnable cell and one broken markdown cell. Delete the markdown cell and update only the runnable cell. The try/except structure should match whatever the broken markdown cell contained, preserving the output rendering and report printing that follows.

---

## Change 2: Make tool selection visible in Phase 3 output

1 of 5 reviewers (Anthropic), but this is the capstone's core new idea — an agent autonomously choosing among three tools — and it's completely invisible in the output. The student runs the agent and gets a report but can't see which tools fired.

Add the following line after the final report print block in Phase 3:

```python
print(f"\nTools called: {[item.type for item in research_result.new_items if hasattr(item, 'type') and item.type.endswith('_call')]}")
```

> ⚠️ NOTE: Verify `result.new_items` and `.type` attributes exist in the current SDK before applying — if they are not available, use the markdown pointer fallback instead: "Open the tracing dashboard (Lesson 03) to see which tools the agent called and in what order."

---

## Change 3: Add the reusable pattern note before Phase 2 agent construction

2 of 5 reviewers (OpenAI, Grok). Students see the demo work but can't identify what to swap for their own project. This is the standard "pattern vs. example-specific" gap.

Add the following sentence before the Phase 2 agent code cell:

```
Pattern to copy: list every tool the agent might need, give it numbered step-by-step instructions for when to use each, and start with default parameters. In your own project, you'd swap three things — the vector store contents, the agent instructions, and the `ResearchReport` fields.
```

---

## Change 4: Explain why `output_type=ResearchReport` is worth the extra work

1 of 5 reviewers (Grok), but the confusion is real — defining a Pydantic class feels like overhead when formatted text would look the same. This directly addresses the "lost why" for structured outputs in a capstone context.

Add the following line as a print statement or markdown cell immediately after the final report print block:

```
# Because we used output_type=ResearchReport, `report` is a validated Python object
# with .executive_summary, .key_findings, etc. — safe to pass directly to other code,
# databases, or downstream agents.
```

Or as a prose sentence in a markdown cell:

```
Because we used `output_type=ResearchReport`, `report` is a validated Python object with `.executive_summary`, `.key_findings`, etc. — safe to pass directly to other code, databases, or downstream agents.
```

---

## Change 5: Replace bare `CodeInterpreterTool()` with the explicit form from Lesson 12

1 of 5 reviewers (Anthropic). Lesson 12 used the explicit `tool_config` form. This capstone uses bare `CodeInterpreterTool()`, which is not valid in the local SDK (`CodeInterpreterTool` requires a `tool_config` argument). Using the no-arg form will raise a `TypeError` at runtime.

**Find** (in the research agent's tools list):
```python
CodeInterpreterTool()
```

**Replace with:**
```python
CodeInterpreterTool(tool_config={
    "type": "code_interpreter",
    "container": {"type": "auto"}  # "auto" lets the SDK reuse a sandbox across calls — faster and cheaper
})
```

> ⚠️ NOTE: Verify that bare `CodeInterpreterTool()` raises a TypeError in the current SDK before applying — if it works with defaults, this change is style-only. Confirm `CodeInterpreterTool()` appears without arguments in the agent's tools list. If it already uses the explicit `tool_config` form, skip this change. Do not add a comment claiming the no-arg form is equivalent — it is not valid in the current SDK.

---

## Change 6: Tighten Exercise 2 to reduce cognitive overload

2 of 5 reviewers (OpenAI, Grok). Exercise 2 asks students to build a judge agent, run the research agent twice, score results, and print scores — all at once. The focal point is unclear.

**Find** (the Exercise 2 objective line):
```
Apply the evaluation pattern from Lesson 09 — define a small golden test set, run the research agent against each case, and score the results with a judge agent.
```

**Replace with:**
```
Apply the judge-agent evaluation pattern from Lesson 9 to one research report (focus on the judge step, not building a full test harness). Good golden tests for this kind of agent check both content coverage (did it include required facts?) and tool-backed grounding (did it cite sources or use internal data when expected?).
```

Then reduce the TODOs in the exercise cell to two: (1) create the judge agent, (2) run the judge on one report.

---

## Change 7: Replace "Pipeline" with "Workflow" in the System Architecture cell

1 of 5 reviewers (Anthropic). "Pipeline" with arrows suggests sequential code the student writes. Capstone 2 (Lesson 18) will have an actual procedural pipeline — pre-spending the term here causes confusion when students hit it again.

**Find** (in the System Architecture markdown cell near the top of the notebook):
```
**Pipeline:** Topic → Research (web + docs) → Analyze → Report
```

**Replace with:**
```
**Workflow:** Topic → agent autonomously chooses tools → structured report. (Capstone 2 will introduce explicit multi-agent pipelines you orchestrate phase-by-phase.)
```

---

## Change 8: Add transition sentence after Phase 1 setup

1 of 5 reviewers (Grok). After two cells of file-writing and SDK calls, students can lose sight of what the notebook is teaching. A one-sentence bridge reorients them before Phase 2.

Add the following as a markdown cell after the vector store upload cell in Phase 1:

```
With the private knowledge base ready, we can now build a single agent that has access to both public web data and our internal survey.
```

---

## Change 9: Define "vector store" at first use in Phase 1

1 of 5 reviewers (Gemini). Students coming from Lesson 12 (Code Interpreter) uploaded files via a different mechanism. The term "vector store" appears without explanation of why it's a separate object.

**Find** (in the Phase 1 intro markdown cell):
```
We'll create an internal reference document and upload it to a vector store — giving the agent access to private context that web search won't find.
```

**Replace with:**
```
We'll create an internal reference document and upload it to a vector store (an OpenAI-managed collection that indexes your files for retrieval by `FileSearchTool`) — giving the agent access to private context that web search won't find.
```
