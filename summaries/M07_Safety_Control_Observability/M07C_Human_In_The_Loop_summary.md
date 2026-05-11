# M07C: Human-in-the-Loop -- Recording Prep

## What this notebook teaches

This notebook teaches students how to pause agent execution at critical decision points and require human approval before a tool actually runs. It solves the problem of irreversible or high-stakes actions (sending emails, processing refunds) where you never want the agent to act autonomously. Students learn the full pause-inspect-approve/reject-resume flow, plus a threshold-based escalation pattern that auto-approves low-risk actions and escalates high-risk ones to a human.

## Concepts to explain on camera

- **Human-in-the-Loop (HITL):**
  - **Term:** A design pattern where the system pauses before executing a high-stakes action and waits for a human to approve or reject it. The tool never fires until the human says go.
  - **Analogy:** Like a bank wire transfer that sits in "pending" until a manager clicks Approve -- the money does not move until the human acts.
  - **What students need to understand:**
    - The agent decides it wants to call a tool, but the SDK intercepts and pauses before execution
    - The pending call is surfaced in `result.interruptions` for inspection
    - Approval means the tool runs; rejection means the agent gets feedback and adapts
    - This is your last line of defense after guardrails and prompt injection protections

- **Interruption:**
  - **Term:** The SDK's representation of a paused tool call that is waiting for human approval. Each interruption object contains the tool name and its arguments so you can inspect exactly what the agent wants to do.
  - **Analogy:** Like a sticky note on your desk saying "Agent wants to send this email -- sign off or toss it."
  - **What students need to understand:**
    - `result.interruptions` is a list -- there can be multiple pending approvals in one run
    - Each interruption has `interruption.raw_item.name` (the tool name) and `interruption.raw_item.arguments` (the JSON arguments)
    - You convert the result to resumable state with `result.to_state()` before approving or rejecting
    - After approving/rejecting, you resume with `Runner.run(agent, state)`

- **Threshold-based escalation:**
  - **Term:** A pattern where your code inspects the tool's arguments and auto-approves low-risk calls programmatically while routing high-risk calls to a human reviewer.
  - **Analogy:** Like an expense policy where anything under $50 is auto-approved but anything above needs your manager's signature.
  - **What students need to understand:**
    - The SDK always pauses for `needs_approval=True` tools -- it does not distinguish auto-approve from human-approve
    - Your code parses `interruption.raw_item.arguments` to read the parameters (e.g., the refund amount)
    - You decide in code whether to call `state.approve()` or escalate
    - This keeps humans in the loop only when it matters, avoiding approval fatigue

## Key SDK pattern

The primary pattern is marking a tool with `needs_approval=True` and then handling the pause-approve-resume cycle:

```python
@function_tool(needs_approval=True)
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email to the recipient. Requires human approval before sending."""
    return f"Email sent to {to}: {subject}"
```

Then the approval flow:

```python
# Step 1: Run -- SDK pauses when it hits a needs_approval tool
result = await Runner.run(email_agent, input="Send an email to alex@example.com...")

# Step 2: Inspect the pending approvals
if result.interruptions:
    state = result.to_state()
    for interruption in state.get_interruptions():
        print(interruption.raw_item.name)       # "send_email"
        print(interruption.raw_item.arguments)   # JSON string of args
        state.approve(interruption)              # or state.reject(interruption, message="...")

# Step 3: Resume from the paused point
resumed_result = await Runner.run(email_agent, state)
print(resumed_result.final_output)
```

Key parameters:
- `needs_approval=True` on `@function_tool()` -- marks the tool as requiring human sign-off; without this the tool runs immediately
- `result.interruptions` -- list of pending tool calls; empty if no approval-required tools were triggered
- `result.to_state()` -- captures the full resumable state so you can approve/reject and resume
- `state.approve(interruption)` -- green-lights the tool to execute on resume
- `state.reject(interruption, message="...")` -- blocks the tool and sends the message back to the agent so it can adapt
- `Runner.run(agent, state)` -- resumes from where the run paused

