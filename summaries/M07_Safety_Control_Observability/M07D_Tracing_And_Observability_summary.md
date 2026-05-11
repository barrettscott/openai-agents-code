# M07D: Tracing & Observability — Recording Prep

## What this notebook teaches

Every `Runner.run()` call in the OpenAI Agents SDK automatically creates a complete trace — no setup code required. This notebook teaches students how to find those traces in the OpenAI dashboard, read them to understand what an agent actually did (tool calls, arguments, token counts, latency), and use them systematically to debug bad runs and compare model performance. It closes the loop between the evaluation patterns from M03C and real observability: when something goes wrong, the trace shows exactly where.

## Concepts to explain on camera

- **Trace:** A complete recording of everything that happened during a single `Runner.run()` call — every model call, every tool call, every input and output, with timing and token counts attached.
  - **Analogy:** A flight recorder for your agent. After a crash (or just a weird answer), you open the black box and replay every decision the agent made, step by step.
  - **What students need to understand:**
    - Traces are created automatically — there is zero configuration
    - You view them at `platform.openai.com` under Logs then Traces
    - Each trace covers one complete `Runner.run()` execution
    - Traces are the ground truth for debugging, cost estimation, and performance benchmarking

- **Span:** One recorded step inside a trace. Each tool call, each model call, and each handoff gets its own span with its own timing and token data.
  - **Analogy:** If the trace is a timeline of the whole flight, a span is one leg of the journey — takeoff, cruising, landing are each separate spans you can inspect individually.
  - **What students need to understand:**
    - A simple run might have 2-3 spans (one model call, one tool call); a complex run will have many more
    - Each span shows its own latency and token count, so you can find the bottleneck
    - Red spans indicate errors — click into them to see the exact failure point
    - More spans (tool calls) than expected usually points to an instruction problem

- **Token usage (input tokens / output tokens):** The count of text units the model consumed (input) and generated (output) during each model call. This is what you get billed for.
  - **Analogy:** Think of it like a metered taxi — input tokens are the distance to the pickup, output tokens are the distance of the ride. The trace shows you the exact meter reading, not an estimate.
  - **What students need to understand:**
    - Token counts in the trace are the ground truth for cost — not estimates, actual numbers
    - Large tool outputs get included in the next model call's context, which can balloon input token counts
    - Comparing token usage between `MODEL` and `REASONING_MODEL` on the same query is how you make data-driven model selection decisions
    - A spike in token usage compared to a baseline is a production signal worth investigating

## Key SDK pattern

This notebook does not introduce a new SDK method — the key pattern is that tracing is implicit in the existing `Runner.run()` call:

```python
result = await Runner.run(inventory_agent, input="How many laptops do we have in stock right now?")
```

There is no tracing parameter to set, no decorator to add, no configuration object. Every `Runner.run()` call automatically records:
- The full input and output
- Every tool call span — name, arguments, return value, duration
- Every model call span — exact prompt sent, response received
- Token usage per step (input and output tokens)
- Total latency and per-step latency
- Handoff spans in multi-agent runs
- Errors and the exact step where they occurred

The student's job is not to write tracing code but to know where to look (`platform.openai.com` -> Logs -> Traces) and what to look for at each span. The notebook provides explicit checklists for this (printed after each run).

## The demos and what each shows

**Part 1 — Generating Traces Worth Reading:** Two runs against an `inventory_agent` with three tools (`search_inventory`, `check_stock`, `get_category_report`) and an `INVENTORY` dict containing laptop, headphones, notebook, and desk lamp. Run 1 asks "How many laptops do we have in stock right now?" — a focused question that should produce exactly one `check_stock` call returning "5 units — In stock." Run 2 asks "Give me a full electronics report and flag anything out of stock" — a broader query that should call `get_category_report` and possibly `check_stock` as a follow-up. Emphasize on camera: comparing these two traces in the dashboard makes the cost of query complexity concrete — Run 2 will show more spans, more tokens, and higher latency. Walk through the printed checklist items on screen.

