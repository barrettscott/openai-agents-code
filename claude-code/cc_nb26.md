# cc_nb26.md — 26_Capstone_3_Customer_Service.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/26_Capstone_3_Customer_Service.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Delete the duplicate guardrail code cell in Phase 2

3 of 5 reviewers (Grok, DeepSeek, Gemini) flagged this as a demo-breaking blocker. There are two consecutive code cells defining `SupportTopicCheck` and `support_topic_guardrail`. The second version adds type hints (`RunContextWrapper`, `TResponseInputItem`) that are not imported in the setup cell — running it causes a `NameError`.

Delete the second (typed) code cell. Keep the first version (without the unimported type hints), which runs correctly with the existing imports.

> ⚠️ VERIFY: Confirm which cell is first and which is second before deleting. The cell to keep is the one whose signature reads `async def support_topic_guardrail(ctx, agent: Agent, input)` or similar without `RunContextWrapper`/`TResponseInputItem`. The cell to delete contains those two unimported types in the signature.

---

## Change 2: Delete the duplicate pipeline function cell and reconcile the threshold

3 of 5 reviewers (Anthropic, Gemini, DeepSeek) flagged this as a demo-breaking blocker. There are two consecutive code cells defining `handle_customer_message` — one with `AUTO_APPROVE_THRESHOLD = 100.0` and an `agent=None` parameter, one with `AUTO_APPROVE_THRESHOLD = 50.0` and no `agent` parameter. The pre-demo markdown references both `$100` and `$50`. The exercise references `agent=` which only exists in the overwritten first version.

Recommended resolution:
1. Keep the version with `agent=None` in the signature (needed by Exercise 1 — see Change 3).
2. Set `AUTO_APPROVE_THRESHOLD = 100.0` (the demo refund is $149.99, which makes the pause meaningful).
3. Delete the other cell.
4. Update both pre-demo markdown cells to consistently say "$100 threshold."

> ⚠️ VERIFY: Confirm the exact threshold value in both code cells and both markdown cells before editing, to ensure all four references are reconciled to the same value.

---

## Change 3: Fix the `handle_customer_message` signature so Exercise 1 is completable

2 of 5 reviewers (Gemini, OpenAI) flagged this as transfer-blocking. Exercise 1 asks students to pass a custom agent to the pipeline function, but the active function signature has no `agent` parameter.

Ensure the kept `handle_customer_message` function (from Change 2) has this signature:

```python
async def handle_customer_message(message, session, auto_approve=True, agent=None):
```

At the top of the function body, add:

```python
run_agent = agent or support_agent
```

Replace the two `Runner.run(support_agent, ...)` calls inside the function with `Runner.run(run_agent, ...)`.

Also add the following orienting sentence before the Practice Exercises section:

```
A useful extension pattern is to parameterize your orchestration function with `agent`, so you can swap in upgraded versions without rewriting the approval flow.
```

---

## Change 4: Replace the vague tracing callout with a concrete inspection checklist

2 of 5 reviewers (Anthropic, Grok) flagged this as transfer-blocking. Lesson 25 trained students to read traces with checklists; this callout says "inspect" with nothing concrete to find, and the capstone is the natural place to reinforce production tracing as a habit.

**Find** (in the "Tracing" callout after Phase 4):
```
Every Runner.run() call in this pipeline creates a trace automatically. Open the OpenAI dashboard to inspect the guardrail check, tool calls, and approval flow in a single trace view.
```

**Replace with:**
```
Every `Runner.run()` call in this pipeline creates a trace automatically. Open `https://platform.openai.com/traces` and look for:
- The `TopicChecker` guardrail span — shows why an off-topic request was blocked before reaching the agent
- The `process_refund` interruption span — marks the exact pause point where the SDK stopped for approval
- The resumed run after approval — appears as a continuation of the same trace
- Token counts across turns — watch how multi-turn sessions grow
```

---

## Change 5: Add Phase 2 header and orientation sentence

1 reviewer (Anthropic) flagged this; the delivery notes also call out the missing Phase 2 heading. The notebook jumps from the Phase 1 security note directly into `SupportTopicCheck` code with no orientation.

Add a new markdown cell before the guardrail code (before Change 1's kept cell):

```markdown
## 🛡️ Phase 2: Block Off-Topic Requests

A guardrail agent screens the message before it reaches the support agent — non-support requests get rejected up front.
```

---

## Change 6: Explain why an agent (not a keyword check) is used for the guardrail

1 reviewer (Gemini) flagged this as transfer-blocking. Students wonder why running a whole agent inside a guardrail is worth the cost.

Add the following sentence before the Phase 2 guardrail code cell (can be the second sentence in the Phase 2 header cell from Change 5):

```
A keyword check would be brittle; a dedicated classifier agent handles typos, paraphrasing, and conversational language more reliably.
```

---

## Change 7: Point the student to watch Turn 2's session payoff

2 of 5 reviewers (Anthropic, OpenAI) flagged this. Turn 2 is the moment sessions prove their value (the customer doesn't repeat the order ID), but nothing directs student attention to it.

Add the following sentence above the multi-turn conversation demo cell:

```
Watch turn 2 — the customer doesn't repeat the order ID. The session is what lets the agent carry ORD-001 forward, so customers don't restate context every turn.
```

---

## Change 8: Explain the interruption/state machinery above the `while` loop

1 reviewer (Grok) flagged this as transfer-blocking. Students see refunds trigger an approval prompt but don't understand how the SDK pause/resume mechanism works — they can't adapt it to their own tools.

Add the following sentence immediately before the `while result.interruptions:` block in `handle_customer_message`:

```
# The SDK pauses execution and populates result.interruptions when it hits a tool marked needs_approval=True.
# We convert the result to a mutable state, decide whether to approve or reject, then resume the run.
```

---

## Change 9: Add comment explaining what `needs_approval=True` does mechanically

2 of 5 reviewers (OpenAI, Grok) flagged this. Students see the decorator flag but don't know what the SDK does when it's set, and don't have the transferable rule for which tools should use it.

Add the following comments above the `process_refund` definition in Phase 1:

```python
# `needs_approval=True` tells the SDK to pause before executing this tool
# and populate `result.interruptions` (handled in the pipeline below).
# General rule: read-only tools auto-run; write actions that change money, records, or external systems pause for approval.
@function_tool(needs_approval=True)
def process_refund(order_id: str, amount: float, reason: str) -> str:
```

---

## Change 10: Add orchestration layer explanation to Phase 4 intro

1 reviewer (OpenAI) flagged this as transfer-blocking. "Orchestration layer" sounds like a named SDK thing; students need to understand it's a plain Python wrapper pattern.

Add the following sentence under the Phase 4 header:

```
Focus on one idea here: `handle_customer_message()` is a plain Python wrapper around `Runner.run()` where you add app-specific rules — approval thresholds, logging, and fallback behavior.
```

---

## Change 11: Warn students about the `input()` prompt at Turn 2

1 reviewer (DeepSeek) flagged this. `auto_approve=True` reads as "everything automatic," but the demo pauses at an `input()` prompt — students may hesitate, not knowing what to type.

Add the following sentence to the pre-demo markdown:

```
You'll see an `input()` prompt at Turn 2 — type `yes` to approve the refund.
```

---

## Change 12: Fix Key Takeaways cell type

The Key Takeaways cell (`## 🎯 Key Takeaways`) is currently typed as a **code** cell — it renders as runnable Python rather than formatted markdown. Convert it to **markdown**.

> ⚠️ VERIFY: Confirm the cell containing `## 🎯 Key Takeaways` is indeed `cell_type: "code"` before converting. If it's already markdown, skip this change.
