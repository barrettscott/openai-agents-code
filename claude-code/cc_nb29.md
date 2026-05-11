# cc_nb29.md — 29_Capstone_4_MCP_Assistant.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/29_Capstone_4_MCP_Assistant.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add Server-Level Approval explanation markdown cell before Phase 2 demo

5/5 reviewers flagged this as the top finding. `require_approval` and the interruption loop are introduced entirely through code with no prose setup. This is the capstone's headline new concept and students coming from Notebook 28 have never seen any of these APIs.

Add a new markdown cell immediately before the Phase 2 main code cell:

```markdown
### 🔐 Server-Level Approval

`require_approval` lets the MCP server pause the run before specific tools execute. The policy is a dict with two keys:
- `"always"`: tools that always require human approval (write, delete)  
- `"never"`: tools that auto-approve (read, list)

Anything not listed runs without approval by default.

When the agent calls a protected tool, `Runner.run()` returns with `result.interruptions` populated. The reusable pattern:

```python
while result.interruptions:
    state = result.to_state()
    for interruption in state.get_interruptions():
        # inspect, then approve or reject
        state.approve(interruption)  # or state.reject(interruption, message="...")
    result = await Runner.run(agent, state)
```
```

---

## Change 2: Pull "saving is a side effect" beat out of a code comment into markdown

3 reviewers (Anthropic, OpenAI, Grok) flagged this. The teaching beat is buried in a confusingly worded comment. Students skip it or misread it. This is an outline-backed point: "Saving results is a side effect — handle it with the same care as write actions."

Add a new markdown cell between Task 1 and Task 2 in the Phase 2 section:

```markdown
> Saving results is still a write — `write_file` triggers the same approval flow as any other write action. A "harmless" save shouldn't get an exemption.
```

Then remove or simplify the existing code comment that says:
```
# Note: saving to a file is a write side effect — in production,
# auto-approving bypasses the same controls you just built in Task 1.
```

Replace that comment with:
```
# Task 2: Web research + save (write approval required — same flow as Task 1)
```

---

## Change 3: Split the Phase 2 monolithic cell at the Task 1 / Task 2 boundary

4 reviewers (Anthropic, OpenAI, Grok, DeepSeek) flagged the single ~90-line cell as burying the lesson. The approval loop pattern repeats — that's the pedagogical point — but students need visual separation to recognize it as a pattern rather than a wall of code.

Split the large Phase 2 code cell into two code cells at the Task 1 / Task 2 boundary. Insert a markdown cell between them:

```markdown
#### Task 2: Web Research + Save

Now let's combine web fetch with a file write — the approval flow is the same pattern we just used for Task 1.
```

Keep the `async with` block wrapping both tasks (i.e., both code cells are still nested inside the server context if that's how the demo is structured, or restructure with the servers opened once and reused).

> ⚠️ VERIFY: Check whether the `async with MCPServerStdio(...)` blocks wrap both tasks in one scope. If so, the split needs to keep both code cells inside a shared context manager, or the servers need to be opened once and passed in. Adjust accordingly.

---

## Change 4: Standardize the two approval loops

1 reviewer (Gemini), but pairs naturally with Change 3 (splitting the cell makes the structural difference visible). The two loops handle the same logic but print different info. Standardize both to the more informative version: print tool name + arguments, accept both `"yes"` and `"y"`.

> ⚠️ VERIFY: Inspect both `while result.interruptions:` loops and confirm which is more informative (prints arguments, accepts `"y"`). Apply that version to whichever loop is currently less complete.

---

## Change 5: Add approval loop template and concrete Option B to Practice Exercise

2 reviewers (DeepSeek, Grok) flagged that the exercise gives no reminder of the method names and no loop skeleton. Grok also noted Option B is too vague. Delivery notes call for "examples so students do not stall."

**Find:**
```
# TODO: Design a task that exercises filesystem + fetch + your new server
#        Handle any write approvals in the loop
```

**Replace with:**
```
# TODO: Design a task that exercises filesystem + fetch + your new server
#        Handle any write approvals using the pattern below:
#
# Example approval loop (reuse this pattern):
# while result.interruptions:
#     state = result.to_state()
#     for interruption in state.get_interruptions():
#         print(f"Approval needed: {interruption.raw_item.name}")
#         print(f"Arguments: {interruption.raw_item.arguments}")
#         decision = input("Approve? (yes/no): ").strip().lower()
#         if decision in ["yes", "y"]:
#             state.approve(interruption)
#         else:
#             state.reject(interruption, message="User rejected this operation.")
#     result = await Runner.run(assistant, state)
```

**Find:**
```
# Option B: Another fetch/search approach
# command: "uvx", args: ["mcp-server-fetch"] with different instructions
```

**Replace with:**
```
# Option B: Memory server — lets the agent store and retrieve persistent notes
# command: "npx", args: ["-y", "@modelcontextprotocol/server-memory"]
```
