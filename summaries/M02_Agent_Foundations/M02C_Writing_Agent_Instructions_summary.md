

---

# M02C: Writing Effective Agent Instructions — Recording Prep

## What this notebook teaches

This notebook teaches students how to write agent instructions that produce consistent, predictable behavior across a wide range of inputs. It covers five specific instruction patterns — durable formatting, clarification triggers, tool-use constraints, completion criteria, and refusal/escalation — and shows that the `instructions` parameter on `Agent()` is the primary lever for controlling what an agent does and doesn't do. This matters because vague instructions produce variable outputs that are nearly impossible to debug, and these patterns become load-bearing once students build tool-heavy and multi-agent systems in M04–M05.

## Concepts to explain on camera

- **System instructions (instructions parameter)**
  - **Term:** The text you pass to the `instructions=` parameter on an `Agent()`. It runs on every turn of conversation and tells the model its role, constraints, and behavioral rules — like a standing set of orders.
  - **Analogy:** Think of it as a job description pinned to the wall of an employee's cubicle. They read it every time a new request comes in before deciding what to do.
  - **What students need to understand:**
    - Instructions are not one-time prompts — they execute on every single user message
    - Vague instructions ("be helpful") give the model room to improvise, which means inconsistent results
    - Specific instructions ("produce exactly one sentence, no jargon") constrain the model's choices and make behavior repeatable
    - Instruction order/position can matter — if a constraint is being ignored, moving it later in the string sometimes helps

- **Durable instructions**
  - **Term:** Instructions that hold up across a wide range of inputs, not just the examples you tested during development. They specify format, length, tone, and stop conditions explicitly.
  - **Analogy:** A recipe that says "season to taste" gives different results every time. A recipe that says "add exactly 1 tsp salt" is durable — it works the same no matter who's cooking.
  - **What students need to understand:**
    - "Be concise" is not durable — "respond in one sentence" is
    - You need to specify what the agent should NOT include (e.g., "do not add commentary or follow-up questions")
    - Completion criteria ("stop after the third bullet") prevent the agent from rambling
    - If behavior varies across runs on the same input, the instruction is underspecified

- **Escalation pattern**
  - **Term:** An instruction that tells the agent to redirect the user to a specific resource (email, department, person) instead of attempting to handle something itself. In this notebook, escalation is purely instructional — the agent tells the user where to go. Real routing/handoff mechanics come in M05A.
  - **Analogy:** A front desk receptionist who says "I can't process refunds, but here's the billing department's direct number" rather than guessing how to fix it themselves.
  - **What students need to understand:**
    - A refusal without an escalation path leaves the user stuck
    - An escalation without a refusal can let the agent attempt something it shouldn't
    - The escalation trigger should be concrete ("billing dispute or account access issue"), not vague ("sensitive topics")
    - This is instruction-level only — actual handoff to another agent is M05A

- **Tool-use constraint**
  - **Term:** An explicit instruction telling the agent when it should and should not call a tool. Without this, the agent decides on its own, and may call tools unnecessarily — wasting tokens and adding latency.
  - **Analogy:** Giving someone a calculator and saying "use this only for math problems" vs. just handing them a calculator with no guidance — they might try to use it for everything.
  - **What students need to understand:**
    - Having a tool available doesn't mean the agent should always use it
    - The constraint goes in the instructions string: "Use the lookup_product_price tool only when the user asks about price"
    - An overly broad tool docstring can also trigger unintended calls — the docstring and instruction work together
    - This pattern becomes critical with real tools (web search, file search, code interpreter) in M04

## Key SDK pattern

This notebook doesn't introduce a new SDK class — it deepens the `Agent()` + `Runner.run()` pattern from M02A by showing how the `instructions` parameter drives all behavioral control. The core pattern repeated throughout:

```python
agent_durable = Agent(
    name="DurableAgent",
    instructions=durable_instructions,
    model=MODEL
)

durable_result = await Runner.run(agent_durable, input=message)

print(durable_result.final_output)
```

