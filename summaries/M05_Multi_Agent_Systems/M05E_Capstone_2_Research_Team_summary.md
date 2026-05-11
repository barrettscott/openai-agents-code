

---

# M05E: Capstone #2 — Multi-Agent Research Team — Recording Prep

## What this notebook teaches

This notebook teaches students how to build a complete multi-agent research pipeline that coordinates four specialist agents through explicit phases in Python code — parallel research/analysis, sequential writing, critique, and revision. It matters because it's the culmination of Module 5: students combine parallel execution (`asyncio.gather`), the debate/critique pattern, built-in tools (web search + code interpreter), and structured evaluation into a single working system. The problem it solves is showing that complex multi-agent workflows don't need a special orchestration framework — plain Python with `async/await` and sequential `Runner.run()` calls is enough.

## Concepts to explain on camera

- **Procedural orchestration / procedural pipeline**
  - **Term:** Instead of agents deciding who to call next (like handoffs), your Python code explicitly controls which agent runs, in what order, and what data flows between them.
  - **Analogy:** A film director calling "Action!" for each scene in sequence, rather than letting the actors decide what scene to shoot next.
  - **What students need to understand:**
    - The pipeline function `research_pipeline()` is just an `async def` — no special SDK orchestration object
    - Each phase's output (e.g., `research_findings`) is passed as a string into the next phase's input prompt
    - You can add, remove, or reorder phases by editing Python code
    - This contrasts with handoffs (M05A) where the agent decides who gets control

- **`asyncio.gather(return_exceptions=True)`**
  - **Term:** Runs multiple async tasks at the same time and waits for all of them to finish. The `return_exceptions=True` flag means if one task crashes, the others still complete — the exception is returned as a value instead of blowing up the whole call.
  - **Analogy:** Sending two research assistants to two different libraries simultaneously. If one library is closed, the other assistant still brings back their findings — you don't cancel the whole project.
  - **What students need to understand:**
    - Without `return_exceptions=True`, one failed agent would crash the entire `gather` and you'd lose both results
    - The returned `results` list contains either a successful result object or an Exception object — you must check with `isinstance(result, Exception)`
    - This is the pattern from M05C applied to a real pipeline
    - Fallback text like `"Web research unavailable for this run."` keeps downstream phases running

- **Partial-failure recovery**
  - **Term:** When one specialist agent fails (network error, API timeout, etc.), the pipeline detects the failure and continues with whatever data it has, rather than crashing entirely.
  - **Analogy:** A news team where the field reporter's satellite link goes down — the anchor still delivers the show using the studio analyst's data instead of going to a blank screen.
  - **What students need to understand:**
    - The `isinstance(research_result, Exception)` check is the detection mechanism
    - Fallback strings like `"Quantitative analysis unavailable for this run."` are injected so the Writer agent always has something to work with
    - This is "no silent degradation" — the pipeline prints a warning (`⚠️ Researcher failed:`) so you know something went wrong
    - Without this, a single API timeout would lose the entire pipeline run

- **Judge agent / rubric-based evaluation**
  - **Term:** A separate agent that scores the pipeline's output against defined criteria, producing a structured score rather than a subjective opinion.
  - **Analogy:** A teacher grading an essay with a rubric — clarity gets a score, evidence quality gets a score, and you get a pass/fail verdict.
  - **What students need to understand:**
    - The `ReportEval` Pydantic model forces the judge to return `score`, `reasoning`, and `passed` — not free-form text
    - The judge uses `REASONING_MODEL` (gpt-5) because evaluation requires deeper judgment
    - This is the exact M03C pattern applied to multi-agent output
    - Over time, you'd save these evaluations as a golden test set to catch regressions

- **`CodeInterpreterTool` with container config**
  - **Term:** A built-in tool that lets the agent write and execute Python code in a sandboxed environment. The `"container": {"type": "auto"}` config tells OpenAI to manage the container lifecycle automatically.
  - **Analogy:** Giving the analyst agent its own laptop to run calculations on — separate from your machine, so it can't break anything.
  - **What students need to understand:**
    - The `tool_config` dict is required for CodeInterpreterTool — it specifies container type
    - `"auto"` means OpenAI may reuse an existing container or spin up a new one (affects cost and latency)
    - The analyst uses this to compute derived metrics, not just generate text about numbers
    - This was introduced in M04C — here it's being used as part of a larger system

