# M07A: Guardrails — Recording Prep

## What this notebook teaches

This notebook teaches students how to add input and output validation to agents so unsafe, off-topic, or low-quality content never reaches the user. It introduces two guardrail types (input and output), two implementation styles (rule-based and LLM-based), and the tripwire mechanism that halts execution on violation. Students also learn when to use parallel vs blocking execution modes to optimize latency or token cost.

## Concepts to explain on camera

- **Guardrail:**
  - **Term:** A validation layer that wraps an agent and intercepts either the user's input or the agent's output, checking it against rules before allowing it through.
  - **Analogy:** Like a bouncer at a nightclub door -- they check IDs on the way in (input guardrail) and might also check bags on the way out (output guardrail). If something fails the check, you don't get through.
  - **What students need to understand:**
    - Guardrails are separate from the agent's instructions -- they are enforced programmatically, not by asking the LLM to behave
    - Input guardrails validate what the user sends before (or while) the agent processes it
    - Output guardrails validate what the agent produces before the user sees it
    - You can stack multiple guardrails on a single agent

- **Tripwire:**
  - **Term:** A boolean flag (`tripwire_triggered=True`) inside a guardrail's return value that, when set, immediately halts execution and raises an exception -- either `InputGuardrailTripwireTriggered` or `OutputGuardrailTripwireTriggered`.
  - **Analogy:** Like a circuit breaker in your house -- the moment the current is too high, it trips and cuts all power. No partial delivery, no graceful degradation, just a hard stop.
  - **What students need to understand:**
    - Tripwires are all-or-nothing -- they raise an exception, not a warning
    - You must wrap `Runner.run()` in try/except to handle tripped guardrails gracefully
    - The exception type tells you whether it was the input or output guardrail that fired
    - Without the try/except, a tripped guardrail crashes your program

- **Parallel vs blocking execution mode:**
  - **Term:** By default, input guardrails run in parallel with the agent (both start at the same time). Setting `run_in_parallel=False` makes the guardrail run first -- if it trips, the agent never starts.
  - **Analogy:** Parallel mode is like a security scanner at an airport that scans your bag while you walk through -- faster, but you've already started walking. Blocking mode is like a locked door that only opens after the scanner clears you.
  - **What students need to understand:**
    - Parallel (default) gives better latency because both run at once, but the agent may process tokens before the guardrail result is known
    - Blocking (`run_in_parallel=False`) saves tokens because the agent never starts if the guardrail trips
    - Use blocking mode when the guardrail is cheap (rule-based) and the agent call is expensive
    - The parameter lives on `InputGuardrail(guardrail_function=..., run_in_parallel=False)`, not on the agent itself

- **`GuardrailFunctionOutput`:**
  - **Term:** The return type every guardrail function must produce. Contains `tripwire_triggered` (bool) and `output_info` (any extra data -- a string, a Pydantic model, whatever you want for debugging).
  - **Analogy:** Like a customs declaration form -- you fill in "pass" or "fail" and attach your reasoning.
  - **What students need to understand:**
    - Every guardrail function must return this exact type
    - `tripwire_triggered=True` means block; `False` means allow
    - `output_info` is for your own debugging -- it can be a string, a Pydantic model, or anything serializable
    - The decorator (`@input_guardrail` or `@output_guardrail`) handles wiring the return value into the SDK

## Key SDK pattern

The primary pattern is attaching guardrails to an agent via `input_guardrails` and `output_guardrails` lists. Here is the core input guardrail pattern from the notebook:

```python
@input_guardrail
async def topic_guardrail(
    ctx: RunContextWrapper,
    agent: Agent,
    input: str | list[TResponseInputItem]
) -> GuardrailFunctionOutput:
    """Block requests unrelated to cooking."""
    input_text = input if isinstance(input, str) else str(input)
    off_topic_terms = ["politics", "stocks", "weather", "sports", "news"]
    is_off_topic = any(term in input_text.lower() for term in off_topic_terms)
    return GuardrailFunctionOutput(
        tripwire_triggered=is_off_topic,
        output_info="Off-topic request detected" if is_off_topic else "OK"
    )

cooking_agent = Agent(
    name="CookingAssistant",
    instructions="You are a cooking assistant. Help with recipes, techniques, and ingredients.",
    model=MODEL,
    input_guardrails=[InputGuardrail(guardrail_function=topic_guardrail)]
)
```

Parameter breakdown:
- **`@input_guardrail` decorator** -- marks the function as a guardrail the SDK can wire up. There is also `@output_guardrail` for output validation.
- **`ctx: RunContextWrapper`** -- the shared context object; same one the agent and tools see. Not used in these demos but required in the signature.
- **`agent: Agent`** -- reference to the agent the guardrail is attached to. Also required in the signature.
- **`input: str | list[TResponseInputItem]`** -- the raw user input. Can be a plain string or the structured input item list from multi-turn conversations.
- **`GuardrailFunctionOutput`** -- the return type. `tripwire_triggered=True` blocks; `output_info` carries debug data.
- **`InputGuardrail(guardrail_function=...)`** -- the wrapper that goes in the agent's `input_guardrails` list. This is also where you set `run_in_parallel=False` for blocking mode.

