

---

# M03C: Testing & Evaluating Agents — Recording Prep

## What this notebook teaches

This notebook teaches students how to systematically evaluate whether an agent actually works — not just in a demo, but across a range of inputs. It introduces three complementary evaluation techniques: deterministic pass/fail checks using string matching, rubric-based quality scoring using a judge agent, and head-to-head version comparison to catch regressions. The core message is that a demo is one carefully chosen input that works, and evaluation is what separates a working agent from one that just looks like it works.

## Concepts to explain on camera

- **Golden test set**
  - **Term:** A curated collection of inputs where you already know what the correct output should look like — each entry has a question and specific facts the answer must include or exclude.
  - **Analogy:** Like an answer key for an exam. You write the questions and answers first, then grade the agent's responses against them.
  - **What students need to understand:**
    - Each test case has `input`, `must_contain`, `must_not_contain`, and a `description`
    - You save this to a JSON file (`golden_tests.json`) and version-control it alongside your agent code
    - Start with 10–20 cases covering happy paths, edge cases, and known failure modes
    - When something breaks in production, add that failing input as a new test case so it never sneaks through again

- **Judge agent**
  - **Term:** A separate, stronger LLM (here `REASONING_MODEL` / `gpt-5`) that reads the agent's output and scores it against a rubric — essentially using AI to grade AI.
  - **Analogy:** Like having a senior colleague review a junior employee's work using a scoring checklist, rather than you reading every response yourself.
  - **What students need to understand:**
    - The judge is a completely separate `Agent` with its own instructions (the rubric) and structured output (`EvalResult`)
    - You use a stronger model for judging because evaluating quality is a harder reasoning task than answering product questions
    - The judge gets the ground truth (product catalog) alongside the agent's response so it can verify factual accuracy
    - Judge scores have some variance between runs — this is normal and expected with LLMs

- **Regression check**
  - **Term:** Running your full test suite before and after a change (new instructions, different model, updated schema) to confirm the change didn't break anything that was previously working.
  - **Analogy:** Like running your full unit test suite after a code refactor — you need to know nothing got worse even if some things got better.
  - **What students need to understand:**
    - Any change to instructions, model, tools, or output schema can cause regressions
    - A change that improves some tests but regresses others is a net loss
    - The version comparison pattern (Part 4) is exactly how you do this in practice
    - This pattern is referenced again in M09B where students use test sets to justify model upgrades with data

- **Rubric-based evaluation**
  - **Term:** Scoring agent output against a defined set of quality criteria (accuracy, helpfulness, clarity) with specific score meanings (5 = accurate, helpful, clear; 1 = wrong or misleading).
  - **Analogy:** Like a restaurant review rubric — instead of just "good" or "bad," you score food quality, service, and ambiance separately on a 1–5 scale.
  - **What students need to understand:**
    - Vague criteria produce inconsistent scores — "be strict" alone isn't enough; you need specific level definitions
    - Numeric scores from structured output (`EvalResult.score`) make comparison across runs and versions straightforward
    - The rubric lives in the judge agent's `instructions` string — changing the rubric changes what "good" means
    - Rubric eval complements pass/fail checks; pass/fail catches factual errors, rubric catches quality issues

- **Structured output (in the evaluation context)**
  - **Term:** Forcing the judge agent to return a Pydantic model (`EvalResult`) instead of free-form text, so every evaluation produces comparable, parseable fields.
  - **Analogy:** Like requiring a grader to fill in a standardized score sheet instead of writing a free-form paragraph — you can compare scores across hundreds of evaluations.
  - **What students need to understand:**
    - `EvalResult` has `score` (1–5 with `Field(ge=1, le=5)`), `factual_accuracy`, `helpful`, `clear` (all booleans), and `feedback` (string)
    - This was introduced in M03A-2 — here it's applied to the evaluation use case specifically
    - Without structured output, you'd have to parse free-form judge text to extract scores — fragile and error-prone

## Key SDK pattern

There is no single new SDK call introduced in this notebook — the core pattern is **applying `Agent`, `Runner.run`, and `output_type` to the evaluation use case**. The key structural pattern is the judge agent:

```python
class EvalResult(BaseModel):
    score: Annotated[int, Field(ge=1, le=5)]
    factual_accuracy: bool
    helpful: bool
    clear: bool
    feedback: str

judge_agent = Agent(
    name="EvaluationJudge",
    instructions=judge_instructions,
    model=REASONING_MODEL,
    output_type=EvalResult
)
```