**Part 2 — Debugging a Bad Run:** A `BuggyInventoryAgent` is created with deliberately broken instructions: "For ALL questions — including specific stock counts — use search_inventory. Do not use check_stock or get_category_report under any circumstances." Both the buggy and correct agents are run on the same input: "I need the exact stock count for laptops — how many units do we have?" The buggy agent calls `search_inventory` (which returns fuzzy price-and-stock strings) instead of `check_stock` (which returns the exact count). Emphasize the six-step debug workflow printed in the notebook: find the trace, check tool calls, check tool inputs, check tool outputs, check the LLM span, identify the mismatch. This is where you tie it back to M03C — traces show exactly where a run diverged from the expected path that a golden test set would catch.

**Part 3 — Comparing Latency and Token Usage:** Two agents with identical instructions and tools but different models — `InventoryAnalyst_mini` using `gpt-5-mini` and `InventoryAnalyst_full` using `gpt-5` — are run on the same `analysis_query`: "Review all inventory categories. Identify which products need restocking and prioritize them by urgency. Explain your reasoning." The notebook prints wall-clock time and response length for each, and the comparison checklist directs students to the dashboard to compare token counts and latency. Emphasize on camera: this is how you make the MODEL vs REASONING_MODEL decision with data, not intuition. The `truncate_response` helper clips output at 1200 chars for readability.

## Gotchas worth knowing before recording

- Traces appear in the dashboard with a 15-30 second delay — if you run a cell and immediately switch to the browser, the trace may not be there yet. Mention this on camera so students do not panic.
- The notebook uses `await Runner.run()` (async) — this works natively in Jupyter but will not work in a plain `.py` script without an async entry point. Students who copy code into scripts will hit errors.
- The `INVENTORY` dict has a `"desk lamp"` with `stock: 0` — this is intentional for the "flag anything out of stock" demo. Do not fix it thinking it is a bug.
- The `check_stock` tool has a plural-stripping fallback (`if key.endswith("s"): product = INVENTORY.get(key[:-1])`) — if the model passes "laptops" instead of "laptop," it still works. Worth mentioning briefly so students see it is a defensive coding pattern, not magic.
- The buggy agent demo depends on the model actually obeying the restrictive instructions. If the model ignores the instructions and calls `check_stock` anyway, the debugging demo falls flat. This is unlikely with `gpt-5-mini` but worth being aware of.
- The `REASONING_MODEL` run in Part 3 will be noticeably slower and more expensive. Have the dashboard open and ready so you can show the token difference without dead air while waiting for the response.
- Model names are `gpt-5-mini` and `gpt-5` — confirm these match the current OpenAI model IDs at recording time.

## How it connects to adjacent notebooks

**Before (M07C: Human-in-the-Loop):** M07C introduced approval workflows and confidence-based escalation. You can mention on camera: "Now that we have guardrails and human-in-the-loop from the last few notebooks, tracing lets us verify all of those controls are actually firing when they should."

**Before (M03C: Testing & Evaluating Agents):** The debugging section explicitly references the golden-test-set evaluation pattern from M03C. On camera, call back to it: "Remember the golden test set from Module 3? Traces show you exactly where a run diverged from the expected path — evaluation tells you something is wrong, tracing tells you why."

**Before (M05A: Handoffs):** The "What Tracing Captures" section mentions handoff spans in multi-agent runs. On camera: "If you built multi-agent systems in Module 5, every handoff shows up as its own span in the trace."

**After (M07E: Capstone #3 — Production Customer Service Agent):** The capstone combines multi-tool agents, sessions, guardrails, human-in-the-loop approval, and tracing into a single production-style project. Set this up on camera: "Everything we have covered in Module 7 — guardrails, prompt injection, human-in-the-loop, and now tracing — comes together in the capstone."
