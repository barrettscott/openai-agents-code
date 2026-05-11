

---

# M05A: Handoffs — Recording Prep

## What this notebook teaches

This notebook introduces the **handoff pattern** — how to split work across multiple specialized agents instead of cramming everything into one. A triage agent receives the user's message and transfers responsibility to the right specialist (billing, tech support, refunds), and that specialist handles the rest. This matters because focused agents with narrow instructions consistently outperform a single generalist, and the SDK makes the routing almost free to implement.

## Concepts to explain on camera

- **Handoff**
  - **Term:** A transfer of responsibility from one agent to another. When the triage agent hands off, the specialist takes over completely — the triage agent does not get control back.
  - **Analogy:** Like calling a company's main line and the receptionist transferring you to the billing department. Once you're transferred, the receptionist hangs up — you're talking to billing now.
  - **What students need to understand:**
    - The handoff is one-way — the specialist finishes the conversation
    - The specialist receives the full conversation history, so no context is lost
    - The SDK implements handoffs as tool calls under the hood (e.g., `transfer_to_billing_agent`)
    - You don't write any routing logic yourself — the triage agent's LLM decides which tool to call based on the message and the tool descriptions

- **Triage agent**
  - **Term:** An agent whose only job is to classify the incoming request and route it to the correct specialist. It should not answer questions itself.
  - **Analogy:** A hospital triage nurse — they assess what's wrong and send you to the right department, but they don't perform surgery.
  - **What students need to understand:**
    - The triage agent needs clear instructions that say "do not answer questions yourself"
    - The `handoffs=[]` parameter on the triage agent is what creates the routing options
    - If the triage agent's instructions are vague, it may answer instead of handing off
    - Each agent in the handoffs list becomes a callable tool the triage agent can invoke

- **`tool_description_override`**
  - **Term:** A custom description you attach to a handoff that tells the triage agent exactly when to use that specialist. It replaces the auto-generated description the SDK would create from the agent's name alone.
  - **Analogy:** Like labeling doors in an office — instead of just "Room 3," you write "Room 3: Billing, Invoices, Payments, and Subscriptions." The more specific the label, the fewer people open the wrong door.
  - **What students need to understand:**
    - Without it, the SDK auto-generates a description from the agent name (e.g., "BillingAgent" becomes something like `transfer_to_billing_agent`)
    - With it, you control the exact text the triage agent sees when deciding which tool to call
    - This is especially important for ambiguous requests where the right specialist isn't obvious
    - It's set via the `handoff()` function, not directly on the Agent constructor

- **`result.last_agent`**
  - **Term:** A property on the Runner result that tells you which agent produced the final response — not which agent started the conversation.
  - **Analogy:** Like checking the signature at the bottom of a letter to see who actually wrote it, not who forwarded it to you.
  - **What students need to understand:**
    - If the handoff worked, `result.last_agent.name` will be the specialist, not the triage agent
    - If `result.last_agent.name` is still the triage agent, the handoff didn't fire — the triage agent answered the question itself
    - This is your primary debugging signal for whether routing worked correctly
    - The tracing dashboard (M07D) gives the full handoff chain if you need more detail

## Key SDK pattern

**Simple handoffs (Part 2):**
```python
triage_agent = Agent(
    name="TriageAgent",
    instructions=triage_instructions,
    model=MODEL,
    handoffs=[billing_agent, tech_agent, refunds_agent]
)
```

- `name="TriageAgent"` — the name of this agent; also used in `result.last_agent.name` to identify who responded
- `instructions=triage_instructions` — the system instructions telling the triage agent to route and never answer questions itself; these are the routing rules
- `model=MODEL` — `gpt-5-mini`; the triage agent doesn't need a reasoning model since it's just classifying a request
- `handoffs=[billing_agent, tech_agent, refunds_agent]` — a list of Agent objects; the SDK automatically generates a transfer tool for each one (e.g., `transfer_to_billing_agent`). The triage agent's LLM decides which to call.

**Custom handoffs with `handoff()` (Part 4):**
```python
triage_agent_v2 = Agent(
    name="TriageAgentV2",
    instructions=triage_v2_instructions,
    model=MODEL,
    handoffs=[
        handoff(
            billing_agent,
            tool_description_override="Transfer to billing for invoice, payment, refund, or subscription questions."
        ),
        handoff(
            tech_agent,
            tool_description_override="Transfer to tech support for bugs, errors, installation, or feature questions."
        )
    ]
)
```

- `handoff(billing_agent, tool_description_override="...")` — wraps the agent in a handoff object with an explicit description. The `tool_description_override` string is what the triage agent's LLM reads when deciding which transfer tool to call. More specific descriptions → better routing.
- Notice v2 only has two handoffs (billing and tech), while v1 had three (billing, tech, refunds). This is intentional for the comparison demo — it's not a mistake.

