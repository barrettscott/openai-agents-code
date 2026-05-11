

---

# M05C: Parallel Execution — Recording Prep

## What this notebook teaches

This notebook teaches students how to run multiple agents simultaneously using `asyncio.gather()` to cut wall-clock time when tasks don't depend on each other. It matters because real agent workflows often involve independent specialist agents (market analysis, tech analysis, risk analysis) that waste time running one-after-another when they could all fire at once. The notebook covers the full "fan-out / fan-in" pattern — launch specialists in parallel, merge results with a synthesis agent — plus the critical error-handling patterns needed when one parallel task fails or hangs.

## Concepts to explain on camera

- **Coroutine / `async` / `await`:**
  - A coroutine is a function defined with `async def` that can pause while waiting for something (like an API call) without blocking other work. `await` is the keyword that says "pause here until this finishes."
  - **Analogy:** Ordering food at a restaurant. You place your order (`await`) and sit down — you're not standing at the kitchen door blocking everyone else. The waiter comes back when it's ready.
  - **What students need to understand:**
    - `Runner.run()` is a coroutine — it returns something you can `await` or pass to `gather()`
    - When you `await` a coroutine, execution pauses on that line until the result comes back
    - In Jupyter notebooks, you can use `await` directly at the top level (no need for `asyncio.run()`)
    - The notebook's sequential code uses `await` three times in a row — each one blocks until finished

- **`asyncio.gather()`:**
  - A function that takes multiple coroutines, starts them all at the same time, and waits until every one finishes. Returns results in the same order you passed them in.
  - **Analogy:** Sending three emails at once instead of waiting for a reply to each before sending the next. All three are "in flight" simultaneously.
  - **What students need to understand:**
    - You pass unawaited coroutines — `Runner.run(agent, input=topic)` not `await Runner.run(agent, input=topic)`
    - Results come back in input order, not completion order
    - By default, if any one task raises an exception, the whole `gather()` fails and you lose everything
    - `return_exceptions=True` changes that behavior — exceptions become values in the results list

- **`asyncio.wait_for()`:**
  - Wraps a single coroutine with a deadline. If the coroutine doesn't finish in time, it raises `asyncio.TimeoutError`.
  - **Analogy:** Setting a timer on the microwave. If the food isn't done when the timer goes off, the microwave stops — it doesn't wait forever.
  - **What students need to understand:**
    - `gather()` has no built-in timeout — `wait_for()` fills that gap
    - You wrap each individual task: `asyncio.wait_for(Runner.run(...), timeout=10)`
    - Catching `asyncio.TimeoutError` lets you return a fallback value instead of crashing
    - You decide the policy: skip, use a default, or abort the whole workflow

- **Fan-out / Fan-in:**
  - A two-phase pattern: "fan out" launches multiple parallel tasks, "fan in" collects all results and merges them into a single output.
  - **Analogy:** A manager assigns three analysts to research different aspects of a topic simultaneously (fan out), then reads all three reports and writes an executive summary (fan in).
  - **What students need to understand:**
    - Phase 1 (fan out) uses `asyncio.gather()` to run specialists in parallel
    - Phase 2 (fan in) passes all outputs into a synthesis agent as a single combined prompt
    - The synthesis agent is sequential — it must wait for all parallel results before it can start
    - This is the dominant pattern for parallel agent workflows

- **`return_exceptions=True`:**
  - An argument to `asyncio.gather()` that changes how failures are handled. Instead of crashing the whole batch, exceptions are returned as values alongside successful results.
  - **Analogy:** A teacher collecting homework from 30 students. Default behavior: if one student's paper is missing, throw out all 30. With `return_exceptions=True`: collect what you can, and note which students didn't turn in their work.
  - **What students need to understand:**
    - Without it, one failed agent kills the entire `gather()` and you lose results from agents that succeeded
    - With it, you check each result using `isinstance(result, Exception)`
    - You still need to decide what to do: proceed with partial results, retry, or abort
    - This is essential for production — the capstone in M05E uses this exact pattern

## Key SDK pattern

The primary pattern is `asyncio.gather()` wrapping multiple `Runner.run()` calls:

