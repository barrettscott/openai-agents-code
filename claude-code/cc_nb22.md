# cc_nb22.md — 22_Guardrails.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/22_Guardrails.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix the broken Part 4 test cell (cell type bug)

3 of 5 reviewers (Anthropic, Gemini, DeepSeek) flagged this. The cell beginning `print("📤 OUTPUT GUARDRAIL TEST")` has `cell_type: "markdown"` instead of `"code"`, so the output guardrail never visibly fires. This is the central demonstration promise of Part 4.

> ⚠️ VERIFY the cell structure before editing. There are two test-related cells: an earlier code cell with only the short-answer guardrail test, and a markdown cell containing the fuller short-plus-long test. The goal is to end up with exactly one Part 4 test cell that runs both tests in sequence (short passes, long triggers).

Apply in this order:
1. Convert the markdown cell beginning `print("📤 OUTPUT GUARDRAIL TEST")` from `"markdown"` to `"code"` — this is the cell with both the short and long test.
2. Delete the earlier code cell that contains only the short-answer test (since it is now a duplicate of the first half of the converted cell).
3. Confirm the resulting notebook has exactly one Part 4 test cell with the full sequence: short-test (passes guardrail) → long-test (triggers tripwire).

Do not leave both cells — students will see duplicated tests and the notebook flow will be broken.

---

## Change 2: Explain why `@input_guardrail` decorator + `InputGuardrail(...)` wrapper are both needed

2 of 5 reviewers (Anthropic, Gemini) flagged this as transfer-blocking. Students have just decorated their function and are now also wrapping it in a class — it looks redundant.

Add the following sentence near the first `cooking_agent` definition in Part 2, immediately before or after the `input_guardrails=[InputGuardrail(...)]` line:

```
The `@input_guardrail` decorator marks the function as a guardrail; wrapping it in `InputGuardrail(...)` is what attaches it to the agent and lets you configure parameters like `run_in_parallel` (Part 5).
```

---

## Change 3: Explain the guardrail function signature

3 of 5 reviewers (Anthropic, Gemini, Grok) flagged this as transfer-blocking. `RunContextWrapper`, `TResponseInputItem`, and `GuardrailFunctionOutput` are imported but never explained; `ctx` and `agent` are in the signature but unused in Part 2, leaving students unsure what they can change.

Add the following as a short markdown note or inline comment immediately above the `topic_guardrail` function definition in Part 2:

```
# The SDK calls every guardrail with the same signature: ctx is the run context (used in Part 3 for tracing),
# agent is the agent being guarded, and input is the user message. You can ignore parameters you don't need.
```

---

## Change 4: Explain `ctx.context` in Part 3

3 of 5 reviewers (Gemini, Grok, DeepSeek) flagged this. `context=ctx.context` appears in `Runner.run()` with no explanation; students don't know if it's required boilerplate or something meaningful.

Add the following inline comment on the `Runner.run()` line inside the LLM-based guardrail in Part 3:

```python
result = await Runner.run(topic_checker_agent, input, context=ctx.context)  # Pass the run context so the checker agent shares state and links into the same trace
```

---

## Change 5: Add bridging sentence to Part 4 — output guardrails are for more than length

2 of 5 reviewers (OpenAI, Grok) flagged this as transfer-blocking. The intro lists realistic uses (PII, policy, quality) but the demo is a 30-word limit — students leave thinking output guardrails are about length.

Add the following sentence after the output guardrail demo in Part 4:

```
This word-count example is deliberately simple — the same pattern (let the agent answer normally, then inspect the final output before it reaches the user) is how you'd block PII leaks, policy violations, or low-quality outputs in a real app.
```

---

## Change 6: Add decision heuristic for blocking mode in Part 5

2 of 5 reviewers (OpenAI, Grok) flagged this as transfer-blocking. Students understand the mechanics of blocking mode but have no concrete guidance for when to choose it.

**Find** (in Part 5 intro):
```
By default, guardrails run in parallel with the agent — both start at the same time. With `run_in_parallel=False`, the guardrail runs first and blocks the agent from starting if the tripwire fires. This saves tokens when the guardrail check is cheap and the agent call is expensive.
```

**Replace with:**
```
Use blocking mode (`run_in_parallel=False`) when your guardrail is cheap (keyword or simple rule) but the main agent is expensive — the agent never starts if the input is invalid, saving both latency and tokens. Stick with the default parallel mode when most requests are expected to pass and you want faster responses.
```

---

## Change 7: Add orienting sentence at the top of Part 5

2 of 5 reviewers (OpenAI, Grok) flagged a pacing break. After absorbing rule-based vs LLM-based, input vs output, and tripwires, Part 5 reuses a Part 2 guardrail with one new parameter — easy to lose the focal point.

Add the following sentence at the very start of the Part 5 markdown:

```
This isn't a new kind of guardrail — it changes *when* an input guardrail runs relative to the main agent.
```

---

## Change 8: Explain `output_info` on first use

2 of 5 reviewers (Anthropic, DeepSeek) flagged this. The field changes from a string (Part 2) to a Pydantic model (Part 4) with no explanation, and is never visibly consumed — students don't know whether it's required or what it does.

Add the following comment near the first `output_info=` use in Part 2's `GuardrailFunctionOutput` return:

```python
# output_info is surfaced in tracing/logs — useful for debugging why a guardrail fired. Can be a string or a Pydantic model (Part 4) when you want structured detail.
```

---

## Change 9: Define "tripwire" on first use in Part 1

1 reviewer (OpenAI) flagged this. "Tripwire" is used in Part 1 before students have seen a guardrail fire; cell 8 already partially defines it but the proposed sentence adds clarity.

REPLACE the existing partial definition in cell 8 with the fuller definition — do not append a redundant sentence after it:

**Find** (in cell 8, the existing partial definition):
```
If a guardrail triggers its **tripwire**, execution halts and raises an exception.
```

**Replace with:**
```
If a guardrail triggers its **tripwire** — the stop signal that fires when a violation is detected — execution halts and raises an exception instead of returning the agent's response.
```

---

## Change 10: Add instructions-vs-guardrails distinction

1 reviewer (DeepSeek) flagged this as transfer-blocking. Students already write topic limits into instructions; without the distinction they don't know when to reach for guardrails.

Add the following sentence to the "The Problem" cell:

```
Instructions encourage good behavior; guardrails enforce it — they can't be overridden by the model.
```

---

## Change 11: Add rule-based vs LLM-based decision heuristic

1 reviewer (OpenAI) flagged this as transfer-blocking. Students can copy each demo but have no heuristic for choosing between them.

Add the following sentence at the end of the Part 2 intro:

```
Use rule-based guardrails when the violation is easy to define in code — keywords, length limits, required formats, or simple allow/block lists. Reach for LLM-based (Part 3) when the check needs judgment.
```