**Running the handoff:**
```python
result = await Runner.run(triage_agent, input=message)
print(result.last_agent.name)   # Which specialist handled it
print(result.final_output)       # The specialist's response
```

## The demos and what each shows

**Part 2 (A Simple Triage System):** This part defines three specialist agents — `billing_agent`, `tech_agent`, and `refunds_agent` — each with focused instructions and distinct personalities (billing is "empathetic and solution-focused," tech is "precise and step-by-step," refunds is "clear about timelines"). Then the `triage_agent` is defined with `handoffs=[billing_agent, tech_agent, refunds_agent]`. No routing code is written — the SDK creates the transfer tools automatically. Emphasize on camera: "We didn't write a single if-statement. The triage agent figures out where to route based on the instructions and the handoff tool names."

**Part 3 (See the Handoffs in Action):** Three test messages run through the triage agent: `"I was charged twice for my subscription this month."` (should go to BillingAgent), `"My app crashes when I try to export a PDF."` (should go to TechSupportAgent), and `"How do I upgrade to the annual plan?"` (should go to BillingAgent). For each, the demo prints which agent handled it via `result.last_agent.name` and the specialist's response. Emphasize: every message goes to `triage_agent`, but three different specialists respond. This is the "aha" moment — point out `result.last_agent.name` showing the specialist, not the triage agent.

**Part 4 (Customizing Handoffs):** This creates `triage_agent_v2` using the `handoff()` function with `tool_description_override` strings. The comparison demo sends the same ambiguous message — `"I need help with my account settings."` — through both v1 and v2 and prints which agent each version routes to. The point: ambiguous requests are where custom descriptions earn their keep. On camera, say something like: "The simple version has to guess from just the agent name. The custom version has explicit criteria to work with." Note that v2 only has billing and tech handoffs (no refunds), so this is specifically about routing quality for the agents it does have, not about coverage.

## Gotchas worth knowing before recording

- **All cells use `await Runner.run()`** — this is async code running in Jupyter, which handles the event loop automatically. Don't worry about `asyncio.run()` here; Jupyter's built-in event loop takes care of it. If a student asks, that's covered in M05C.

- **The triage agent may answer instead of handing off.** If the instructions aren't firm enough ("Do not answer questions yourself. Always hand off."), the LLM might just answer the billing question directly. If this happens on camera, it's actually a great teaching moment — show how strengthening the instructions or adding `tool_description_override` fixes it.

- **The ambiguous message comparison (Part 4) may route the same way for v1 and v2.** `"I need help with my account settings."` is genuinely ambiguous, and both versions might pick the same specialist. If they do, don't panic — say "Even when they agree, the custom description gives us confidence the decision was deliberate, not a coin flip. The real benefit shows up at scale with harder edge cases." If they differ, that's the ideal outcome to highlight.

- **`triage_agent_v2` only has two handoffs (billing and tech), not three.** This is easy to miss on camera. If you send a refund-related message to v2, it has nowhere to route it and may try to answer directly or force it into billing. Don't test v2 with a refunds message unless you want to show that behavior intentionally.

- **`truncate_response()` is defined in setup but only used in the Part 3 loop.** It's a readability helper that cuts long responses at 1200 characters. If students ask, it has nothing to do with the SDK — it's just for clean notebook output.

- **Variable naming: `triage_agent` vs `triage_agent_v2`.** Part 3 uses `triage_agent` (v1). Part 4 defines `triage_agent_v2` for the custom handoffs and then runs both side by side. Make sure you're clear about which version you're running when.

- **`result.last_agent` being the triage agent means the handoff didn't fire.** If you see `TriageAgent` as the handler, the triage agent answered instead of routing. The troubleshooting section covers this, but be ready to explain it live if it happens.

- **The `handoff` function must be imported.** The setup cell imports it: `from agents import Agent, Runner, handoff`. If you restart the kernel and skip the setup cell, the Part 4 code will fail with an import error.

## How it connects to adjacent notebooks

**What came before:** This is the first notebook in Module 5 (Multi-Agent Systems). Students have built single agents with tools (M02B), written effective instructions (M02C), added structured outputs (M03A-2), and used built-in tools (Module 4). On camera: "Up until now, every task was handled by a single agent. That works until you need different expertise, different tone, or different tools for different kinds of requests. That's what this module is about." The customer support domain also sets up nicely for M07E (Capstone #3), which builds a full production customer service agent — you can mention "we'll come back to customer support with guardrails and human-in-the-loop later in the course."

**What comes next:** M05B (Agents as Tools) teaches the alternative to handoffs — calling an agent like a function where the orchestrator keeps control and gets the result back. On camera at the end: "Now you know how to transfer control to a specialist permanently. But what if you want to call a specialist, get the answer back, and keep coordinating? That's agents-as-tools — next notebook." This distinction (handoff = transfer control vs. agent-as-tool = delegate and keep control) is the core decision framework for M05B, so plant the seed here.

---