- **`name="EvaluationJudge"`** — Identifies this agent in traces; useful when debugging which agent said what.
- **`instructions=judge_instructions`** — Contains the full 1–5 scoring rubric. This is where you define what each score level means. Vague instructions here produce inconsistent scores.
- **`model=REASONING_MODEL`** — Uses `gpt-5` instead of `gpt-5-mini` because evaluating quality requires stronger reasoning than answering product questions. This is a deliberate cost tradeoff.
- **`output_type=EvalResult`** — Forces the judge to return structured data with a numeric score, boolean flags, and feedback text. Without this, you'd get free-form text you'd have to parse.

The judge is called like any other agent, but with a carefully constructed input string that includes ground truth data, the original question, and the agent's response:

```python
judge_input = (
    f"Product catalog (ground truth):\n{catalog}\n\n"
    f"Question: {test_case['input']}\n\n"
    f"Agent response: {agent_output}\n\n"
    f"Evaluate this response. Use the product catalog to verify factual accuracy."
)
judge_result = await Runner.run(judge_agent, input=judge_input)
evaluation = judge_result.final_output  # This is an EvalResult instance
```

The `evaluation.score`, `evaluation.factual_accuracy`, etc. are directly accessible as typed Python attributes — no parsing needed.

## The demos and what each shows

**Setup and Agent Under Test:** The notebook creates a `product_agent` with a small hardcoded `PRODUCTS` dictionary (4 items: Wireless Headphones, Phone Case, Laptop Stand, USB-C Cable) and two tools (`get_product`, `list_products`). Emphasize on camera that this agent is deliberately simple — the point is to learn the evaluation framework, not build a complex agent. The product data is hardcoded so we have known ground truth to check against. Call out P002 (Phone Case) specifically — it has `stock: 0`, which is the out-of-stock case that test T02 checks.

**Part 1 — Building a Golden Test Set:** Five test cases are defined in `TEST_CASES`, each with `id`, `input`, `must_contain`, `must_not_contain`, and `description`. Walk through T01 (price lookup — must contain "49.99" and must NOT contain "149.99" or "19.99" to catch the agent mixing up products) and T02 (stock check — the `must_contain` list has multiple valid phrasings: "out of stock", "not in stock", "unavailable", "0" because the agent might word it differently). The test set is then saved to `golden_tests.json`. Emphasize on camera: this file should be committed to version control so you can trace regressions to specific code changes.

**Part 2 — Pass/Fail Checks:** The `check_output()` function does case-insensitive substring matching for `must_contain` and `must_not_contain`. The full test suite runs all 5 cases through `Runner.run(product_agent, input=...)` and reports pass/fail with specific failure reasons. Emphasize that this is deterministic — the same check always gives the same result for the same output. Point out on camera that when a test fails, the failure message tells you exactly what was missing (e.g., "Missing: '49.99'"). This is your first line of defense: fast, cheap (one API call per test), and catches factual errors.

**Part 3 — Rubric-Based Evaluation with a Judge Agent:** Only the first 3 test cases (`eval_cases = TEST_CASES[:3]`) are used here to save API costs. The judge agent receives the full product catalog as ground truth, the original question, and the agent's response — all concatenated into one input string. The key thing to emphasize: the judge can verify factual accuracy because it has the catalog. Without ground truth, it could only judge tone and clarity. Show the output format: star ratings, boolean flags (Accurate ✅, Helpful ✅, Clear ✅), and written feedback. Mention that judge scores may vary slightly between runs — this is normal LLM behavior, not a bug.

**Part 4 — Comparing Two Agent Versions:** Two agents are created: `agent_v1` (the original `product_agent`) and `agent_v2` with improved instructions that add "Lead with the direct answer", "suggest alternatives for out-of-stock items", and "Keep responses under 3 sentences." Both are run on the same 3 test cases, both get pass/fail checks AND judge scores, and results are compared side by side. The winner is determined by average judge score. Emphasize on camera: the instructions are the only variable — same model, same tools, same test cases. This is how you make data-driven decisions about prompt changes. Call out the final note in the notebook: "this comparison uses 3 test cases — real decisions need a larger test set."

## Gotchas worth knowing before recording