## Key SDK pattern

The primary pattern is **procedural multi-agent orchestration** — calling `Runner.run()` on different specialist agents in a controlled sequence, with `asyncio.gather()` for parallel phases. Here's the core structure from the notebook:

```python
async def research_pipeline(question):
    # Phase 1: Parallel
    results = await asyncio.gather(
        Runner.run(researcher_agent, input=question),
        Runner.run(
            analyst_agent,
            input=f"Analyze the quantitative aspects of this question: {question}"
        ),
        return_exceptions=True
    )
    
    # Check for failures
    research_result, analysis_result = results
    if isinstance(research_result, Exception):
        research_findings = "Web research unavailable for this run."
    else:
        research_findings = research_result.final_output

    # Phase 2: Sequential — writer gets both outputs
    writer_input = (
        f"Write a research report on: {question}\n\n"
        f"RESEARCH FINDINGS:\n{research_findings}\n\n"
        f"DATA ANALYSIS:\n{analysis_findings}"
    )
    write_result = await Runner.run(writer_agent, input=writer_input)
    
    # Phase 3: Sequential — critic reviews draft
    critique_result = await Runner.run(critic_agent, input=f"Critique this research report:\n\n{draft}")
    
    # Phase 4: Sequential — writer revises with critique
    revision_result = await Runner.run(writer_agent, input=revision_input)
```

Parameter-by-parameter:

- **`Runner.run(agent, input=...)`** — Runs a single agent to completion. `input` is a string prompt. Returns a result object where `.final_output` is the agent's text response. This is the same `Runner.run()` from M02A but called with `await` because we're in an async function.
- **`asyncio.gather(*coroutines, return_exceptions=True)`** — Takes multiple `Runner.run()` calls and runs them simultaneously. `return_exceptions=True` means a failed call returns its Exception object in the results list rather than crashing.
- **`results` unpacking** — `research_result, analysis_result = results` — order matches the order you passed the coroutines to `gather`.
- **`isinstance(result, Exception)`** — The check that makes partial failure recovery work. If True, the agent failed; substitute fallback text.
- **String composition for `writer_input`** — The "glue" between phases. Each phase's `.final_output` is assembled into a prompt string for the next agent. No special SDK data-passing mechanism — just f-strings.

## The demos and what each shows

**Phase 1 — Building Specialist Agents:** This cell defines four agents: `researcher_agent` (with `WebSearchTool()`), `analyst_agent` (with `CodeInterpreterTool`), `writer_agent` (no tools, just text generation), and `critic_agent` (no tools, review-only). Emphasize on camera that the Writer and Critic have no tools — they're pure text agents — while Researcher and Analyst each have exactly one built-in tool. Point out that the Critic instructions say "Do not rewrite — only critique" — this is the M05D debate pattern where roles are kept separate. Also highlight the `CodeInterpreterTool` config dict with `"container": {"type": "auto"}` since students haven't seen that specific syntax in this context.

**Phase 2 — The Pipeline Function (`research_pipeline`):** This is the heart of the capstone. Walk through the four phases on camera, emphasizing the parallel-then-sequential structure. Phase 1 runs Researcher and Analyst simultaneously via `asyncio.gather(return_exceptions=True)`. Then the failure-checking block demonstrates partial recovery — if either agent returned an Exception, fallback text is substituted. Phases 2–4 are sequential `await Runner.run()` calls where each phase's output feeds the next. Emphasize that this is "just Python" — no special orchestration SDK. The timing printouts (`time.time() - start`) give students visibility into how long each phase takes. The function returns a dict with every intermediate result, which matters for debugging and evaluation.

**Phase 3 — Running the Demo:** The demo asks `"What is the current state of open source AI models and how do they compare to proprietary models?"`. On camera, narrate the phase printouts as they appear — "Phase 1 is running Researcher and Analyst in parallel... now Phase 2, the Writer is synthesizing... Phase 3, the Critic reviews... Phase 4, the Writer revises." The final report is printed via `truncate_response()` (capped at 1200 chars). Emphasize the total time — probably 30–60 seconds — and point out that Phase 1's parallel execution saved time versus running both sequentially. The "What the Pipeline Did" markdown cell is a good on-camera recap.

