# cc_nb24.md — 24_Human_In_The_Loop.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/24_Human_In_The_Loop.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add `result.to_state()` to the Part 1 conceptual overview

4 of 5 reviewers (Anthropic, OpenAI, Gemini, Grok) flagged this as transfer-blocking. Part 1's four-step flow makes `state` sound like an ambient object, then Part 2 opens with `state = result.to_state()` with no warning — the exact line students need to copy into their own code.

**Find** (in Part 1, "How HITL Works", step 3):
```
3. You inspect what the agent wants to do — call `state.approve(interruption)` or `state.reject(interruption)`.
```

**Replace with:**
```
3. You convert the run result with `state = result.to_state()`, then call `state.approve(interruption)` or `state.reject(interruption)`.
```

---

## Change 2: Add comment that `arguments` is a JSON string, not a dict

3 of 5 reviewers (Anthropic, Gemini, DeepSeek) flagged this as transfer-blocking. `arguments` prints like a dict in Part 2, then suddenly needs `json.loads()` in Part 4 — students who write `interruption.raw_item.arguments["amount"]` get a confusing `TypeError`.

Add the following inline comment above the `json.loads()` line in Part 4's `handle_refund_request`:

```python
# arguments comes back as a JSON string — parse it to read individual fields
args = json.loads(interruption.raw_item.arguments)
```

Also add a brief note after the `arguments` print in Part 2's Step 1 inspection block:

```python
# Note: arguments is a JSON string — use json.loads() to inspect specific fields
```

---

## Change 3: Explain `raw_item` and what `interruption` contains

3 of 5 reviewers (Anthropic, Grok, DeepSeek) flagged this. Students wonder why `raw_item` is needed when they might expect `interruption.name` to work directly.

Add the following sentence after the interruption inspection block in Part 2, Step 1:

```
Each interruption wraps a `raw_item` — that's the pending tool call, with `.name` and `.arguments` (as a JSON string) available for inspection.
```

---

## Change 4: Name the reusable pattern in Part 2 "Why This Works"

1 reviewer (OpenAI) flagged this as transfer-blocking; Grok raised the related concern that the 40-line Part 4 helper makes the minimal pattern invisible. Without a named template, students follow the demo but can't extract what to copy.

Add the following sentence to the Part 2 "Why This Works" cell:

```
This is the reusable pattern: run until interrupted, convert to `state`, approve or reject each interruption, then resume with that same state.
```

---

## Change 5: Explain the `while` loop reasoning in Part 4

2 of 5 reviewers (Anthropic, Gemini) flagged this. The comment says "additional approvals" but the demo only shows one, so students don't know whether to use `if` or `while` in their own code.

**Find** (in Part 4, `handle_refund_request`):
```python
# Loop handles cases where resuming triggers additional approvals
while result.interruptions:
```

**Replace with:**
```python
# Use a while loop because the agent may call another approval-required tool after resuming
# — e.g., agent looks up the order, then asks to refund, then asks to send a confirmation email
while result.interruptions:
```

---

## Change 6: Add SDK-level vs instruction-level distinction in Part 2

1 reviewer (OpenAI) flagged this. Coming from Notebook 23's instruction-based confirmation pattern, students see this as "another way to do the same thing" rather than a meaningful enforcement upgrade.

Add the following sentence to the Part 2 opening:

```
This matters because approval now happens in the SDK — the tool cannot run just because the agent ignored its instructions.
```

---

## Change 7: Tell students what line to replace in production (Part 4)

1 reviewer (OpenAI) flagged this as transfer-blocking. The hardcoded `decision = "yes"` has a comment saying "In production, prompt a human" but students don't know which specific line to replace.

Add the following sentence near the hardcoded `decision = "yes"` in Part 4:

```
In your own app, this is the line you'd replace with a real approval source — a button click in a UI, a Slack reply, or a CLI `input()` prompt.
```

---

## Change 8: Clarify that the threshold is developer-defined, not SDK-provided

1 reviewer (OpenAI) flagged this. Beginners reading "threshold" and "escalate" may think these are SDK concepts rather than Python rules they write themselves.

Add the following sentence to the Part 4 intro:

```
The SDK does not choose this threshold for you — you write the rule in Python, such as `if amount > 50: # require approval`.
```

---

## Change 9: Add a "Why This Works" beat after the Part 3 rejection demo

1 reviewer (Grok) flagged this. There is no closing cell after the rejection demo; students see the agent adapt gracefully but don't understand why rejection matters in a real project.

Add the following markdown sentence immediately after the Part 3 rejection demo output:

```
Rejection is useful because the agent receives context it can use to adapt instead of failing or retrying the same forbidden action.
```
