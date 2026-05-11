# M07E: Capstone #3 -- Production Customer Service Agent -- Recording Prep

## What this notebook teaches

This capstone combines every safety and control feature from Module 7 -- guardrails (M07A), prompt injection awareness (M07B), human-in-the-loop approval (M07C), and tracing (M07D) -- plus sessions from Module 6, into a single production customer service agent. It teaches students how these independent features compose into a layered architecture where the orchestration function (`handle_customer_message`) is the integration point. The core lesson is that production agents need multiple safety layers working together, not just one.

## Concepts to explain on camera

- **Threshold-based auto-approval**
  - **Term:** A pattern where low-risk actions execute automatically and high-risk actions pause for human review, based on a numeric threshold.
  - **Analogy:** Like a corporate expense policy -- employees can approve purchases under $50 themselves, but anything above needs a manager's signature.
  - **What students need to understand:**
    - `AUTO_APPROVE_THRESHOLD = 50.0` is the cutoff in this notebook
    - ORD-002 (Phone Case, $19.99) would auto-approve; ORD-001 (Wireless Headphones, $149.99) triggers manual review
    - The threshold lives in your orchestration code, not in the agent or the SDK
    - In production you would tune this number based on business risk tolerance

- **Interruptions and state (approval flow mechanics)**
  - **Term:** When a tool marked with `needs_approval=True` is called, the SDK pauses execution and returns an `interruptions` list. You inspect each interruption, approve or reject it, then resume the run.
  - **Analogy:** Like a document workflow where certain pages need a signature before the process continues -- the system stops, waits for your stamp, then picks up where it left off.
  - **What students need to understand:**
    - `result.interruptions` is truthy when the agent tried to call a tool that needs approval
    - `result.to_state()` captures the frozen run so you can modify it
    - `state.approve(interruption)` or `state.reject(interruption, message=...)` records the decision
    - After approving/rejecting, you pass `state` back into `Runner.run()` to resume

- **Read vs write tool safety principle**
  - **Term:** Read-only tools (like `lookup_order`, `search_faq`) can auto-execute because they have no side effects. Write tools (like `process_refund`) are irreversible and need stronger controls.
  - **Analogy:** Looking at your bank balance is harmless and can happen automatically; transferring money is irreversible and should require confirmation.
  - **What students need to understand:**
    - This is the M07B principle applied concretely -- `lookup_order` and `search_faq` run freely, `process_refund` pauses
    - The SDK enforces this via `@function_tool(needs_approval=True)`
    - The orchestration layer decides how to handle the pause (auto-approve vs manual)
    - Point this out explicitly on camera -- it ties M07B theory to M07C practice

## Key SDK pattern

The primary pattern is the approval-loop orchestration inside `handle_customer_message`. Here is the core structure:

```python
result = await Runner.run(support_agent, input=message, session=session)

while result.interruptions:
    state = result.to_state()

    for interruption in state.get_interruptions():
        tool_name = interruption.raw_item.name
        args = json.loads(interruption.raw_item.arguments)

        if tool_name == "process_refund":
            amount = args.get("amount", 0)
            if auto_approve and amount <= AUTO_APPROVE_THRESHOLD:
                state.approve(interruption)
            else:
                decision = input("      Approve? (yes/no): ").strip().lower()
                if decision in ["yes", "y"]:
                    state.approve(interruption)
                else:
                    state.reject(interruption, message="Refund requires additional verification.")
        else:
            state.approve(interruption)

    result = await Runner.run(support_agent, state)
```

Parameter-by-parameter:

- `session=session` -- a `SQLiteSession("customer_demo_001")` instance that gives the agent memory across turns; same object passed every call
- `result.interruptions` -- list of tool calls that need approval; the `while` loop keeps running until none remain
- `result.to_state()` / `state.get_interruptions()` -- freezes the run and exposes each pending tool call for inspection
- `interruption.raw_item.name` and `interruption.raw_item.arguments` -- the tool name and JSON args the model chose, so you can route approval logic per tool
- `state.approve(interruption)` / `state.reject(interruption, message=...)` -- records the human decision
- `Runner.run(support_agent, state)` -- resumes the run from where it paused, with the approval decisions applied

