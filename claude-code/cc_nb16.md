# cc_nb16.md — 16_Parallel_Execution.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_3_Multi_Agent_Systems/16_Parallel_Execution.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Delete the duplicated markdown cell and add a Part 7 header

5/5 reviewers flag this. The `run_with_timeout` function definition appears twice — once in a markdown cell (rendered as plain text, no code fence) and again in the real code cell immediately below. On top of that, the entire timeout section has no Part header, so it reads as a stray helper rather than the third major reliability pattern. This is the single most disorienting moment in the notebook.

**Step 1 — Delete** the markdown cell that contains the `run_with_timeout` function rendered as plain text (immediately before the runnable code cell with the same function). It is a copy-paste artifact.

**Step 2 — Insert** the following new markdown cell immediately before the `run_with_timeout` code cell (after the Part 6 Why This Works cell):

```markdown
## ⏱️ Part 7: Timeouts and Fallbacks

`return_exceptions=True` handles errors, but a hung or slow agent can still block the whole `gather()`. Timeouts give every task a deadline.
```

---

## Change 2: Generalize the partial-result threshold and clarify the demo variables

4 reviewers (Anthropic, OpenAI, Grok, DeepSeek) flag this from different angles. The thresholds `0` and `< 2` look standard but are tied to this demo's three tasks; the artificial `bad_task` list breaks the link to the happy-path agents; and `results`/`topics` appear without acknowledgment of where they came from.

**Step 1 — Add** the following comment at the top of the "Partial Result Handling" code cell:

```python
# Using `results` from the previous cell, `topics` from the first demo
```

**Step 2 — Add** the following sentence after the synthesis print block in that same cell (after the final `print(...)` that shows synthesis output):

```
In your own pipeline, replace the artificial `bad_task` list with `asyncio.gather(...)` of your real specialist agents from Part 4, and set the threshold based on which results are critical — e.g., abort if a required specialist failed, proceed if at least one of each category succeeded.
```

---

## Change 3: Name the fan-out / fan-in pattern in the body

2 reviewers (Anthropic, Gemini) flag that "Fan-in" debuts in Key Takeaways but was never named in the body. Students hit it in the takeaways like a vocab quiz on terms that were never taught.

Add the following sentence to Part 4's intro markdown cell (the one introducing the parallel + merge pattern):

```
This is the **fan-out / fan-in** pattern — fan out to launch specialists in parallel, fan in to merge their outputs.
```

---

## Change 4: Gloss "coroutine" in the Async Recap

2 reviewers (Gemini, Grok) flag this, with a split on fix: Gemini wants to replace the term, Grok wants to gloss it. Grok's approach is stronger — students will encounter "coroutine" in real-world Python async docs and Stack Overflow and shouldn't be sheltered from the standard term.

**Find:**
```
`asyncio.gather()` — launches multiple coroutines at the same time and waits for all of them.
```

**Replace with:**
```
`asyncio.gather()` — launches multiple async tasks (called coroutines) at the same time and waits for all of them.
```

---

## Change 5: Name the sequential → parallel conversion pattern (lower priority)

1 reviewer (OpenAI), blocks transfer per reviewer. Worth including because it's one sentence and directly actionable for students adapting the pattern to their own code.

Add the following sentence after the asyncio.gather intro line in Part 3:

```
In your own code, the conversion is usually simple: take several independent `await Runner.run(...)` calls and place those same coroutines inside one `await asyncio.gather(...)` call instead.
```
