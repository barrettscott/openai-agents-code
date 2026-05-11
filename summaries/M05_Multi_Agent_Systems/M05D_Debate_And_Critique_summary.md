

---

# M05D: Debate & Critique Pattern — Recording Prep

## What this notebook teaches

This notebook teaches students how to improve agent output quality by separating the roles of proposer and critic into distinct agents, then chaining their outputs together. It solves the problem that a single agent produces its first reasonable answer and stops — it has no mechanism to challenge its own assumptions. Students learn three escalating patterns: a basic proposer/critic pipeline (Part 2), a critique loop with a quality gate and iteration limit (Part 3), and a true debate with independent opposing positions plus synthesis (Part 4).

## Concepts to explain on camera

- **Proposer / Critic Architecture:** A design pattern where one agent generates output (the proposer) and a separate agent evaluates that output for weaknesses (the critic). The proposer then revises based on the critique.
  - **Analogy:** Like having a writer and an editor — the writer drafts, the editor red-pens it, and the writer revises. The editor never rewrites; they only point out problems.
  - **What students need to understand:**
    - The critic agent's instructions explicitly say "Do not rewrite — only critique" — this separation is what makes the pattern work
    - The proposer gets the critique as part of its next input, so it has specific feedback to act on
    - This is more effective than telling a single agent to "write something good and then improve it"
    - The agents are completely independent — no handoffs, no shared state, just sequential `Runner.run()` calls

- **Critique Loop Termination Rules:** The logic that determines when a refine-and-retry loop should stop — either the output meets a quality threshold or a maximum iteration count is reached.
  - **Analogy:** Like setting an alarm for "keep editing until your essay scores an 8/10 or you've revised it 3 times, whichever comes first." Without the alarm, you'd edit forever.
  - **What students need to understand:**
    - `max_iterations` is a hard safety limit — without it, the loop runs indefinitely and racks up API costs
    - `quality_threshold` is the "good enough" gate — if the judge scores the output at or above this, the loop stops early
    - A threshold of 9 or 10 may never be met, causing unnecessary iterations — 7 or 8 is more practical
    - Each iteration means multiple API calls (judge + critic + writer), so costs multiply per loop

- **Quality Gate (Judge Agent with Structured Output):** An agent that scores output on a numeric scale using `output_type=QualityScore`, returning a Pydantic model instead of free-form text, so the loop can programmatically compare the score to a threshold.
  - **Analogy:** Like a rubric-based grader who gives you a number, not a paragraph — the loop needs a number it can compare with `>=`, not a paragraph it has to interpret.
  - **What students need to understand:**
    - The judge uses `output_type=QualityScore` so `score_result.final_output.score` is an integer, not a string
    - This is the structured outputs pattern from M03A-2 applied in a real workflow
    - The judge uses `REASONING_MODEL` (gpt-5) because quality evaluation requires judgment, not just generation
    - `Annotated[int, Field(ge=1, le=10)]` constrains the score to 1–10 using Pydantic validation

- **Output Chaining:** The technique of taking `result.final_output` from one `Runner.run()` call and embedding it in the input string of the next `Runner.run()` call, creating a pipeline where each agent builds on the previous one's work.
  - **Analogy:** Like a relay race where each runner passes a baton — the baton is `result.final_output`, and you hand it to the next agent by putting it in their input string.
  - **What students need to understand:**
    - There is no special SDK mechanism — it's literally f-string concatenation: `f"Critique this:\n\n{first_draft}"`
    - Each agent gets a fresh `Runner.run()` call with no memory of previous calls
    - The full context (original draft + critique) must be included in the revision input or the writer won't know what to address
    - This is the same `result.final_output` pattern from M02A, just used in sequence

- **Synthesis Agent:** A third agent that receives the outputs from both sides of a debate and produces a balanced conclusion, rather than picking a winner.
  - **Analogy:** Like a judge in a formal debate who's listened to both sides and writes a summary of the strongest points from each.
  - **What students need to understand:**
    - The synthesizer receives all three outputs (advocate position, skeptic position, rebuttal) in a single input
    - It uses `REASONING_MODEL` because synthesizing multiple arguments requires deeper reasoning
    - Running advocate and skeptic independently (not sequentially) produces richer arguments because neither is reacting to the other
    - The rebuttal step forces the advocate to engage with real counterarguments

## Key SDK pattern

This notebook does not introduce a new SDK feature. The core pattern is **output chaining via sequential `Runner.run()` calls** — the same `Agent` and `Runner.run()` from M02A, applied in sequence:

```python
# Step 1: Proposer generates
draft_result = await Runner.run(writer_agent, input=topic)
first_draft = draft_result.final_output

# Step 2: Critic reviews (proposer's output becomes critic's input)
critique_input = f"Please critique this explanation:\n\n{first_draft}"
critique_result = await Runner.run(editor_agent, input=critique_input)
critique = critique_result.final_output

# Step 3: Proposer revises (both draft AND critique become revision input)
revision_input = (
    f"Here is your original explanation:\n\n{first_draft}\n\n"
    f"Here is the editor's critique:\n\n{critique}\n\n"
    f"Please rewrite the explanation addressing all the feedback."
)
revision_result = await Runner.run(writer_agent, input=revision_input)
revised_draft = revision_result.final_output
```

- **`writer_agent` and `editor_agent`** — Standard `Agent` objects with different `instructions`. The writer creates content; the editor only critiques (its instructions say "Do not rewrite — only critique").
- **`result.final_output`** — The string output from each run. This is the "baton" being passed between agents.
- **The f-string input** — This is where the chaining happens. The revision input includes both the original draft AND the critique so the writer has full context.
- **`await Runner.run()`** — Each call is independent. The agents don't share conversation history or sessions. All context must be passed explicitly in the input string.

The secondary pattern is the **structured quality gate**:

```python
class QualityScore(BaseModel):
    score: Annotated[int, Field(ge=1, le=10)]

quality_judge_agent = Agent(
    name="QualityJudge",
    instructions=judge_instructions,
    model=REASONING_MODEL,
    output_type=QualityScore
)
```

- **`output_type=QualityScore`** — Forces the judge to return a Pydantic model with a `.score` field, so the loop can do `if score >= quality_threshold`.
- **`Annotated[int, Field(ge=1, le=10)]`** — Pydantic validation constraining the score to integers between 1 and 10.
- **`model=REASONING_MODEL`** — Uses gpt-5 for the judge because quality evaluation is a reasoning-heavy task.

## The demos and what each shows

**Part 2 — Basic Proposer / Critic (three cells: Step 1, Step 2, Step 3):** This is the simplest version of the pattern. The `writer_agent` writes a first draft explaining "Explain what an API is." The `editor_agent` critiques it (looking for unclear language, weak examples, logical gaps). Then the `writer_agent` revises using both the original draft and the critique as input. Run each cell separately and pause to read the output. Emphasize on camera: the critic's instructions say "Do not rewrite — only critique" — this role separation is the whole point. Also emphasize that the revision input includes BOTH the original and the critique via f-string, because the writer agent has no memory of its first draft. After running all three, compare the first draft to the revised draft and point out specific improvements the critique caused.

**Part 3 — The Critique Loop with Termination Rules:** This introduces the `critique_loop()` function that wraps the proposer/critic pattern in a `for` loop with two stopping conditions: `quality_threshold=8` and `max_iterations=3`. A `QualityJudge` agent with `output_type=QualityScore` scores each draft on a 1–10 scale. If the score meets the threshold, the loop breaks early; otherwise, the editor critiques and the writer revises. The demo runs on "Explain what recursion is in programming." Emphasize on camera: look at which iteration it stops at. If it hits the threshold on iteration 1, the loop saved two unnecessary revision cycles. If it runs all 3 iterations, the threshold might have been too high. Call out the cost implication — each iteration is three API calls (judge + editor + writer), so `max_iterations=3` means up to 10 API calls total (1 initial draft + 3×3). This is why the ⚠️ warning about always setting `max_iterations` matters.