**Evaluation — Judge Agent:** The `ReportEval` Pydantic model enforces structured scoring with `score` (1–5, constrained by `Field(ge=1, le=5)`), `reasoning`, and `passed`. The `judge_agent` uses `REASONING_MODEL` (gpt-5) with `output_type=ReportEval`. When run, it produces a structured evaluation of the final report. Emphasize on camera that this is the M03C judge pattern applied at the end of a real multi-agent pipeline — "evaluation isn't just a Module 3 thing, it follows you into every project." The score, pass/fail verdict, and reasoning are printed clearly.

## Gotchas worth knowing before recording

- **The demo cell uses `await` at the top level** — this works in Jupyter notebooks natively but would require `asyncio.run()` in a regular Python script. Don't try to explain this unless a student asks; just know it's a Jupyter convenience.
- **`result` is a plain dict, not a Pydantic model** — the pipeline function returns `{"question": ..., "research": ..., ...}`. The eval section accesses `result["final_report"]` with bracket notation. The *judge's* output (`evaluation`) is a `ReportEval` Pydantic object accessed with dot notation (`evaluation.score`). This dict-vs-object difference could confuse students if not pointed out.
- **The `truncate_response()` helper truncates at 1200 chars** — if the final report looks cut off on screen, that's intentional. Call it out: "I'm truncating for readability — the full report is stored in the `result` dict."
- **Code Interpreter container startup can add 10–20 seconds of latency** — if Phase 1 seems to hang, it's likely the container spinning up. Narrate this: "The code interpreter needs a container, so the analyst might be a bit slower to start."
- **Web search results vary between runs** — the final report will be different every time you record. Don't promise specific content. Say "your results will vary depending on what the web search finds."
- **Cost is real** — the notebook warns about this. Multiple model calls (researcher, analyst, writer, critic, writer again, judge) plus web search queries plus code interpreter container time. Budget a few dollars for recording takes.
- **The `Annotated[int, Field(ge=1, le=5)]` syntax** in `ReportEval` — students saw `Field` in M03A but the `Annotated` wrapper might look unfamiliar. Briefly explain: "This constrains the score to 1 through 5 — Pydantic will reject anything outside that range."
- **`writer_agent` is reused for both the initial draft (Phase 2) and the revision (Phase 4)** — same agent, different input. This is intentional and worth calling out: "We're reusing the same Writer agent — the instructions don't change, but the input does."
- **The `research_pipeline` function doesn't use `output_type`** — all four specialist agents return free-form text. Only the judge agent uses structured output. This is a deliberate design choice worth mentioning: "The pipeline works with text flowing between phases. We only need structured output at the evaluation step."
- **Exercise 1 scaffold repeats most of the pipeline code** — students might wonder why. Explain that the exercise adds Phase 5 to an otherwise identical pipeline. The repetition is intentional so they can focus on just the new phase.
- **Exercise 2 asks students to build exactly what the demo already showed** (the judge agent) — the difference is that Exercise 2 also asks them to save results to a JSON file for golden test set building. If students notice the overlap, acknowledge it: "Yes, I showed you the pattern — now you implement it yourself and add the persistence step."

## How it connects to adjacent notebooks

**What came before:** This capstone directly combines patterns from the four preceding Module 5 notebooks. On camera, call back to: "In M05A we learned handoffs — here we're using procedural orchestration instead, where Python decides who runs when. In M05B we saw agents-as-tools — here each specialist is called directly via `Runner.run()`. In M05C we built the `asyncio.gather` pattern — that's exactly what Phase 1 uses for parallel research and analysis. And in M05D we built the proposer/critic debate loop — that's Phases 2–4 where the Writer drafts, the Critic reviews, and the Writer revises." Also call back to Module 4 built-in tools: "The Researcher uses `WebSearchTool` from M04A, the Analyst uses `CodeInterpreterTool` from M04C." And for evaluation: "The judge agent applies the rubric-based evaluation pattern from M03C — that module wasn't just an exercise, it's how we verify that multi-agent pipelines actually work."

**What comes next:** Module 6 introduces memory and state. Bridge with: "Right now, every pipeline run starts from scratch — the agents have no memory of previous research. In M06A, we'll give agents sessions so they remember conversation context, and in M06B we'll add persistent memory with SQLite so agents can remember things across separate runs." This also sets up M07 (safety/guardrails): "We flagged that web search results are untrusted input — in M07B we'll address prompt injection and what happens when untrusted data flows through a pipeline like this."

---