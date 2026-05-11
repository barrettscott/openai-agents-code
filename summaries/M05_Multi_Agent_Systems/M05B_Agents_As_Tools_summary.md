

---

# M05B: Agents as Tools — Recording Prep

## What this notebook teaches

This notebook teaches how to use one agent as a callable tool inside another agent, so an orchestrator can call multiple specialists and combine their results into a single response — without ever giving up control of the conversation. This is the key alternative to handoffs: instead of transferring the user to a specialist, the orchestrator dispatches work to specialists behind the scenes and synthesizes the answers. It matters because real-world tasks (product reviews, balanced analyses, multi-perspective reports) often require input from several focused agents, and this pattern is the foundation for the multi-agent pipelines students will build in M05C–M05E.

## Concepts to explain on camera

- **Orchestrator (pattern)**
  - **Plain English:** A single "boss" agent that coordinates the work of other agents. It receives the user's request, decides which specialists to call, collects their outputs, and writes the final response.
  - **Analogy:** A news editor who assigns stories to reporters, reads all their drafts, then writes the final article that goes to print. The reporters never talk to the reader directly.
  - **What students need to understand:**
    - The orchestrator is the only agent the user ever "sees" — it owns the conversation start to finish
    - It decides the order in which to call specialists (sequentially in this notebook; parallel comes in M05C)
    - `result.last_agent.name` will always be the orchestrator's name, proving control never transferred
    - The orchestrator's instructions are what glue the specialists together — weak instructions mean it may skip a specialist

- **Agent as Tool**
  - **Plain English:** Taking a full agent — with its own instructions and personality — and exposing it to another agent as if it were a regular function tool. The orchestrator calls it, passes input, gets text back.
  - **Analogy:** Hiring a freelance consultant. You give them a brief, they do their work, hand you a report, and leave. You still write the final deliverable yourself.
  - **What students need to understand:**
    - The specialist agent runs a full `Runner.run()` behind the scenes, but the orchestrator just sees a string result
    - The specialist has no access to the orchestrator's conversation history — it only gets the input the orchestrator sends
    - This is different from handoffs where the specialist takes over and responds directly to the user
    - You can mix agents-as-tools with regular `@function_tool` tools in the same `tools=[]` list

- **Handoff vs Agent-as-Tool (distinction)**
  - **Plain English:** Two ways to involve a second agent. Handoff = "you take it from here." Agent-as-tool = "do this subtask and report back to me."
  - **Analogy:** Handoff is transferring a customer call to another department — you hang up. Agent-as-tool is putting the customer on hold, calling another department yourself, getting the answer, then coming back to the customer.
  - **What students need to understand:**
    - Handoffs: specialist responds to user, orchestrator is done, `result.last_agent.name` is the specialist
    - Agent-as-tool: orchestrator responds to user, specialist is invisible, `result.last_agent.name` is the orchestrator
    - Handoffs cannot call multiple specialists per turn; agent-as-tool can
    - Use handoffs for routing (M05A), use agents-as-tools for coordinating

- **`.as_tool()` method**
  - **Plain English:** A method on any `Agent` instance that wraps it so it can be placed in another agent's `tools=[]` list. You give it a name and description, just like you'd describe any function tool.
  - **Analogy:** Putting a name badge and job title on a specialist so the orchestrator knows who to call and when.
  - **What students need to understand:**
    - `tool_name` is what the orchestrator uses to call it — match this to what you reference in the orchestrator's instructions
    - `tool_description` tells the model *when and why* to call this tool — vague descriptions mean the model may not call it
    - It returns the specialist's `final_output` as a string to the orchestrator
    - You don't need `@function_tool` — `.as_tool()` handles the wrapping automatically

## Key SDK pattern

The primary pattern is wrapping an agent with `.as_tool()` and placing it in another agent's `tools=[]` list:

```python
pros_agent = Agent(
    name="ProsAnalyst",
    instructions="You identify and explain the advantages and benefits of any topic. Be specific and concise.",
    model=MODEL
)

cons_agent = Agent(
    name="ConsAnalyst",
    instructions="You identify and explain the disadvantages and risks of any topic. Be specific and concise.",
    model=MODEL
)

analysis_orchestrator = Agent(
    name="AnalysisOrchestrator",
    instructions=orchestrator_instructions,
    model=MODEL,
    tools=[
        pros_agent.as_tool(
            tool_name="pros_analyst",
            tool_description="Analyzes advantages and benefits of a topic."
        ),
        cons_agent.as_tool(
            tool_name="cons_analyst",
            tool_description="Analyzes disadvantages and risks of a topic."
        )
    ]
)

result = await Runner.run(analysis_orchestrator, input="Working from home vs working in an office")
```

Parameter-by-parameter:

- **`pros_agent` / `cons_agent`** — Regular `Agent` instances, each with focused `instructions`. These are the specialists. They have no tools of their own — they just generate text based on their expertise.
- **`.as_tool(tool_name="pros_analyst", tool_description="...")`** — Wraps the agent so the orchestrator can call it like a function. `tool_name` is the callable name the model sees; `tool_description` tells the model what the tool does and when to use it.
- **`tools=[...]`** on the orchestrator — The list of tools available to `analysis_orchestrator`. Here, both are agent-tools, but you could mix in `@function_tool` functions too.
- **`orchestrator_instructions`** — Critically, these tell the orchestrator to "always call both" tools and "synthesize their outputs." Without explicit instructions, the model might skip one.
- **`result.last_agent.name`** — Will be `"AnalysisOrchestrator"`, proving the orchestrator stayed in control. If this showed a specialist name, you'd know you accidentally used a handoff instead.