**Parameters in plain English:**

- `name=` — A label for the agent. Used in tracing/dashboard. Students saw this in M02A.
- `instructions=` — **This is the star of this notebook.** A string (often multi-line) that defines role, constraints, format rules, tool-use conditions, refusal behavior, and escalation paths. Runs on every turn.
- `model=MODEL` — Uses `gpt-5-mini` set in the setup cell. Same constant from M02A.
- `tools=` — Appears in Part 3 only, as `tools=[lookup_product_price]`. Attaches the function tool from M02B so the agent can call it — but the instructions control *when* it gets called.
- `Runner.run(agent, input=)` — Sends the user message, agent processes it using its instructions, returns a result.
- `result.final_output` — The agent's text response. Same as M02A.

The key teaching point: the SDK structure is identical to M02A. The only thing that changes is what you write in `instructions`, and that single parameter is what makes agents reliable or unreliable.

## The demos and what each shows

**Part 1 — Durable System Instructions (Vague vs Durable):** Two agents summarize the same message: `"Summarize this for me: AI agents use tools to take actions in the world."` The vague agent (`"Be helpful and summarize things."`) produces an unpredictable response. The durable agent uses a four-line instruction set specifying role, format (one sentence), tone (plain language), and stop condition (no commentary). Emphasize on camera: run both, compare the outputs side by side, and point out that the vague one will likely give you different results if you run it again — that's the whole problem. The durable one constrains the model enough that it converges on similar output across runs.

**Part 2 — When to Ask Clarifying Questions:** A scheduling agent (`agent_clarify`) is given an incomplete request (`"Book a meeting with the team."`) and a complete one (`"Book a meeting with the team on Friday at 2pm. Attendees: Alice, Bob, Carol."`). The instructions name three specific required fields: date, time, and attendees. On the incomplete request, the agent asks for what's missing. On the complete request, it proceeds. Emphasize on camera: same agent, same instructions, two different behaviors — and both are correct. The key line is `"If any of these are missing, ask for them — do not assume."` Also emphasize the flip side: `"If all required details are present, proceed without asking a follow-up question"` prevents the agent from being unnecessarily cautious.

**Part 3 — When NOT to Use a Tool:** Uses `lookup_product_price` (a simple dictionary lookup tool from `@function_tool`). The agent (`agent_constrained`) receives two inputs: `"What is a widget?"` (general question — tool should NOT fire) and `"How much does a widget cost?"` (price question — tool SHOULD fire). The key instruction line is `"Use the lookup_product_price tool only when the user asks about price."` Emphasize on camera: without that line, the agent might call the tool for the general question too, wasting a tool call. Also note that the tool's docstring (`"Look up the current price of a product by name."`) works with the instruction — if the docstring were broader, it could override the constraint.

**Part 4 — Response Style & Completion Criteria:** A code review agent reviews a deliberately imperfect Python function (`get_user` that reads a file without closing it, uses bare `json.loads`, accesses dict by key without error handling). The instructions demand exactly three bullet points, no summary/intro/closing, and "stop after the third bullet." Emphasize on camera: the completion criterion is what makes this work — without "stop after the third bullet," the model often adds a concluding sentence or offers to elaborate. The code sample is intentionally flawed to give the agent obvious things to find.

**Part 5 — Refusal and Escalation Patterns:** A customer support agent (`agent_refusal`) is tested with three inputs: an in-scope question (`"Does your product support two-factor authentication?"`), an out-of-scope question (`"Can you help me write a cover letter?"`), and an escalation trigger (`"I was charged twice last month."`). The instructions define scope (features, pricing, troubleshooting), a refusal behavior (politely decline + suggest appropriate resource), and a specific escalation path (`billing@example.com`). Emphasize on camera: three different behaviors from one instruction set. Call out explicitly that this is instruction-level escalation only — the agent *tells* the user to email billing. In M05A, students will learn how to actually hand off to another agent.

## Gotchas worth knowing before recording