**Part 4 — True Debate with Two Opposing Positions:** This is a different pattern — not proposer/critic, but advocate vs. skeptic with synthesis. The debate topic is "Microservices are better than monoliths for most software teams." Round 1 runs the `advocate_agent` and `skeptic_agent` in parallel using `asyncio.gather()` since they don't depend on each other — call back to M05C here. Round 2 has the advocate rebut the skeptic's arguments (sequential, because it needs the skeptic's output). Round 3 feeds all three outputs (advocate position, skeptic position, rebuttal) to the `synthesis_agent` which uses `REASONING_MODEL` to produce a balanced conclusion. Emphasize on camera: the key insight is that independent positions surface stronger tradeoffs than sequential critique. The skeptic isn't reacting to the advocate's argument — it's making its own independent case against the topic. Also note that `truncate_response()` is used here to keep printed output readable, with a max of 600 chars per position.

## Gotchas worth knowing before recording

- **The `truncate_response()` helper truncates display, not the actual data.** It's defined in the setup cell and only used for `print()` output. The full text is still passed to the next agent. If a student sees the truncation message ("Truncated from X chars") they might think the agent received truncated input — clarify this proactively.

- **The `writer_agent` is reused across Part 2 and Part 3** — it's defined once and used for both the basic pipeline and the critique loop. If you re-run the Part 2 cells after running Part 3, it still works because agents are stateless. But don't skip the Part 2 agent definition cells — the Part 3 loop depends on `writer_agent` and `editor_agent` being defined.

- **The quality judge may score the initial draft at 8+ immediately**, causing the loop to stop at iteration 1 with no critique/revision cycle. This looks anticlimactic but is correct behavior — the threshold was already met. If this happens on camera, say "Great — the first draft was already good enough. Let's lower the threshold to 6 and re-run to see the revision in action," or raise it to 9.

- **`Annotated[int, Field(ge=1, le=10)]` uses `typing.Annotated`** which is imported alongside `BaseModel` and `Field` from pydantic. The import is `from typing import Annotated`. Students who haven't seen `Annotated` before may be confused — briefly explain it adds metadata to the type hint that Pydantic uses for validation.

- **Part 4 uses `asyncio.gather()` for Round 1 only.** Rounds 2 and 3 are sequential `await Runner.run()` calls because they depend on prior outputs. Students might wonder why you don't run everything in parallel — the answer is data dependency. The rebuttal needs the skeptic's output; the synthesis needs all three.

- **The debate pattern uses different agent names than the critique pattern** (`advocate_agent`, `skeptic_agent`, `synthesis_agent` vs. `writer_agent`, `editor_agent`, `quality_judge_agent`). These are independent — the debate agents don't reference the critique agents and vice versa. Don't accidentally call the wrong variable name if you ad-lib.

- **`REASONING_MODEL` ("gpt-5") is used for both `quality_judge_agent` and `synthesis_agent`**, while all other agents use `MODEL` ("gpt-5-mini"). This is deliberate — judging quality and synthesizing multiple arguments are reasoning-heavy tasks. Mention this as a practical model selection choice; it previews the model selection framework in M09B.

- **No structured output on the debate agents.** The advocate, skeptic, and synthesizer all return free-form text. Only the `QualityJudge` uses `output_type`. Students might ask why — the debate agents don't need structured output because their text goes to other agents, not to programmatic logic.

- **The notebook uses `await Runner.run()` (async) throughout.** This works natively in Jupyter but not in regular Python scripts without `asyncio.run()`. If a student copies this code to a `.py` file, it will fail. This is the same async pattern from M05C.

- **The `critique_loop` function returns `current_draft` but the calling cell assigns it to `final_output`** — a variable name that also appears as `result.final_output` in other cells. These are completely different things. `final_output` here is just a local variable name for the function's return value.

## How it connects to adjacent notebooks

**Builds on M05C (Parallel Execution):** Part 4's `asyncio.gather()` for running advocate and skeptic in parallel is exactly the pattern from M05C. On camera, say something like: "Remember `asyncio.gather` from the parallel execution notebook? We're using it here because the advocate and skeptic don't depend on each other — they can argue at the same time." Also builds on M05A and M05B conceptually — handoffs and agents-as-tools are different ways agents interact; debate/critique is yet another coordination pattern. Mention that unlike handoffs (where control transfers) or agents-as-tools (where one agent calls another), debate/critique uses plain sequential `Runner.run()` calls with output chaining.

**Builds on M03A-2 (Structured Outputs):** The `QualityScore` Pydantic model with `output_type=` is the structured outputs pattern from M03A-2. On camera: "This is why we learned structured outputs — so the judge returns a number the loop can compare, not a paragraph we'd have to parse."

**Builds on M02A (Agent foundations):** The entire notebook is built on `Agent`, `Runner.run()`, and `result.final_output` from M02A. No new SDK features are introduced.

**Enables M05E (Capstone #2 — Multi-Agent Research Team):** The capstone combines handoffs, agents-as-tools, parallel execution, AND critique into a full pipeline with a Critic agent as one of the phases. On camera, close with: "Next up, we combine everything from Module 5 — handoffs, parallel execution, and this debate pattern — into a full multi-agent research team." The critique loop pattern directly appears in the capstone's sequential writing/critique/revision phases.

---