```python
market_result, tech_result, risk_result = await asyncio.gather(
    Runner.run(market_agent, input=topic),
    Runner.run(tech_agent, input=topic),
    Runner.run(risk_agent, input=topic)
)
```

**Parameter-by-parameter:**
- **`Runner.run(market_agent, input=topic)`** — this is the same `Runner.run()` from M02A, unchanged. The key is that you do NOT `await` it here — you pass the raw coroutine to `gather()` so it can manage the timing.
- **`market_result, tech_result, risk_result`** — tuple unpacking. `gather()` guarantees results come back in the same order as inputs: first result matches `market_agent`, second matches `tech_agent`, third matches `risk_agent`. This is true regardless of which agent finishes first.
- **`await asyncio.gather(...)`** — the single `await` here pauses until ALL three coroutines complete. Under the hood, all three API calls are in flight simultaneously.

With error handling, add one argument:
```python
results = await asyncio.gather(
    bad_task("Python"),
    bad_task("SKIP"),
    bad_task("JavaScript"),
    return_exceptions=True
)
```
- **`return_exceptions=True`** — exceptions become values in the `results` list instead of being raised. You then check each with `isinstance(result, Exception)`.

## The demos and what each shows

**Part 2 — Sequential Baseline:** Three agents (`market_agent`, `tech_agent`, `risk_agent`) analyze the topic `"Building an AI-powered personal finance app"` one at a time using three separate `await Runner.run()` calls. The elapsed time is captured in `sequential_time`. This exists purely to establish a baseline — emphasize the total time and the fact that each agent had to wait for the previous one to finish before starting. Point out that these three tasks are completely independent — no agent needs another's output.

**Part 3 — Parallel with `asyncio.gather()`:** The exact same three agents analyzing the exact same topic, but now all three `Runner.run()` calls are passed into a single `asyncio.gather()`. The speedup line `sequential_time / parallel_time` shows approximately 3x improvement. Emphasize on camera: same agents, same prompts, same results, dramatically less wall time. The "Why This Works" callout is worth reading aloud — all three API calls are in flight at the same time.

**Part 4 — Parallel + Merge (Fan-out / Fan-in):** This is the real-world pattern. Phase 1 runs the three specialists in parallel. Phase 2 creates a `synthesis_agent` that receives all three outputs as a single combined prompt (`synthesis_input` is a formatted string with labeled sections: `MARKET ANALYSIS`, `TECHNICAL ANALYSIS`, `RISK ANALYSIS`). The synthesis agent produces an executive summary. Emphasize the two-phase timing: Phase 1 is parallel (fast), Phase 2 is sequential (must wait for Phase 1). This is the pattern students will use in the M05E capstone.

**Part 5 — Cost & Speed Tradeoffs (markdown only):** No code to run. Walk through the table: latency drops from `A + B + C` to `max(A, B, C)`, but token cost and API call count are identical. Mention the exception: several mini-model agents in parallel can beat one reasoning-model call on both speed and cost. This connects to model selection in M09B.

**Part 6 — When One Task Fails:** Two cells demonstrate the difference. First cell: `bad_task("SKIP")` raises a `ValueError`, and the entire `gather()` fails — Python and JavaScript results are lost. Second cell: same tasks with `return_exceptions=True` — now all three return, and the `isinstance(result, Exception)` check separates successes from failures. Third cell shows the partial result handling pattern: count good vs failed, decide whether there are enough results to proceed, and if so, pass them to a `partial_synthesis_agent` with a note about which topics were unavailable. Emphasize that `failed` topics are explicitly communicated to the synthesis agent — no silent degradation.

**Part 7 — Timeout and Fallback:** Introduces `run_with_timeout()`, a helper that wraps `Runner.run()` inside `asyncio.wait_for()` with a `timeout_seconds` parameter. Returns `None` on timeout instead of crashing. The demo uses `slow_agent` with a simple prompt (`"What is 2 + 2?"`) and a generous 15-second timeout, so it will succeed. Mention on camera that students should try lowering `timeout_seconds` to 1-2 seconds with a more complex prompt to see the fallback branch (`"Analysis unavailable due to timeout."`). The pattern is: wrap each task, check for `None`, supply a fallback.