## The demos and what each shows

**Part 2 -- Rule-Based Input Guardrail:** Creates `cooking_agent` with `topic_guardrail` that checks for off-topic keywords like "politics", "stocks", "sports". Tests it with "How do I make a good pasta sauce?" (passes) and "What are today's top news stories?" (blocked because "news" is in the off-topic list). Emphasize on camera: this guardrail costs zero LLM tokens -- it is pure Python. Also emphasize the try/except pattern around `Runner.run()` catching `InputGuardrailTripwireTriggered`.

**Part 3 -- LLM-Based Input Guardrail:** Creates `topic_checker_agent` with `output_type=TopicCheck` (a Pydantic model with `is_cooking_related: bool` and `reasoning: str`). The `llm_topic_guardrail` runs this checker agent inside the guardrail function via `Runner.run(topic_checker_agent, input, context=ctx.context)`. Tests with three inputs: "What herbs pair well with chicken?" (passes), "Can you write me a Python script?" (blocked), and "What wine goes with pasta?" (passes as food-adjacent). Emphasize on camera: the keyword guardrail from Part 2 would miss "Can you write me a Python script?" because none of the off-topic keywords appear in it -- this is why LLM-based guardrails exist.

**Part 4 -- Output Guardrail:** Creates `concise_agent` with `length_guardrail` that blocks responses over 30 words. Uses `LengthCheck` Pydantic model in `output_info` with `is_acceptable` and `word_count`. Tests with "What is 2 + 2?" (short answer, passes) and "List and explain five key differences between Python and JavaScript" (likely triggers the 30-word limit). Emphasize on camera: the output guardrail function signature takes `output: str` instead of `input`, and you catch `OutputGuardrailTripwireTriggered` instead of `InputGuardrailTripwireTriggered`.

**Part 5 -- Blocking Mode:** Creates `blocking_agent` reusing the same `topic_guardrail` from Part 2 but with `run_in_parallel=False`. Tests with "What's the latest news in sports?" -- the guardrail runs first, trips, and the agent never starts. Emphasize on camera: no tokens spent on the main agent at all. Compare this to the default parallel mode where both start simultaneously.

## Gotchas worth knowing before recording

- The output guardrail test for the long response ("List and explain five key differences between Python and JavaScript") may not always trigger -- LLM responses vary in length. The notebook includes a note about this. If it passes on camera, explain that the response happened to be under 30 words and that is fine -- it shows the guardrail is checking, not that it always blocks.
- The guardrail function signatures are strict: input guardrails take `(ctx, agent, input)`, output guardrails take `(ctx, agent, output)`. Mixing these up produces confusing errors. Call this out when switching from Part 2 to Part 4.
- `input` parameter in the guardrail is `str | list[TResponseInputItem]` -- the notebook handles this with `input if isinstance(input, str) else str(input)`. Mention this briefly so students know why the type check is there.
- The LLM guardrail in Part 3 calls `Runner.run()` inside the guardrail function -- that is a full agent run within the guardrail. Students may not expect a guardrail to itself invoke an agent. Make this clear: the guardrail agent (`topic_checker_agent`) is separate from the main agent (`smart_cooking_agent`).
- The `context=ctx.context` pass-through in `llm_topic_guardrail` is easy to miss. Mention it so students know how to forward context when running sub-agents inside guardrails.
- The blocking mode demo reuses `topic_guardrail` from Part 2 -- no new guardrail function is needed. Point this out so students see that the same guardrail can be used in parallel or blocking mode just by changing `run_in_parallel`.
- Output guardrails only run on the last agent in a run -- the troubleshooting section mentions this. Worth a quick callout if students will later use handoffs (M05A).

## How it connects to adjacent notebooks

**Before (M06C: Vector Memory with ChromaDB):** Students finished Module 6 on memory and state. No direct dependency, but you can note: "Now that our agents can remember things and use tools, we need to make sure they stay safe -- that is what Module 7 is about." This is the first notebook in the Safety, Control & Observability module, so frame it as a shift in focus from capabilities to constraints.

**After (M07B: Prompt Injection & Tool Safety):** Guardrails protect against accidental misuse, but M07B covers intentional attacks -- prompt injection via web pages, documents, and tool outputs. On camera, tee it up: "Guardrails handle the case where users go off-topic or responses are low quality. But what happens when an attacker deliberately tries to hijack your agent? That is what we cover next."