## The demos and what each shows

**Part 1 — Handoff vs Agent-as-Tool (markdown only).** This is a conceptual comparison with ASCII diagrams and a table. No code runs here. Walk through the two flow diagrams slowly: handoff is a one-way transfer (`Triage → Specialist takes over`), agent-as-tool is a round trip (`Orchestrator → Specialist → result back → Orchestrator responds`). Emphasize the table's four rows — who responds to the user, can you call multiple specialists, does the orchestrator stay in control, and when to use each. Say explicitly: "In M05A we built handoffs. The limitation was that once we handed off, we were done. This notebook solves that."

**Part 2 — The `.as_tool()` Method (pros/cons analysis).** This is the first runnable demo. Two specialist agents (`ProsAnalyst` and `ConsAnalyst`) are created, then wrapped with `.as_tool()` and given to `AnalysisOrchestrator`. The orchestrator is run with the input `"Working from home vs working in an office"`. The key thing to show on camera: `result.last_agent.name` prints `"AnalysisOrchestrator"` — the orchestrator responded, not either specialist. The output contains both pros and cons synthesized together. Emphasize that the orchestrator's instructions explicitly say "Always call both the pros_analyst and cons_analyst tools" — this is what makes the model reliably call both. Point out that `tool_name="pros_analyst"` matches the name used in the instructions string. Call back to M02C: "Remember when I said instructions become critical in multi-agent systems? This is why."

**Part 3 — Multi-Specialist Orchestration (product review pipeline).** This scales the pattern to three specialists: `FeaturesAnalyst`, `PricingAnalyst`, and `VerdictWriter`. The `ReviewOrchestrator` calls all three in sequence — features first, then pricing, then verdict (which needs the outputs of the first two). The input is `"GitHub Copilot — AI coding assistant, $10/month for individuals"`. This demo proves that agents-as-tools can be chained: the orchestrator feeds earlier results into later tool calls. Emphasize that the `review_instructions` are numbered 1–4, giving the model a clear execution order. Show that `result.last_agent.name` is still `"ReviewOrchestrator"`. Point out this is a sequential pipeline — "In the next notebook, M05C, we'll learn how to run some of these in parallel."

## Gotchas worth knowing before recording

- **The notebook uses `await Runner.run(...)` directly in cells** — this works in Jupyter because Jupyter has a running event loop. If a student tries this in a `.py` script, they'll get a `RuntimeError`. You don't need to dwell on this, but if you get the question: "wrap it in `asyncio.run()`" — or point them to M05C/M09A.
- **The orchestrator may not call all specialists every time.** LLMs are non-deterministic. If the orchestrator skips a tool on camera, re-run the cell. The troubleshooting section addresses this — stronger instructions help. Be ready to say: "If this happens, check your instructions — the more explicit you are, the more reliable it is."
- **`tool_name` and the name used in instructions must align.** In the notebook, `tool_name="pros_analyst"` matches the instruction text `"call both the pros_analyst and cons_analyst tools"`. If students rename one but not the other, the model may not call the tool. Call this out explicitly.
- **`truncate_response()` is a helper, not SDK.** It's defined in Setup and used to keep output readable. Mention it briefly ("This just trims long outputs so we can see the structure") so students don't think it's an SDK feature.
- **Specialist agents have no tools of their own** — they rely purely on model knowledge. This is intentional for this notebook. Students may ask "can a specialist also have tools?" — yes, and they'll see that in the capstone (M05E). Don't get sidetracked, just say "absolutely, and we'll do that soon."
- **The `result.last_agent.name` check is the proof point.** If it ever shows a specialist name instead of the orchestrator name, something is wrong — likely a handoff was used instead of `.as_tool()`. The troubleshooting section covers this. Emphasize it on camera: "This one line tells you whether you built a handoff or an agent-as-tool system."
- **The variable `orchestrator_instructions` is defined in a separate cell from the orchestrator agent in Part 2.** Make sure to run cells in order. If you jump ahead, `orchestrator_instructions` won't exist and you'll get a `NameError`.
- **Part 3 reuses the same pattern but with `review_instructions` as the variable name.** Don't accidentally reference `orchestrator_instructions` from Part 2 — they're different variables for different orchestrators.

## How it connects to adjacent notebooks

**Builds on M05A: Handoffs.** Students already know how to route a user from a triage agent to a specialist with `handoffs=[]`. Call back to that: "In M05A, we built a customer support system where the triage agent handed off to billing or tech support. The problem was, once we handed off, the triage agent was done. It couldn't call billing AND tech support and combine the answers. That's what this notebook solves." Also call back to M02C (Writing Effective Agent Instructions): the orchestrator's instructions are what make this pattern reliable — "this is the payoff from the instruction-writing patterns we practiced."

**Leads into M05C: Parallel Execution.** In this notebook, the orchestrator calls specialists sequentially — one after another. In M05C, students learn `asyncio.gather` to run specialists concurrently. Tee it up: "Right now, our orchestrator is calling each specialist one at a time and waiting. In the next notebook, we'll run them in parallel — so two or three specialists work at the same time, and we get results faster." Also note that M05D (Debate & Critique) and M05E (Capstone #2) will combine both handoffs and agents-as-tools with parallel execution, so this pattern is foundational for the rest of Module 5.

---