- **LLM non-determinism on vague instructions:** The vague agent in Part 1 may give a perfectly fine response on your recording run. That's actually the point — it *might* be fine, but it won't be *consistently* fine. If it happens to look great, say something like: "This looks fine right now, but run it five times and you'll get five different formats. That's the problem durable instructions solve."

- **The scheduling agent might still ask a question on the complete input.** If the model decides "Friday" is ambiguous (which Friday?), it may ask a clarification question even with all three fields present. This is actually a reasonable thing for the model to do — acknowledge it on camera and note that you could add "Accept relative dates like 'Friday' without asking for a specific date" to the instructions to close that gap.

- **`lookup_product_price` uses `.lower()` internally** — so "Widget" and "widget" both work. But the tool returns `"Product not found."` for anything not in the dictionary. If you ad-lib a test with a product not in the dict, you'll get that fallback string. Not a bug, just know it's there.

- **The tool constraint might not always hold.** LLMs follow instructions probabilistically, not deterministically. The agent *might* call `lookup_product_price` on "What is a widget?" even with the constraint instruction. If this happens on camera, it's a great teaching moment: "This is why we test — instructions reduce unwanted behavior but don't guarantee elimination. In M03C we'll build systematic evaluation to catch this."

- **Part 5's in-scope question is about 2FA for a fictional product.** The agent has no actual product knowledge, so it will generate a plausible but made-up answer. This is fine for the demo — the point is that it *answers* rather than refusing. But don't let students think the answer is grounded in real data.

- **All agents in this notebook are stateless (no sessions).** Each `Runner.run()` call is independent. The scheduling agent won't remember the first call when you run the second. This is by design — sessions come in M06A — but students might wonder why you don't continue the conversation with the scheduling agent after it asks clarifying questions.

- **The `await` keyword in every `Runner.run()` call** — students saw this in M02A but may still wonder about it. You don't need to deep-dive async here (that's M05C), just remind them "this is how we call the runner in a Jupyter notebook — we'll explain the async part fully later."

- **The multi-line string pattern for instructions** uses parenthesized string concatenation with `\n` newlines (e.g., `"line one.\n" "line two.\n"`). Students who haven't seen implicit string concatenation in Python might be confused. Worth a quick "Python lets you put strings next to each other in parentheses and they get joined automatically."

- **The code sample in Part 4 (`get_user` function)** doesn't import `json` — that's intentional. The agent should catch that as one of the three issues. If it doesn't, that's model variability, not a notebook bug.

## How it connects to adjacent notebooks

**Builds on M02A (How Agents Work):** Students already know `Agent()`, `Runner.run()`, and `result.final_output`. Callback suggestion: "Remember in M02A we built our first agent with a simple instruction string? Now we're learning how to write instructions that actually hold up in production." Also builds on M02B (Building Tools for Agents) — the `@function_tool` decorator and `tools=` parameter appear in Part 3. Callback: "We built the `@function_tool` pattern in M02B — now we're learning how to tell the agent *when* to use a tool and when not to."

**Sets up M03A-1 (Pydantic Basics) and M03A-2 (Structured Outputs):** This notebook controls output format through instructions alone (e.g., "exactly three bullet points"). The next module introduces Pydantic and `output_type=` to enforce format programmatically. Transition suggestion: "We've been controlling output format with instructions — and it works, but it's not guaranteed. In Module 3, we'll use structured outputs to enforce format at the code level so you get a validated Python object back, not just text you hope is formatted correctly."

**Critical foundation for M04–M05:** The notebook explicitly states that instructions become "load-bearing" in tool-heavy and multi-agent systems. In M04, agents with web search, file search, and code interpreter need precise tool-use constraints. In M05, handoff instructions determine whether a triage agent routes correctly. Mention this forward reference on camera: "Everything we're learning here about scoping, refusal, and tool constraints — this is what makes multi-agent pipelines in Module 5 actually work. Without clear instructions, agents silently misbehave, and that's the hardest kind of bug to find."

---