## The demos and what each shows

**Part 2 -- Basic Approval Flow (email agent):** Two tools are created: `draft_email` (no approval) and `send_email` (needs approval). The agent is asked to send an email to `alex@example.com` with subject "Meeting Tomorrow." The run pauses, `result.interruptions` shows the pending `send_email` call with its arguments, then the code approves and resumes. This proves that `needs_approval=True` stops the tool from executing and that the tool only fires after `state.approve()`. Emphasize on camera: the `draft_email` tool ran fine without pausing -- only `send_email` triggered the interruption.

**Part 3 -- Rejecting a Tool Call:** The same email agent is asked to send to `all-company@example.com` saying servers are down. This time the code rejects with the message `"All-company emails require manager approval. Please save as draft instead."` The agent receives that rejection message and adapts its response accordingly -- it does not crash. Emphasize on camera: the `message` parameter in `state.reject()` is what gives the agent context to recover gracefully. Without it, the agent has no idea what went wrong.

**Part 4 -- Threshold-Based Escalation (refund agent):** A `support_agent` has `lookup_order` (no approval) and `process_refund` (needs approval). The `handle_refund_request()` function uses an `auto_approve_threshold` of $50. Two test cases run: order ORD-001 ($15.99 phone case) is auto-approved because it is below threshold; order ORD-002 ($349.00 headphones) triggers manual review because it exceeds the threshold. The `while result.interruptions` loop handles cases where resuming might trigger additional approvals. Emphasize on camera: the SDK does not know about the $50 threshold -- that logic lives entirely in your code. The SDK just pauses every `needs_approval` call and lets you decide.

## Gotchas worth knowing before recording

- The `@function_tool(needs_approval=True)` decorator uses parentheses with the keyword argument -- `@function_tool` without parentheses will NOT set needs_approval and the tool will run without pausing. This is called out in the troubleshooting section and is the number one student mistake.
- `interruption.raw_item.arguments` is a JSON string, not a dict. The threshold demo parses it with `json.loads(interruption.raw_item.arguments)` before reading `args.get("amount")`. If you forget `json.loads`, you will get a string error on camera.
- The threshold demo uses `while result.interruptions` (not `if`) because resuming after approval could trigger another tool call that also needs approval. Mention this on camera -- it is a subtle but important production pattern.
- The rejection demo has a hardcoded `decision = "yes"` for the manual review path to keep the demo non-interactive. Mention this is simulated -- in production you would prompt a human via a UI or messaging system.
- `result.to_state()` must be called only when `result.interruptions` is non-empty. Calling it on a completed run and then trying `state.get_interruptions()` will return an empty list and confuse students.
- The `send_email` and `process_refund` tools are stubs (they return strings, not real API calls). Make clear on camera these are simulated -- the HITL pattern is the point, not the email/refund implementation.

## How it connects to adjacent notebooks

**Before (M07B -- Prompt Injection & Tool Safety):** That notebook established the read-vs-write tool distinction and why write actions need stronger controls. Call back to it on camera: "In M07B we talked about write tools being dangerous -- HITL is how you add the final safety gate before a write tool actually executes." The concept of confirming dangerous actions from M07B is directly implemented here.

**After (M07D -- Tracing & Observability):** The next notebook shows how to read traces in the OpenAI dashboard to inspect tool calls, handoffs, and debug bad runs. Bridge on camera: "Now that you have guardrails, injection defenses, and human approval gates, the next question is: how do you see what actually happened inside a run? That is tracing."

**Capstone connection (M07E):** The Module 7 capstone builds a production customer service agent that combines guardrails, prompt injection awareness, and HITL for refund approvals. The threshold-based escalation pattern from Part 4 is directly reused there. Worth mentioning on camera: "Everything you just learned here -- the approval flow, the threshold logic -- you will wire into a full production agent in the capstone."
