# cc_nb25.md — 25_Tracing_And_Observability.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/25_Tracing_And_Observability.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add dashboard access instructions before the first checklist

4 of 5 reviewers (Anthropic, Gemini, Grok, DeepSeek) flagged this as the priority fix. Every inspection checklist in Parts 1, 2, 3, and both exercises sends students to "find the trace" with no URL or navigation context — the lesson's entire observational spine depends on this.

Add a new markdown cell at the start of Part 1, before Run 1, containing:

```
Open the OpenAI traces dashboard at `https://platform.openai.com/traces` in a separate tab. Every `Runner.run()` you execute appears there within a few seconds, tagged by agent name. Keep this tab open as you work through the lesson.
```

---

## Change 2: Add the "what to do with a trace in your own work" sentence to Part 2

4 of 5 reviewers (Anthropic, OpenAI, Grok, DeepSeek) converged on this from different angles. The debugging demo ends with "this is how you use a trace to go from symptom → root cause → fix" but never names the diagnostic framework a student should reapply.

Add the following two sentences after the existing debugging checklist at the end of Part 2:

```
In your own projects, open the trace for any failed eval case (Lesson 09) or guardrail violation first — it turns the symptom into a concrete root cause. The diagnostic question is always the same: was the failure in tool selection, tool arguments, tool output, or the instructions that drove them?
```

---

## Change 3: Add the reusable model-comparison pattern before the Part 3 code cell

1 reviewer (OpenAI) flagged this as transfer-blocking. Students see two hardcoded agents compared once; they need to understand this as a reusable method, not a one-off demo.

Add the following sentence immediately before the Part 3 model-comparison code cell:

```
This is the reusable pattern for your own projects: hold the task constant, change one variable at a time (model, instructions, or tool set), then compare the traces.
```

---

## Change 4: Move the span definition before the "What Tracing Captures" bullet list

4 of 5 reviewers (Anthropic, Gemini, Grok, OpenAI) flagged the span term appearing before it's defined. "Handoff spans" appears in the bullet list, but the definition (`A **span** is one recorded step in the trace…`) is one cell later.

Move the one-sentence span definition cell to immediately precede the "What Tracing Captures" bullet list. Also append to that definition:

```
In the dashboard, a trace is the full run; spans are the rows inside it (one per model call, tool call, or handoff).
```

> ⚠️ NOTE: Verify cell ordering before applying — if the span definition is already positioned before the "What Tracing Captures" bullet list, skip the move and only append the new sentence to the definition.

---

## Change 5: Add a bridge sentence at the top of Part 3

2 of 5 reviewers (OpenAI, Grok) flagged the abrupt mental shift. Parts 1–2 use traces to find bugs; Part 3 pivots to cost/performance decisions with no transition.

Add the following sentence under the Part 3 header:

```
So far we've used traces to explain wrong behavior; now we'll use the same trace data for a different job — comparing cost, latency, and model choice.
```

---

## Change 6: Add a token definition and reframe the first token-count checklist item

2 of 5 reviewers (Anthropic, DeepSeek) flagged this from related angles. The word "token" appears repeatedly with no definition; the first checklist asks for a count but gives the student no baseline or context for the number.

In the "What Tracing Captures" cell, add the following sentence after the bullet list:

```
A token is roughly a word or subword chunk — model pricing and context limits are based on tokens.
```

**Find** (in the Run 1 checklist):
```
□ What is the total token count?
```

**Replace with:**
```
□ What is the total token count? (You'll compare this to Run 2 next — for now, just note the number.)
```

---

## Change 7: Rename "LLM spans" to "model-call (LLM) spans" throughout the notebook

Grok flagged that "LLM span" appears in checklists without being distinguished from tool-call spans. Apply a broader rename across the entire notebook — not just cell 14.

Replace every instance of "LLM span" with "model-call (LLM) span" throughout the notebook (apply to all occurrences, not just one cell).

**Example — Find** (in the "What These Traces Show" cell or a checklist):
```
how many LLM spans fired
```

**Example — Replace with:**
```
how many model-call (LLM) spans fired
```

Also apply the same replacement to any other cell in the notebook that contains the phrase "LLM span" (e.g., Key Takeaways "Check the LLM span last" and any other checklist or prose occurrences).