## Gotchas worth knowing before recording

- **Sequential timing varies between runs.** The `sequential_time` and `parallel_time` values will differ each recording. The speedup ratio should be roughly 2-3x but might be lower if OpenAI is fast that day or rate limits kick in. Don't promise "exactly 3x" — say "roughly 3x" or "close to 3x."

- **Rate limits can make parallel look no faster than sequential.** If the OpenAI account has low rate limits (e.g., Tier 1), three simultaneous requests might get throttled. The troubleshooting section mentions this. If it happens on camera, acknowledge it: "Rate limits can throttle concurrent requests — this is expected behavior, not a bug."

- **The `bad_task("SKIP")` demo uses a simulated failure, not a real API error.** The `ValueError` is raised in Python before any API call happens. Make that clear — in production, the exception might be an API timeout, a rate limit error, or a network failure. The pattern is the same regardless of the exception type.

- **The timeout demo is set to succeed on purpose.** The `timeout_seconds=15` with `"What is 2 + 2?"` will almost certainly complete in time. The notebook comment says to lower it to 1-2 seconds to see the fallback. If you want to show the fallback branch on camera, either lower the timeout or give the agent a much more complex prompt.

- **Don't accidentally `await` inside `gather()`.** The troubleshooting section calls this out: pass `Runner.run(agent, input=topic)` not `await Runner.run(agent, input=topic)`. If you `await` first, you get a resolved result, not a coroutine, and `gather()` has nothing to run concurrently. This is the #1 mistake students will make.

- **Tuple unpacking must match agent order.** `market_result, tech_result, risk_result` must be in the same order as the `Runner.run()` calls inside `gather()`. If you swap the agents inside `gather()` but not the variable names, you'll silently assign the wrong results to the wrong variables.

- **The `truncate_response()` helper truncates at 1200 chars.** If agent outputs are short, you won't see the truncation message. If they're long, you'll see `"... 💡 (Truncated from X chars)"`. Either way it's fine — just be aware it's there so you're not surprised by the output format.

- **`topics` list must match the order of `bad_task()` calls in the `zip()`.** The variable `topics = ["Python", "SKIP", "JavaScript"]` is defined separately from the `gather()` call. If you reorder the `bad_task()` calls during a live edit, the `zip()` labels will be wrong.

- **No slides-only M05C-1 recording required during this notebook.** The async conceptual foundation is a separate slides/video module (M05C-1). This notebook opens with a brief async recap in markdown but doesn't teach async from scratch. If students are confused, point them to M05C-1.

## How it connects to adjacent notebooks

**Builds on M05A (Handoffs) and M05B (Agents as Tools):** Both introduced multi-agent patterns where agents interact sequentially — one hands off to another, or an orchestrator calls agents one at a time. Say on camera: "In M05A and M05B, agents took turns. Now we're running them simultaneously." The key distinction: handoffs and agent-as-tool are for when Agent B needs Agent A's output; parallel is for when agents are independent.

**Builds on M02A (How Agents Work):** `Runner.run()` and `result.final_output` are the same patterns from Module 2. Say on camera: "Same `Runner.run()` call you've been using since Module 2 — the only difference is we're passing multiple of them to `asyncio.gather()` instead of awaiting them one at a time."

**Connects to M05C-1 (Async Python Basics — slides):** The slides module covers the conceptual foundation of async/await. The notebook's "Async Recap" section is intentionally brief because the concept was covered in slides. If recording order allows, mention: "If async syntax is new, make sure you've watched the async basics video first."

**Directly enables M05D (Debate & Critique Pattern):** The next notebook puts agents in dialogue. Say on camera: "Next, instead of running agents in parallel on different tasks, we'll have agents challenge each other's outputs — debate and critique."

**Directly enables M05E (Capstone #2 — Multi-Agent Research Team):** The capstone uses `asyncio.gather()` with `return_exceptions=True` for parallel research and analysis phases, followed by sequential writing, critique, and revision. The fan-out/fan-in pattern from Part 4 and the partial result handling from Part 6 are both used directly. Say on camera: "Everything you just learned — parallel execution, partial result handling, the fan-out/fan-in pattern — you'll put it all together in the capstone."

---