- **All `Runner.run` calls use `await`** — this notebook runs in an async Jupyter context. If you accidentally drop the `await`, you'll get a coroutine object instead of a result. Every cell that calls `Runner.run` must use `await Runner.run(...)`.
- **`must_contain` uses OR-style matching for T02** — the test case `T02` has `must_contain: ["out of stock", "not in stock", "unavailable", "0"]` but the `check_output()` function actually requires ALL of them to be present (AND logic). This could cause T02 to fail if the agent says "out of stock" but doesn't also say "0". Read the code carefully before recording: the function loops through `must_contain` and flags each missing phrase. The intent is that any ONE of these phrases is acceptable, but the implementation requires ALL. If T02 fails on camera, this is why — you may need to explain this as a design decision or acknowledge it as a limitation of simple string matching. *Update: looking more closely, "0" could appear in "0 units" in the tool output that the agent might echo, and "out of stock" is the status text. The agent's response might naturally include enough of these, but be prepared for a fail here.*
- **T05 ("highest rating") requires the agent to check ALL products** — this is the hardest test case because `get_product` only looks up one product at a time. The agent needs to call `list_products` or call `get_product` multiple times. If the agent doesn't do this reasoning step, T05 may fail or give the wrong product. This is actually a great teaching moment — it shows that test cases reveal agent reasoning gaps.
- **Judge score variance** — If you record Parts 3 and 4, the scores may differ from what you expect or from a previous run. Don't promise specific scores. Say "let's see what the judge thinks" rather than "this should get a 5."
- **Part 4 makes a LOT of API calls** — 3 test cases × 2 agent versions × (1 agent call + 1 judge call) = 12 API calls, with 6 of those hitting `REASONING_MODEL` (gpt-5). This cell will be noticeably slower and more expensive than the others. Mention on camera that this is why you use a subset of test cases during development.
- **The `catalog` variable is rebuilt in both Part 3 and Part 4** — it's the same string construction both times. Don't get confused thinking it's a different variable. It formats all products with prices, stock status, and ratings as ground truth for the judge.
- **`Annotated[int, Field(ge=1, le=5)]`** — Students may not have seen `Annotated` before. Briefly explain that `ge=1, le=5` means "greater than or equal to 1, less than or equal to 5" — it constrains the judge's score to a valid range. This was covered in M03A-1 (Pydantic Basics) so a quick callback is sufficient.
- **The `truncate_response()` helper in setup** — It's defined but only used once (in Part 2's test runner, `truncate_response(output, 120)`). Don't spend time on it; it just keeps printed output readable.
- **No explicit tool-use correctness check in the code** — The notebook outline mentions "Checking tool-use correctness" but the actual code only checks the final output text, not which tools were called. The judge agent implicitly evaluates whether the agent used tools correctly (via factual accuracy), but there's no `result.tool_calls` inspection. If a student asks, acknowledge this is a simplification — you could inspect the trace to see tool calls (forward reference to M07D).

## How it connects to adjacent notebooks

**Builds on M03A-1 and M03A-2 (Pydantic and Structured Outputs):** The `EvalResult` model uses `BaseModel`, `Field`, and `Annotated` from Pydantic, and `output_type=EvalResult` on the judge agent uses the structured output pattern from M03A-2. Say on camera: "Remember when we learned structured outputs? Here's a real use case — forcing the judge to return comparable scores instead of free-form text."

**Builds on M03B (Error Handling):** T04 tests graceful handling of a missing product (P999), which connects to the error handling patterns from M03B. Say: "In the last notebook we built error handling into tools — now we're testing that it actually works."

**Builds on M02A–M02C (Agent, Runner, tools, instructions):** The entire notebook uses `Agent`, `Runner.run`, `@function_tool`, and instruction design. These are all foundations from Module 2.

**Forward reference to M07D (Tracing):** The notebook explicitly says "When an eval fails and you can't tell why from the output alone, tracing shows the full tool call sequence — covered in M07D." Mention this on camera but don't dwell on it.

**Forward reference to M04D, M05E, M07E (Capstones):** Each capstone includes an evaluation exercise using the golden test set pattern from this notebook. Say on camera: "This evaluation framework isn't just for this module — every capstone project from here on asks you to evaluate your agent using these same patterns."

**Forward reference to M09B (Architecture Decisions):** M09B explicitly says "use your M03C test sets to justify model upgrades with data." The version comparison in Part 4 is exactly how students will do this. Mention: "When we get to architecture decisions later in the course, this is how you'll decide whether a more expensive model is worth it — data, not gut feeling."

**Next notebook is M04A (Web Search):** The course moves from foundational reliability patterns into built-in tools. Say on camera: "Now that you know how to build agents, handle errors, and evaluate whether they actually work, we're ready to give agents access to real capabilities — starting with web search."

---