## The demos and what each shows

**Phase 5 -- Multi-turn conversation demo.** Three messages are sent through `handle_customer_message` using the same `SQLiteSession("customer_demo_001")`: (1) "Hi, my order is ORD-001. I need help with my headphones." triggers `lookup_order`, (2) "They stopped working after 3 days. Can I get a refund?" triggers `process_refund` with $149.99 -- above the $50 threshold, so it pauses for manual approval via `input()`, and (3) "What's your return policy?" triggers `search_faq`. This proves sessions carry context (the agent remembers ORD-001 from Turn 1 when processing the refund in Turn 2), the guardrail lets legitimate support questions through, and the approval flow activates only for the write tool. Emphasize on camera that Turn 2 will pause and wait for keyboard input -- warn students to click the cell output area in JupyterLab.

**Guardrail test -- off-topic request.** Sends "Can you help me write a Python script?" through the same pipeline. The `support_topic_guardrail` runs the `TopicChecker` agent (which uses `SupportTopicCheck` with `is_support_related: bool`), it returns `False`, the guardrail sets `tripwire_triggered=True`, and `InputGuardrailTripwireTriggered` is caught, returning the hardcoded refusal message. This proves the guardrail blocks non-support requests before the main agent ever sees them. Emphasize that this is the exact pattern from M07A, now wired into a full pipeline.

## Gotchas worth knowing before recording

- **Turn 2 will pause for `input()`.** The $149.99 refund exceeds `AUTO_APPROVE_THRESHOLD = 50.0`, so JupyterLab will show an input prompt. If you do not click the cell output area, the prompt may not be visible. Mention this before running the cell.
- **The model supplies `amount` and `reason` to `process_refund`.** The notebook has a comment noting that "amount and eligibility are supplied by the model based on the order lookup. Production code should validate both against the order record before approving." Worth calling out on camera -- the model could hallucinate a different amount.
- **Session cleanup at the end.** `await customer_session.clear_session()` wipes the SQLite session. If you re-run the demo without clearing, old conversation history may affect results. If something looks odd, clear the session first.
- **The guardrail test reuses `customer_session`.** This means the off-topic request still has the previous conversation in session memory. The guardrail blocks it anyway because it evaluates the current message, not the session history.
- **`ORDERS` and `FAQS` are hardcoded dicts/lists.** If students ask about "real" databases, acknowledge this is simulated -- the pattern is the same regardless of backend.
- **The `while result.interruptions` loop handles multiple interruptions.** In this demo only one tool needs approval, but the loop structure handles cases where the agent calls multiple approval-required tools in a single turn.
- **Exercise 1 is substantial.** It asks students to add `check_shipping_status`, create a stricter guardrail that rejects competitor comparisons, and build a golden test set with a judge agent (callback to M03C). Set expectations that this is a project-level exercise, not a quick fill-in.

## How it connects to adjacent notebooks

**Builds on:** This capstone directly integrates M07A (guardrails -- the `InputGuardrail` and `@input_guardrail` pattern), M07B (prompt injection awareness -- the security note about untrusted customer messages and the read-vs-write tool principle), M07C (human-in-the-loop -- `needs_approval=True`, interruptions, `state.approve/reject`), M07D (tracing -- every run is automatically traced in the OpenAI dashboard), and M06A (sessions -- `SQLiteSession` for multi-turn memory). On camera, reference each of these explicitly: "Remember in M07A we built guardrails? Here's the same pattern wired into a real agent." "In M07C we learned about approval flows -- now we're combining that with a threshold to auto-approve small refunds."

**Leads to:** M08A: MCP Fundamentals. The notebook explicitly states this as the next step. The bridge is that this capstone showed how to build tools with `@function_tool` -- MCP gives agents access to external tool servers without writing custom integration code. On camera: "We built three custom tools here. In Module 8, you'll connect to MCP servers that provide tools automatically."
